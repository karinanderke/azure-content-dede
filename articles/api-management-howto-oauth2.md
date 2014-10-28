<properties pageTitle="How to authorize developer accounts using OAuth 2.0 in Azure API Management" metaKeywords="" description="Learn how to authorize users using OAuth 2.0 in API Management." metaCanonical="" services="api-management" documentationCenter="API Management" title="How to authorize developer accounts using OAuth 2.0 in Azure API Management" authors="sdanie" solutions="" manager="" editor="" />

<tags ms.service="api-management" ms.workload="mobile" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="01/01/1900" ms.author="sdanie"></tags>

# So autorisieren Sie Entwicklerkonten mithilfe von OAuth 2.0 in Azure API Management

API Management bietet Benutzern die Möglichkeit, mithilfe der OAuth 2.0-Autorisierung auf das Entwicklerportal zuzugreifen. Diese Anleitung beschreibt, wie Sie eine Instanz des API-Verwaltungsdiensts zur Verwendung der OAuth 2.0-Autorisierung konfigurieren.

> Weitere Informationen zu OAuth 2.0 finden Sie in der Spezifikation unter [http://oauth.net/2/][http://oauth.net/2/].

## In diesem Thema

-   [Voraussetzungen][Voraussetzungen]
-   [Konfigurieren eines OAuth 2.0-Autorisierungsservers in API Management][Konfigurieren eines OAuth 2.0-Autorisierungsservers in API Management]
-   [Konfigurieren einer API zum Verwenden der OAuth 2.0-Benutzerauthentifizierung][Konfigurieren einer API zum Verwenden der OAuth 2.0-Benutzerauthentifizierung]
-   [Testen der OAuth 2.0-Benutzerauthentifizierung im Entwicklerportal][Testen der OAuth 2.0-Benutzerauthentifizierung im Entwicklerportal]

## <a name="prerequisites"> </a>Voraussetzungen

Diese Anleitung beschreibt, wie Sie eine Instanz des API-Verwaltungsdiensts zur Verwendung der OAuth 2.0-Autorisierung für Entwicklerkonten konfigurieren. Es wird jedoch nicht behandelt, wie Sie einen OAuth 2.0-Anbieter konfigurieren. Die Konfiguration ist je nach OAuth 2.0-Anbieter unterschiedlich, die Schritte sind jedoch ähnlich, und die beim Konfigurieren von OAuth 2.0 in Ihrer Instanz des API-Verwaltungsdiensts erforderlichen Informationen sind gleich. In diesem Thema werden Beispiele zur Verwendung von Azure Active Directory als OAuth 2.0-Anbieter gezeigt.

> Weitere Informationen zum Konfigurieren von OAuth 2.0 mithilfe von Azure Active Directory finden Sie im Beispiel [WebApp-GraphAPI-DotNet][WebApp-GraphAPI-DotNet].

## <a name="step1"> </a>Konfigurieren eines OAuth 2.0-Autorisierungsservers in API Management

Klicken Sie zunächst in der **Verwaltungskonsole** im Azure-Portal auf Ihren API-Verwaltungsdienst. Daraufhin gelangen Sie zum Verwaltungsportal für die API-Verwaltung.

![API-Verwaltungskonsole][api-management-management-console]

> Falls Sie noch keine API-Verwaltungs-Dienstinstanz erstellt haben, finden Sie weitere Informationen im Abschnitt [Erstellen einer API-Verwaltungsinstanz][Erstellen einer API-Verwaltungsinstanz] im Lernprogramm [Erste Schritte mit der Azure API-Verwaltung][Erste Schritte mit der Azure API-Verwaltung].

Klicken Sie auf **Sicherheit** im Menü **API-Verwaltung** auf der linken Seite, klicken Sie auf **OAuth 2.0** und dann auf **Autorisierungsserver hinzufügen**.

![OAuth 2.0][api-management-oauth2]

Nachdem Sie auf **Autorisierungsserver hinzufügen** geklickt haben, wird das Formular für neue Autorisierungsserver angezeigt.

![Neuer Server][api-management-oauth2-server-1]

Geben Sie in die Felder **Name** und **Beschreibung** einen Namen bzw. eine optionale Beschreibung ein.

> Anhand dieser Felder wird der OAuth 2.0-Autorisierungsserver innerhalb der aktuellen Instanz des API-Verwaltungsdiensts identifiziert. Die Werte der Felder stammen nicht vom OAuth 2.0-Server.

Geben Sie die **URL der Client-Registrierungsseite** ein. Auf dieser Seite können Benutzer ihre Konten erstellen und verwalten. Die Seite variiert je nach OAuth 2.0-Anbieter.

Der nächste Abschnitt des Formulars enthält die Einstellungen **Autorisierungscode-Berechtigungstypen**, **Autorisierungsendpunkt-URL** und **Autorisierungsanforderungsmethode**.

![Neuer Server][api-management-oauth2-server-2]

Geben Sie die **Autorisierungscode-Berechtigungstypen** an, indem Sie die gewünschten Typen aktivieren. **Autorisierungscode** wird standardmäßig festgelegt.

Geben Sie die **Autorisierungsendpunkt-URL** ein. Für Azure Active Directory lautet diese URL ähnlich wie die folgende, wobei `<client_id>` durch die Client-ID ersetzt wird, die Ihre Anwendung gegenüber dem OAuth 2.0-Server authentifiziert.

    https://login.windows.net/<client_id>/oauth2/authorize

Die **Autorisierungsanforderungsmethode** legt fest, wie die Autorisierungsanforderung an den OAuth 2.0-Server gesendet wird. Standardmäßig ist **GET** ausgewählt.

Im nächsten Abschnitt werden **Token-Endpunkt-URL**, **Client-Authentifizierungsmethoden**, **Sendemethode für Zugriffstoken** und **Standardmäßiger Geltungsbereich** angegeben.

![Neuer Server][api-management-oauth2-server-3]

Für einen Azure Active Directory OAuth 2.0-Server hat die **Token-Endpunkt-URL** das folgende Format, wobei `<APPID>` das Format `yourapp.onmicrosoft.com` hat.

    https://login.windows.net/<APPID>/oauth2/token

Die Standardeinstellung für **Client-Authentifizierungsmethoden** lautet **Basis**, und die **Sendemethode für Zugriffstoken** ist **Autorisierungsheader**. Diese Werte werden zusammen mit **Standardmäßiger Geltungsbereich** in diesem Abschnitt des Formulars festgelegt.

Der Abschnitt **Client-Berechtigungen** enthält **Client-ID** und **Geheimer Clientschlüssel**, die bei der Erstellung und Konfiguration Ihres OAuth 2.0-Servers festgelegt werden. Sobald **Client-ID** und **Geheimer Clientschlüssel** angegeben wurden, wird die **redirect\_uri** für den **Autorisierungscode** erzeugt. Die URI wird zum Konfigurieren der Antwort-URL in Ihrer OAuth 2.0-Serverkonfiguration verwendet.

![Neuer Server][api-management-oauth2-server-4]

Falls für **Autorisierungscode-Berechtigungstypen** die Einstellung **Ressourcenbesitzer-Kennwort** festgelegt ist, werden diese Berechtigungen im Abschnitt **Ressourcenbesitzer-Kennwortberechtigungen** festgelegt. Andernfalls lassen Sie den Abschnitt leer.

![Neuer Server][api-management-oauth2-server-5]

Nachdem Sie das Formular ausgefüllt haben, klicken Sie auf **Speichern**, um die OAuth 2.0-Autorisierungsserverkonfiguration für die API-Verwaltung zu speichern. Nach dem Speichern der Serverkonfiguration können Sie APIs wie im nächsten Abschnitt beschrieben zur Nutzung konfigurieren.

## <a name="step2"> </a>Konfigurieren einer API zum Verwenden der OAuth 2.0-Benutzerauthentifizierung

Klicken Sie auf **APIs** im Menü **API-Verwaltung** auf der linken Seite, klicken Sie auf die gewünschte API und danach auf **Sicherheit**. Aktivieren Sie anschließend das Kontrollkästchen für **OAuth 2.0**.

![Benutzerautorisierung][api-management-user-authorization]

Wählen Sie in der Dropdownliste den gewünschten **Autorisierungsserver**, und klicken Sie dann auf **Speichern**.

![Benutzerautorisierung][api-management-user-authorization-save]

## <a name="step3"> </a>Testen der OAuth 2.0-Benutzerauthentifizierung im Entwicklerportal

Nachdem Sie Ihren OAuth 2.0-Autorisierungsserver und Ihre API zu dessen Nutzung konfiguriert haben, können Sie ihn testen, indem Sie zum Entwicklerportal wechseln und eine API aufrufen. Klicken Sie im Menü oben rechts auf **Entwicklerportal**.

![Entwicklerportal][api-management-developer-portal-menu]

Klicken Sie auf **APIs** im Hauptmenü und wählen Sie **Echo API** aus.

![Echo API][api-management-apis-echo-api]

> Falls nur eine API konfiguriert oder für Ihr Konto sichtbar ist, können Sie auf APIs klicken, um direkt zu den Operationen für diese API zu gelangen.

Wählen Sie die Operation **GET Resource** aus, klicken Sie auf **Konsole öffnen**, und wählen Sie dann aus der Dropdownliste **Autorisierungscode** aus.

![Konsole öffnen][api-management-open-console]

Wenn der **Autorisierungscode** ausgewählt wurde, wird ein Popup-Fenster mit dem Anmeldeformular des OAuth 2.0-Anbieters angezeigt. In diesem Beispiel wird das Anmeldeformular von Azure Active Directory bereitgestellt.

> Falls Sie Popup-Fenster deaktiviert haben, werden Sie dazu aufgefordert, sie im Browser zu aktivieren. Wählen Sie danach erneut **Autorisierungscode** aus. Daraufhin wird das Anmeldeformular angezeigt.

![Anmelden][api-management-oauth2-signin]

Nachdem Sie sich angemeldet haben, werden die **Anforderungsheader** mit einem `Authorization : Bearer`-Header ausgefüllt, der die Anforderung autorisiert.

![Token des Anforderungsheaders][api-management-request-header-token]

Nun können Sie die gewünschten Werte für die restlichen Parameter konfigurieren und die Anforderung absenden.

  [http://oauth.net/2/]: http://oauth.net/2/
  [Voraussetzungen]: #prerequisites
  [Konfigurieren eines OAuth 2.0-Autorisierungsservers in API Management]: #step1
  [Konfigurieren einer API zum Verwenden der OAuth 2.0-Benutzerauthentifizierung]: #step2
  [Testen der OAuth 2.0-Benutzerauthentifizierung im Entwicklerportal]: #step3
  [WebApp-GraphAPI-DotNet]: https://github.com/AzureADSamples/WebApp-GraphAPI-DotNet
  [Erstellen einer API-Verwaltungsinstanz]: ../api-management-get-started/#create-service-instance
  [Erste Schritte mit der Azure API-Verwaltung]: ../api-management-get-started
  [api-management-management-console]: ./media/api-management-howto-oauth2/api-management-management-console.png
  [api-management-oauth2]: ./media/api-management-howto-oauth2/api-management-oauth2.png
  [api-management-user-authorization]: ./media/api-management-howto-oauth2/api-management-user-authorization.png
  [api-management-user-authorization-save]: ./media/api-management-howto-oauth2/api-management-user-authorization-save.png
  [api-management-oauth2-signin]: ./media/api-management-howto-oauth2/api-management-oauth2-signin.png
  [api-management-request-header-token]: ./media/api-management-howto-oauth2/api-management-request-header-token.png
  [api-management-developer-portal-menu]: ./media/api-management-howto-oauth2/api-management-developer-portal-menu.png
  [api-management-open-console]: ./media/api-management-howto-oauth2/api-management-open-console.png
  [api-management-oauth2-server-1]: ./media/api-management-howto-oauth2/api-management-oauth2-server-1.png
  [api-management-oauth2-server-2]: ./media/api-management-howto-oauth2/api-management-oauth2-server-2.png
  [api-management-oauth2-server-3]: ./media/api-management-howto-oauth2/api-management-oauth2-server-3.png
  [api-management-oauth2-server-4]: ./media/api-management-howto-oauth2/api-management-oauth2-server-4.png
  [api-management-oauth2-server-5]: ./media/api-management-howto-oauth2/api-management-oauth2-server-5.png
  [api-management-apis-echo-api]: ./media/api-management-howto-oauth2/api-management-apis-echo-api.png