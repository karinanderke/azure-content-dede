<properties
   pageTitle="Verwendung des Facebook-Connectors in Logik-Apps | Microsoft Azure App Service"
   description="Erstellen und Konfigurieren des Facebook-Connectors oder einer API-App und Verwenden in einer Logik-App in Azure App Service"
   services="app-service\logic"
   documentationCenter=".net,nodejs,java"
   authors="rajeshramabathiran"
   manager="dwrede"
   editor=""/>

<tags
   ms.service="app-service-logic"
   ms.devlang="multiple"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="integration"
   ms.date="11/11/2015"
   ms.author="rajram"/>


# Erste Schritte mit dem Facebook-Connector und das Hinzufügen zur Logik-App
Stellen Sie eine Verbindung mit einem Facebook-Konto her, um eine Nachricht oder ein Foto zu veröffentlichen. Logik-Apps können basierend auf einer Vielzahl von Datenquellen ausgelöst werden und Connectors anbieten, um Daten als Teil des Datenflusses abzurufen und zu verarbeiten.

Der Facebook-Connector ermöglicht Folgendes:

- Verwenden Sie Trigger zum Abrufen neuer Beiträge von der Pinwand des Benutzers oder zum Veröffentlichen eines neuen Beitrags. Wenn ein neuer Beitrag empfangen wird, wird eine neue Instanz des Datenflusses ausgelöst, und die in der Anforderung empfangenen Daten werden zur Verarbeitung an den Datenfluss weitergegeben.
- Verwenden Sie Aktionen wie "Beitrag veröffentlichen", "Foto veröffentlichen" usw. Diese Aktionen erhalten eine Antwort und stellen sie den Aktionen im Datenfluss zur Nutzung zur Verfügung.

Sie können den Facebook-Connector dem geschäftlichen Workflow hinzufügen und Daten im Rahmen dieses Workflows in einer Logik-App verarbeiten.

## Trigger und Aktionen

Trigger | Aktionen
--- | ---
<ul><li>Neuer Beitrag in Benutzerchronik</li><li>Neuer Beitrag auf der Seite</li></ul> | <ul><li>Beitrag veröffentlichen</li><li>Foto veröffentlichen</li></ul>



## Erstellen des Facebook-Connectors für Ihre Logik-App
Ein Connector kann innerhalb einer Logik-App erstellt werden oder direkt aus dem Azure Marketplace. So erstellen Sie einen Connector aus dem Marketplace:

1. Wählen Sie im Azure-Startmenü **Marketplace** aus.
2. Suchen Sie nach "Facebook-Connector", wählen Sie ihn aus, und klicken Sie dann auf **Erstellen**.
3. Geben Sie den Namen, den App Service-Plan und andere Eigenschaften ein: ![][1]
4.	Klicken Sie auf **Erstellen**.


## Verwenden des Facebook-Connectors in Ihrer Logik-App
Sobald Ihre API-App erstellt wurde, können Sie jetzt den Facebook-Connector als Trigger oder Aktion für Ihre Logik-App verwenden. Gehen Sie hierzu wie folgt vor:

1.	Öffnen Sie in der Logik-App **Trigger und Aktionen**, um den Logik-Apps-Designer zu öffnen und den Datenfluss zu konfigurieren: ![][3]
2.	Der Facebook-Connector wird im Katalog aufgeführt: ![][4]
3. Wählen Sie den Facebook-Connector aus, um ihn automatisch im Designer hinzuzufügen. Klicken Sie auf **Autorisieren**, geben Sie Ihre Anmeldeinformationen ein, und klicken Sie auf **Zulassen**: ![][5] ![][6] ![][7] ![][8]
4.	Wählen Sie einen Trigger aus: ![][9]

Sie können nun die vom Facebook-Trigger abgerufenen Beiträge in anderen Aktionen verwenden. Im unten stehenden Datenfluss wird immer, wenn ein neuer Beitrag in der Facebook-Chronik des Benutzers gepostet wird, derselbe Beitrag in der Twitter-Chronik des Benutzers gepostet: ![][10]

Auf ähnliche Weise können Sie Datenflüsse mit Facebook-Connector-Aktionen erstellen. Im folgenden Datenfluss werden neue Nachrichten, die in der Yammer-Gruppe veröffentlicht werden, abgerufen und auf der vom Benutzer verwalteten Facebook-Seite veröffentlicht: ![][11]

> [AZURE.TIP]Zum Abrufen der Facebook-Seiten-ID oder der Yammer-Gruppen-ID suchen Sie nach dem numerischen Code in der URL.

## Mehr mit Ihrem Connector machen
Nachdem der Connector nun erstellt ist, können Sie ihn mit Logik-App in einem Geschäftsworkflow hinzufügen. Informationen finden Sie unter [Was sind Logik-Apps?](app-service-logic-what-are-logic-apps.md).

>[AZURE.NOTE]Wenn Sie Azure Logik-Apps ausprobieren möchten, ehe Sie sich für ein Azure-Konto anmelden, können Sie unter [Logik-App testen](https://tryappservice.azure.com/?appservice=logic) sofort kostenlos eine kurzlebige Starter-Logik-App in App Service erstellen. Keine Kreditkarte erforderlich, keine Verpflichtungen.

Anzeigen der Swagger-REST-API-Referenz unter [Referenz zu Connectors und API-Apps](http://go.microsoft.com/fwlink/p/?LinkId=529766).

Sie können auch Leistungsstatistiken überprüfen und die Sicherheit zum Connector steuern. Informationen finden Sie unter [Verwalten und Überwachen integrierter API-Apps und Connectors](app-service-logic-monitor-your-connectors.md).

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

<!---HONumber=Nov15_HO3-->