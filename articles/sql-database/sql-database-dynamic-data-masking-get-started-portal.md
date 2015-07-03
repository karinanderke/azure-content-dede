<properties 
   pageTitle="Erste Schritte mit der dynamischen Datenmaskierung für SQL-Datenbanken (Azure-Portal)" 
   description="Einstieg in die dynamische Datenmaskierung für SQL-Datenbanken im Azure-Portal" 
   services="sql-database" 
   documentationCenter="" 
   authors="nadavhelfman" 
   manager="jeffreyg" 
   editor="v-romcal"/>

<tags
   ms.service="sql-database"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services" 
   ms.date="03/25/2015"
   ms.author="nadavh; ronmat; v-romcal; sstein"/>

# Erste Schritte mit der dynamischen Datenmaskierung für SQL-Datenbanken (Azure-Portal)

> [AZURE.SELECTOR]
- [Dynamische Datenmaskierung - Azure-Vorschauportal](sql-database-dynamic-data-masking-get-started.md)

## Übersicht

Die dynamische Datenmaskierung für SQL-Datenbanken schränkt die Offenlegung vertraulicher Daten ein, indem sie für nicht berechtigte Benutzer maskiert werden. Die dynamische Datenmaskierung wird als Vorschau für die Dienstebenen Basic, Standard und Premium der Version V12 von Azure SQL-Datenbank angeboten.

Die dynamische Datenmaskierung hilft beim Verhindern des unbefugten Zugriffs auf sensible Daten, indem Kunden festlegen dürfen, welcher Anteil der sensiblen Daten mit minimalen Auswirkungen auf die Anwendungsschicht offengelegt wird. Es handelt sich um eine richtlinienbasierte Sicherheitsfunktion, die die sensiblen Daten im Resultset einer Abfrage in festgelegten Datenbankfeldern ausblendet, ohne dass die Daten in der Datenbank geändert werden.

In einem Callcenter kann ein Supportmitarbeiter beispielsweise Anrufer anhand mehrerer Ziffern ihrer Sozialversicherungs- oder Kreditkartennummer identifizieren, wobei diese Datenelemente dem Supportmitarbeiter jedoch nicht vollständig angezeigt werden sollen. Ein Entwickler kann eine Maskierungsregel, die auf jedes Abfrageergebnis angewendet wird, so definieren, dass alle Ziffern von Sozialversicherungs- oder Kreditkartennummern bis auf die letzten vier im Resultset maskiert werden. In einem weiteren Beispiel kann ein Entwickler durch Verwenden der entsprechenden Datenmaske zum Schutz personenbezogener Daten Produktionsumgebungen zu Problembehandlungszwecken abfragen, ohne gegen Vorschriften zu verstoßen.

## Grundlagen der dynamischen Datenmaskierung für SQL-Datenbanken

Sie richten die dynamische Datenmaskierungsrichtlinie im Azure-Portal ein und schließen die Einrichtung ab, indem Sie die sicherheitsaktivierte Verbindungszeichenfolge einsetzen, die von der Anwendung oder anderen Clients genutzt wird, die auf die Datenbank zugreifen.

> [AZURE.NOTE] Informationen zum Einrichten der dynamischen Datenmaskierung im Azure-Vorschauportal finden Sie unter [Erste Schritte mit der dynamischen Datenmaskierung für SQL-Datenbanken (Azure-Vorschauportal)](sql-database-dynamic-data-masking-get-started.md). 

### Berechtigungen für die dynamische Datenmaskierung

Die dynamische Datenmaskierung kann von den Rollen "Azure-Datenbankadministrator", "Serveradministrator" oder "Sicherheitsbeauftragter" konfiguriert werden.

### Richtlinie für die dynamische Datenmaskierung

* **Privilegierte Anmeldungen** - Eine Gruppe von Anmeldungen, die Daten ohne Maskierung in den SQL-Abfrageergebnissen erhalten.
  
* **Maskierungsregeln** - Eine Gruppe von Regeln, die die zu maskierenden Felder und verwendete Maskierungsfunktion definieren. Mithilfe eines Datenbanktabellen- und Spaltennamens bzw. eines Aliasnamens können die vorgesehenen Felder bestimmt werden.

