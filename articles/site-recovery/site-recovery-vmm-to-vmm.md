<properties 
	pageTitle="Einrichten von Schutz zwischen lokalen VMM-Standorten" 
	description="Azure Site Recovery koordiniert Replikation, Failover und Wiederherstellung virtueller Hyper-V-Computer zwischen lokalen VMM-Standorten.." 
	services="site-recovery" 
	documentationCenter="" 
	authors="raynew" 
	manager="jwhit" 
	editor="tysonn"/>

<tags 
	ms.service="site-recovery" 
	ms.workload="backup-recovery" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="03/19/2015" 
	ms.author="raynew"/>

# Einrichten von Schutz zwischen lokalen VMM-Standorten


## Übersicht


Azure Site Recovery unterstützt Ihre Strategie für Geschäftskontinuität und Notfallwiederherstellung, indem Replikation, Failover und Wiederherstellung virtueller Computer in einer Vielzahl von Bereitstellungsszenarien aufeinander abgestimmt werden. Eine vollständige Liste der Bereitstellungsszenarien finden Sie unter [Übersicht zu Azure Site Recovery](../hyper-v-recovery-manager-overview.md).

Dieser Szenarioleitfaden beschreibt die Bereitstellung von Site Recovery zum Organisieren und Automatisieren von Schutz für Arbeitsauslastungen auf virtuellen Computern auf Hyper-V-Hostservern, die sich in privaten VMM-Clouds befinden. In diesem Szenario werden virtuelle Computer von einem primären VMM-Standort mithilfe von Hyper-V-Replikaten auf einen sekundären VMM-Standort repliziert.

Das Handbuch enthält Informationen zu Voraussetzungen für das Szenario und zeigt, wie Sie einen Site Recovery-Tresor einrichten, den Azure Site Recovery-Anbieter auf dem Quell- und dem Ziel-VMM-Server installieren, den Server im Tresor registrieren, Schutzeinstellungen für VMM-Clouds konfigurieren, die auf alle geschützten virtuellen Computer angewendet werden, und den Schutz für diese virtuellen Computer aktivieren. Zum Schluss testen Sie das Failover, um sicherzustellen, dass alles wie erwartet funktioniert.

