﻿<properties 
    pageTitle="Herstellen einer Verbindung zwischen DocumentDB und Azure Search unter Verwendung von Indexern | Azure" 
    description="Dieser Artikel veranschaulicht die Verwendung des Azure-Such-Indexers mit DocumentDB als Datenquelle."
    services="documentdb" 
    documentationCenter="" 
    authors="aliuy" 
    manager="jhubbard" 
    editor="mimig"/>

<tags 
    ms.service="documentdb" 
    ms.devlang="rest-api" 
    ms.topic="article" 
    ms.tgt_pltfrm="NA" 
    ms.workload="data-services" 
    ms.date="03/19/2015" 
    ms.author="andrl"/>

#Herstellen einer Verbindung zwischen DocumentDB und Azure Search unter Verwendung von Indexern

Wenn Sie ein hervorragendes Suchverlaufsverhalten in Bezug auf DocumentDB-Daten implementieren möchten, verwenden Sie den Azure Search Indexer für DocumentDB! In diesem Artikel erfahren Sie, wie Sie Azure DocumentDB mit der Azure-Suche integrieren, ohne dass Sie Code zum Beibehalten der Indexdienst-Infrastruktur schreiben müssen!

Zur Realisierung müssen Sie [ein Azure-Suchkonto einrichten](search-get-started.md#start-with-the-free-service) (Sie müssen kein Upgrade auf die Standardsuche durchführen) und anschließend den [Azure-Such-REST-API aufrufen](https://msdn.microsoft.com/library/azure/dn798935.aspx), um eine DocumentDB-**Datenquelle** und einen **Indexer** für diese Datenquelle zu erstellen.

##<a id="Concepts"></a>Azure Search Indexer-Konzepte

Azure Search unterstützt die Erstellung und Verwaltung von Datenquellen (einschließlich DocumentDB) und von Indexern, die gegenläufig zu diesen Datenquellen fungieren.

Eine **Datenquelle** gibt an, welche Daten indiziert werden müssen, und sie legt die Anmeldeinformationen für den Zugriff auf die Daten sowie die Richtlinien zur Azure-Suchaktivierung fest, um Änderungen an den Daten effizient identifizieren zu können (wie z. B. geänderte oder gelöschte Dokumente in der Auflistung). Die Datenquelle wird als unabhängige Ressource definiert, sodass sie von mehreren Indexern verwendet werden kann.

Ein **Indexer** beschreibt, wie die Daten von der Datenquelle in einen Zielsuchindex fließen. Sie sollten jeweils einen Indexer pro Kombination aus Zielindex und Datenquelle erstellen. Während mehrere Indexer zwar in denselben Index schreiben können, kann ein Indexer nur in einen einzigen Index schreiben. Ein Indexer wird für Folgendes verwendet: 

- Eine einmalige Kopie der Daten zum Auffüllen eines Indexes ausführen.
- Einen Index mit Änderungen an der Datenquelle nach einem Zeitplan synchronisieren. Der Zeitplan ist Teil der Indexer-Definition.
- Bedarfs-Updates für einen Index je nach Notwendigkeit abrufen. 

##<a id="CreateDataSource"></a>Schritt 1: Erstellen einer Datenquelle

Stellen Sie eine HTTP POST-Anforderung zum Erstellen einer neuen Datenquelle im Azure-Suchdienst einschließlich der folgenden Anforderungsheader aus.
    
    POST https://[Search service name].search.windows.net/datasources?api-version=[api-version]
    Content-Type: application/json
    api-key: [Search service admin key]

Die `api-version` ist erforderlich. Gültige Werte sind `2015-02-28` oder eine höhere Version.

Der Anforderungstext umfasst die Datenquellendefinition, welche die folgenden Felder enthalten sollte:

- **name**: Der Name der Datenquelle.

- **Typ**: Verwenden Sie `documentdb`.

- **Anmeldeinformationen**:

    - **ConnectionString**: Erforderlich. Geben Sie die Verbindungsinformationen zur Azure DocumentDB-Datenbank im folgenden Format an: `AccountEndpoint=<DocumentDB endpoint url>;AccountKey=<DocumentDB auth key>;Database=<DocumentDB database id>`

- **Container**:

    - **name**: Erforderlich. Geben Sie die zu indizierende DocumentDB-Auflistung an. 

    - **Abfrage**: Optional. Sie können eine Abfrage spezifizieren, um ein beliebiges JSON-Dokument in ein Flatfile-Schema zu reduzieren, welches Azure Search indizieren kann.

- **dataChangeDetectionPolicy**: Optional. Siehe [Richtlinie zur Erkennung von Datenänderungen](#DataChangeDetectionPolicy) weiter unten.

- **dataDeletionDetectionPolicy**: Optional. Siehe [Richtlinie zur Erkennung von Datenlöschungen](#DataDeletionDetectionPolicy) weiter unten.

###<a id="DataChangeDetectionPolicy"></a>Erfassen von geänderten Dokumenten

Der Zweck der Richtlinie der Datenänderungserkennung besteht darin, die geänderten Datenelemente effizient identifizieren zu können. Zurzeit ist `High Water Mark` die einzige unterstützte Richtlinie, die die von DocumentDB bereitgestellte `_ts`-Eigenschaft für den Zeitstempel der letzten Verwendung nutzt. Diese wird folgendermaßen festgelegt:

    { 
        "@odata.type" : "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
        "highWaterMarkColumnName" : "_ts" 
    } 

Sie müssen `_ts` außerdem in der Projektion und der `WHERE`-Klausel für Ihre Abfrage hinzufügen. Beispiel:

    SELECT s.id, s.Title, s.Abstract, s._ts FROM Sessions s WHERE s._ts > @HighWaterMark


###<a id="DataDeletionDetectionPolicy"></a>Erfassen von gelöschten Dokumenten

Wenn Zeilen aus der Quelltabelle gelöscht werden, sollten Sie diese Zeilen auch aus dem Suchindex löschen. Die Richtlinie einer Datenlöschungserkennung dient einer effizienten Identifizierung gelöschter Datenelemente. Zurzeit ist `Soft Delete` die einzige unterstützte Richtlinie (die Löschung wird durch ein bestimmtes Kennzeichen markiert). Diese wird folgendermaßen festgelegt:

    { 
        "@odata.type" : "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
        "softDeleteColumnName" : "the property that specifies whether a document was deleted", 
        "softDeleteMarkerValue" : "the value that identifies a document as deleted" 
    }

> [AZURE.NOTE] Sie müssen die Eigenschaft in Ihre SELECT-Klausel einbeziehen, wenn Sie eine benutzerdefinierte Projektion verwenden.

###<a id="CreateDataSourceExample"></a>Beispiel für einen Anforderungstext

Anhand des folgenden Beispiels wird eine Datenquelle mit einer benutzerdefinierten Abfrage und Richtlinienhinweisen erstellt:

    {
        "name": "mydocdbdatasource",
        "type": "documentdb",
        "credentials": {
            "connectionString": "AccountEndpoint=https://myDocDbEndpoint.documents.azure.com;AccountKey=myDocDbAuthKey;Database=myDocDbDatabaseId"
        },
        "container": {
            "name": "myDocDbCollectionId",
            "query": "SELECT s.id, s.Title, s.Abstract, s._ts FROM Sessions s WHERE s._ts > @HighWaterMark" 
        },
        "dataChangeDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
            "highWaterMarkColumnName": "_ts"
        },
        "dataDeletionDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
            "softDeleteColumnName": "isDeleted",
            "softDeleteMarkerValue": "true"
        }
    }

###Antwort

Sie erhalten die Antwort "HTTP 201 Created", wenn die Datenquelle erfolgreich erstellt wurde.

##<a id="CreateIndex"></a>Schritt 2: Erstellen eines Index

Erstellen Sie einen Azure Search-Zielindex, wenn Sie bislang noch über keinen verfügen. Sie können dies von der [Azure-Portal-Benutzeroberfläche](search-get-started.md#test-service-operations) aus tun, oder indem Sie [Index-API erstellen](https://msdn.microsoft.com/library/azure/dn798941.aspx) verwenden.

	POST https://[Search service name].search.windows.net/indexes?api-version=[api-version]
	Content-Type: application/json
	api-key: [Search service admin key]


Stellen Sie sicher, dass das Schema des Ziel-Indexes mit dem Schema der JSON-Quelldokumente oder mit der Ausgabe Ihrer benutzerdefinierten Abfrageprojektion kompatibel ist.

###Abbildung A: Zuordnung zwischen JSON-Datentypen und Azure Search-Datentypen

<table style="font-size:12">
    <tr>
        <th>JSON-Datentyp</th>
        <th>Kompatible Ziel-Index-Feldtypen</th>
    </tr>
    <tr>
        <td>Bool</td>
        <td>Edm.Boolean, Edm.String</td>
    </tr>
    <tr>
        <td>Zahlen, die wie Ganzzahlen aussehen</td>
        <td>Edm.Int32, Edm.Int64, Edm.String</td>
    </tr>
    <tr>
        <td>Zahlen, die wie Gleitkommas aussehen</td>
        <td>Edm.Double, Edm.String</td>
    </tr>
    <tr>
        <td>String</td>
        <td>Edm.String</td>
    </tr>
    <tr>
        <td>Arrays primitiver Typen, z.B. [ "a", "b", "c" ]</td>
        <td>Collection(Edm.String)</td>
    </tr>
    <tr>
        <td>Zeichenfolgen, die wie Datumsangaben aussehen</td>
        <td>Edm.DateTimeOffset, Edm.String</td>
    </tr>
    <tr>
        <td>JSON-Objekte</td>
        <td>N/V</td>
    </tr>
</table>

###<a id="CreateIndexExample"></a>Beispiel für einen Anforderungstext

Anhand des folgenden Beispiels wird ein Index mit einer ID und einem Beschreibungsfeld erstellt:

    {
       "name": "mysearchindex",
       "fields": [{
         "name": "id",
         "type": "Edm.String",
         "key": true,
         "searchable": false
       }, {
         "name": "description",
         "type": "Edm.String",
         "filterable": false,
         "sortable": false,
         "facetable": false,
         "suggestions": true
       }]
     }

###Antwort

Sie erhalten die Antwort "HTTP 201 Created", wenn der Index erfolgreich erstellt wurde.

##<a id="CreateIndexer"></a>Schritt 3: Erstellen eines Indexers

Sie können einen neuen Indexer innerhalb des Azure-Suchdiensts mithilfe einer HTTP POST-Anforderung mit den folgenden Headern erstellen.
    
    POST https://[Search service name].search.windows.net/indexers?api-version=[api-version]
    Content-Type: application/json
    api-key: [Search service admin key]

Der Anforderungstext umfasst die Indexerdefinition, welche die folgenden Felder enthalten sollte:

- **name**: Erforderlich. Der Name des Indexers.

- **dataSourceName**: Erforderlich. Der Name einer vorhandenen Datenquelle.

- **targetIndexName**: Erforderlich. Der Name eines vorhandenen Indexes.

- **Zeitplan**: Optional. Siehe [Indizierungszeitplan](#IndexingSchedule) weiter unten.

###<a id="IndexingSchedule"></a>Ausführen von Indexern nach einem Zeitplan

Ein Indexer kann optional einen Zeitplan angeben. Wenn ein Zeitplan vorliegt, wird der Indexer regelmäßig gemäß Zeitplan ausgeführt. Der Zeitplan besitzt die folgenden Attribute:

- **Intervall**: Erforderlich. Ein Zeitdauerwert, der ein Intervall oder den Zeitraum für Indexer-Ausführungen angibt. Das kleinste zulässige Intervall beträgt 5 Minuten. Das längste ist ein Tag. Es muss als XSD-Wert "dayTimeDuration" formatiert sein (eine eingeschränkte Teilmenge eines [ISO 8601](http://www.w3.org/TR/xmlschema11-2/#dayTimeDuration)-Zeitdauerwerts). Das Muster hierfür lautet: `P[nD][T[nH][nM]]`. Examples: `PT15M` für ein 15-Minuten-Intervall, `PT2H` für ein 2-Stunden-Intervall. 

- **startTime**: Erforderlich. Ein UTC-DateTime-Wert, der angibt, wann die Ausführung des Indexers beginnen soll. 

###<a id="CreateIndexerExample"></a>Beispiel für einen Anforderungstext

Anhand des folgenden Beispiels wird ein Indexer erstellt. Dieser kopiert Daten aus der Sammlung, die von der Datenquelle `myDocDbDataSource` referenziert wird, nach einem Zeitplan in den Index `mySearchIndex`. Der Zeitplan beginnt am 1. Jan. 2015 UTC und wird stündlich ausgeführt.

    {
        "name" : "mysearchindexer",
        "dataSourceName" : "mydocdbdatasource",
        "targetIndexName" : "mysearchindex",
        "schedule" : { "interval" : "PT1H", "startTime" : "2015-01-01T00:00:00Z" }
    }

###Antwort

Sie erhalten die Antwort "HTTP 201 Created", wenn der Indexer erfolgreich erstellt wurde.

##<a id="RunIndexer"></a>Schritt 4: Ausführen eines Indexers

Zusätzlich zur Ausführung in regelmäßigen Abständen nach einem Zeitplan kann ein Indexer bei Bedarf auch aufgerufen werden, indem die folgende HTTP POST-Anforderung ausgegeben wird: 

    POST https://[Search service name].search.windows.net/indexers/[indexer name]/run?api-version=[api-version]
    api-key: [Search service admin key]

###Antwort

Sie erhalten die Antwort "HTTP 202 Accepted", wenn der Indexer erfolgreich aufgerufen wurde.

##<a name="GetIndexerStatus"></a>Schritt 5: Abrufen des Indexer-Status

Sie können eine HTTP GET-Anforderung ausgeben, um den aktuellen Status und den Ausführungsverlauf eines Indexers abzurufen: 

    GET https://[Search service name].search.windows.net/indexers/[indexer name]/status?api-version=[api-version]
    api-key: [Search service admin key]

###Antwort

Sie erhalten die Antwort "HTTP 200 OK", die zusammen mit einem Antworttext zurückgegeben wird. Dieser enthält Informationen über den Gesamtintegritätsstatus des Indexers, den letzten Indexer-Aufruf, sowie den Verlauf der jüngsten Indexer-Aufrufe (sofern vorhanden). 

Die Antwort sollte etwa wie folgt aussehen:

    {
        "status":"running",
        "lastResult": {
            "status":"success",
            "errorMessage":null,
            "startTime":"2014-11-26T03:37:18.853Z",
            "endTime":"2014-11-26T03:37:19.012Z",
            "errors":[],
            "itemsProcessed":11,
            "itemsFailed":0,
            "initialTrackingState":null,
            "finalTrackingState":null
         },
        "executionHistory":[ {
            "status":"success",
             "errorMessage":null,
            "startTime":"2014-11-26T03:37:18.853Z",
            "endTime":"2014-11-26T03:37:19.012Z",
            "errors":[],
            "itemsProcessed":11,
            "itemsFailed":0,
            "initialTrackingState":null,
            "finalTrackingState":null
        }]
    }

Der Ausführungsverlauf enthält bis zu 50 der jüngsten abgeschlossenen Ausführungen. Diese sind in umgekehrter chronologischer Reihenfolge sortiert (somit ist die neueste Ausführung als Erstes in der Antwort aufgelistet).

##<a name="NextSteps"></a>Nächste Schritte

Glückwunsch! Sie wissen nun, wie Azure DocumentDB mit Azure Search unter Verwendung des Indexers für DocumentDB integriert wird.

 - Um weitere Informationen zu Azure DocumentDB zu erhalten, klicken Sie [hier](/services/documentdb/).

 - Um weitere Informationen zu Azure Search zu erhalten, klicken Sie [hier](/services/search/).

<!--HONumber=49-->