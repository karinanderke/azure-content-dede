<properties 
	pageTitle="Schützen einer Azure API-App" 
	description="Erfahren Sie, wie Sie eine Azure API-App mithilfe von Visual Studio schützen." 
	services="app-service\api" 
	documentationCenter=".net" 
	authors="tdykstra" 
	manager="wpickett" 
	editor="jimbe"/>

<tags 
	ms.service="app-service-api" 
	ms.workload="web" 
	ms.tgt_pltfrm="dotnet" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="03/24/2015" 
	ms.author="tdykstra"/>

# Schützen einer API-App: Hinzufügen der Authentifizierung über Azure Active Directory oder ein soziales Netzwerk

## Übersicht

Dieses Lernprogramm zeigt, wie Sie eine API-App schützen, sodass nur authentifizierte Benutzer darauf zugreifen können. Das Lernprogramm zeigt außerdem Code, dass Sie in einer ASP.NET-API zum Abrufen von Informationen über den angemeldeten Benutzer verwenden können.

Sie führen die folgenden Schritte aus:

- Aufrufen der API-App, um deren Funktionalität sicherzustellen
- Anwenden von Authentifizierungsregeln auf die API-App
- Erneutes Aufrufen der API-App, um sicherzustellen, dass nicht authentifizierte Anforderungen abgelehnt werden
- Anmelden beim konfigurierten Anbieter
- Erneutes Aufrufen der API-App, um sicherzustellen, dass der authentifizierte Zugriff funktioniert
- Schreiben und Testen von Code, die Ansprüche für den angemeldeten Benutzer abruft.

## Voraussetzungen

In diesem Lernprogramm wird die API-App verwendet, die Sie in [Erstellen einer API-App](app-service-dotnet-create-api-app.md) erstellt und in [Bereitstellen einer API-App](app-service-dotnet-deploy-api-app.md) bereitgestellt haben.

## Verwenden des Browsers zum Aufrufen der API-App 

Die einfachste Möglichkeit zu prüfen, ob Ihre API-App öffentlich zugänglich ist, besteht darin, sie über einen Browser aufzurufen.

1. Wechseln Sie in Ihrem Browser zum [Azure-Vorschauportal].

3. Klicken Sie auf der Startseite auf **Durchsuchen > API-Apps**, und klicken Sie dann auf den Namen der API-App, die Sie schützen möchten.

	![Durchsuchen](./media/app-service-api-dotnet-add-authentication/browse.png)

	![Auswählen einer API](./media/app-service-api-dotnet-add-authentication/select.png)

3. Klicken Sie im Blatt **API-App** auf **URL**, um ein Browserfenster zu öffnen, in dem Ihre API-App aufgerufen wird.

	![Blatt "API-App"](./media/app-service-api-dotnet-add-authentication/chooseapiappurl.png)

2. Hängen Sie in der Adressleiste des Browsers `/api/contacts/get/` an die URL.

	Angenommen, die URL Ihrer API-App lautet folgendermaßen:

    	https://microsoft-apiappeeb5bdsasd744e188be7fa26f239bd4b.azurewebsites.net/

	Dann lautet die vollständige URL so:

    	https://microsoft-apiappeeb5bdsasd744e188be7fa26f239bd4b.azurewebsites.net/api/contacts/get/

	Die verschiedenen Browser verarbeiten API-Aufrufe unterschiedlich. Nachstehend wird ein erfolgreicher Aufruf über einen Chrome-Browser gezeigt.

	![Get-Antwort in Chrome](./media/app-service-api-dotnet-add-authentication/chromeget.png)

2. Speichern Sie die verwendete URL; Sie werden sie später in diesem Lernprogramm erneut verwenden.

## Schützen der API-App

Als Sie Ihre API-App bereitgestellt haben, ist die Bereitstellung als Bestandteil einer Ressourcengruppe erfolgt. Sie können Web-Apps und andere API-Apps zur selben Ressourcengruppe hinzufügen, und jede API-App innerhalb der Ressourcengruppe kann eine von drei Zugriffseinstellungen verwenden: <!--todo: diagram showing different accessibility settings-->

