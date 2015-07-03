<properties 
   pageTitle="Wiederherstellen einer Azure SQL-Datenbank mithilfe der Zeitpunktwiederherstellung im Azure-Portal" 
   description="Zeitpunktwiederherstellung, Microsoft Azure SQL-Datenbank, Datenbank wiederherstellen, Azure-Verwaltungsportal, Azure-Portal" 
   services="sql-database" 
   documentationCenter="" 
   authors="elfisher" 
   manager="jeffreyg" 
   editor="v-romcal"/>

<tags
   ms.service="sql-database"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="storage-backup-recovery" 
   ms.date="02/24/2015"
   ms.author="elfish; v-romcal"/>

# Wiederherstellen einer Azure SQL-Datenbank mithilfe der Zeitpunktwiederherstellung im Azure-Portal

> [AZURE.SELECTOR]
- [Zeitpunktwiederherstellung - PowerShell](http://azure.microsoft.com/documentation/articles/sql-database-point-in-time-restore-tutorial-powershell/)
- [Zeitpunktwiederherstellung - REST-API](http://azure.microsoft.com/documentation/articles/sql-database-point-in-time-restore-tutorial-rest/)

## Übersicht

In diesem Lernprogramm wird gezeigt, wie Sie eine Azure SQL-Datenbank mithilfe der Zeitpunktwiederherstellung im [Azure-Portal](http://manage.windowsazure.com/) wiederherstellen. Azure SQL-Datenbank verfügt über integrierte Sicherungen zur Unterstützung einer eigenständigen Zeitpunktwiederherstellung für die Dienstebenen Basic, Standard und Premium.

Bei einer Zeitpunktwiederherstellung wird eine neue Datenbank erstellt. Der Dienst wählt die Dienstebene basierend auf der beim Wiederherstellungspunkt verwendeten Sicherung automatisch aus. Stellen Sie sicher, dass Ihnen auf dem logischen Server ein Kontingent zum Erstellen einer weiteren Datenbank zur Verfügung steht. Wenn Sie ein erhöhtes Kontingent anfordern möchten, wenden Sie sich an den [Azure-Support](http://azure.microsoft.com/support/options/).

## Einschränkungen und Sicherheit

* Die Zeitpunktwiederherstellung ist für alle Dienstebenen (Basic, Standard und Premium) aktiviert.

* Die Beibehaltungsdauer für Sicherungen beträgt bei Basic 7 Tage, bei Standard 14 Tage und bei Premium 35 Tage.
 
* Nur der Administrator oder Co-Administrator für das Abonnement kann die Wiederherstellungsanforderung senden.

* Der Hauptanmeldungsbenutzer auf Serverebene wird Besitzer der wiederhergestellten Datenbank.

* Die Dienstebenen Web Edition und Business Edition unterstützen keine Zeitpunktwiederherstellung.
 
	* Wenn Sie eine Web Edition- oder Business Edition-Datenbank haben, können Sie den Datenbankkopierbefehl zum Abrufen einer transaktional konsistenten Kopie Ihrer Datenbank verwenden und dann die kopierte Datenbank in ein Microsoft Azure-Speicherkonto exportieren. Weitere Informationen finden Sie unter [Vorgehensweise: Verwenden des Datenbankkopierbefehls (Azure SQL-Datenbank)](http://msdn.microsoft.com/library/azure/ff951631.aspx) und [Vorgehensweise: Verwenden des Import/Export-Diensts in Azure SQL-Datenbank](http://msdn.microsoft.com/library/azure/hh335292.aspx).

	* Web Edition und Business Edition werden im September 2015 eingestellt. Weitere Informationen finden Sie unter [Häufig gestellte Fragen zur Einstellung von Web Edition und Business Edition](http://msdn.microsoft.com/library/azure/dn741330.aspx).

## Vorgehensweise: Wiederherstellen einer Azure SQL-Datenbank mithilfe der Zeitpunktwiederherstellung im Azure-Portal

<iframe src="http://channel9.msdn.com/Blogs/Windows-Azure/Restore-a-SQL-Database-Using-Point-in-Time-Restore/player" width="960" height="540" allowFullScreen frameBorder="0"></iframe>

1. Melden Sie sich am Azure-Portal mit Ihrem Microsoft-Konto an.

2. Klicken Sie im linken Navigationsbereich auf **SQL-DATENBANKEN**.
  
3. Klicken Sie in der Liste **DATENBANKEN** auf die Datenbank, die Sie wiederherstellen möchten. 

4. Klicken Sie unten auf der Seite auf der Befehlsleiste auf **WIEDERHERSTELLEN**. Dadurch wird das Dialogfeld **Wiederherstellungseinstellungen angeben** geöffnet.

5. Wählen Sie einen **DATENBANKNAMEN**. Standardmäßig wird ein Datenbankname für Sie ausgewählt, den Sie nach Wunsch ändern können.

6. Wählen Sie den Zeitpunkt, gemäß dem die Datenbank wiederhergestellt werden soll. Der Zeitpunkt muss innerhalb der Beibehaltungsdauer für die Datenbank liegen.
	
7. Klicken Sie auf das Häkchen, um die Wiederherstellungsanforderung zu senden.

Eine Wiederherstellung kann einige Zeit in Anspruch nehmen. Zum Überwachen des Status der Wiederherstellung klicken Sie rechts unten auf der Seite auf der Befehlsleiste auf das Symbol "Status" und dann auf **DETAILS**.

## Nächste Schritte

Weitere Informationen finden Sie unter den folgenden Links:

[Azure SQL-Datenbank-Geschäftskontinuität](http://msdn.microsoft.com/library/azure/hh852669.aspx)

[Sichern und Wiederherstellen der Azure SQL-Datenbank](http://msdn.microsoft.com/library/azure/jj650016.aspx)

[Azure SQL-Datenbank - Zeitpunktwiederherstellung (Blog)](http://azure.microsoft.com/blog/2014/10/01/azure-sql-database-point-in-time-restore/)
<!--HONumber=47-->
 