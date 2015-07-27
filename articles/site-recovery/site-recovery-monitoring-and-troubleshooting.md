<properties
	pageTitle="Leitfaden zur Überwachung und Problembehandlung für den Schutz von VMM- und Hyper-V-Standorten" 
	description="Azure Site Recovery koordiniert Replikation, Failover und Wiederherstellung virtueller Computer auf lokalen Servern zu Azure oder einem sekundären Rechenzentrum. Verwenden Sie diesen Artikel zur Überwachung und Problembehandlung für den Schutz von VMM- und Hyper-V-Standorten." 
	services="site-recovery" 
	documentationCenter="" 
	authors="anbacker" 
	manager="mkjain" 
	editor=""/>

<tags 
	ms.service="site-recovery" 
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="storage-backup-recovery" 
	ms.date="07/03/2015" 
	ms.author="anbacker"/>
	
# Überwachung und Problembehandlung für den Schutz von VMM- und Hyper-V-Websites

In diesem Leitfaden zur Überwachung und Problembehandlung erhalten Sie Informationen zur Nachverfolgung der Replikationsintegrität und zu den Problembehandlungsverfahren für Azure Site Recovery.

## Grundlegendes zu den Komponenten

### Bereitstellung des VMM-Standorts für die Replikation zwischen lokalen Standorten.

Als Bestandteil der Einrichtung der Notfallwiederherstellung zwischen zwei lokalen Standorten muss Azure Site Recovery Provider heruntergeladen und auf dem VMM-Server installiert werden. Für Azure Site Recovery Provider muss eine Internetverbindung bestehen, um sicherzustellen, dass alle über das Azure-Portal ausgelösten Vorgänge in lokale Vorgänge umgewandelt werden, z. B. Aktivieren des Schutzes, Herunterfahren primärer virtueller Computer bei Failovers usw.

![](media/site-recovery-monitoring-and-troubleshooting/image1.png)

### Bereitstellung des VMM-Standorts für die Replikation zwischen lokalen Standorten und Azure.

Als Bestandteil der Einrichtung der Notfallwiederherstellung zwischen lokalen Standorten und Azure muss Azure Site Recovery Provider heruntergeladen und auf dem VMM-Server installiert werden. Zudem muss der Azure Recovery Services-Agent auf jedem Hyper-V-Host installiert werden.

![](media/site-recovery-monitoring-and-troubleshooting/image2.png)

### Bereitstellung des Hyper-V-Standorts für die Replikation zwischen lokalen Standorten und Azure.

Dies entspricht der Bereitstellung des VMM-Standorts mit der einzigen Ausnahme, dass der Provider und der Agent auf dem Hyper-V-Host installiert werden.

## Überwachen von Konfigurations-, Schutz- und Wiederherstellungsvorgängen

Jeder Vorgang in ASR wird überwacht und auf der Registerkarte "Aufträge" nachverfolgt. Navigieren Sie bei Fehlern bei Konfigurations-, Schutz- oder Wiederherstellungsvorgängen zur Registerkarte "Aufträge", um zu prüfen, ob dort Fehler angezeigt werden.

![](media/site-recovery-monitoring-and-troubleshooting/image3.png)

Wenn in der Ansicht "Aufträge" Fehler angezeigt werden, wählen Sie den entsprechenden Auftrag aus, und klicken Sie auf "Fehlerdetails" für diesen Auftrag.

![](media/site-recovery-monitoring-and-troubleshooting/image4.png)

Anhand der Fehlerdetails können Sie die mögliche Ursache ermitteln und erhalten Empfehlungen zum Beheben des Problems.

![](media/site-recovery-monitoring-and-troubleshooting/image5.png)

Im Fall oben scheint ein anderer aktiver Vorgang die Ursache dafür zu sein, dass bei der Konfiguration des Schutzes Fehler auftreten. Beheben Sie das Problem entsprechend der Empfehlung, und klicken Sie anschließend auf "Neu starten", um den Vorgang erneut zu starten.

![](media/site-recovery-monitoring-and-troubleshooting/image6.png)