- **Öffentlich (anonym)** – Beliebige Benutzer können die API-App von außerhalb der Ressourcengruppe und ohne Anmeldung aufrufen.
- **Öffentlich (authentifiziert)** – Nur authentifizierte Benutzer können die API-App von außerhalb der Ressourcengruppe aufrufen.
- **Intern** – Nur andere API-Apps oder Web-Apps in derselben Ressourcengruppe dürfen die API-App aufrufen. (Aufrufe von Web-Apps werden als extern betrachtet, selbst wenn sich die Web-Apps in derselben Ressourcengruppe befinden.)

Als Visual Studio die Ressourcengruppe für Sie erstellt hat, wurde auch ein *Gateway* erstellt. Ein Gateway ist eine spezielle Web-App, die alle Anforderungen verarbeitet, die an API-Apps in der Ressourcengruppe gerichtet sind.

Wenn Sie im [Azure-Vorschauportal] zum Blatt für die Ressourcengruppe wechseln, können Sie Ihre API-App und das Gateway im Diagramm anzeigen.

![Diagramm der Ressourcengruppe](./media/app-service-api-dotnet-add-authentication/rgdiagram.png)

### Konfigurieren der API-App zum Verwenden der Authentifizierung

Um Ihre API-App so zu konfigurieren, dass nur authentifizierte Anforderungen akzeptiert werden, legen Sie den Zugriff **Öffentlich (authentifiziert)** fest. Anschließend konfigurieren Sie das Gateway für eine Authentifizierung über einen Anbieter wie Azure Active Directory, Google oder Facebook.

1. Wechseln Sie zurück zum Blatt **API-App** für die API-App, die Sie schützen möchten.

2. Klicken Sie im Blatt **API-App** auf **Einstellungen** und anschließend auf **Anwendungseinstellungen**.

	![Klicken Sie auf "Einstellungen"](./media/app-service-api-dotnet-add-authentication/clicksettings.png)

	![Klicken Sie auf "Anwendungseinstellungen"](./media/app-service-api-dotnet-add-authentication/clickbasicsettings.png)

3. Ändern Sie im Blatt **Anwendungseinstellungen** die **Zugriffsebene** in **Öffentlich (authentifiziert)**, und klicken Sie dann auf **Speichern**.

	![Klicken Sie auf "Grundlegende Einstellungen"](./media/app-service-api-dotnet-add-authentication/setpublicauth.png)

	Ihre API-App ist jetzt vor einem nicht authentifizierten Zugriff geschützt. Als Nächstes müssen Sie das Gateway konfigurieren, um den Authentifizierungsanbieter anzugeben.

### <a id="gateway"></a>Konfigurieren des Gateways zum Verwenden eines Authentifizierungsanbieters

4. Wechseln Sie zurück zum Blatt **API-App**, und klicken Sie auf den Link zum Gateway.

	![Klicken Sie auf "Gateway"](./media/app-service-api-dotnet-add-authentication/gateway.png)

7. Klicken Sie im Blatt **Gateway** auf **Einstellungen** und anschließend auf **Identität**.

	![Klicken Sie auf "Einstellungen"](./media/app-service-api-dotnet-add-authentication/clicksettingsingateway.png)

	![Klicken Sie auf "Identität"](./media/app-service-api-dotnet-add-authentication/clickidentity.png)

	Vom Blatt **Identität** aus können Sie zu unterschiedlichen Blättern zum Konfigurieren der Authentifizierung mithilfe von Azure Active Directory und verschiedenen anderen Anbietern navigieren.

	![Blatt "Identität"](./media/app-service-api-dotnet-add-authentication/identityblade.png)
  
3. Wählen Sie den gewünschten Identitätsanbieter, und führen Sie die Schritte zum Konfigurieren Ihrer API-App für diesen Anbieter aus, die im entsprechenden Artikel aufgeführt sind. Diese Artikel wurden für mobile Apps geschrieben, die Vorgehensweise für API-Apps ist jedoch identisch. Für einige der Verfahren ist das [Azure-Portal] erforderlich.

 - [Microsoft-Konto](../app-service-mobile/app-service-mobile-how-to-configure-microsoft-authentication-preview.md)
 - [Facebook-Anmeldung](../app-service-mobile/app-service-mobile-how-to-configure-facebook-authentication-preview.md)
 - [Twitter-Anmeldung](../app-service-mobile/app-service-mobile-how-to-configure-twitter-authentication-preview.md)
 - [Google-Anmeldung](../app-service-mobile/app-service-mobile-how-to-configure-google-authentication-preview.md)
 - [Azure Active Directory](../app-service-mobile/app-service-mobile-how-to-configure-active-directory-authentication-preview.md)

