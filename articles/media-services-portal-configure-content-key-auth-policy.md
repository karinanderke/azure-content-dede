﻿<properties 
	pageTitle="Konfigurieren einer Autorisierungsrichtlinie für Inhaltsschlüssel im Portal" 
	description="Erfahren Sie, wie Sie eine Autorisierungsrichtlinie für einen Inhaltsschlüssel konfigurieren." 
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
	ms.date="02/20/2015" 
	ms.author="juliako"/>



# Konfigurieren einer Autorisierungsrichtlinie für Inhaltsschlüssel 
[AZURE.INCLUDE [media-services-selector-content-key-auth-policy](../includes/media-services-selector-content-key-auth-policy.md)]

Dieser Artikel gehört zur Reihe [Media Services: Video-on-Demand-Workflow](../media-services-video-on-demand-workflow) und [Media Services: Livestreaming-Workflow](../media-services-live-streaming-workflow) . 


## Übersicht

Mit Microsoft Azure Media Services können Sie Inhalte verschlüsselt übermitteln, und zwar mit AES (Advanced Encryption Standard unter Verwendung eines 128-Bit-Verschlüsselungsschlüssels) und PlayReady-DRM. Media Services bietet einen **Schlüssel-\Lizenzübermittlungsdienst**, von dem die Clients einen Schlüssel oder eine Lizenz zur Wiedergabe verschlüsselten Inhalts abrufen können. 

In diesem Thema wird veranschaulicht, wie Sie das **Azure-Verwaltungsportal** zur Konfiguration der Autorisierungsrichtlinie für Inhaltsschlüssel verwenden. Der Schlüssel kann später verwendet werden, um Inhalte dynamisch zu verschlüsseln. Zur Zeit können Sie die folgenden Streamingformate verschlüsseln: HLS, MPEG-DASH und Smooth Streaming. Das HDS-Streamingformat oder progressive Downloads können nicht verschlüsselt werden.
 
Wenn ein Player einen Datenstrom anfordert, der für die dynamische Verschlüsselung konfiguriert ist, verwendet Media Services den konfigurierten Schlüssel, um Inhalte dynamisch mit der AES- oder PlayReady-Verschlüsselung zu verschlüsseln. Um den Stream zu entschlüsseln, fordert der Player den Schlüssel vom Schlüsselübermittlungsdienst an. Um zu entscheiden, ob der Benutzer berechtigt ist, den Schlüssel zu erhalten, wertet der Dienst die Autorisierungsrichtlinien aus, die Sie für den Schlüssel angegeben haben.


Wenn Sie mehrere Inhaltsschlüssel verwenden oder eine andere **Schlüssel-/Lizenzübermittlungsdienst**-URL als der des Schlüsselübermittlungsdiensts von Media Services angeben möchten, verwenden Sie das Media Services .NET SDK oder REST-APIs.

[Konfigurieren einer Autorisierungsrichtlinie für Inhaltsschlüssel mit dem .NET-SDK](../media-services-dotnet-configure-content-key-auth-policy/)

[Konfigurieren einer Autorisierungsrichtlinie für Inhaltsschlüssel mit der REST-API](../media-services-rest-configure-content-key-auth-policy/)

### Folgende Überlegungen sollten berücksichtigt werden:

