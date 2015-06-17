<properties
	pageTitle="Aktivieren der Offlinesynchronisierung für Ihre mobile App (iOS)"
	description="Erfahren Sie, wie Sie mobile App Service-Apps verwenden, um Offlinedaten in Ihrer iOS-Anwendung zwischenzuspeichern und zu synchronisieren."
	documentationCenter="ios"
	authors="lindydonna"
	manager="dwrede"
	editor=""
	services="app-service\mobile"/>

<tags
	ms.service="app-service-mobile"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-ios"
	ms.devlang="objective-c"
	ms.topic="article"
	ms.date="02/23/2015"
	ms.author="donnam"/>

# Aktivieren der Offlinesynchronisierung für Ihre mobile iOS-App

[AZURE.INCLUDE [app-service-mobile-selector-offline-preview](../../includes/app-service-mobile-selector-offline-preview.md)]

In diesem Lernprogramm wird die Offlinesynchronisierungsfunktion von Mobile Apps für iOS behandelt. Offlinesynchronisierung ermöglicht Endbenutzern die Interaktion mit einer mobilen App – Anzeigen, Hinzufügen und Ändern von Daten –, auch wenn keine Netzwerkverbindung vorhanden ist. Änderungen werden in einer lokalen Datenbank gespeichert. Sobald das Gerät wieder online ist, werden diese Änderungen mit dem Remote-Back-End synchronisiert.

Die Offlinesynchronisierung hat eine Reihe potenzieller Nutzen:

* Verbesserte Reaktionsfähigkeit der App durch das lokale Caching von Serverdaten auf dem Gerät
* Macht Apps unempfindlich gegen Unterbrechungen der Netzwerkverbindung
* Endanwender können Daten selbst dann erstellen und ändern, wenn sie keinen Netzwerkzugriff haben, also in Szenarien mit wenig oder keiner Konnektivität
* Daten auf mehreren Geräten synchronisieren und Konflikte erkennen, wenn derselbe Datensatz von zwei Geräte aus geändert wird

Falls Sie noch keine Erfahrung mit Mobile Apps haben, sollten Sie zunächst das Lernprogramm [Erstellen einer iOS-App] abschließen.

## <a name="get-app"></a>Abrufen der Offline-ToDo-Beispiel-App

Klonen Sie im [Mobile Apps-Beispielrepository auf GitHub] das Repository, und öffnen Sie das Projekt [iOS-Offlinebeispiel] in Xcode.