Die nachfolgenden Screenshots zeigen beispielhaft, welche Seiten im [Azure-Portal] und welche Blätter im [Azure-Vorschauportal] angezeigt werden sollten, nachdem Sie die Azure Active Directory-Authentifizierung eingerichtet haben.

Im Azure-Vorschauportal zeigt das Blatt **Azure Active Directory** eine **Client-ID** aus der Anwendung, die Sie auf der Registerkarte "Azure Active Directory" im Azure-Portal erstellt haben. Die Einstellung **Zulässige Mandanten** enthält Ihren Azure Active Directory-Mandanten (z. B. "contoso.onmicrosoft.com").

![Blatt "Azure Active Directory"](./media/app-service-api-dotnet-add-authentication/tdinaadblade.png)

Im Azure-Portal enthält die Registerkarte **Konfiguration** der Anwendung, die Sie auf der Registerkarte **Azure Active Directory** erstellt haben, die Einstellungen **Anmelde-URL**, **App-ID-URI** und **Antwort-URL** aus dem Blatt **Azure Active Directory** im Azure-Vorschauportal.

![](./media/app-service-api-dotnet-add-authentication/oldportal1.png)

![](./media/app-service-api-dotnet-add-authentication/oldportal2.png)

![](./media/app-service-api-dotnet-add-authentication/oldportal3.png)

![](./media/app-service-api-dotnet-add-authentication/oldportal4.png)

(Die Antwort-URL im Screenshot zeigt zweimal dieselbe URL, einmal mit `http:` und einmal mit `https:`.)

## Sicherstellen, dass die Authentifizierung funktioniert

**Hinweis:** Wenn Sie beim Ausführen der folgenden Schritte Probleme bei der Anmeldung haben, öffnen Sie ein Browserfenster im privaten oder im Inkognito-Modus.
 
1. Öffnen Sie ein Browserfenster, und geben Sie in der Adressleiste die URL ein, mit der (wie zuvor) die `Get`-Methode Ihrer API-App aufgerufen wird.

	Dieses Mal führt der Zugriffsversuch auf die API-App zu einer Fehlermeldung.

	![Get-Fehlerantwort in Chrome](./media/app-service-api-dotnet-add-authentication/chromegetfail.png)

2. Wechseln Sie im Browser zur Anmelde-URL. Die URL folgt diesem Muster:

    	http://[gatewayurl]/login/[providername]

	Sie können die Gateway-URL aus dem Blatt **Gateway** im [Azure-Vorschauportal] abrufen. (Sie gelangen zum Blatt **Gateway**, indem Sie im Blatt **Ressourcengruppe** auf das Gateway im angezeigten Diagramm klicken.)

	![Gateway-URL](./media/app-service-api-dotnet-add-authentication/gatewayurl.png)

	Die [Providername] muss einer der folgenden Werte sein:
	
	* "Microsoftaccount"
	* "Facebook"
	* "Twitter"
	* "Google"
	* "Aad"

	Nachfolgend wird eine beispielhafte Anmelde-URL für Azure Active Directory gezeigt:

		https://dropboxrgaeb4ae60b7cb4f3d966dfa43.azurewebsites.net/login/aad/

	Beachten Sie, dass im Gegensatz zur vorherigen URL der Name der API-App nicht enthalten ist: das Gateway authentifiziert Sie, nicht die API-App. Das Gateway übernimmt die Authentifizierung für alle API-Apps in der Ressourcengruppe.

3. Geben Sie Ihre Anmeldeinformationen ein, wenn der Browser eine Anmeldeseite anzeigt.
 
	Wenn Sie die Azure Active Directory-Anmeldung konfiguriert haben, verwenden Sie einen der auf der Registerkarte **Benutzer** aufgeführten Benutzer für die Anwendung, die Sie auf der Registerkarte "Azure Active Directory" im [Azure-Portal] erstellt haben, z. B. admin@contoso.onmicrosoft.com.

	![AAD-Benutzer](./media/app-service-api-dotnet-add-authentication/aadusers.png)

	![Anmeldeseite](./media/app-service-api-dotnet-add-authentication/ffsignin.png)

