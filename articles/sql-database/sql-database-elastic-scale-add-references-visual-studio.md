<properties 
	pageTitle="Hinzufügen von Elastic Scale-Verweisen der Azure SQL DB zu einem Visual Studio-Projekt" 
	description="So fügen Sie .NET-Referenzen für Elastic Scale-APIs über Nuget zu Visual Studio-Projekten hinzu" 
	services="sql-database" 
	documentationCenter="" 
	manager="stuartozer" 
	authors="stuartozer" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.workload="sql-database" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="02/16/2015" 
	ms.author="stuartozer@microsoft.com"/>

# Gewusst wie: Hinzufügen von Elastic Scale-Verweisen der Azure SQL DB zu einem Visual Studio-Projekt 

### Voraussetzungen: 

- Installieren Sie [NuGet Visual Studio Extension Gallery](http://docs.nuget.org/docs/start-here/installing-nuget) für Visual Studio 

### So fügen Sie einen Elastic Scale-Verweis in Visual Studio hinzu 

1. Öffnen Sie ein vorhandenes Projekt, oder erstellen Sie ein neues Projekt im Dialogfeld "Neues Projekt", das Sie über **Datei** --> **Neu** --> **Projekt** öffnen. 
2. Klicken Sie im Projektmappen-Explorer mit der rechten Maustaste auf **Verweise**, und wählen Sie **NuGet-Pakete verwalten** aus.
3. Wählen Sie im Menü auf der linken Seite des Fensters "NuGet-Pakete verwalten" **Online** und dann **nuget.org** oder "Alle" aus. 
4. Geben Sie im Dialogfeld **Online suchen** **Elastic Scale** ein, wählen Sie die **Elastic Scale-Clientbibliotheken** aus, und klicken Sie auf **Installieren**.
4. Lesen Sie die Lizenz, und klicken Sie auf **Ich stimme zu**. 

Die Elastic Scale-Verweise der Azure SQL-Datenbank wurden nun zu Ihrem Projekt hinzugefügt. 

[AZURE.INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

<!--HONumber=47-->
 