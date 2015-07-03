<properties
	pageTitle="Verwenden von Python mit Hive und Pig in Azure HDInsight"
	description="Erfahren Sie, wie Sie Benutzerdefinierte Python-Funktionen mit Hive und Pig in Azure HDInsight verwenden können."
	services="hdinsight"
	documentationCenter=""
	authors="Blackmist"
	manager="paulettm"
	editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.workload="big-data"
	ms.tgt_pltfrm="na"
	ms.devlang="python"
	ms.topic="article"
	ms.date="04/23/2015" 
	ms.author="larryfr"/>

#Verwenden von Python mit Hive und Pig in HDInsight

Hive und Pig sind großartig für die Arbeit mit Daten in HDInsight. Manchmal benötigt man aber eine allgemeinere Sprache. Sowohl Hive als auch Pig ermöglichen Ihnen die Erstellung benutzerdefinierter Funktionen (User Defined Functions, UDF) mithilfe einer Vielzahl von Programmiersprachen. In diesem Artikel erfahren Sie, wie Sie eine Python-UDF von Hive und Pig aus verwenden.

> [AZURE.NOTE]Die Schritte in diesem Artikel beziehen sich auf die HDInsight-Clusterversionen 2.1, 3.0, 3.1 und 3.2.


##<a name="python"></a>Python auf HDInsight

Python 2.7 ist auf HDInsight-Clustern ab Version 3.0 standardmäßig installiert. Hive kann mit dieser Version von Python für die Streamverarbeitung verwendet werden (Daten werden zwischen Hive und Python mithilfe von STDOUT/STDIN weitergegeben).

HDInsight enthält außerdem Jython, eine in Java geschriebene Python-Implementierung. Pig kann ohne Streaming mit Jython kommunizieren, weshalb es bei der Arbeit mit Pig vorzuziehen ist.

###<a name="hivepython"></a>Hive und Python

Python kann als UDF von Hive aus mittels der Hive QL **TRANSFORM**-Anweisung verwendet werden. Das folgende HiveQL ruft beispielsweise ein Python-Skript auf, das in der Datei **streaming.py** gespeichert ist.

**HDInsight unter Linux**

	add file wasb:///streaming.py;

	SELECT TRANSFORM (clientid, devicemake, devicemodel)
	  USING 'streaming.py' AS
	  (clientid string, phoneLable string, phoneHash string)
	FROM hivesampletable
	ORDER BY clientid LIMIT 50;

**HDInsight unter Windows**

	add file wasb:///streaming.py;

	SELECT TRANSFORM (clientid, devicemake, devicemodel)
	  USING 'D:\Python27\python.exe streaming.py' AS
	  (clientid string, phoneLable string, phoneHash string)
	FROM hivesampletable
	ORDER BY clientid LIMIT 50;

> [AZURE.NOTE]Bei Windows-basierten HDInsight-Clustern muss die **USING**-Klausel den vollständigen Pfad zu "python.exe" enthalten. Dieser lautet immer `D:\Python27\python.exe`.

Das Beispiel tut Folgendes:

1. Die Anweisung **add file** am Anfang der Datei fügt die Datei **streaming.py** zum verteilten Cache hinzu, sodass sie von allen Knoten im Cluster aus zugänglich ist.

2. Die Anweisung **SELECT TRANSFORM ... USING** wählt Daten aus **hivesampletable** aus und übergibt "clientid", "devicemake" und "devicemodel" an das Skript **streaming.py**.

3. Die **AS**-Klausel beschreibt die Felder, die von **streaming.py** zurückgegeben werden.

<a name="streamingpy"></a> Dies ist die Datei **streaming.py**, die im HiveQL-Beispiel verwendet wird.

	#!/usr/bin/env python

	import sys
	import string
	import hashlib

	while True:
	  line = sys.stdin.readline()
	  if not line:
	    break

	  line = string.strip(line, "\n ")
	  clientid, devicemake, devicemodel = string.split(line, "\t")
	  phone_label = devicemake + ' ' + devicemodel
	  print "\t".join([clientid, phone_label, hashlib.md5(phone_label).hexdigest()])

Da wir Streaming verwenden, muss das Skript Folgendes tun:

1. Lesen der Daten von STDIN. Das wird durch Verwendung von `sys.stdin.readline()` in diesem Beispiel erreicht.

2. Das Zeilenumbruchzeichen am Ende wird mit `string.strip(line, "\n ")` entfernt, da wir nur die Textdaten und keinen Indikator für das Zeilenende benötigen.