### Beta-SDK
Laden Sie das neueste [iOS-Beta-SDK](http://aka.ms/gc6fex) herunter, um einer vorhandenen App Offlineunterstützung hinzuzufügen.

## <a name="review-sync"></a>Überprüfen des Mobile Apps-Codes für die Synchronisierung

Mit der Mobile App-Offlinesynchronisierung können Endbenutzer mit einer lokalen Datenbank interagieren, wenn das Netzwerk nicht verfügbar ist. Um diese Funktionen in Ihrer App zu verwenden, initialisieren Sie den Synchronisierungskontext von `MSClient`, und verweisen Sie auf einen lokalen Datenspeicher. Erstellen Sie dann über die `MSSyncTable`-Schnittstelle einen Verweis für die Tabelle.

In diesem Abschnitt wird der für die Offlinesynchronisierung relevante Code am Beispiel erläutert.

1. Beachten Sie in **QSTodoService.m**, dass der Typ des Members `syncTable` `MSSyncTable` ist. Bei der Offlinesynchronisierung wird diese Synchronisierungstabellen-Schnittstelle anstelle von `MSTable` verwendet. Wenn eine Synchronisierungstabelle verwendet wird, werden alle Vorgänge im lokalen Speicher gespeichert und nur über explizite Push- und Pullvorgänge mit dem Remote-Back-End synchronisiert.

    Um einen Verweis auf eine Synchronisierungstabelle zu erstellen, verwenden Sie die `syncTableWithName`-Methode. Zum Entfernen der Offlinesynchronisierung verwenden Sie stattdessen `tableWithName`.

3. Bevor Tabellenvorgänge durchgeführt werden können, muss der lokale Speicher initialisiert werden. Dies ist der entsprechende Code in der `QSTodoService.init`-Methode:

        MSCoreDataStore *store = [[MSCoreDataStore alloc] initWithManagedObjectContext:context];

        self.client.syncContext = [[MSSyncContext alloc] initWithDelegate:nil dataSource:store callback:nil];

    Dadurch wird ein lokaler Speicher mithilfe der `MSCoreDataStore`-Schnittstelle erstellt, die in der Mobile Apps-SDK bereitgestellt wird. Sie können stattdessen auch einen anderen lokalen Speicher bereitstellen, indem Sie das `MSSyncContextDataSource`-Protokoll implementieren.

    Der erste Parameter von `initWithDelegate` wird zum Angeben eines Konflikthandlers verwendet. Da wir `nil` weitergegeben haben, verwenden wir den Standardkonflikthandler, der bei jedem Konflikten einen Fehler ausgibt.

<!-- For details on how to implement a custom conflict handler, see the tutorial [Handling conflicts with offline support for Mobile Services]. -->

4. Die Methoden `pullData` und `syncData` führen die eigentliche Synchronisierung aus: `syncData` führt zunächst einen Pushvorgang für neue Änderungen aus, anschließend werden mithilfe von `pullData` die Daten vom Remote-Back-End abgerufen.

        -(void)syncData:(QSCompletionBlock)completion
        {
            // push all changes in the sync context, then pull new data
            [self.client.syncContext pushWithCompletion:^(NSError *error) {
                [self logErrorIfNotNil:error];
                [self pullData:completion];
            }];
        }

    Im Gegenzug erhält die Methode `pullData` die neuen Daten, die einer Abfrage entsprechen:

        -(void)pullData:(QSCompletionBlock)completion
        {
            MSQuery *query = [self.syncTable query];

            // Pulls data from the remote server into the local table.
            // We're pulling all items and filtering in the view
            // query ID is used for incremental sync
            [self.syncTable pullWithQuery:query queryId:@"allTodoItems" completion:^(NSError *error) {
                [self logErrorIfNotNil:error];

                // Let the caller know that we have finished
                if (completion != nil) {
                    dispatch_async(dispatch_get_main_queue(), completion);
                }
            }];
        }

    In `syncData` rufen wir zuerst `pushWithCompletion` im Synchronisierungskontext auf. Diese Methode ist ein Mitglied von `MSSyncContext` (statt der Synchronisierungstabelle selbst), da sie Änderungen auf alle Tabellen übertragen wird. Nur Datensätze, die lokal geändert wurden (mit CRUD-Operationen), werden an den Server gesendet. Anschließend wird das Hilfsprogramm `pullData` aufgerufen, das wiederum `MSSyncTable.pullWithQuery` aufruft, um Remotedaten abzurufen und in der lokalen Datenbank zu speichern.

    Beachten Sie, dass in diesem Beispiel der Pushvorgang nicht unbedingt notwendig war. Wenn im Synchronisierungskontext der Tabelle, die einen Pushvorgang durchführt, Änderungen ausstehen, wird durch den Pullvorgang immer zuerst ein Pushvorgang ausgeführt. Wenn Sie über mehr als eine Synchronisierungstabelle verfügen, ist es jedoch am besten, explizit einen Pushvorgang auszuführen, um sicherzustellen, dass die verknüpften Tabellen konsistent sind.

    Mit der Methode `pullWithQuery` können Sie eine Abfrage zum Filtern der Datensätze angeben, die Sie abrufen möchten. In diesem Beispiel ruft die Abfrage einfach alle Datensätze in der Remotetabelle `TodoItem` ab.

    Der zweite Parameter für `pullWithQuery` ist eine Abfrage-ID, die für die *inkrementelle Synchronisierung* verwendet wird. Die inkrementelle Synchronisierung ruft nur die Datensätze ab, die seit der letzten Synchronisierung geändert wurden. Dafür wird der Zeitstempel `UpdatedAt` (im lokalen Speicher `ms_updatedAt`) des Datensatzes verwendet. Die Abfrage-ID muss eine beschreibende Zeichenfolge sein, die für jede logische Abfrage in Ihrer App eindeutig ist. Um die inkrementelle Synchronisierung zu deaktivieren, übergeben Sie als Abfrage-ID `nil`. Beachten Sie, dass dies möglicherweise ineffizient ist, da auf diese Weise bei jedem Pullvorgang alle Datensätze abgerufen werden.

<!--     >[AZURE.NOTE] To remove records from the device local store when they have been deleted in your mobile service database, you should enable [Soft Delete]. Otherwise, your app should periodically call `MSSyncTable.purgeWithQuery` to purge the local store.
 -->

5. In der `QSTodoService`-Klasse wird die `syncData`-Methode nach den Vorgängen zum Ändern von Daten, `addItem` und `completeItem`, aufgerufen. Sie wird auch von `QSTodoListViewController.refresh` aufgerufen, sodass Benutzer die aktuellsten Daten abrufen, wenn sie die Akualisierungsbewegung durchführen. Die App führt außerdem eine Synchronisierung beim Start durch, da `QSTodoListViewController.init` einen Aufruf von `refresh` durchführt.

    Da bei jeder Änderung von Daten `syncData` aufgerufen wird, setzt diese App voraus, dass der Benutzer beim Bearbeiten von Daten stets online ist. In einem anderen Abschnitt werden wir die App modifizieren, sodass Benutzer auch im Offlinemodus Daten bearbeiten können.

## <a name="review-core-data"></a>Überprüfen des Core-Datenmodells

Bei Verwendung des Core-Offlinedatenspeichers müssen Sie bestimmte Tabellen und Felder in Ihrem Datenmodell definieren. Die Beispiel-App enthält bereits ein Datenmodell mit dem richtigen Format. In diesem Abschnitt durchlaufen wir diese Tabellen und ihre Verwendung.

- Öffnen Sie **QSDataModel.xcdatamodeld**. Es sind vier Tabellen definiert – drei, die durch das SDK verwendet werden, und eine Tabelle für die Todo-Elemente selbst:
      * MS_TableOperations: Zum Nachverfolgen der Elemente, die mit dem Server synchronisiert werden müssen
      * MS_TableOperationErrors: Zum Nachverfolgen von Fehlern, die während der Offlinesynchronisierung auftreten
      * MS_TableConfig: Zum Nachverfolgen der letzten aktualisierten Zeit für die letzte Synchronisierung für alle Pullvorgänge
      * TodoItem: Zum Speichern der TODO-Elemente. Die Systemspalten **ms_createdAt**, **ms_updatedAt** und **ms_version** sind optionale Systemeigenschaften.

>[AZURE.NOTE]Das Mobile App-SDK reserviert Spaltennamen, die mit "**`ms_`**" beginnen. Dieses Präfix sollte nur für Systemspalten verwendet werden, andernfalls werden die Spaltennamen bei Verwendung des Remote-Back-Ends geändert.

- Zur Verwendung der Funktion zur Offlinesynchronisierung müssen Sie die Systemtabellen definieren, wie unten dargestellt.

    ### Systemtabellen

    **MS_TableOperations**

    ![][defining-core-data-tableoperations-entity]

    | Attribut | Typ |
    |----------- |   ------    |
    | id | Integer 64 |
    | itemId | String |
    | Eigenschaften | Binärdaten |
    | Tabelle | String |
    | tableKind | Integer 16 |

    <br>**MS_TableOperationErrors**

    ![][defining-core-data-tableoperationerrors-entity]

    | Attribut | Typ |
    |----------- |   ------    |
    | id | String |
    | operationId | Integer 64 |
    | Eigenschaften | Binärdaten |
    | tableKind | Integer 16 |

    <br>**MS_TableConfig**

    ![][defining-core-data-tableconfig-entity]

    | Attribut | Typ |
    |----------- |   ------    |
    | id | Integer 64 |
    | key | String |
    | keyType | Integer 64 |
    | Tabelle | String |
    | value | String |

    ### Datentabelle

    ![][defining-core-data-todoitem-entity]

    **TodoItem**


    | Attribut | Typ | Hinweis |
    |-----------   |  ------ | -------------------------------------------------------|
    | id | String | Primärschlüssel im Remotespeicher |
    | complete | Boolean | Todo-Elementfeld |
    | Text | String | Todo-Elementfeld |
    | ms_createdAt | Datum | (optional) Zuordnung zur __createdAt-Systemeigenschaft | | ms_updatedAt | Datum | (optional) Zuordnung zur __updatedAt-Systemeigenschaft | | ms_version | String | (optional) Zum Erkennen von Konflikten, Zuordnung zu __version |


## <a name="setup-sync"></a>Ändern des Synchronisierungsverhaltens der App

In diesem Abschnitt ändern Sie die Anwendung, sodass beim Starten der App oder beim Einfügen und Aktualisieren von Elementen keine Synchronisierung erfolgt, sondern nur, wenn die Aktualisierungsschaltfläche betätigt wird.

1. Ändern Sie in **QSTodoListViewController.m**, die **viewDidLoad**-Methode, um den Aufruf von `[self refresh]` am Ende der Methode zu entfernen. Die Daten werden jetzt beim Starten der App nicht mit dem Server synchronisiert, sondern stattdessen die Inhalte des lokalen Speichers.

2. Ändern Sie in **QSTodoService.m** die Definition von `addItem`, sodass nach dem Einfügen des Elements keine Synchronisierung erfolgt. Entfernen Sie den `self syncData`-Block, und ersetzen Sie ihn durch Folgendes:

            if (completion != nil) {
                dispatch_async(dispatch_get_main_queue(), completion);
            }

3. Ändern Sie die Definition von `completeItem` wie oben beschrieben. Entfernen Sie den Block für `self syncData`, und ersetzen Sie ihn durch Folgendes:

            if (completion != nil) {
                dispatch_async(dispatch_get_main_queue(), completion);
            }

## <a name="test-app"></a>Testen der App


In diesem Abschnitt werden Sie im Simulator Wi-Fi deaktivieren, um ein Offlineszenario zu erzeugen. Wenn Sie Datenelemente hinzufügen, werden diese im lokalen Core-Datenspeicher vorgehalten, jedoch nicht mit dem mobilen Back-End synchronisiert.

1. Deaktivieren Sie Wi-Fi im iOS-Simulator.

2. Fügen Sie einige TODO-Elemente hinzu, oder schließen Sie einige Elemente ab. Beenden Sie den Simulator (oder Erzwingen Sie das Schließen der App), und starten Sie ihn neu. Stellen Sie sicher, dass die Änderungen erhalten geblieben sind.

3. Zeigen Sie den Inhalt der TodoItem-Remotetabelle an:
   - Klicken Sie für das JavaScript-Back-End auf dem Verwaltungsportal auf die Registerkarte "Daten", um den Inhalt der `TodoItem`-Tabelle anzuzeigen.
   - Zeigen Sie für das .NET-Back-End den Inhalt der Tabelle entweder mit einem SQL-Tool wie SQL Server Management Studio oder einen REST-Client wie Fiddler oder Postman an.

    Stellen Sie sicher, dass die neuen Elemente *nicht* auf dem Server synchronisiert wurden:

4. Aktivieren Sie Wi-Fi im iOS-Simulator, und führen Sie die Aktualisierungsbewegung aus, indem Sie die Liste der Elemente nach unten ziehen. Eine Synchronisierungsanzeige und der Hinweis "Syncing.." werden angezeigt.

5. Zeigen Sie die TodoItem-Daten erneut an. Es sollten jetzt die neuen und geänderten TodoItems angezeigt werden.

## Zusammenfassung

Um die Offlinesynchronisierungsfunktion zu unterstützen, wurde die Schnittstelle `MSSyncTable` verwendet und `MSClient.syncContext` mit einem lokalen Speicher initialisiert. In diesem Fall war der lokale Speicher eine auf Core-Daten basierende Datenbank.

Wenn Sie einen lokalen Core-Datenspeicher verwenden, müssen Sie mehrere Tabellen mit [korrekten Systemeigenschaften][Review the Core Data model] definieren.

Die normalen CRUD-Operationen für Mobile Apps funktionieren so, als ob die App immer noch verbunden wäre. Alle Operationen werden jedoch nur im lokalen Speicher ausgeführt.

Zur Synchronisierung des lokalen Speichers mit dem Server haben Sie die Methoden `MSSyncTable.pullWithQuery` und `MSClient.syncContext.pushWithCompletion` verwendet.

*  Anschließend haben Sie `Review the Core Data model` aufgerufen, um die Änderungen per Push-Vorgang auf den Server zu übertragen. Diese Methode ist ein Element von `MSSyncContext` und nicht der Synchronisierungstabelle, da Änderungen per Push auf alle Tabellen übertragen werden.

    Nur Datensätze, die lokal geändert wurden (mit CRUD-Operationen), werden an den Server gesendet.

* Um Daten per Pull-Vorgang von einer Tabelle auf dem Server in die App zu übertragen, haben Sie `MSSyncTable.pullWithQuery` aufgerufen.

    Ein Pull-Vorgang führt immer zuerst einen Push-Vorgang aus. Dadurch wird sichergestellt, dass alle Tabellen im lokalen Speicher und die Beziehungen konsistent bleiben.

    Beachten Sie, dass `pullWithQuery` zum Filtern von Daten verwendet werden kann, die auf dem Client gespeichert sind. Dazu muss der `query`-Parameter angepasst werden.

* Um die inkrementelle Synchronisierung zu aktivieren, übergeben Sie eine Abfrage-ID an `pullWithQuery`. Die Abfrage-ID wird verwendet, um den zuletzt aktualisierten Zeitstempel aus den Ergebnissen des letzten Pullvorgangs zu speichern. Die Abfrage-ID muss eine beschreibende Zeichenfolge sein, die für jede logische Abfrage in Ihrer App eindeutig ist. Wenn die Abfrage einen Parameter aufweist, muss derselbe Parameterwert Teil der Abfrage-ID sein.

    Wenn Sie die inkrementelle Synchronisierung deaktivieren möchten, übergeben Sie `nil` als Abfrage-ID. In diesem Fall werden alle Datensätze bei jedem Aufruf von `pullWithQuery` abgerufen, was möglicherweise ineffizient ist.

<!-- * To remove records from the device local store when they have been deleted in your mobile service database, you should enable [Soft Delete]. Otherwise, your app should periodically call `MSSyncTable.purgeWithQuery` to remove records from the local database, in case they have been deleted in the remote service.
 -->

## Zusätzliche Ressourcen

* [Cloud Cover: Offlinesynchronisierung in Azure Mobile Services]

* [Azure Friday: Offlinefähige Apps in Azure Mobile Services] (Hinweis: Demos sind für Windows, Funktionserläuterungen gelten jedoch für alle Plattformen)

<!-- URLs. -->

[Erstellen einer iOS-App]: ../app-service-mobile-dotnet-backend-ios-get-started.md

[core-data-1]: ./media/mobile-services-ios-get-started-offline-data/core-data-1.png
[core-data-2]: ./media/mobile-services-ios-get-started-offline-data/core-data-2.png
[core-data-3]: ./media/mobile-services-ios-get-started-offline-data/core-data-3.png
[defining-core-data-main-screen]: ./media/mobile-services-ios-get-started-offline-data/defining-core-data-main-screen.png
[defining-core-data-model-editor]: ./media/mobile-services-ios-get-started-offline-data/defining-core-data-model-editor.png
[defining-core-data-tableoperationerrors-entity]: ./media/app-service-mobile-ios-get-started-offline-data-preview/defining-core-data-tableoperationerrors-entity.png
[defining-core-data-tableoperations-entity]: ./media/app-service-mobile-ios-get-started-offline-data-preview/defining-core-data-tableoperations-entity.png
[defining-core-data-tableconfig-entity]: ./media/app-service-mobile-ios-get-started-offline-data-preview/defining-core-data-tableconfig-entity.png
[defining-core-data-todoitem-entity]: ./media/app-service-mobile-ios-get-started-offline-data-preview/defining-core-data-todoitem-entity.png
[update-framework-1]: ./media/mobile-services-ios-get-started-offline-data/update-framework-1.png
[update-framework-2]: ./media/mobile-services-ios-get-started-offline-data/update-framework-2.png

[Core Data Model Editor Help]: https://developer.apple.com/library/mac/recipes/xcode_help-core_data_modeling_tool/Articles/about_cd_modeling_tool.html
[Creating an Outlet Connection]: https://developer.apple.com/library/mac/recipes/xcode_help-interface_builder/articles-connections_bindings/CreatingOutlet.html
[Build a User Interface]: https://developer.apple.com/library/mac/documentation/ToolsLanguages/Conceptual/Xcode_Overview/Edit_User_Interfaces/edit_user_interface.html
[Adding a Segue Between Scenes in a Storyboard]: https://developer.apple.com/library/ios/recipes/xcode_help-IB_storyboard/chapters/StoryboardSegue.html#//apple_ref/doc/uid/TP40014225-CH25-SW1
[Adding a Scene to a Storyboard]: https://developer.apple.com/library/ios/recipes/xcode_help-IB_storyboard/chapters/StoryboardScene.html

[Core Data]: https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreData/cdProgrammingGuide.html
[Download the preview SDK here]: http://aka.ms/Gc6fex
[How to use the Mobile Services client library for iOS]: ../mobile-services-ios-how-to-use-client-library.md
[iOS-Offlinebeispiel]: https://github.com/Azure/mobile-services-samples/tree/master/TodoOffline/iOS/blog20140611
[Mobile Apps-Beispielrepository auf GitHub]: https://github.com/Azure/mobile-services-samples

[Get started with Mobile Services]: ../mobile-services-ios-get-started.md
[Get started with data]: ../mobile-services-ios-get-started-data.md
[Handling conflicts with offline support for Mobile Services]: ../mobile-services-ios-handling-conflicts-offline-data.md
[Soft Delete]: ../mobile-services-using-soft-delete.md

[Cloud Cover: Offlinesynchronisierung in Azure Mobile Services]: http://channel9.msdn.com/Shows/Cloud+Cover/Episode-155-Offline-Storage-with-Donna-Malayeri
[Azure Friday: Offlinefähige Apps in Azure Mobile Services]: http://azure.microsoft.com/documentation/videos/azure-mobile-services-offline-enabled-apps-with-donna-malayeri/
 

<!---HONumber=GIT-SubDir_Tue_AM_dede-->