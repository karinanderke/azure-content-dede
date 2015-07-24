<properties
	pageTitle="Konfigurieren der Firewalleinstellungen (Azure SQL-Datenbank)"
	description="Konfigurieren der Firewall für Azure SQL-Datenbanken"
	services="sql-database"
	documentationCenter=""
	authors="BYHAM"
	manager="jeffreyg"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="get-started-article" 
	ms.date="06/22/2015"
	ms.author="rickbyh"/>


# Konfigurieren der Firewalleinstellungen (Azure SQL-Datenbank)

 Microsoft Azure SQL-Datenbank verwendet Firewallregeln, um Verbindungen mit Ihren Servern und Datenbanken zu erlauben. Sie können auf Ihrem Azure SQL-Datenbank-Server auf Serverebene und Datenbankebene Firewalleinstellungen für die Masterdatenbank oder eine Benutzerdatenbank definieren, um den selektiven Zugriff auf die Datenbank zu ermöglichen.

**Wichtig:** Um Anwendungen von Azure die Verbindung mit dem Datenbankserver zu ermöglichen, müssen Azure-Verbindungen aktiviert sein. Weitere Informationen zu Firewallregeln und dem Aktivieren von Verbindungen aus Azure finden Sie unter [Firewall für die Azure SQL-Datenbank](https://msdn.microsoft.com/library/azure/ee621782.aspx).


## Firewallregeln auf Serverebene

Firewallregeln auf Serverebene können über das Microsoft Azure-Verwaltungsportal, Transact-SQL, Azure PowerShell oder die REST-API erstellt und verwaltet werden.

### Verwalten Sie Firewallregeln auf Serverebene über das neue Azure-Portal
1. Besuchen Sie das Azure-Portal unter https://portal.azure.com, und melden Sie sich mit Ihrem Azure-Konto als Administrator oder Mitwirkender an.
2. Klicken Sie im linken Banner auf "ALLE DURCHSUCHEN", scrollen Sie nach unten, und klicken Sie dann auf SQL-Server.
3. Klicken Sie in der angezeigten Liste der SQL-Server auf den Server, für den Sie Firewallregeln konfigurieren möchten.

	![Firewall][1]

4. Klicken Sie oben im Server-Blatt auf Einstellungen, und klicken Sie dann auf "Firewall", um das Blatt für die Firewalleinstellungen des Servers zu öffnen.
5. Fügen Sie eine Firewallregel hinzu, oder ändern Sie eine.

	* Um die IP-Adresse des aktuellen Computers hinzuzufügen, klicken Sie am oberen Rand des Blatts auf **Add Client IP**.
	* Um zusätzliche IP-Adressen hinzuzufügen, geben Sie **REGELNAME**, **START-IP**-Adresse und **END-IP**-Adresse ein.
	* Um eine vorhandene Regel zu ändern, klicken Sie auf eines der Felder in der Regel, und ändern Sie den betreffenden Wert.
	* Um eine vorhandene Regel zu löschen, klicken Sie auf die Regel, dann auf die Auslassungspunkte (...) am Ende der Zeile, und klicken Sie dann auf **Löschen**.
6. Klicken Sie am oberen Rand des Blatts mit den Firewall-Einstellungen auf "Speichern", um die Änderungen zu speichern. ![Firewall-Blatt][2] 

## Verwalten von Firewallregeln auf Serverebene über das Verwaltungsportal 

1. Klicken Sie im Verwaltungsportal auf **SQL-Datenbanken**. Alle Datenbanken und ihre entsprechenden Server werden hier aufgelistet.
1. Klicken Sie oben auf der Seite auf **Servers**.
2. Klicken Sie auf den Pfeil neben dem Server, für den Sie die Firewallregeln verwalten möchten.
3. Klicken Sie oben auf der Seite auf **Konfigurieren**.

	*  Um den aktuellen Computer hinzuzufügen, klicken Sie auf "Zu den zulässigen IP-Adressen hinzufügen".
	*  Um zusätzliche IP-Adressen hinzuzufügen, geben Sie den Regelnamen, die Start-IP-Adresse und die End-IP-Adresse ein.
	*  Um eine vorhandene Regel zu ändern, klicken Sie auf eines der Felder in der Regel, und ändern Sie den betreffenden Wert.
	*  Um eine vorhandene Regel zu löschen, zeigen Sie mit Maus auf die Regel, bis ein Kreuz am Ende der Zeile angezeigt wird. Klicken Sie auf das Kreuz, um die Regel zu entfernen.
8. Klicken Sie unten auf der Seite auf **Speichern**, um die Änderungen zu speichern.

## Verwalten von Firewallregeln auf Serverebene über Transact-SQL
1. Starten Sie ein Abfragefenster über das Verwaltungsportal oder über SQL Server Management Studio.
2. Stellen Sie sicher, dass Sie mit der master-Datenbank verbunden sind.
3. Firewallregeln auf Serverebene können innerhalb des Abfragefensters erstellt, aktualisiert oder gelöscht werden.
4. Führen Sie zum Erstellen oder Aktualisieren von Firewallregeln auf Serverebene die gespeicherte Prozedur "sp_set_firewall_rule" aus. Im folgenden Beispiel wird ein Bereich von IP-Adressen auf dem Server Contoso aktiviert.

		EXEC sp_set_firewall_rule @name = N'ContosoFirewallRule', @start_ip_address = '192.168.1.1', @end_ip_address = '192.168.1.10'

	Um eine Firewallregel auf Serverebene zu löschen, führen Sie die gespeicherte Prozedur "sp_delete_firewall_rule" aus. Im folgenden Beispiel wird die Regel mit dem Namen "ContosoFirewallRule" gelöscht.
 
		EXEC sp_delete_firewall_rule @name = N'ContosoFirewallRule'
 
## Verwalten von Firewallregeln auf Serverebene über Azure PowerShell
1. Starten Sie Azure PowerShell.
2. Firewallregeln auf Serverebene können mit Azure PowerShell erstellt, aktualisiert und gelöscht werden. 

	Um eine neue Firewallregel auf Serverebene zu erstellen, führen Sie das Cmdlet "New-AzureSqlDatabaseServerFirewallRule" aus. Im folgenden Beispiel wird ein Bereich von IP-Adressen auf dem Server Contoso aktiviert.
 
		New-AzureSqlDatabaseServerFirewallRule –StartIPAddress 192.168.1.1 –EndIPAddress 192.168.1.10 –RuleName ContosoFirewallRule –ServerName Contoso
 
	Um eine neue Firewallregel auf Serverebene zu ändern, führen Sie das Cmdlet "Set-AzureSqlDatabaseServerFirewallRule" aus. Im folgenden Beispiel wird der Bereich zulässiger IP-Adressen für die Regel mit dem Namen "ContosoFirewallRule" geändert.
 
		Set-AzureSqlDatabaseServerFirewallRule –StartIPAddress 192.168.1.4 –EndIPAddress 192.168.1.10 –RuleName ContosoFirewallRule –ServerName Contoso

	Um eine vorhandene Firewallregel auf Serverebene zu löschen, führen Sie das Cmdlet "Remove-AzureSqlDatabaseServerFirewallRule" aus. Im folgenden Beispiel wird die Regel mit dem Namen "ContosoFirewallRule" gelöscht.

		Remove-AzureSqlDatabaseServerFirewallRule –RuleName ContosoFirewallRule –ServerName Contoso
 
## Verwalten von Firewallregeln auf Serverebene über die REST-API
1. Beim Verwalten von Firewallregeln über die REST-API muss eine Authentifizierung erfolgen. Informationen hierzu finden Sie unter "Authentifizieren von Dienstverwaltungsanforderungen".
2. Regeln auf Serverebene können über die REST-API erstellt, aktualisiert oder gelöscht werden.

	Um eine Firewallregel auf Serverebene zu erstellen oder zu aktualisieren, führen Sie die POST-Methode mit folgendem Inhalt aus:
 
		https://management.core.windows.net:8443/{subscriptionId}/services/sqlservers/servers/Contoso/firewallrules
	
	Anforderungstext

		<ServiceResource xmlns="http://schemas.microsoft.com/windowsazure">
		  <Name>ContosoFirewallRule</Name>
		  <StartIPAddress>192.168.1.4</StartIPAddress>
		  <EndIPAddress>192.168.1.10</EndIPAddress>
		</ServiceResource>
 

	Um eine vorhandene Firewallregel auf Serverebene zu entfernen, führen Sie die DELETE-Methode mit folgendem Inhalt aus:
	 
		https://management.core.windows.net:8443/{subscriptionId}/services/sqlservers/servers/Contoso/firewallrules/ContosoFirewallRule
 
## Firewallregeln auf Datenbankebene

1. Starten Sie nach dem Erstellen einer Firewall auf Serverebene für Ihre IP-Adresse ein Abfragefenster über das Verwaltungsportal oder über SQL Server Management Studio.
2. Stellen Sie eine Verbindung mit der Datenbank her, für die Sie eine Firewallregel auf Datenbankebene erstellen möchten.

	Um eine neue Firewallregel auf Datenbankebene zu erstellen oder eine vorhandene zu ändern, führen Sie die gespeicherte Prozedur "sp_set_database_firewall_rule" aus. Im folgenden Beispiel wird eine neue Firewallregel mit dem Namen "ContosoFirewallRule" erstellt.
 
		EXEC sp_set_database_firewall_rule @name = N'ContosoFirewallRule', @start_ip_address = '192.168.1.11', @end_ip_address = '192.168.1.11'
 
	Um eine vorhandene Firewallregel auf Datenbankebene zu löschen, führen Sie die gespeicherte Prozedur "sp_delete_database_firewall_rule" aus. Im folgenden Beispiel wird die Regel mit dem Namen "ContosoFirewallRule" gelöscht.
 
		EXEC sp_delete_database_firewall_rule @name = N'ContosoFirewallRule'


## Verwalten von Firewallregeln mithilfe der Dienstverwaltungs-REST-API

* [Create Firewall Rule](https://msdn.microsoft.com/library/azure/dn505712.aspx)
* [Delete Firewall Rule](https://msdn.microsoft.com/library/azure/dn505706.aspx)
* [Get Firewall Rule](https://msdn.microsoft.com/library/azure/dn505698.aspx)
* [List Firewall Rules](https://msdn.microsoft.com/library/azure/dn505715.aspx)

## Verwalten von Firewallregeln mithilfe von PowerShell

* [New-AzureSqlDatabaseServerFirewallRule](https://msdn.microsoft.com/library/azure/dn546724.aspx)
* [Remove-AzureSqlDatabaseServerFirewallRule](https://msdn.microsoft.com/library/azure/dn546727.aspx)
* [Set-AzureSqlDatabaseServerFirewallRule](https://msdn.microsoft.com/library/azure/dn546739.aspx)
* [Get-AzureSqlDatabaseServerFirewallRule](https://msdn.microsoft.com/library/azure/dn546731.aspx)
 
## Nächste Schritte

Ein Lernprogramm zum Erstellen einer Datenbank finden Sie unter [Erstellen einer ersten Azure SQL-Datenbank](sql-database-get-started.md). Hilfe beim Herstellen einer Verbindung mit einer Azure SQL-Datenbank von Open-Source-Anwendungen oder Anwendungen von Drittanbietern finden Sie unter [Richtlinien zum programmgesteuerten Herstellen einer Verbindung mit Azure SQL-Datenbank](https://msdn.microsoft.com/library/azure/ee336282.aspx). Informationen zum Navigieren zu Datenbanken finden Sie unter [Verwalten von Datenbanken, Anmeldungen und Benutzern in der Azure SQL-Datenbank](https://msdn.microsoft.com/library/azure/ee336235.aspx).

<!--Image references-->
[1]: ./media/sql-database-configure-firewall-settings/AzurePortalBrowseForFirewall.png
[2]: ./media/sql-database-configure-firewall-settings/AzurePortalFirewallSettings.png
<!--anchors-->

<!---HONumber=July15_HO2-->