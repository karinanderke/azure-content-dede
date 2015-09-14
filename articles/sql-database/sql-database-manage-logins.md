<properties
   pageTitle="Verwalten von Datenbanken und Anmeldungen in Azure SQL-Datenbank | Microsoft Azure"
	description="Es wird beschrieben, wie Sie den Serverebenenprinzipal und andere Konten verwenden können, um in SQL-Datenbank Anmeldungen und Datenbanken zu verwalten."
	services="sql-database"
	documentationCenter=""
	authors="BYHAM"
	manager="jeffreyg"
	editor=""
	tags=""/>

<tags
   ms.service="sql-database"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="data-management"
	ms.date="08/20/2015"
	ms.author="rickbyh"/>

# Verwalten von Datenbanken und Anmeldungen in Azure SQL-Datenbank

Wenn Sie sich in Microsoft Azure SQL-Datenbank für den Dienst registrieren, werden vom Bereitstellungsprozess ein Azure SQL-Datenbankserver, eine Datenbank mit dem Namen **master** und eine Anmeldung erstellt, bei der es sich um den Serverebenenprinzipal Ihres Azure SQL-Datenbank-Servers handelt. Diese Anmeldung ähnelt dem Prinzipal auf Serverebene (**sa**) für eine lokale Instanz von SQL Server.

Das Serverebenenprinzipal-Konto von Azure SQL-Datenbank verfügt immer über die Berechtigung, die gesamte Sicherheit auf Server- und Datenbankebene zu verwalten. In diesem Thema wird beschrieben, wie Sie den Serverebenenprinzipal und andere Konten verwenden können, um in SQL-Datenbank Anmeldungen und Datenbanken zu verwalten.

