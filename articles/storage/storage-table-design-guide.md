<properties 
   pageTitle="Azure-Tabellenspeicher - Handbuch für den Entwurf | Microsoft Azure" 
   description="Entwurf skalierbarer und leistungsfähiger Tabellen in Azure-Tabellenspeicher" 
   services="storage" 
   documentationCenter="na" 
   authors="Tamram,jahogg" 
   manager="adinah" 
   editor=""/>

<tags
   ms.service="storage"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="storage" 
   ms.date="02/26/2015"
   ms.author="tamram"/>

# Azure-Tabellenspeicher - Handbuch für den Entwurf: Entwurf skalierbarer und leistungsfähiger Tabellen

## Übersicht

Beim Entwurf von skalierbaren und leistungsfähigen Tabellen müssen Sie eine Reihe von Faktoren beachten, beispielsweise Leistung, Skalierbarkeit und Kosten. Wenn Sie früher schon Schemas für relationale Datenbanken entworfen haben, werden Sie mit diesen Überlegungen vertraut sein. Aber trotz einigen Ähnlichkeiten zwischen dem Modell des Azure-Tabellenspeicherdienstes und relationalen Modellen gibt es auch viele wichtige Unterschiede. Diese Unterschiede führen in der Regel zu sehr unterschiedlichen Entwürfen, die einer mit relationalen Datenbanken vertrauten Person kontraproduktiv oder falsch erscheinen, aber sinnvoll sind, wenn Sie einen Entwurf für einen NoSQL-Schlüssel-Wert-Speicher wie z. B. den Azure-Tabellenspeicherdienst durchführen. Viele der Unterschiede zu Ihrem Entwurf spiegeln die Tatsache wider, dass der Tabellenspeicherdienst zur Unterstützung von Cloud-Anwendungen konzipiert ist, die Milliarden von Entitäten (Zeilen in der Terminologie für relationale Datenbanken) aus Daten oder Datensätzen beinhalten und die sehr hohe Transaktionsvolumen unterstützen müssen. Aus diesem Grund müssen Sie andere Überlegungen anstellen, wie Sie Ihre Daten speichern und verstehen, wie der Tabellenspeicherdienst funktioniert. Ein gut konzipierter NoSQL-Datenspeicher kann bei Ihrer Lösung eine viel tiefere Skalierung ermöglichen (und zu geringeren Kosten), als dies bei einer Lösung möglich ist, die eine relationale Datenbank verwendet. Dieses Handbuch unterstützt Sie bei folgenden Themen.  

## Informationen zum Azure-Tabellenspeicherdienst

In diesem Abschnitt werden einige der wichtigsten Funktionen des Tabellenspeicherdienstes beleuchtet, die für den Entwurf mit Leistung und Skalierbarkeit besonders relevant sind. Falls Sie keine Erfahrung mit Azure Storage und dem Tabellenspeicherdienst haben, lesen Sie zuerst die Informationen unter [Einführung in Microsoft Azure Storage](storage-introduction.md) und [Verwenden des Tabellenspeichers mit .NET](storage-dotnet-how-to-use-tables.md) , bevor Sie mit diesem Artikel fortfahren. Obwohl der Schwerpunkt dieses Handbuchs auf dem Tabellenspeicherdienst liegt, werden auch Azure-Warteschlange und Blob-Dienste erörtert und wie Sie diese zusammen mit dem Tabellenspeicherdienst in einer Lösung verwenden.  
 
Was ist der Tabellenspeicherdienst? Wie aus dem Namen zu ersehen ist, verwendet der Tabellenspeicherdienst ein tabellarisches Format zum Speichern von Daten. In der Standard-Terminologie stellt jede Zeile der Tabelle eine Entität dar und die Spalten speichern die verschiedenen Eigenschaften dieser Entität. Jede Entität besitzt ein Schlüsselpaar zur eindeutigen Identifizierung und eine Zeitstempel-Spalte, die vom Tabellenspeicherdienst verwendet wird, um nachzuverfolgen, wann die Entität zuletzt aktualisiert wurde (Dies geschieht automatisch und Sie können den Zeitstempel nicht manuell mit einem beliebigen Wert überschreiben). Der Tabellenspeicherdienst verwendet diesen Zeitstempel der letzten Änderung (LMT) zum Verwalten der optimistischen Nebenläufigkeit.  

>[AZURE.NOTE] Die REST-API-Vorgänge des Tabellenspeicherdienstes geben auch einen **ETag**-Wert zurück, den sie von der LMT abgeleitet haben. In diesem Dokument werden wir die Begriffe ETag und LMT austauschbar verwenden, da sie auf dieselben zugrunde liegenden Daten verweisen.  

Das folgende Beispiel zeigt einen einfachen Tabellenentwurf zum Speichern von Mitarbeiter- und Abteilungsentitäten. Viele der in diesem Handbuch weiter unten gezeigten Beispiele basieren auf diesem einfachen Entwurf.  
 
<table>
<tr>
<th>Partitionsschlüssel</th>
<th>Zeilenschlüssel</th>
<th>Timestamp</th>
<th></th>
</tr>
<tr>
<td>Marketing</td>
<td>00001</td>
<td>2014-08-22T00:50:32Z</td>
<td>
<table>
<tr>
<th>Vorname</th>
<th>Nachname</th>
<th>Alter</th>
<th>E-Mail</th>
</tr>
<tr>
<td>Don</td>
<td>Hall</td>
<td>34</td>
<td>donh@contoso.com</td>
</tr>
</table>
</tr>
<tr>
<td>Marketing</td>
<td>00002</td>
<td>2014-08-22T00:50:34Z</td>
<td>
<table>
<tr>
<th>Vorname</th>
<th>Nachname</th>
<th>Alter</th>
<th>E-Mail</th>
</tr>
<tr>
<td>Jun</td>
<td>Cao</td>
<td>47</td>
<td>junc@contoso.com</td>
</tr>
</table>
</tr>
<tr>
<td>Marketing</td>
<td>Abteilung</td>
<td>2014-08-22T00:50:30Z</td>
<td>
<table>
<tr>
<th>Abteilungsname</th>
<th>EmployeeCount</th>
</tr>
<tr>
<td>Marketing</td>
<td>153</td>
</tr>
</table>
</td>
</tr>
<tr>
<td>Vertriebsabteilung</td>
<td>00010</td>
<td>2014-08-22T00:50:44Z</td>
<td>
<table>
<tr>
<th>Vorname</th>
<th>Nachname</th>
<th>Alter</th>
<th>E-Mail</th>
</tr>
<tr>
<td>Ken</td>
<td>Kwok</td>
<td>23</td>
<td>kenk@contoso.com</td>
</tr>
</table>
</td>
</tr>
</table>