* **Maskieren nach** - Kann in Quelle oder Ziel erfolgen. Die Maskierung kann auf Quellebene konfiguriert werden, indem der **Tabellenname** und **Spaltenname** bestimmt werden, oder auf Ergebnisebene, indem der in der Abfrage verwendete **Alias** bestimmt wird. Wenn Sie mit der Datenarchitektur Ihrer Datenbank vertraut sind und die Anzeige aller Abfrageergebnisse begrenzen möchten, bevorzugen Sie ggf. Sie eine Quellmaskenregel. Sie können eine Ergebnismaskenregel hinzufügen, wenn Sie die Anzeige auf Abfrageergebnisse beschränken möchten, ohne die Datenarchitektur der Datenbank zu analysieren, oder auf ein Feld, das aus verschiedenen Quellen stammen kann.  
  
* **Erweiterte Einschränkung** - Schränkt die Anzeige sensibler Daten ein, kann sich aber negativ auf die Funktionalität einiger Anwendungen auswirken.

>[AZURE.TIPP] Verwenden Sie die Option **Extended restriction** zum Beschränken des Zugriffs für Entwickler, die einen direkten Zugriff auf die Datenbank nutzen und SQL-Abfragen zu Problembehandlungszwecken ausführen.

* **Maskierungsfunktionen** - Eine Reihe von Methoden, die die Anzeige von Daten in verschiedenen Szenarien steuern.

| Maskierungsfunktion | Maskierungslogik |
|----------|---------------|
| **Standard**  |**Vollständige Maskierung gemäß den Datentypen der festgelegten Felder**<br/><br/>• Verwenden Sie XXXXXXXX oder weniger Xe, wenn die Größe des Felds weniger als 8 Zeichen für STRING-Datentypen (nchar, ntext, nvarchar) beträgt.<br/>• Verwenden Sie einen Nullwert für numerische Datentypen (bigint, bit, decimal, int, money, numeric, smallint, smallmoney, tinyint, float, real).<br/>• Verwenden Sie aktuelle Zeit für DATE/TIME-Datentypen (date, datetime2, datetime, datetimeoffset, smalldatetime, time).<br/>• Für SQL-Varianten wird der Standardwert des aktuellen Typs verwendet.<br/>• Für XML wird das Dokument <masked/> verwendet.<br/>• Verwenden Sie einen leeren Wert für Sonderdatentypen (timestamp, table, hierarchyid, GUID, binary, image, varbinary, räumliche Typen).
| **Kreditkarte** |**Maskierungsmethode, die die letzten vier Ziffern der festgelegten Felder anzeigt** und eine Konstantenzeichenfolge als Präfix in Form einer Kreditkarte hinzufügt.<br/><br/>XXXX-XXXX-XXXX-1234|
| **Sozialversicherungsnummer** |**Maskierungsmethode, die die letzten zwei Ziffern der festgelegten Felder anzeigt** und eine Konstantenzeichenfolge als Präfix in Form einer US-amerikanischen Sozialversicherungsnummer hinzufügt.<br/><br/>XXX-XX-XX12 |
| **E-Mail-Adresse** | **Maskierungsmethode, die den ersten Buchstaben und die Domäne anzeigt** und eine Konstantenzeichenfolge als Präfix in Form einer E-Mail-Adresse verwendet.<br/><br/>aXX@XXXX.com |
| **Zufallszahl** | **Maskierungsmethode, die eine Zufallszahl** entsprechend den ausgewählten Grenzen und den tatsächlichen Datentypen generiert. Wenn die festgelegten Grenzen gleich sind, ist die Maskierungsfunktion eine konstante Zahl.<br/><br/>![Navigationsbereich][Image1] |
| **Benutzerdefinierter Text** | **Maskierungsmethode, die den ersten und letzten Buchstaben anzeigt** und in der Mitte eine benutzerdefinierte Auffüllzeichenfolge hinzufügt.<br/>Präfix[Auffüllung]Suffix<br/><br/>![Navigationsbereich][Image2] |

  
<a name="Anchor1"></a>
### Verbindungszeichenfolge mit aktivierter Sicherheit

Beim Einrichten der dynamische Datenmaskierung stellt Azure für die Datenbank eine Verbindungszeichenfolge mit aktivierter Sicherheit bereit. Nur bei Clients, die diese Verbindungszeichenfolge verwenden, werden sensible Daten gemäß der Richtlinie für die dynamische Datenmaskierung maskiert. Sie müssen auch vorhandene Clients (z. B. Anwendungen) so aktualisieren, dass sie das neue Format der Verbindungszeichenfolge verwenden.

* Format der ursprünglichen Verbindungszeichenfolge: <*server name*>.database.windows.net
* Verbindungszeichenfolge mit aktivierter Sicherheit: <*server name*>.database.**secure**.windows.net

