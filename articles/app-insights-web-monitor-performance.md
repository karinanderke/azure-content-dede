<properties title="Monitor your app's health and usage with Application Insights" pageTitle="Monitor your app's health and usage with Application Insights" description="Get started with Application Insights. Analyze usage, availability and performance of your on-premises or Microsoft Azure applications." metaKeywords="analytics monitoring application insights" authors="awills"  manager="kamrani" />

<tags ms.service="application-insights" ms.workload="tbd" ms.tgt_pltfrm="ibiza" ms.devlang="na" ms.topic="article" ms.date="2014-09-24" ms.author="awills"></tags>

# Die Leistung in Webanwendungen überwachen

*Application Insights befindet sich in der Vorschau.*

Stellen sie sicher, dass Ihre Anwendung optimal funktioniert, und stellen Sie Fehler umgehend fest. [Application Insights][start] informiert Sie über alle Leistungsprobleme und Ausnahmefälle. So können Sie die Ursachen schnell ermitteln und diagnostizieren.

Application Insights kann sowohl auf lokalen und virtuellen Maschinen gehostete ASP.NET-Webanwendungen als auch Microsoft Azure-Websites überwachen.

-   [Leistungsüberwachung einrichten](#setup)
-   [Daten anzeigen](#view)
-   [Was bedeutet was?](#metrics)
-   [Probleme diagnostizieren](#diagnosis)
-   [Nächste Schritte](#next)

## <a name="setup"></a>Leistungsüberwachung einrichten

Falls Sie Application Insights Ihrem Projekt noch nicht hinzugefügt haben (d. h., wenn es nicht über ApplicationInsights.config verfügt), gehen Sie nach einer der folgenden Methoden vor, um zu beginnen:

-   [Application Insights Ihrem App-Projekt in Visual Studio hinzufügen][start] (empfohlene Methode). Sie können sowohl passive Leistungsüberwachung als auch Diagnoseprotokollierung und Nutzungsverfolgung einsetzen.
-   [Leistung einer Live-Website jetzt überwachen][redfield] – Bei dieser Methode müssen Sie das App-Projekt nicht aktualisieren bzw. die Webseite nicht erneut bereitstellen.

## <a name="view"></a>Berichte anzeigen

Starten Sie Ihre Anwendung mit F5, und probieren Sie es aus – öffnen Sie verschiedene Seiten.

In Visual Studio sehen Sie eine Anzahl der empfangenen Ereignisse.

![](./media/appinsights/appinsights-09eventcount.png)

Öffnen Sie Application Insights von Ihrem Projekt aus.

![Klicken Sie mit der rechten Maustaste auf Ihr Projekt, und öffnen Sie das Azure-Portal](./media/appinsights/appinsights-04-openPortal.png)

Suchen Sie Daten in den **Application health**-Tiles. Zuerst sehen Sie lediglich einen oder zwei Punkte. Beispiel:

![Zu mehr Daten durchklicken](./media/appinsights/appinsights-41firstHealth.png)

Beim Betrieb im Debugmodus wird Telemetrie über die Pipeline geliefert, sodass Ihnen innerhalb von wenigen Sekunden Daten angezeigt werden. Sobald Sie Ihre Anwendung bereitstellen, sammeln sich die Daten langsamer an.

Falls Sie zu Beginn keine Daten sehen, warten Sie eine Minuten und klicken dann auf "Aktualisieren".

### Metriken erforschen

Klicken Sie auf ein beliebiges Tile, um weitere Details und Ergebnisse über einen längeren Zeitraum anzuzeigen. Klicken Sie beispielsweise auf das Tile "Requests", und wählen Sie dann einen Zeitraum aus.

![Zu mehr Daten durchklicken und einen Zeitraum auswählen](./media/appinsights/appinsights-48metrics.png)

Klicken Sie auf einen Graphen, um auszuwählen, welche Metrik dieser anzeigen soll:

![Auf Graph klicken, um Metrik auszuwählen](./media/appinsights/appinsights-61perfchoices.png)

> [AZURE.NOTE] **Wählen Sie alle Metriken ab**, um die insgesamt verfügbare Auswahl anzuzeigen. Die Metriken werden in Gruppen unterteilt. Wenn ein Mitglied einer Gruppe ausgewählt wird, werden nur die weiteren Mitglieder dieser Gruppe angezeigt.

## <a name="metrics"></a>Was bedeutet was? Leistungs-Tiles und Berichte

Ihnen steht eine Vielzahl von Leistungsmetriken zur Verfügung. Lassen Sie uns mit denen beginnen, die standardmäßig im Anwendungsfenster angezeigt werden.

### Requests

Die in einem angegebenen Zeitraum empfangenen HTTP-Anforderungen. Vergleichen Sie den Wert mit den Ergebnissen anderer Berichte, um das Verhalten Ihrer Anwendung mit wechselnder Last zu beurteilen.

HTTP-Anforderungen umfassen alle GET- oder POST-Anforderungen für Seiten, Daten und Bilder.

Klicken Sie auf das Tile, um Zählwerte für bestimmte URLs zu erhalten.

### Average response time

Misst die Zeit zwischen dem Eingang einer Webanforderung bei Ihrer Anwendung und der zurückgegebenen Antwort.

Die Punkte zeigen einen variablen Durchschnittswert. Falls viele Anforderungen eingehen, gibt es ggf. vom Durchschnitt deutlich nach oben oder unten abweichende Spitzen im Graphen.

Achten Sie auf ungewöhnliche Spitzen. Im Allgemeinen können Sie mit einer längeren Antwortzeit bei steigender Zahl der Anforderungen rechnen. Falls der Anstieg nicht proportional ist, gelangt Ihre Anwendung möglicherweise an Ressourcengrenzen wie die CPU-Leistung oder die Kapazität eines verwendeten Diensts.

Klicken Sie auf das Tile, um Zeiten für bestimmte URLs zu erhalten.

![](./media/appinsights/appinsights-42reqs.png)

### Slowest requests

![](./media/appinsights/appinsights-44slowest.png)

Zeigt, welche Anforderungen möglicherweise eine Leistungsfeinabstimmung erfordern.

### Failed requests

![](./media/appinsights/appinsights-46failed.png)

Zählwert von Anforderungen, die nicht abgefangene Ausnahmefehler verursacht haben.

Klicken Sie auf das Tile, um Details zu bestimmten Fehlern anzuzeigen, und wählen Sie einzelne Anforderungen aus, um die jeweiligen Details anzuzeigen.

Nur eine repräsentative Menge an Fehler wird zur individuellen Überprüfung zurückgehalten.

### Other metrics

Um andere verfügbare Metriken aufzurufen, klicken Sie auf einen Graphen und wählen dann alle Metriken ab. Daraufhin werden alle verfügbaren Metriken angezeigt. Klicken Sie auf (i), um die jeweilige Definition der Metrik anzuzeigen.

![Alle Metriken abwählen, um alle verfügbaren anzuzeigen](./media/appinsights/appinsights-62allchoices.png)

Wenn Sie eine Metrik auswählen, werden alle anderen ausgeblendet, die nicht im selben Diagramm angezeigt werden können.

## <a name="diagnosis"></a>Probleme diagnostizieren

Im Folgenden finden Sie einige Tipps zum Feststellen und Diagnostizieren von Leistungsproblemen:

-   Richten Sie [Webtests][availability] ein, um benachrichtigt zu werden, falls Ihre Website nicht erreichbar ist oder fehlerhaft bzw. langsam reagiert.
-   Vergleichen Sie den Request-Zählwert mit anderen Metriken, um festzustellen, ob Fehler oder langsame Reaktionen mit der Last zusammenhängen.
-   [Fügen Sie Ihrem Code Trace-Anweisungen hinzu bzw. suchen Sie diese][diagnostic], um Probleme besser einzukreisen.

## <a name="next"></a>Nächste Schritte

[Webtests][availability] – Lassen Sie in regelmäßigen Abständen aus aller Welt Webanforderungen zu Ihrer Anwendung senden.

[Diagnostische Spuren protokollieren und suchen][diagnostic] – Fügen Sie Trace-Aufrufe ein, und durchsuchen Sie die Ergebnisse, um Probleme zu lokalisieren.

[Nutzungsnachverfolgung][usage] – Erfahren Sie, wie Ihre Anwendung genutzt wird.

[Problembehandlung][qna] – sowie Fragen und Antworten

## Weitere Informationen

-   [Application Insights – erste Schritte][start]
-   [Live-Webserver jetzt überwachen][redfield]
-   [Leistung in Webanwendungen überwachen][perf]
-   [Diagnoseprotokolle durchsuchen][diagnostic]
-   [Nachverfolgung der Verfügbarkeit mit Webtests][availability]
-   [Nutzung nachverfolgen][usage]
-   [Fragen und Antworten, Problembehandlung][qna]

<!--Link references-->


[start]: ../app-insights-start-monitoring-app-health-usage/
[redfield]: ../app-insights-monitor-performance-live-website-now/
[perf]: ../app-insights-web-monitor-performance/
[diagnostic]: ../app-insights-search-diagnostic-logs/ 
[availability]: ../app-insights-monitor-web-app-availability/
[usage]: ../app-insights-web-track-usage/
[qna]: ../app-insights-troubleshoot-faq/
[webclient]: ../app-insights-start-monitoring-app-health-usage/#webclient
