<properties 
	pageTitle="Andere Ports als 1433 für ADO.NET 4.5, ODBC 11 und SQL-Datenbank V12 | Microsoft Azure"
	description="Bei Clientverbindungen mit Azure SQL-Datenbank V12 wird der Proxy manchmal umgangen und direkt mit der Datenbank interagiert. Andere Ports als 1433 werden wichtig."
	services="sql-database"
	documentationCenter=""
	authors="MightyPen"
	manager="jeffreyg"
	editor="" />



<tags 
	ms.service="sql-database" 
	ms.workload="data-management" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="08/05/2015" 
	ms.author="genemi"/>



# Andere Ports als 1433 für ADO.NET 4.5, ODBC 11 und SQL-Datenbank V12


Dieses Thema beschreibt die Änderungen, die Azure SQL-Datenbank V12 im Hinblick auf das Verbindungsverhalten von Clients mit ADO.NET 4.5 oder einer höheren Version aufweist.


## Erläuterungen zu Versionen


#### ADO.NET


- ADO.NET 4.0 unterstützt das Protokoll TDS 7.3, aber nicht 7.4.
- ADO.NET 4.5 und höher unterstützt das Protokoll TDS 7.4.
- ADO.NET 4.5 verwendet intern ODBC 11.
 - Die hier angegebenen Informationen für ADO.NET 4.5 gelten auch für ODBC 11.


#### SQL-Datenbank V11 und V12


In diesem Thema werden die Unterschiede bei der Clientverbindung zwischen SQL-Datenbank V11 und V12 hervorgehoben.


*Hinweis:* Die Transact-SQL-Anweisung `SELECT @@version;` gibt einen Wert zurück, der mit einer Zahl wie "11." oder "12." beginnt, und diese Zahlen stimmen mit den Versionsnamen V11 und V12 für die SQL-Datenbank überein.


## SQL-Datenbank V11: Port 1433


Wenn Ihr Clientprogramm ADO.NET 4.5 zum Herstellen einer Verbindung und Abfragen mit SQL-Datenbank V11 verwendet, sieht der interne Ablauf wie folgt aus:


1. ADO.NET versucht, eine Verbindung mit der SQL-Datenbank herzustellen.

2. ADO.NET verwendet Port 1433 zum Aufrufen eines Middlewaremoduls, und die Middleware stellt eine Verbindung mit der SQL-Datenbank her.

3. Die SQL-Datenbank sendet die Antwort zurück an die Middleware, welche die Antwort an ADO.NET an Port 1433 weiterleitet.


**Terminologie:** Bei dem zuvor beschriebenen Ablauf wird davon gesprochen, dass ADO.NET mithilfe der *Proxyroute* mit der SQL-Datenbank interagiert. Wäre keine Middleware einbezogen, würde von einer Verwendung der *direkten Route* gesprochen.


## SQL-Datenbank V12: "Außerhalb" im Vergleich zu "Innerhalb"


Bei Verbindungen mit V12 muss gefragt werden, ob das Clientprogramm *außerhalb* oder *innerhalb* der Azure-Cloudgrenzen ausgeführt wird. In den Unterabschnitten werden zwei häufige Szenarien erläutert.


#### *Außerhalb:* Client wird auf dem Desktopcomputer ausgeführt


Port 1433 ist der einzige Port, der auf dem Desktopcomputer geöffnet sein muss, auf dem die Clientanwendung der SQL-Datenbank gehostet ist.


#### *Innerhalb:* Client wird auf einer Azure-VM ausgeführt


Wenn der Client innerhalb der Azure-Cloudgrenzen ausgeführt wird, verwendet er die sogenannte *direkte Route* für die Interaktion mit dem SQL-Datenbankserver. Nachdem eine Verbindung hergestellt wurde, ist bei weiteren Interaktionen zwischen dem Client und der Datenbank kein Middleware-Proxy eingebunden.


Der Ablauf ist wie folgt:


1. ADO.NET 4.5 (oder höher) initiiert eine kurze Interaktion mit der Azure-Cloud und empfängt eine dynamisch ermittelte Portnummer.
 - Die dynamisch ermittelte Portnummer liegt im Bereich von 11000 bis 11999.

2. ADO.NET stellt dann eine direkte Verbindung mit dem SQL-Datenbankserver ohne Einbindung von Middleware her.

3. Abfragen werden direkt an die Datenbank gesendet, und Ergebnisse werden direkt an den Client zurückgegeben.


Stellen Sie sicher, dass der Portbereich von 11000 bis 11999 auf Ihrem Azure-Clientcomputer für ADO.NET 4.5-Clientinteraktionen mit SQL-Datenbank V12 verfügbar ist.

- Insbesondere dürfen Ports in diesem Bereich keine anderen ausgehenden Blockierungen aufweisen.
- Die Windows-Firewall auf Ihrer Azure-VM steuert die Porteinstellungen.


## Implizite Wiederholungslogik, die in der Proxyroute enthalten ist


In einer Produktionsumgebung wird empfohlen, dass Clients, die eine Verbindung mit Azure SQL-Datenbank V11 oder V12 herstellen, eine Wiederholungslogik in ihrem Code implementieren. Dabei kann es sich um benutzerdefinierten Code handeln oder um Code, der eine API wie Enterprise Library nutzt.


Die Proxyroute, die zuvor in diesem Thema behandelt wurde, ist für die Frage der Wiederholungslogik relevant:


- In V11 stellte das Middlewaremodul, das als Proxy fungierte, auch einen gewissen Grad an Wiederholungslogik bereit, mit der vorübergehende Fehler behandelt wurden.

- In V12 bietet der Proxy keine Wiederholungslogik.


In beiden Szenarien wird empfohlen, dass Clients eine Wiederholungslogik im eigenen Code implementieren. Die Notwendigkeit einer Wiederholungslogik im Client ist bei der neuesten Proxyroute höher, da diese keine Wiederholungslogik bereitstellt.


Codebeispiele für die Wiederholungslogik finden Sie unter [Clientcodebeispiele für die ersten Schritte mit SQL-Datenbank](sql-database-develop-quick-start-client-code-samples.md).


## Verwandte Links


- [Neuerungen in SQL-Datenbank V12](sql-database-v12-whats-new.md)

- Überlegungen zur Wiederholungslogik: [Abschnitt "In V12 stellt der Gateway keine Wiederholungslogik mehr bereit" des Themas "Verbindungen mit SQL-Datenbanken: Wichtige Empfehlungen"](sql-database-connect-central-recommendations.md#gatewaynoretry)

- ADO.NET 4.6 wurde am 20. Juli 2015 veröffentlicht. Eine Ankündigung im Blog des .NET-Teams finden Sie [hier](http://blogs.msdn.com/b/dotnet/archive/2015/07/20/announcing-net-framework-4-6.aspx).

- ADO.NET 4.5 wurde am 15. August 2012 veröffentlicht. Eine Ankündigung im Blog des .NET-Teams finden Sie [hier](http://blogs.msdn.com/b/dotnet/archive/2012/08/15/announcing-the-release-of-net-framework-4-5-rtm-product-and-source-code.aspx).
 - Ein Blogbeitrag zu ADO.NET 4.5.1 steht [hier](http://blogs.msdn.com/b/dotnet/archive/2013/06/26/announcing-the-net-framework-4-5-1-preview.aspx) bereit.

- [Liste der TDS-Protokollversionen](http://www.freetds.org/userguide/tdshistory.htm)

<!---HONumber=August15_HO6-->