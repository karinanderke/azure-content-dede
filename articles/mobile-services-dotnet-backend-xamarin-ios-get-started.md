﻿<properties urlDisplayName="Get Started with Mobile Services for Xamarin iOS apps" pageTitle="Erste Schritte mit Mobile Services für Xamarin iOS-Apps - Azure Mobile Services" metaKeywords="" description="Follow this tutorial to get started using Azure Mobile Services for Xamarin iOS development" metaCanonical="" services="" documentationCenter="Mobile" title="Get Started with Mobile Services for Xamarin iOS apps" authors="donnam" solutions="" manager="dwrede" editor="mollybos" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-xamarin-ios" ms.devlang="dotnet" ms.topic="article" ms.date="11/11/2014" ms.author="donnam" />

# <a name="getting-started"> </a>Erste Schritte mit Mobile Services

[WACOM.INCLUDE [mobile-services-selector-get-started](../includes/mobile-services-selector-get-started.md)]

In diesem Lernprogramm erfahren Sie, wie Sie mit den Azure Mobile Services einen cloudbasierten Backend-Dienst zu einer Xamarin iOS-App hinzufügen können. In diesem Lernprogramm erstellen Sie einen neuen mobilen Dienst und eine einfache To-Do-Listen-App, die App-Daten im neuen mobilen Dienst speichert. Der von Ihnen erstellte mobile Dienst verwendet die unterstützten .NET-Sprachen unter Verwendung von Visual Studio für die serverseitige Geschäftslogik und zur Verwaltung des mobilen Diensts. Informationen zum Erstellen eines mobilen Diensts, dessen serverseitige Geschäftslogik Sie in JavaScript schreiben können, finden Sie in der [JavaScript-Back-End-Version] dieses Themas.