Bisher gleicht dies einer Tabelle in einer relationalen Datenbank, wobei die wesentlichen Unterschiede in den Pflichtspalten und in der Fähigkeit liegen, verschiedene Entitätstypen in derselben Tabelle zu speichern. Darüber hinaus verfügen alle benutzerdefinierten Eigenschaften wie **Vorname** oder **Alter** über einen Datentyp, z. B. Ganzzahl oder Zeichenfolge, genauso wie für eine Spalte in einer relationalen Datenbank. Obwohl dies bei einer relationalen Datenbank unwahrscheinlich ist, bedeutet die Art der Tabelle ohne Schema, dass eine Eigenschaft nicht denselben Datentyp bei jeder Entität haben muss. Um komplexe Datentypen in einer einzelnen Eigenschaft zu speichern, müssen Sie ein serialisiertes Format wie z. B. JSON oder XML verwenden. Weitere Informationen zum Tabellenspeicherdienst wie z. B. unterstützte Datentypen, unterstützte Datumsbereiche, Namensregeln und Größenbeschränkungen finden Sie unter [Verständnis des Tabellenspeicherdienst-Datenmodells](http://msdn.microsoft.com/library/azure/dd179338.aspx) auf MSDN.

Wie Sie sehen werden, ist Ihre Auswahl bei **PartitionKey** und **RowKey** die Grundlage für einen guten Tabellenentwurf. Jede in einer Tabelle gespeicherte Entität muss eine eindeutige Kombination aus **PartitionKey** und **RowKey** besitzen. Wie bei Schlüssel in einer relationalen Datenbanktabelle sind die Werte von **PartitionKey** und **RowKey** indiziert, um einen gruppierten Index zu erstellen, der schnelles Suchen ermöglicht. Allerdings erstellt der Tabellenspeicherdienst keine sekundären Indexe. Deshalb sind dies die einzigen zwei indizierten Eigenschaften (einige der später beschriebenen Muster zeigen, wie Sie diese scheinbare Einschränkung umgehen können).  

Eine Tabelle besteht aus einer oder mehreren Partitionen. Sie werden feststellen, dass viele der von Ihnen zu treffenden Entwurfsentscheidungen mit der Auswahl eines geeigneten **PartitionKey** und **RowKey** zu tun haben, um Ihre Lösung zu optimieren. Eine Lösung könnte aus einer einzelnen Tabelle bestehen, die alle Entitäten in Partitionen unterteilt enthält, aber in der Regel wird eine Lösung aus mehrere Tabellen bestehen. Tabellen helfen Ihnen bei der logischen Organisation Ihrer Entitäten, bei der Verwaltung des Zugriffs auf die Daten mithilfe von Zugriffssteuerungslisten und Sie können eine komplette Tabelle mit einem einzelnen Speichervorgang ablegen.  

### Tabellenpartitionen  
Kontoname, Tabellenname und **PartitionKey** bestimmen zusammen die Partition innerhalb des Tabellenspeicherdienstes, in dem der Tabellenspeicherdienst die Entität speichert. Partitionen sind Teil des Adressierungsschemas für Entitäten, definieren zusätzlich einen Bereich für Transaktionen (siehe [Entitätsgruppentransaktionen](#entity-group-transactions) unten) und bilden die Grundlage für die Skalierung des Tabellenspeicherdienstes. Weitere Informationen zu Partitionen finden Sie unter [Ziele für Skalierbarkeit und Leistung des Azure-Speichers](http://msdn.microsoft.com/library/azure/dn249410.aspx).  

Im Tabellenspeicherdienst bedient ein einzelner Knoten eine oder mehrere komplette Partitionen und skaliert durch dynamischen Lastenausgleich Partitionen über Knoten hinweg. Wenn ein Knoten ausgelastet ist, kann der Tabellenspeicherdienst den Partitionsbereich  *teilen*, der von diesem Knoten aus andere Knoten bedient. Wenn der Datenverkehr abnimmt, kann der Dienst die Partitionsbereiche von ruhigen Knoten zurück in einem einzelnen Knoten  *zusammenführen*.  

Weitere Informationen über die internen Details des Tabellenspeicherdienstes und insbesondere zur Verwaltung der Partitionen durch den Dienst finden Sie im Dokument [Microsoft Azure Storage: Ein hoch verfügbarer
Cloud-Speicherdienst mit starker Konsistenz](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx).  

### Entitätsgruppentransaktionen 
Im Tabellenspeicherdienst sind Entitätsgruppentransaktionen (EGTs) der einzige integrierte Mechanismus, mit dem atomische Aktualisierungen für mehrere Entitäten durchgeführt werden können. EGTs werden in einigen Dokumenten auch als *Batchtransaktionen* bezeichnet. EGTs können nur mit Entitäten betrieben werden, die in der gleichen Partition gespeichert sind (Freigabe desselben Partitionsschlüssels in einer bestimmten Tabelle). Deshalb müssen Sie jedes Mal, wenn Sie eine atomisches Transaktionsverhalten über mehrere Entitäten benötigen, sicherstellen, dass sich die Entitäten in derselben Partition befinden. Dies ist häufig der Grund, dass mehrere Entitätstypen in derselben Tabelle (und Partition) gehalten werden und nicht mehrere Tabellen für verschiedene Entitätstypen verwendet werden. Eine einzelne EGT kann mit höchstens 100 Entitäten verwendet werden.  

EGTs führen auch zu einem potenziellen Kompromiss, den Sie in Ihrem Entwurf bewerten müssen. Die Verwendung von mehreren Partitionen erhöht die Skalierbarkeit der Anwendung, da Azure über mehr Möglichkeiten für den Lastenausgleich zwischen Knoten verfügt. Dies kann jedoch die Fähigkeit Ihrer Anwendung beeinträchtigen, atomische Transaktionen auszuführen und die starke Konsistenz Ihrer Daten beizubehalten. Darüber hinaus gibt es bestimmte Skalierbarkeitsziele auf der Ebene einer Partition, die den Durchsatz von Transaktionen einschränken können, der für einen einzelnen Knoten zu erwarten ist. Weitere Informationen über die Skalierbarkeitsziele für Azure Storage-Konten und den Tabellenspeicherdienst finden Sie unter [Ziele für Skalierbarkeit und Leistung des Azure-Speichers](http://msdn.microsoft.com/library/azure/dd179338.aspx) auf MSDN. In späteren Abschnittes dieses Handbuchs werden verschiedene Entwurfsstrategien besprochen, die Sie im Umgang mit Kompromissen wie diesen unterstützen. Es wird auch besprochen, wie Sie, basierend auf den besonderen Anforderungen Ihrer Clientanwendung, die beste Wahl für Ihren Partitionsschlüssel treffen.  

### Überlegungen zur Kapazität
Die folgende Tabelle enthält einige Schlüsselwerte, auf die Sie achten müssen, wenn Sie eine Lösung für einen Tabellenspeicherdienst entwerfen:  

|Gesamtkapazität eines Azure Storage-Kontos|500 TB|
|------------------------------------------|------|
|Anzahl von Tabellen in einem Azure Storage-Konto | Begrenzung nur durch die Kapazität des Speicherkontos |
|Anzahl von Partitionen in einer Tabelle | Begrenzung nur durch die Kapazität des Speicherkontos |
|Anzahl von Entitäten in einer Partition | Begrenzung nur durch die Kapazität des Speicherkontos|
|Größe einer einzelnen Entität | Bis zu 1 MB mit maximal 255 Eigenschaften (einschließlich **PartitionKey**, **RowKey** und **Timestamp**) |
|Größe von **PartitionKey** | Eine Zeichenfolge mit bis zu 1 KB |
| Größe von **RowKey** | Eine Zeichenfolge mit bis zu 1 KB |
|Größe einer Entitätsgruppentransaktion | Eine Transaktion kann höchstens 100 Entitäten umfassen, und die Nutzlast muss weniger als 4 MB groß sein. Eine EGT kann eine Entität nur einmal aktualisieren. |

Weitere Informationen finden Sie unter [Grundlegendes zum Tabellenspeicherdienst-Datenmodell](http://msdn.microsoft.com/library/azure/dd179338.aspx) auf MSDN.  

### Kostenbetrachtung  
Tabellenspeicher ist relativ günstig, aber Sie sollten die Kostenschätzungen für Kapazitätsauslastung und Transaktionsmenge als Bestandteil Ihrer Auswertung bei allen Lösungen mit aufnehmen, die den Tabellenspeicherdienst verwenden. In vielen Szenarien ist jedoch die Speicherung denormalisierter oder doppelter Daten zur Verbesserung der Leistung oder der Skalierbarkeit für Ihre Lösung ein zulässiger Ansatz. Weitere Informationen zu den Preisen finden Sie unter [Preisübersicht für Speicher](http://azure.microsoft.com/pricing/details/storage/).  

### Vergleich von Azure-Tabellen und SQL Azure  
Einen Vergleich zwischen Azure SQL-Datenbank (einem relationalen Datenbankdienst) und dem Tabellenspeicherdienst finden Sie unter [Azure-Tabellenspeicher und Azure SQL-Datenbank - Vergleich und Gegenüberstellung](http://msdn.microsoft.com/library/azure/jj553018.aspx) auf MSDN.  

## Richtlinien für den Entwurf von Tabellen  
Diese Listen fassen einige der wichtigsten Richtlinien zusammen, die Sie beim Entwurf von Tabellen im Auge behalten sollten und die in diesem Handbuch später im Detail behandelt werden. Diese Richtlinien unterscheiden sich wesentlich von den Richtlinien, die für den Entwurf relationaler Datenbanken üblich sind.  

So entwerfen Sie Ihre Tabellenspeicherdienstlösung für größtmögliche Effizienz von *Lesevorgängen*:

-	***Entwerfen Sie Tabellen für Abfragen in Anwendungen mit intensiven Lesevorgängen.*** Denken Sie beim Entwurf Ihrer Tabellen an die Abfragen (insbesondere an diejenigen, bei denen die Latenz von großer Bedeutung ist), die Sie ausführen werden, bevor Sie überlegen, wie Sie Ihre Entitäten aktualisieren werden. Dies führt in der Regel zu einer effizienten und leistungsstarken Lösung.
-	***Geben Sie sowohl "PartitionKey" als auch "RowKey" in Ihren Abfragen ein.*** *Punktabfragen* wie diese stellen die effizientesten Tabellenspeicherdienstabfragen dar.  
-	***Erwägen Sie die Speicherung duplizierter Kopien von Entitäten.*** Tabellenspeicher ist günstig. Erwägen Sie daher, ein und dieselbe Entität mehrfach zu speichern (mit verschiedenen Schlüsseln), um effizientere Abfragen zu ermöglichen.  
-	***Erwägen Sie die Denormalisierung Ihrer Daten.*** Tabellenspeicher ist günstig. Erwägen Sie daher, Ihre Daten zu denormalisieren. Speichern Sie z. B. zusammengefasste Entitäten, damit Abfragen für zusammengefasste Daten nur auf eine einzelne Entität zugreifen müssen.
-	***Verwenden Sie zusammengesetzte Schlüsselwerte.*** Sie verfügen nur über die Schlüssel **PartitionKey** und **RowKey**. Verwenden Sie z. B. zusammengesetzte Schlüsselwerte, um Zugriffspfade mit alternativen Schlüsseln für Entitäten zu ermöglichen.
-	***Verwenden Sie die Abfrageprojektion.*** Sie können die Datenmenge reduzieren, die Sie über das Netzwerk übertragen, indem Sie Abfragen verwenden, die nur die benötigten Felder auswählen.

So entwerfen Sie Ihre Tabellenspeicherdienstlösung für größtmögliche Effizienz von *Schreibvorgängen*:  

-	***Erstellen Sie keine Hot-Partitionen.*** Wählen Sie Schlüssel aus, die es Ihnen ermöglichen, Ihre Anforderungen zu einem bestimmten Zeitpunkt auf mehrere Partitionen zu verteilen.
-	***Vermeiden Sie Lastspitzen.*** Sorgen Sie für einen reibungslosen Datenverkehr über einen angemessenen Zeitraum und vermeiden Sie Lastspitzen.
-	***Erstellen Sie nicht unbedingt eine separate Tabelle für jeden Entitätstyp.*** Wenn atomische Transaktionen über Entitätstypen hinweg erforderlich sind, können Sie diese verschiedenen Entitätstypen in derselben Tabelle in derselben Partition speichern.
-	***Bedenken Sie den maximalen Durchsatz, der erzielt werden muss.*** Sie müssen die Skalierbarkeitsziele für den Tabellenspeicherdienst berücksichtigen und sicherstellen, dass diese durch Ihren Entwurf nicht überschritten werden.  

Beim Durchlesen dieses Handbuch finden Sie Beispiele für die Umsetzung dieser Grundlagen in die Praxis.  

## Entwurf für Abfragen  
Anwendungen für einen Tabellenspeicherdienst können intensiv lesen, intensiv schreiben oder beides. Dieser Abschnitt konzentriert sich auf die Dinge, die Sie beim Entwurf Ihrer Tabellenspeicherdienstes beachten sollten, um Lesevorgänge effizient zu unterstützen. In der Regel ist ein Entwurf, der Lesevorgänge effizient unterstützt, auch für Schreibvorgänge effizient. Es gibt jedoch weitere Gesichtspunkte, die beim Entwurf von Schreibvorgängen zu beachten sind und die im folgenden Abschnitt, [Entwurf für die Datenänderung](#design-for-data-modification), erläutert werden.

Ein guter Ausgangspunkt für den Entwurf Ihrer Lösung für einen Tabellenspeicherdienst mit effizientem Lesen der Daten ist, sich zu fragen: "Welche Abfragen meiner Anwendung müssen ausgeführt werden, um die benötigten Daten abzurufen, die vom Tabellenspeicherdienst benötigt werden?"  

>[AZURE.NOTE] Für einen Tabellenspeicherdienst ist es wichtig, den Entwurf von Anfang an richtig zu machen, da Änderungen zu einem späteren Zeitpunkt schwierig und teuer sind. Beispielsweise ist es bei einer relationalen Datenbank oft möglich, Leistungsprobleme in Angriff zu nehmen, indem Sie einfach einer vorhandenen Datenbank Indexe hinzufügen. Diese Möglichkeit steht bei einem Tabellenspeicherdienst nicht zur Verfügung.  

Dieser Abschnitt konzentriert sich auf die Hauptprobleme, mit denen Sie sich beim Entwurf der Tabellen für die Abfrage befassen müssen. Die in diesem Abschnitt behandelten Themen umfassen:  
-	[Auswirkungen der Wahl von PartitionKey und RowKey auf die Abfrageleistung](#how-your-choice-of-partitionkey-and-rowkey-impacts-query-performance)
-	[Auswahl eines geeigneten PartitionKey](#choosing-an-appropriate-partitionkey)
-	[Optimierung von Abfragen mit einem Schlüssel-Wert-Speicher für den Tabellenspeicherdienst](#optimizing-queries-with-a-key-value-store-for-the-table-service)
-	[Sortieren von Daten in einem Schlüssel-Wert-Speicher im Tabellenspeicherdienst](#sorting-data-in-a-key-value-store-in-the-table-service)

### Auswirkungen der Wahl von PartitionKey und RowKey auf die Abfrageleistung  

In den folgenden Beispielen wird angenommen, dass der Tabellenspeicherdienst Entitäten von Mitarbeitern mit der folgenden Struktur speichert (bei die meisten Beispielen fehlt die Eigenschaft **Zeitstempel** aus Gründen der Übersichtlichkeit):  

|*Spaltenname* |*Datentyp*|
|--------------|-----------|
|**PartitionKey** (Abteilungsname)|String|
|**RowKey** (Mitarbeiter-ID)|String|
|**FirstName**|String|
|**LastName**|String|
|**Age**|Ganze Zahl|
|**EmailAddress**|String|

Im Abschnitt [Übersicht über den Azure-Tabellenspeicherdienst](#azure-table-service-overview) weiter oben werden einige der wichtigsten Funktionen des Azure-Tabellenspeicherdienstes beschrieben, die direkten Einfluss auf den Entwurf für Abfragen haben. Dadurch ergeben sich die folgenden allgemeinen Richtlinien für den Entwurf von Abfragen für den Tabellenspeicherdienst. Beachten Sie, dass die in den Beispielen unten verwendete Filtersyntax aus dem REST-API-Tabellenspeicherdienst stammt. Weitere Informationen finden Sie unter [Abfragen von Entitäten](http://msdn.microsoft.com/library/azure/dd179421.aspx) auf MSDN.  

-	Eine ***Punktabfrage*** ist die effizienteste Suche und wird bei sehr umfangreichen Suchvorgängen oder für Suchvorgänge empfohlen, die eine sehr niedrige Latenz erfordern. Eine solche Abfrage kann Indizes verwenden, um durch die Angabe der **PartitionKey** und **RowKey**-Werte eine einzelne Entität sehr effizient zu suchen. Beispiel:
$filter=(PartitionKey eq 'Sales') und (RowKey eq '2')  
-	Die zweitbeste Lösung ist eine ***Bereichsabfrage***, die den **PartitionKey** verwendet und einen Bereich von **RowKey**-Werten filtert, um mehr als eine Entität zurückzugeben. Der **PartitionKey**-Wert identifiziert eine bestimmte Partition und die **RowKey**-Werte identifizieren eine Teilmenge der Entitäten in dieser Partition. Beispiel:
$filter=PartitionKey eq 'Sales' und RowKey ge 'S' und RowKey lt 'T'  
-	Die drittbeste Lösung ist ein ***Partitions-Scan***, der den **PartitionKey** sowie Filter auf einer anderen Eigenschaft ohne Schlüssel verwendet und möglicherweise mehr als eine Entität zurückgibt. Der **PartitionKey**-Wert identifiziert eine bestimmte Partition und die Eigenschaftswerte wählen eine Teilmenge der Entitäten in dieser Partition aus. Beispiel:
$filter=PartitionKey eq 'Sales' und LastName eq 'Smith'  
-	Ein ***Tabellen-Scan*** umfasst den **PartitionKey** nicht und ist sehr ineffizient, da er alle Partitionen, aus denen Ihre Tabelle besteht, auf übereinstimmende Entitäten untersucht. Er führt einen Tabellen-Scan durch, unabhängig davon, ob der Filter **RowKey** verwendet wird. Beispiel:
$filter=LastName eq 'Jones'  
-	Abfragen, die mehrere Entitäten zurückgeben, geben diese sortiert in der Reihenfolge **PartitionKey** und **RowKey** zurück. Um eine Neusortierung der Entitäten im Client zu vermeiden, müssen Sie einen **RowKey** mit der am häufigsten verwendeten Sortierreihenfolge auswählen.  

Beachten Sie, dass die Verwendung von "**oder**" für die Festlegung eines Filters, der auf **RowKey**-Werten basiert, zu einem Partition-Scan führt und nicht als eine Bereichsabfrage behandelt wird. Aus diesem Grund sollten Sie Abfragen vermeiden, die z. B. folgende Filter verwenden:
$filter=PartitionKey eq 'Sales' und (RowKey eq '121' oder RowKey eq '322')  

Beispiele für clientseitigen Code, der die Storage Client Library zur Ausführung effizienter Abfragen verwendet, finden Sie unter:  

-	[Eine einzelne Entität mithilfe der Storage Client Library abrufen](#retrieving-a-single-entity-using-the-storage-client-library)
-	[Abrufen von mehreren Entitäten mithilfe von LINQ](#retrieving-multiple-entities-using-linq)
-	[Serverseitige Projektion](#server-side-projection)  

Beispiele für clientseitigen Code, der mehrere Entitätstypen handhaben kann, die in der gleichen Tabelle gespeichert sind, finden Sie unter:  

-	[Arbeiten mit heterogenen Entitätstypen](#working-with-heterogeneous-entity-types)  

### Auswählen eines geeigneten PartitionKey  

Bei der Auswahl des **PartitionKey** sollte ein Abgleich zwischen der Notwendigkeit, die Verwendung von EGTs freizugeben (zur Sicherstellung der Konsistenz) und der Anforderung zur Verteilung von Entitäten auf mehrere Partitionen (um eine skalierbare Lösung sicherzustellen) vorgenommen werden.  

Ein Extremfall ist, dass Sie alle Entitäten in einer einzelnen Partition speichern würden. Dies würde aber möglicherweise die Skalierbarkeit Ihrer Lösung begrenzen und verhindern, dass der Tabellenspeicherdienst Anforderungen für den Lastenausgleich vornehmen kann. Im anderen Extremfall könnten Sie zum einen eine Entität pro Partition speichern, die skalierbar wäre, und zum anderen den Tabellenspeicherdienst für Lastenausgleich-Anforderungen. Dies würde aber die Verwendung von EGTs verhindern.  

Ein idealer **PartitionKey** ermöglicht die Verwendung von effizienten Abfragen und verfügt über ausreichende Partitionen, um sicherzustellen, dass Ihre Lösung skalierbar ist. In der Regel werden Sie feststellen, dass Ihre Entitäten über eine geeignete Eigenschaft verfügen, die Ihre Entitäten über ausreichende Partitionen verteilt.  

>[AZURE.NOTE]Beispielsweise kann in einem System, auf dem Informationen über Benutzer und Mitarbeiter gespeichert werden, die Benutzer-ID eine guter **PartitionKey** sein.  

Bei der Auswahl des **PartitionKey** sind weitere Aspekte zu bedenken, die sich darauf beziehen, wie Sie Entitäten einfügen, aktualisieren und löschen. Siehe Abschnitt [Design für die Datenänderung](#design-for-data-modification) weiter unten.  

### Optimieren von Abfragen für den Tabellenspeicherdienst  

Der Tabellenspeicherdienst indiziert die Entitäten automatisch unter Verwendung von **PartitionKey**- und **RowKey**-Werten in einen einzelnen gruppierten Index. Aus diesem Grund stellen Punktabfragen die effizienteste Verwendung dar. Allerdings gibt es mit Ausnahme des gruppierten Indexes für **PartitionKey** und **RowKey** keine weiteren Indexe.   

Viele Entwürfe müssen Anforderungen erfüllen, um die Suche nach Entitäten auf Grundlage mehrerer Kriterien zu ermöglichen. Suchen Sie z. B. Mitarbeiterentitäten auf Grundlage der E-Mail-Adresse, Mitarbeiter-ID oder Nachname. Die folgenden Muster im Abschnitt [Entwurfsmuster für die Tabelle](#table-design-patterns) befassen sich mit diesen Arten von Anforderungen und beschreiben Umgehungen für den Umstand, dass der Tabellenspeicherdienst keine sekundären Indizes bietet:  

-	[Sekundäres Indexmuster für Intra-Partition](#intra-partition-secondary-index-pattern) - Speichern mehrerer Kopien jeder Entität mit unterschiedlichen **RowKey**-Werten (in derselben Partition) zur Aktivierung schneller und effizienter Suchvorgänge und alternativer Sortierreihenfolgen mit unterschiedlichen **RowKey**-Werten.  
-	[Sekundäres Indexmuster für Inter-Partition](#inter-partition-secondary-index-pattern) - Speichern mehrerer Kopien der einzelnen Entitäten mithilfe verschiedener RowKey-Werte in separaten Partitionen oder in separaten Tabellen, um schnelle und effiziente Suchvorgänge und alternativen Sortierreihenfolgen mit anderen **RowKey** Werte zu ermöglichen.  
-	[Indexmuster für Entitäten](#index-entities-pattern) - Verwalten von Index-Entitäten, um effiziente Suchvorgänge zu ermöglichen, die Entitäten-Listen zurückgeben.  

### Sortieren von Daten im Tabellenspeicherdienst  

Der Tabellenspeicherdienst gibt Entitäten zurück, die in aufsteigender Reihenfolge sortiert sind, die auf **PartitionKey** und anschließend auf **RowKey** basieren. Diese Schlüssel sind String-Werte. Um sicherzustellen, dass die numerischen Werte ordnungsgemäß sortieren, sollten sie mit einer festen Länge konvertiert und mit Nullen aufgefüllt werden. Wenn z. B. der Mitarbeiter-ID-Wert, der als **RowKey** verwenden wird, ein ganzzahliger Wert ist, sollten Sie die Mitarbeiter-ID **123** auf **00000123** konvertieren.  

Viele Anwendungen verfügen über Anforderungen für die Verwendung von Daten, die in unterschiedlicher Reihenfolge sortiert sind,z. B. bei Sortierung von Mitarbeitern nach Name oder durch das Verknüpfen eines Datums. Die folgenden Muster im Abschnitt [Entwurfsmuster für die Tabelle](#table-design-patterns) zeogem, wie alternative Sortierreihenfolgen für die Entitäten erstellt werden:  

-	[Sekundäres Indexmuster für Intra-Partition](#intra-partition-secondary-index-pattern) -Speichern mehrerer Kopien der einzelnen Entitäten mit verschiedenen RowKey-Werten (in der gleichen Partition), um schnelle und effiziente Suchvorgänge und alternative Sortierreihenfolgen unter Verwendung verschiedener RowKey-Werte zu ermöglichen.  
-	[Sekundäres Indexmuster für Inter-Partition](#inter-partition-secondary-index-pattern) - Speichern mehrerer Kopien der einzelnen Entitäten mit verschiedenen RowKey-Werte in separaten Partitionen in separaten Tabellen für schnelle und effiziente Suchvorgänge und alternative Sortierreihenfolgen mit verschiedenen RowKey-Werten. 
-	[Log Tail-Muster](#log-tail-pattern) - Abrufen der *n* Entitäten, die zuletzt einer Partition hinzugefügt wurden, indem Sie einen **RowKey**-Wert verwenden, mit dem nach Datum und Zeit in umgekehrter Reihenfolge sortiert wird.  

## Entwurf für die Datenänderung
Dieser Abschnitt konzentriert sich auf Gesichtspunkte zum Entwurf für die Optimierung von Einfügungen, Aktualisierungen und Löschungen. In einigen Fällen müssen Sie einen Kompromiss finden zwischen Entwürfen, die nach Abfragen optimiert sind und Entwürfen, die nach Datenänderung optimiert sind, so wie sie dies auch bei Entwürfen für relationale Datenbanken vornehmen (obwohl die Techniken für die Verwaltung von Entwurfskompromissen für eine relationale Datenbank unterschiedlich sind). Der Abschnitt [Entwurfsmuster für die Tabelle](#table-design-patterns) beschreibt einige detaillierte Entwurfsmuster für den Tabellenspeicherdienst und stellt einige dieser Kompromisse heraus. In der Praxis werden Sie feststellen, dass viele für Abfragen von Entitäten optimierte Entwürfe sich auch zum Ändern von Entitäten eignen.  
 
### Optimieren der Leistung der Vorgänge Einfügen, Aktualisieren und Löschen  

Zum Aktualisieren oder Löschen einer Entität müssen Sie in der Lage sein, diese durch Verwendung der **PartitionKey**- und **RowKey**-Werte zu identifizieren. In diesem Zusammenhang sollte die Wahl von **PartitionKey** und **RowKey** für das Ändern von Entitäten ähnlichen Kriterien bei Ihrer Wahl für Punktabfragen folgen, da Sie Entitäten so effizient wie möglich identifizieren möchten. Sie möchten bestimmt keinen ineffiziente Partitions- oder Tabellen-Scan verwenden, um eine Entität zu ermitteln, um damit die **PartitionKey**- und **RowKey**-Werte zu finden, die Sie aktualisieren oder löschen müssen.  

Die folgenden Muster im Abschnitt [Entwurfsmuster für die Tabelle](#table-design-patterns) befassen sich mit der Optimierung der Leistung oder Einfüge-, Aktualisierungs- und Löschvorgängen:  

-	[Muster für das Löschen hoher Volumen](#high-volume-delete-pattern) - Aktivieren Sie das Löschen einer großen Anzahl von Entitäten, indem Sie alle Entitäten zum gleichzeitigen Löschen in deren eigenen separaten Tabelle speichern. Die Entitäten werden durch Löschen der Tabelle gelöscht.  
-	[Datenreihenmuster](#data-series-pattern) - Speichern Sie vollständige Datenreihen in einer einzelnen Entität, um die Anzahl der Anforderungen zu minimieren, die Sie vornehmen.  
-	[Breite Entitätenmuster](#wide-entities-pattern) - Verwenden Sie mehrere physische Entitäten zum Speichern von logischen Entitäten mit mehr als 252 Eigenschaften.  
-	[Muster für große Entitäten](#large-entities-pattern) - Verwenden Sie Blob-Speicher zum Speichern großer Eigenschaftswerte.  

### Gewährleistung der Konsistenz in den gespeicherten Entitäten  

Der andere wichtige Faktor, der Ihrer Schlüsselauswahl für die Optimierung von Datenänderungen beeinflusst ist, wie Konsistenz mit atomischen Transaktionen sichergestellt werden. Sie können nur eine EGT verwenden, um mit Entitäten zu arbeiten, die in derselben Partition gespeichert ist.  

Die folgenden Muster im Abschnitt [Entwurfsmuster für die Tabelle](#table-design-patterns) befassen sich mit der Konsistenzverwaltung:  

-	[Sekundäres Indexmuster für Intra-Partition](#intra-partition-secondary-index-pattern) - Speichern mehrerer Kopien jeder Entität mit unterschiedlichen **RowKey**-Werten (in derselben Partition) zur Aktivierung schneller und effizienter Suchvorgänge und alternativer Sortierreihenfolgen mit unterschiedlichen **RowKey**-Werten.  
-	[Sekundäres Indexmuster für Inter-Partition](#inter-partition-secondary-index-pattern) - Speichern mehrerer Kopien der einzelnen Entitäten mithilfe verschiedener RowKey-Werte in separaten Partitionen oder in separaten Tabellen, um schnelle und effiziente Suchvorgänge und alternativen Sortierreihenfolgen mit anderen **RowKey** Werte zu ermöglichen.  
-	[Eventual Consistency Transaktionsmuster](#eventually-consistent-transactions-pattern) - Aktivieren Sie letztendlich Eventual Consistency über Partitions- oder Speichersystemgrenzen hinweg, indem Sie Azure-Warteschlangen verwenden.
-	[Indexmuster für Entitäten](#index-entities-pattern) - Verwalten von Index-Entitäten, um effiziente Suchvorgänge zu ermöglichen, die Entitäten-Listen zurückgeben.  
-	[Denormalisierungsmuster](#denormalization-pattern) - Fassen Sie verknüpfte Daten in einer einzelnen Entität zusammen, damit Sie alle Daten abrufen können, die Sie für eine einzelne Abfrage benötigen.  
-	[Datenreihenmuster](#data-series-pattern) - Speichern Sie vollständige Datenreihen in einer einzelnen Entität, um die Anzahl der Anforderungen zu minimieren, die Sie vornehmen.  

Informationen zu EGTs finden Sie im Abschnitt [Entitätsgruppentransaktionen](#entity-group-transactions).  

### Sicherstellen des Entwurfs für effiziente Änderungen erleichtert das effiziente Abfragen  

In vielen Fällen ergibt ein Entwurf für effiziente Abfragen effiziente Änderungen, aber Sie sollten immer bewerten, ob dies auch bei Ihrem speziellen Szenario der Fall ist. Einige der Muster im Abschnitt [Entwurfsmuster für die Tabelle](#table-design-patterns) bewerten ausschließlich Kompromisse zwischen Abfragen und der Änderung von Entitäten und Sie sollten immer die Anzahl der einzelnen Vorgänge berücksichtigen.  

Die folgenden Muster im Abschnitt [Entwurfsmuster für die Tabelle](#table-design-patterns) befassen sich mit den Kompromissen zwischen einem Entwurf für effizienten Abfragen und einem Entwurf für effiziente Datenänderung:  

-	[Zusammengesetzte Schlüsselmuster](#compound-key-pattern) - Verwenden Sie zusammengesetzte **RowKey**-Werte, damit der Client in der Lage ist, mit einer einzelnen Abfrage nach verknüpften Daten zu suchen.  
-	[Log Tail-Muster](#log-tail-pattern) - Abrufen der *n* Entitäten, die zuletzt einer Partition hinzugefügt wurden, indem Sie einen **RowKey**-Wert verwenden, mit dem nach Datum und Zeit in umgekehrter Reihenfolge sortiert wird.  

## Modellieren von Beziehungen  

Erstellen von Domänenmodellen ist ein wichtiger Schritt bei der Entwicklung komplexer Systeme. Normalerweise verwenden Sie den Modellierungsprozess, um Entitäten zu identifizieren und die Beziehungen untereinander als Möglichkeit zum Verständnis der Geschäftsdomäne und zur Information für die Konzipierung Ihres Systems. Dieser Abschnitt konzentriert sich darauf, wie Sie einige der allgemeinen Beziehungstypen, die in Domänenmodellen als Entwurf für den Tabellenspeicherdienst zu finden sind, übersetzen können. Der Zuordnungsprozess von einem logischen Datenmodell zu einem physisch basierten NoSQL-Datenmodell unterscheidet sich sehr von dem, was für den Entwurf einer relationalen Datenbank verwendet wird. Der Entwurf von relationalen Datenbanken setzt einen Daten-Normalisierungsprozess voraus, optimiert auf Minimierung der Redundanz, sowie eine deklarative Abfragefunktionen, die abstrahiert, wie die Implementierung für die Datenbank funktioniert.  

### 1:n-Beziehungen  

1:n-Beziehungen zwischen Objekten einer Unternehmensdomäne kommen sehr häufig vor. Beispielsweise verfügt eine Abteilung über viele Mitarbeiter. Es gibt mehrere Möglichkeiten der Implementierung von 1:n-Beziehungen im Tabellenspeicherdienst, jede davon mit Vor-und Nachteilen, die für bestimmte Szenarien von Bedeutung sein können.  

Betrachten Sie das Beispiel eines großen multinationalen Unternehmens mit Zehntausenden Entitäten an Abteilungen und Mitarbeitern, in denen jede Abteilung viele Mitarbeiter und jeder Mitarbeiter einer bestimmte Abteilung zugeordnet ist. Ein Ansatz besteht darin, separate Abteilungs- und Mitarbeiterentitäten wie folgt zu speichern:  

![][1]
 
Dieses Beispiel zeigt eine implizite 1:n-Beziehung zwischen Typen, die auf dem Wert von **PartitionKey** basieren. Jede Abteilung kann viele Mitarbeiter beinhalten.  

Dieses Beispiel zeigt auch die Abteilungsentität und die zugehörigen Mitarbeiterentitäten in derselben Partition. Sie können verschiedene Partitionen, Tabellen oder sogar Speicherkonten für verschiedene Entitätstypen auswählen.  

Ein alternativer Ansatz ist, die Daten zu denormalisieren und nur Mitarbeiterentitäten mit denormalisierten Abteilungsdaten zu speichern, wie im folgenden Beispiel gezeigt. In diesem speziellen Szenario ist der Denormalisierungsansatz möglicherweise nicht der beste Ansatz, falls eine Anforderung besteht, die Details eines Abteilungsmanagers ändern zu müssen. Um dies durchführen zu können, müssen Sie jeden Mitarbeiter in der Abteilung aktualisieren.  

![][2]
 
Weitere Informationen finden Sie unter [Denormalisierungsmuster](#denormalization-pattern) weiter unten in diesem Handbuch.  

In der folgenden Tabelle werden die Vor- und Nachteile der oben genannten einzelnen Ansätze zum Speichern der Entitäten für Mitarbeiter und Abteilung zusammengefasst, die eine 1:n-Beziehung haben. Sie sollten auch berücksichtigen, wie oft verschiedene Vorgänge ausgeführt werden sollen. Es ist möglicherweise akzeptabel, einen Entwurf zu haben, der einen teuren Vorgang beinhaltet, wenn dieser Vorgang nur selten vorkommt.  

<table>
<tr>
<th>Vorgehensweise</th>
<th>Vorteile</th>
<th>Nachteile</th>
</tr>
<tr>
<td>Getrennte Entitätstypen, dieselbe Partition, dieselbe Tabelle</td>
<td>
<ul>
<li>Sie können die Abteilungsentität mit einem einzelnen Vorgang aktualisieren.</li>
<li>Sie können eine EGT zur Beibehaltung der Konsistenz verwenden, wenn Sie eine Anforderung zum Ändern der Abteilungsentität haben, wenn Sie eine Mitarbeiterentität aktualisieren, einfügen oder löschen. Beispiel: Wenn Sie für jede Abteilung eine Abteilungsmitarbeiterzahl verwalten.</li>
</ul>
</td>
<td>
<ul>
<li>Sie müssen möglicherweise eine Mitarbeiter- und Abteilungsentität für einige Clientaktivitäten abrufen.</li>
<li>Speichervorgänge erfolgen in derselben Partition. Bei hohen Transaktionsvolumen kann dies einen Hotspot zur Folge haben.</li>
<li>Sie können mit einer EGT keinen Mitarbeiter in eine neue Abteilung verschieben.</li>
</ul>
</td>
</tr>
<tr>
<td>Separate Entitätstypen, verschiedene Partitionen oder Tabellen oder Speicherkonten</td>
<td>
<ul>
<li>Sie können eine Abteilungsentität oder Mitarbeiterentität mit einem einzelnen Vorgang aktualisieren.</li>
<li>Bei hohen Transaktionsvolumen kann es hilfreich sein, die Last auf mehrere Partitionen zu verteilen.</li>
</ul>
</td>
<td>
<ul>
<li>Sie müssen möglicherweise eine Mitarbeiter- und Abteilungsentität für einige Clientaktivitäten abrufen.</li>
<li>Sie können keine EGTs verwenden, um die Konsistenz beim Aktualisieren, Einfügen oder Löschen eines Mitarbeiter und beim Aktualisieren einer Abteilung beizubehalten. Beispiel: Eine Mitarbeiterzahl einer Abteilungsentität wird aktualisiert.</li>
<li>Sie können mit einer EGT keinen Mitarbeiter in eine neue Abteilung verschieben.</li>
</ul>
</td>
</tr>
<tr>
<td>Denormalisieren in einen einzelnen Entitätstyp</td>
<td>
<ul>
<li>Sie können alle benötigten Informationen mit einer einzelnen Anforderung abrufen.</li>
</ul>
</td>
<td>
<ul>
<li>Es ist möglicherweise teuer, die Konsistenz beizubehalten, wenn Sie Abteilungsdaten aktualisieren müssen (dies würde bedeuten, dass Sie in diesem Fall alle Mitarbeiter in einer Abteilung aktualisieren müssen).</li>
</ul>
</td>
</tr>
</table>
	
Wie Sie zwischen diesen Optionen wählen und welche Vor- und Nachteile am wichtigsten sind, hängt von Ihren speziellen Anwendungsszenarien ab. Beispiele: Wie oft ändern Sie Abteilungsentitäten? Benötigen alle Abfragen für Mitarbeiter die Zusatzinformationen für die Abteilung? Wie nahe liegen Sie an den Skalierbarkeitsgrenzen auf Ihren Partitionen oder Ihrem Speicherkonto?  

### 1:1-Beziehungen  

Domänenmodelle können 1:1-Beziehungen zwischen Entitäten enthalten. Müssen Sie eine 1:1-Beziehung in den Tabellenspeicherdienst implementieren, müssen Sie auch auswählen, wie zwei verknüpfte Entitäten verknüpft werden, wenn sie beide abrufen müssen. Diese Verknüpfung kann entweder implizit sein, basierend auf einer Konvention in den Schlüsselwerten oder explizit durch das Speichern einer Verknüpfung in Form von **PartitionKey**- und **RowKey**-Werten in den einzelnen Entitäten zur verknüpften Entität. Überlegungen dazu, ob die verknüpften Entitäten in der gleichen Partition gespeichert werden sollten, finden Sie im Abschnitt [1:n-Beziehungen](#one-to-many-relationships).  

Beachten Sie, dass es auch Implementierungsaspekte gibt, die Sie veranlassen können, 1:1-Beziehungen in den Tabellenspeicherdienst zu implementieren:  

-	Verarbeiten großer Entitäten (weitere Informationen finden Sie unter [Arbeiten mit großen Entitäten](#working-with-large-entities)).  
-	Implementieren von Zugriffssteuerungen (weitere Informationen finden Sie unter [Steuern des Zugriffs mit Shared Access Signatures](#controlling-access-with-shared-access-signatures)).  

### Verbinden des Clients  

Es gibt zwar Möglichkeiten zum Modellieren von Beziehungen im Tabellenspeicherdienst, Sie sollten aber nicht vergessen, dass die zwei wichtigsten Gründe für die Benutzung des Tabellenspeicherdienstes Skalierbarkeit und Leistung sind. Wenn Sie glauben, dass Sie viele Beziehungen modellieren, welche die Leistung und Skalierbarkeit Ihrer Lösung beeinträchtigen, dann sollten Sie sich fragen, ob notwendig ist, alle Datenbeziehungen in Ihren Tabellenentwurf einzubauen. Sie können möglicherweise den Entwurf vereinfachen und die Skalierbarkeit und Leistung Ihrer Lösung verbessern, wenn Sie Ihre Client-Anwendung alle erforderlichen Verknüpfungen ausführen lassen.  

Beispiel: Wenn Sie kleine Tabellen haben, die Daten enthalten, die nicht sehr häufig geändert werden, dann können Sie diese Daten einmal abrufen und auf dem Client zwischenspeichern. Dies kann wiederholte Roundtrips zum Abrufen derselben Daten vermeiden. In den Beispielen, die wir in diesem Handbuch vorgestellt haben, wird der Satz von Abteilungen in einem kleinen Unternehmen wahrscheinlich klein sein und sich selten ändern. Dadurch wird es zu einem guten Kandidaten für Daten, welche die Clientanwendung einmal herunterladen und als Suchdaten zwischenspeichern kann.  

### Vererbungsbeziehungen  

Wenn die Clientanwendung einen Satz Klassen verwendet, die Teil einer Vererbungsbeziehung zur Darstellung von Geschäftsentitäten sind, können Sie problemlos die Entitäten im Tabellenspeicherdienst beibehalten. Beispiel: Sie haben möglicherweise den folgenden Satz Klassen in Ihrer Clientanwendung definiert, in denen **Person** eine abstrakte Klasse ist.

![][3]
 
Sie können Instanzen der zwei konkreten Klassen im Tabellenspeicherdienst beibehalten, indem Sie eine einzelne Personentabelle mit Entitäten verwenden, die wie folgt aussieht:  

![][4]
 
Weitere Informationen zum Arbeiten mit mehreren Entitätstypen in der gleichen Tabelle im Clientcode finden Sie im Abschnitt [Arbeiten mit heterogenen Entitätstypen](#working-with-heterogeneous-entity-types) weiter unten in diesem Handbuch. Hier finden Sie Beispiele darüber, wie der Entitätstyp im Clientcode erkannt wird.  

## Entwurfsmuster für die Tabelle
In den vorherigen Abschnitten haben Sie in ausführlichen Diskussionen erfahren, wie Sie Ihren Tabellenentwurf für Abrufe von Entitätsdaten mithilfe von Abfragen und das Einfügen, Aktualisieren und Löschen von Entitätsdaten optimieren können. Dieser Abschnitt beschreibt einige Muster, die zur Verwendung mit Tabellenspeicherdienst-Lösungen geeignet sind. Darüber hinaus sehen Sie, wie Sie einige der zuvor in diesem Handbuch angesprochenen Probleme und Kompromisse praktisch behandeln können. Das folgende Diagramm fasst die Beziehungen zwischen den verschiedenen Mustern zusammen:  

![][5]
 
In der Muster-Karte oben werden einige Beziehungen zwischen Muster (Blau) und Antimuster (Orange) hervorgehoben, die in diesem Handbuch dokumentiert sind. Es gibt selbstverständlich viele weitere Muster, die in Betracht gezogen werden können. Beispiel: Eines der Hauptszenarien für den Tabellenspeicherdienst ist die Speicherung von [materialisierten Sichten](https://msdn.microsoft.com/library/azure/dn589782.aspx) aus dem [Command Query Responsibility Segregation](https://msdn.microsoft.com/library/azure/jj554200.aspx)-Muster (CQRS).  

### Sekundäres Indexmuster für Intra-Partition
Speichern mehrerer Kopien jeder Entität mit unterschiedlichen **RowKey**-Werten (in derselben Partition) zur Aktivierung schneller und effizienter Suchvorgänge und alternativen Sortierreihenfolgen mit unterschiedlichen **RowKey**-Werten. Aktualisierungen zwischen Kopien können durch die Verwendung von EGT konsistent sein.  

#### Kontext und Problem
Der Tabellenspeicherdienst indiziert automatisch Entitäten mit den **PartitionKey**- und **RowKey**-Werten. Dadurch wird eine Client-Anwendung in die Lage versetzt, eine Entität effizient unter Verwendung dieser Werte abzurufen. Beispiel: Durch Verwendung der unten gezeigten Tabellenstruktur kann eine Client-Anwendung eine Punktabfrage verwenden, um eine einzelne Mitarbeiterentität mit den Abteilungsnamen und der Mitarbeiter-ID abzurufen (die **PartitionKey**- und **RowKey**-Werte). Ein Client kann auch Entitäten abrufen, die nach Mitarbeiter-ID in jeder Abteilung sortiert sind.

![][6]
 
Wenn Sie auch eine Mitarbeiterentität finden möchten, die auf dem Wert einer anderen Eigenschaft basiert, wie z. B. die E-Mail-Adresse, müssen Sie einen weniger effizienten Partition-Scan verwenden, um eine Übereinstimmung zu finden. Der Grund ist, dass der Tabellenspeicherdienst keine sekundären Indizes bietet. Darüber hinaus steht keine Option zum Anfordern einer Liste der Mitarbeiter zur Verfügung, die in einer anderen Reihenfolge als in der **RowKey**-Reihenfolge sortiert ist.  

#### Lösung
Zum Umgehen des fehlenden sekundären Indizes können Sie mehrere Kopien der einzelnen Entitäten speichern, wobei jede Kopie einen unterschiedlichen **RowKey**-Wert verwendet. Wenn Sie eine Entität mit den unten angezeigten Strukturen speichern, können Sie Mitarbeiterentitäten auf Grundlage der E-Mail-Adresse oder der Mitarbeiter-ID effizient abrufen. Die Präfix-Werte für **RowKey**, "empid_" und "email_" ermöglichen Ihnen, nach einen einzelnen Mitarbeiter oder einen Bereich an Mitarbeitern zu suchen, indem Sie einen E-Mail-Bereich oder Mitarbeiter-IDs verwenden.  

![][7]
 
Die folgenden beiden Filterkriterien (eine Suche nach Mitarbeiter-ID und eine Suche nach E-Mail-Adresse) definieren Punktabfragen:  

-	$filter=(PartitionKey eq 'Sales') and (RowKey eq 'empid_000223')  
-	$filter=(PartitionKey eq 'Sales') and (RowKey eq 'email_jonesj@contoso.com')  

Bei einer Abfrage für eine Bereich an Mitarbeiterentitäten können Sie eine Auswahl festlegen, die in der Mitarbeiter-ID-Reihenfolge sortiert ist, oder einen Bereich, der in E-Mail-Adressen-Reihenfolge sortiert ist, indem Sie nach Entitäten mit dem entsprechenden Präfix in **RowKey** abfragen.  

-	Finden alle Mitarbeiter in der Vertriebsabteilung, für die eine Mitarbeiter-ID im Bereich 000100 bis 000199 verwendet wird:
$filter=(PartitionKey eq 'Sales') und (RowKey ge 'empid_000100') und (RowKey le 'empid_000199')  
-	Alle Mitarbeiter in der Vertriebsabteilung finden, mit einer E-Mail-Adresse, beginnend mit dem Buchstaben 'a' use:
$filter=(PartitionKey eq 'Sales') und (RowKey ge 'email_a') und (RowKey lt 'email_b')  

 Beachten Sie, dass die in den obigen Beispielen verwendete Filtersyntax aus dem REST-API-Tabellenspeicherdienst stammt. Weitere Informationen finden Sie unter [Abfragen von Entitäten](http://msdn.microsoft.com/library/azure/dd179421.aspx) auf MSDN.  

#### Probleme und Überlegungen  

Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:  

-	Der Tabellenspeicher ist relativ günstig zu verwenden, sodass der Kostenaufwand für das Speichern von redundanten Daten kein wichtiger Faktor sein sollte. Sie sollten jedoch immer die Kosten für den Entwurf bewerten, basierend auf dem angenommenen Speicherbedarf und nur doppelte Entitäten hinzufügen, um die Abfragen zu unterstützen, welche die Clientanwendung ausführen wird.  
-	Da die sekundären Index-Entitäten in derselben Partition wie die ursprünglichen Entitäten gespeichert werden, sollten Sie sicherstellen, dass Sie die Skalierbarkeitsziele für eine einzelne Partition nicht überschreiten.  
-	Sie können die doppelten Entitäten zueinander konsistent halten mithilfe von EGTs für die automatische Aktualisierung der beiden Kopien der Entität. Dies bedeutet, dass alle Kopien einer Entität in der gleichen Partition gespeichert werden sollten. Weitere Informationen finden Sie im Abschnitt [Verwenden von Entitätsgruppentransaktionen](#entity-group-transactions).  
-	Der Wert, der für **RowKey** verwendet wird, muss für jede Entität eindeutig sein. Berücksichtigen Sie die Verwendung von zusammengesetzten Schlüsselwerten.  
-	Das Auffüllen der numerischen Werte in **RowKey** (z. B. für die Mitarbeiter-ID 000223) ermöglicht korrekte Sortierung und Filterung, die auf einer oberen und unteren Grenze basieren.  
-	Sie müssen nicht unbedingt alle Eigenschaften der Entität duplizieren. Beispiel: Wenn die Abfragen, welche die Entitäten nach E-Mail-Adresse in **RowKey** suchen, niemals das Alter des Mitarbeiters benötigen, könnten diese Entitäten die folgende Struktur haben:

![][8]
 
-	Es empfiehlt sich, in der Regel doppelte Daten zu speichern und sicherzustellen, dass Sie alle benötigten Daten mit einer einzigen Abfrage abrufen können, anstatt eine Abfrage zum Finden einer Entität und eine weitere zum Suchen nach den benötigten Daten durchzuführen.  

#### Verwendung dieses Musters  

Verwenden Sie dieses Muster, wenn Ihre Clientanwendung Entitäten abrufen muss, die eine Vielzahl verschiedener Schlüssel verwenden, wenn Ihr Client Entitäten in unterschiedlichen Sortierreihenfolgen abrufen muss, und wo Sie jede Entität unter Verwendung einer Vielzahl von eindeutigen Werten identifizieren können. Allerdings sollten Sie sich sicher sein, dass die Skalierungsgrenzen der Partition nicht überschritten werden, wenn Sie eine Suche nach Entitäten unter Verwendung verschiedenen **RowKey**-Werte durchführen.  

#### Zugehörige Muster und Anleitungen  

Die folgenden Muster und Anleitungen können auch relevant sein, wenn dieses Muster implementiert wird:  
 
-	[Sekundäres Indexmuster für Inter-Partition](#inter-partition-secondary-index-pattern)
-	[Zusammengesetzte Schlüsselmuster](#compound-key-pattern)
-	[Entitätsgruppentransaktionen](#entity-group-transactions)
-	[Arbeiten mit heterogenen Entitätstypen](#working-with-heterogeneous-entity-types)

### Sekundäres Indexmuster für Inter-Partition
Speichern mehrerer Kopien der einzelnen Entitäten mithilfe verschiedener **RowKey**-Werte in separaten Partitionen oder in separaten Tabellen ermöglichen schnelle und effiziente Suchvorgänge und alternative Sortierreihenfolgen mit anderen **RowKey** Werte.  

#### Kontext und Problem
Der Tabellenspeicherdienst indiziert automatisch Entitäten mit den **PartitionKey**- und **RowKey**-Werten. Dadurch wird eine Client-Anwendung in die Lage versetzt, eine Entität effizient unter Verwendung dieser Werte abzurufen. Beispiel: Durch Verwendung der unten gezeigten Tabellenstruktur kann eine Client-Anwendung eine Punktabfrage verwenden, um eine einzelne Mitarbeiterentität mit den Abteilungsnamen und der Mitarbeiter-ID abzurufen (die **PartitionKey**- und **RowKey**-Werte). Ein Client kann auch Entitäten abrufen, die nach Mitarbeiter-ID in jeder Abteilung sortiert sind.  

![][9]
 
Wenn Sie auch eine Mitarbeiterentität finden möchten, die auf dem Wert einer anderen Eigenschaft basiert, wie z. B. die E-Mail-Adresse, müssen Sie einen weniger effizienten Partition-Scan verwenden, um eine Übereinstimmung zu finden. Der Grund ist, dass der Tabellenspeicherdienst keine sekundären Indizes bietet. Darüber hinaus steht keine Option zum Anfordern einer Liste der Mitarbeiter zur Verfügung, die in einer anderen Reihenfolge als in der **RowKey**-Reihenfolge sortiert ist.  

Sie werden eine sehr große Anzahl von Transaktionen für diese Entitäten abschätzen und möchten das Risiko minimieren, dass der Tabellenspeicherdienst Ihren Client drosselt.  

#### Lösung  
Um das Fehlen von sekundären Indizes zu umgehen, können Sie mehrere Kopien der einzelnen Entitäten mit jeder Kopie verwenden, indem Sie unterschiedliche **PartitionKey**- und **RowKey**-Werte verwenden. Wenn Sie eine Entität mit den unten angezeigten Strukturen speichern, können Sie Mitarbeiterentitäten auf Grundlage der E-Mail-Adresse oder der Mitarbeiter-ID effizient abrufen. Die Präfix-Werte für **PartitionKey**, "empid_" und "email_" ermöglichen Ihnen zu identifizieren, welchen Index Sie für eine Abfrage verwenden möchten.  

![][10]
 
Die folgenden beiden Filterkriterien (eine Suche nach Mitarbeiter-ID und eine Suche nach E-Mail-Adresse) definieren Punktabfragen:  

-	$filter=(PartitionKey eq 'empid_Sales') und (RowKey eq '000223')
-	$filter=(PartitionKey eq 'email_Sales') und (RowKey eq 'jonesj@contoso.com')  

Bei einer Abfrage für eine Bereich an Mitarbeiterentitäten können Sie eine Auswahl festlegen, die in der Mitarbeiter-ID-Reihenfolge sortiert ist, oder einen Bereich, der in E-Mail-Adressen-Reihenfolge sortiert ist, indem Sie nach Entitäten mit dem entsprechenden Präfix in **RowKey** abfragen.  

-	Finden aller Mitarbeiter in der Vertriebsabteilung mit einer Mitarbeiter-ID im Bereich **000100** bis **000199**, sortiert nach Mitarbeiter-ID-Reihenfolge: 
$filter=(PartitionKey eq 'empid_Sales') und (RowKey ge '000100') und (RowKey le '000199')  
-	Finden aller Mitarbeiter in der Vertriebsabteilung mit einer e-Mail-Adresse, beginnend mit 'a' sorted in email address order use:
$filter=(PartitionKey eq 'email_Sales') und (RowKey ge 'a') and (RowKey lt 'b')  

Beachten Sie, dass die in den obigen Beispielen verwendete Filtersyntax aus dem REST-API-Tabellenspeicherdienst stammt. Weitere Informationen finden Sie unter [Abfragen von Entitäten](http://msdn.microsoft.com/library/azure/dd179421.aspx) auf MSDN.  

#### Probleme und Überlegungen  
Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:  

-	Sie können die doppelten Entitäten Eventually Consistent halten, indem Sie [Eventual Consistency Transaktionsmuster] verwenden, um(#eventually-consistent-transactions-pattern) die primären und sekundären Index-Entitäten verwalten.  
-	Der Tabellenspeicher ist relativ günstig zu verwenden, sodass der Kostenaufwand für das Speichern von redundanten Daten kein wichtiger Faktor sein sollte. Sie sollten jedoch immer die Kosten für den Entwurf bewerten, basierend auf dem angenommenen Speicherbedarf und nur doppelte Entitäten hinzufügen, um die Abfragen zu unterstützen, welche die Clientanwendung ausführen wird.  
-	Der Wert, der für **RowKey** verwendet wird, muss für jede Entität eindeutig sein. Berücksichtigen Sie die Verwendung von zusammengesetzten Schlüsselwerten.  
-	Das Auffüllen der numerischen Werte in **RowKey** (z. B. für die Mitarbeiter-ID 000223) ermöglicht korrekte Sortierung und Filterung, die auf einer oberen und unteren Grenze basieren.  
-	Sie müssen nicht unbedingt alle Eigenschaften der Entität duplizieren. Beispiel: Wenn die Abfragen, welche die Entitäten nach E-Mail-Adresse in **RowKey** suchen, niemals das Alter des Mitarbeiters benötigen, könnten diese Entitäten die folgende Struktur haben:

	![][11]
 
-	Es ist in der Regel besser, doppelte Daten zu speichern und sicherzustellen, dass Sie alle benötigten Daten mit einer einzigen Abfrage abrufen können, anstatt eine Abfrage zum Finden einer Entität unter Verwendung des sekundären Indexes und eine weitere Abfrage zum Suchen nach den benötigten Daten im primären Index durchzuführen.  

#### Verwendung dieses Musters  
Verwenden Sie dieses Muster, wenn Ihre Clientanwendung Entitäten abrufen muss, die eine Vielzahl verschiedener Schlüssel verwenden, wenn Ihr Client Entitäten in unterschiedlichen Sortierreihenfolgen abrufen muss, und wo Sie jede Entität unter Verwendung einer Vielzahl von eindeutigen Werten identifizieren können. Verwenden Sie dieses Muster, wenn Sie ein Überschreiten der Skalierungsgrenzen der Partition verhindern möchten, wenn Sie eine Suche nach Entitäten durchführen, die verschiedene **RowKey**-Werte verwenden.  

#### Zugehörige Muster und Anleitungen
Die folgenden Muster und Anleitungen können auch relevant sein, wenn dieses Muster implementiert wird:  
 
-	[Eventual Consistency Transaktionsmuster](#eventually-consistent-transactions-pattern)  
-	[Sekundäres Indexmuster für Intra-Partition](#intra-partition-secondary-index-pattern)  
-	[Zusammengesetzte Schlüsselmuster](#compound-key-pattern)  
-	[Entitätsgruppentransaktionen](#entity-group-transactions)  
-	[Arbeiten mit heterogenen Entitätstypen](#working-with-heterogeneous-entity-types)  

### Eventual Consistency Transaktionsmuster  

Aktivieren Sie Eventual Consistency über Partitions- oder Speichersystemgrenzen hinweg, indem Sie Azure-Warteschlangen verwenden.  

#### Kontext und Problem  

EGTs ermöglichen atomische Transaktionen über mehrere Entitäten, die den gleichen Partitionsschlüssel gemeinsam nutzen. Aus Gründen der Leistung und Skalierbarkeit möchten Sie Entitäten speichern, die Anforderungen an die Datenkonsistenz in separaten Partitionen oder in einem separaten Speichersystem haben. In diesem Fall können keine EGTs zur Gewährleistung der Konsistenz verwendet werden. Beispiel: Sie haben eine Anforderung, bei der in den folgenden Fällen Eventual Consistency beibehalten werden muss:  

-	Entitäten, die in verschiedenen Partitionen in derselben Tabelle, in verschiedenen Tabellen oder in verschiedenen Speicherkonten gespeichert sind.  
-	Eine Entität, die im Tabellenspeicherdienst gespeichert ist und einen Blob, der im Blob-Dienst gespeichert ist.  
-	Eine Entität, die im Tabellenspeicherdienst gespeichert ist und eine Datei in einem Dateisystem.  
-	Speicherung einer Entität im Tabellenspeicherdienst, der jedoch mithilfe des Azure-Suchdienstes indiziert ist.  

#### Lösung  

Mithilfe von Azure-Warteschlangen können Sie eine Lösung implementieren, die Eventual Consistency über zwei oder mehr Partitionen oder Speichersysteme bietet.
Zur Veranschaulichung dieses Ansatzes gehen Sie davon aus, dass Sie eine Anforderung haben, veraltete Mitarbeiterentitäten archivieren zu können. Alte Mitarbeiterentitäten werden selten abgefragt und sollten aus allen Aktivitäten ausgeschlossen werden, die sich mit den aktuellen Mitarbeitern beschäftigen. Um diese Anforderung zu implementieren, speichern Sie die aktiven Mitarbeiter in der Tabelle **Aktuell** und alte Mitarbeiter in der Tabelle **Archiv**. Zum Archivieren eines Mitarbeiters löschen Sie die Entität aus der Tabelle **Aktuell** und fügen die Entität in die Tabelle **Archiv** ein. Sie können aber keine EGT verwenden, um diese beiden Vorgänge auszuführen. Um das Risiko zu vermeiden, das aufgrund eines Fehlers eine Entität in beiden oder in keiner Tabelle angezeigt wird, muss der Archivierungsvorgang letztendlich Eventual Consistency aufweisen. Das folgende Sequenzdiagramm beschreibt die Schritte bei diesem Vorgang. Mehr Details zu Ausnahmepfaden werden im folgenden Text zur Verfügung gestellt.  

![][12]
 
Ein Client startet den Archiv-Vorgang, indem er eine Meldung auf einer Azure-Warteschlange ablegt. In diesem Beispiel wird Mitarbeiter #456 archiviert. Eine Workerrolle fragt die Warteschlange auf neue Meldungen ab. Wird eine gefunden, liest sie die Meldung und hinterlässt eine verborgene Kopie in der Warteschlange. Die Workerrolle ruft als Nächstes eine Kopie der Entität aus der Tabelle **Aktuell** ab, fügt eine Kopie in die Tabelle **Archiv** ein, und löscht dann die ursprüngliche Entität aus der Tabelle **Aktuell**. Falls keine Fehler in den vorangegangenen Schritten aufgetreten sind, löscht die Workerrolle schlussendlich die verborgene Nachricht aus der Warteschlange.  

In diesem Beispiel fügt Schritt 4 den Mitarbeiter in die Tabelle **Archiv** ein. Es könnte den Mitarbeiter einem Blob im Blob-Dienst oder einer Datei in einem Dateisystem hinzufügen.  

#### Wiederherstellung bei Fehlern  

Es ist wichtig, dass die Vorgänge in den Schritten **4** und **5** *idempotent* sein müssen für den Fall, dass die Workerrolle den Archivierungsvorgang neu starten muss. Bei Verwendung des Tabellenspeicherdienstes für Schritt **4** sollten Sie einen Vorgang "Einfügen oder ersetzen" bzw. für Schritt **5** "Löschen, wenn vorhanden" in der von Ihnen verwendeten Client-Bibliothek durchführen. Wenn Sie ein anderes Speichersystem verwenden, müssen Sie einen entsprechenden idempotenten Vorgang verwenden.  

Wenn die Worker-Rolle Schritt **6**, niemals abschließt, erscheint nach einem Zeitüberlauf die Meldung wieder in der Warteschlange, damit die Workerrolle erneut versuchen kann, sie erneut zu verarbeiten. Die Workerrolle kann überprüfen, wie oft eine Meldung in der Warteschlange gelesen wurde und bei Bedarf markieren, dass dies eine "unzustellbare" Meldung ist und sie zur Untersuchung an eine eigene Warteschlange senden. Weitere Informationen zum Lesen von Warteschlangenmeldungen und zur Überprüfung des Zählers für Entfernungsvorgänge aus der Warteschlange finden Sie unter [Get Messages](https://msdn.microsoft.com/library/azure/dd179474.aspx).  

Einige Fehler aus der Tabelle und den Warteschlangendiensten sind vorübergehende Fehler. Ihre Clientanwendung sollte diese über eine geeignete Wiederholungslogik bedienen können.  

#### Probleme und Überlegungen
Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:  

-	Diese Lösung bietet keine Transaktionsisolation. Beispielsweise könnte ein Client die Tabellen **Aktuell** und **Archiv** lesen, wenn sich die Workerrolle zwischen den Schritten **4** und **5**, befunden hat und eine uneinheitliche Ansicht der Daten festgestellt wird. Beachten Sie, dass die Daten Eventually Consistent ist.  
-	Sie müssen darauf achten, dass die Schritte 4 und 5 idempotent sind, um Eventual Consistency zu gewährleisten.  
-	Sie können die Lösung skalieren, indem Sie mehrerer Warteschlangen und Workerrollen-Instanzen verwenden.  

#### Verwendung dieses Musters  
Verwenden Sie dieses Muster, um Eventual Consistency zwischen Entitäten zu gewährleisten, die in verschiedenen Partitionen oder Tabellen vorhanden sind. Sie können dieses Muster zur Gewährleistung von Eventual Consistency für alle Vorgänge zum Tabellenspeicherdienst, Blob-Dienst und andere Nicht-Azure Tabellenspeicher-Datenquellen wie Datenbank oder Dateisystem verwenden.  

#### Zugehörige Muster und Anleitungen  
Die folgenden Muster und Anleitungen können auch relevant sein, wenn dieses Muster implementiert wird:  
-	[Entitätsgruppentransaktionen](#entity-group-transactions)  
-	[Zusammenführen oder ersetzen](#merge-or-replace)  

>[AZURE.NOTE] Falls Transaktionsisolation für Ihre Lösung wichtig ist, sollten Sie einen Neuentwurf Ihrer Tabellen erwägen, damit Sie EGTs verwenden können.  

### Indexmuster für Entitäten
Verwalten von Index-Entitäten, um effiziente Suchvorgänge zu ermöglichen, die Listen mit Entitäten zurückgeben.  

#### Kontext und Problem  

Der Tabellenspeicherdienst indiziert automatisch Entitäten mit den **PartitionKey**- und **RowKey**-Werten. Dadurch kann eine Clientanwendung, eine Entität effizient mithilfe einer Punktabfrage abrufen. Beispiel: Durch die Verwendung der unten gezeigten Tabellenstruktur kann eine Clientanwendung effizient eine einzelne Mitarbeiterentität abrufen, indem es den Abteilungsnamen und die Mitarbeiter-ID abruft(**PartitionKey** und **RowKey**).  
 
![][13]

Wenn Sie auch in der Lage sein möchten, eine Liste der Mitarbeiterentitäten abzurufen, die auf dem Wert einer anderen nicht eindeutigen Eigenschaft basiert, z. B. Nachname, müssen Sie einen weniger effizienten Partition-Scan verwenden, um Übereinstimmungen zu finden, anstatt einen Index zum direkt Nachschlagen zu verwenden. Der Grund ist, dass der Tabellenspeicherdienst keine sekundären Indizes bietet.  

#### Lösung  

Um die Suche nach dem Nachnamen mit der oben dargestellten Entitätsstruktur zu aktivieren, müssen Sie die Listen der Mitarbeiter-IDs verwalten. Wenn Sie die Mitarbeiterentitäten mit einem bestimmten Nachnamen aufrufen möchten, z. B. Jones, müssen Sie zuerst die Liste der Mitarbeiter-IDs für die Mitarbeiter finden, deren Nachname Jones ist und dann diese Mitarbeiterentitäten abrufen. Es gibt drei wichtige Optionen zur Speicherung der Mitarbeiter-ID-Listen:  

-	Verwenden von Blob-Speicher.  
-	Erstellen von Index-Entitäten in der gleichen Partition wie für die Mitarbeiterentitäten.  
-	Erstellen von Index-Entitäten in einer separaten Partition oder Tabelle.  

<u>Option 1: Verwenden von Blob-Speicher</u>  

Für die erste Option erstellen Sie einen Blob für jeden eindeutigen Nachnamen und speichern in jedem Blob-Speicher eine Liste mit **PartitionKey**-Werten (Abteilung) und **RowKey**-Werten (Mitarbeiter-ID) für die Mitarbeiter mit diesem Nachnamen. Beim Hinzufügen oder Löschen eines Mitarbeiters sollten Sie sicherstellen, dass der Inhalt des relevanten Blob mit den Mitarbeiterentitäten Eventually Consistent ist.  

<u>Option 2:</u> Erstellen von Indexentitäten in der gleichen Partition  

Verwenden Sie für die zweite Option die Index-Entitäten, die folgende Daten speichern:  

![][14]
 
Die Eigenschaft **EmployeeIDs** enthält eine Liste der Mitarbeiter-IDs für Mitarbeiter, deren Nachnamen in **RowKey** gespeichert sind.  

Wenn Sie die zweite Option verwenden, beschreiben die folgenden Schritte den Prozess, den Sie befolgen sollten, wenn Sie einen neuen Mitarbeiter hinzufügen. In diesem Beispiel werden wir einen Mitarbeiter mit der ID 000152 und einen Nachnamen Jones in der Vertriebsabteilung hinzufügen:  
1.	Rufen Sie die Index-Entität mit einem **PartitionKey**-Wert "Vertrieb" und **RowKey**-Wert "Jones" ab. Speichern Sie das ETag der Entität, die in Schritt 2 verwendet wird.  
2.	Erstellen Sie ein EGT, mit der die neue Mitarbeiterentität eingefügt wird (**PartitionKey**-Wert "Sales" und **RowKey**-Wert "000152") und aktualisieren Sie die Index-Entität (**PartitionKey**-Wert "Sales" und **RowKey**-Wert "Jones") durch Hinzufügen der neuen Mitarbeiter-ID in der Liste im Feld EmployeeIDs.  
3.	Falls die ETG aufgrund eines Fehlers der vollständigen Nebenläufigkeit (jemand hat gerade die Index-Entität geändert) nicht erfolgreich ist, müssen Sie erneut mit Schritt 1 beginnen.  

Wenn Sie die zweite Option verwenden, können Sie einen ähnlichen Ansatz beim Löschen eines Mitarbeiters wählen. Das Ändern des Nachnamen des Mitarbeiters ist etwas komplexer, da Sie eine ETG ausführen, die drei Entitäten aktualisiert: die Mitarbeiterentität, die Index-Entität für den alten Nachnamen und die Index-Entität für den neuen Nachnamen. Sie müssen jede Entität abrufen, bevor Sie Änderungen vornehmen, um die ETag-Werte abzurufen, mit denen Sie dann die Updates unter Verwendung von optimistischer Nebenläufigkeit ausführen.  

Wenn Sie die zweite Option verwenden, beschreiben die folgenden Schritte den Prozess, den Sie befolgen sollten, wenn Sie alle Mitarbeiter mit einem bestimmten Nachnamen in einer Abteilung nachschlagen. In diesem Beispiel schlagen wir alle Mitarbeiter in der Vertriebsabteilung mit dem Nachnamen Jones nach:  

1.	Rufen Sie die Index-Entität mit einem **PartitionKey**-Wert "Vertrieb" und **RowKey**-Wert "Jones" ab.  
2.	Analysieren Sie die Liste der Mitarbeiter-IDs im Feld EmployeeIDs.  
3.	Falls Sie zusätzliche Informationen zu den einzelnen Angestellten benötigen (z. B. ihre e-Mail-Adressen), dann rufen Sie jede Mitarbeiterentität ab, die den **PartitionKey**-Wert "Sales" und die **RowKey**-Werte aus der Liste der Mitarbeiter verwenden, die Sie in Schritt 2 ermittelt haben.  

<u>Option 3:</u> Erstellen von Indexentitäten in einer separaten Partition oder Tabelle.  

Verwenden Sie für die dritte Option Index-Entitäten, die folgende Daten speichern:  

![][15]
 
Die Eigenschaft **EmployeeIDs** enthält eine Liste der Mitarbeiter-IDs für Mitarbeiter, deren Nachnamen in **RowKey** gespeichert sind.  

Mit der dritten Option können Sie keine EGTs zur Aufrechterhaltung der Konsistenz verwenden, da sich die Index-Entitäten in einer anderen Partition wie die Mitarbeiterentitäten befinden. Sie sollten sicherstellen, dass die Index-Entitäten Eventual Consistency mit den Mitarbeiterentitäten aufweisen.  

#### Probleme und Überlegungen  

Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:  
-	Diese Lösung erfordert mindestens zwei Abfragen zum Abrufen von übereinstimmenden Entitäten, eine zur Abfrage der Index-Entitäten, um eine Liste mit **RowKey**-Werte zu erhalten und dann Abfragen zum Abruf jeder Entität aus dieser Liste.  
-	Angesichts der Tatsache, dass eine einzelne Entität eine Maximalgröße von 1 MB hat, wird bei Option Nr. 2 und Option Nr. 3 der Lösungen angenommen, dass die Liste der Mitarbeiter-IDs für einen angegebenen Nachnamen nie größer als 1 MB ist. Wenn die Liste der Mitarbeiter-IDs aller Voraussicht nach größer als 1 MB wird, verwenden Sie die Option Nr. 1 und speichern Sie die Indexdaten im Blob-Speicher ab.  
-	Bei Verwendung der Option Nr. 2 (Verwendung von EGTs zum Hinzufügen und Löschen von Mitarbeitern und Ändern des Nachnamens eines Mitarbeiters) müssen Sie bewerten, ob die Transaktionsmenge sich den Skalierungsgrenzen in einer gegebenen Partition nähert. Wenn dies der Fall ist, sollten Sie eine Lösung mit Eventual Consistency (Option Nr. 1 oder Nr. 3) erwägen, die Warteschlangen zur Behandlung der Update-Anforderungen verwendet und Ihnen das Speichern Ihrer Index-Entitäten in einer von den Mitarbeiterentitäten separaten Partition ermöglicht.  
-	In Option Nr. 2 in dieser Lösung wird davon ausgegangen, dass Sie innerhalb einer Abteilung nach Nachnamen nachschlagen möchten: z. B. eine Liste von Mitarbeitern abrufen mit einem Nachnamen Jones in der Vertriebsabteilung. Wenn Sie eine Suche aller Mitarbeiter mit einem Nachnamen Jones in der gesamten Organisation durchführen möchten, verwenden Sie entweder die Option Nr. 1 oder Nr. 3.   
-	Sie können eine auf einer Warteschlange basierende Lösung implementieren, die Eventual Consistency bietet (siehe [Eventual Consistency Transaktionsmuster](#eventually-consistent-transactions-pattern) für weitere Details).  

#### Verwendung dieses Musters  

Verwenden Sie dieses Muster, wenn Sie eine Reihe von Entitäten nachschlagen möchten, die einen gemeinsamen Eigenschaftswert haben, z. B. alle Mitarbeiter mit dem Nachnamen Jones.  

#### Zugehörige Muster und Anleitungen  

Die folgenden Muster und Anleitungen können auch relevant sein, wenn dieses Muster implementiert wird:  
-	[Zusammengesetzte Schlüsselmuster](#compound-key-pattern)  
-	[Eventual Consistency Transaktionsmuster](#eventually-consistent-transactions-pattern)  
-	[Entitätsgruppentransaktionen](#entity-group-transactions)  
-	[Arbeiten mit heterogenen Entitätstypen](#working-with-heterogeneous-entity-types)  

### Denormalisierungsmuster  

Kombinieren Sie zugehörige Daten zusammen in eine einzelne Entität, mit der Sie alle Daten abrufen, die Sie mit einer einzelnen Abfrage benötigen.  

#### Kontext und Problem  

In einer relationalen Datenbank normalisieren Sie typischerweise Daten, um Duplizierungen zu entfernen, die zu Abfragen führen, die Daten aus mehreren Tabellen abrufen. Wenn Sie Ihre Daten in Azure-Tabellen normalisieren, müssen Sie mehrere Roundtrips vom Client zum Server durchlaufen, um Ihre zugeordneten Daten abzurufen. Beispiel: Mit der Tabellenstruktur unten benötigen Sie zwei Roundtrips zum Abrufen der Details für eine Abteilung. Mit dem ersten Roundtrip werden die Abteilungsentitäten mit der Manager-ID abgerufen. Mit der zweiten Anforderung werden die Details des Managers aus einer Mitarbeiterentität abgerufen.  

![][16]
 
#### Lösung  

Anstatt die Daten in zwei separaten Entitäten zu speichern, denormalisieren Sie die Daten und bewahren eine Kopie der Details des Managers in der Abteilungsentität auf. Beispiel:  

![][17]
 
Sie können von Abteilungsentitäten, die mit diesen Eigenschaften gespeichert sind, jetzt alle benötigten Details zu einer Abteilung abrufen, indem Sie eine Punktabfrage verwenden.  

#### Probleme und Überlegungen  

Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:  

-	Im Zusammenhang mit dem zweifachen Speichern von Daten fallen Mehrkosten an. Der Leistungsvorteil (der sich aus weniger Anforderungen an den Speicherdienst ergibt) überwiegt in der Regel die marginal höheren Speicherkosten (und diese Kosten werden teilweise durch eine Reduzierung der Anzahl der Transaktionen ausgeglichen, die Sie benötigen, um die Details einer Abteilung abrufen).  
-	Sie müssen die Konsistenz der beiden Entitäten verwalten, die Informationen über Manager speichern. Sie können das Konsistenzproblem behandeln, indem Sie EGTs zum Aktualisieren von mehreren Entitäten in einer einzelnen atomischen Transaktion verwenden. In diesem Fall werden die Abteilungsentität und die Mitarbeiterentität für den Manager der Abteilung in der gleichen Partition gespeichert.  

#### Verwendung dieses Musters
Verwenden Sie dieses Muster, wenn Sie häufig nach zugeordneten Informationen suchen müssen. Dieses Muster reduziert die Anzahl der Abfragen, die Ihr Client vornehmen muss, um die benötigten Daten abzurufen.  

#### Zugehörige Muster und Anleitungen
Die folgenden Muster und Anleitungen können auch relevant sein, wenn dieses Muster implementiert wird:  
-	[Zusammengesetzte Schlüsselmuster](#compound-key-pattern)  
-	[Entitätsgruppentransaktionen](#entity-group-transactions)  
-	[Arbeiten mit heterogenen Entitätstypen](#working-with-heterogeneous-entity-types)   

### Zusammengesetzte Schlüsselmuster  

Verwenden Sie zusammengesetzte **RowKey**-Werte, damit ein Client in die Lage versetzt wird, mit einer einzelnen Punktabfrage nach zugehörigen Daten zu suchen.  

#### Kontext und Problem  

In einer relationalen Datenbank ist es ganz natürlich, Verknüpfungen in Abfragen zu verwenden, um zugehörige Datenelemente in einer einzelnen Abfrage an den Client zurückzugeben. Beispiel: Sie können die Mitarbeiter-ID verwenden, um eine Liste von verknüpften Entitäten nachzuschlagen, die Leistungs- und Überprüfungsdaten für diesen Mitarbeiter enthalten.  

Angenommen, Sie speichern Mitarbeiterentitäten im Tabellenspeicherdienst mithilfe der folgenden Struktur:  

![][18]
 
Sie müssen auch Vergangenheitsdaten speichern, die im Zusammenhang mit Überprüfungen und Leistung für jedes Jahr, in dem der Mitarbeiter für Ihre Organisation gearbeitet hat, stehen und Sie müssen auf diese Informationen nach Jahr zugreifen können. Eine Möglichkeit ist, eine weitere Tabelle zu erstellen, in der Entitäten mit der folgenden Struktur gespeichert werden:  

![][19]
 
Beachten Sie, dass Sie bei diesem Ansatz entscheiden könnten, dass Sie einige Informationen (z. B. Vorname und Nachname) in die neue Entität duplizieren, um zu ermöglichen, dass Sie Ihre Daten mit einer einzelnen Anforderung abrufen. Sie können jedoch keine starke Konsistenz beibehalten, da Sie keine EGT verwenden können, die beide Entitäten nicht trennbar aktualisiert.  

#### Lösung
Speichern Sie einen neuen Entitätstyp in der ursprünglichen Tabelle unter Verwendung von Entitäten mit der folgenden Struktur ab:  

![][20]
 
Beachten Sie, dass **RowKey** jetzt ein zusammengesetzter Schlüssel ist, der sich aus der Mitarbeiter-ID und dem Jahr der Review-Daten zusammensetzt und der es Ihnen ermöglicht, die Leistungs- und Review-Daten des Mitarbeiters mit einer einzigen Anforderung für eine einzelne Entität abzurufen.  

Im folgende Beispiel wird erläutert, wie Sie alle Review-Daten für einen bestimmten Mitarbeiter (z. B. Mitarbeiter 000123 in der Vertriebsabteilung) abrufen können:  

$filter=(PartitionKey eq 'Sales') und (RowKey ge 'empid_000123') und (RowKey lt 'empid_000124')&$select=RowKey,Manager Rating,Peer Rating,Comments  

#### Probleme und Überlegungen
Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:  

-	Sie sollten ein geeignetes Trennzeichen verwenden, das die Analyse des **RowKey**-Werts erleichtert: z. B. **000123_2012**.  
-	Sie speichern diese Entität auch in der gleichen Partition wie andere Entitäten mit verknüpften Daten für denselben Mitarbeiter, was bedeutet, dass Sie EGTs verwenden können, um starke Konsistenz zu gewährleisten.
-	Sie sollten berücksichtigen, wie oft Sie die Daten abfragen, um zu bestimmen, ob dieses Muster geeignet ist.  Beispiel: Wenn Sie auf die Review-Daten selten und auf die wichtigsten Mitarbeiterdaten häufig zugreifen, sollten Sie diese als separate Entitäten beibehalten.  

#### Verwendung dieses Musters  

Verwenden Sie dieses Muster, wenn Sie eine oder mehrere Entitäten, die Sie häufig abfragen, speichern müssen.  

#### Zugehörige Muster und Anleitungen  

Die folgenden Muster und Anleitungen können auch relevant sein, wenn dieses Muster implementiert wird:  

-	[Entitätsgruppentransaktionen](#entity-group-transactions)  
-	[Arbeiten mit heterogenen Entitätstypen](#working-with-heterogeneous-entity-types)  
-	[Eventual Consistency Transaktionsmuster](#eventually-consistent-transactions-pattern)  

### Log Tail-Muster  

Abrufen der *n* Entitäten, die zuletzt einer Partition hinzugefügt wurden, indem Sie einen **RowKey**-Wert verwenden, mit dem nach Datum und Zeit in umgekehrter Reihenfolge sortiert wird.  

#### Kontext und Problem  

Eine gängige Anforderung ist, die zuletzt erstellten Entitäten abzurufen, z. B. die letzten zehn Kostenabrechnungen, die von einem Mitarbeiter übermittelt wurden. Tabellenabfragen unterstützen einen **$top**-Abfragevorgang, um die ersten *n*Entitäten eines Satzes zurückzugeben. Es gibt keinen entsprechenden Abfragevorgang, mit dem die letzten n Entitäten eines Satzes zurückgegeben werden können.  

#### Lösung  

Speichern von Entitäten unter Verwendung eines **RowKey**, der in der umgekehrten Datum/Zeit-Reihenfolge natürlich sortiert. Damit ist der letzte Eintrag immer der erste Eintrag in der Tabelle.  

Beispiel: Um die zehn neuesten Kostenabrechnungen, die von einem Mitarbeiter übermittelt wurden, abrufen zu können, können Sie einen umgekehrten Tick-Wert verwenden, der aus dem aktuellen Wert für Datum/Uhrzeit abgeleitet ist. Das folgende C#-Codebeispiel zeigt eine Möglichkeit, einen geeigneten "invertierten Ticks"-Wert für einen **RowKey** zu erstellen, der von der neuesten bis zur ältesten Information sortiert:  

`string invertedTicks = string.Format("{0:D19}", DateTime.MaxValue.Ticks - DateTime.UtcNow.Ticks);`  

Sie können wieder zum Wert Datum/Uhrzeit mithilfe des folgenden Codes zurückkehren:  

`DateTime dt = new DateTime(DateTime.MaxValue.Ticks - Int64.Parse(invertedTicks));`  

Die Tabellenabfrage sieht folgendermaßen aus:  

`https://myaccount.table.core.windows.net/EmployeeExpense(PartitionKey='empid')?$top=10`  

#### Probleme und Überlegungen  

Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:  

-	Sie müssen den umgekehrten Tick-Wert mit führenden Nullen auffüllen, um sicherzustellen, dass der Zeichenfolgewert wie erwartet sortiert.  
-	Sie müssen die Skalierbarkeitsziele auf der Ebene einer Partition kennen. Vorsicht, keine Hotspot-Partitionen erstellen.  

#### Verwendung dieses Musters  

Verwenden Sie dieses Muster, wenn Sie auf Entitäten in umgekehrter Datum/Uhrzeit-Reihenfolge oder auf die zuletzt hinzugefügten Entitäten zugreifen müssen.  

#### Zugehörige Muster und Anleitungen  

Die folgenden Muster und Anleitungen können auch relevant sein, wenn dieses Muster implementiert wird:  

-	[Voranstellen / Anfügen Antimuster](#prepend-append-anti-pattern)  
-	[Abrufen von Entitäten](#retrieving-entities)  

### Muster für das Löschen hoher Volumen  

Aktivieren Sie das Löschen einer hohen Anzahl von Entitäten, indem Sie alle Entitäten zum gleichzeitigen Löschen in ihrer eigenen separaten Tabelle speichern. Löschen Sie die Entitäten durch Löschen der Tabelle.  

#### Kontext und Problem  

Viele Anwendungen löschen alte Daten, die für eine Clientanwendung nicht mehr verfügbar sein müssen oder die von der Anwendung auf einem anderen Speichermedium archiviert wurden. Sie kennzeichnen solche Daten in der Regel mit einem Datum. Sie haben z. B. eine Anforderung zum Löschen von Datensätzen aller Anmeldeanforderungen, die älter 60 Tage sind.  

Eine möglicher Entwurf ist die Verwendung von Datum und Uhrzeit der Anmeldeanforderungen in **RowKey**:  
 
![][21]

Dieser Ansatz vermeidet Partition-Hotspots, da die Anwendung Anmeldeentitäten für jeden Benutzer in eine separate Partition einfügen und löschen kann. Allerdings kann dieser Ansatz teuer und zeitaufwendig sein, wenn die Anzahl Ihrer Entitäten groß ist, da Sie zunächst einen Tabellen-Scan ausführen müssen, um die zu löschenden Entitäten zu identifizieren. Danach müssen Sie jede alte Entität löschen. Beachten Sie, dass Sie die Anzahl der Roundtrips zum Server reduzieren können, die zum Löschen der alten Entitäten notwendig sind, indem Sie mehrere Löschanforderungen in EGTs stapeln.  

#### Lösung  

Verwenden Sie für Anmeldeversuche jeden Tag eine separate Tabelle. Sie können den oben genannten Entitätsentwurf verwenden, um beim Einfügen von Entitäten Hotspots zu vermeiden. Löschen von alten Entitäten ist jetzt einfach eine Frage des Löschens von einer Tabelle pro Tag (ein einzelner Speichervorgang), statt jeden Tag Hunderte und Tausende von individuellen Anmeldeentitäten zu suchen und zu löschen.  

#### Probleme und Überlegungen  

Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:  

-	Unterstützt der Entwurf Ihrer Anwendung andere Möglichkeiten einer Datenverwendung, z. B. die Suche nach bestimmten Entitäten, die Verknüpfung mit anderen Daten oder das Generieren von zusammengefassten Informationen?  
-	Vermeidet Ihren Entwurf Hotspots, wenn Sie neue Entitäten einfügen?  
-	Rechnen Sie mit einer Verzögerungszeit, wenn Sie denselben Tabellennamen nach dem Löschen wiederverwenden möchten. Es empfiehlt sich, immer einen eindeutigen Tabellennamen zu verwenden.  
-	Rechnen Sie mit einer Drosselung, wenn Sie zuerst eine neue Tabelle verwenden, während der Dienst die Zugriffsmuster lernt und die Partitionen auf den Knoten verteilt. Sie sollten berücksichtigen, wie häufig Sie neue Tabellen erstellen müssen.  

#### Verwendung dieses Musters  

Verwenden Sie dieses Muster, wenn Sie eine große Anzahl von Entitäten haben, die gleichzeitig gelöscht werden müssen.  

#### Zugehörige Muster und Anleitungen  

Die folgenden Muster und Anleitungen können auch relevant sein, wenn dieses Muster implementiert wird:  

-	[Entitätsgruppentransaktionen](#entity-group-transactions)
-	[Ändern von Entitäten](#working-with-heterogeneous-entity-types)  

### Datenreihenmuster  

Speichern Sie vollständige Datenreihen in einer einzelnen Entität, um die Anzahl der von Ihnen vorgenommenen Anforderungen zu minimieren.  

#### Kontext und Problem  

Ein häufiges Szenario für eine Anwendung ist, eine Datenreihe zu speichern, die in der Regel auf einmal abgerufen werden muss. Beispiel: Ihre Anwendung erfasst, wie viele IM-Meldungen jeder Mitarbeiter pro Stunde sendet. Diese Informationen verwenden Sie dann, um zu zeichnen, wie viele Meldungen jeder Benutzer in den vergangenen 24 Stunden gesendet hat. Ein Entwurf könnte sein, für jeden Mitarbeiter 24 Entitäten zu speichern:  

![][22]
 
Bei diesem Entwurf können Sie problemlos die Entität finden und aktualisieren, um sie für jeden Mitarbeiter immer dann zu aktualisieren, wenn die Anwendung den Meldungszählerwert aktualisieren muss. Allerdings müssen Sie zum Abrufen der Informationen, mit der ein Diagramm der Aktivität für die vergangenen 24 Stunden zu zeichnen ist, 24 Entitäten abrufen.  

#### Lösung  

Verwenden Sie den folgenden Entwurf mit einer separaten Eigenschaft,um den Meldungszähler für jede Stunde zu speichern:  

![][23]
 
Bei diesem Entwurf können Sie mit einem Zusammenführungsvorgang den Meldungszähler für einen Mitarbeiter für eine bestimmte Stunde aktualisieren. Nun können Sie alle Informationen, die Sie zum Zeichnen des Diagramms benötigen, mit einer Anforderung für eine einzelne Entität abrufen.  

#### Probleme und Überlegungen  

Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:  
-	Wenn eine vollständige Datenreihe nicht in eine einzelne Entität passt (eine Entität kann bis zu 252 Eigenschaften haben), verwenden Sie einen alternativen Datenspeicher, z. B einen Blob.  
-	Wenn bei Ihnen mehrere Clients gleichzeitig eine Entität aktualisieren, müssen Sie den **ETag** verwenden, um vollständige optimistische Nebenläufigkeit zu implementieren. Wenn Sie viele Clients haben, können eine Vielzahl von Konflikten auftreten.  

#### Verwendung dieses Musters  

Verwenden Sie dieses Muster, wenn Sie eine Datenreihe aktualisieren und abrufen müssen, die einer einzelnen Entität zugeordnet ist.  

#### Zugehörige Muster und Anleitungen  

Die folgenden Muster und Anleitungen können auch relevant sein, wenn dieses Muster implementiert wird:  

-	[Muster für große Entität](#large-entity-pattern)  
-	[Zusammenführen oder ersetzen](#working-with-heterogeneous-entity-types)  
-	[Eventual Consistency Transaktionsmuster](#eventually-consistent-transactions-pattern) (wenn Sie die Datenreihe in einem Blob speichern)  

### Breite Entitätenmuster  

Verwenden Sie mehrere physische Entitäten, um logische Entitäten mit mehr als 252 Eigenschaften zu speichern.  

#### Kontext und Problem  

Eine einzelne Entität kann nicht mehr als 252 Eigenschaften (mit Ausnahme der obligatorischen Systemeigenschaften) beinhalten und nicht mehr als 1 MB Daten insgesamt speichern. In einer relationalen Datenbank würden Sie in der Regel die Grenzen umgehen können bezüglich der Größe einer Zeile, indem Sie eine neue Tabelle hinzufügen und eine 1:1-Beziehung erzwingen.  

#### Lösung  

Durch Verwendung des Tabellenspeicherdienstes können Sie mehrere Entitäten speichern, um ein einzelnes großes Geschäftsobjekt mit mehr als 252 Eigenschaften darzustellen. Beispiel: Wenn Sie die Anzahl der IM-Meldungen, die von jedem Mitarbeiter in den letzten 365 Tagen gesendet wurde, speichern möchten, können Sie den folgenden Entwurf verwenden, der zwei Entitäten mit unterschiedlichen Schemas verwendet:  

![][24]
 
Falls Sie eine Änderung vornehmen müssen, die eine Aktualisierung beider Entitäten erfordert, um sie synchron zu halten, können Sie ein EGT verwenden. Ansonsten können Sie einen einzelnen Zusammenführungsvorgang verwenden, um den Meldungszähler für einen bestimmten Tag zu aktualisieren. Zum Abruf aller Daten für einen einzelnen Mitarbeiter müssen Sie beide Entitäten abrufen, was Sie mit zwei effizienten Anforderungen durchführen können, die beide einen **PartitionKey**- und einen **RowKey**-Wert verwenden.  

#### Probleme und Überlegungen  

Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:  

-	Der Abruf einer vollständigen logischen Entität umfasst mindestens zwei Speichertransaktionen, jeweils eine zum Abrufen der einzelnen physischen Entitäten.  

#### Verwendung dieses Musters  

Verwenden Sie dieses Muster, wenn Sie Entitäten speichern müssen, deren Größe oder Anzahl an Eigenschaften die Grenzwerte für eine einzelne Entität im Tabellenspeicherdienst überschreiten.  

#### Zugehörige Muster und Anleitungen  

Die folgenden Muster und Anleitungen können auch relevant sein, wenn dieses Muster implementiert wird:  

-	[Entitätsgruppentransaktionen](#entity-group-transactions)
-	[Zusammenführen oder ersetzen](#working-with-heterogeneous-entity-types)

### Muster für große Entitäten  

Verwenden Sie Blob-Speicher zum Speichern großer Eigenschaftswerte.  

#### Kontext und Problem  

Eine einzelne Entität kann nicht mehr als 1 MB Daten insgesamt speichern. Wenn eine oder mehrere Ihrer Eigenschaften Werte speichern, die dazu führen, dass die Gesamtgröße der Entität diesen Wert überschreitet, können Sie nicht die gesamte Entität im Tabellenspeicherdienst speichern.  

#### Lösung  

Wenn Ihre Entität 1 MB Größe überschreitet, da eine oder mehrere Eigenschaften eine große Datenmenge enthalten, können Sie die Daten im Blob-Dienst und anschließend die Adresse des BLOBs in einer Eigenschaft der Entität speichern. Beispiel: Sie können das Foto des Mitarbeiters im Blob-Speicher und eine Verknüpfung zum Photo in der Eigenschaft **Photo** der Mitarbeiterentität speichern:  

![][25] 

#### Probleme und Überlegungen  

Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:  

-	Zur Verwaltung von Eventual Consistency zwischen der Entität im Tabellenspeicherdienst und den Daten im Blob-Dienst verwenden Sie das [Eventual Consistency Transaktionsmuster](#eventually-consistent-transactions-pattern) zur Verwaltung Ihrer Entitäten.    
-	Das Abrufen einer vollständigen Entität umfasst mindestens zwei Speichertransaktionen: eine zum Abrufen der Entität und eine zum Abrufen der Blob-Daten.  

#### Verwendung dieses Musters  

Verwenden Sie dieses Muster, wenn Sie Entitäten speichern müssen, deren Größe die Grenzwerte für eine einzelne Entität im Tabellenspeicherdienst überschreitet.  

#### Zugehörige Muster und Anleitungen  

Die folgenden Muster und Anleitungen können auch relevant sein, wenn dieses Muster implementiert wird:  

-	[Eventual Consistency Transaktionsmuster](#eventually-consistent-transactions-pattern)  
-	[Breite Entitätenmuster](#large-entity-pattern)

### Antimuster voranstellen/anfügen  

Erhöhen Sie die Skalierbarkeit, wenn Sie eine große Anzahl von Einfügungen vornehmen möchten, indem Sie die Einfügungen auf mehrere Partitionen verteilen.  

#### Kontext und Problem  

Voranstellen oder Anfügen von Entitäten an Ihre gespeicherten Entitäten führen in der Regel dazu, dass in der Anwendung neue Entitäten auf die erste oder letzte Partition aus einer Folge von Partitionen hinzugefügt werden. In diesem Fall finden alle Einfügungen zu einem beliebigen Zeitpunkt in der gleichen Partition statt und erzeugen einen Hotspot, der Lastenausgleich-Einfügungen durch den Tabellenspeicherdienst für mehrere Knoten verhindert und wahrscheinlich dazu führt, dass Ihre Anwendung die Skalierbarkeitsziele der Partition erreicht. Beispiel: Wenn Sie eine Anwendung haben, die Zugriffe auf Netzwerk und Ressourcen von Mitarbeitern protokolliert, könnte eine wie unten gezeigte Entitätsstruktur dazu führen, dass die der aktuellen Stunde zugehörigen Partition zum Hotspot wird, wenn das Transaktionsvolumen das Skalierbarkeitsziel für eine einzelne Partition erreicht:  

![][26]
 
#### Lösung  

Die folgende alternative Entitätsstruktur vermeidet einen Hotspot auf einer bestimmten Partition, wenn die Anwendung Ereignisse protokolliert:  

![][27]

Beachten Sie, dass in diesem Beispiel sowohl der **PartitionKey** als auch der **RowKey** zusammengesetzte Schlüssel sind. Der **PartitionKey** verwendet sowohl Abteilung als auch Mitarbeiter-ID, um die Protokollierung auf mehrere Partitionen zu verteilen.  

#### Probleme und Überlegungen  

Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:  

-	Unterstützt die alternative Schlüssel-Struktur, die das Erstellen von Hot-Partitionen bei Einfügungen vermeidet, die Abfragen effizient, die von der Clientanwendung vorgenommen werden?  
-	Bedeutet die erwartete Anzahl von Transaktionen, dass Sie wahrscheinlich die Skalierbarkeitsziele für eine einzelne Partition erreichen und vom Speicherdienst gedrosselt werden?  

#### Verwendung dieses Musters  

Vermeiden Sie die Voranstellen/Anfügen-Antimuster, wenn die Anzahl an Transaktionen wahrscheinlich dazu führen wird, dass beim Zugriff auf eine Hot-Partition vom Speicherdienst eine Drosselung erfolgt.  

#### Zugehörige Muster und Anleitungen  

Die folgenden Muster und Anleitungen können auch relevant sein, wenn dieses Muster implementiert wird:  

-	[Zusammengesetzte Schlüsselmuster](#compound-key-pattern)  
-	[Log Tail-Muster](#log-tail-pattern)  
-	[Ändern von Entitäten](#working-with-heterogeneous-entity-types)  

### Log-Anti-Muster  

Sie sollten in der Regel den Blob-Dienst anstelle des Tabellenspeicherdienstes zum Speichern von Protokolldaten verwenden.  

#### Kontext und Problem  

Eine allgemeiner Anwendungsfall für Protokolldaten ist, eine Auswahl von Protokolleinträgen für einen bestimmten Datum/Uhrzeitbereich abzurufen. Beispielsweise finden Sie alle Fehler und kritische Meldungen, die Ihre Anwendung zwischen 15:04 und 15:06 an einem bestimmten Datum protokolliert hat. Es empfiehlt sich nicht, das Datum und die Uhrzeit der Protokollmeldung zu verwenden, um die Partition zu bestimmen, in die Protokolleinträge gespeichert werden sollen. Dies resultiert in einer Hot-Partition, da jederzeit alle Protokolleinträge denselben **PartitionKey**-Wert gemeinsam nutzen (Informationen finden Sie im Abschnitt [Antimuster voranstellen/anfügen](#prepend-append-anti-pattern)). Beispiel: Das folgende Entitätsschema für eine Protokollmeldung resultiert in einer Hot-Partition, da die Anwendung alle Protokollmeldungen in die Partition für das aktuelle Datum und Stunde schreibt:  

![][28]
 
In diesem Beispiel beinhaltet **RowKey** Datum und Uhrzeit der Protokollmeldung, um sicherzustellen, dass die Protokollmeldungen nach Datum und Uhrzeit sortiert gespeichert sind, sowie eine Meldungs-ID für den Fall, dass mehrere Protokollmeldungen das Datum und die Uhrzeit freigeben.  

Ein anderer Ansatz ist die Verwendung von **PartitionKey**, der sicherstellt, dass die Anwendung Meldungen über eine Bereich von Partitionen schreibt. Beispiel: Wenn die Quelle der Protokollmeldung einen Weg bietet, Meldungen über viele Partitionen zu verteilen, können Sie das folgende Entitätsschema verwenden:  

![][29]
 
Das Problem mit diesem Schema ist jedoch, dass Sie zum Abrufen der gesamten Protokollmeldungen für eine bestimmte Zeitspanne jede Partition in der Tabelle durchsuchen müssen.

#### Lösung  

Der vorherige Abschnitt hat das Problem bei dem Versuch hervorgehoben, den Tabellenspeicherdienst zum Speichern von Protokolleinträgen zu verwenden und schlug zwei nicht zufriedenstellende Entwürfe vor. Eine Lösung führte zu einer Hot-Partition mit dem Risiko einer schlechten Leistung beim Schreiben von Protokollmeldungen. Die andere Lösung führte zu einer schlechten Leistung bei Abfragen aufgrund der Anforderung, jede Partition in der Tabelle zum Abrufen von Protokollmeldungen für eine bestimmte Zeitspanne zu scannen. BLOB-Speicher bietet eine bessere Lösung für diese Art Szenario und dies ist die Funktionsweise, wie Azure Storage Analytics die gesammelten Protokolldaten speichert.  

In diesem Abschnitt wird erläutert, wie Storage Analytics Protokolldaten im Blob-Speicher speichert, als anschauliches Beispiel zu diesem Ansatz, um Daten zu speichern, die Sie in der Regel nach Bereich abfragen.  

Storage Analytics speichert Protokollmeldungen in einem Trennzeichen-getrennten Format in mehrere Blobs. Das Trennzeichen-getrennte Format erleichtert es einer Clientanwendung, die Daten in der Protokollmeldung zu analysieren.  

Storage Analytics verwendet eine Namenskonvention für Blobs, die es Ihnen ermöglicht, einen Blob (oder Blobs) zu finden, der die Protokollmeldungen enthält, nach denen Sie suchen. Beispiel: Ein Blob mit dem Namen "queue/2014/07/31/1800/000001.log" enthält die Protokollmeldungen, die dem Warteschlangendienst für die Stunde zugeordnet sind, beginnend um 18:00 Uhr am 31. Juli 2014. Die "000001" zeigt an, dass dies die erste Protokolldatei für diesen Zeitraum ist. Storage Analytics zeichnet auch die Zeitstempel der Protokollmeldungen von Vor- und Nachname an, die in der Datei als Teil der Blob-Metadaten gespeichert sind. Die API für Blob-Speicher ermöglicht Ihnen, Blobs in einem Container zu finden, die auf einem Namenspräfix basieren. Um alle Blobs zu finden, die Protokolldaten für die Warteschlange für die Stunde beinhalten, die um 18:00 Uhr beginnt, können Sie das Präfix "queue/2014/07/31/1800" verwenden."  

Storage Analytics puffert Protokollmeldungen intern, und aktualisiert dann den geeigneten Blob oder erstellt einen neuen Blob mit dem neuesten Satz Protokolleinträge. Dies reduziert die Anzahl der Schreibvorgänge, die für den Blob-Dienst ausgeführt werden müssen.  

Wenn Sie eine ähnliche Lösung in Ihre eigene Anwendung implementieren, müssen Sie berücksichtigen, wie Sie den Kompromiss zwischen Zuverlässigkeit (sofortiges Schreiben jedes Protokolleintrags in den Blob-Speicher) und Kosten und Skalierbarkeit (Pufferung der Aktualisierungen in Ihrer Anwendung und Schreiben im Stapel zum Blob-Speicher) verwalten.  

#### Probleme und Überlegungen  

Beachten Sie die folgenden Punkte bei der Entscheidung, wie Sie Protokolldaten speichern:  

-	Wenn Sie einen Tabellenentwurf erstellen, der potenzielle Hot-Partitionen vermeidet, werden Sie feststellen, dass Sie auf Ihre Protokolldaten nicht effizient zugreifen können.  
-	Zur Verarbeitung von Protokolldaten muss ein Client häufig viele Datensätze laden.  
-	Obwohl Protokolldaten häufig strukturiert sind, ist Blob-Speicher möglicherweise eine bessere Lösung.  

### Überlegungen zur Implementierung  

Dieser Abschnitt beschreibt einige Überlegungen, die Sie berücksichtigen sollten, wenn Sie die in den vorherigen Abschnitten beschriebenen Muster implementieren. Im größten Teil dieses Abschnitts werden Beispiele verwendet, die in C# geschrieben sind und die Storage Client Library (Version 4.3.0 zum Redaktionszeitpunkt) verwendet.  

### Abrufen von Entitäten  

Wie im Abschnitt [Entwurf für Abfragen](#design-for-querying)" erläutert, ist die effizienteste Abfrage eine Punktabfrage. In einigen Szenarien müssen Sie jedoch mehrere Entitäten abrufen. Dieser Abschnitt beschreibt einige allgemeine Ansätze zum Abrufen von Entitäten mithilfe der Storage Client Library.  

#### Ausführen einer Punktabfrage mithilfe der Storage Client Library  

Die einfachste Möglichkeit zum Ausführen einer Punktabfrage ist die Verwendung des Tabellenvorgangs **Abrufen**, wie im folgenden C#-Codeausschnitt gezeigt, der eine Entität mit dem **PartitionKey**-Wert "Sales" und dem **RowKey**-Wert "212" abruft:  

	TableOperation retrieveOperation = 
		TableOperation.Retrieve<EmployeeEntity>("Sales", "212");
	var retrieveResult = employeeTable.Execute(retrieveOperation);
	if (retrieveResult.Result != null)
	{
    EmployeeEntity employee = (EmployeeEntity)retrieveResult.Result;
    ...
	}  

Beachten Sie, wie in diesem Beispiel von der abgerufenen Entität erwartet wird, dass sie vom Typ **EmployeeEntity** ist.  

#### Abrufen von mehreren Entitäten mithilfe von LINQ  

Sie können mehrere Entitäten abrufen, indem Sie LINQ mit Storage Client Library verwenden und eine Abfrage mit einer **WHERE**-Klausel angeben. Um einen Tabellen-Scan zu vermeiden, sollten Sie immer den **PartitionKey**-Wert in die WHERE-Klausel mit aufnehmen, sowie möglichst auch den **RowKey**-Wert, um Tabellen- und Partition-Scans zu vermeiden. Der Tabellenspeicherdienst unterstützt eine begrenzte Anzahl von Vergleichsoperatoren (größer als, größer als oder gleich, kleiner als, kleiner als oder gleich, gleich und ungleich) zur Verwendung in der WHERE-Klausel. Der folgende C#-Codeausschnitt findet alle Mitarbeiter, deren Nachname mit "B" beginnt (vorausgesetzt, dass in **RowKey** der Nachname gespeichert ist) und in der Vertriebsabteilung ist (vorausgesetzt dass in **PartitionKey** der Name der Abteilung gespeichert ist):  

	TableQuery<EmployeeEntity> employeeQuery = 
  			employeeTable.CreateQuery<EmployeeEntity>();
	var query = (from employee in employeeQuery
                where employee.PartitionKey == "Sales" &&
                employee.RowKey.CompareTo("B") >= 0 &&
                employee.RowKey.CompareTo("C") < 0
                select employee).AsTableQuery();
	var employees = query.Execute();  

Beachten Sie, wie die Abfrage sowohl einen **RowKey** als auch einen **PartitionKey** festlegt, um eine bessere Leistung sicherzustellen.  

Das folgende Codebeispiel zeigt die entsprechende Funktionalität mithilfe der Fluent-API (weitere Informationen über Fluent-APIs im Allgemeinen finden Sie unter [Bewährte Methoden zum Entwurf einer Fluent-API](http://visualstudiomagazine.com/articles/2013/12/01/best-practices-for-designing-a-fluent-api.aspx)):  

	TableQuery<EmployeeEntity> employeeQuery = new TableQuery<EmployeeEntity>().Where(
 	 TableQuery.CombineFilters(
    	TableQuery.CombineFilters(
      	TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales"),
      TableOperators.And,
      TableQuery.GenerateFilterCondition(
        "RowKey", QueryComparisons.GreaterThanOrEqual, "B")
    ),
    TableOperators.And,
    TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.LessThan, "C")
 	 )
	);
	var employees = employeeTable.ExecuteQuery(employeeQuery);  


>[AZURE.NOTE] Das Beispiel verschachtelt mehrere **CombineFilters**-Methoden, um die drei Filterbedingungen einschließen zu können.  

#### Abrufen einer großen Anzahl von Entitäten aus einer Abfrage  

Eine optimale Abfrage gibt eine einzelne Entität zurück, basierend auf einem **PartitionKey**-Wert und einem **RowKey**-Wert. In einigen Szenarien haben Sie jedoch möglicherweise eine Anforderung, um viele Entitäten von der gleichen Partition oder sogar von vielen Partitionen zurückzugeben.  

In solchen Szenarien sollten Sie die Leistung Ihrer Anwendung immer vollständig testen.  

Eine Abfrage für den Tabellenspeicherdienst kann maximal 1000 Entitäten gleichzeitig zurückgeben und für maximal fünf Sekunden ausgeführt werden. Wenn der Ergebnissatz mehr als 1.000 Entitäten enthält, wenn die Abfrage nicht innerhalb von fünf Sekunden abgeschlossen wurde oder die Abfrage die Partitionsgrenzen überschreitet, gibt der Tabellenspeicherdienst ein Fortsetzungstoken zurück, das der Clientanwendung ermöglicht, den nächsten Satz Entitäten anzufordern. Weitere Informationen über die Funktionsweise von Fortsetzungstokens finden Sie unter [Abfragetimeout und Paginierung](http://msdn.microsoft.com/library/azure/dd135718.aspx) auf MSDN.  

Bei Verwendung der Storage Client Library kann diese automatisch Fortsetzungstoken für Sie handhaben, da es Entitäten aus dem Tabellenspeicherdienst zurückgibt. Das folgende C#-Codebeispiel verwendet die Storage Client Library und handhabt automatisch Fortsetzungstoken, wenn er vom Tabellenspeicherdienst in der Antwort zurückgegeben wird:  

	string filter = TableQuery.GenerateFilterCondition(
  		"PartitionKey", QueryComparisons.Equal, "Sales");
	TableQuery<EmployeeEntity> employeeQuery =
  		new TableQuery<EmployeeEntity>().Where(filter);

	var employees = employeeTable.ExecuteQuery(employeeQuery);
	foreach (var emp in employees)
	{
  		...
	}  

Der folgende C#-Code handhabt Fortsetzungstoken explizit:  

	string filter = TableQuery.GenerateFilterCondition(
  		"PartitionKey", QueryComparisons.Equal, "Sales");
	TableQuery<EmployeeEntity> employeeQuery =
  		new TableQuery<EmployeeEntity>().Where(filter);

	TableContinuationToken continuationToken = null;

	do
	{
  		var employees = employeeTable.ExecuteQuerySegmented(
    		employeeQuery, continuationToken);
  	foreach (var emp in employees)
  	{
   	 ...
  	}
  	continuationToken = employees.ContinuationToken;
	} while (continuationToken != null);  

Durch die explizite Verwendung von Fortsetzungstoken können Sie steuern, wann Ihre Anwendung das nächste Datensegment abruft. Beispiel: Wenn Ihre Clientanwendung Benutzer in die Lage versetzt, durch die Entitäten zu blättern, die in einer Tabelle gespeichert sind, kann ein Benutzer sich entscheiden, nicht durch alle über die Abfrage abgerufenen Entitäten zu blättern. Somit würde Ihre Anwendung nur einen Fortsetzungstoken verwenden, um das nächste Segment abzurufen, wenn der Benutzer mit dem Blättern durch alle Entitäten im aktuellen Segment fertig ist. Dieser Ansatz hat mehrere Vorteile:  

-	Er versetzt Sie in die Lage, die Datenmenge für den Abruf vom Tabellenspeicherdienst zu begrenzen, die sie über das Netzwerk verschieben.  
-	Er versetzt Sie in die Lage, asynchrone I/O in .NET auszuführen.  
-	Sie können den Fortsetzungstoken in einen permanenten Speicher serialisieren, damit bei einem Absturz der Anwendung fortgesetzt werden kann.  

>[AZURE.NOTE] Ein Fortsetzungstoken gibt in der Regel ein Segment mit 1.000 Entitäten oder weniger zurück. Dies ist auch der Fall, wenn Sie die Anzahl der Rückgabe-Einträge einer Abfrage begrenzen, indem Sie **Nehmen** verwenden, um die ersten n Entitäten zurückzugeben, die mit Ihren Suchkriterien übereinstimmen. Der Tabellenspeicherdienst kann ein Segment zurückgeben, das weniger als n Entitäten beinhaltet, sowie einen Fortsetzungstoken, mit dem Sie die restlichen Entitäten abrufen können.  

Der folgende C#-Code zeigt, wie die Anzahl der in einem Segment zurückgegebenen Entitäten geändert werden kann:  

	employeeQuery.TakeCount = 50;  

#### Serverseitige Projektion  

Eine einzelne Entität kann bis zu 255 Eigenschaften haben und bis zu 1 MB groß sein. Wenn Sie die Tabelle durchsuchen und Entitäten abrufen, benötigen Sie möglicherweise nicht alle Eigenschaften und Sie können unnötiges Übertragen von Daten vermeiden (trägt zum Verringern der Latenz und Kosten bei). Sie können auch eine serverseitige Projektion verwenden, um nur die benötigten Eigenschaften zu übertragen. Im folgende Beispiel wird nur die **E-Mail**-Eigenschaft abgerufen (zusammen mit **PartitionKey**, **RowKey**, **Zeitstempel** und **ETag**) aus den Entitäten, die von der Abfrage ausgewählt sind.  

	string filter = TableQuery.GenerateFilterCondition(
  		"PartitionKey", QueryComparisons.Equal, "Sales");
	List<string> columns = new List<string>() { "Email" };
	TableQuery<EmployeeEntity> employeeQuery =
  		new TableQuery<EmployeeEntity>().Where(filter).Select(columns);

	var entities = employeeTable.ExecuteQuery(employeeQuery);
	foreach (var e in entities)
	{
  		Console.WriteLine("RowKey: {0}, EmployeeEmail: {1}", e.RowKey, e.Email);
	}  

Beachten Sie, dass der **RowKey**-Wert auch dann verfügbar ist, wenn er nicht in der Liste der abzurufenden Eigenschaften enthalten war.  

### Ändern von Entitäten  

Die Storage Client Library versetzt Sie in die Lage, Ihre im Tabellenspeicherdienst gespeicherte Entitäten durch Einfügen, Löschen und Aktualisieren zu ändern. Sie können EGTs verwenden, um mehrere Einfüge-, Aktualisierungs- und Löschvorgänge zu stapeln, um die Zahl der benötigten Roundtrips zu reduzieren und die Leistung Ihrer Lösung zu verbessern.  

Beachten Sie, dass Ausnahmen ausgelöst werden, wenn die Storage Client Library eine EGT ausführt, da diese den Index der Entität beinhaltet, welche einen Fehler bei der Stapelverarbeitung verursacht hat. Dies ist hilfreich, wenn Sie Code debuggen, der EGTs verwendet.  

Sie sollten auch berücksichtigen, wie Ihr Design beeinflusst wird und wie Ihre Anwendung Nebenläufigkeit und Aktualisierungsvorgänge handhabt.  

#### Verwalten von Nebenläufigkeit  

Der Dienst implementiert standardmäßig Prüfungen der optimistischen Nebenläufigkeit auf der Ebene der einzelnen Entitäten für die Vorgänge **Einfügen**, **Zusammenführen** und **Löschen**, obwohl es für einen Client möglich ist, den Tabellenspeicherdienst zu zwingen, diese Prüfungen zu umgehen. Weitere Informationen zur Verwaltung von Nebenläufigkeit durch den Tabellenspeicherdienst finden Sie unter [Verwalten von Nebenläufigkeit in Microsoft Azure Storage](storage-concurrency.md) auf der Microsoft Azure-Website.  

#### Zusammenführen oder ersetzen  

Die **Ersetzen**-Methode der **TableOperation**-Klasse ersetzt immer die vollständige Entität im Tabellenspeicherdienst. Wenn Sie keine Eigenschaft in die Anforderung einschließen und diese Eigenschaft in der gespeicherten Entität existiert, entfernt die Anforderung diese Eigenschaft von der gespeicherten Entität. Sie müssen alle Eigenschaften in die Anforderung mit einschließen, wenn Sie nicht möchten, dass eine Eigenschaft aus einer gespeicherten Entität explizit entfernen wird.  

Sie können die **Zusammenführen**-Methode der **TableOperation**-Klasse verwenden, um die Datenmenge zu reduzieren, die Sie an den Tabellenspeicherdienst senden, wenn Sie eine Entität aktualisieren möchten. Die **Zusammenführen**-Methode ersetzt alle Eigenschaften in der gespeicherten Entität mit Eigenschaftswerten aus der Entität, die in der Anforderung enthalten sind, belässt aber alle Eigenschaften in der gespeicherten Entität, die in der Anforderung nicht enthalten sind. Dies ist hilfreich, wenn Sie große Entitäten haben und nur eine kleine Anzahl von Eigenschaften in einer Anforderung aktualisieren müssen.  

>[AZURE.NOTE] Die **Ersetzen**- und **Zusammenführen**-Methoden schlagen fehl, wenn die Entität nicht vorhanden ist. Als Alternative können Sie die **InsertOrReplace**- und **InsertOrMerge**-Methoden verwenden, die eine neue Entität erstellen, wenn sie nicht vorhanden ist.  

### Arbeiten mit heterogenen Entitätstypen  

Der Tabellenspeicherdienst ist ein *schema-less*-Tabellenspeicher, was bedeutet, dass eine einzelne Tabelle Entitäten mit mehreren Typen speichern kann, sodass Ihr Entwurf von einem hohen Maß an Flexibilität profitiert. Das folgende Beispiel veranschaulicht eine Tabelle, in der sowohl Mitarbeiter- als auch Abteilungsentitäten gespeichert sind:  

<table>
<tr>
<th>Partitionsschlüssel</th>
<th>Zeilenschlüssel</th>
<th>Timestamp</th>
<th></th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>Vorname</th>
<th>Nachname</th>
<th>Alter</th>
<th>E-Mail</th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>Vorname</th>
<th>Nachname</th>
<th>Alter</th>
<th>E-Mail</th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>Abteilungsname</th>
<th>EmployeeCount</th>
</tr>
<tr>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>Vorname</th>
<th>Nachname</th>
<th>Alter</th>
<th>E-Mail</th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
</table>	 	 	 

Beachten Sie, dass jede Entität trotzdem über **PartitionKey**-, **RowKey**-, und **Zeitstempel**-Werte, verfügen muss, aber eine beliebige Anzahl an Eigenschaften haben kann. Darüber hinaus gibt es nichts, was den Typ einer Entität anzeigen kann, es sei denn, Sie entscheiden sich dafür, diese Information irgendwo zu speichern. Es gibt zwei Optionen für die Identifizierung des Entitätstyps:  

-	Voranstellen des Entitätstyps an **RowKey** (oder möglicherweise an **PartitionKey**). Beispielsweise **EMPLOYEE_000123** oder **DEPARTMENT_SALES** als **RowKey**-Werte.  
-	Verwenden Sie eine separate Eigenschaft zum Aufzeichnen des Entitätstyps, wie in der folgenden Tabelle dargestellt.  

<table>
<tr>
<th>Partitionsschlüssel</th>
<th>Zeilenschlüssel</th>
<th>Timestamp</th>
<th></th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>Vorname</th>
<th>Nachname</th>
<th>Alter</th>
<th>E-Mail</th>
</tr>
<tr>
<td>Mitarbeiter</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>Vorname</th>
<th>Nachname</th>
<th>Alter</th>
<th>E-Mail</th>
</tr>
<tr>
<td>Mitarbeiter</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>Abteilungsname</th>
<th>EmployeeCount</th>
</tr>
<tr>
<td>Abteilung</td>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>Vorname</th>
<th>Nachname</th>
<th>Alter</th>
<th>E-Mail</th>
</tr>
<tr>
<td>Mitarbeiter</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
</table>	 	 	 	 

Die erste Option mit dem voranstellen Entitätstyp an **RowKey** ist sinnvoll, wenn eine Möglichkeit besteht, dass zwei Entitäten mit unterschiedlichen Typen über denselben Schlüsselwert verfügen können. Hier werden auch Entitäten des gleichen Typs in der Partition zusammen gruppiert.  
 
Die in diesem Abschnitt beschriebenen Verfahren sind besonders wichtig für die Diskussion über [Vererbungsbeziehungen](#inheritance-relationships) weiter oben in diesem Handbuch im Abschnitt [Modellieren von Beziehungen](#modelling-relationships).  

>[AZURE.NOTE] Sie sollten die Angabe einer Versionsnummer im Wert des Entitätstyps berücksichtigen, um der Clientanwendung zu ermöglichen, POCO-Objekte zu entwickeln und mit verschiedenen Versionen zu arbeiten.  

Im restlichen Teil dieses Abschnitts werden einige Funktionen der Storage Client Library beschrieben, mit denen die Arbeit mit mehreren Entitätstypen in derselben Tabelle erleichtert wird.  

#### Abrufen von heterogenen Entitätstypen  

Bei Verwendung der Storage Client Library stehen Ihnen drei Optionen für die Arbeit mit mehreren Entitätstypen zur Verfügung.  

Wenn Sie den Entitätstyp kennen, der mit bestimmten **RowKey**- und **PartitionKey**-Werten gespeichert ist, können Sie den Entitätstyp angeben, wenn Sie die Entität wie in den vorherigen beiden Beispielen dargestellt abrufen, die Entitäten des Typs **EmployeeEntity** abrufen: [Abrufen einer einzelnen Entität mithilfe der Speicherclientbibliothek](#retrieving-a-single-entity-using-the-storage-client-library) und [Abrufen von mehreren Entitäten mithilfe von LINQ](#retrieving-multiple-entities-using-linq).  

Die zweite Option ist die Verwendung des **DynamicTableEntity**-Typs (ein Eigenschaftenbehälter) anstelle eines konkreten POCO-Entitätstyps (diese Option kann auch die Leistung verbessern, da keine Notwendigkeit zum Serialisieren und Deserialisieren der Entität besteht). Der folgende C#-Code ist ein möglicher Abruf von mehreren Entitäten mit unterschiedlichen Typen aus der Tabelle, gibt jedoch alle Entitäten als **DynamicTableEntity**-Instanzen zurück. Anschließend wird mithilfe der **EventType**-Eigenschaft der Typ der einzelnen Entitäten bestimmt:  

	string filter = 	TableQuery.CombineFilters(
    	TableQuery.GenerateFilterCondition("PartitionKey",
      QueryComparisons.Equal, "Sales"),
    	TableOperators.And,
    	TableQuery.CombineFilters(
        TableQuery.GenerateFilterCondition("RowKey", 
          			QueryComparisons.GreaterThanOrEqual, "B"),
        	TableOperators.And,
        	TableQuery.GenerateFilterCondition("RowKey", 
          QueryComparisons.LessThan, "F")
        )
    );
	TableQuery<DynamicTableEntity> entityQuery =
  	new TableQuery<DynamicTableEntity>().Where(filter);
	var employees = employeeTable.ExecuteQuery(entityQuery);

	IEnumerable<DynamicTableEntity> entities = employeeTable.ExecuteQuery(entityQuery);
	foreach (var e in entities)
	{
    EntityProperty eventTypeProperty;
    if (e.Properties.TryGetValue("EntityType", out eventTypeProperty))
    {
        if (eventTypeProperty.StringValue == "Employee")
        {
            // Use eventTypeProperty, RowKey, PartitionKey, Etag, and Timestamp
      	  }
   	 }
	}  

Beachten Sie, dass Sie zum Abruf anderer Eigenschaften die **TryGetValue**-Methode für die **Eigenschaften**-Eigenschaft der **DynamicTableEntity**-Klasse verwenden müssen.  

Eine dritte Möglichkeit ist, den **DynamicTableEntity**-Typ und eine **EntityResolver**-Instanz in Kombination zu verwenden. Dadurch können Sie mehrere POCO-Typen in der gleichen Abfrage auflösen. In diesem Beispiel verwendet der **EntityResolver**-Delegat die **EventType**-Eigenschaft, um zwischen den beiden Typen der Entität zu unterscheiden, die von der Abfrage zurückgegeben werden. Die **Beheben** -Methode verwendet den **Konfliktlöser**-Delegaten zum Auflösen der **DynamicTableEntity**-Instanzen in **TableEntity**-Instanzen.  

	EntityResolver<TableEntity> resolver = (pk, rk, ts, props, etag) =>
	{
  
  		TableEntity resolvedEntity = null;
  		if (props["EntityType"].StringValue == "Department")
  		{
    		resolvedEntity = new DepartmentEntity();
  		}
  		else if (props["EntityType"].StringValue == "Employee")
  		{
    		resolvedEntity = new EmployeeEntity();
  		}
  		else throw new ArgumentException("Unrecognized entity", "props");

  		resolvedEntity.PartitionKey = pk;
  		resolvedEntity.RowKey = rk;
  		resolvedEntity.Timestamp = ts;
  		resolvedEntity.ETag = etag;
  		resolvedEntity.ReadEntity(props, null);
  		return resolvedEntity;
	};

	string filter = TableQuery.GenerateFilterCondition(
  		"PartitionKey", QueryComparisons.Equal, "Sales");
	TableQuery<DynamicTableEntity> entityQuery =
  		new TableQuery<DynamicTableEntity>().Where(filter); 

	var entities = employeeTable.ExecuteQuery(entityQuery, resolver); 
	foreach (var e in entities)
	{
  		if (e is DepartmentEntity)
  		{
    	...
  		}
  		if (e is EmployeeEntity)
  		{
    	...
  		}
	}  

#### Ändern von heterogenen Entitätstypen  

Zum Löschen einer Entität müssen Sie den Typ wissen. Sie wissen den Typ einer Entität immer, wenn sie ihn Einfügen. Sie können jedoch den **DynamicTableEntity**-Typ zum Aktualisieren einer Entität verwenden, ohne dessen Typ zu wissen und ohne eine POCO-Entitätsklasse zu verwenden. Im folgenden Codebeispiel wird eine einzelne Entität abgerufen und überprüft, ob die **EmployeeCount**-Eigenschaft vorhanden ist, bevor sie aktualisiert wird.  

	TableResult result = 
  		employeeTable.Execute(TableOperation.Retrieve(partitionKey, rowKey));
	DynamicTableEntity department = (DynamicTableEntity)result.Result;

	EntityProperty countProperty;

	if (!department.Properties.TryGetValue("EmployeeCount", out countProperty))
	{
  		throw new 
    		InvalidOperationException("Invalid entity, EmployeeCount property not found.");
	}
	countProperty.Int32Value += 1;
	employeeTable.Execute(TableOperation.Merge(department));  

### Steuern des Zugriffs mit Shared Access Signature  

Sie können Shared Access Signature (SAS)-Token verwenden, um Clientanwendungen zu ermöglichen, Tabellenentitäten direkt zu ändern (und abzufragen)ohne die Notwendigkeit, direkt mit dem Tabellenspeicherdienst zu authentifizieren. Es gibt in der Regel drei Hauptvorteile, wenn Sie SAS in Ihrer Anwendung verwenden:  

-	Sie müssen Ihren Speicherkontoschlüssel nicht auf eine unsichere Plattform (z. B. ein mobiles Gerät) verteilen, um auf das Gerät zuzugreifen und Entitäten im Tabellenspeicherdienst zu ändern.  
-	Sie können einige der Aufgaben auslagern, die Web- und Workerrollen beim Verwalten von Entitäten in Client-Geräten durchführen, z. B. für Computer der Endbenutzer und mobile Geräte.  
-	Sie können einen eingeschränkten und einen zeitbegrenzten Satz von Berechtigungen für einen Client zuweisen (z. B. den schreibgeschützten Zugriff auf bestimmte Ressourcen).  

Weitere Informationen zur Verwendung von SAS-Token mit dem Tabellenspeicherdienst finden Sie unter [Shared Access Signature, Teil 1: Grundlagen zum SAS-Modell](storage-dotnet-shared-access-signature-part-1.md).  

Sie müssen jedoch weiterhin die SAS-Token generieren, die eine Clientanwendung für die Entitäten im Tabellenspeicherdienst gewähren. Sie sollten dies in einer Umgebung tun, die sicheren Zugriff auf Ihre Speicherkontoschlüssel gewährleistet. In der Regel verwenden Sie eine Web- oder Worker-Rolle zum Generieren der SAS-Token und übermittelt sie an die Client-Anwendungen, die Zugriff auf Ihre Entitäten benötigen. Da immer noch Aufwand zum Generieren und Bereitstellen von SAS-Token für Clients notwendig ist, sollten Sie erwägen, wie Sie diesen Aufwand am besten reduzieren, besonders in Szenarien mit großen Datenmengen.  

Es ist möglich, einen SAS-Token zu generieren, der Zugriff auf eine Teilmenge der Entitäten in einer Tabelle erteilt. Standardmäßig erstellen Sie ein SAS-Token für die gesamte Tabelle, aber es ist auch möglich anzugeben, dass dem SAS-Token entweder Zugriff auf einen Bereich von **PartitionKey**-Werten oder einen Bereich von **PartitionKey**- und **RowKey**-Werten erteilt wird. Sie können auswählen, ob Sie SAS-Token für einzelne Benutzer des Systems in einer Art generieren, das jedes Benutzer-SAS-Token nur Zugriff auf ihre eigenen Entitäten im Tabellenspeicherdienst ermöglicht.  

### Asynchrone und parallele Vorgänge  

Angenommen, Sie verteilen Ihre Anfragen über mehrere Partitionen. In diesem Fall können Sie Durchsatz und Client-Reaktionsfähigkeit mithilfe von asynchronen oder parallelen Abfragen verbessern. 
Beispiel: Sie haben möglicherweise zwei oder mehr Workerrollen-Instanzen, die auf die Tabellen parallel zugreifen. Sie können einzelne Worker-Rollen haben, die für bestimmte Gruppen von Partitionen zuständig sind oder einfach mehrere Workerrollen-Instanzen, von denen jede auf alle Partitionen in einer Tabelle zugreifen kann.  

Innerhalb einer Client-Instanz können Sie den Durchsatz verbessern, indem Sie Speichervorgänge asynchron ausführen. Die Storage Client Library erleichtert das Schreiben von asynchronen Abfragen und Änderungen. Beispiel: Sie können mit der synchronen Methode starten, mit der alle Entitäten in einer Partition wie im folgenden C#-Code gezeigt abgerufen werden:  

	private static void ManyEntitiesQuery(CloudTable employeeTable, string department)
	{
  		string filter = TableQuery.GenerateFilterCondition(
    		"PartitionKey", QueryComparisons.Equal, department);
  		TableQuery<EmployeeEntity> employeeQuery =
    		new TableQuery<EmployeeEntity>().Where(filter); 

  		TableContinuationToken continuationToken = null;

  		do
  		{
    		var employees = employeeTable.ExecuteQuerySegmented(
      			employeeQuery, continuationToken); 
    		foreach (var emp in employees)
    	{
      	...
    	}
    		continuationToken = employees.ContinuationToken;
  		} while (continuationToken != null);
	}  

Sie können diesen Code einfach ändern, damit die Abfrage wie folgt asynchron ausgeführt wird:  

	private static async Task ManyEntitiesQueryAsync(CloudTable employeeTable, string department)
	{
  		string filter = TableQuery.GenerateFilterCondition(
    		"PartitionKey", QueryComparisons.Equal, department);
  		TableQuery<EmployeeEntity> employeeQuery =
    		new TableQuery<EmployeeEntity>().Where(filter); 
  		TableContinuationToken continuationToken = null;

  		do
  		{
    		var employees = await employeeTable.ExecuteQuerySegmentedAsync(
      			employeeQuery, continuationToken); 
    		foreach (var emp in employees)
    		{
     		 ...
    		}
    		continuationToken = employees.ContinuationToken;
  			} while (continuationToken != null);
	}  

In diesem Asynchron-Beispiel sehen Sie die folgenden Änderungen zur synchronen Version:  

-	Die Methodensignatur enthält jetzt den **Async**-Modifizierer und gibt eine Instanz **Aufgabe** zurück.  
-	Statt die **ExecuteSegmented**-Methode abzurufen, ruft die Methode jetzt die **ExecuteSegmentedAsync**-Methode auf und verwendet den **await**-Modifizierer, um Ergebnisse asynchron abrufen.  

Die Clientanwendung kann diese Methode mehrmals aufrufen (mit unterschiedlichen Werten für die Parameter zu **Abteilung**). Damit wird jede Abfrage in einem separaten Thread ausgeführt.  

Beachten Sie, dass es keine asynchrone Version der Methode **Ausführen** in der **TableQuery**-Klasse gibt, da die **IEnumerable**-Schnittstelle keine asynchrone Enumeration unterstützt.  

Sie können Entitäten auch asynchron einfügen, aktualisieren und löschen. Das folgende C#-Beispiel zeigt eine einfache, synchrone Methode zum Einfügen oder Ersetzen einer Mitarbeiterentität:  

	private static void SimpleEmployeeUpsert(CloudTable employeeTable,
  		EmployeeEntity employee)
	{
  		TableResult result = employeeTable
    		.Execute(TableOperation.InsertOrReplace(employee));
  		Console.WriteLine("HTTP Status: {0}", result.HttpStatusCode);
	}  

You can easily modify this code so that the update runs asynchronously as follows:  

	private static async Task SimpleEmployeeUpsertAsync(CloudTable employeeTable, 
  		EmployeeEntity employee)
	{
  		TableResult result = await employeeTable
    		.ExecuteAsync(TableOperation.InsertOrReplace(employee));
  		Console.WriteLine("HTTP Status: {0}", result.HttpStatusCode);
	}  

In diesem Asynchron-Beispiel sehen Sie die folgenden Änderungen zur synchronen Version:  

-	Die Methodensignatur enthält jetzt den **Async**-Modifizierer und gibt eine Instanz **Aufgabe** zurück.  
-	Statt die **Ausführen**-Methode aufzurufen, um die Entität zu aktualisieren, ruft die Methode jetzt die **ExecuteAsync**-Methode auf und verwendet den **await**-Modifizierer, um Ergebnisse asynchron abrufen.  

Die Clientanwendung kann mehrere asynchrone Methoden wie diese aufrufen und jeder Methodenaufruf wird auf einem separaten Thread ausgeführt.  


### Guthaben
Wir möchten den folgenden Mitgliedern der Azure-Teams für ihre Beiträge danken: Dominic Betts, Jason Hogg, Jean Ghanem, Jai Haridas, Jeff Irwin, Vamshidhar Kommineni, Vinay Shah und Serdar Ozler.  

Wir möchten auch den folgenden Microsoft-MVPs für ihre wertvollen Rückmeldungen zu den Reviews danken: Igor Papirov und Edward Bakker. 


 
[1]: ./media/storage-table-design-guide/storage-table-design-IMAGE01.png
[2]: ./media/storage-table-design-guide/storage-table-design-IMAGE02.png
[3]: ./media/storage-table-design-guide/storage-table-design-IMAGE03.png
[4]: ./media/storage-table-design-guide/storage-table-design-IMAGE04.png
[5]: ./media/storage-table-design-guide/storage-table-design-IMAGE05.png
[6]: ./media/storage-table-design-guide/storage-table-design-IMAGE06.png
[7]: ./media/storage-table-design-guide/storage-table-design-IMAGE07.png
[8]: ./media/storage-table-design-guide/storage-table-design-IMAGE08.png
[9]: ./media/storage-table-design-guide/storage-table-design-IMAGE09.png
[10]: ./media/storage-table-design-guide/storage-table-design-IMAGE10.png
[11]: ./media/storage-table-design-guide/storage-table-design-IMAGE11.png
[12]: ./media/storage-table-design-guide/storage-table-design-IMAGE12.png
[13]: ./media/storage-table-design-guide/storage-table-design-IMAGE13.png
[14]: ./media/storage-table-design-guide/storage-table-design-IMAGE14.png
[15]: ./media/storage-table-design-guide/storage-table-design-IMAGE15.png
[16]: ./media/storage-table-design-guide/storage-table-design-IMAGE16.png
[17]: ./media/storage-table-design-guide/storage-table-design-IMAGE17.png
[18]: ./media/storage-table-design-guide/storage-table-design-IMAGE18.png
[19]: ./media/storage-table-design-guide/storage-table-design-IMAGE19.png
[20]: ./media/storage-table-design-guide/storage-table-design-IMAGE20.png
[21]: ./media/storage-table-design-guide/storage-table-design-IMAGE21.png
[22]: ./media/storage-table-design-guide/storage-table-design-IMAGE22.png
[23]: ./media/storage-table-design-guide/storage-table-design-IMAGE23.png
[24]: ./media/storage-table-design-guide/storage-table-design-IMAGE24.png
[25]: ./media/storage-table-design-guide/storage-table-design-IMAGE25.png
[26]: ./media/storage-table-design-guide/storage-table-design-IMAGE26.png
[27]: ./media/storage-table-design-guide/storage-table-design-IMAGE27.png
[28]: ./media/storage-table-design-guide/storage-table-design-IMAGE28.png
[29]: ./media/storage-table-design-guide/storage-table-design-IMAGE29.png

<!--HONumber=47-->
 