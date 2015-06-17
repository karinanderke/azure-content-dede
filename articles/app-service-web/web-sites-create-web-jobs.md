<properties 
	pageTitle="Ausführen von Hintergrundaufgaben mit Webaufträgen" 
	description="Erfahren Sie, wie Sie Hintergrundaufgaben in Azure-Web-Apps ausführen." 
	services="app-service\web" 
	documentationCenter="" 
	authors="tdykstra" 
	writer="tdykstra" 
	manager="wpickett" 
	editor="jimbe"/>

<tags 
	ms.service="app-service-web" 
	ms.workload="web" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="03/24/2015" 
	ms.author="tdykstra"/>

# Ausführen von Hintergrundaufgaben mit Webaufträgen

## Übersicht

Sie können Programme oder Skripts in Webaufträgen auf drei Arten in Ihrer [App Service](http://go.microsoft.com/fwlink/?LinkId=529714)-Web-App ausführen: bei Bedarf, kontinuierlich oder nach einem Zeitplan. Für die Nutzung von Webaufträgen fallen keine zusätzlichen Kosten an.

Dieser Artikel zeigt, wie Sie Webaufträge mithilfe des [Azure-Portals](http://go.microsoft.com/fwlink/?LinkId=529715) bereitstellen. Informationen zum Bereitstellen über Visual Studio oder einen kontinuierlichen Bereitstellungsprozess finden Sie unter [Bereitstellen von Azure-Webaufträgen in Azure-Web-Apps](websites-dotnet-deploy-webjobs.md).

Das Azure WebJobs-SDK vereinfacht zahlreiche WebJobs-Programmieraufgaben. Weitere Informationen finden Sie unter [Was ist das WebJobs-SDK?](websites-dotnet-webjobs-sdk.md).

## <a name="acceptablefiles"></a>Akzeptable Dateitypen für Skripts oder Programme

Die folgenden Dateitypen werden akzeptiert:

* .cmd, .bat, .exe (bei Verwendung von windows cmd)
* .ps1 (bei Verwendung von powershell)
* .sh (bei Verwendung von bash)
* .php (bei Verwendung von php)
* .py (bei Verwendung von python)
* .js (bei Verwendung von node)
* .jar (bei Verwendung von Java)

## <a name="CreateOnDemand"></a>Erstellen eines bedarfsgesteuerten Webauftrags im Portal

1. Klicken Sie auf dem Blatt **Web-App** im [Azure-Portal](http://portal.azure.com) auf **Alle Einstellungen > Webaufträge**, um das Blatt **Webaufträge** anzuzeigen.
	
	![Webauftrag, Blatt](./media/web-sites-create-web-jobs/wjblade.png)
	
5. Klicken Sie auf **Hinzufügen**. Das Dialogfeld **Webauftrag hinzufügen** wird angezeigt.
	
	![Webauftrag hinzufügen, Blatt](./media/web-sites-create-web-jobs/addwjblade.png)
	
2. Geben Sie unter **Name** einen Namen für den Webauftrag ein. Der Name muss mit einem Buchstaben oder einer Ziffer beginnen und darf nur die Sonderzeichen "-" und "_" enthalten.
	
4. Wählen Sie im Feld für die Ausführungshäufigkeit die Option **Bedarfsgesteuert ausführen** aus.
	
3. Klicken Sie im Feld **Dateiupload** auf das Ordnersymbol und navigieren Sie zu der ZIP-Datei mit dem Skript. Die ZIP-Datei sollte die ausführbare Datei ((.exe, .cmd, .bat, .sh, .php, .py, .js) und alle Hilfsdateien enthalten, die zum Ausführen des Programms oder Skripts benötigt werden.
	
5. Aktivieren Sie **Erstellen**, um das Skript in Ihre Web-App hochzuladen.
	
	Der Name, den Sie dem Webauftrag gegeben haben, wird in der Liste auf dem Blatt **Webaufträge** angezeigt.
	
6. Zum Ausführen des Webauftrags klicken Sie mit der rechten Maustaste in die Liste und klicken dann auf **Ausführen**.
	
	![Webauftrag ausführen](./media/web-sites-create-web-jobs/runondemand.png)
	
## <a name="CreateContinuous"></a>Erstellen eines kontinuierlich ausgeführten Webauftrags

1. Führen Sie zum Erstellen eines kontinuierlich ausgeführten Webauftrags dieselben Schritte wie zum Erstellen eines einmalig ausgeführten Webauftrags aus, wählen Sie im Feld **Ausführungshäufigkeit** jedoch **Kontinuierlich** aus.

2. Klicken Sie zum Starten oder Beenden eines kontinuierlich ausgeführten Webauftrags in der Liste mit der rechten Maustaste auf den Webauftrag, und klicken Sie dann auf **Starten** oder **Beenden**.
	
> [AZURE.NOTE]Wenn die Web-App auf mehr als einer Instanz ausgeführt wird, wird auf allen Instanzen ein kontinuierlich ausgeführter Webauftrag ausgeführt. Bedarfsgesteuerte und geplante Webaufträge werden auf einer einzigen Instanz ausgeführt, die für den Lastenausgleich durch Microsoft Azure ausgewählt wurde.
	
> Aktivieren Sie für fortlaufende WebJobs zuverlässig und für alle Instanzen ausgeführt werden, die immer auf * Konfigurationseinstellung für die Web-app andernfalls können sie beendet werden, sobald der SCM-Hostwebsite zu lange im Leerlauf ist.

## <a name="CreateScheduled"></a>Erstellen geplanter Webaufträge

Das Azure-Verwaltungsportal bietet noch nicht die Möglichkeit, einen geplanten Webauftrag zu erstellen. Bis diese Funktion implementiert wurde, können Sie hierfür das [alte Portal](http://manage.windowsazure.com) verwenden.

1. Navigieren Sie im [alten Portal](http://manage.windowsazure.com) zur Webauftragsseite, und klicken Sie auf **Hinzufügen**.

1. Wählen Sie im Feld **Ausführungshäufigkeit** die Option **Gemäß einem Zeitplan ausführen** aus.
	
	![Neuer geplanter Auftrag][NewScheduledJob]
	
2. Wählen Sie die Scheduler-Region für den Auftrag aus, und klicken Sie dann auf den Pfeil unten rechts im Dialogfeld, um zum nächsten Bildschirm zu wechseln.

3. Wählen Sie im Dialogfeld **Auftrag erstellen** den Typ der **Wiederholung** aus: **Einmaliger Auftrag** oder **Periodischer Auftrag**.
	
	![Wiederholung planen][SchdRecurrence]
	
4. Wählen Sie außerdem eine **Startzeit** aus: **Jetzt** oder **Bestimmte Zeit**.
	
	![Geplante Startzeit][SchdStart]
	
5. Wenn Sie zu einer bestimmten Zeit starten möchten, wählen Sie die Startzeitwerte unter **Startet am** aus.
	
	![Start zu einer bestimmten Zeit planen][SchdStartOn]
	
6. Wenn Sie einen periodischen Auftrag ausgewählt haben, können Sie mit der Option **Wiederholen alle** die Häufigkeit und mit der Option **Endet am** eine Endzeit angeben.
	
	![Wiederholung planen][SchdRecurEvery]
	
7. Wenn Sie **Wochen** auswählen, können Sie das Kontrollkästchen **In einem bestimmten Zeitplan** aktivieren und die Wochentage angeben, an denen der Auftrag ausgeführt werden soll.
	
	![Wochentage planen][SchdWeeksOnParticular]
	
8. Wenn Sie **Monate** auswählen und das Kontrollkästchen **In einem bestimmten Zeitplan** aktivieren, können Sie festlegen, dass der Auftrag an bestimmten bezifferten Tagen im Monat ausgeführt wird.
	
	![Bestimmte Daten im Monat planen][SchdMonthsOnPartDays]
	
9. Wenn Sie **Wochentage** auswählen, können Sie angeben, an welchem Wochentag bzw. welchen Wochentagen im Monat der Auftrag ausgeführt werden soll.
	
	![Bestimmte Wochentage in einem Monat planen][SchdMonthsOnPartWeekDays]
	
10. Schließlich können Sie mit der Option **Vorkommen** auswählen, in welcher Woche im Monat (erste, zweite, dritte usw.) der Auftrag an den angegebenen Wochentagen ausgeführt werden soll.
	
	![Bestimmte Wochentage in bestimmten Wochen in einem Monat planen][SchdMonthsOnPartWeekDaysOccurences]
	
11. Nachdem Sie mindestens einen Auftrag erstellt haben, werden die Namen zusammen mit dem Status, dem Zeitplantyp und anderen Informationen auf der Registerkarte WebJobs angezeigt. Die Verlaufsinformationen für die letzten 30 Webaufträge werden gespeichert.
	
	![Auftragsliste][WebJobsListWithSeveralJobs]
	
### <a name="Scheduler"></a>Geplante Aufträge und Azure Scheduler

Geplante Aufträge können auf den Seiten im Azure Scheduler des [alten Portals](http://manage.windowsazure.com) weiter konfiguriert werden.

1.	Klicken Sie auf der Seite **WebJobs** auf den Link Zeitplan des Auftrags, um zur Azure Scheduler-Portalseite zu navigieren. 
	
	![Link zu Azure Scheduler][LinkToScheduler]
	
2. Klicken Sie auf der Seite Scheduler auf den Auftrag.
	
	![Auftrag auf der Scheduler-Portalseite][SchedulerPortal]
	
3. Die Seite **Auftragsaktion** wird geöffnet. Dort können Sie den Auftrag weiter konfigurieren.
	
	![Seite "Auftragsaktion" im Scheduler][JobActionPageInScheduler]
	
## <a name="ViewJobHistory"></a>Anzeigen des Auftragsverlaufs

1. Klicken Sie zum Anzeigen des Ausführungsverlaufs eines Auftrags (einschließlich der mit dem Webaufträge-SDK erstellten Aufträge) auf dem Webauftragsblatt in der Spalte **Protokolle** auf den zugehörigen Link. (Sie können das Zwischenablagesymbol verwenden, um die URL der Protokolldateiseite in die Zwischenablage zu kopieren, wenn Sie dies wünschen.)
	
	![Link "Protokolle"](./media/web-sites-create-web-jobs/wjbladelogslink.png)
		
2. Durch das Klicken auf den Link wird die Detailseite des Webauftrags geöffnet. Auf dieser Seite werden der Name des ausgeführten Befehls, die letzten Ausführungszeiten und Angaben zu Erfolg oder Fehlern angezeigt. Klicken Sie unter **Recent job runs** auf eine Zeit, um weitere Details anzuzeigen.
	
	![WebJobDetails][WebJobDetails]
	
3. Die Seite **WebJob Run Details** wird geöffnet. Klicken Sie auf **Toggle Output**, um den Text des Protokollinhalts anzuzeigen. Das Ausgabeprotokoll liegt im Textformat vor.
	
	![Ausführungsdetails zu WebJobs][WebJobRunDetails]
	
4. Klicken Sie auf den Link **Download**, um den Ausgabetext in einem separaten Browserfenster zu öffnen. Klicken Sie mit der rechten Maustaste auf den Link und verwenden Sie die Browseroptionen, um den Dateiinhalt zu speichern, um den Text selbst herunterzuladen.
	
	![Protokollausgabe herunterladen][DownloadLogOutput]
	
5. Der Link  **Webaufträge** oben auf der Seite stellt eine einfache Möglichkeit dar, eine Liste der Webaufträge im Dashboard mit den Verlaufsdaten anzuzeigen.
	
	![Link zur Liste der Webaufträge][WebJobsLinkToDashboardList]
	
	![Liste der Webaufträge im Verlaufsdashboard][WebJobsListInJobsDashboard]
	
	Wenn Sie auf einen dieser Links klicken, werden Sie zur Seite WebJob Details für den ausgewählten Auftrag weitergeleitet.


## <a name="WHPNotes"></a>Hinweise
	
- Ab März 2014 können Web-Apps im kostenlosen Modus nach 20 Minuten ablaufen, wenn keine Anforderungen bei der scm (Bereitstellungs)-Website eingehen und das Web-App-Portal in Azure nicht geöffnet ist. Dieses Verhalten wird durch Anforderungen bei der tatsächlichen Website nicht zurückgesetzt.
- Code für einen fortlaufenden Auftrag muss zur Ausführung in einer Endlosschleife geschrieben sein.
- Kontinuierliche Aufträge werden nur dann fortlaufend ausgeführt, wenn die Web-App aktiv ist.
- In den Modi "Basic" und "Standard" ist das Feature "Immer aktiviert" verfügbar. Ist es aktiviert, wird verhindert, dass Web-Apps in den Leerlauf wechseln.
- Ein Debugging kann nur für kontinuierlich ausgeführte Webaufträge ausgeführt werden. Für geplante oder bei Bedarf ausgeführte Webaufträge wird Debugging nicht unterstützt.

>[AZURE.NOTE]Wenn Sie Azure App Service ausprobieren möchten, ehe Sie sich für ein Azure-Konto anmelden, können Sie unter [App Service testen](http://go.microsoft.com/fwlink/?LinkId=523751) sofort kostenlos eine kurzlebige Starter-Web-App in App Service erstellen. Keine Kreditkarte erforderlich, keine Verpflichtungen.

## <a name="NextSteps"></a>Nächste Schritte
 
Weitere Informationen finden Sie unter [Azure WebJobs Recommended Resources][WebJobsRecommendedResources] (in englischer Sprache).

## Änderungen
* Hinweise zu den Veränderungen von Websites zum App Service finden Sie unter: [Azure App Service und vorhandene Azure-Dienste](http://go.microsoft.com/fwlink/?LinkId=529714).
* Hinweise zu den Veränderungen des neuen Portals gegenüber dem alten finden Sie unter [Referenz zur Navigation im Azure-Portal](http://go.microsoft.com/fwlink/?LinkId=529715)

[PSonWebJobs]: http://blogs.msdn.com/b/nicktrog/archive/2014/01/22/running-powershell-web-jobs-on-azure-websites.aspx
[WebJobsRecommendedResources]: http://go.microsoft.com/fwlink/?LinkId=390226

[OnDemandWebJob]: ./media/web-sites-create-web-jobs/01aOnDemandWebJob.png
[WebJobsList]: ./media/web-sites-create-web-jobs/02aWebJobsList.png
[NewContinuousJob]: ./media/web-sites-create-web-jobs/03aNewContinuousJob.png
[NewScheduledJob]: ./media/web-sites-create-web-jobs/04aNewScheduledJob.png
[SchdRecurrence]: ./media/web-sites-create-web-jobs/05SchdRecurrence.png
[SchdStart]: ./media/web-sites-create-web-jobs/06SchdStart.png
[SchdStartOn]: ./media/web-sites-create-web-jobs/07SchdStartOn.png
[SchdRecurEvery]: ./media/web-sites-create-web-jobs/08SchdRecurEvery.png
[SchdWeeksOnParticular]: ./media/web-sites-create-web-jobs/09SchdWeeksOnParticular.png
[SchdMonthsOnPartDays]: ./media/web-sites-create-web-jobs/10SchdMonthsOnPartDays.png
[SchdMonthsOnPartWeekDays]: ./media/web-sites-create-web-jobs/11SchdMonthsOnPartWeekDays.png
[SchdMonthsOnPartWeekDaysOccurences]: ./media/web-sites-create-web-jobs/12SchdMonthsOnPartWeekDaysOccurences.png
[RunOnce]: ./media/web-sites-create-web-jobs/13RunOnce.png
[WebJobsListWithSeveralJobs]: ./media/web-sites-create-web-jobs/13WebJobsListWithSeveralJobs.png
[WebJobLogs]: ./media/web-sites-create-web-jobs/14WebJobLogs.png
[WebJobDetails]: ./media/web-sites-create-web-jobs/15WebJobDetails.png
[WebJobRunDetails]: ./media/web-sites-create-web-jobs/16WebJobRunDetails.png
[DownloadLogOutput]: ./media/web-sites-create-web-jobs/17DownloadLogOutput.png
[WebJobsLinkToDashboardList]: ./media/web-sites-create-web-jobs/18WebJobsLinkToDashboardList.png
[WebJobsListInJobsDashboard]: ./media/web-sites-create-web-jobs/19WebJobsListInJobsDashboard.png
[LinkToScheduler]: ./media/web-sites-create-web-jobs/31LinkToScheduler.png
[SchedulerPortal]: ./media/web-sites-create-web-jobs/32SchedulerPortal.png
[JobActionPageInScheduler]: ./media/web-sites-create-web-jobs/33JobActionPageInScheduler.png
 

<!---HONumber=GIT-SubDir_Tue_AM_dede-->