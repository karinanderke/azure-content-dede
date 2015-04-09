<properties
   pageTitle="Verwenden von Hadoop Hive in HDInsight | Azure"
   description="Verwenden von Hadoop Hive in HDInsight über PowerShell."
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.devlang=""
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="02/18/2015"
   ms.author="larryfr"/>

# Ausführen von Hive-Abfragen mit PowerShell

[AZURE.INCLUDE [hive-selector](../includes/hdinsight-selector-use-hive.md)]

Dieses Dokument enthält ein Beispiel zur Verwendung von PowerShell zum Ausführen von Hive-Abfragen auf einem Hadoop-Cluster in HDInsight.

> [AZURE.NOTE] Dieses Dokument bietet keine detaillierte Beschreibung dazu, wie die in diesem Beispiel verwendeten HiveQL-Anweisungen vorgehen. Informationen zu der in diesem Beispiel verwendeten HiveQL finden Sie unter <a href="../hdinsight-use-hive/" target="_blank">Verwenden von Hive mit Hadoop unter HDInsight</a>.


## <a id="prereq"></a>Voraussetzungen

Damit Sie die in dieser Artikel aufgeführten Schritte ausführen können, benötigen Sie Folgendes:

* Einen Azure HDInsight-Cluster (Hadoop in HDInsight), der auf Windows oder Linux basiert

* <a href="http://azure.microsoft.com/documentation/articles/install-configure-powershell/" target="_blank">Azure PowerShell</a>


## <a id="powershell"></a>Ausführen von Hive-Abfragen mit PowerShell

Azure PowerShell stellt *cmdlets* bereit, mit denen Sie Hive-Abfragen in HDInsight remote ausführen können. Intern erfolgt dies durch REST-Aufrufe von <a href="https://cwiki.apache.org/confluence/display/Hive/WebHCat" target="_blank">WebHCat</a> (früher als "Templeton" bezeichnet), das auf dem HDInsight-Cluster ausgeführt wird.

Die folgenden Cmdlets werden zum Ausführen der Hive-Abfragen auf einem HDInsight-Remotecluster verwendet.

* **Add-AzureAccount**: authentifiziert PowerShell für Ihr Azure-Abonnement

* **New-AzureHDInsightHiveJobDefinition** erstellt mithilfe der angegebenen HiveQL-Anweisungen eine neue *job definition*.

* **Start-AzureHDInsightJob** sendet die Auftragsdefinition an HDInsight, startet den Auftrag und gibt ein  *job*-Objekt zurück, mit dem der Status des Auftrags geprüft werden kann.

* **Wait-AzureHDInsightJob**: verwendet das job-Objekt, um den Status des Auftrags zu prüfen Es wird gewartet, bis der Auftrag abgeschlossen oder die Wartezeit überschritten ist.

* **Get-AzureHDInsightJobOutput**: wird zum Abrufen der Ausgabe des Auftrags verwendet

* **Invoke-Hive**: wird zum Ausführen von HiveQL-Anweisungen und abgeschlossenen Blöcken verwendet Anschließend werden die Ergebnisse zurückgegeben.

* **Use-AzureHDInsightCluster**: legt den aktuellen Cluster für die Verwendung mit dem Befehl **Invoke-Hive** fest

Die folgenden Schritte veranschaulichen, wie diese Cmdlets zum Ausführen eines Auftrags auf dem HDInsight-Cluster verwendet werden. 

1. Speichern Sie den folgenden Code mithilfe eines Editors als **hivejob.ps1**. Ersetzen Sie **CLUSTERNAME** durch den Namen des HDInsight-Clusters.

		#Login to your Azure subscription
		# Is there an active Azure subscription?
		$sub = Get-AzureSubscription -ErrorAction SilentlyContinue
		if(-not($sub))
		{
		    Add-AzureAccount
		}
		
		#Specify the cluster name
		$clusterName = "CLUSTERNAME" 
		
		#HiveQL
		$queryString = "DROP TABLE log4jLogs;" +
				       "CREATE EXTERNAL TABLE log4jLogs(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' STORED AS TEXTFILE LOCATION 'wasb:///example/data/';" +
				       "SELECT t4 AS sev, COUNT(*) AS cnt FROM log4jLogs WHERE t4 = '[ERROR]' GROUP BY t4;"
		
		#Create an HDInsight Hive job definition
		$hiveJobDefinition = New-AzureHDInsightHiveJobDefinition -Query $queryString
		
		#Submit the job to the cluster
		Write-Host "Start the Hive job..." -ForegroundColor Green
		$hiveJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $hiveJobDefinition
		
		#Wait for the Hive job to complete
		Write-Host "Wait for the job to complete..." -ForegroundColor Green
		Wait-AzureHDInsightJob -Job $hiveJob -WaitTimeoutInSeconds 3600
		
		# Print the output
		Write-Host "Display the standard output..." -ForegroundColor Green
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $hiveJob.JobId -StandardOutput

