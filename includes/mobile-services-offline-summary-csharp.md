﻿Um die Offlinefunktionen mobiler Dienste zu unterstützen, verwendeten wir die `IMobileServiceSyncTable`-Schnittstelle und initialisierten `MobileServiceClient.SyncContext` mit einem lokalen Speicher. In diesem Fall war der lokale Speicher eine SQLite-Datenbank.

Die normalen CRUD-Operationen für mobile Dienste funktionieren so, als ob die App immer noch verbunden wäre. Alle Operationen erfolgen jedoch nur im lokalen Speicher.

Als wir den lokalen Speicher mit dem Server synchronisieren wollten, verwendeten wir die `IMobileServiceSyncTable.PullAsync`- und `MobileServiceClient.SyncContext.PushAsync`-Methoden.

*  Um Änderungen an den Server zu übertragen, riefen wir  `IMobileServiceSyncContext.PushAsync()` auf. Diese Methode ist ein Element von `IMobileServicesSyncContext` anstatt der Synchronisierungstabelle, da sie Änderungen auf alle Tabellen überträgt.

    Nur Datensätze, die lokal geändert wurden (mit CRUD-Operationen), werden an den Server gesendet.
   
* Um Daten aus einer Tabelle auf dem Server auf die App zu übertragen, riefen wir `IMobileServiceSyncTable.PullAsync` auf.

    Ein Pull-Vorgang führt immer zuerst einen Push-Vorgang aus. Dadurch wird sichergestellt, dass alle Tabellen im lokalen Speicher zusammen mit Beziehungen konsistent bleiben.

    Es gibt auch Überladungen von `PullAsync()`, die es ermöglichen, eine Abfrage zu bestimmen, um die Daten zu filtern, die auf dem Client gespeichert sind. Wenn eine Abfrage nicht weitergegeben wird, bezieht `PullAsync()` alle Zeilen in der entsprechenden Tabelle (oder Abfrage). Sie können die Abfrage weitergeben, um nur die Änderungen zu filtern, mit denen Ihre App synchronisiert werden soll.

* Um inkrementelle Synchronisierung zu aktivieren, geben Sie eine Abfrage-ID an `PullAsync()` weiter. Die Abfrage-ID wird verwendet, um den zuletzt aktualisierten Zeitstempel aus den Ergebnissen des letzten Pullvorgangs zu speichern. Die Abfrage-ID sollte eine beschreibende Zeichenfolge sein, die für jede logische Abfrage in Ihrer App einmalig ist. Wenn die Abfrage einen Parameter aufweist, dann muss derselbe Parameterwert Teil der Abfrage-ID sein.

    Wenn Sie beispielsweise nach Benutzer-ID filtern, muss er Teil der Abfrage-ID sein:

        erwarte "PullAsync"("todoItems" + userid, syncTable.Where(u => u.UserId = userid));

    Wenn Sie sich von der inkrementellen Synchronisierung abmelden möchten, übergeben Sie `null` als Abfrage-ID. In diesem Fall werden alle Datensätze bei jedem Aufruf von `PullAsync`abgerufen, was potenziell ineffizient ist.

* Um Datensätze aus den lokalen Speicher des Gerätes zu entfernen, wenn diese in der Datenbank Ihres mobilen Dienstes gelöscht worden sind, sollten Sie [vorläufiges Löschen] aktivieren. Andernfalls sollte Ihre App regelmäßig `IMobileServiceSyncTable.PurgeAsync()` aufrufen, um den lokalen Speicher zu löschen.
<!--HONumber=42-->