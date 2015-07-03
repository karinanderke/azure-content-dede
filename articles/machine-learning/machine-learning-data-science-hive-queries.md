<properties 
	pageTitle="Übermitteln von Hive-Abfragen an HDInsight Hadoop-Cluster in Cloud Data Science | Azure" 
	description="Verarbeiten von Daten aus Hive-Tabellen" 
	metaKeywords="" 
	services="machine-learning" 
	solutions="" 
	documentationCenter="" 
	authors="hangzh-msft" 
	manager="jacob.spoelstra" 
	editor="cgronlun"/>

<tags 
	ms.service="machine-learning" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="03/23/2015" 
	ms.author="hangzh;bradsev"/> 

#<a name="heading"></a> Übermitteln von Hive-Abfragen an HDInsight Hadoop-Cluster in Cloud Data Science

In diesem Dokument werden die verschiedenen Möglichkeiten zum Übermitteln von Hive-Abfragen an Hadoop-Cluster beschrieben, die durch einen HDInsight-Dienst in Azure verwaltet werden. Hive-Abfragen können folgendermaßen übermittelt werden: 

* an der Hadoop-Befehlszeile auf dem Hauptknoten des Clusters
* in einem IPython Notebook 
* im Hive-Editor
* mit Azure PowerShell-Skripts 

Es werden generische Hive-Abfragen bereitgestellt, die zum Durchsuchen der Daten oder zum Generieren der Funktionen verwendet werden, die eingebettete Hive-UDFs (User Defined Function, benutzerdefinierte Funktion) nutzen. 