2. Bei der Streamverarbeitung enthält eine einzelne Zeile alle Werte, jeweils getrennt durch ein Tabulatorzeichen. Deshalb kann `string.split(line, "\t")` zum Teilen der Eingabe bei jedem Tab verwendet werden, sodass nur die Felder zurückgeben werden.

3. Wenn die Verarbeitung abgeschlossen ist, muss die Ausgabe als eine Zeile nach STDOUT geschrieben werden mit einem Tabulator zwischen jedem Feld. Dies wird mithilfe von `print "\t".join([clientid, phone_label, hashlib.md5(phone_label).hexdigest()])` erreicht.

4. All das ereignet sich in einer `while`-Schleife, die sich wiederholt, bis keine `line` mehr gelesen wird. Zu diesem Zeitpunkt bricht `break` die Schleife ab, und das Skript wird beendet.

Danach verkettet das Skript einfach die Eingabewerte für `devicemake` und `devicemodel` und berechnet einen Hash der verketteten Werte. Ziemlich einfach, aber es beschreibt die Grundlagen zur Funktionsweise aller Python-Skripts, die von Hive aus aufgerufen werden: Schleife, lese Eingabe, bis es keine mehr gibt, teile jede Eingabezeile an den Tabulatorzeichen, Verarbeitung, schreibe eine einzige Zeile mit tabulatorgetrennter Ausgabe.

