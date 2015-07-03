<properties 
	pageTitle="Überwachen eines Media Services-Kontos - Azure" 
	description="Beschreibt die Konfiguration der Überwachung für Ihr Media Services-Konto in Azure." 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="02/05/2015" 
	ms.author="juliako"/>





#<a id="monitormediaservicesaccount"></a>Überwachen eines Media Services-Kontos

Dieser Artikel gehört zur Reihe [Media Services: Video-on- Demand-Workflow](media-services-video-on-demand-workflow.md) und [Media Services: Livestreaming-Workflow](media-services-live-streaming-workflow.md). 

Das Azure Media Services-Dashboard stellt Nutzungsmetriken und Kontoinformationen dar, die Sie zum Verwalten Ihres Media Services-Konto verwenden können.

Sie können die Anzahl der Codierungsjobs in der Warteschlange, fehlgeschlagene Codierungsaufgaben, aktive Codierungsjobs, die durch die Eingabe- und Ausgabedaten des Encoders dargestellt werden, sowie die Blob-Speichernutzung Ihres Media Services-Kontos überwachen. Wenn Sie Inhalte per Streaming an Kunden übertragen, können Sie außerdem verschiedene Streaming-Metriken abrufen. Sie können Ihre Daten der letzten 6 Stunden, 24 Stunden oder 7 Tage überwachen.
 
>[AZURE.NOTE] Beim Überwachen der Speicherdaten im Windows Azure-Verwaltungsportal entstehen zusätzliche Kosten. Weitere Informationen finden Sie unter [Speicheranalyse und Speicheranalysekosten](http://go.microsoft.com/fwlink/?LinkId=256667).

##<a id="configuremonitoring"></a>Gewusst wie: Überwachen eines Media Services-Kontos

1. Klicken Sie im [Verwaltungsportal](http://go.microsoft.com/fwlink/?LinkID=256666) auf **Media Services** und dann auf den Media Services-Kontonamen, um das Dashboard zu öffnen. 

	![MediaServices_Dashboard][dashboard]

2. Um die Codierungsjobs oder -daten zu überwachen, beginnen Sie einfach mit dem Senden von Codierungsjobs an Media Services oder mit dem Streaming von Inhalten an Kunden mithilfe von Azure Media On-Demand Streaming. Nach ca. einer Stunde sollten auf dem Dashboard Überwachungsdaten angezeigt werden.

##<a id="configuringstorage"></a>Gewusst wie: Überwachen der Blob-Speichernutzung (optional)
1. Klicken Sie im Abschnitt **Auf einen Blick** auf den Namen des **Speicherkontos**.
2. Klicken Sie auf der Speicherkontoseite auf den Link zur **Konfigurationsseite**, und blättern Sie bis zu den Einstellungen zur **Überwachung** für die Blob-, Tabellen- und Warteschlangendienste nach unten.

	>[AZURE.NOTE] Blobs sind der einzige unterstützte Speichertyp in Media Services.

	![StorageOptions][storage_options_scoped]

3. Legen Sie unter **Überwachung** die Überwachungsstufe und die Datenaufbewahrungsrichtlinie für Blobs fest:

-  Zum Festlegen der Überwachungsstufe wählen Sie eine der folgenden Optionen:

      **Minimal** - erfasst Kennzahlen wie Eingang/Ausgang, Verfügbarkeit, Latenz sowie Erfolgsprozentwerte, die für die Blob-, Tabellen- und Warteschlangendienste aggregiert werden.

      **Ausführlich** - sammelt zusätzlich zu den minimalen Kennzahlen den gleichen Satz Kennzahlen für jeden Speichervorgang in der Azure-Speicherdienst-API. Ausführliche Metriken ermöglichen eine genauere Analyse von Problemen, die bei Anwendungsvorgängen auftreten. 

      **Aus** - deaktiviert die Überwachung. Vorhandene Überwachungsdaten bleiben bis zum Ende des Aufbewahrungszeitraums gespeichert.

- Zum Festlegen der Datenaufbewahrungsrichtlinie geben Sie im Feld **Aufbewahrung (Tage)** die Anzahl der Tage ein, für die Daten aufbewahrt werden sollen (zwischen 1 und 365 Tage). Wenn Sie keine Aufbewahrungsrichtlinie festlegen möchten, geben Sie null (0) ein. Ist keine Aufbewahrungsrichtlinie festgelegt, müssen Sie die Überwachungsdaten selbst löschen. Wir empfehlen, die Aufbewahrungsrichtlinie entsprechend dem Zeitraum festzulegen, den Sie Speicheranalysedaten für Ihr Konto behalten möchten, sodass alte und nicht verwendete Analysedaten kostenlos vom System gelöscht werden können.

4. Klicken Sie nach Abschluss der Überwachungskonfiguration auf **Speichern**.
Ähnlich wie bei den Media Services-Metriken sollten nach ca. einer Stunde Überwachungsdaten auf dem Dashboard angezeigt werden.
Metriken werden im Speicherkonto in vier Tabellen namens $MetricsTransactionsBlob, $MetricsTransactionsTable, $MetricsTransactionsQueue und $MetricsCapacityBlob gespeichert. Weitere Informationen finden Sie unter [Kennzahlen zur Speicheranalyse](http://go.microsoft.com/fwlink/?LinkId=256668).


<!-- Images -->
[Dashboard]: ./media/media-services-monitor-services-account/media-services-dashboard.png
[storage_options_scoped]: ./media/media-services-monitor-services-account/storagemonitoringoptions_scoped.png



<!--HONumber=52--> 