> [AZURE.IMPORTANT]Mit SQL-Datenbank V12 können sich Benutzer gegenüber der Datenbank authentifizieren, indem sie Benutzer für eigenständige Datenbanken verwenden. Benutzer für eigenständige Datenbanken benötigen keine Anmeldungen. Dies verbessert die Portabilität von Datenbanken, reduziert aber auch die Fähigkeit des Serverebenenprinzipals, den Zugriff auf die Datenbank zu steuern. Das Aktivieren von Benutzern für eigenständige Datenbanken ist mit wichtigen Auswirkungen auf die Sicherheit verbunden. Weitere Informationen finden Sie unter [Eigenständige Datenbankbenutzer - machen Sie Ihre Datenbank portabel](https://msdn.microsoft.com/library/ff929188.aspx), [CREATE USER (Transact-SQL)](https://msdn.microsoft.com/library/ms173463.aspx) und [Eigenständige Datenbanken](https://msdn.microsoft.com/library/ff929071.aspx).

## Übersicht über die Verwaltung der Sicherheit von SQL-Datenbank

Die Sicherheitsverwaltung in SQL-Datenbank ähnelt der Sicherheitsverwaltung für eine lokale Instanz von SQL Server. Das Verwalten der Sicherheit auf der Datenbankebene ist nahezu identisch. Es bestehen lediglich Unterschiede bei den verfügbaren Parametern. Da SQL-Datenbanken auf einen oder mehrere physische Computer skaliert werden können, verwendet Azure SQL-Datenbank für die Verwaltung auf Serverebene eine andere Strategie. In der folgenden Tabelle ist zusammengefasst, inwiefern sich die Sicherheitsverwaltung für eine lokale Instanz von SQL Server gegenüber Azure SQL-Datenbank unterscheidet.

| Unterschied | Lokaler SQL Server | Azure SQL-Datenbank |
|------------------------------------------------|-----------------------------------------------------------------------------|--------------------------------------------------|
| Verwalten der Sicherheit auf Serverebene | Ordner „Sicherheit“ im Objekt-Explorer von SQL Server Management Studio | Masterdatenbank und per Azure-Portal |
| Sicherheitsrolle auf Serverebene zum Erstellen von Anmeldungen | Feste Serverrolle „securityadmin“ – Weitere Informationen unter „Serverebenenrollen“ | Datenbankrolle „loginmanager“ in der Masterdatenbank |
| Befehle zum Verwalten von Benutzernamen | CREATE LOGIN, ALTER LOGIN, DROP LOGIN | CREATE LOGIN, ALTER LOGIN, DROP LOGIN (Es gelten einige Parametereinschränkungen, und es muss eine Verbindung mit der Masterdatenbank bestehen.) |
| Ansicht mit allen Anmeldungen | sys.server\_principals | sys.sql\_logins (Es muss eine Verbindung mit der Masterdatenbank bestehen.)|
| Serverebenenrolle zum Erstellen von Datenbanken | Feste Datenbankrolle „dbcreator“ Weitere Informationen unter „Serverebenenrollen“ | Datenbankrolle „dbmanager“ in der Masterdatenbank |
| Befehl zum Erstellen einer Datenbank | CREATE DATABASE | CREATE DATABASE (Es gelten einige Parametereinschränkungen, und es muss eine Verbindung mit der Masterdatenbank bestehen.) |
| Ansicht mit allen Datenbanken | sys.databases | sys.databases (Es muss eine Verbindung mit der Masterdatenbank bestehen.) |

### Verwaltung auf Serverebene und Masterdatenbank

Ihr Azure SQL-Datenbank-Server ist eine Abstraktion, bei der eine Gruppierung von Datenbanken definiert wird. Datenbanken, die Ihrem Azure SQL-Datenbank-Server zugeordnet sind, können sich im Microsoft-Rechenzentrum auf separaten physischen Computern befinden. Führen Sie die Verwaltung auf Serverebene für alle Datenbanken durch, indem Sie eine Einzeldatenbank mit dem Namen „master“ verwenden.

Die Masterdatenbank überwacht alle Anmeldungen und verfolgt, welche Anmeldungen zum Erstellen von Datenbanken oder anderen Anmeldungen berechtigt sind. Es muss immer eine Verbindung mit der Masterdatenbank bestehen, wenn Sie Anmeldungen oder Datenbanken erstellen, ändern oder verwerfen. Die Datenbank **master** verfügt auch über die Ansichten ``sys.sql_logins`` und ``sys.databases``, mit denen Sie die Anmeldungen bzw. Datenbanken anzeigen können.

> [AZURE.NOTE]Der Befehl ``USE`` zum Wechseln zwischen Datenbanken wird nicht unterstützt. Stellen Sie direkt eine Verbindung mit der Zieldatenbank her.

Sie können die Sicherheit auf Datenbankebene für Benutzer und Objekte in Microsoft Azure SQL-Datenbank genauso verwalten, wie Sie dies für eine lokale Instanz von SQL Server durchführen. Unterschiede bestehen nur in Bezug auf die Parameter, die für die entsprechenden Befehle verfügbar sind. Weitere Informationen finden Sie unter [Sicherheitsrichtlinien und Einschränkungen von Azure SQL-Datenbank](https://msdn.microsoft.com/library/azure/ff394108.aspx).

### Verwalten von Anmeldungen

Verwalten Sie Anmeldungen mit der Serverebenenprinzipal-Anmeldung, indem Sie eine Verbindung mit der Masterdatenbank herstellen. Sie können die Anweisungen ``CREATE LOGIN``, ``ALTER LOGIN`` oder ``DROP LOGIN`` verwenden. Im folgenden Beispiel wird eine Anmeldung mit dem Namen **login1** erstellt:

```
-- first, connect to the master database
CREATE LOGIN login1 WITH password='<ProvidePassword>';
```

> [AZURE.NOTE]Sie müssen beim Erstellen einer Anmeldung ein sicheres Kennwort verwenden. Weitere Informationen finden Sie unter [Sichere Kennwörter](https://msdn.microsoft.com/library/ms161962.aspx).

#### Verwenden neuer Anmeldungen

Um mit den von Ihnen erstellten Anmeldungen eine Verbindung mit Microsoft Azure SQL-Datenbank herzustellen, müssen Sie jeder Anmeldung zuerst mit dem Befehl ``CREATE USER`` Berechtigungen auf Datenbankebene gewähren. Weitere Informationen finden Sie unter „Gewähren von Berechtigungen auf Datenbankebene für eine Anmeldung“.

Da einige Tools Tabular Data Stream (TDS) unterschiedlich implementieren, müssen Sie den Namen des Azure SQL-Datenbank-Servers ggf. in der Verbindungszeichenfolge an die Anmeldung anfügen, indem Sie die Notation ``<login>@<server>`` verwenden. Trennen Sie die Anmeldung und den Namen des Azure SQL-Datenbank-Servers mit dem Symbol ``@`` voneinander. Wenn die Anmeldung beispielsweise **login1** heißt und der vollqualifizierte Name Ihres Azure SQL-Datenbank-Servers **servername.database.windows.net** lautet, sollte der username-Parameter der Verbindungszeichenfolge wie folgt lauten: ****login1@servername**. Der Text, den Sie für den Anmeldenamen auswählen können, wird hierdurch also eingeschränkt. Weitere Informationen finden Sie unter [CREATE LOGIN (Transact-SQL)](https://msdn.microsoft.com/library/ms189751.aspx).

## Gewähren von Berechtigungen auf Serverebene für eine Anmeldung

Damit die Sicherheit auf Serverebene von anderen Anmeldungen als dem Serverebenenprinzipal verwaltet werden kann, verfügt Microsoft Azure SQL-Datenbank über zwei Sicherheitsrollen: **loginmanager** zum Erstellen von Anmeldungen und **dbmanager** zum Erstellen von Datenbanken. Diesen Datenbankrollen können nur Benutzer in der Datenbank **master** hinzugefügt werden.

> [AZURE.NOTE]Zum Erstellen von Anmeldungen oder Datenbanken müssen Sie mit der Datenbank **master** verbunden sein (einer logischen Darstellung von **master**).

### Rolle „loginmanager“

Wie die feste Serverrolle **securityadmin** für eine lokale Instanz von SQL Server auch, verfügt die Datenbankrolle **loginmanager** in Microsoft Azure SQL-Datenbank über die Berechtigung zum Erstellen von Anmeldungen. Nur mit der Serverebenenprinzipal-Anmeldung (per Bereitstellungsprozess erstellt) oder von Mitgliedern der Datenbankrolle **loginmanager** können neue Anmeldungen erstellt werden.

### Rolle „dbmanager“

Die Datenbankrolle **dbmanager** von Microsoft Azure SQL-Datenbank ähnelt der festen Serverrolle **dbcreator** für eine lokale Instanz von SQL Server. Nur mit der Serverebenenprinzipal-Anmeldung (per Bereitstellungsprozess erstellt) oder von Mitgliedern der Datenbankrolle **dbmanager** können Datenbanken erstellt werden. Sobald ein Benutzer Mitglied der Datenbankrolle **dbmanager** ist, kann er eine Datenbank mit dem Befehl ``CREATE DATABASE`` von Azure SQL-Datenbank erstellen, aber dieser Befehl muss in der Masterdatenbank ausgeführt werden. Weitere Informationen finden Sie unter [CREATE DATABASE (SQL Server Transact-SQL)](https://msdn.microsoft.com/library/ms176061.aspx).

### Zuweisen von SQL-Datenbank-Serverebenenrollen

Führen Sie die folgenden Schritte aus, um eine Anmeldung und einen dazugehörigen Benutzer zu erstellen, mit der Datenbanken oder andere Anmeldungen erstellt werden können:

1. Stellen Sie eine Verbindung mit der Datenbank **master** her, indem Sie die Anmeldeinformationen der Serverebenenprinzipal-Anmeldung (per Bereitstellungsprozess erstellt) oder die Anmeldeinformationen eines vorhandenen Mitglieds der Datenbankrolle **loginmanager** verwenden.
2. Erstellen Sie eine Anmeldung mit dem Befehl ``CREATE LOGIN``. Weitere Informationen finden Sie unter [CREATE LOGIN (Transact-SQL)](https://msdn.microsoft.com/library/ms189751.aspx).
3. Erstellen Sie in der Masterdatenbank einen neuen Benutzer für diese Anmeldung, indem Sie den Befehl ``CREATE USER`` verwenden. Weitere Informationen finden Sie unter [CREATE USER (Transact-SQL)](https://msdn.microsoft.com/library/ms173463.aspx).
4. Verwenden Sie die gespeicherte Prozedur ``sp_addrolememeber``, um der Datenbankrolle **dbmanager**, der Datenbankrolle „loginmanager“ oder beiden Rollen einen neuen Benutzer hinzuzufügen.

Im folgenden Codebeispiel wird veranschaulicht, wie Sie eine Anmeldung mit dem Namen **login1** erstellen. Außerdem wird ein entsprechender Datenbankbenutzer mit dem Namen **login1User** erstellt, der Datenbanken oder andere Anmeldungen erstellen kann, während eine Verbindung mit der Datenbank **master** besteht:

```
-- first, connect to the master database
CREATE LOGIN login1 WITH password='<ProvidePassword>';
CREATE USER login1User FROM LOGIN login1;
EXEC sp_addrolemember 'dbmanager', 'login1User';
EXEC sp_addrolemember 'loginmanager', 'login1User';
```

> [AZURE.NOTE]Sie müssen beim Erstellen einer Anmeldung ein sicheres Kennwort verwenden. Weitere Informationen finden Sie unter [Sichere Kennwörter](https://msdn.microsoft.com/library/ms161962.aspx).

## Gewähren von Datenbankzugriff für eine Anmeldung

Alle Anmeldungen müssen in der Masterdatenbank erstellt werden. Nachdem ein Benutzername erstellt wurde, können Sie für diese Anmeldung ein Benutzerkonto in einer anderen Datenbank erstellen. Microsoft Azure SQL-Datenbank unterstützt Datenbankrollen genauso, wie dies bei einer lokalen Instanz von SQL Server der Fall ist.

Führen Sie die folgenden Schritte aus, um ein Benutzerkonto in einer anderen Datenbank zu erstellen (vorausgesetzt, Sie haben keine Anmeldung oder Datenbank erstellt):

1. Stellen Sie eine Verbindung mit der Datenbank **master** her (mit einer Anmeldung mit den Rollen **loginmanager** und **dbmanager**).
2. Erstellen Sie mit dem Befehl ``CREATE LOGIN`` eine neue Anmeldung. Weitere Informationen finden Sie unter [CREATE LOGIN (Transact-SQL)](https://msdn.microsoft.com/library/ms189751.aspx). Windows-Authentifizierung wird nicht unterstützt.
3. Erstellen Sie mit dem Befehl ``CREATE DATABASE`` eine neue Datenbank. Weitere Informationen finden Sie unter [CREATE DATABASE (SQL Server Transact-SQL)](https://msdn.microsoft.com/library/ms176061.aspx).
4. Stellen Sie eine Verbindung mit der neuen Datenbank her (mit der Anmeldung, mit der die Datenbank erstellt wurde).
5. Erstellen Sie einen neuen Benutzer in der neuen Datenbank, indem Sie den Befehl ``CREATE USER`` verwenden. Weitere Informationen finden Sie unter [CREATE USER (Transact-SQL)](https://msdn.microsoft.com/library/ms173463.aspx).

Im folgenden Codebeispiel wird veranschaulicht, wie Sie eine Anmeldung mit dem Namen **login1** und eine Datenbank mit dem Namen **database1** erstellen:

```
-- first, connect to the master database
CREATE LOGIN login1 WITH password='<ProvidePassword>';
CREATE DATABASE database1;
```

> [AZURE.NOTE]Sie müssen beim Erstellen einer Anmeldung ein sicheres Kennwort verwenden. Weitere Informationen finden Sie unter [Sichere Kennwörter](https://msdn.microsoft.com/library/ms161962.aspx).

Im nächsten Beispiel wird veranschaulicht, wie Sie einen Datenbankbenutzer mit dem Namen **login1User** in der Datenbank **database1** erstellen, der zur Anmeldung **login1** gehört:

```
-- Establish a new connection to the database1 database
CREATE USER login1User FROM LOGIN login1;
```

Dieses Berechtigungsmodell auf Datenbankebene in Microsoft Azure SQL-Datenbank ist mit einer lokalen Instanz von SQL Server identisch. Informationen hierzu finden Sie in den folgenden Themen der Referenz in der SQL Server-Onlinedokumentation.

- [Verwalten von Anmeldungen, Benutzern und Schemas: Vorgehensweisen](https://msdn.microsoft.com/library/aa337552.aspx) 
- [Lektion 2: Konfigurieren von Berechtigungen für Datenbankobjekte](https://msdn.microsoft.com/library/ms365345.aspx) 

> [AZURE.NOTE]Sicherheitsbezogene Transact-SQL-Anweisungen in Microsoft Azure SQL-Datenbank können sich in Bezug auf die verfügbaren Parameter leicht unterscheiden. Weitere Informationen finden Sie unter [Transact-SQL-Referenz für die Azure SQL-Datenbank](sql-database-transact-sql-information.md).

## Anzeigen von Anmeldungen und Datenbanken

Verwenden Sie zum Anzeigen von Anmeldungen und Datenbanken auf Ihrem Azure SQL-Datenbank-Server die Ansicht ``sys.sql_logins`` bzw. ``sys.databases`` der Masterdatenbank. Im folgenden Beispiel wird veranschaulicht, wie Sie eine Liste aller Anmeldungen und Datenbanken auf Ihrem Azure SQL-Datenbank-Server anzeigen.

```
-- first, connect to the master database
SELECT * FROM sys.sql_logins;
SELECT * FROM sys.databases; 
```

## Weitere Informationen

[Sicherheitsrichtlinien und Einschränkungen von Azure SQL-Datenbank](sql-database-security-guidelines.md)

<!---HONumber=August15_HO9-->