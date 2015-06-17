<properties 
	pageTitle="Azure WebJobs-Dokumentationsressourcen" 
	description="Empfohlene Ressourcen zum Erlernen der Verwendung von Azure-Webaufträgen und des Azure-Webaufträge-SDK" 
	services="app-service\web" 
	documentationCenter=".net" 
	authors="tdykstra" 
	manager="wpickett" 
	editor="jimbe"/>

<tags 
	ms.service="app-service-web" 
	ms.workload="web" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="06/07/2015" 
	ms.author="tdykstra"/>

# Azure WebJobs-Dokumentationsressourcen

## Übersicht

Dieses Thema enthält Links zu Dokumentationsressourcen zur Verwendung von Azure-Webaufträgen und dem Azure-Webaufträge-SDK. Azure-Webaufträge bieten eine einfache Möglichkeit zum Ausführen von Skripts oder Programmen als Hintergrundprozesse in [App Service-Web-Apps](http://go.microsoft.com/fwlink/?LinkId=529714). Sie können eine ausführbare Datei, z. B. eine CMD-, BAT-, EXE (.NET)-, PS1-, SH-, PHP, PY-, JS- und JAR-Datei, hochladen und ausführen. Diese Programme werden als Webaufträge nach einem Zeitplan (Cron) oder laufend ausgeführt.

Das Webaufträge-SDK vereinfacht die Verwendung des Azure-Speichers. Das Webaufträge-SDK verfügt über ein Bindungs- und Auslösersystem, das Microsoft Azure-Speicher-BLOBs, Warteschlangen und Tabellen sowie Service Bus-Warteschlangen verwendet.

Das Erstellen, Bereitstellen und Verwalten von Webaufträgen erfolgt dank der integrierten Tools in Visual Studio nahtlos. Sie können Webaufträge aus Vorlagen erstellen, veröffentlichen und verwalten (Ausführen/Anhalten/Überwachen/Debuggen).

Das Webauftrags-Dashboard im Azure-Portal bietet leistungsstarke Verwaltungsfunktionen, mit denen Sie vollständige Kontrolle über die Ausführung von Webaufträgen haben, einschließlich der Möglichkeit, einzelne Funktionen innerhalb der Webaufträge aufzurufen. Das Dashboard zeigt auch Funktionslaufzeiten und Protokollausgaben an.

##<a name="getstarted"></a>Erste Schritte mit Webaufträgen und dem Webaufträge-SDK

* [Einführung in Azure-Webaufträge](http://www.hanselman.com/blog/IntroducingWindowsAzureWebJobs.aspx)
* [Azure WebJobs sind großartig, und Sie werden gestartet, mit ihnen jetzt!](http://www.troyhunt.com/2015/01/azure-webjobs-are-awesome-and-you.html) (Blogbeitrag von Troy Hunt).
* [Funktionen von Azure-Webaufträgen](/blog/2014/10/22/webjobs-goes-into-full-production/)
* [Was ist das Webaufträge-SDK?](websites-dotnet-webjobs-sdk.md)
* [Anleitungen für Hintergrundaufträge von Microsoft Patterns and Practices](https://github.com/mspnp/azure-guidance/blob/master/Background-Jobs.md)
* [Ankündigung der Version 1.0.0 RTM des Microsoft Azure-Webaufträge-SDK](/blog/2014/10/25/announcing-the-1-0-0-rtm-of-microsoft-azure-webjobs-sdk/)
* [Erste Schritte mit dem Azure-Webaufträge-SDK](websites-dotnet-webjobs-sdk-get-started.md)
* [Verwenden von Azure-Warteschlangenspeicher mit dem Webaufträge-SDK](websites-dotnet-webjobs-sdk-storage-queues-how-to.md)
* [Verwenden von Azure-BLOB-Speicher mit dem Webaufträge-SDK](websites-dotnet-webjobs-sdk-storage-blobs-how-to.md)
* [Verwenden von Azure-Tabellenspeicher mit dem Webaufträge-SDK](websites-dotnet-webjobs-sdk-storage-tables-how-to.md)
* [Verwenden von Azure Service Bus mit dem Webaufträge-SDK](websites-dotnet-webjobs-sdk-service-bus.md)
* [Kurzübersicht zum Azure-Webaufträge-SDK (PDF zum Herunterladen)](http://go.microsoft.com/fwlink/?LinkID=524028&clcid=0x409)
* Videos
	* [Webaufträge und das Webaufträge-SDK](http://channel9.msdn.com/Shows/Cloud+Cover/Episode-153-WebJobs-with-Pranav-Rastogi?utm_source=dlvr.it&utm_medium=twitter)
	* [Videoreihe zu Azure-Webaufträgen auf Channel 9](http://channel9.msdn.com/Tags/azurefridaywebjobs)
	* [Einführung in Webaufträge-Tools für Visual Studio](http://channel9.msdn.com/Shows/Web+Camps+TV/Introducing-WebJobs-Tooling-for-Visual-Studio-with-Brady-Gaster) 
	* [Webaufträge-Tools und Remotedebuggen](http://channel9.msdn.com/Shows/Web+Camps+TV/WebJobs-GA-Series-Episode-1-WebJobs-Tooling-with-Brady-Gaster)

Weitere Informationen finden Sie auch in den Abschnitten [Bereitstellen von Webaufträgen](#deploy) und [Testen und Debuggen von Webaufträgen](#debug).

##<a name="deploy"></a>Bereitstellen von Webaufträgen

* [Bereitstellen von Azure-Webaufträgen mit Visual Studio](websites-dotnet-deploy-webjobs.md)
* [Bereitstellen von Webaufträgen mithilfe des Azure-Verwaltungsportals](web-sites-create-web-jobs.md)
* [Aktivieren der befehlszeilengesteuerten oder der kontinuierlichen Bereitstellung von Azure-Webaufträgen](http://azure.microsoft.com/blog/2014/08/18/enabling-command-line-or-continuous-delivery-of-azure-webjobs/)
* [Git-Bereitstellung einer .NET-Konsolen-App für Azure mithilfe von Webaufträgen](http://blog.amitapple.com/post/73574681678/git-deploy-console-app/)
* [Ein F\#-WebJob Bereitstellen in Azure.](http://blogs.msdn.com/b/dave_crooks_dev_blog/archive/2015/02/18/deploying-f-web-job-to-azure.aspx)
* Videos
	* [Einführung in Webaufträge-Tools für Visual Studio](http://channel9.msdn.com/Shows/Web+Camps+TV/Introducing-WebJobs-Tooling-for-Visual-Studio-with-Brady-Gaster) 
	* [Webaufträge-Tools und Remotedebuggen](http://channel9.msdn.com/Shows/Web+Camps+TV/WebJobs-GA-Series-Episode-1-WebJobs-Tooling-with-Brady-Gaster) 

##<a name="schedule"></a>Planen von Webaufträgen

* [Das Dialogfeld "Azure-Webauftrag hinzufügen"](websites-dotnet-deploy-webjobs.md#configure)
* [Erstellen geplanter Webaufträge im Azure-Verwaltungsportal](web-sites-create-web-jobs.md#CreateScheduled)

##<a name="debug"></a>Testen und Debuggen von Webaufträgen

* [Neue Entwickler- und Debuggingfunktionen für Azure-Webaufträge in Visual Studio](http://blogs.msdn.com/b/webdev/archive/2014/11/12/new-developer-and-debugging-features-for-azure-webjobs-in-visual-studio.aspx)
* [Anzeigen des Webauftrags-Dashboards](websites-dotnet-webjobs-sdk-get-started.md#view-the-webjobs-sdk-dashboard)
* [Schreiben von Protokollen mit dem Webaufträge-SDK und Anzeigen im Dashboard](websites-dotnet-webjobs-sdk-storage-queues-how-to.md#logs)
* [Remotedebuggen von Webaufträgen](web-sites-dotnet-troubleshoot-visual-studio.md#remotedebugwj)
* [Wer hat dieses Blob geschrieben?](http://blogs.msdn.com/b/jmstall/archive/2014/02/19/who-wrote-that-blob.aspx) 
* [Hosten von interaktivem Code in der Cloud](http://blogs.msdn.com/b/jmstall/archive/2014/04/26/hosting-interactive-code-in-the-cloud.aspx)
* [Abrufen eines Dashboards für die lokale Entwicklung mit dem Webaufträge-SDK](http://blogs.msdn.com/b/jmstall/archive/2014/01/27/getting-a-dashboard-for-local-development-with-the-webjobs-sdk.aspx)
* [Hinzufügen der Ablaufverfolgung zu Azure-Webaufträgen](http://blogs.msdn.com/b/mcsuksoldev/archive/2014/09/04/adding-trace-to-azure-web-sites-and-web-jobs.aspx)
* [Microsoft Azure-Speicher: Überwachung, Diagnose und Problembehandlung](../storage-monitoring-diagnosing-troubleshooting/)
* Videos
	* [Webaufträge-Tools und Remotedebuggen](http://channel9.msdn.com/Shows/Web+Camps+TV/WebJobs-GA-Series-Episode-1-WebJobs-Tooling-with-Brady-Gaster) 

##<a name="scale"></a>Skalieren von Webaufträgen

* [Skalieren Ihrer Webanwendung mit Azure-Websites](http://msdn.microsoft.com/magazine/dn786914.aspx)
* [Azure App Service: Entwickeln sehr vieler einsatzbereiter Web-Apps](https://channel9.msdn.com/Events/Build/2014/3-626). Behandelt die Skalierung von Web-Apps mit Webaufträgen, einschließlich des Webaufträge-SDK.
* Videos
	* [Horizontales Skalieren von Webaufträgen](http://channel9.msdn.com/Shows/Azure-Friday/Azure-WebJobs-105-Scaling-out-Web-Jobs)

##<a name="additional"></a>Zusätzliche Ressourcen für Webaufträge

* [Blogbeitrag zur allgemeinen Verfügbarkeit von Azure-Webaufträgen von Magnus Mårtensson](http://magnusmartensson.com/azure-webjobs-ga)
* [Dokumentation zu Webauftragseinstellungen in GitHub](https://github.com/projectkudu/kudu/wiki/Web-jobs).
* [Ausführen von PowerShell-Webaufträgen in Azure App Service](http://blogs.msdn.com/b/nicktrog/archive/2014/01/22/running-powershell-web-jobs-on-azure-websites.aspx)
* [Erhalten von Benachrichtigungen, wenn die von Azure ausgelösten Webaufträge abgeschlossen sind](http://blog.amitapple.com/post/2014/03/webjobs-notification/)
* [Aufbewahrungsrichtlinie für einfache Web-App-Sicherung mit Webaufträgen](http://azure.microsoft.com/blog/2014/04/28/simple-web-site-backup-retention-policy-with-webjobs/)
* [Azure-Web-Apps und Cloud-Dienste langsam bei erster Anforderung](http://wp.sjkp.dk/windows-azure-websites-and-cloud-services-slow-on-first-request/). Zeigt die Verwendung von Webaufträgen zur Simulation der Funktion "AlwaysOn", die nur für die Standardpreisstufe verfügbar ist.
* [Ordnungsgemäßes Herunterfahren von Webaufträgen](http://blog.amitapple.com/post/2014/05/webjobs-graceful-shutdown/#.U72Il_5OWUl). Informationen zum ordnungsgemäßen Herunterfahren des Webaufträge-SDK finden Sie unter [Ordnungsgemäßes Herunterfahren](websites-dotnet-webjobs-sdk-storage-queues-how-to.md#graceful).)
* [Automatisierung von Sicherungen mit Azure-Webaufträgen und AzCopy](http://markjbrown.com/azure-webjobs-azcopy/)
* Videos
	* [Videos zu Azure-Webaufträgen von Magnus Mårtensson](https://www.youtube.com/playlist?list=PLqp1ZOYYUSd81yEzMYLTw8cz91wx_LU9r)
	* [Videoreihe zu Azure-Webaufträgen auf Channel 9](http://channel9.msdn.com/Tags/azurefridaywebjobs)

##<a name="additionalsdk"></a>Zusätzliche Ressourcen für das Webaufträge-SDK

* [Quellcode des Webaufträge-SDK](https://github.com/Azure/azure-webjobs-sdk)
* [Wie funktioniert [BlobTrigger]?](http://blogs.msdn.com/b/jmstall/archive/2014/04/17/how-does-blobinput-work.aspx) 
* [Erweiterte Bindungen mit dem Azure-Webaufträge-SDK](http://victorhurdugaci.com/advanced-bindings-with-the-windows-azure-web-jobs-sdk/)
* [Webauftrag zum Hochladen von FREB-Dateien in Azure-Speicher mithilfe des Webaufträge-SDK](http://thenextdoorgeek.com/post/WAWS-WebJob-to-upload-FREB-files-to-Azure-Storage-using-the-WebJobs-SDK)
* [Hosten von Azure-Webaufträgen außerhalb von Azure mit den Protokollierungsvorteilen eines auf Azure geosteten Webauftrags](http://bypassion.dk/?p=510)
* [Erstellen eines Tools zum Importieren von Daten mit Azure-Webaufträgen](http://www.freshconsulting.com/building-data-import-tool-azure-webjobs/)
* Videos
	* [Videoreihe zu Azure-Webaufträgen auf Channel 9](http://channel9.msdn.com/Tags/azurefridaywebjobs)

##<a name="samples"></a>Beispielanwendungen für Webaufträge

* [Vom Webaufträge-Team von GitHub bereitgestellte Beispielanwendungen](https://github.com/azure/azure-webjobs-sdk-samples)
* [Einfache Azure-Web-App mit Webauftrags-Back-End mit dem Webaufträge-SDK](http://code.msdn.microsoft.com/Simple-Azure-Website-with-b4391eeb)
* [SiteMonitR](http://code.msdn.microsoft.com/SiteMonitR-dd4fcf77). Veranschaulicht die Verwendung von geplanten und ereignisgesteuerten Webaufträgen. Weitere Informationen finden Sie im Blogbeitrag [Neuerstellung von SiteMonitR mithilfe des Azure-Webaufträge-SDK](http://www.bradygaster.com/post/rebuilding-the-sitemonitr-using-windows-azure-webjobs).

##<a name="blogs"></a>Blogs

* [Azure-Blog](/blog)
* [Apple-Blog von Amit](http://blog.amitapple.com/). Behandelt Webaufträge (aber nicht das SDK).
* [Blog von Magnus Mårtensson](http://magnusmartensson.com/)

##<a name="gethelp"></a>Hilfe zu Webaufträgen

* [Stapelüberlauf bei Webaufträgen](http://stackoverflow.com/questions/tagged/azure-webjobs)
* [Stapelüberlauf beim Webaufträge-SDK](http://stackoverflow.com/questions/tagged/azure-webjobssdk)
* [Forum für Azure und ASP.NET](http://forums.asp.net/1247.aspx)
* [Forum für Azure App Service-Web-Apps](http://social.msdn.microsoft.com/Forums/azure/home?forum=windowsazurewebsitespreview)
* [User Voice-Website für Azure-Web-Apps](http://feedback.azure.com/forums/169385-websites)
* [Twitter](http://twitter.com/). Verwenden Sie das Hashtag \#AzureWebJobs
* [Ein WebJobs-Fehler oder ein Problem melden](https://github.com/projectkudu/kudu/wiki/Reporting-WebJobs-issues)

## Änderungen
* Hinweise zu den Veränderungen von Websites zum App Service finden Sie unter: [Azure App Service und vorhandene Azure-Dienste](http://go.microsoft.com/fwlink/?LinkId=529714).
* Hinweise zu den Veränderungen des neuen Portals gegenüber dem alten finden Sie unter [Referenz zur Navigation im Azure-Portal](http://go.microsoft.com/fwlink/?LinkId=529715)
 

<!---HONumber=GIT-SubDir_Tue_AM_dede-->