4. Wenn die Meldung "Anmeldung abgeschlossen" angezeigt wird, geben Sie erneut die URL zur Get-Methode Ihrer API-App ein.

	Nun, da Sie authentifiziert wurden, ist der Aufruf erfolgreich. Das Gateway erkennt, dass Sie ein authentifizierter Benutzer sind, und übergibt Ihre Anforderung an die API-App.

	![Anmeldung abgeschlossen](./media/app-service-api-dotnet-add-authentication/logincomplete.png)

	![Get-Antwort in Chrome](./media/app-service-api-dotnet-add-authentication/chromeget.png)

## Verwenden von Postman zum Senden einer Post-Anforderung

Wenn Sie sich am Gateway anmelden, gibt das Gateway ein Authentifizierungstoken zurück. Dieses Token muss in alle Anforderungen von externen Quellen eingeschlossen werden, die über das Gateway erfolgen. Wenn Sie über einen Browser auf eine API zugreifen, wird das Token typischerweise in einem Cookie gespeichert und zusammen mit allen nachfolgenden Aufrufen an die API gesendet.

Damit Sie sehen, was im Hintergrund geschieht, verwenden Sie in diesem Abschnitt des Lernprogramms ein Browsertool zum Erstellen und Senden einer Post-Anforderung. Anschließend rufen Sie das Autorisierungstoken aus dem Cookie ab und schließen es in einen HTTP-Header ein. Dieser Abschnitt ist optional: Sie haben im vorherigen Abschnitt bereits überprüft, dass die API-App ausschließlich authentifizierte Zugriffe akzeptiert.

Diese Anweisungen zeigen, wie Sie das Postman-Tool im Chrome-Browser verwenden, aber Sie können dieselbe Aufgabe ebenso mit einem beliebigen REST-Clienttool und den Entwicklertools für den Browser durchführen.

1. Durchlaufen Sie in einem Chrome-Browserfenster die im vorherigen Abschnitt gezeigten Schritte zur Authentifizierung, und öffnen Sie dann die Entwicklertools (F12).

	![Wechseln Sie zur Registerkarte "Ressourcen"](./media/app-service-api-dotnet-add-authentication/resources.png)

3. Suchen Sie auf der Registerkarte **Ressourcen** der Chrome-Entwicklertools nach den Cookies für Ihr Gateway, und klicken Sie dreimal auf den für das Cookie **x-zumo-auth**, um alle Informationen auszuwählen.

	**Hinweis:** Stellen Sie sicher, dass Sie den gesamten Cookiewert abrufen. Bei einem Doppelklick wird möglicherweise nur der erste Teil ausgewählt.

5. Klicken Sie mit der rechten Maustaste auf den **Value** für das Cookie **x-zumo-auth**, und klicken Sie dann auf **Kopieren**.

	![Kopieren des Authentifizierungstokens](./media/app-service-api-dotnet-add-authentication/copyzumotoken.png)

4. Sofern noch nicht geschehen, installieren Sie die Postman-Erweiterung in Ihrem Chrome-Browser.

6. Öffnen Sie die Postman-Erweiterung.

7. Geben Sie im Feld für die Anforderungs-URL die URL zur Get-Methode Ihrer API-App ein, die Sie bereits zuvor verwendet haben. Lassen Sie jedoch den Wert `get/` am Ende weg.
 
		http://[apiappurl]/api/contacts
    
8. Klicken Sie auf **Headers**, und fügen Sie einen *x-zumo-auth*-Header hinzu. Fügen Sie den Tokenwert aus der Zwischenablage in das Feld **Value** ein.

9. Fügen Sie einen *Content-Type*-Header mit dem Wert *application/json* hinzu.

10. Klicken Sie auf **form-data**, und fügen Sie einen *contact*-Schlüssel mit dem folgenden Wert hinzu:

		{   "Id": 0,   "Name": "Li Yan",   "EmailAddress": "yan@contoso.com" }

11. Klicken Sie auf "Send".

	Die API-App gibt eine *201 Erstellt*-Antwort zurück.

	![Header und Text einfügen](./media/app-service-api-dotnet-add-authentication/addcontact.png)

12. Um sicherzustellen, dass diese Anforderung ohne das Authentifizierungstoken nicht funktionieren würde, löschen Sie den Authentifizierungsheader, und klicken Sie erneut auf "Send".

	Sie erhalten eine *403 Verboten*-Antwort.

	![403 Verboten-Antwort](./media/app-service-api-dotnet-add-authentication/403forbidden.png)

