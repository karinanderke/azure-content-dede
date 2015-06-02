<properties 
	pageTitle="Aktualisieren der alten Visual Studio Online-Version von Application Insights" 
	description="Aktualisieren vorhandener Projekte"
	services="application-insights" 
    documentationCenter=""
	authors="alancameronwills" 
	manager="ronmart"/>

<tags 
	ms.service="application-insights" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="ibiza" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/21/2015" 
	ms.author="awills"/>
 
# Aktualisieren der alten Visual Studio Online-Version von Application Insights

Dieses Dokument ist für Sie nur von Interesse, wenn Sie über ein Projekt verfügen, das weiterhin die ältere Version von Application Insights verwendet, die Bestandteil von Visual Studio Online war. Diese Version wird zur gegebenen Zeit außer Betrieb genommen, weshalb Sie auf die neue Version aktualisieren sollten, die ein Dienst in Microsoft Azure ist.

## Welche Version habe ich?

Wenn Sie Application Insights über Visual Studio 2013 Update 3 oder höher zu Ihrem Projekt hinzugefügt haben, wird höchstwahrscheinlich die neue Azure-Version verwendet.

Öffnen Sie "ApplicationInsights.config". Wenn die Datei die Knoten `ActiveProfile` und `Profiles` enthält, handelt es sich um die alte Version, die aktualisiert werden sollte.

Oder sehen Sie sich Ihr Projekt im Projektmappen-Explorer von Visual Studio an, und wählen Sie "Microsoft.ApplicationInsights" unter "Verweise" aus. Suchen Sie im Fenster "Eigenschaften" die Version. Wenn der Wert kleiner als 0.12 ist, sollten Sie sie aktualisieren.

## Wenn Sie ein Visual Studio-Projekt haben...

1. Öffnen Sie das Projekt in Visual Studio 2013 Update 3 oder höher.
2. Löschen Sie "ApplicationInsights.config". 
3. Entfernen Sie die Application Insights NuGet-Pakete aus dem Projekt. Klicken Sie hierzu im Projektmappen-Explorer mit der rechten Maustaste auf das Projekt, und wählen Sie "NuGet-Pakete verwalten" aus.
4. SDK: Klicken Sie mit der rechten Maustaste auf das Projekt, und wählen Sie [Application Insights hinzufügen][greenbrown]. Hierdurch wird das SDK zum Projekt hinzugefügt und außerdem eine neue Application Insights-Ressource in Azure erstellt.
5. Protokollierung: Wenn Ihr Code Aufrufe an die alte API wie z. B. "LogEvent()" enthält, werden Sie diese finden, wenn Sie versuchen, die Projektmappe zu erstellen. Aktualisieren Sie sie für das [Verwenden der neuen API][track].
6. Webseiten: Wenn Ihr Projekt Webseiten enthält, ersetzen Sie die Skripts in den Abschnitten <head>. Es gibt in der Regel nur eine Kopie auf einer Masterseite, z. B. "Views\\Shared_Layout.cshtml". [Rufen Sie das neue Skript vom Blatt "Schnellstart" in Ihrer Application Insights-Ressource in Azure ab][usage]. Wenn Ihre Webseiten im Text Telemetrieaufrufe wie z. B. "logEvent" oder "logPage" enthalten, [aktualisieren Sie sie für die Verwendung der neuen API][track].
7. Servermonitor: Wenn Ihre App ein in IIS ausgeführter Dienst ist, deinstallieren Sie Microsoft Monitoring Agent vom Server, und [installieren dann den Application Insights-Statusmonitor][redfield].
8. Webtests: Wenn Sie mit Webverfügbarkeitstests gearbeitet haben, [erstellen Sie diese im neuen Portal samt ihren Warnungen neu][availability].
9. Warnungen: Richten Sie [Warnungen zu Metriken][alerts] im Azure-Portal ein.
10. Leistungsindikatoren: Wenn Sie Leistungsindikatoren verwendet haben, können Sie Ihren eigenen Code zum Erfassen der Leistungsindikatoren in regelmäßigen Abständen schreiben und sie mithilfe von [TrackMetric()][track] senden.

## Wenn Sie einen Java-Webdienst haben...

1. Deaktivieren Sie auf dem Servercomputer den alten Agent durch Entfernen von Verweisen auf den APM-Agent aus der Startdatei des Webdiensts. Bearbeiten Sie auf einem TomCat-Server die Datei "catalina.bat". Bearbeiten Sie auf einem JBoss-Server die Datei "Run.bat". 
2. Starten Sie den Webdienst neu.
3. [Fügen Sie im Microsoft Azure-Portal eine neue Application Insights-Ressource hinzu][java]. Fügen Sie auf Ihrem Entwicklungscomputer [das Java SDK][java] Ihrem Webprojekt hinzu. 
Sie können jetzt aus dem Servercode [benutzerdefinierte Telemetriedaten senden][track].
4. Ersetzen Sie die Skripts in den Abschnitten <head> Ihrer Webseiten. (Möglicherweise ist in einem serverseitigen Include nur eine Kopie enthalten.) [Rufen Sie das neue Skript vom Blatt "Schnellstart" in Ihrer neuen Application Insights-Ressource in Azure ab][usage]. 
Wenn Ihre Webseiten im Text Telemetrieaufrufe wie z. B. "logEvent" oder "logPage" enthalten, [aktualisieren Sie sie für die Verwendung der neuen API][track].



<!--Link references-->

[alerts]: app-insights-alerts.md
[availability]: app-insights-monitor-web-app-availability.md
[greenbrown]: app-insights-start-monitoring-app-health-usage.md
[java]: app-insights-java-get-started.md
[redfield]: app-insights-monitor-performance-live-website-now.md
[track]: app-insights-custom-events-metrics-api.md
[usage]: app-insights-web-track-usage.md


<!--HONumber=54-->