- Zur Verwendung der dynamischen Paketerstellung und Verschlüsselung müssen Sie über mindestens eine Skalierungseinheit (auch als Streamingeinheit bezeichnet) verfügen. Weitere Informationen finden Sie unter [Skalieren eines Mediendiensts](../media-services-manage-origins#scale_streaming_endpoints). 
- Ihr Medienobjekt muss einen Satz von MP4-Dateien bzw. Smooth Streaming-Dateien mit adaptiver Bitrate enthalten. Weitere Informationen finden Sie unter [Codieren eines Medienobjekts](../media-services-encode-asset/).  
- ContentKeyAuthorizationPolicy und die zugehörigen Objekte (Richtlinienoptionen und Einschränkungen) werden vom Schlüsselübermittlungsdienst für 15 Minuten zwischengespeichert.  Wenn Sie ContentKeyAuthorizationPolicy erstellen und angeben, dass eine "Token"-Einschränkung verwendet werden soll, diese anschließend testen und dann die Richtlinie auf eine "Open"-Einschränkung aktualisieren, dauert es ungefähr 15 Minuten, bis die Richtlinie zur "Open"-Version der Richtlinie wechselt.


## Vorgehensweise: Konfigurieren der Schlüsselautorisierungsrichtlinie

Um die Schlüsselautorisierungsrichtlinie zu konfigurieren, wählen Sie die Seite **INHALTSSCHUTZ** aus.
	
Media Services unterstützt mehrere Möglichkeiten zur Authentifizierung von Benutzern, die Schlüssel anfordern. Autorisierungsrichtlinien für Inhaltsschlüssel können die Autorisierungseinschränkung **Open**, **Token** oder **IP** aufweisen (**IP** kann mit REST oder dem .NET SDK konfiguriert werden). 

### Open-Einschränkung

Bei Verwendung einer **Open**-Einschränkung übermittelt das System den Schlüssel an jede Person, die einen Schlüssel anfordert. Diese Einschränkung kann zu Testzwecken nützlich sein.

![OpenPolicy][open_policy]

### Token-Einschränkung

Zum Auswählen der durch "Token" eingeschränkten Richtlinie drücken Sie die Schaltfläche **TOKEN**.

Eine durch **Token** eingeschränkte Richtlinie gilt nur zusammen mit einem Token, das von einem **Secure Token Service** (STS) ausgestellt wurde. Media Services unterstützt Token im **Simple Web Tokens** ([SWT](https://msdn.microsoft.com/library/gg185950.aspx#BKMK_2))-Format und **JSON Web Token** (JWT)-Format. Informationen finden Sie unter [JWT-Tokenauthentifizierung](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/).

**Secure Token Services** werden von Media Services nicht bereitgestellt. Sie können einen benutzerdefinierten STS erstellen oder Microsoft Azure ACS zum Ausstellen von Token nutzen. Der STS muss für die Erstellung eines mit dem angegebenen Schlüssel signierten Tokens und die Ausstellungsansprüche konfiguriert sein, die Sie in der Konfiguration der Tokeneinschränkung angegeben haben. Der Schlüsselübermittlungsdienst von Media Services gibt den Verschlüsselungsschlüssel an den Client zurück, wenn das Token gültig ist und die Ansprüche im Token mit den für den Inhaltsschlüssel konfigurierten Ansprüchen übereinstimmen. Weitere Informationen finden Sie unter [Ausstellen von Token mithilfe von Azure ACS](http://mingfeiy.com/acs-with-key-services).

Beim Konfigurieren der durch **TOKEN** eingeschränkten Richtlinie müssen Sie Werte für **VerificationKey**, **Issuer** und **Audience** festlegen. Der primäre Verifizierungsschlüssel enthält den Schlüssel, mit dem das Token signiert wurde, und der Aussteller entspricht dem STS (Secure Token Service), von dem das Token ausgestellt wurde. Audience (manchmal auch Scope) beschreibt den Verwendungszweck des Tokens oder die Ressource, auf die durch das Token Zugriff gewährt wird. Der Schlüsselübermittlungsdienst von Media Services überprüft, ob die Werte im Token mit den Werten in der Vorlage übereinstimmen.  

### PlayReady

Wenn Sie Inhalte mit **PlayReady** schützen, müssen Sie in Ihrer Autorisierungsrichtlinie u. a. eine XML-Zeichenfolge zur Definition der PlayReady-Lizenzvorlage angeben. Standardmäßig ist die folgende Richtlinie festgelegt:
		
	<PlayReadyLicenseResponseTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1">
	  <LicenseTemplates>
	    <PlayReadyLicenseTemplate><AllowTestDevices>true</AllowTestDevices>
	      <ContentKey i:type="ContentEncryptionKeyFromHeader" />
	      <LicenseType>Nonpersistent</LicenseType>
	      <PlayRight>
	        <AllowPassingVideoContentToUnknownOutput>Allowed</AllowPassingVideoContentToUnknownOutput>
	      </PlayRight>
	    </PlayReadyLicenseTemplate>
	  </LicenseTemplates>
	</PlayReadyLicenseResponseTemplate>

Klicken Sie auf die Schaltfläche zum **Importieren der Richtlinien-XML**, und geben Sie eine andere XML an, die dem [hier](https://msdn.microsoft.com/library/azure/dn783459.aspx) definierten XML-Schema entspricht.

## Nächste Schritte
Nachdem Sie die Autorisierungsrichtlinie für Inhaltsschlüssel jetzt konfiguriert haben, fahren Sie mit dem Thema [Gewusst wie: Aktivieren der Verschlüsselung im Azure-Verwaltungsportal](../media-services-manage-content#encrypt/) .


[open_policy]: ./media/media-services-key-authorization-policy/media-services-protect-content-with-open-restriction.png
[token_policy]: ./media/media-services-key-authorization-policy/media-services-protect-content-with-token-restriction.png


<!--HONumber=47-->