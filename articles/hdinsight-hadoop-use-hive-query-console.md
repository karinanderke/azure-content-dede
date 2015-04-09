﻿<properties
   pageTitle="Verwenden von Hadoop Hive in HDInsight | Azure"
   description="Erfahren Sie, wie Sie Hive mit der webbasierten HDInsight-Abfragekonsole verwenden."
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

# Ausführen von Hive-Abfragen mithilfe der Abfragekonsole

[AZURE.INCLUDE [hive-selector](../includes/hdinsight-selector-use-hive.md)]

In diesem Artikel erfahren Sie, wie Sie mit der HDInsight-Abfragekonsole Hive-Abfragen auf einem Hadoop-Cluster in HDInsight in Ihrem Browser ausführen.

> [AZURE.NOTE] Die Abfragekonsole ist nur auf Windows-basierten HDInsight-Clustern verfügbar.

## <a id="prereq"></a>Voraussetzungen

Damit Sie die in dieser Artikel aufgeführten Schritte ausführen können, benötigen Sie Folgendes:

* Einen Windows-basierten Hadoop-Cluster in HDInsight

* Einen zeitgemäßen Webbrowser

## <a id="run"></a> Ausführen von Hive-Abfragen mithilfe der Abfragekonsole

1. Öffnen Sie das <a href="https://manage.windowsazure.com" target="_blank">Azure-Verwaltungsportal</a>, und wählen Sie den HDInsight-Cluster aus. Wählen Sie am Ende der Seite die Option **Abfragekonsole** aus. Geben Sie nach der entsprechenden Aufforderung den Benutzernamen und das Kennwort ein, die Sie beim Erstellen des Clusters verwendet haben.

    > [AZURE.NOTE] Sie können die Abfragekonsole auch durch Eingabe von **https://CLUSTERNAME.azurehdinsight.net** in Ihrem Browser aufrufen.

2. Wählen Sie am Seitenanfang in den Links die Option **Hive-Editor** aus. Dadurch wird ein Formular angezeigt, mit dem Sie HiveQL-Anweisungen eingeben können, die Sie auf dem HDInsight-Cluster ausführen möchten. 
	
	![the hive editor](./media/hdinsight-hadoop-use-hive-query-console/queryconsole.png)
	
	Ersetzen Sie den Text `Select * from hivesampletable` durch die folgenden HiveQL-Anweisungen.

        DROP TABLE log4jLogs;
        CREATE EXTERNAL TABLE log4jLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
        ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
        STORED AS TEXTFILE LOCATION 'wasb:///example/data/';
        SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' GROUP BY t4;

    Diese Anweisungen führen die folgenden Aktionen aus.

    * **DROP TABLE**: löscht Tabelle und Datendatei, falls die Tabelle bereits existiert
    * **CREATE EXTERNAL TABLE** erstellt eine neue  'externe' Tabelle in Hive. Externe Tabellen speichern nur die Tabellendefinition in Hive. Die Daten verbleiben am ursprünglichen Speicherort.

    > [AZURE.NOTE] Wenn erwartet wird, dass die zugrunde liegenden Daten über eine externe Quelle, z. B. einen automatisierten Prozess zum Hochladen von Daten, oder über einen anderen MapReduce-Vorgang aktualisiert, aber von Hive immer die neuesten Daten verwendet werden, sollten externe Tabellen verwendet werden.
    >
    > Durch das Ablegen einer externen Tabelle werden **nicht** die Daten, sondern nur die Tabellendefinitionen gelöscht.

    * **ROW FORMAT**: teilt Hive mit, wie die Daten formatiert werden In diesem Fall werden die Felder in den einzelnen Protokollen durch Leerzeichen getrennt.
    * **STORED AS TEXTFILE LOCATION** teilt Hive den Speicherort der Daten (das Verzeichnis "example/data") und die Informationen mit, dass die Speicherung als Text erfolgt.
    * Mit **SELECT** wählen Sie die Anzahl aller Zeilen aus, bei denen die Spalte **t4** den Wert **[ERROR]** enthält. Dadurch sollte der Wert **3** zurückgegeben werden, da dieser Wert in drei Zeilen enthalten ist.

2. Wählen Sie **Übermitteln** aus. Die **Auftragssitzung** am unteren Seitenrand sollte Details für den Auftrag anzeigen.

3. Sobald das Feld **Status** zu **Abgeschlossen** wechselt, wählen Sie **Details anzeigen** für den Auftrag aus. Auf der Detailseite enthält die **Auftragsausgabe** den Eintrag `[ERROR]	3`. Sie können die Schaltfläche **Herunterladen** unter diesem Feld verwenden, um eine Datei herunterzuladen, die die Ausgabe des Auftrags enthält.


## <a id="summary"></a>Zusammenfassung

Wie Sie sehen können, bietet die Abfragekonsole eine einfache Möglichkeit, um Hive-Abfragen auf einem HDInsight-Cluster auszuführen, den Auftragsstatus zu überwachen und die Ausgabe abzurufen. 

Um weitere Informationen zu Hive mithilfe der Abfragekonsole zu erhalten, wählen Sie am oberen Rand der Abfragekonsole **Erste Schritte** aus, und verwenden Sie dann die Beispiele. In jedem Beispiel werden Sie durch die Datenanalyse mit Hive geführt, einschließlich der Erläuterungen zu HiveQL-Anweisungen, die in diesem Beispiel verwendet werden.

## <a id="nextsteps"></a>Nächste Schritte

Allgemeine Informationen zu Hive in HDInsight:

* [Verwenden von Hive mit Hadoop in HDInsight](../hdinsight-use-hive/)

Informationen zu anderen Möglichkeiten, wie Sie mit Hadoop in HDInsight arbeiten können:

* [Verwenden von Pig mit Hadoop in HDInsight](../hdinsight-use-pig/)

* [Verwenden von MapReduce mit Hadoop in HDInsight](../hdinsight-use-mapreduce/)

[1]: ../hdinsight-hadoop-visual-studio-tools-get-started/

[hdinsight-sdk-documentation]: http://msdnstage.redmond.corp.microsoft.com/library/dn479185.aspx

[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/

[apache-tez]: http://tez.apache.org
[apache-hive]: http://hive.apache.org/
[apache-log4j]: http://en.wikipedia.org/wiki/Log4j
[hive-on-tez-wiki]: https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez
[import-to-excel]: http://azure.microsoft.com/documentation/articles/hdinsight-connect-excel-power-query/


[hdinsight-use-oozie]: ../hdinsight-use-oozie/
[hdinsight-analyze-flight-data]: ../hdinsight-analyze-flight-delay-data/



[hdinsight-storage]: ../hdinsight-use-blob-storage

[hdinsight-provision]: ../hdinsight-provision-clusters/
[hdinsight-submit-jobs]: ../hdinsight-submit-hadoop-jobs-programmatically/
[hdinsight-upload-data]: ../hdinsight-upload-data/
[hdinsight-get-started]: ../hdinsight-get-started/

[Powershell-install-configure]: ../install-configure-powershell/
[powershell-here-strings]: http://technet.microsoft.com/library/ee692792.aspx

[image-hdi-hive-powershell]: ./media/hdinsight-use-hive/HDI.HIVE.PowerShell.png
[img-hdi-hive-powershell-output]: ./media/hdinsight-use-hive/HDI.Hive.PowerShell.Output.png
[image-hdi-hive-architecture]: ./media/hdinsight-use-hive/HDI.Hive.Architecture.png

<!--HONumber=47-->