2. Öffnen Sie eine neue **Microsoft Azure PowerShell**-Eingabeaufforderung. Navigieren Sie zum Speicherort der Datei **hivejob.ps1**, und verwenden Sie Folgendes zum Ausführen des Skripts.

		.\hivejob.ps1

7. Nachdem der Auftrag abgeschlossen ist, muss er Informationen zurückgeben, die den folgenden ähneln.

		Display the standard output...
		[ERROR]	3

4. Wie bereits erwähnt, kann **Invoke-Hive** dazu verwendet werden, um eine Abfrage auszuführen und auf die Antwort zu warten. Verwenden Sie die folgenden Befehle, wobei Sie **CLUSTERNAME** durch den Namen des Clusters ersetzen.

		Use-AzureHDInsightCluster CLUSTERNAME
		Invoke-Hive -Query @"
		CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) STORED AS ORC;
		INSERT OVERWRITE TABLE errorLogs SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]';
		SELECT * FROM errorLogs;
		"@

	Die Ausgabe sieht dann wie folgt aus.

		2012-02-03	18:35:34	SampleClass0	[ERROR]	incorrect	id	
		2012-02-03	18:55:54	SampleClass1	[ERROR]	incorrect	id	
		2012-02-03	19:25:27	SampleClass4	[ERROR]	incorrect	id

	> [AZURE.NOTE] Für längere HiveQL-Abfragen können Sie PowerShell Here-Zeichenfolgen oder HiveQL-Skriptdateien verwenden. Der folgende Codeausschnitt zeigt, wie Sie eine HiveQL-Skriptdatei mit dem  *Invoke-Hive*-Cmdlet ausführen können. Die HiveQL-Skriptdatei muss auf WASB hochgeladen werden.
	>
	> `Invoke-Hive -File "wasb://<ContainerName>@<StorageAccountName>/<Path>/query.hql"`
	>
	> Weitere Informationen zu Here-Strings erhalten Sie unter <a href="http://technet.microsoft.com/library/ee692792.aspx" target="_blank">Verwenden von Windows PowerShell-Here-Strings</a>.

## <a id="troubleshooting"></a>Problembehandlung

Wenn nach Abschluss des Auftrags keine Informationen zurückgegeben werden, ist während der Verarbeitung möglicherweise ein Fehler aufgetreten. Um Fehlerinformationen für diesen Auftrag anzuzeigen, fügen Sie Folgendes am Ende der Datei **hivejob.ps1** hinzu. Anschließend speichern Sie die Datei und führen sie erneut aus.

	# Print the output of the Hive job.
	Write-Host "Display the standard output ..." -ForegroundColor Green
	Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $hiveJob.JobId -StandardError

Dadurch werden die beim Ausführen des Auftrags an STDERR auf dem Server geschriebenen Informationen zurückgegeben, die möglicherweise bei der Ermittlung helfen können, warum der Auftrag fehlgeschlagen ist.

## <a id="summary"></a>Zusammenfassung

Wie Sie sehen können, bietet Azure PowerShell eine einfache Möglichkeit, um Hive-Abfragen auf einem HDInsight-Cluster auszuführen, den Auftragsstatus zu überwachen und die Ausgabe abzurufen.

## <a id="nextsteps"></a>Nächste Schritte

Allgemeine Informationen zu Hive in HDInsight:

* [Verwenden von Hive mit Hadoop in HDInsight](../hdinsight-use-hive/)

Informationen zu anderen Möglichkeiten, wie Sie mit Hadoop in HDInsight arbeiten können:

* [Verwenden von Pig mit Hadoop in HDInsight](../hdinsight-use-pig/)

* [Verwenden von MapReduce mit Hadoop in HDInsight](../hdinsight-use-mapreduce/)

<!--HONumber=47-->