Wenn beim Einrichten dieses Szenarios Probleme auftreten, stellen Sie Ihre Fragen im [Azure Recovery Services-Forum](http://go.microsoft.com/fwlink/?LinkId=313628).


## Vorbereitung
Stellen Sie sicher, dass diese Voraussetzungen erfüllt werden:
### Voraussetzungen für Azure

- Sie benötigen ein [Microsoft Azure](http://azure.microsoft.com/)-Konto. Wenn Sie keins besitzen, beginnen Sie mit einer [kostenlosen Testversion](http://aka.ms/try-azure). Darüber hinaus können Sie sich über die [Preisgestaltung für Azure Site Recovery-Manager](http://go.microsoft.com/fwlink/?LinkId=378268) informieren.
- Um zu verstehen, wie Informationen und Daten verwendet werden, lesen Sie die [Datenschutzbestimmungen zu Microsoft Azure](http://go.microsoft.com/fwlink/?LinkId=324899) und die zusätzlichen <a href="#privacy">Informationen zum Datenschutz für Site Recovery</a> am Ende dieses Themas.

### VMM-Voraussetzungen
- Sie benötigen mindestens einen VMM-Server.
- Auf dem VMM-Server sollte mindestens System Center 2012 SP1 mit den neuesten kumulativen Updates ausgeführt werden. 
- Auf allen VMM-Servern mit virtuellen Computern, die Sie schützen möchten, muss der Azure Site Recovery-Anbieter ausgeführt werden. Dieser wird bei der Bereitstellung von Azure Site Recovery installiert.
- Wenn Sie den Schutz mit einem einzelnen VMM-Server einrichten möchten, müssen mindestens zwei Clouds auf dem Server konfiguriert sein.
- Wenn Sie den Schutz mit zwei VMM-Servern bereitstellen möchten, muss für jeden Server mindestens eine Cloud auf dem primären VMM-Server, der geschützt werden soll, und eine Cloud auf dem sekundären VMM-Server konfiguriert sein, der für Schutz und Wiederherstellung verwendet werden soll.
- Für alle VMM-Clouds muss das Hyper-V-Kapazitätsprofil eingestellt sein.
- Die Quellcloud, die Sie schützen möchten, muss Folgendes enthalten:
	- Eine oder mehrere VMM-Hostgruppen.
	- Einen oder mehrere Hyper-V-Hostserver in jeder Hostgruppe.
	- Einen oder mehrere virtuelle Computer auf dem Hostserver. 
- Erfahren Sie mehr über das Einrichten von VMM-Clouds:
	- Weitere Informationen über private VMM-Clouds erhalten Sie unter [What's New in Private Cloud with System Center 2012 R2 VMM](http://go.microsoft.com/fwlink/?LinkId=324952) und unter [VMM 2012 and the clouds](http://go.microsoft.com/fwlink/?LinkId=324956) (in englischer Sprache). 
	- Informieren Sie sich über das [Konfigurieren der VMM-Cloud-Struktur](https://msdn.microsoft.com/library/azure/dn469075.aspx#BKMK_Fabric).
	- Sobald Ihre Cloud-Strukturelemente eingerichtet sind, finden Sie Informationen zum Erstellen privater Clouds unter [Erstellen einer privaten Cloud in VMM](http://go.microsoft.com/fwlink/?LinkId=324953) und unter [Walkthrough: Creating private clouds with System Center 2012 SP1 VMM](http://go.microsoft.com/fwlink/?LinkId=324954) (in englischer Sprache).

### Hyper-V-Voraussetzungen

- Auf dem Host- und dem Ziel-Hyper-V-Server muss mindestens Windows Server 2012 mit Hyper-V-Rolle ausgeführt werden, und die neuesten Updates müssen installiert sein.
- Wenn Sie Hyper-V in einem Cluster ausführen, wird der Clusterbroker nicht automatisch erstellt, wenn Sie einen Cluster mit statischen IP-Adressen verwenden. Sie müssen den Clusterbroker manuell konfigurieren. Anweisungen finden Sie unter [Hyper-V-Replikatbroker konfigurieren](http://go.microsoft.com/fwlink/?LinkId=403937).
- Alle Hyper-V-Hostserver oder Cluster, deren Schutz Sie verwalten möchten, müssen in einer VMM-Cloud enthalten sein.
 
### Voraussetzungen für die Netzwerkzuordnung
Die Netzwerkzuordnung stellt sicher, dass die virtuelle Replikatcomputer nach einem Failover optimal auf Hyper-V-Hostservern platziert werden und sich mit den entsprechenden VM-Netzwerken verbinden können. Wenn Sie keine Netzwerkzuordnung konfigurieren, werden virtuelle Replikatcomputer nach einem Failover nicht mit VM-Netzwerken verbunden. Wenn Sie eine Netzwerkzuordnung bereitstellen möchten, benötigen Sie Folgendes:

- Die virtuellen Computer, die Sie auf dem VMM-Quellserver schützen möchten, sollten mit einem VM-Netzwerk verbunden sein. Dieses Netzwerk sollte mit einem logischen Netzwerk verbunden sein, das der Cloud zugeordnet ist.
- Für die Zielcloud auf dem sekundären VMM-Server, die Sie für die Wiederherstellung verwenden, sollte ein entsprechendes VM-Netzwerk konfiguriert sein. Dieses sollte wiederum mit einem entsprechenden logischen Netzwerk verknüpft sein, das der Zielcloud zugeordnet ist.

Informationen zur Netzwerkzuordnung finden Sie unter:

- [Konfigurieren von logischen Netzwerken in VMM](http://go.microsoft.com/fwlink/?LinkId=386307)
- [Konfiguration von VM-Netzwerken und Gateways in VMM](http://go.microsoft.com/fwlink/?LinkId=386308)

### Voraussetzungen für die Speicherzuordnung
Standardmäßig werden beim Replizieren eines virtuellen Computers auf einem Hyper-V-Host-Quellserver auf einen Hyper-V-Host-Zielserver die replizierten Daten am Standardspeicherort gespeichert, der für den Hyper-V-Zielhost in Hyper-V-Manager angegeben wurde. Um die Ablage von Replikationsdaten zu steuern, können Sie eine Speicherzuordnung konfigurieren. Zu diesem Zweck müssen Sie die Speicherklassifizierungen auf dem Quell- und dem Ziel-VMM-Server einrichten, bevor Sie mit der Bereitstellung beginnen.
Anweisungen hierzu finden Sie unter [Erstellen von Speicherklassifizierungen in VMM](http://go.microsoft.com/fwlink/?LinkId=400937).


## Schritt 1: Erstellen eines Site Recovery-Tresors

1. Melden Sie sich über den VMM-Server, den Sie registrieren möchten, beim [Verwaltungsportal](https://portal.azure.com) an.

2. Erweitern Sie **Data Services** > **Recovery Services**, und klicken Sie auf **Site Recovery-Tresor**.


3. Klicken Sie auf **Neu erstellen** > **Schnellerfassung**.
	
4. Geben Sie unter **Name** einen Anzeigenamen ein, mit dem der Tresor identifiziert wird.

5. Wählen Sie unter **Region** die geografische Region für den Tresor aus. Welche Regionen unterstützt werden, erfahren Sie unter "Geografische Verfügbarkeit" unter [Azure Site Recovery-Preisdetails](http://go.microsoft.com/fwlink/?LinkId=389880).</a>

6. Klicken Sie auf **Tresor erstellen**.

	![Create vault](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_CreateVault.png)

Überprüfen Sie in der Statusleiste, ob der Tresor erstellt wurde. Der Tresor wird auf der Hauptseite von Recovery Services als **aktiv** aufgelistet.

## Schritt 2: Generieren eines Tresorregistrierungsschlüssels

Generieren Sie einen Registrierungsschlüssel im Tresor. Nachdem Sie den Azure Site Recovery-Anbieter heruntergeladen und auf dem VMM-Server installiert haben, verwenden Sie diesen Schlüssel, um den VMM-Server im Tresor zu registrieren.

1. Klicken Sie auf der Seite <b>Recovery Services</b> auf den Tresor, um die Seite Schnellstart zu öffnen. Schnellstart kann auch jederzeit über das Symbol geöffnet werden.

	![Quick Start Icon](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_QuickStartIcon.png)

2. Wählen Sie in der Dropdownliste die Option **Zwischen zwei lokalen Hyper-V-Standorten** aus.
3. Klicken Sie unter **VMM-Server vorbereiten** auf **Registrierungsschlüsseldatei erstellen**. Die Schlüsseldatei wird automatisch generiert und ist nach ihrer Erstellung für 5 Tage gültig. Wenn Sie nicht vom VMM-Server aus auf das Azure-Portal zugreifen, müssen Sie diese Datei auf den Server kopieren. 

	![Registration key](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_E2ERegisterKey.png)
	
## Schritt 3: Installieren des Azure Site Recovery-Anbieters	

1. Klicken Sie auf der Seite <b>Schnellstart</b> unter **VMM-Server** vorbereiten auf <b>Microsoft Azure Site Recovery-Anbieter für die Installation auf VMM-Servern herunterladen</b>, um die neueste Version der Anbieterinstallationsdatei herunterzuladen.

2. Führen Sie diese Datei auf dem VMM-Quell- und -Zielserver aus. Wenn VMM in einem Cluster bereitgestellt wird und Sie den Anbieter erstmals installieren, führen Sie die Installation auf einem aktiven Knoten durch und schließen Sie sie ab, um den VMM-Server im Tresor zu registrieren. Installieren Sie den Anbieter anschließend auf den anderen Knoten. Beachten Sie, dass Sie ein Upgrade des Anbieters auf allen Knoten durchführen müssen, da auf allen Knoten dieselbe Anbieterversion ausgeführt werden sollte.

3. Geben Sie unter **Voraussetzungsprüfung** an, dass der VMM-Dienst für die Anbietereinrichtung angehalten werden soll. Der Dienst wird angehalten und nach Abschluss der Einrichtung automatisch neu gestartet. Bei der Installation auf einem VMM-Cluster werden Sie aufgefordert, die Clusterrolle zu beenden.

	![Prerequisites](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_ProviderPrereq.png)

4. Unter **Microsoft Update** können Sie Updates abonnieren. Mit dieser Einstellung werden Anbieterupdates gemäß Ihrer Microsoft Update-Richtlinie installiert.

	![Microsoft Updates](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_ProviderUpdate.png)

Fahren Sie nach der Installation des Anbieters mit der Einrichtung fort, um den Server im Tresor zu registrieren.

5. Geben Sie auf der Seite Internetverbindung an, wie der Anbieter auf dem VMM-Server die Verbindung mit dem Internet herstellen soll. Wählen Sie <b>Standard-Systemproxyeinstellungen verwenden</b> aus, um die auf dem Server konfigurierten Standardeinstellungen für die Internetverbindung zu verwenden.

	![Internet Settings](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_ProviderProxy.png)

	- Wenn Sie einen benutzerdefinierten Proxy verwenden möchten, sollten Sie diesen vor der Installation des Anbieters einrichten. Wenn Sie benutzerdefinierte Proxyeinstellungen konfigurieren, wird ein Test ausgeführt, um die Proxyverbindung zu überprüfen.
	- Wenn Sie einen benutzerdefinierten Proxy verwenden oder Ihr Standardproxy eine Authentifizierung verlangt, müssen Sie die Proxydetails einschließlich der Proxyadresse und des Ports eingeben.
	- Sie sollten die folgenden Adressen vom Routing über den Proxy ausschließen:
		- Die URL für die Verbindung mit Azure Site Recovery: *.hypervrecoverymanager.windowsazure.com
		- *.accesscontrol.windows.net
		- *.backup.windowsazure.com
		- *.blob.core.windows.net 
		- *.store.core.windows.net 
	- Wenn Sie ausgehende Verbindungen mit einem Azure-Domänencontroller erlauben müssen, lassen Sie die unter [IP-Bereiche des Azure-Rechenzentrums](http://go.microsoft.com/fwlink/?LinkId=511094) beschriebenen IP-Adressen sowie die Protokolle HTTP (80) und HTTPS (443) zu. 
	- Wenn Sie einen benutzerdefinierten Proxy verwenden, wird ein ausführendes VMM-Konto (DRAProxyAccount) automatisch mit den angegebenen Proxyanmeldeinformationen erstellt. Konfigurieren Sie den Proxyserver so, dass dieses Konto erfolgreich authentifiziert werden kann. In der VMM-Konsole können die Einstellungen des ausführenden VMM-Kontos geändert werden. Zu diesem Zweck öffnen Sie den Arbeitsbereich "Einstellungen", erweitern Sie "Sicherheit", klicken Sie auf "Ausführende Konten", und ändern Sie das Kennwort für DRAProxyAccount. Sie müssen den VMM-Dienst neu starten, damit diese Einstellung wirksam wird.
6. Wählen Sie unter **Registrierungsschlüssel** den Schlüssel aus, den Sie von Azure Site Recovery heruntergeladen und auf den VMM-Server kopiert haben.
7. Prüfen Sie unter **Tresorname** den Namen des Tresors, in dem der Server registriert wird.
8. Geben Sie unter **Servername** einen Anzeigenamen ein, um den VMM-Server im Tresor zu identifizieren. Geben Sie in einer Clusterkonfiguration den Rollennamen des VMM-Clusters an. 

	![Server registration](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_ProviderRegKeyServerName.png)


9. Wählen Sie unter **Anfängliche Cloud-Metadatensynchronisierung** aus, ob die Metadaten aller Clouds auf dem VMM-Server mit dem Tresor synchronisiert werden sollen. Diese Aktion muss für jeden VMM-Server nur einmal ausgeführt werden. Wenn Sie nicht alle Clouds synchronisieren möchten, können Sie diese Einstellung deaktiviert lassen und in den Cloudeigenschaften in der VMM-Konsole jede Cloud einzeln synchronisieren.


7. Die Option **Datenverschlüsselung** ist für den Schutz von lokalem zu lokalem Standort nicht relevant.

	![Server registration](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_ProviderSyncEncrypt.png)

8. Klicken Sie auf <b>Registrieren</b>, um den Prozess abzuschließen. Die Metadaten vom VMM-Server werden von Azure Site Recovery abgerufen. Der Server wird im Tresor auf der Seite **Server** auf der Registerkarte **Ressourcen** angezeigt.

Nach der Registrierung können Sie die Anbietereinstellungen in der VMM-Konsole oder über die Befehlszeile ändern.

## Schritt 4: Konfigurieren der Cloudschutzeinstellungen

Nachdem Sie die VMM-Server registriert haben, können Sie Schutzeinstellungen für die Clouds konfigurieren. Wenn Sie bei der Anbieterinstallation die Option **Clouddaten mit Tresor synchronisieren** aktiviert haben, werden alle Clouds auf dem VMM-Server auf der Registerkarte <b>Geschützte Elemente</b> im Tresor angezeigt. Andernfalls können Sie über die Registerkarte **Allgemein** auf der Eigenschaftenseite der Cloud in der VMM-Konsole eine bestimmte Cloud mit Azure Site Recovery synchronisieren.

![Published Cloud](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_CloudsList.png)

1. Klicken Sie auf der Seite Schnellstart auf **Schutz für VMM-Clouds einrichten**.
2. Wählen Sie auf der Registerkarte **VMM-Clouds** die Cloud aus, die Sie konfigurieren möchten, und wechseln Sie zur Registerkarte **Konfiguration**. 
3. Wählen Sie unter <b>Ziel</b> die Option <b>VMM</b> aus.
4. Wählen Sie unter <b>Zielspeicherort</b> den lokalen VMM-Server aus, der die Cloud verwaltet, die Sie für die Wiederherstellung verwenden möchten.
4. Wählen Sie unter <b>Zielcloud</b> die Zielcloud aus, die Sie für das Failover virtueller Computer in der Quellcloud verwenden möchten. Beachten Sie Folgendes:
	- Es wird empfohlen, eine Zielcloud auszuwählen, die die Voraussetzungen für die Wiederherstellung der virtuellen Computer erfüllt, die Sie schützen möchten.
	- Eine Cloud kann nur zu einem einzigen Cloudpaar gehören - entweder als primäre oder als Zielcloud.
6. Geben Sie unter <b>Kopierhäufigkeit</b> an, wie oft die Daten zwischen dem Quell- und dem Zielspeicherort synchronisiert werden sollen. Beachten Sie, dass diese Einstellung nur relevant ist, wenn auf dem Hyper-V-Host Windows Server 2012 R2 ausgeführt wird. Für andere Server wird die Standardeinstellung "5 Minuten" verwendet.
7. Geben Sie unter <b>Zusätzliche Wiederherstellungspunkte</b> an, ob zusätzliche Wiederherstellungspunkte erstellt werden sollen. Wenn die Standardeinstellung 0 (Null) aktiviert ist, wird nur der letzte Wiederherstellungspunkt für einen primären virtuellen Computer auf einem Replikathostserver gespeichert. Beachten Sie, dass für die Aktivierung mehrerer Wiederherstellungspunkte zusätzlicher Speicherplatz für die Momentaufnahmen erforderlich ist, die an jedem Wiederherstellungspunkt gespeichert werden. Standardmäßig werden Wiederherstellungspunkte jede Stunde erstellt, sodass jeder Wiederherstellungspunkt die Daten einer Stunde enthält. Der Wiederherstellungspunktwert, den Sie für den virtuellen Computer in der VMM-Konsole zuweisen, sollte nicht kleiner als der Wert sein, den Sie in der Azure Site Recovery-Konsole zuweisen.
8. Geben Sie unter <b>Häufigkeit von anwendungskonsistenten Momentaufnahmen</b> an, wie oft anwendungskonsistente Momentaufnahmen erstellt werden sollen. Hyper-V verwendet zwei Sicherungstypen: eine Standardsicherung, die eine inkrementelle Sicherung des gesamten virtuellen Computers bereitstellt, und eine anwendungskonsistente Momentaufnahme, die eine Zeitpunkt-Momentaufnahme der Anwendungsdaten innerhalb des virtuellen Computers erfasst. Anwendungskonsistente Momentaufnahmen verwenden den Volumeschattenkopie-Dienst (Volume Shadow Copy Service, VSS), um sicherzustellen, dass Anwendungen sich bei der Erstellung der Momentaufnahme in einem konsistenten Zustand befinden. Beachten Sie, dass die Leistung von Anwendungen auf virtuellen Quellcomputern durch die Aktivierung anwendungskonsistenter Momentaufnahmen beeinträchtigt wird. Stellen Sie sicher, dass der festgelegte Wert kleiner als die konfigurierte Anzahl der zusätzlichen Wiederherstellungspunkte ist.

	![Configure protection settings](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_CloudSettings.png)

9. Geben Sie unter <b>Datenübertragungskomprimierung</b> an, ob replizierte Daten, die übertragen werden, komprimiert werden sollen. 
10. Geben Sie unter <b>Authentifizierung</b> an, wie der Datenverkehr zwischen dem primären und dem Hyper-V-Hostserver für die Wiederherstellung authentifiziert wird. Wählen Sie HTTPS, sofern Sie keine funktionierende Kerberos-Umgebung konfiguriert haben. Azure Site Recovery konfiguriert automatisch Zertifikate für die HTTPS-Authentifizierung. Es ist keine manuelle Konfiguration notwendig. Wenn Sie Kerberos auswählen, wird ein Kerberos-Ticket für die gegenseitige Authentifizierung der Hostserver verwendet. Standardmäßig werden die Ports 8083 und 8084 (für Zertifikate) in der Windows-Firewall auf den Hyper-V-Hostservern geöffnet. Bitte beachten Sie, dass diese Einstellung nur für Hyper-V-Hostserver relevant ist, die unter Windows Server 2012 R2 laufen.
11. Ändern Sie unter <b>Port</b> die Portnummer, an der der Quell- und der Zielhostcomputer auf Replikationsdatenverkehr lauschen. Beispielsweise können Sie die Einstellung ändern, wenn Sie die QoS-Netzwerk-Bandbreiteneinschränkung (Quality of Service) für Replikationsdatenverkehr anwenden möchten. Stellen Sie sicher, dass der Port von keiner anderen Anwendung verwendet wird und in den Firewalleinstellungen geöffnet wurde.
12. Geben Sie unter <b>Replikationsmethode</b> an, wie die anfängliche Replikation der Daten vom Quell- zum Zielspeicherort erfolgt, bevor die reguläre Replikation beginnt. 
	- <b>Über Netzwerk</b> - Das Kopieren von Daten über das Netzwerk kann zeitaufwendig und ressourcenintensiv sein. Es wird empfohlen, diese Option zu verwenden, wenn die Cloud virtuelle Computer mit relativ kleinen virtuellen Festplatten enthält und der primäre VMM-Server über eine große Bandbreite mit dem sekundären VMM-Server verbunden ist. Sie können angeben, dass der Kopiervorgang sofort gestartet werden soll, oder eine Uhrzeit auswählen. Wenn Sie Netzwerkreplikation verwenden, empfehlen wir, deren Ausführung außerhalb der Spitzenzeiten einzuplanen.
	- <b>Offline</b> - Mit dieser Methode wird die erste Replikation unter Verwendung externer Medien durchgeführt. Dies ist sinnvoll, wenn Sie einen Abfall der Netzwerkleistung vermeiden möchten oder wenn Sie mit geografisch fernen Orten arbeiten. Wenn Sie diese Methode anwenden möchten, müssen Sie den Exportspeicherort für die Quellcloud und den Importspeicherort angeben. Wenn sie den Schutz für einen virtuellen Computer aktivieren, wird die virtuelle Festplatte an den angegebenen Exportspeicherort kopiert. Sie senden sie an den Zielstandort und kopieren sie an den Importspeicherort. Das System kopiert die importierten Informationen auf die virtuellen Replikatcomputer. Eine vollständige Liste der Voraussetzungen für die Offline-Replikation finden sie unter <a href="http://go.microsoft.com/fwlink/?LinkId=323469">Schritt 3: Konfigurieren der Schutzeinstellungen für VMM-Clouds</a> im Bereitstellungshandbuch.
13. Wählen Sie **Virtuellen Replikatcomputer löschen** aus, um anzugeben, dass der virtuelle Replikatcomputer gelöscht werden soll, wenn er nicht mehr geschützt wird, weil Sie die Option **Schutz für virtuellen Computer löschen** auf der Registerkarte Virtuelle Computer der Cloud-Eigenschaften ausgewählt haben. Wenn diese Einstellung aktiviert ist, und Sie deaktivieren den Schutz, wird der virtuelle Computer aus Azure Site Recovery entfernt, die Site Recovery-Einstellungen für den virtuellen Computer werden aus der VMM-Konsole entfernt, und das Replikat wird gelöscht.
	![Configure protection settings](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_CloudSettingsRep.png)

<p>Nachdem Sie die Einstellungen gespeichert haben, wird ein Auftrag erstellt, der auf der Registerkarte <b>Aufträge</b> überwacht werden kann. Alle Hyper-V-Hostserver in der VMM-Quellcloud werden für die Replikation konfiguriert. Cloud-Einstellungen können auf der Registerkarte <b>Konfigurieren</b> bearbeitet werden. Wenn Sie den Zielspeicherort oder die Zielcloud ändern möchten, müssen Sie die Cloudkonfiguration entfernen und die Cloud anschließend neu konfigurieren.</p>

### Vorbereiten der ersten Offlinereplikation

Sie müssen eine Reihe von Aktionen durchführen, um die erste Replikation offline vorzubereiten:

- Geben Sie auf dem Quellserver einen Pfad an, von dem der Datenexport stattfinden soll. Weisen Sie Vollzugriff für NTFS und Freigabeberechtigungen für den VMM-Dienst auf dem Exportpfad zu. Auf dem Zielserver geben Sie einen Pfad an, von dem der Datenimport stattfinden soll. Weisen Sie für diesen Importpfad die gleichen Berechtigungen zu. 
- Wenn der Import- oder Exportpfad freigegeben ist, weisen Sie Mitgliedschaften in den Gruppen "Administratoren", "Hauptbenutzer", "Druck-Operatoren" oder "Server-Operatoren" für das VMM-Dienstkonto auf dem Remotecomputer zu, auf dem sich die Freigabe befindet.
- Wenn Sie zum Hinzufügen von Hosts ausführende Konten verwenden, weisen Sie auf dem Import- und Exportpfad Lese- und Schreibberechtigungen für die ausführenden Konten in VMM zu.
- Die Import- und Exportfreigaben sollten sich nicht auf Computern befinden, die als Hyper-V-Hostserver eingesetzt werden, da die Loopbackkonfiguration von Hyper-V nicht unterstützt wird.
- Aktivieren und konfigurieren Sie die eingeschränkte Delegierung in Active Directory auf jedem Hyper-V-Hostserver mit virtuellen Computern, die Sie schützen möchten, um den Remotecomputern zu vertrauen, auf denen sich der Import- und der Exportpfad befinden. Gehen Sie dabei wie folgt vor:
	1. Öffnen Sie auf dem Domänencontroller **Active Directory-Benutzer und-Computer**.
	2. Klicken Sie in der Konsolenstruktur auf **Domänenname** > **Computer**.
	3. Klicken Sie mit der rechten Maustaste auf den Namen des Hyper-V-Hostservers > **Eigenschaften**.
	4. Klicken Sie auf der Registerkarte **Delegierung** auf **Computer bei Delegierungen angegebener Dienste vertrauen**.
	5. Klicken Sie auf **Beliebiges Authentifizierungsprotokoll verwenden**.
	6. Klicken Sie auf **Hinzufügen** > **Benutzer und Computer**.
	7. Geben Sie den Namen des Computers ein, der den Exportpfad hostet > **OK**. Halten Sie die STRG-Taste gedrückt, und klicken Sie in der Liste der verfügbaren Dienste auf **CIFS** > **OK**. Wiederholen Sie diesen Vorgang für den Namen des Computers, der den Importpfad hostet. Wiederholen Sie den Vorgang bei Bedarf für zusätzliche Hyper-V-Hostserver.
	
## Schritt 5: Konfigurieren der Netzwerkzuordnung
1. Klicken Sie auf der Seite "Schnellstart" auf **Netzwerke zuordnen**.
2. Wählen Sie den VMM-Quellserver aus, von dem aus Sie Netzwerke zuordnen möchten, und dann den Ziel-VMM-Server, dem die Netzwerke zugeordnet werden soll. Die Liste der Quellnetzwerke und ihre zugeordneten Zielnetzwerke werden angezeigt. Für Netzwerke, die derzeit nicht zugeordnet sind, wird ein leerer Wert angezeigt. Klicken Sie auf das Informationssymbol neben den Namen des Quell- und Zielnetzwerks, um die Subnetze für jedes Netzwerk anzuzeigen.
3. Wählen Sie ein Netzwerk unter **Netzwerk auf Quelle** > **Zuordnen**. Der Dienst erkennt die VM-Netzwerke auf dem Zielserver und zeigt sie an. 

	![Configure network mapping](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_NetworkMapping1.png)

4. Wählen Sie im Dialogfeld eines der VM-Netzwerke vom VMM-Zielserver aus.

	![Select a target network](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_NetworkMapping2.png)

5. Wenn Sie ein Zielnetzwerk auswählen, werden die geschützten Clouds angezeigt, die das Quellnetzwerk verwenden. Die verfügbaren Zielnetzwerke, die den zum Schutz verwendeten Clouds zugeordnet sind, werden ebenfalls angezeigt. Es wird empfohlen, ein Zielnetzwerk auszuwählen, das für alle für den Schutz verwendeten Clouds verfügbar ist.
6. Aktivieren Sie das Kontrollkästchen, um den Zuordnungsprozess abzuschließen. Ein Auftrag beginnt mit der Verfolgung des Zuordnungsprozesses. Dieser kann auf der Registerkarte **Aufträge** angezeigt werden.


## Schritt 6: Konfigurieren der Speicherzuordnung
Standardmäßig werden beim Replizieren eines virtuellen Computers auf einem Hyper-V-Host-Quellserver auf einen Hyper-V-Host-Zielserver die replizierten Daten am Standardspeicherort gespeichert, der für den Hyper-V-Zielhost in Hyper-V-Manager angegeben wurde. Um die Ablage von Replikationsdaten zu steuern, können Sie Speicherzuordnungen folgendermaßen konfigurieren:

- Definieren Sie Speicherklassifizierungen auf dem Quell- und Ziel-VMM-Server. Anweisungen hierzu finden Sie unter [Erstellen von Speicher-Klassifizierungen in VMM](http://go.microsoft.com/fwlink/?LinkId=400937). Klassifizierungen müssen den Hyper-V-Hostservern in Quell- und Ziel-Clouds zur Verfügung stehen. Klassifizierungen müssen nicht den gleichen Speichertyp aufweisen. Ordnen Sie z. B. eine Quellklassifizierung mit SMB-Freigaben einer Zielklassifizierung zu, die CSVs enthält.
- Nachdem die Klassifizierungen eingerichtet wurden, können Sie Zuordnungen erstellen.
1. Wählen Sie auf der Seite **Schnellstart** > **Speicher zuordnen**.
1. Klicken Sie auf die Registerkarte **Speicher** > **Speicherklassifizierungen zuordnen**.
1. Wählen Sie auf der Registerkarte **Speicherklassifizierungen zuordnen** die Klassifizierungen auf dem Quell- und dem Ziel-VMM-Server aus. Speichern Sie die Einstellungen.

	![Select a target network](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_StorageMapping1.png)


## Schritt 7: Aktivieren des Schutzes für virtuelle Computer
Nach der korrekten Konfiguration von Servern, Clouds und Netzwerken können Sie den Schutz für die virtuellen Computer in der Cloud aktivieren.

1. Klicken Sie auf der Registerkarte **Virtuelle Computer** in der Cloud, in dem sich der virtuelle Computer befindet, auf **Schutz aktivieren** > **Virtuelle Computer hinzufügen**. 
2. Wählen Sie aus der Liste der virtuellen Computer in der  Cloud denjenigen aus, den Sie schützen möchten.


![Enable virtual machine protection](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_EnableProtectionVM.png)

Sie können den Fortschritt der Schutzaktivierung einschließlich der ersten Replikation auf der Registerkarte **Aufträge** verfolgen. Nachdem der Auftrag zum Abschließen des Schutzes ausgeführt wurde, ist der virtuelle Computer bereit für den Failover. Nach dem Aktivieren des Schutzes und der Replikation der virtuellen Computer können Sie die virtuellen Computer in Azure sehen.

Beachten Sie, dass Sie auch Schutz für virtuelle Computer in der VMM-Konsole aktivieren können. Klicken Sie auf der Symbolleiste in der Azure Site Recovery-Registerkarte der Eigenschaften des virtuellen Computers auf "Schutz aktivieren".


![Virtual machine protection job](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_VMJobs.png)

### Integrieren vorhandener virtueller Computer
Wenn in VMM virtuelle Computer vorhanden sind, die über Hyper-V-Replikate repliziert werden, müssen Sie diese für den Schutz durch Azure Site Recovery wie folgt integrieren:
1. Stellen Sie sicher, dass Sie über primäre und sekundäre Clouds verfügen. Stellen Sie sicher, dass der Hyper-V-Hostserver für den vorhandenen virtuellen Computer sich in der primären Cloud befindet und dass der Hyper-V-Hostserver für den virtuellen Replikatcomputer sich in der sekundären Cloud befindet. Stellen Sie sicher, dass Sie die Schutzeinstellungen für die Clouds konfiguriert haben. Die Einstellungen sollten den aktuell für Hyper-V Replikate konfigurierten entsprechen. Andernfalls funktioniert die Replikation virtueller Computer möglicherweise nicht wie erwartet.
2. Aktivieren Sie anschließend den Schutz für den primären virtuellen Computer. Azure Site Recovery und VMM stellen sicher, dass derselbe Replikathost und virtuelle Computer erkannt werden. Azure Site Recovery verwendet die Replikation wieder und stellt sie anhand der Einstellungen, die während der Cloudkonfiguration festgelegt wurden, erneut her.


## Testen der Bereitstellung

Um Ihre Bereitstellung zu testen, können Sie ein Testfailover für einen einzelnen virtuellen Computer durchführen oder einen Wiederherstellungsplan erstellen, der mehrere virtuelle Computer umfasst, und ein Testfailover für diesen Plan durchführen.  Das Test-Failover simuliert Ihre Failover- und Wiederherstellungsmechanismen in einem isolierten Netzwerk.

### Erstellen eines Wiederherstellungsplans

1. Klicken Sie auf der Registerkarte **Wiederherstellungspläne** auf **Erstellen**.
2. Geben Sie einen Namen für den Wiederherstellungsplan sowie die VMM-Quell- und Zielserver an. Der Quellserver muss virtuelle Computer aufweisen, die für Failover und Wiederherstellung aktiviert sind. Wählen Sie **Hyper-V** aus, um nur Clouds anzuzeigen, die für die Hyper-V-Replikation konfiguriert sind.

	![Create recovery plan](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_RP1.png)

3. Wählen Sie in **Virtuellen Computer auswählen** Replikationsgruppen aus. Alle der Replikationsgruppe zugeordneten virtuellen Computer werden ausgewählt und dem Wiederherstellungsplan hinzugefügt. Diese virtuellen Computer werden der Standardgruppe im Wiederherstellungsplan, Gruppe 1, hinzugefügt. Sie können bei Bedarf weitere Gruppen hinzufügen. Beachten Sie, dass nach der Replikation virtuelle Computer gemäß der Reihenfolge der Wiederherstellungsplangruppen gestartet werden.

	![Add virtual machines](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_RP2.png)	

Nachdem ein Wiederherstellungsplan erstellt wurde, wird er auf der Registerkarte **Wiederherstellungspläne** angezeigt. 

###Durchführung eines Test-Failovers

1. Wählen Sie den gewünschten Wiederherstellungsplan auf der Registerkarte **Wiederherstellungspläne** aus, und klicken Sie auf **Testfailover**.
2. Wählen Sie auf der Seite **Testfailover bestätigen** den Eintrag **Kein** aus. Bei Aktivierung dieser Option werden nach einem Failover die virtuellen Computer im Replikat mit keinem Netzwerk verbunden. Es wird getestet, ob das Failover für den virtuellen Computer wie erwartet erfolgt, ohne dass Ihre Netzwerkumgebung für die Replikation getestet wird. Wenn Sie ein umfassenderes Testfailover ausführen möchten, lesen Sie <a href="http://go.microsoft.com/fwlink/?LinkId=522291">Testen einer Bereitstellung zwischen lokalen Standorten</a> in MSDN.

	![Select test network](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_TestFailover1.png)


7. Der virtuelle Computer für den Test wird auf dem Host erstellt, auf dem auch der replizierte virtuelle Computer vorhanden ist. Er wird nicht der Cloud hinzugefügt, in der sich der replizierte virtuelle Computer befindet.

### Ausführen eines Wiederherstellungsplans
Nach der Replikation hat der replizierte virtuelle Computer eine IP-Adresse, die nicht der IP-Adresse des primären virtuellen Computers entspricht. Wenn Sie Adressen mit DHCP ausstellen, wird DNS automatisch aktualisiert. Wenn Sie nicht DHCP verwenden und sicherstellen möchten, dass die Adressen identisch sind, müssen Sie verschiedene Skripts ausführen.

#### Skript zum Abrufen der IP-Adresse
Führen Sie diese Beispielskript aus, um die IP-Adresse abzurufen.
    **$vm = Get-SCVirtualMachine -Name <VM_NAME>
	$na = $vm[0].VirtualNetworkAdapters>
	$ip = Get-SCIPAddress -GrantToObjectID $na[0].id
	$ip.address**  

#### Skript zum Aktualisieren von DNS
Führen Sie dieses Beispielskript aus, um DNS zu aktualisieren, wobei Sie die IP-Adresse angeben, die Sie mithilfe des vorherigen Beispielskripts abgerufen haben.

	**[string]$Zone,
	[string]$name,
	[string]$IP
	)
	$Record = Get-DnsServerResourceRecord -ZoneName $zone -Name $name
	$newrecord = $record.clone()
	$newrecord.RecordData[0].IPv4Address  =  $IP
	Set-DnsServerResourceRecord -zonename com -OldInputObject $record -NewInputObject $Newrecord**



<a name="privacy"></a><h2>Datenschutzinformationen für Site Recovery</h2>

Dieser Abschnitt enthält zusätzliche Datenschutzinformationen für den Microsoft Azure Site Recovery-Dienst ("Dienst"). Datenschutzbestimmungen für Microsoft Azure-Dienste finden Sie in den
[Datenschutzbestimmungen für Microsoft Azure](http://go.microsoft.com/fwlink/?LinkId=324899)

**Funktion: Registrierung**

- **Funktionsbeschreibung**: Registriert Server beim Dienst, sodass virtuelle Computer geschützt werden können
- **Gesammelte Informationen**: Sammelt, verarbeitet und überträgt nach dem Registrieren des Diensts Informationen zum Verwaltungszertifikat von dem VMM-Server, der für die Bereitstellung der Notfallwiederherstellung festgelegt wurde. Dies geschieht anhand des Dienstnamens des VMM-Servers und der Namen der Clouds für virtuelle Computer auf dem VMM-Server.
- **Verwendung von Informationen**:
	- Verwaltungszertifikat - Wird verwendet, um den registrierten VMM-Server für den Zugriff auf den Dienst zu identifizieren und zu authentifizieren. Der Dienst verwendet den öffentlichen Schlüsselteil des Zertifikats zum Sichern eines Tokens, auf das nur der registrierte VMM-Server zugreifen kann. Der Server muss dieses Token für den Zugriff auf die Dienstfunktionen verwenden.
	- Name des VMM-Servers - Der VMM-Servername ist für die Identifizierung und die Kommunikation mit dem entsprechenden VMM-Server erforderlich, auf dem sich die Clouds befinden.
	- Namen der Clouds auf dem VMM-Server - Der Cloudname wird bei Verwendung der unten beschriebenen Funktion zur Dienst-Cloud-Kopplung/-Entkopplung benötigt. Wenn Sie eine Cloud von einem primären Rechenzentrum aus mit einer anderen Cloud im Wiederherstellungsrechenzentrum koppeln möchten, werden die Namen aller Clouds im Wiederherstellungsrechenzentrum aufgeführt.

- **Auswahlmöglichkeiten**: Diese Informationen sind ein wesentlicher Bestandteil des Dienstregistrierungsprozesses, da sie Sie und den Dienst bei der Identifikation des VMM-Servers, für den Sie den Azure Site Recovery-Schutz bereitstellen möchten, sowie bei der Identifikation des richtigen registrierten VMM-Servers unterstützen. Wenn Sie nicht möchten, dass diese Informationen an den Dienst gesendet werden, verwenden Sie diesen Dienst nicht. Wenn Sie den Server registrieren und die Registrierung dann später aufheben möchten, können Sie dazu die VMM-Serverinformationen im Dienstportal (dem Azure-Portal) löschen.

**Funktion: Azure Site Recovery-Schutz aktivieren**

- **Funktionsbeschreibung**: Der auf dem VMM-Server installierte Azure Site Recovery-Anbieter ist für die Kommunikation mit dem Dienst zuständig. Der Anbieter ist eine Dynamic Link Library (DLL), die im VMM-Prozess gehostet wird. Nachdem der Anbieter installiert ist, wird die Funktion "Datacenter Recovery" in der VMM-Administratorkonsole aktiviert. Alle neuen oder vorhandenen virtuellen Computer in einer Cloud können eine Eigenschaft namens "Datacenter Recovery" aktivieren, um den virtuellen Computer zu schützen. Sobald diese Eigenschaft festgelegt ist, sendet der Anbieter den Namen und die ID des virtuellen Computers an den Dienst. Der virtuelle Schutz wird in der Hyper-V-Replikationstechnologie von Windows Server 2012 oder Windows Server 2012 R2 aktiviert. Die Daten des virtuellen Computers werden von einem Hyper-V-Host auf einen anderen repliziert (der sich in der Regel in einem anderen "Recovery"-Rechenzentrum befindet).

- **Gesammelte Informationen**: Der Dienst sammelt, verarbeitet und überträgt Metadaten für den virtuellen Computer, einschließlich Name, ID, virtuelles Netzwerk und Name der Cloud, zu 
der er gehört.

- **Verwendung von Informationen**: Der Dienst füllt anhand der obigen Informationen die Informationen zum virtuellen Computer in Ihrem Dienstportal auf.

- **Auswahlmöglichkeiten**: Dies ist ein wesentlicher Bestandteil des Diensts und kann nicht deaktiviert werden. Wenn Sie nicht möchten, dass diese Informationen an den Dienst gesendet werden, aktivieren Sie nicht den Azure Site Recovery-Schutz für virtuelle Computer. Beachten Sie, dass alle Daten über HTTPS vom Anbieter an den Dienst gesendet werden.

**Funktion: Plan für die Wiederherstellung**

- **Funktionsbeschreibung**: Diese Funktion unterstützt Sie beim Erstellen eines Orchestrierungsplans für das "Recovery"-Rechenzentrum. Sie können die Reihenfolge definieren, in der die virtuellen Computer oder eine Gruppe virtueller Computer am Wiederherstellungsstandort gestartet werden sollen. Außerdem können Sie die Ausführung beliebiger automatisierter Skripts oder manueller Aktionen angeben, die zum Zeitpunkt der Wiederherstellung für jeden virtuellen Computer ausgeführt werden. Failover (im nächsten Abschnitt behandelt) wird in der Regel auf der Ebene des Wiederherstellungsplans ausgelöst, um eine koordinierte Wiederherstellung einzuleiten.

- **Gesammelte Informationen**: Der Dienst sammelt, verarbeitet und überträgt Metadaten für den Wiederherstellungsplan, einschließlich der Metadaten für virtuelle Computer sowie der Metadaten von Automatisierungsskripts und Hinweisen für manuelle Aktionen.

- **Verwendung von Informationen**: Die oben beschriebenen Metadaten werden zum Erstellen des Wiederherstellungsplans in Ihrem Dienstportal verwendet.

- **Auswahlmöglichkeiten**: Dies ist ein wesentlicher Bestandteil des Diensts und kann nicht deaktiviert werden. Wenn Sie nicht möchten, dass diese Informationen an den Dienst gesendet werden, erstellen Sie in diesem Dienst keine Wiederherstellungspläne.

**Funktion: Netzwerkzuordnung**

- **Funktionsbeschreibung**: Mithilfe dieser Funktion können Sie Netzwerkinformationen aus dem primären Rechenzentrum dem Rechenzentrum für die Wiederherstellung zuordnen. Wenn die virtuellen Computer am Wiederherstellungsstandort wiederhergestellt werden, hilft diese Zuordnung beim Herstellen der Netzwerkkonnektivität.

- **Gesammelte Informationen**: Im Rahmen der Funktion für die Netzwerkzuordnung erfasst, verarbeitet und überträgt der Dienst die Metadaten der logischen Netzwerke für die einzelnen Standorte (primär und Rechenzentrum).

- **Verwendung von Informationen**: Der Dienst verwendet die Metadaten zum Auffüllen des Dienstportals, in dem Sie die Netzwerkinformationen zuordnen können.

- **Auswahlmöglichkeiten**: Dies ist ein wesentlicher Bestandteil des Diensts und kann nicht deaktiviert werden. Wenn Sie nicht möchten, dass diese Informationen an den Dienst gesendet werden, verwenden Sie die Funktion zur Netzwerkzuordnung nicht.

**Funktion: Failover - geplant, ungeplant, Test**

- **Funktionsbeschreibung**: Mit dieser Funktion erfolgt ein Failover eines virtuellen Computers aus einem VMM-verwalteten Rechenzentrum auf ein anderes VMM-verwaltetes Rechenzentrum. Die Failoveraktion wird durch den Benutzer in ihrem Dienstportal ausgelöst. Mögliche Gründe für ein Failover sind ein ungeplantes Ereignis (z. B. im Fall einer Naturkatastrophe), ein geplantes Ereignis (z. B. ein Lastenausgleich im Rechenzentrum) oder ein Testfailover (z. B. ein Probelauf für einen Wiederherstellungsplan).

Der Anbieter auf dem VMM-Server wird vom Dienst über das Ereignis benachrichtigt und führt über VMM-Schnittstellen eine Failoveraktion auf dem Hyper-V-Host durch. Das eigentliche Failover des virtuellen Computers von einem Hyper-V-Host auf einen anderen (der in der Regel in einem anderen "Recovery"-Rechenzentrum ausgeführt wird) erfolgt über die Hyper-V-Replikationstechnologie von Windows Server 2012 oder Windows Server 2012 R2. Nachdem das Failover abgeschlossen ist, sendet der auf dem VMM-Server des "Recovery"-Rechenzentrums installierte Anbieter die Erfolgsinformationen an den Dienst.

- **Gesammelte Informationen**: Der Dienst füllt anhand der obigen Informationen den Status der Failoveraktionsinformationen in Ihrem Dienstportal auf.

- **Verwendung von Informationen**: Der Dienst verwendet die obigen Informationen wie folgt:

	- Verwaltungszertifikat - Wird verwendet, um den registrierten VMM-Server für den Zugriff auf den Dienst zu identifizieren und zu authentifizieren. Der Dienst verwendet den öffentlichen Schlüsselteil des Zertifikats zum Sichern eines Tokens, auf das nur der registrierte VMM-Server zugreifen kann. Der Server muss dieses Token für den Zugriff auf die Dienstfunktionen verwenden.
	- Name des VMM-Servers - Der VMM-Servername ist für die Identifizierung und die Kommunikation mit dem entsprechenden VMM-Server erforderlich, auf dem sich die Clouds befinden.
	- Namen der Clouds auf dem VMM-Server - Der Cloudname wird bei Verwendung der unten beschriebenen Funktion zur Dienst-Cloud-Kopplung/-Entkopplung benötigt. Wenn Sie eine Cloud von einem primären Rechenzentrum aus mit einer anderen Cloud im Wiederherstellungsrechenzentrum koppeln möchten, werden die Namen aller Clouds im Wiederherstellungsrechenzentrum aufgeführt.

- **Auswahlmöglichkeiten**: Dies ist ein wesentlicher Bestandteil des Diensts und kann nicht deaktiviert werden. Wenn Sie nicht möchten, dass diese Informationen an den Dienst gesendet werden, verwenden Sie diesen Dienst nicht.


<!--HONumber=49--> 