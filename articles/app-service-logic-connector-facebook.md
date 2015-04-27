﻿<properties 
   pageTitle="Facebook-Connector-API-App" 
   description="Verwenden des Facebook-Connectors" 
   services="app-service\logic" 
   documentationCenter=".net,nodejs,java" 
   authors="adgoda" 
   manager="dwrede" 
   editor=""/>

<tags
   ms.service="app-service-logic"
   ms.devlang="multiple"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="integration" 
   ms.date="03/20/2015"
   ms.author="adgoda"/>


# Verwenden des Facebook-Connectors in Logik-Apps #

Logik-Apps können basierend auf einer Vielzahl von Datenquellen auslösen und Connectors das Abrufen und Verarbeiten von Daten im Rahmen des Datenflusses ermöglichen. 

Mit dem Facebook-Connector können Sie "Neuer Beitrag in Benutzerchronik", "Neuer Beitrag auf Seite", "Beitrag veröffentlichen", "Foto veröffentlichen" usw. aus Ihrem Facebook-Konto abrufen.

- Der Facebook-Connector-Trigger ruft "Neuer Beitrag in Benutzerchronik" und "Neuer Beitrag auf Seite" ab. Wenn ein neuer Tweet empfangen wird, wird eine neue Instanz des Datenflusses ausgelöst, und die in der Anforderung empfangenen Daten werden zur Verarbeitung an den Datenfluss weitergegeben. 
- Mit Facebook-Connector-Aktionen können Sie "Beitrag veröffentlichen", "Foto veröffentlichen" und so weiter durchführen. Diese Aktionen erhalten eine Antwort und stellen sie den Aktionen im Datenfluss zur Nutzung zur Verfügung.

## Erstellen eines Facebook-Connectors für Ihre Logik-App ##
Zur Verwendung des Facebook-Connectors müssen Sie zunächst eine Instanz der Facebook-Connector-API-App erstellen. Dies kann wie folgt durchgeführt werden:

1.	Öffnen Sie den Azure Marketplace mit der Option "+NEU" unten links im Azure-Portal.
2.	Wechseln Sie zu "Web und Mobil > API-Apps", und suchen Sie nach "Facebook-Connector".
3.	Konfigurieren Sie den Facebook-Connector wie folgt:
 
	![][1]
4.	Klicken Sie zum Erstellen auf "OK".
5.	Sobald die API-App-Instanz erstellt wurde, können Sie in derselben Ressourcengruppe eine Logik-App zur Verwendung des Facebook-Connectors erstellen. 
	- Die API-App-Instanz des Facebook-Connectors kann auch von der Logik-App aus erstellt werden. 
	- Öffnen Sie den Logik-App-Editor, und klicken Sie auf den Facebook-Connector, der in der Galerie auf der rechten Seite zur Verfügung steht.
	- Dadurch wird eine API-App-Instanz des Facebook-Connectors in derselben Ressourcengruppe erstellt, in der die Logik-App erstellt wurde.


## Verwenden des Facebook-Connectors in Ihrer Logik-App ##
Sobald Ihre API-App erstellt wurde, können Sie jetzt den Facebook-Connector als Trigger oder Aktion für Ihre Logik-App verwenden. Gehen Sie hierzu wie folgt vor:

1.	Erstellen Sie eine neue Logik-App, und wählen Sie dieselbe Ressourcengruppe aus, in der sich der Facebook-Connector befindet.
 
	![][2]
2.	Öffnen Sie "Trigger und Aktionen", um den Logik-Apps-Designer zu öffnen und den Datenfluss zu konfigurieren. 
 
	![][3]
3.	Der Facebook-Connector wird im Abschnitt "Zuletzt verwendet" in der Galerie auf der rechten Seite angezeigt. Wählen Sie ihn aus.
 
	![][4]
4.	Sie können die Facebook-Connector-API-App im Editor ablegen, indem Sie in der Galerie auf der rechten Seite unter "Zuletzt verwendet" auf "Facebook-Connector" klicken. Klicken Sie auf die Schaltfläche "Autorisieren". Geben Sie Ihre Facebook-Anmeldeinformationen an.
  
	![][5]
5.	Zulassen von "Azure AppService-Logik-Apps" 

	![][6]
	![][7]
	![][8]     
6.	Wählen Sie einen Trigger aus.
 
	![][9]
7.	Sie können nun die vom Facebook-Trigger abgerufenen Beiträge in anderen Aktionen verwenden. Im unten stehenden Datenfluss wird immer, wenn ein neuer Beitrag in der Facebook-Chronik des Benutzers gepostet wird, derselbe Beitrag in der Twitter-Chronik des Benutzers gepostet.
 
	![][10]
8.	Auf ähnliche Weise können Sie Datenflüsse mit Facebook-Connector-Aktionen erstellen. Im folgenden Datenfluss werden neue Nachrichten, die in der Yammer-Gruppe gepostet werden, abgerufen und auf der vom Benutzer verwalteten Facebook-Seite veröffentlicht.
 
	![][11]


	<!--Image references-->
[1]: ./media/app-service-logic-connector-facebook/img1.png
[2]: ./media/app-service-logic-connector-facebook/img2.png
[3]: ./media/app-service-logic-connector-facebook/img3.png
[4]: ./media/app-service-logic-connector-facebook/img4.png
[5]: ./media/app-service-logic-connector-facebook/img5.png
[6]: ./media/app-service-logic-connector-facebook/img6.png
[7]: ./media/app-service-logic-connector-facebook/img7.png
[8]: ./media/app-service-logic-connector-facebook/img8.png
[9]: ./media/app-service-logic-connector-facebook/img9.png
[10]: ./media/app-service-logic-connector-facebook/img10.png
[11]: ./media/app-service-logic-connector-facebook/img11.png

<!--HONumber=49-->