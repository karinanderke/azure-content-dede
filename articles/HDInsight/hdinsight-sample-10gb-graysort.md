<properties
	pageTitle="Das 10-GB GraySort-Beispiel | Azure"
	description="Erfahren Sie mehr darüber, wie Sie ein allgemeines GraySort für eine sehr große Datenmenge, normalerweise mindestens 100 TB, auf Hadoop mit HDInsights mithilfe von Azure PowerShell ausführen können."
	editor="cgronlun"
	manager="paulettm"
	services="hdinsight"
	documentationCenter=""
	authors="bradsev"/>

<tags
	ms.service="hdinsight"
	ms.workload="big-data"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="03/30/2015" 
	ms.author="bradsev"/>

# Das 10 GB GraySort-Beispiel mit Hadoop in HDInsight

In diesem Thema führen Sie ein allgemeines GraySort Hadoop MapReduce-Programm mithilfe von Azure PowerShell in Azure HDInsight aus. GraySort ist ein Sortierungs-Benchmark, der als Metrik die Sortiergeschwindigkeit (TB/Minute) verwendet, die beim Sortieren sehr großer Datenmengen erreicht wird (normalerweise mindestens 100 TB).

In diesem Beispiel werden bescheidene 10 GB an Daten verwendet, um eine zügige Ausführung zu ermöglichen. Die Anwendung verwendet die MapReduce-Anwendungen von Owen O'Malley und Arun Murthy, die den jährlichen Allzweck-Terabyte-Sortierungs-Benchmark ("daytona") 2009 mit einem Durchsatz von 0,578 TB/min gewonnen haben (100 TB in 173 Minuten). Weitere Informationen zu diesem und anderen Sortier-Benchmarks finden Sie unter [Sortbenchmark](http://sortbenchmark.org/).

Dieses Beispiel verwendet drei Sätze von MapReduce-Programmen:

1. **TeraGen** ist ein MapReduce, das Sie zum Generieren der zu sortierenden Datensätze verwenden können.
2. **TeraSort** prüft die Eingangsdaten und verwendet MapReduce, um die Daten in eine Gesamtreihenfolge zu bringen. TeraSort ist eine Standardsortierung mit MapReduce-Funktionen, mit Ausnahme eines eigenen Partitionierers, der eine sortierte Liste mit N-1 geprüften Schlüsseln verwendet, um den Schlüsselbereich für die einzelnen Reduce-Vorgänge zu definieren. Insbesondere werden alle Schlüssel mit einer Probe[i-1] <= Schlüssel < Probe[i] an "reduce i" gesendet. Dadurch wird sichergestellt, dass die Ausgaben von "reduce i" alle kleiner als die Ausgabe von "reduce i+1" sind.
3. **TeraValidate** ist ein MapReduce-Programm, das die globale Sortierung der Ausgabe prüft. Es erstellt eine Map pro Datei im Ausgabeverzeichnis, und jede Map stellt sicher, dass jeder Schlüssel kleiner oder gleich dem vorherigen Schlüssel ist. Die Map-Funktion generiert außerdem Einträge für den ersten und letzten Schlüssel der einzelnen Dateien, und die Reduce-Funktion stellt sicher, dass der erste Schlüssel von Datei i größer als der letzte Schlüssel von Datei i-1 ist. Probleme werden als Ausgabe der Reduce-Funktion zusammen mit den Schlüsseln gemeldet, die sich nicht in der richtigen Reihenfolge befinden.

Ein- und Ausgabeformate der drei Anwendungen lesen und schreiben die Textdateien jeweils im richtigen Format. Der "replication"-Wert der Reduce-Funktion ist auf 1 anstelle des Standardwerts 3 festgelegt, da die Ausgabedaten für den Benchmark-Wettbewerb nicht auf verschiedene Knoten repliziert werden müssen.


**Sie erfahren Folgendes:** * Verwenden von Azure PowerShell zum Ausführen einer Reihe von MapReduce-Programmen in Azure HDInsight * Aussehen eines MapReduce-Programms in Java


**Voraussetzungen:**