Die Option "Neu starten" ist nicht für alle Vorgänge verfügbar. Bei den Vorgängen, für die diese Option nicht verfügbar ist, müssen Sie zurück zum Objekt navigieren und den Vorgang wiederholen. Jeder Auftrag kann während der Ausführung jederzeit über die Schaltfläche "Abbrechen" abgebrochen werden.

![](media/site-recovery-monitoring-and-troubleshooting/image7.png)

## Überwachen der Replikationsintegrität für virtuelle Computer

ASR stellt für alle geschützten Entitäten eine zentrale und Remoteüberwachung über das Azure-Portal bereit. Navigieren Sie zu den geschützten Elementen, und wählen Sie dann VMM-Clouds oder Schutzgruppen aus. Die Registerkarte "VMM-Clouds" steht nur für VMM-basierte Bereitstellungen zur Verfügung. Bei allen anderen Szenarios sind die geschützten Entitäten auf der Registerkarte "Schutzgruppen" aufgeführt.

![](media/site-recovery-monitoring-and-troubleshooting/image8.png)

Wählen Sie dann die geschützte Entität in der entsprechenden Cloud oder Schutzgruppe aus. Nach Auswahl der geschützten Einheit werden alle zulässigen Vorgänge im unteren Bereich angezeigt.

![](media/site-recovery-monitoring-and-troubleshooting/image9.png)

Im Beispiel oben ist die Integrität des virtuellen Computers gefährdet. Sie können auf die Schaltfläche "Fehlerdetails" im unteren Bereich klicken, um den Fehler anzuzeigen. Beheben Sie das Problem basierend auf den Angaben unter "Mögliche Ursachen" und "Empfehlung". In diesem Fall muss der virtuelle Computer neu synchronisiert werden. Klicken Sie dazu im Portal auf die Schaltfläche "Resynchronisieren".

![](media/site-recovery-monitoring-and-troubleshooting/image10.png)

![](media/site-recovery-monitoring-and-troubleshooting/image11.png)

Hinweis: Wenn aktive Vorgänge ausgeführt werden oder fehlgeschlagen sind, navigieren Sie entsprechend der Beschreibung weiter oben zur Ansicht "Aufträge", um den Fehler für den spezifischen Auftrag anzuzeigen.

## Problembehandlung bei lokalen Standorten

Stellen Sie eine Verbindung mit der lokalen Hyper-V-Manager-Konsole her, wählen Sie den virtuellen Computer aus, und prüfen Sie die Replikationsintegrität.

![](media/site-recovery-monitoring-and-troubleshooting/image12.png)

In diesem Fall wird die *Replikationsintegrität* als "Kritisch" angegeben. Klicken Sie auf *Replikationsstatus anzeigen*, um Details anzuzeigen.

![](media/site-recovery-monitoring-and-troubleshooting/image13.png)

#### Ereignisanzeige

| Szenarios | Ereignisquellen |
|-------------------------	|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| Schutz des VMM-Standorts | VMM-Server <ul><li> **Applications and Service Logs/Microsoft/VirtualMachineManager/Server/Admin** </li></ul> Hyper-V-Host <ul><li> **Applications and Service Logs/MicrosoftAzureRecoveryServices/Replication** (für Azure als Ziel)</li><li> **Applications and Service Logs/Microsoft/Windows/Hyper-V-VMMS/Admin** </li></ul> |
| Schutz des Hyper-V-Standorts | <ul><li> **Applications and Service Logs/MicrosoftAzureRecoveryServices/Replication** </li><li> **Applications and Service Logs/Microsoft/Azure Site Recovery/Provider/Operational** </li><li> **Applications and Service Logs/Microsoft/Windows/Hyper-V-VMMS/Admin** </li><ul>|

#### Protokollierungsoptionen für die Hyper-V-Replikation

