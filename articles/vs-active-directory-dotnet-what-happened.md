﻿<properties title="Getting Started with Active Directory Authentication - What Happened" pageTitle="Erste Schritte mit Active Directory-Authentifizierung - Was ist passiert?" metaKeywords="Azure, Getting Started, Active Directory" description="" services="active-directory" documentationCenter="" authors="ghogen, kempb" />
  
<tags ms.service="active-directory" ms.workload="web" ms.tgt_pltfrm="vs-what-happened" ms.devlang="na" ms.topic="article" ms.date="10/8/2014" ms.author="ghogen, kempb" />

> [AZURE.SELECTOR]
> - [Erste Schritte](/documentation/articles/vs-active-directory-dotnet-getting-started/)
> - [Was ist passiert?](/documentation/articles/vs-active-directory-dotnet-what-happened/)

###<span id="whathappened">Was ist mit meinem Projekt passiert?</span>
 
Verweise wurden hinzugefügt.

#####NuGet-Paketverweise

- `Microsoft.IdentityModel.Protocol.Extensions`
- `Microsoft.Owin`
- `Microsoft.Owin.Host.SystemWeb`
- `Microsoft.Owin.Security`
- `Microsoft.Owin.Security.Cookies`
- `Microsoft.Owin.Security.OpenIdConnect`
- `Owin`
- `System.IdentityModel.Tokens.Jwt`

#####.NET-Verweise

- `Microsoft.IdentityModel.Protocol.Extensions`
- `Microsoft.Owin`
- `Microsoft.Owin.Host.SystemWeb`
- `Microsoft.Owin.Security`
- `Microsoft.Owin.Security.Cookies`
- `Microsoft.Owin.Security.OpenIdConnect`
- `Owin`
- `System`
- `System.Data`
- `System.Drawing`
- `System.IdentityModel`
- `System.IdentityModel.Tokens.Jwt`
- `System.Runtime.Serialization`

#####Ihrem Projekt wurden Codedateien hinzugefügt 

Eine Authentifizierungsstartklasse (App_Start/Startup.Auth.cs) wurde Ihrem Projekt hinzugefügt. Sie enthält Startlogik für die Azure AD-Authentifizierung. Außerdem wurde eine Controllerklasse (Controllers/AccountController.cs) hinzugefügt, die SignIn()- und SignOut()-Methoden enthält. Schließlich wurde noch eine Teilansicht (Views/Shared/_LoginPartial.cshtml) hinzugefügt, die einen Aktionslink für SignIn/SignOut enthält. 

#####Ihrem Projekt wurde Startcode hinzugefügt
 
Wenn Sie bereits eine Startklasse in Ihrem Projekt verwendet haben, wurde die Methode **Configuration** so aktualisiert, dass sie einen Aufruf von ConfigureAuth(app) enthält. Andernfalls wurde Ihrem Projekt eine Startklasse hinzugefügt. 

#####Ihre Datei app.config oder web.config weist neue Konfigurationswerte auf 

Die folgenden Konfigurationseinträge wurden hinzugefügt. 
	<pre>
	`<appSettings>
	    <add key="ida:ClientId" value="ClientId from the new Azure AD App" /> 
	    <add key="ida:Tenant" value="Your selected Azure AD Tenant" /> 
	    <add key="ida:AADInstance" value="https://login.windows.net/{0}" /> 
	    <add key="Ida:PostLogoutRedirectURI" value="Your project start page" /> 
	</appSettings>` </pre>

#####Eine Azure Active Directory-App (AD) wurde erstellt 
Eine Azure AD-Anwendung wurde in dem Verzeichnis erstellt, das Sie im Assistenten ausgewählt haben. 

[Weitere Informationen zu Azure Active Directory](http://azure.microsoft.com/services/active-directory/)