- Sie benötigen ein Azure-Konto. Hinweise zum Erstellen eines Kontos finden Sie auf der Seite [Einen Monat kostenlos testen](http://azure.microsoft.com/pricing/free-trial/).

- Sie benötigen ein bereitgestelltes HDInsight-Cluster. Anweisungen zu den verschiedenen Optionen beim Erstellen eines solchen Clusters finden Sie unter [Bereitstellen eines HDInsight-Clusters](hdinsight-provision-clusters.md).

- Sie müssen Azure PowerShell installiert und für die Verwendung mit Ihrem Konto konfiguriert haben. Anweisungen zur entsprechenden Vorgehensweise finden Sie unter [Installieren und Konfigurieren von Azure PowerShell][powershell-install-configure].


##Ausführen des Beispiels mit Azure PowerShell

Für das Beispiel müssen drei Aufgaben ausgeführt werden, die den einzelnen MapReduce-Programmen entsprechen, die in der Einführung vorgestellt wurden:

1. Führen Sie den **TeraGen** MapReduce-Auftrag aus, um die zu sortierenden Daten zu generieren.
2. Sortieren der Daten mit dem **TeraSort** MapReduce-Auftrag.
3. Prüfen der korrekten Sortierung der Daten mit dem **TeraValidate** MapReduce-Auftrag.


**So führen Sie das TeraGen-Programm aus**

1. Öffnen Sie Azure PowerShell. Anweisungen zum Öffnen des Azure PowerShell-Konsolenfensters finden Sie unter [Installieren und Konfigurieren von Azure PowerShell][powershell-install-configure].
2. Legen Sie die zwei Variablen in den folgenden Befehlen fest, und führen Sie die Befehle aus:

		# Provide the Azure subscription name and the HDInsight cluster name
		$subscriptionName = "myAzureSubscriptionName"
		$clusterName = "myClusterName"

4. Führen Sie den folgenden Befehl aus, um eine MapReduce-Jobdefinition zu erstellen:

		# Create a MapReduce job definition for the TeraGen MapReduce program
		$teragen = New-AzureHDInsightMapReduceJobDefinition -JarFile "/example/jars/hadoop-mapreduce-examples.jar" -ClassName "teragen" -Arguments "-Dmapred.map.tasks=50", "100000000", "/example/data/10GB-sort-input"

	Das Argument *"-Dmapred.map.tasks=50"* gibt an, dass für die Ausführung des Jobs 50 Maps erstellt werden sollen. Das Argument *100000000* gibt die Menge der zu generierenden Daten an. Das letzte Argument */example/data/10GB-sort-input* gibt an, in welchem Ordner die Ausgabedaten gespeichert werden sollen (das Eingangsverzeichnis für die folgende Sortierphase).

5. Führen Sie die folgenden Befehle aus, um den Auftrag zu übermitteln, auf den Abschluss des Auftrags zu warten und dann den Standardfehler auszugeben:

		# Run the TeraGen MapReduce job
		# Wait for the job to finish
		# Print output and standard error file of the MapReduce job
		Select-AzureSubscription $subscriptionName
		$teragen | Start-AzureHDInsightJob -Cluster $clustername | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 | Get-AzureHDInsightJobOutput -Cluster $clustername -StandardError


**So führen Sie das TeraSort-Programm aus**

1. Öffnen Sie Azure PowerShell.
2. Legen Sie die zwei Variablen in den folgenden Befehlen fest, und führen Sie die Befehle aus:

		# Provide the Azure subscription name and the HDInsight cluster name
		$subscriptionName = "myAzureSubscriptionName"
		$clusterName = "myClusterName"

3. Führen Sie den folgenden Befehl aus, um den MapReduce-Job zu definieren:

		# Create a MapReduce job definition for the TeraSort MapReduce program
		$terasort = New-AzureHDInsightMapReduceJobDefinition -JarFile "/example/jars/hadoop-mapreduce-examples.jar" -ClassName "terasort" -Arguments "-Dmapred.map.tasks=50", "-Dmapred.reduce.tasks=25", "/example/data/10GB-sort-input", "/example/data/10GB-sort-output"

	Das Argument *"-Dmapred.map.tasks=50"* gibt an, dass für die Ausführung des Jobs 50 Maps erstellt werden sollen. Das Argument *100000000* gibt die Menge der zu generierenden Daten an. Die letzten beiden Argumente geben Eingabe- und Ausgabeordner an.

4. Führen Sie den folgenden Befehl aus, um den Auftrag zu übermitteln, auf den Abschluss des Auftrags zu warten und dann den Standardfehler auszugeben:

		# Run the TeraSort MapReduce job
		# Wait for the job to finish
		# Print output and standard error file of the MapReduce job
		Select-AzureSubscription $subscriptionName
		$terasort | Start-AzureHDInsightJob -Cluster $clustername | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 | Get-AzureHDInsightJobOutput -Cluster $clustername -StandardError


**So führen Sie das TeraValidate-Programm aus**

1. Öffnen Sie Azure PowerShell.
2. Legen Sie die zwei Variablen in den folgenden Befehlen fest, und führen Sie die Befehle aus:

		# Provide the Azure subscription name and the HDInsight cluster name
		$subscriptionName = "myAzureSubscriptionName"
		$clusterName = "myClusterName"

3. Führen Sie den folgenden Befehl aus, um den MapReduce-Job zu definieren:

		#	Create a MapReduce job definition for the TeraValidate MapReduce program
		$teravalidate = New-AzureHDInsightMapReduceJobDefinition -JarFile "/example/jars/hadoop-mapreduce-examples.jar" -ClassName "teravalidate" -Arguments "-Dmapred.map.tasks=50", "-Dmapred.reduce.tasks=25", "/example/data/10GB-sort-output", "/example/data/10GB-sort-validate"

	Das Argument *"-Dmapred.map.tasks=50"* gibt an, dass für die Ausführung des Jobs 50 Maps erstellt werden sollen. Das Argument *-Dmapred.reduce.tasks=25* gibt an, dass für die Ausführung des Auftrags 25 Reduce-Tasks erstellt werden sollen. Die letzten beiden Argumente geben Eingabe- und Ausgabeordner an.


4. Führen Sie die folgenden Befehle aus, um den MapReduce-Auftrag zu übermitteln, auf den Abschluss des Auftrags zu warten und dann den Standardfehler auszugeben:

		# Run the TeraSort MapReduce job
		# Wait for the job to finish
		# Print output and standard error file of the MapReduce job
		Select-AzureSubscription $subscriptionName
		$teravalidate | Start-AzureHDInsightJob -Cluster $clustername | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 | Get-AzureHDInsightJobOutput -Cluster $clustername -StandardError


##Der Java-Code für das TeraSort MapReduce-Programm

Sie finden den Code für das TeraSort-MapReduce-Programm zu Ihrer Information in diesem Abschnitt.


	/**
	 * Licensed to the Apache Software Foundation (ASF) under one
	 * or more contributor license agreements.  See the NOTICE file
	 * distributed with this work for additional information
	 * regarding copyright ownership.  The ASF licenses this file
	 * to you under the Apache License, Version 2.0 (the
	 * "License"); you may not use this file except in compliance
	 * with the License.  You may obtain a copy of the License at
	 *
	 *     http://www.apache.org/licenses/LICENSE-2.0
	 *
	 * Unless required by applicable law or agreed to in writing, software
	 * distributed under the License is distributed on an "AS IS" BASIS,
	 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
	 * See the License for the specific language governing permissions and
	 * limitations under the License.
	 */

	package org.apache.hadoop.examples.terasort;

	import java.io.IOException;
	import java.io.PrintStream;
	import java.net.URI;
	import java.util.ArrayList;
	import java.util.List;

	import org.apache.commons.logging.Log;
	import org.apache.commons.logging.LogFactory;
	import org.apache.hadoop.conf.Configured;
	import org.apache.hadoop.filecache.DistributedCache;
	import org.apache.hadoop.fs.FileSystem;
	import org.apache.hadoop.fs.Path;
	import org.apache.hadoop.io.NullWritable;
	import org.apache.hadoop.io.SequenceFile;
	import org.apache.hadoop.io.Text;
	import org.apache.hadoop.mapred.FileOutputFormat;
	import org.apache.hadoop.mapred.JobClient;
	import org.apache.hadoop.mapred.JobConf;
	import org.apache.hadoop.mapred.Partitioner;
	import org.apache.hadoop.util.Tool;
	import org.apache.hadoop.util.ToolRunner;

	/**
	 * Generates the sampled split points, launches the job,
	 * and waits for it to finish.
	 * <p>
	 * To run the program:
	 * <b>bin/hadoop jar hadoop-examples-*.jar terasort in-dir out-dir</b>
	 */

	public class TeraSort extends Configured implements Tool {
	  private static final Log LOG = LogFactory.getLog(TeraSort.class);

	  /**
	   * A partitioner that splits text keys into roughly equal
	   * partitions in a global sorted order.
	   */

	  static class TotalOrderPartitioner implements Partitioner<Text,Text>{
	    private TrieNode trie;
	    private Text[] splitPoints;

	    /**
	     * A generic trie node
	     */
	    static abstract class TrieNode {
	      private int level;
	      TrieNode(int level) {
	        this.level = level;
	      }
	      abstract int findPartition(Text key);
	      abstract void print(PrintStream strm) throws IOException;
	      int getLevel() {
	        return level;
	      }
	    }

	    /**
	     * An inner trie node that contains 256 children based on the next
	     * character.
	     */
	    static class InnerTrieNode extends TrieNode {
	      private TrieNode[] child = new TrieNode[256];

	      InnerTrieNode(int level) {
	        super(level);
	      }
	      int findPartition(Text key) {
	        int level = getLevel();
	        if (key.getLength() <= level) {
	          return child[0].findPartition(key);
	        }
	        return child[key.getBytes()[level]].findPartition(key);
	      }
	      void setChild(int idx, TrieNode child) {
	        this.child[idx] = child;
	      }
	      void print(PrintStream strm) throws IOException {
	        for(int ch=0; ch < 255; ++ch) {
	          for(int i = 0; i < 2*getLevel(); ++i) {
	            strm.print(' ');
	          }
	          strm.print(ch);
	          strm.println(" ->");
	          if (child[ch] != null) {
	            child[ch].print(strm);
	          }
	        }
	      }
	    }

	    /**
	     * A leaf trie node that does string compares to figure out where the given
	     * key belongs between lower..upper.
	     */
	    static class LeafTrieNode extends TrieNode {
	      int lower;
	      int upper;
	      Text[] splitPoints;
	      LeafTrieNode(int level, Text[] splitPoints, int lower, int upper) {
	        super(level);
	        this.splitPoints = splitPoints;
	        this.lower = lower;
	        this.upper = upper;
	      }
	      int findPartition(Text key) {
	        for(int i=lower; i<upper; ++i) {
	          if (splitPoints[i].compareTo(key) >= 0) {
	            return i;
	          }
	        }
	        return upper;
	      }
	      void print(PrintStream strm) throws IOException {
	        for(int i = 0; i < 2*getLevel(); ++i) {
	          strm.print(' ');
	        }
	        strm.print(lower);
	        strm.print(", ");
	        strm.println(upper);
	      }
	    }


	    /**
	     * Read the cut points from the given sequence file.
	     * @param fs the file system
	     * @param p the path to read
	     * @param job the job config
	     * @return the strings to split the partitions on
	     * @throws IOException
	     */
	    private static Text[] readPartitions(FileSystem fs, Path p,
	                                         JobConf job) throws IOException {
	      SequenceFile.Reader reader = new SequenceFile.Reader(fs, p, job);
	      List<Text> parts = new ArrayList<Text>();
	      Text key = new Text();
	      NullWritable value = NullWritable.get();
	      while (reader.next(key, value)) {
	        parts.add(key);
	        key = new Text();
	      }
	      reader.close();
	      return parts.toArray(new Text[parts.size()]);  
	    }

	    /**
	     * Given a sorted set of cut points, build a trie that will find the correct
	     * partition quickly.
	     * @param splits the list of cut points
	     * @param lower the lower bound of partitions 0..numPartitions-1
	     * @param upper the upper bound of partitions 0..numPartitions-1
	     * @param prefix the prefix that we have already checked against
	     * @param maxDepth the maximum depth we will build a trie for
	     * @return the trie node that will divide the splits correctly
	     */
	    private static TrieNode buildTrie(Text[] splits, int lower, int upper,
	                                      Text prefix, int maxDepth) {
	      int depth = prefix.getLength();
	      if (depth >= maxDepth || lower == upper) {
	        return new LeafTrieNode(depth, splits, lower, upper);
	      }
	      InnerTrieNode result = new InnerTrieNode(depth);
	      Text trial = new Text(prefix);
	      // append an extra byte on to the prefix
	      trial.append(new byte[1], 0, 1);
	      int currentBound = lower;
	      for(int ch = 0; ch < 255; ++ch) {
	        trial.getBytes()[depth] = (byte) (ch + 1);
	        lower = currentBound;
	        while (currentBound < upper) {
	          if (splits[currentBound].compareTo(trial) >= 0) {
	            break;
	          }
	          currentBound += 1;
	        }
	        trial.getBytes()[depth] = (byte) ch;
	        result.child[ch] = buildTrie(splits, lower, currentBound, trial,
	                                     maxDepth);
	      }
	      // pick up the rest
	      trial.getBytes()[depth] = 127;
	      result.child[255] = buildTrie(splits, currentBound, upper, trial,
	                                    maxDepth);
	      return result;
	    }

	    public void configure(JobConf job) {
	      try {
	        FileSystem fs = FileSystem.getLocal(job);
	        Path partFile = new Path(TeraInputFormat.PARTITION_FILENAME);
	        splitPoints = readPartitions(fs, partFile, job);
	        trie = buildTrie(splitPoints, 0, splitPoints.length, new Text(), 2);
	      } catch (IOException ie) {
	        throw new IllegalArgumentException("can't read paritions file", ie);
	      }
	    }

	    public TotalOrderPartitioner() {
	    }

	    public int getPartition(Text key, Text value, int numPartitions) {
	      return trie.findPartition(key);
	    }

	  }

	  public int run(String[] args) throws Exception {
	    LOG.info("starting");
	    JobConf job = (JobConf) getConf();
	    Path inputDir = new Path(args[0]);
	    inputDir = inputDir.makeQualified(inputDir.getFileSystem(job));
	    Path partitionFile = new Path(inputDir, TeraInputFormat.PARTITION_FILENAME);
	    URI partitionUri = new URI(partitionFile.toString() +
	                               "#" + TeraInputFormat.PARTITION_FILENAME);
	    TeraInputFormat.setInputPaths(job, new Path(args[0]));
	    FileOutputFormat.setOutputPath(job, new Path(args[1]));
	    job.setJobName("TeraSort");
	    job.setJarByClass(TeraSort.class);
	    job.setOutputKeyClass(Text.class);
	    job.setOutputValueClass(Text.class);
	    job.setInputFormat(TeraInputFormat.class);
	    job.setOutputFormat(TeraOutputFormat.class);
	    job.setPartitionerClass(TotalOrderPartitioner.class);
	    TeraInputFormat.writePartitionFile(job, partitionFile);
	    DistributedCache.addCacheFile(partitionUri, job);
	    DistributedCache.createSymlink(job);
	    job.setInt("dfs.replication", 1);
	    TeraOutputFormat.setFinalSync(job, true);
	    JobClient.runJob(job);
	    LOG.info("done");
	    return 0;
	  }

	  /**
	   * @param args
	   */

	  public static void main(String[] args) throws Exception {
	    int res = ToolRunner.run(new JobConf(), new TeraSort(), args);
	    System.exit(res);
	  }

	}


##Zusammenfassung

In diesem Beispiel haben Sie gesehen, wie Sie eine Reihe von MapReduce-Aufträgen mit Azure HDInsight ausführen können, wobei die Ausgabe eines Auftrags als Eingabe des nächsten Auftrags in der Reihe verwendet wird.

##Nächste Schritte

Lernprogramme, in denen Sie durch das Ausführen anderer Beispiele geführt und Anweisungen zum Ausführen von Pig-, Hive- und MapReduce-Aufträgen in Azure HDInsight mit Azure PowerShell bereitgestellt werden, finden Sie in den folgenden Themen:

* [Erste Schritte mit Azure HDInsight][hdinsight-get-started]
* [Beispiel: Pi-Schätzung][hdinsight-sample-pi-estimator]
* [Beispiel: Wortzählung][hdinsight-sample-wordcount]
* [Beispiel: C#-Streaming][hdinsight-sample-csharp-streaming]
* [Verwenden von Pig mit HDInsight][hdinsight-use-pig]
* [Verwenden von Hive mit HDInsight][hdinsight-use-hive]
* [Azure HDInsight SDK-Dokumentation][hdinsight-sdk-documentation]

[hdinsight-sdk-documentation]: http://msdnstage.redmond.corp.microsoft.com/library/dn479185.aspx


[Powershell-install-configure]: ../install-configure-powershell.md

[hdinsight-get-started]: ../hdinsight-get-started.md

[hdinsight-samples]: hdinsight-run-samples.md
[hdinsight-sample-10gb-graysort]: hdinsight-sample-10gb-graysort.md
[hdinsight-sample-csharp-streaming]: hdinsight-sample-csharp-streaming.md
[hdinsight-sample-pi-estimator]: hdinsight-sample-pi-estimator.md
[hdinsight-sample-wordcount]: hdinsight-sample-wordcount.md

[hdinsight-use-hive]: hdinsight-use-hive.md
[hdinsight-use-pig]: hdinsight-use-pig.md

<!--HONumber=54--> 