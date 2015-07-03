<properties 
	pageTitle="Erste Schritte mit der Authentifizierung (Windows Store) | Mobile Dev Center" 
	description="Erfahren Sie, wie Sie Mobile Services verwenden, um die Benutzer Ihrer Windows Store-App über verschiedene Identitätsanbieter, einschließlich Google, Facebook, Twitter und Microsoft, zu authentifizieren." 
	services="mobile-services" 
	documentationCenter="windows" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.workload="mobile" 
	ms.tgt_pltfrm="mobile-windows-store" 
	ms.devlang="dotnet" 
	ms.topic="article" 
	ms.date="02/26/2015" 
	ms.author="glenga"/>

# Hinzufügen von Authentifizierung zur Mobile Services-App

[AZURE.INCLUDE [mobile-services-selector-get-started-users](../../includes/mobile-services-selector-get-started-users.md)] 

In diesem Thema erfahren Sie, wie Sie Benutzer in Azure Mobile Services über Ihre universelle Windows-App authentifizieren. In diesem Lernprogramm fügen Sie eine Authentifizierung zu dem Schnellstartprojekt hinzu. Sie verwenden dazu einen Identitätsanbieter, der von Mobile Services unterstützt wird. Nach der erfolgreichen Authentifizierung und Autorisierung durch Mobile Services wird der Benutzer-ID-Wert angezeigt.

Dieses Lernprogramm zeigt Ihnen die grundlegenden Schritte zur Aktivierung von Authentifizierung in Ihrer App:

1. [Registrieren Ihrer App für Authentifizierung und Konfigurieren von Mobile Services]
2. [Einschränken von Tabellenberechtigungen für authentifizierte Benutzer]
3. [Hinzufügen von Authentifizierung zur App]
5. [Speichern von Authentifizierungstoken auf dem Client]

Dieses Lernprogramm baut auf dem Mobile Services-Schnellstart auf. Sie müssen zunächst das Lernprogramm [Erste Schritte mit Mobile Services] abschließen.

>[AZURE.NOTE]In diesem Thema erfahren Sie, wie Sie Benutzer in Windows Store- und Windows Phone Store 8.1-Apps authentifizieren. Informationen zu Windows Phone 8.0- und Windows Phone Silverlight 8.1-Apps finden Sie unter [Erste Schritte bei der Authentifizierung in Mobile Services](mobile-services-windows-phone-get-started-users.md).