Alle Ereignisse für Hyper-V Replica werden im Protokoll "Hyper-V-VMMS\Admin" unter **Applications and Services Logs\Microsoft\Windows** protokolliert. Darüber hinaus kann ein analytisches Protokoll für Hyper-V-VMMS aktiviert werden. Um dieses Protokoll zu aktivieren, müssen Sie zunächst die analytischen und Debugprotokolle in der Ereignisanzeige einblenden. Öffnen Sie die Ereignisanzeige, und klicken Sie im Menü **Ansicht** auf **Analytische und Debugprotokolle einblenden**.

![](media/site-recovery-monitoring-and-troubleshooting/image14.png)

Unter "Hyper-V-VMMS" wird ein analytisches Protokoll angezeigt.

![](media/site-recovery-monitoring-and-troubleshooting/image15.png)

Klicken Sie im Bereich **Aktionen** auf **Protokoll aktivieren**. Nach der Aktivierung wird das Protokoll in **Systemmonitor** als Ereignisablaufverfolgungssitzung unter **Datensammlersätze** angezeigt.

![](media/site-recovery-monitoring-and-troubleshooting/image16.png)

Beenden Sie zum Anzeigen der gesammelten Informationen zunächst die Ablaufverfolgungssitzung, indem Sie das Protokoll deaktivieren. Speichern Sie dann das Protokoll, und öffnen Sie es in der Ereignisanzeige, oder konvertieren Sie es mithilfe anderer Tools in das gewünschte Format.


## Grundlegendes zum Lebenszyklus virtueller Computer

![](media/site-recovery-monitoring-and-troubleshooting/image17.png)


## Microsoft-Support

### Protokollsammlung