## Abrufen von Informationen über den angemeldeten Benutzer

In diesem Abschnitt ändern Sie den Code in der Anwendung noch-API, abgerufen und die Namen und e-Mail-Adresse des angemeldeten Benutzers zurückgibt.

1. Öffnen Sie in Visual Studio das API-app-Projekt, die Bereitstellung in [Bereitstellung einer API-Anwendung](app-service-dotnet-deploy-api-app.md) und haben für dieses Lernprogramm aufgerufen wurde.

3. Öffnen der Datei apiapp.json, und fügen Sie eine Zeile, die die API-Anwendung gibt Azure Active Directory-Authentifizierung verwendet.

		"authentication": [{"type": "aad"}]

	Die letzte apiapp.json-Datei wird im folgende Beispiel ähneln:

		{
		    "$schema": "http://json-schema.org/schemas/2014-11-01/apiapp.json#",
		    "id": "ContactsList",
		    "namespace": "microsoft.com",
		    "gateway": "2015-01-14",
		    "version": "1.0.0",
		    "title": "ContactsList",
		    "summary": "",
		    "author": "",
		    "endpoints": {
		        "apiDefinition": "/swagger/docs/v1",
		        "status": null
		    },
		    "authentication": [{"type": "aad"}]
		}

	In diesem Lernprogramm verwendet Azure Active Directory als Beispiel. Für andere Anbieter verwenden Sie den entsprechenden Bezeichner. Hier sind die gültigen Anbieter:

	* "Aad"
	* "Microsoftaccount"
	* "Google"
	* "Twitter"
	* "Facebook". 

2. In der *ContactsController.cs* Datei, ersetzen Sie den Code in der `Get` -Methode durch den folgenden Code.

		var runtime = Runtime.FromAppSettings(Request);
		var user = runtime.CurrentUser;
		TokenResult token = await user.GetRawTokenAsync("aad");
		var name = (string)token.Claims["name"];
		var email = (string)token.Claims["http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn"];
		return new Contact[]
		{
		    new Contact { Id = 1, EmailAddress = email, Name = name }
		};

	Anstatt die drei Beispielkontakte zurückgesetzt Kontaktinformationen für den angemeldeten Benutzer.

	Im Beispielcode wird die Azure Active Directory verwendet. Für andere Anbieter verwenden Sie den entsprechenden Namen und Ansprüche Tokenbezeichner, wie im vorherigen Schritt gezeigt.

	Weitere Informationen zu Azure Active Directory-Ansprüche, die verfügbar sind, finden Sie unter [Token unterstützt und Anspruchstypen](https://msdn.microsoft.com/library/dn195587.aspx).

3. Fügen Sie eine using-Anweisung für `Microsoft.Azure.AppService.ApiApps.Service`.

		using Microsoft.Azure.AppService.ApiApps.Service;

3. Bereitstellen Sie das Projekt erneut.

	Visual Studio speichert die Einstellungen bei der Bereitstellung des Projekts beim folgenden der [Bereitstellen](app-service-dotnet-deploy-api-app.md) Lernprogramm. Mit der rechten Maustaste in des Projekts, klicken Sie auf **Veröffentlichen**, und klicken Sie dann auf **Veröffentlichen** in den **Web veröffentlichen** Dialogfeld.

6. Führen Sie die Prozedur, die Sie zuvor ausgeführt haben, um eine Get-Anforderung an die geschützte API-Anwendung zu senden.

	Die Response-Nachricht zeigt den Namen und die ID der Identität, die Sie zur Anmeldung verwendet.

	![Response-Nachricht mit der angemeldete Benutzer](./media/app-service-api-dotnet-add-authentication/chromegetuserinfo.png)

## Nächste Schritte

Sie haben erfahren, wie Sie eine Azure API-App schützen, indem Sie sie für eine Authentifizierung über Azure Active Directory oder ein soziales Netzwerk konfigurieren. Weitere Informationen finden Sie unter [Was sind API-Apps?](app-service-api-apps-why-best-platform.md).

[Azure-Portal]: https://manage.windowsazure.com/
[Azure-Vorschauportal]: https://portal.azure.com/


 

<!---HONumber=GIT-SubDir_Tue_AM_dede-->