>Dieses Lernprogramm demonstriert den von Mobile Services verwalteten Authentifizierungsfluss bei Verwendung einer Vielzahl von Identitätsanbietern. Diese Methode lässt sich einfach konfigurieren und unterstützt verschiedene Anbieter. Wenn Sie stattdessen Live Connect mit clientverwalteter Authentifizierung verwenden und eine Möglichkeit für einmaliges Anmelden in Ihrer Windows Phone App bereitstellen möchten, finden Sie weitere Informationen im Thema [Authentifizieren Ihrer Windows Store-App mit auf dem Client verwalteter Authentifizierung mit Microsoft-Konto](mobile-services-windows-store-dotnet-single-sign-on.md). Bei Verwendung der clientseitig verwalteten Authentifizierung hat die App Zugriff auf zusätzliche Benutzerdaten, die vom Identitätsanbieter verwaltet werden. Sie erhalten dieselben Benutzerdaten in Ihrem mobilen Dienst, indem Sie die **user.getIdentities()**-Funktion in Serverskripts aufrufen. Weitere Informationen finden Sie in [diesem Beitrag](http://go.microsoft.com/fwlink/p/?LinkId=506605).

##<a name="register"></a> Registrieren Ihrer App für Authentifizierung und Konfigurieren von Mobile Services

[AZURE.INCLUDE [mobile-services-register-authentication](../../includes/mobile-services-register-authentication.md)] 

##<a name="permissions"></a> Einschränken von Berechtigungen für authentifizierte Benutzer

[AZURE.INCLUDE [mobile-services-restrict-permissions-windows](../../includes/mobile-services-restrict-permissions-windows.md)] 
 
>[AZURE.NOTE]Wenn Sie die Tools von Visual Studio verwenden, um die App mit einem mobilen Dienst zu verbinden, werden zwei Gruppen von **MobileServiceClient**-Definitionen erzeugt, und zwar eine pro Plattform. Dies ist ein guter Moment, den generierten Code zu vereinfachen, indem Sie die in `#if...#endif` gekapselten **MobileServiceClient**-Definitionen zu einer einzigen ungekapselten Definition zusammenfassen, die von beiden Versionen der App genutzt wird. Dies ist nicht erforderlich, wenn Sie die Schnellstart-App aus dem Azure-Verwaltungsportal heruntergeladen haben.

##<a name="add-authentication"></a> Hinzufügen von Authentifizierung zur App

[AZURE.INCLUDE [mobile-services-windows-universal-dotnet-authenticate-app](../../includes/mobile-services-windows-universal-dotnet-authenticate-app.md)] 

Nun können alle von einem der vertrauenswürdigen Identitätsanbieter authentifizierten Benutzer auf die Tabelle *TodoItem* zugreifen. Um benutzerspezifische Daten besser zu schützen, müssen Sie zudem eine Autorisierung implementieren. Hierzu rufen Sie die Benutzer-ID eines Benutzers ab, und ermitteln damit, welche Zugriffsebene dieser Benutzer für eine bestimmte Ressource erhalten soll.

##<a name="tokens"></a>Speichern des Autorisierungstokens auf dem Client

[AZURE.INCLUDE [mobile-services-windows-store-dotnet-authenticate-app-with-token](../../includes/mobile-services-windows-store-dotnet-authenticate-app-with-token.md)] 

## <a name="next-steps"> </a>Nächste Schritte

Im nächsten Lernprogramm [Dienstweite Autorisierung von Mobile Services-Benutzern](mobile-services-javascript-backend-service-side-authorization.md) werden Sie den von Mobile Services auf Basis eines authentifizierten Benutzers bereitgestellten Benutzer-ID-Wert verwenden, um von Mobile Services zurückgegebene Daten zu filtern. Erfahren Sie mehr über die Verwendung von Mobile Services mit .NET unter [Mobile Services .NET – Erstellen einer konzeptionellen Referenz].

<!-- Anchors. -->
[Registrieren Ihrer App für Authentifizierung und Konfigurieren von Mobile Services]: #register
[Einschränken von Tabellenberechtigungen für authentifizierte Benutzer]: #permissions
[Hinzufügen von Authentifizierung zur App]: #add-authentication
[Speichern von Authentifizierungstoken auf dem Client]: #tokens
[Next Steps]: #next-steps


<!-- URLs. -->
[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253

[Erste Schritte mit Mobile Services]: mobile-services-javascript-backend-windows-store-dotnet-get-started.md
[Get started with data]: ../mobile-services-javascript-backend-windows-store-dotnet-get-started-data.md
[Get started with authentication]: ../mobile-services-javascript-backend-windows-store-dotnet-get-started-users.md
[Get started with push notifications]: ../mobile-services-javascript-backend-windows-store-dotnet-get-started-push.md
[Authorize users with scripts]: ../mobile-services-windows-store-dotnet-authorize-users-in-scripts.md
[JavaScript and HTML]: mobile-services-windows-store-javascript-get-started-users.md

[Azure Management Portal]: https://manage.windowsazure.com/
[Mobile Services .NET – Erstellen einer konzeptionellen Referenz]: mobile-services-windows-dotnet-how-to-use-client-library.md
[Register your Windows Store app package for Microsoft authentication]: ../mobile-services-how-to-register-store-app-package-microsoft-authentication.md

<!--HONumber=54--> 