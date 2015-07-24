<properties
   pageTitle="Verwenden von Azure Data Factory mit SQL Data Warehouse | Microsoft Azure"
   description="Tipps für die Verwendung von Azure Data Factory (ADF) mit Azure SQL Data Warehouse für die Entwicklung von Lösungen."
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="06/22/2015"
   ms.author="lodipalm"/>

# Verwenden von Azure Data Factory mit SQL Data Warehouse

Azure Data Factory bietet eine vollständig verwaltete Methode zur Orchestrierung der Datenübertragung und zur Ausführung von gespeicherten Prozeduren in SQL Data Warehouse. Dadurch können Sie auf einfachere Weise komplexe Extraktions-, Umwandlungs- und Ladeprozeduren (Extract/Transform/Load, ETL) mit SQL Data Warehouse einrichten und planen. Eine vollständige Übersicht über Azure Data Factory finden Sie in der [Azure Data Factory-Dokumentation][].

## Datenverschiebung 

Azure Data Factory ermöglicht eine Datenverschiebung zwischen lokalen Datenquellen und anderen Azure-Diensten. Die aktuelle Azure Data Factory-Integration unterstützt Datenverschiebung in die folgende Speicherorte und aus diesen:

+ Azure Blob Storage
+ Azure SQL-Datenbank
+ Lokaler SQL Server
+ SQL Server unter IaaS

Informationen zum Einrichten einer Datenkopieraktivität finden Sie unter [Kopieren von Daten mit Azure Data Factory][].

> [AZURE.NOTE]Zum aktuellen Zeitpunkt kann Azure Data Factory nicht zum Übertragen von Daten in Spalten, die keine Nullwerte zulassen, in SQL Data Warehouse verwendet werden.

## Gespeicherte Prozeduren
 Auf die gleiche Weise wie beim Planen von Datenübertragungen kann Azure Data Factory auch zum Orchestrieren der Ausführung von gespeicherten Prozeduren verwendet werden. Dies ermöglicht die Erstellung komplexerer Pipelines und erweitert die Möglichkeiten von Azure Data Factory, sodass die Rechenleistung von SQL Data Warehouse genutzt wird.

## Nächste Schritte
Einen Überblick über die Integration finden Sie unter [SQL Data Warehouse-Integration (Übersicht)][]. Weitere Hinweise zur Entwicklung finden Sie in der [SQL Data Warehouse-Entwicklungsübersicht][].

<!--Image references-->

<!--Article references-->

[Kopieren von Daten mit Azure Data Factory]: ./data-factory-copy-activity/
[SQL Data Warehouse-Entwicklungsübersicht]: ./sql-data-warehouse-overview-develop/
[SQL Data Warehouse-Integration (Übersicht)]: ./sql-data-warehouse-overview-integration/

<!--MSDN references-->

<!--Other Web references-->
[Azure Data Factory-Dokumentation]: https://azure.microsoft.com/documentation/services/data-factory/

<!---HONumber=July15_HO1-->