Im Hinblick auf den Schutz von VMM-Standorten finden Sie Informationen zum Sammeln der erforderlichen Protokolle unter [ASR Log Collection using Support Diagnostics Platform (SDP) Tool](http://social.technet.microsoft.com/wiki/contents/articles/28198.asr-data-collection-and-analysis-using-the-vmm-support-diagnostics-platform-sdp-tool.aspx) (in englischer Sprache).

Laden Sie für den Schutz von Hyper-V-Zweigstellen und SMB-Standorten dieses [Tool](https://dcupload.microsoft.com/tools/win7files/DIAG_ASRHyperV_global.DiagCab) herunter, und führen Sie es auf dem Hyper-V-Host aus, um die Protokolle zu sammeln.

Informationen zum Sammeln der erforderlichen Protokolle für physische oder VMware-Szenarios finden Sie unter [Azure Site Recovery Log Collection for VMware and Physical site protection](http://social.technet.microsoft.com/wiki/contents/articles/30677.azure-site-recovery-log-collection-for-vmware-and-physical-site-protection.aspx) (in englischer Sprache).

Das SDP-Tool sammelt die Protokolldatei lokal. Diese finden Sie auch in einem zufällig benannten Unterordner unter **%LocalAppData%\ElevatedDiagnostics**.

### Öffnen eines Supporttickets

Um ein Supportticket für ASR zu öffnen, wenden Sie sich an den Azure-Support unter der URL <http://aka.ms/getazuresupport>.

## KB-Artikel

-   [How to preserve the drive letter for protected virtual machines (in englischer Sprache)
    > http://support.microsoft.com/kb/3031135

-   [How to troubleshoot Azure Recovery
    > Services (in englischer Sprache)\] (http://support.microsoft.com/kb/3005185)

-   [How to Enable Debug Logging for the Azure Site Recovery in Hyper-V
    > Site Protection (in englischer Sprache)\] (http://support.microsoft.com/kb/3033922)

-   [ASR: "The cluster resource could not be found" error when you try
    > to enable protection for a virtual machine (in englischer Sprache)\] (http://support.microsoft.com/kb/3010979)
    
-   [Understand & Troubleshoot Hyper-V Replica
    > Guide (in englischer Sprache)\] (http://www.microsoft.com/en-in/download/details.aspx?id=29016)
	
	
	
## Häufige Fehler bei der automatischen Systemwiederherstellung und deren Lösungen

Im Folgenden werden häufige Fehler aufgeführt, die passieren können, sowie deren Lösungen. Jeder Fehler wird auf einer separaten Wikiseite dokumentiert.

### Einrichtung

-   [Das ausgewählte Zertifikat kann nicht überprüft werden. Wählen Sie ein anderes Zertifikat aus.](http://social.technet.microsoft.com/wiki/contents/articles/25569.the-selected-certificate-cannot-be-validated-please-select-a-different-certificate.aspx)

-   [Der VMM-Server kann aufgrund eines internen Fehlers nicht registriert werden. In der Auftragsansicht im Site Recovery-Portal finden Sie weitere Details zu diesem Fehler. Führen Sie den Setup erneut aus, um den Server zu registrieren.](http://social.technet.microsoft.com/wiki/contents/articles/25570.the-vmm-server-cannot-be-registered-due-to-an-internal-error-please-refer-to-the-jobs-view-in-the-site-recovery-portal-for-more-details-on-the-error-run-setup-again-to-register-the-server.aspx)

-   [Zum Tresor Hyper-V-Recovery Manager kann keine Verbindung hergestellt werden. Überprüfen Sie die Proxyeinstellungen oder versuchen Sie es später erneut.](http://social.technet.microsoft.com/wiki/contents/articles/25571.a-connection-cant-be-established-to-the-hyper-v-recovery-manager-vault-verify-the-proxy-settings-or-try-again-later.aspx)

### Konfiguration

-   [Der Hyper-V-Hostcluster enthält mindestens einen statischen Netzwerkadapter, oder keine verbundenen Adapter sind für die Verwendung von DHCP konfiguriert.](http://social.technet.microsoft.com/wiki/contents/articles/25498.hyper-v-host-cluster-contains-at-least-one-static-network-adapter-or-no-connected-adapters-are-configured-to-use-dhcp.aspx)

-   [Das Hyper-V-Profil wird nicht in den Capability Profiles für die Cloud aktiviert.](http://social.technet.microsoft.com/wiki/contents/articles/25499.the-hyper-v-profile-isn-t-enabled-in-the-capability-profiles-for-cloud.aspx)

-   [Die Schutzkonfiguration für "%cloudName;" konnte nicht angewendet werden. Ein neu hinzugefügter Hyper-V-Host oder -Cluster konnte nicht konfiguriert werden, weil der Cloudschutz nicht konfiguriert ist.](http://social.technet.microsoft.com/wiki/contents/articles/25500.protection-configuration-for-cloudname-couldn-t-be-applied-a-newly-added-hyper-v-host-or-cluster-couldn-t-be-configured-because-cloud-protection-isn-t-configured.aspx)

-   [VMM besitzt keine Berechtigungen zum Ausführen einer Aktion](http://social.technet.microsoft.com/wiki/contents/articles/31110.vmm-does-not-have-permissions-to-complete-an-action.aspx)

### Schutz

-   [Das Betriebssystem wird nicht unterstützt.](http://social.technet.microsoft.com/wiki/contents/articles/31103.the-operating-system-is-not-supported.aspx)

-   [Aktivieren des Schutzes fehlgeschlagen, da der Agent nicht auf dem Hostcomputer installiert ist.](http://social.technet.microsoft.com/wiki/contents/articles/31105.enable-protection-failed-since-agent-not-installed-on-host-machine.aspx)

-   [Ein geeigneter Host für den replizierten virtuellen Computer wurde nicht gefunden – aufgrund geringer Compute-Ressourcen.](http://social.technet.microsoft.com/wiki/contents/articles/25501.a-suitable-host-for-the-replica-virtual-machine-can-t-be-found-due-to-low-compute-resources.aspx)

-   [Ein geeigneter Host für den replizierten virtuellen Computer wurde nicht gefunden – weil kein logisches Netzwerk angeschlossen ist.](http://social.technet.microsoft.com/wiki/contents/articles/25502.a-suitable-host-for-the-replica-virtual-machine-can-t-be-found-due-to-no-logical-network-attached.aspx)

-   [Keine Verbindung mit dem replizierten Host-Computer – Verbindung konnte nicht hergestellt werden.](http://social.technet.microsoft.com/wiki/contents/articles/31106.cannot-connect-to-the-replica-host-machine-connection-could-not-be-established.aspx)

### Wiederherstellen

-   VMM kann den Host-Vorgang nicht abschließen –

    -   [Ein Failover auf den ausgewählten Wiederherstellungspunkt für den virtuellen Computer: Allgemeiner Zugriffsfehler.](http://social.technet.microsoft.com/wiki/contents/articles/25504.fail-over-to-the-selected-recovery-point-for-virtual-machine-general-access-denied-error.aspx)

    -   [Hyper-V konnte kein Failover auf den ausgewählten Wiederherstellungspunkt für virtuelle Computer ausgeben: Der Vorgang wurde abgebrochen. Versuchen Sie es mit einem neueren Wiederherstellungspunkt. (0x80004004)](http://social.technet.microsoft.com/wiki/contents/articles/25503.hyper-v-failed-to-fail-over-to-the-selected-recovery-point-for-virtual-machine-operation-aborted-try-a-more-recent-recovery-point-0x80004004.aspx)

    -   Eine Verbindung mit dem Server konnte nicht hergestellt werden (0x00002EFD).

        -   [Fehler von Hyper-V beim Aktivieren der umgekehrten Replikation für den virtuellen Computer.](http://social.technet.microsoft.com/wiki/contents/articles/25505.a-connection-with-the-server-could-not-be-established-0x00002efd-hyper-v-failed-to-enable-reverse-replication-for-virtual-machine.aspx)

        -   [Fehler von Hyper-V beim Aktivieren der Replikation für den virtuellen Computer.](http://social.technet.microsoft.com/wiki/contents/articles/25506.a-connection-with-the-server-could-not-be-established-0x00002efd-hyper-v-failed-to-enable-replication-for-virtual-machine-virtual-machine.aspx)

    -   [Failover für virtuellen Computer konnte nicht übergeben werden.](http://social.technet.microsoft.com/wiki/contents/articles/25508.could-not-commit-failover-for-virtual-machine.aspx)

-   [Der Wiederherstellungsplan enthält virtuelle Computer, die für ein geplantes Failover nicht bereit sind.](http://social.technet.microsoft.com/wiki/contents/articles/25509.the-recovery-plan-contains-virtual-machines-which-are-not-ready-for-planned-failover.aspx)

-   [Der virtuelle Computer ist nicht bereit für ein geplantes Failover.](http://social.technet.microsoft.com/wiki/contents/articles/25507.the-virtual-machine-isn-t-ready-for-planned-failover.aspx)

-   [Der virtuelle Computer wird nicht ausgeführt und nicht ausgeschaltet.](http://social.technet.microsoft.com/wiki/contents/articles/25510.virtual-machine-is-not-running-and-is-not-powered-off.aspx)

-   [Out-of-Band-Vorgang auf einem virtuellen Computer und ](http://social.technet.microsoft.com/wiki/contents/articles/25507.the-virtual-machine-isn-t-ready-for-planned-failover.aspx)commit-Failover fehlgeschlagen.

-   Test-Failover

    -   [Failover konnte nicht initiiert werden, da der Test-Failover ausgeführt wird.](http://social.technet.microsoft.com/wiki/contents/articles/31111.failover-could-not-be-initiated-since-test-failover-is-in-progress.aspx)

-   Für die Wiederherstellung in Azure

    -   [Failover konnte nicht gestartet werden, da der VM-Name nicht unterstützte Zeichen enthält.](http://social.technet.microsoft.com/wiki/contents/articles/31107.failover-could-not-be-started-due-to-unsupported-characters-in-the-vm-name.aspx)

### Konfigurationsserver, Prozessserver, Masterziel

Konfigurationsserver (Configuration Server, CS), Prozessserver (Process Server, PS), Masterziel (Master Target, MT)

-   [Der ESXi-Host, auf dem der PS/CS als virtueller Computer gehostet wird, schlägt mit einem violetten Bildschirm fehl.](http://social.technet.microsoft.com/wiki/contents/articles/31107.vmware-esxi-host-experiences-a-purple-screen-of-death.aspx)

<!---HONumber=July15_HO2-->