Beispiele für Abfragen speziell für Szenarios mit den [NYC Taxi Trip-Daten](http://chriswhong.com/open-data/foil_nyc_taxi/) stehen auch im [GitHub-Repository](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/DataScienceProcess/DataScienceScripts) bereit. Für diese Abfragen ist bereits ein Datenschema angegeben, sodass sie bereit für die Übermittlung zur Ausführung sind. 

Im letzten Abschnitt werden Parameter beschrieben, mit denen Sie die Leistung der Hive-Abfragen optimieren können.

## Voraussetzungen
In diesem Artikel wird davon ausgegangen, dass Sie Folgendes abgeschlossen haben:
 
* Sie haben ein Azure-Speicherkonto erstellt. Anweisungen finden Sie unter [Erstellen eines Azure-Speicherkontos](../hdinsight-get-started.md#storage) 
* Sie haben einen Hadoop-Cluster mit dem HDInsight-Dienst bereitgestellt.  Anweisungen finden Sie unter [Bereitstellen eines HDInsight-Clusters](../hdinsight-get-started.md#provision).
* Sie haben die Daten in Hive-Tabellen auf Azure HDInsight Hadoop-Clustern hochgeladen. Wenn dies nicht der Fall ist, führen Sie die unter [Erstellen und Laden von Daten in Hive-Tabellen](machine-learning-data-science-hive-tables.md) beschriebenen Schritte zum Hochladen von Daten in Hive-Tabellen aus.
* Sie haben den Remotezugriff auf den Cluster aktiviert. Anweisungen finden Sie unter [Zugreifen auf den Hauptknoten von Hadoop-Clustern](machine-learning-data-science-customize-hadoop-cluster.md#remoteaccess). 


- [Übermitteln von Hive-Abfragen](#submit)
- [Durchsuchen von Daten und Verarbeiten von Funktionen](#explore)
- [Weiterführende Themen: Verbessern der Geschwindigkeit von Abfragen mit Hive-Parametern](#tuning)

## <a name="submit"></a>Übermitteln von Hive-Abfragen

###1. Über die Hadoop-Befehlszeile im Hauptknoten des Hadoop-Clusters

Wenn die Abfrage komplex ist, führen Hive-Abfragen direkt am Hauptknoten des Hadoop-Clusters i. d. R. schneller zu Ergebnissen als das Senden im Hive-Editor oder über Azure PowerShell-Skripts. 

Melden Sie sich am Hauptknoten des Hadoop-Clusters an, öffnen Sie die Hadoop-Befehlszeile auf dem Desktop des Hauptknotens, und geben Sie den Befehl `cd %hive_home%\bin` ein.

Es gibt drei Möglichkeiten zum Übermitteln von Hive-Abfragen an der Hadoop-Befehlszeile. 

####Übermitteln der Hive-Abfragen direkt an der Hadoop-Befehlszeile 

Sie können einen Befehl wie `hive -e "<die Hive-Abfrage>;` ausführen, um die Hive-Abfragen direkt an der Hadoop-Befehlszeile zu übermitteln. Hier ist ein Beispiel. Das rote Kästchen enthält den Befehl, der die Hive-Abfrage übermittelt, und das grüne Kästchen umschließt die Ausgabe der Hive-Abfrage.

![Create workspace][10]

####Übermitteln der Hive-Abfragen in HQL-Dateien

Wenn die Hive-Abfrage sehr komplex ist und aus mehreren Zeilen besteht, ist das direkte Bearbeiten der Abfragen an der Befehlszeile oder an der Hive-Konsole nicht sehr praktikabel. Alternativ können die Hive-Abfragen mit einem Text-Editor auf dem Hauptknoten des Hadoop-Clusters in einer HQL-Datei in einem lokalen Verzeichnis auf dem Hauptknoten gespeichert werden. Anschließend kann die Hive-Abfrage in der HQL-Datei mithilfe des `-f`-Arguments übermittelt werden:
	
	`hive -f "<path to the .hql file>"`

![Create workspace][15]


####Unterdrücken der Fortschrittsanzeige bei Hive-Abfragen

Standardmäßig wird bei Hive-Abfragen, die über die Hadoop-Befehlszeile übermittelt werden, der Fortschritt des Map/Reduce-Auftrags auf dem Bildschirm angezeigt. Um diese Fortschrittsanzeige zum Map/Reduce-Auftrag zu unterdrücken, können Sie das Argument `-S` ("S" als Großbuchstabe) wie folgt an der Befehlszeile einfügen:

	hive -S -f "<path to the .hql file>"
	hive -S -e "<Hive queries>"

####Übermitteln von Hive-Abfragen an der Hive-Befehlskonsole

Sie können auch zunächst die Hive-Befehlskonsole starten, indem Sie den Befehl `hive` an der Hadoop-Befehlszeile eingeben, und dann die Hive-Abfragen über die Hive-Befehlskonsole übermitteln. Beispiel: In diesem Beispiel kennzeichnen die beiden roten Kästchen die Befehle, die zum Starten der Hive-Befehlskonsole verwendet werden, und die Hive-Abfrage, die über die Hive-Befehlskonsole übermittelt wird. Das grüne Kästchen markiert die Ausgabe der Hive-Abfrage. 

![Create workspace][11]

In den vorherigen Beispielen wurden die Ergebnisse der Hive-Abfrage direkt auf dem Bildschirm ausgeben. Sie können die Ausgabe aber auch in eine lokale Datei auf dem Hauptknoten oder in ein Azure-Blob schreiben. Sie können dann andere Tools verwenden, um die Ausgabe der Hive-Abfragen weiter zu analysieren.

####Ausgeben der Hive-Abfrageergebnisse in eine lokale Datei 

Für die Ausgabe der Hive-Abfrageergebnisse in ein lokales Verzeichnis auf dem Hauptknoten müssen Sie die Hive-Abfrage an der Hadoop-Befehlszeile wie folgt übermitteln:

	`hive -e "<hive query>" > <local path in the head node>`

Im folgenden Beispiel wird die Ausgabe der Hive-Abfrage in die Datei `hivequeryoutput.txt` im Verzeichnis `C:\apps\temp` geschrieben.

![Create workspace][12]

####Ausgeben der Hive-Abfrageergebnisse in ein Azure-Blob

Sie können die Hive-Abfrageergebnisse auch in ein Azure-Blob im Standardcontainer des Hadoop-Clusters ausgeben. Die Hive-Abfrage muss wie folgt lauten:

	`insert overwrite directory wasb:///<directory within the default container> <select clause from ...>`

Im folgenden Beispiel wird die Ausgabe des Hive-Abfrage in das Blob-Verzeichnis `queryoutputdir` im Standardcontainer des Hadoop-Clusters geschrieben. Hier müssen Sie nur den Namen des Verzeichnisses ohne den Namen des Blobs angeben. Es wird ein Fehler ausgelöst, wenn Sie sowohl den Verzeichnis- als auch den Blob-Namen angeben, z. B. mit `wasb:///queryoutputdir/queryoutput.txt`. 

![Create workspace][13]

Wenn Sie den Standardcontainer des Hadoop-Clusters mit Tools wie Azure-Speicher-Explorer öffnen, wird die Ausgabe der Hive-Abfrage wie folgt angezeigt. Sie können Filter (markiert durch den roten Kasten) anwenden, um nur das Blob mit den angegebenen Buchstaben im Namen abzurufen.

![Create workspace][14]

###2. Über den Hive-Editor oder Azure PowerShell-Befehle

Sie können auch die Abfrage-Konsole (Hive-Editor) verwenden, indem Sie in einem Webbrowser die URL `https://<Hadoop cluster name>.azurehdinsight.net/Home/HiveEditor` eingeben. (Sie werden aufgefordert, die Anmeldeinformationen für den Hadoop-Cluster anzugeben.) Sie können aber auch wie unter [Übermitteln von Hive-Aufträgen mit PowerShell](../hdinsight/hdinsight-submit-hadoop-jobs-programmatically.md#hive-powershell) beschrieben vorgehen. 

## <a name="explore"></a>Durchsuchen von Daten, Verarbeiten von Funktionen und Optimieren von Hive-Parametern

Für die folgenden Schritte in diesem Abschnitt zum Durchsuchen von Daten verwenden wir Hive in Azure HDInsight Hadoop-Clustern. In einem weiterführenden Thema werden einige Hive-Parameter beschrieben, mit denen die Leistung der Hive-Abfragen optimiert werden kann:

1. [Durchsuchen von Daten](#hive-dataexploration)
2. [Generieren von Funktionen](#hive-featureengineering)
3. [Weiterführende Themen: Verbessern der Geschwindigkeit von Abfragen mit Hive-Parametern](#tune-parameters)

> [AZURE.NOTE] Bei den Beispiel-Hive-Abfragen wird davon ausgegangen, dass die Daten in Hive-Tabellen in Azure HDInsight Hadoop-Clustern hochgeladen wurden. Wenn dies nicht der Fall ist, führen Sie die unter [Erstellen und Laden von Daten in Hive-Tabellen](machine-learning-data-science-hive-tables.md)  beschriebenen Schritte zum Hochladen von Daten in Hive-Tabellen aus.

###<a name="hive-dataexploration"></a>Durchsuchen von Daten
Hier finden Sie einige Hive-Beispielskripts, die zum Durchsuchen von Daten in den Hive-Tabellen verwendet werden können.

1. Abrufen der Anzahl der Vorkommen pro Partition:
	`SELECT <partitionfieldname>, count(*) from <databasename>.<tablename> group by <partitionfieldname>;`

2. Abrufen der Anzahl der Vorkommen pro Tag:
	`SELECT to_date(<date_columnname>), count(*) from <databasename>.<tablename> group by to_date(<date_columnname>);` 

3. Abrufen der Ebenen in einer Kategoriespalte:  
	`SELECT  distinct <column_name> from <databasename>.<tablename>`

4. Abrufen der Anzahl von Ebenen in Kombination zweier kategorischer Spalten: 
	`SELECT <column_a>, <column_b>, count(*) from <databasename>.<tablename> group by <column_a>, <column_b>`

5. Abrufen der Verteilung der numerischen Spalten:  
	`SELECT <column_name>, count(*) from <databasename>.<tablename> group by <column_name>`

6. Extrahieren von Datensätzen durch Zusammenführen von zwei Tabellen: 

	    SELECT 
			a.<common_columnname1> as <new_name1>,
			a.<common_columnname2> as <new_name2>,
    		a.<a_column_name1> as <new_name3>,
    		a.<a_column_name2> as <new_name4>,
    		b.<b_column_name1> as <new_name5>,
    		b.<b_column_name2> as <new_name6>
    	FROM
    		(
    		SELECT <common_columnname1>, 
    			<common_columnname2>,
				<a_column_name1>,
				<a_column_name2>,
			FROM <databasename>.<tablename1>
			) a
			join
			(
			SELECT <common_columnname1>, 
    			<common_columnname2>,
				<b_column_name1>,
				<b_column_name2>,
			FROM <databasename>.<tablename2>
			) b
			ON a.<common_columnname1>=b.<common_columnname1> and a.<common_columnname2>=b.<common_columnname2>

###<a name="hive-featureengineering"></a>Generieren von Funktionen

In diesem Abschnitt werden Methoden zum Generieren von Funktionen mithilfe von Hive-Abfragen beschrieben: 

> [AZURE.NOTE]
> Wenn Sie zusätzliche Funktionen generieren, können Sie diese als Spalten in der vorhandenen Tabelle hinzufügen oder eine neue Tabelle mit den zusätzlichen Funktionen und einem Primärschlüssel erstellen, die dann mit der ursprünglichen Tabelle zusammengeführt werden kann.  

1. [Häufigkeitsbasierte Funktionsgenerierung](#hive-frequencyfeature)
2. [Risiken kategorischer Variablen in binären Klassifizierungen](#hive-riskfeature)
3. [Extrahieren von Funktionen aus "datetime"-Feldern](#hive-datefeatures)
4. [Extrahieren von Funktionen aus "Text"-Feldern](#hive-textfeatures)
5. [Berechnen der Entfernung zwischen GPS-Koordinaten](#hive-gpsdistance)

####<a name="hive-frequencyfeature"></a>Häufigkeitsbasierte Funktionsgenerierung

Manchmal ist es sinnvoll, die Häufigkeit der Ebenen einer kategorischen Variable oder die Häufigkeiten von Ebenenkombinationen mehrerer kategorischer Variablen zu berechnen. Mit dem folgenden Skript können Sie die Häufigkeiten berechnen:

		select 
			a.<column_name1>, a.<column_name2>, a.sub_count/sum(a.sub_count) over () as frequency
		from
		(
			select 
				<column_name1>,<column_name2>, count(*) as sub_count 
			from <databasename>.<tablename> group by <column_name1>, <column_name2>
		)a
		order by frequency desc;
	

####<a name="hive-riskfeature"></a>Risiken kategorischer Variablen in binären Klassifizierungen

Bei binären Klassifizierungen müssen manchmal nicht numerische kategorische Variablen in numerische Funktionen umgewandelt werden, da einige Modelle nur numerische Funktionen akzeptieren. Dazu werden die nicht numerischen Ebenen durch numerische Risiken ersetzt. In diesem Abschnitt werden einige generische Hive-Abfragen zur Berechnung der Risikowerte (logarithmische Wahrscheinlichkeiten) einer kategorischen Variable gezeigt. 


	    set smooth_param1=1;
	    set smooth_param2=20;
	    select 
	    	<column_name1>,<column_name2>, 
			ln((sum_target+${hiveconf:smooth_param1})/(record_count-sum_target+${hiveconf:smooth_param2}-${hiveconf:smooth_param1})) as risk
	    from
	    	(
	    	select 
	    		<column_nam1>, <column_name2>, sum(binary_target) as sum_target, sum(1) as record_count
	    	from
	    		(
	    		select 
	    			<column_name1>, <column_name2>, if(target_column>0,1,0) as binary_target
	    		from <databasename>.<tablename> 
	    		)a
	    	group by <column_name1>, <column_name2>
	    	)b 

In diesem Beispiel werden die Variablen `smooth_param1` und `smooth_param2` zum Glätten der aus den Daten berechneten Risikowerte festgelegt. Die Risiken liegen zwischen "-Inf" und "Inf". Ein Risiko > 0 steht für die Wahrscheinlichkeit, dass das Ziel 1 größer als 0,5 ist. 

Nach dem Berechnen der Risikotabelle können Sie einer Tabelle Risikowerte zuweisen, indem Sie sie mit der Risikotabelle zusammenführen. Die Hive-Abfrage für das Zusammenführen wurde im vorherigen Abschnitt beschrieben.

####<a name="hive-datefeature"></a>Extrahieren von Funktionen aus "datetime"-Feldern

Hive bietet eine Reihe von UDFs für die Verarbeitung von "datetime"-Feldern. In Hive lautet das Standardformat für "datetime" "JJJJ-MM-TT 00:00:00" (z. B. "1970-01-01 12:21:32"). In diesem Abschnitt werden Beispiele für das Extrahieren des Tages eines Monats und des Monats aus einem "datetime"-Feld gezeigt. Außerdem gibt es Beispiele zum Konvertieren einer "datetime"-Zeichenfolge in ein anderes Format als das Standardformat. 

    	select day(<datetime field>), month(<datetime field>) 
		from <databasename>.<tablename>;

Bei dieser Hive-Abfrage wird vorausgesetzt, dass das `<datetime field>` im Standardformat für "datetime" vorliegt.

Liegt ein "datetime"-Feld nicht im Standardformat vor, muss zunächst das "datetime"-Feld in einen Unix-Zeitstempel konvertiert werden, der dann in eine "datetime"-Zeichenfolge im Standardformat konvertiert wird. Wenn "datetime" das Standardformat aufweist, können Sie die eingebetteten "datetime"-UDFs anwenden, um Funktionen zu extrahieren.

		select from_unixtime(unix_timestamp(<datetime field>,'<pattern of the datetime field>'))
		from <databasename>.<tablename>;

Wenn bei dieser Abfrage `<datetime field>` ein Muster wie `03/26/2015 12:04:39` aufweist, sollte `'<pattern of the datetime field>'` das Format `'MM/TT/JJJJ HH:mm:ss'` haben. Zum Testen können Sie folgenden Code ausführen:

		select from_unixtime(unix_timestamp('05/15/2015 09:32:10','MM/dd/yyyy HH:mm:ss'))
		from hivesampletable limit 1;

In dieser Abfrage enthält `hivesampletable` standardmäßig alle Azure HDInsight Hadoop-Cluster, wenn die Cluster bereitgestellt wurden. 


####<a name="hive-textfeature"></a>Extrahieren von Funktionen aus "Text"-Feldern

Angenommen, die Hive-Tabelle enthält ein Textfeld, das eine Zeichenfolge von durch Leerzeichen voneinander getrennten Wörtern ist. Sie können dann mit der folgenden Abfrage die Länge der Zeichenfolge und die Anzahl der Wörter in der Zeichenfolge extrahieren.

    	select length(<text field>) as str_len, size(split(<text field>,' ')) as word_num 
		from <databasename>.<tablename>;

####<a name="hive-gpsdistance"></a>Berechnen der Entfernung zwischen GPS-Koordinaten

Die in diesem Abschnitt angegebene Abfrage kann direkt auf die "NYC Taxi Trip"-Daten angewendet werden. Diese Abfrage soll veranschaulichen, wie Sie die eingebetteten mathematischen Funktionen in Hive zum Generieren von Funktionen verwenden. 

Die in dieser Abfrage verwendeten Felder sind GPS-Koordinaten von Start- und Zielorten mit den Bezeichnungen "pickup_longitude", "pickup_latitude", "dropoff_longitude" und "dropoff_latitude". Die Abfragen zur Berechnung der direkten Entfernung zwischen den Start- und Zielkoordinaten sind:

		set R=3959;
		set pi=radians(180);
		select pickup_longitude, pickup_latitude, dropoff_longitude, dropoff_latitude, 
			${hiveconf:R}*2*2*atan((1-sqrt(1-pow(sin((dropoff_latitude-pickup_latitude)
			*${hiveconf:pi}/180/2),2)-cos(pickup_latitude*${hiveconf:pi}/180)
			*cos(dropoff_latitude*${hiveconf:pi}/180)*pow(sin((dropoff_longitude-pickup_longitude)*${hiveconf:pi}/180/2),2)))
			/sqrt(pow(sin((dropoff_latitude-pickup_latitude)*${hiveconf:pi}/180/2),2)
			+cos(pickup_latitude*${hiveconf:pi}/180)*cos(dropoff_latitude*${hiveconf:pi}/180)*
			pow(sin((dropoff_longitude-pickup_longitude)*${hiveconf:pi}/180/2),2))) as direct_distance 
		from nyctaxi.trip 
		where pickup_longitude between -90 and 0
		and pickup_latitude between 30 and 90
		and dropoff_longitude between -90 and 0
		and dropoff_latitude between 30 and 90
		limit 10; 

Die mathematischen Gleichungen zur Berechnung der Entfernung zwischen zwei GPS-Koordinaten finden Sie in [diesem Artikel](http://www.movable-type.co.uk/scripts/latlong.html) (in englischer Sprache) von Peter Lapisu. In seinem JavaScript-Code ist die toRad()-Funktion lediglich `lat_or_lon*pi/180` und rechnet Gradangaben in Radianten um. Dabei stehen `lat_or_lon` für Längen- bzw. Breitengrad. Da Hive keine Funktion `atan2` bietet, aber die Funktion `atan`, wird in der Abfrage oben `atan2` von `atan` implementiert. Dies basiert auf der Definition in [Wikipedia](http://en.wikipedia.org/wiki/Atan2). 

![Create workspace][1]

Eine vollständige Liste der eingebettete Hive-UDFs finden Sie [hier](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-MathematicalFunctions) (in englischer Sprache). 

## <a name="tuning"></a> Weiterführende Themen: Verbessern der Geschwindigkeit von Abfragen mit Hive-Parametern

Die Standardeinstellungen für die Parameter von Hive-Clustern eignen sich möglicherweise nicht für die Hive-Abfragen und die von den Abfragen verarbeiteten Daten. In diesem Abschnitt werden einige Parameter beschrieben, mit denen Sie die Leistung der Hive-Abfragen optimieren können. Sie müssen die Abfragen zur Parameteroptimierung vor den Abfragen der Verarbeitungsdaten einfügen. 

1. Java-Heapspeicher: Für Abfragen, bei denen große DataSets zusammengeführt oder lange Datensätze verarbeitet werden, besteht ein typischer Fehler darin, dass **nicht genügend Heapspeicher** vorhanden ist. Dies kann durch Festlegen der Parameter `mapreduce.map.java.opts` und `mapreduce.task.io.sort.mb` auf die gewünschten Werte verbessert werden. Beispiel:

		set mapreduce.map.java.opts=-Xmx4096m;
		set mapreduce.task.io.sort.mb=-Xmx1024m;
	

	Dieser Parameter ordnet dem Java-Heapspeicher 4 GB Arbeitsspeicher zu. Außerdem macht er die Sortierung effizienter, da er ihr mehr Arbeitsspeicher zuweist. Sie sollten etwas mit diesem Parameter experimentieren, wenn bei einem Auftrag Fehler im Zusammenhang mit dem Heapspeicher auftreten.

2. DFS-Blockgröße: Mit diesem Parameter wird die kleinste Dateneinheit im Dateisystem festgelegt. Wenn beispielsweise die DFS-Blockgröße 128 MB beträgt, werden alle Daten bis zu einer Größe von 128 MB in einem Block gespeichert. Daten, die größer als 128 MB sind, werden zusätzliche Blöcke zugewiesen. Die Auswahl einer sehr geringen Blockgröße führt zu starkem zusätzlichem Verbrauch in Hadoop, da der Namensknoten viel mehr Anforderungen verarbeiten muss, um den entsprechenden Block für die Datei zu finden. Eine empfohlene Einstellung bei der Arbeit mit Daten im Gigabyteberiech (oder größer) ist:

		set dfs.block.size=128m;

3. Optimieren der Join-Vorgänge in Hive: Join-Vorgänge im Map/Reduce-Framework erfolgen i. d. R. in der Reduce-Phase. In einigen Fällen können enorme Gewinne erzielt werden, wenn die Join-Vorgänge in die Map-Phase verlegt werden (sogenannte "mapjoins"). Damit Hive dies möglichst häufig durchführt, legen Sie Folgendes fest:

		set hive.auto.convert.join=true;

4. Festlegen der Anzahl von Mappern in Hive: Hadoop erlaubt das Festlegen der Anzahl von Reducern - die Anzahl der Mapper kann aber i. d. R. nicht vom Benutzer festgelegt werden. Mit einem Trick erhalten Sie ein gewisses Maß an Kontrolle über diese Anzahl: Wählen Sie die Hadoop-Variablen "mapred.min.split.size" und "mapred.max.split.size" aus. Die Größe der einzelnen Zuordnungsaufgaben wird durch Folgendes bestimmt:

		num_maps = max(mapred.min.split.size, min(mapred.max.split.size, dfs.block.size))

	Der Standardwert von "mapred.min.split.size" ist 0, der von "mapred.max.split.size" ist Long.MAX und der von "dfs.block.size" ist 64 MB. Bei entsprechender Datengröße können Sie durch Festlegen dieser Parameter die Anzahl der verwendeten Mapper optimieren. 

5. Einige andere weiterführende Optionen zur Optimierung der Leistung finden Sie unten. Diese ermöglichen das Festlegen des zugeordneten Speichers für "map"- und "reduce"-Aufgaben, die beim Optimieren der Leistung nützlich sein können. Beachten Sie aber, dass `mapreduce.reduce.memory.mb` nicht größer sein darf als die Größe des physischen Speichers der einzelnen Workerknoten im Hadoop-Cluster.

		set mapreduce.map.memory.mb = 2048;
		set mapreduce.reduce.memory.mb=6144;
		set mapreduce.reduce.java.opts=-Xmx8192m;
		set mapred.reduce.tasks=128;
		set mapred.tasktracker.reduce.tasks.maximum=128;

[1]: ./media/machine-learning-data-science-hive-queries/atan2new.png
[10]: ./media/machine-learning-data-science-hive-queries/run-hive-queries-1.png
[11]: ./media/machine-learning-data-science-hive-queries/run-hive-queries-2.png
[12]: ./media/machine-learning-data-science-hive-queries/output-hive-results-1.png
[13]: ./media/machine-learning-data-science-hive-queries/output-hive-results-2.png
[14]: ./media/machine-learning-data-science-hive-queries/output-hive-results-3.png
[15]: ./media/machine-learning-data-science-hive-queries/run-hive-queries-3.png



<!--HONumber=49--> 