<properties 
	pageTitle="" 
	description="Beschreibt, was mit Ihrem Azure Mobile Services .NET-Projekt in Visual Studio passiert ist." 
	services="mobile-services" 
	documentationCenter="" 
	authors="patshea123" 
	manager="douge" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.workload="mobile" 
	ms.tgt_pltfrm="NA" 
	ms.devlang="dotnet" 
	ms.topic="article" 
	ms.date="05/06/2015" 
	ms.author="patshea123"/>

# Was ist mit meinem Projekt passiert?

> [AZURE.SELECTOR]
> - [Getting Started](vs-mobile-services-dotnet-getting-started.md)
> - [What Happened](vs-mobile-services-dotnet-what-happened.md)

### <span id="whathappened">Was ist mit meinem Projekt passiert?</span>

##### Verweise wurden hinzugefügt

Das NuGet-Paket für Azure Mobile Services wurde Ihrem Projekt hinzugefügt. Die folgenden .NET-Verweise wurde Ihrem Projekt hinzugefügt:

- `Microsoft.WindowsAzure.Mobile`
- `Microsoft.WindowsAzure.Mobile.Ext`
- `Newtonsoft.Json`
- `System.Net.Http.Extensions`
- `System.Net.Http.Primitives` 

##### Werte für die Verbindungszeichenfolge für Mobile Services

In Ihrer Datei **App.xaml.cs** wurde ein MobileServiceClient-Objekt mit der Anwendungs-URL des ausgewählten mobilen Diensts und seinem Anwendungsschlüssel erstellt.

[Weitere Informationen zu mobilen Diensten](http://azure.microsoft.com/documentation/services/mobile-services/)
<!--HONumber=54--> 