Sie können auch die Einstellung **ZUGRIFF MIT AKTIVIERTER SICHERHEIT** von **OPTIONAL** in **ERFORDERLICH** ändern, was sicherstellt, dass es keine Möglichkeit gibt, auf die Datenbank mit der ursprünglichen Verbindungszeichenfolge zuzugreifen und die Richtlinie für die dynamische Datenmaskierung zu ignorieren. Wenn Sie mit der dynamischen Datenmaskierung unter Verwendung bestimmter Clients (z. B. einer Anwendung in der Entwicklungsphase oder SSMS) experimentieren, wählen Sie **OPTIONAL**. Wählen Sie für Produktionsumgebungen **ERFORDERLICH**.<br/><br/>

![Navigationsbereich][Image3]<br/><br/>

## Einrichten der dynamischen Datenmaskierung für Ihre Datenbank im Azure-Portal

1. Starten Sie das Azure-Portal unter [https://manage.windowsazure.com](https://manage.windowsazure.com).

2. Klicken Sie auf die zu maskierende Datenbank und dann auf die Registerkarte **ÜBERWACHUNG UND SICHERHEIT**.

3. Klicken Sie unter **Dynamische Datenmaskierung** auf **AKTIVIERT**, um diese Funktion zu aktivieren.  

4. Geben Sie die privilegierten Anmeldungen ein, die Zugriff auf sensible Daten ohne Maskierung haben sollen.

	>[AZURE.TIPP] Damit die Anwendungsschicht sensible Daten für die privilegierten Anwendungsbenutzer anzeigen kann, fügen Sie die Anwendungsanmeldung, die zum Abfragen der Datenbank verwendet wird, und die Datenbank hinzu. Es wird ausdrücklich empfohlen, dass diese Liste nur eine minimale Anzahl von Anmeldungen enthält, um die Anzeige sensibler Daten zu minimieren.

	![Navigationsbereich][Image4]

5. Klicken Sie unten auf der Seite auf der Menüleiste auf **MASKE HINZUFÜGEN**, um das Konfigurationsfenster für die Maskierungsregel zu öffnen.

6. Wählen Sie **Maskieren nach**, um anzugeben, ob die Maskierung für die Quelle oder das Ziel erfolgen soll. Die Maskierung kann auf Quellebene konfiguriert werden, indem der **Tabellenname** und **Spaltenname** bestimmt werden, oder auf Ergebnisebene, indem der in der Abfrage verwendete **Alias** bestimmt wird. Wenn Sie mit der Datenarchitektur Ihrer Datenbank vertraut sind und die Anzeige aller Abfrageergebnisse begrenzen möchten, bevorzugen Sie ggf. Sie eine Quellmaskenregel. Sie können eine Ergebnismaskenregel hinzufügen, wenn Sie die Anzeige auf Abfrageergebnisse beschränken möchten, ohne die Datenarchitektur der Datenbank zu analysieren, oder auf ein Feld, das aus verschiedenen Quellen stammen kann.

7. Geben Sie den **Tabellennamen** und **Spaltennamen** bzw. den **Aliasnamen** ein, um die Felder zu bestimmen, die maskiert werden sollen.

8. Wählen Sie in der Liste der Kategorien für das Maskieren vertraulicher Daten eine **MASKIERUNGSFUNKTION**.

	![Navigationsbereich][Image5] 
 	
9. Klicken Sie im Fenster der Datenmaskierungsregel auf **Aktualisieren**, um die Gruppe von Maskierungsregeln in der Richtlinie für die dynamische Maskierung zu aktualisieren.

10. Erwägen Sie das Aktivieren von **ERWEITERTE BESCHRÄNKUNGEN VERWENDEN**, wodurch die Anzeige vertraulicher Daten mittels Ad-hoc-Abfragen begrenzt wird.

11. Klicken Sie auf **Speichern**, um die neue oder aktualisierte Maskierungsregel zu speichern.

## Einrichten der dynamischen Datenmaskierung für Ihre Datenbank mithilfe der REST-API

Siehe [Vorgänge für Azure SQL-Datenbanken](https://msdn.microsoft.com/library/dn505719.aspx).

[Image1]: ./media/sql-database-dynamic-data-masking-get-started-portal/1_DDM_Random_number.png
[Image2]: ./media/sql-database-dynamic-data-masking-get-started-portal/2_DDM_Custom_text.png
[Image3]: ./media/sql-database-dynamic-data-masking-get-started-portal/3_DDM_Current_Preview.png
[Image4]: ./media/sql-database-dynamic-data-masking-get-started-portal/4_DMM_Policy_Classic_Portal.png
[Image5]: ./media/sql-database-dynamic-data-masking-get-started-portal/5_DDM_Add_Masking_Rule_Classic_Portal.png

<!--HONumber=49--> 