>[WACOM.NOTE]In diesem Thema wird das Erstellen eines neuen mobilen Dienstprojekts und einer universellen Windows-App mithilfe des Azure-Verwaltungsportals erläutert. Unter Verwendung von Visual Studio 2013 Update 2 können Sie außerdem neue mobile Dienstprojekte zu einer vorhandenen Visual Studio-Lösung hinzufügen. Weitere Informationen finden Sie unter [Schnellstart: Hinzufügen von mobilen Diensten (.NET-Back-End)](http://msdn.microsoft.com/de-de/library/windows/apps/dn629482.aspx)

Unten sehen Sie einen Screenshot aus der fertigen App:

![][0]


Das Abschließen dieses Lernprogramms ist eine Voraussetzung für alle anderen Mobile Services-Lernprogramme für Xamarin iOS-Apps. 

>[WACOM.NOTE]Sie benötigen ein Azure-Konto, um dieses Lernprogramm auszuführen. Falls Sie kein Konto besitzen, können Sie sich für eine Azure-Testversion registrieren. So erhalten Sie bis zu 10 kostenlose mobile Dienste, die Sie auch nach Ablauf der Testversion weiter nutzen können. Ausführliche Informationen erhalten Sie unter <a href="http://www.windowsazure.com/de-de/pricing/free-trial/?WT.mc_id=A0E0E5C02&returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fde-de%2Fdocumentation%2Farticles%2Fmobile-services-dotnet-backend-xamarin-ios-get-started" target="_blank">Kostenloses Azure-Testkonto</a>.<br />Für dieses Lernprogramm ist <a href="https://go.microsoft.com/fwLink/p/?LinkID=257546" target="_blank">Visual Studio Professional 2013</a> erforderlich. Dazu ist eine kostenlose Testversion verfügbar.

## Erstellen eines neuen mobilen Dienstes

[WACOM.INCLUDE [mobile-services-dotnet-backend-create-new-service](../includes/mobile-services-dotnet-backend-create-new-service.md)]

## Erstellen einer neuen Xamarin iOS-App

Sobald Sie den mobilen Dienst erstellt haben, können Sie einem einfachen Schnellstart im Verwaltungsportal folgen, um eine neue App zu erstellen oder eine vorhandene App für die Verbindung zum mobilen Dienst zu ändern. 

In diesem Abschnitt laden Sie eine neue Xamarin iOS-App und ein Dienstprojekt für Ihren mobilen Dienst herunter.

1. Klicken Sie im Verwaltungsportal auf **Mobile Services** und anschließend auf den mobilen Dienst, den Sie gerade erstellt haben.
   
2. Klicken Sie auf der Registerkarte "Schnellstart" unter **Plattform auswählen** auf **Xamarin**, und erweitern Sie **Neue Xamarin-App erstellen**.

   	![][6]

   	Dadurch werden drei einfache Schritte zum Erstellen einer Xamarin iOS-App angezeigt, die mit dem mobilen Dienst verbunden wird.

  	![][7]

3. Falls noch nicht geschehen, müssen Sie <a href="https://go.microsoft.com/fwLink/p/?LinkID=257546" target="_blank">Visual Studio Professional 2013</a> auf den lokalen Computer oder virtuellen Computer herunterladen und installieren.  

4. Laden Sie [Xcode] V. 4.4 oder eine höhere Version sowie [Xamarin Studio] herunter. Sie können auch Xamarin für Visual Studio verwenden.

5. Wählen Sie unter **Service in der Cloud herunterladen und veröffentlichen** die Option **iOS** und klicken Sie auf  **Herunterladen**. 

  	Daraufhin wird eine Lösung heruntergeladen, die Projekte für den mobilen Dienst und für die _To-Do-Listen_-Beispielanwendung, die mit dem mobilen Dienst verbunden ist, enthält. Speichern Sie die komprimierte Projektdatei auf Ihrem lokalen Computer, und notieren Sie sich den Speicherort.

6. Laden Sie das Veröffentlichungsprofil herunter, speichern Sie die heruntergeladene Datei auf dem lokalen Computer, und notieren Sie sich den Speicherort.

## Testen des mobilen Dienstes

[WACOM.INCLUDE [mobile-services-dotnet-backend-test-local-service](../includes/mobile-services-dotnet-backend-test-local-service.md)]

## Veröffentlichen des mobilen Diensts

[WACOM.INCLUDE [mobile-services-dotnet-backend-publish-service](../includes/mobile-services-dotnet-backend-publish-service.md)]

## Ausführen der Xamarin iOS-App

Der letzte Schritt dieses Lernprogramms besteht im Erstellen und Ausführen der neuen App.

1. Navigieren Sie entweder in Visual Studio oder in Xamarin Studio zum Client-Projekt mit der mobilen Dienstlösung.

	![][8]

	![][9]

2. Klicken Sie auf die Schaltfläche **Ausführen**, um das Client-Projekt zu erstellen und die App im iPhone-Simulator zu starten.

3. Geben Sie in der App eine Beschreibung ein, zum Beispiel _Lernprogramm ausführen_, und klicken Sie anschließend auf das Plus-Symbol (**+**).

	![][10]

	Dadurch wird eine POST-Anforderung an den neuen, in Azure gehosteten mobilen Dienst gesendet. Daten von der Anforderung werden in die TodoItem-Tabelle eingefügt. In der Tabelle gespeicherte Einträge werden von dem mobilen Dienst zurückgegeben, und die Daten werden in der Liste angezeigt.

>[WACOM.NOTE]Sie können den Code überprüfen, der auf den mobilen Dienst zum Abfragen und Einfügen von Daten zugreift; Sie finden ihn in der C#-Datei "QSTodoService.cs C#".

    
## Nächste Schritte
Da Sie den Schnellstart jetzt abgeschlossen haben, erfahren Sie, wie zusätzliche wichtige Aufgaben in Mobile Services ausgeführt werden: 

* [Erste Schritte mit der Offline-Datensynchronisierung]
  <br/>Erfahren Sie mehr darüber, wie Sie die Offline-Datensynchronisierung verwenden können, um die App reaktionsfähig und stabil zu machen.

* [Erste Schritte mit der Authentifizierung]
  <br/>Informationen über die Authentifizierung von Benutzern der App mit einem Identitätsanbieter.

* [Erste Schritte mit Pushbenachrichtigungen] 
  <br/>Informationen über das Versenden einer grundlegenden Pushbenachrichtigung an die App.

* [Problembehandlung beim Mobile Services .NET-Back-End]
  <br/> Erfahren Sie mehr zur Diagnose und zum Beheben von Problemen, die mit einem Mobile Services .NET-Back-End auftreten können. 

<!-- Anchors. -->
[Erste Schritte mit Mobile Services]:#getting-started
[Erstellen eines neuen mobilen Dienstes]:#create-new-service
[Nächste Schritte]:#next-steps



<!-- Images. -->
[0]: ./media/mobile-services-dotnet-backend-xamarin-ios-get-started/mobile-quickstart-completed-ios.png
[6]: ./media/mobile-services-dotnet-backend-xamarin-ios-get-started/mobile-portal-quickstart-xamarin-ios.png
[7]: ./media/mobile-services-dotnet-backend-xamarin-ios-get-started/mobile-quickstart-steps-xamarin-ios.png
[8]: ./media/mobile-services-dotnet-backend-xamarin-ios-get-started/mobile-xamarin-project-ios-vs.png
[9]: ./media/mobile-services-dotnet-backend-xamarin-ios-get-started/mobile-xamarin-project-ios-xs.png
[10]: ./media/mobile-services-dotnet-backend-xamarin-ios-get-started/mobile-quickstart-startup-ios.png

<!-- URLs. -->
[Erste Schritte mit der Offline-Datensynchronisierung]: /de-de/documentation/articles/mobile-services-xamarin-ios-get-started-offline-data
[Erste Schritte mit der Authentifizierung]: /de-de/documentation/articles/mobile-services-dotnet-backend-xamarin-ios-get-started-users
[Erste Schritte mit Pushbenachrichtigungen]: /de-de/documentation/articles/mobile-services-dotnet-backend-xamarin-ios-get-started-push
[Visual Studio Professional 2013]: https://go.microsoft.com/fwLink/p/?LinkID=257546
[Mobile Services SDK]: http://go.microsoft.com/fwlink/?LinkId=257545
[JavaScript und HTML]: mobile-services-win8-javascript/
[Verwaltungsportal]: https://manage.windowsazure.com/
[JavaScript-Back-End-Version]: /de-de/documentation/articles/partner-xamarin-mobile-services-ios-get-started
[Erste Schritte mit Daten in Mobile Services mit Visual Studio 2012]: /de-de/documentation/articles/mobile-services-windows-store-dotnet-get-started-data-vs2012
[Problembehandlung beim Mobile Services .NET-Back-End]: /de-de/documentation/articles/mobile-services-dotnet-backend-how-to-troubleshoot/


[Xamarin Studio]: http://xamarin.com/download
[Xcode]: https://go.microsoft.com/fwLink/?LinkID=266532&clcid=0x409
[Xamarin für Windows]: https://go.microsoft.com/fwLink/?LinkID=330242&clcid=0x409