Wie Sie dieses Beispiel auf Ihrem HDInsight-Cluster ausführen, erfahren Sie unter [Ausführen der Beispiele](#running).

###<a name="pigpython"></a>Pig und Python

Ein Python-Skript kann als UDF von Pig aus mittels der **GENERATE**-Anweisung verwendet werden. Das folgende HiveQL verwendet beispielsweise ein Python-Skript, das in der Datei **jython.py** gespeichert ist.

	Register 'wasb:///jython.py' using jython as myfuncs;
    LOGS = LOAD 'wasb:///example/data/sample.log' as (LINE:chararray);
    LOG = FILTER LOGS by LINE is not null;
    DETAILS = FOREACH LOG GENERATE myfuncs.create_structure(LINE);
    DUMP DETAILS;

So funktioniert dieses Beispiel:

1. Es registriert die Datei, die das Python-Skript (**jython.py**) enthält, mithilfe von **Jython** und gibt es als **myfuncs** an Pig weiter. Jython ist ein Python-Implementierung in Java und läuft auf demselben virtuellen Java-Computer wie Pig. Damit kann das Python-Skript genau wie ein traditioneller Funktionsaufruf behandelt werden, im Gegensatz zum Streamingansatz von Hive.

2. Die nächste Zeile lädt die Datei mit den Beispieldaten, **sample.log**, in **LOGS**. Da diese Protokolldatei kein gleichbleibendes Schema hat, definiert sie außerdem jeden Datensatz (in diesem Fall **LINE**) als ein **chararray**. Chararray ist im Grunde genommen eine Zeichenfolge.

3. Die dritte Zeile filtert etwaige NULL-Werte heraus und speichert die Ergebnisse des Vorgangs in **LOG**.

4. Dann wiederholt es über die Datensätze in **LOG** hinweg und verwendet **GENERATE** zum Aufruf der Methode **create_structure**, die im Skript **jython.py** enthalten ist und als **myfuncs** geladen wird. **LINE** wird für die Übergabe des aktuellen Datensatzes an die Funktion verwendet.

5. Schließlich werden die ausgegebenen Daten mit dem Befehl **DUMP** in STDOUT geschrieben. Das geschieht nur, damit die Ergebnisse direkt nach dem Ende des Vorgangs angezeigt werden; in einem echten Skript würde man die Daten normalerweise in einer neuen Datei speichern (**STORE**).

<a name="jythonpy"></a> Dies ist die Datei **jython.py**, die im Pig-Beispiel verwendet wird:

	@outputSchema("log: {(date:chararray, time:chararray, classname:chararray, level:chararray, detail:chararray)}")
	def create_structure(input):
	  if (input.startswith('java.lang.Exception')):
	    input = input[21:len(input)] + ' - java.lang.Exception'
	  date, time, classname, level, detail = input.split(' ', 4)
	  return date, time, classname, level, detail

Sie erinnern sich daran, dass wir vorhin die **LINE**-Eingabe einfach als Chararray definiert haben, denn es gab kein gleichbleibendes Schema für die Eingabe. Was **jython.py** macht, ist die Daten in ein gleichbleibendes Schema für die Ausgabe zu transformieren. Das funktioniert folgendermaßen:

1. Die **@outputSchema**-Anweisung definiert das Format der Daten, die an Pig zurückgegeben werden. In diesem Fall ist das ein **Datenbehälter**, also ein Pig-Datentyp. Der Behälter enthält folgende Felder, die alle Chararray (Zeichenfolgen) sind:

	* date – das Datum, an dem der Protokolleintrag erstellt wurde
	* time – die Zeit, zu der der Protokolleintrag erstellt wurde
	* classname – der Klassenname, für den der Eintrag erstellt wurde
	* level – die Protokollierungsebene
	* detail – ausführliche Details des Protokolleintrags

2. Dann definiert **def create_structure(input)** die Funktion, der Pig Positionen weitergibt.

3. Die Beispieldatei **sample.log** entspricht weitgehend dem Schema für Datum, Uhrzeit, Klassenname, Ebene und Detail, das wir zurückgeben möchten. Sie enthält aber auch ein paar Zeilen, die mit der Zeichenfolge "*java.lang.Exception*" beginnen, die geändert werden muss, damit sie dem Schema entspricht. Die **if**-Anweisung überprüft deren Vorhandensein, weist dann die Eingabedaten an, die Zeichenfolge "*java.lang.Exception*" ans Ende zu stellen, sodass die Daten dem erwarteten Ausgabeschema entsprechen.

4. Als Nächstes wird der Befehl **split** zum Aufteilen der Daten bei den ersten vier Leerzeichen verwendet. Das führt zu fünf Werten, die **date**, **time**, **classname**, **level** und **detail** zugewiesen werden.

5. Zuletzt werden die Werte an Pig zurückgegeben.

Wenn die Daten an Pig zurückgegeben werden, haben sie ein gleichbleibendes Schema, wie in der **@outputSchema**-Anweisung definiert.

##<a name="running"></a>Ausführen der Beispiele

Wenn Sie einen Linux-basierten HDInsight-Cluster verwenden, gehen Sie anhand der Schritte zu **SSH** vor. Wenn Sie einen Windows-basierten HDInsight-Cluster und einen Windows-Client verwenden, wenden Sie die Schritte zu **PowerShell** an.

###SSH

Weitere Informationen finden Sie unter <a href="../hdinsight-hadoop-linux-use-ssh-unix/" target="_blank">Verwenden von SSH mit Linux-basiertem Hadoop in HDInsight unter Linux, Unix oder OS X</a> oder <a href="../hdinsight-hadoop-linux-use-ssh-windows/" target="_blank">Verwenden von SSH mit Linux-basiertem Hadoop in HDInsight unter Windows</a>.

1. Erstellen Sie mithilfe der Python-Beispiele [streaming.py](#streamingpy) und [jython.py](#jythonpy) lokale Kopien der Dateien auf Ihrem Entwicklungscomputer.

2. Verwenden Sie `scp` zum Kopieren der Dateien in Ihren HDInsight-Cluster. Eine Verbindung zu einem Cluster namens **mycluster** wird beispielsweise wie folgt hergestellt:

		scp streaming.py jython.py myuser@mycluster-ssh.azurehdinsight.net:

3. Verwenden Sie zum Verbinden mit dem Cluster SSH. Eine Verbindung zu einem Cluster namens **mycluster** als Benutzer **myuser** wird beispielsweise wie folgt hergestellt:

		ssh myuser@mycluster-ssh.azurehdinsight.net

4. Fügen Sie in der SSH-Sitzung die zuvor in den WASB-Speicher für den Cluster hochgeladenen Python-Dateien hinzu.

		hadoop fs -copyFromLocal streaming.py /streaming.py
		hadoop fs -copyFromLocal jython.py /jython.py

Nach dem Hochladen der Dateien gehen Sie folgendermaßen vor, um die Hive- und Pig-Aufträge auszuführen.

####Hive

1. Verwenden Sie den `hive`-Befehl zum Starten der Hive-Shell. Nach dem Laden der Shell sollte eine `hive>`-Eingabeaufforderung angezeigt werden.

2. Geben Sie an der `hive>`-Eingabeaufforderung Folgendes ein:

		add file wasb:///streaming.py;
		SELECT TRANSFORM (clientid, devicemake, devicemodel)
		  USING 'streaming.py' AS
		  (clientid string, phoneLabel string, phoneHash string)
		FROM hivesampletable
		ORDER BY clientid LIMIT 50;

3. Nach Eingabe der letzten Zeile sollte der Auftrag gestartet werden. Die Ausgabe sollte ähnliche Ergebnisse wie diese liefern:

		100041	RIM 9650	d476f3687700442549a83fac4560c51c
		100041	RIM 9650	d476f3687700442549a83fac4560c51c
		100042	Apple iPhone 4.2.x	375ad9a0ddc4351536804f1d5d0ea9b9
		100042	Apple iPhone 4.2.x	375ad9a0ddc4351536804f1d5d0ea9b9
		100042	Apple iPhone 4.2.x	375ad9a0ddc4351536804f1d5d0ea9b9

####Pig

1. Verwenden Sie den `pig`-Befehl zum Starten der Shell. Nach dem Laden der Shell sollte eine `grunt>`-Eingabeaufforderung angezeigt werden.

2. Geben Sie die folgenden Anweisungen an der `grunt>`-Eingabeaufforderung ein:

		Register wasb:///jython.py using jython as myfuncs;
	    LOGS = LOAD 'wasb:///example/data/sample.log' as (LINE:chararray);
	    LOG = FILTER LOGS by LINE is not null;
	    DETAILS = foreach LOG generate myfuncs.create_structure(LINE);
	    DUMP DETAILS;

3. Nach Eingabe der folgenden Zeile sollte der Auftrag gestartet werden. Die Ausgabe sollte ähnliche Ergebnisse wie diese liefern:

		((2012-02-03,20:11:56,SampleClass5,[TRACE],verbose detail for id 990982084))
		((2012-02-03,20:11:56,SampleClass7,[TRACE],verbose detail for id 1560323914))
		((2012-02-03,20:11:56,SampleClass8,[DEBUG],detail for id 2083681507))
		((2012-02-03,20:11:56,SampleClass3,[TRACE],verbose detail for id 1718828806))
		((2012-02-03,20:11:56,SampleClass3,[INFO],everything normal for id 530537821))

###PowerShell

Bei diesen Schritten wird Azure PowerShell verwendet. Wen es nicht bereits auf Ihrem Entwicklungsrechner installiert und konfiguriert ist, sehen Sie sich bitte [Installieren und Konfigurieren von Azure PowerShell](../install-configure-powershell.md) an und folgen Sie den Schritten dort.

1. Erstellen Sie mithilfe der Python-Beispiele [streaming.py](#streamingpy) und [jython.py](#jythonpy) lokale Kopien der Dateien auf Ihrem Entwicklungscomputer.

2. Verwenden Sie das folgende PowerShell-Skript zum Hochladen der Dateien **streaming.py** und **jython.py** auf den Server. Ersetzen Sie den Namen Ihres HDInsight-Clusters und den Pfad zu den Dateien **streaming.py** und **jython.py** in den ersten drei Skriptzeilen.

		$clusterName = YourHDIClusterName
		$pathToStreamingFile = "C:\path\to\streaming.py"
		$pathToJythonFile = "C:\path\to\jython.py"

		$hdiStore = get-azurehdinsightcluster -name $clusterName
		$storageAccountName = $hdiStore.DefaultStorageAccount.StorageAccountName.Split(".",2)[0]
		$storageAccountKey = $hdiStore.defaultstorageaccount.storageaccountkey
		$defaultContainer = $hdiStore.DefaultStorageAccount.StorageContainerName

		$destContext = new-azurestoragecontext -storageaccountname $storageAccountName -storageaccountkey $storageAccountKey
		set-azurestorageblobcontent -file $pathToStreamingFile -Container $defaultContainer -Blob "streaming.py" -context $destContext
		set-azurestorageblobcontent -file $pathToJythonFile -Container $defaultContainer -Blob "jython.py" -context $destContext

	Dieses Skript ruft Informationen für Ihren HDInsight-Cluster ab, extrahiert dann das Konto und den Schlüssel für das standardmäßigen Speicherkonto und lädt die Dateien in den Stamm des Containers.

	> [AZURE.NOTE]Andere Methoden zum Hochladen der Skripts finden sich im Dokument [Hochladen von Daten für Hadoop-Aufträge in HDInsight](hdinsight-upload-data.md).

Verwenden Sie nach dem Hochladen der Dateien die folgenden PowerShell-Skripts zum Starten der Jobs. Wenn der Job abgeschlossen ist, sollte die Ausgabe zur PowerShell-Konsole geschrieben werden.

####Hive

    # Replace 'YourHDIClusterName' with the name of your cluster
	$clusterName = YourHDIClusterName

	$HiveQuery = "add file wasb:///streaming.py;" +
	             "SELECT TRANSFORM (clientid, devicemake, devicemodel) " +
	               "USING 'D:\Python27\python.exe streaming.py' AS " +
	               "(clientid string, phoneLabel string, phoneHash string) " +
	             "FROM hivesampletable " +
	             "ORDER BY clientid LIMIT 50;"

	$jobDefinition = New-AzureHDInsightHiveJobDefinition -Query $HiveQuery -StatusFolder '/hivepython'

	$job = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $jobDefinition
	Write-Host "Wait for the Hive job to complete ..." -ForegroundColor Green
	Wait-AzureHDInsightJob -Job $job
    # Uncomment the following to see stderr output
    # Get-AzureHDInsightJobOutput -StandardError -JobId $job.JobId -Cluster $clusterName
	Write-Host "Display the standard output ..." -ForegroundColor Green
	Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $job.JobId -StandardOutput

Die Ausgabe für den **Hive**-Job sollte ungefähr wie folgt aussehen:

	100041	RIM 9650	d476f3687700442549a83fac4560c51c
	100041	RIM 9650	d476f3687700442549a83fac4560c51c
	100042	Apple iPhone 4.2.x	375ad9a0ddc4351536804f1d5d0ea9b9
	100042	Apple iPhone 4.2.x	375ad9a0ddc4351536804f1d5d0ea9b9
	100042	Apple iPhone 4.2.x	375ad9a0ddc4351536804f1d5d0ea9b9

####Pig

	# Replace 'YourHDIClusterName' with the name of your cluster
	$clusterName = YourHDIClusterName

	$PigQuery = "Register wasb:///jython.py using jython as myfuncs;" +
	            "LOGS = LOAD 'wasb:///example/data/sample.log' as (LINE:chararray);" +
	            "LOG = FILTER LOGS by LINE is not null;" +
	            "DETAILS = foreach LOG generate myfuncs.create_structure(LINE);" +
	            "DUMP DETAILS;"

	$jobDefinition = New-AzureHDInsightPigJobDefinition -Query $PigQuery -StatusFolder '/pigpython'

	$job = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $jobDefinition
	Write-Host "Wait for the Pig job to complete ..." -ForegroundColor Green
	Wait-AzureHDInsightJob -Job $job
    # Uncomment the following to see stderr output
    # Get-AzureHDInsightJobOutput -StandardError -JobId $job.JobId -Cluster $clusterName
	Write-Host "Display the standard output ..." -ForegroundColor Green
	Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $job.JobId -StandardOutput

Die Ausgabe für den **Pig**-Job sollte ungefähr wie folgt aussehen:

	((2012-02-03,20:11:56,SampleClass5,[TRACE],verbose detail for id 990982084))
	((2012-02-03,20:11:56,SampleClass7,[TRACE],verbose detail for id 1560323914))
	((2012-02-03,20:11:56,SampleClass8,[DEBUG],detail for id 2083681507))
	((2012-02-03,20:11:56,SampleClass3,[TRACE],verbose detail for id 1718828806))
	((2012-02-03,20:11:56,SampleClass3,[INFO],everything normal for id 530537821))

##<a name="troubleshooting"></a>Problembehandlung

Beide für das Ausführen der Beispiele verwendete PowerShell-Beispielskripte enthalten eine Kommentarzeile, die Fehler bei der Ausgabe des Jobs anzeigt. Wenn Sie nicht die erwartete Ausgabe für den Job sehen, kommentieren Sie die folgende Zeile aus und sehen Sie nach, ob die Fehlerinformation auf ein Problem hinweist.

	# Get-AzureHDInsightJobOutput -StandardError -JobId $job.JobId -Cluster $clusterName

Die Fehlermeldung (STDERR) und das Ergebnis des Jobs (STDERR) werden ebenfalls im standardmäßigen BLOB-Container an folgenden Orten für Ihren Cluster protokolliert.

<table>
<tr>
<td>Für diesen Job...</td><td>Sehen Sie sich diese Dateien im BLOB-Container an</td>
</tr>
<td>Hive</td><td>/HivePython/stderr</br>/HivePython/stdout</td>
</tr>
<td>Pig</td><td>/PigPython/stderr</br>/PigPython/stdout</td>
</tr>
</table>

##<a name="next"></a>Nächste Schritte

Wenn Sie Python-Module laden müssen, die standardmäßig nicht bereitgestellt werden, sehen Sie sich [Bereitstellen eines Moduls für Azure HDInsight](http://blogs.msdn.com/b/benjguin/archive/2014/03/03/how-to-deploy-a-python-module-to-windows-azure-hdinsight.aspx), wo Sie ein Beispiel finden, wie das geht.

Informationen zu anderen Möglichkeiten der Verwendung von Pig und Hive sowie Informationen zur Verwendung von MapReduce finden Sie in diesen Themen:

* [Verwenden von Hive mit HDInsight](hdinsight-use-hive.md)

* [Verwenden von Pig mit HDInsight](hdinsight-use-pig.md)

* [Verwenden von MapReduce mit HDInsight](hdinsight-use-mapreduce.md)

<!--HONumber=54--> 