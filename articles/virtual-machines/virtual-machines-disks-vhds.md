<properties
	pageTitle="Informationen zu Datenträgern und VHDs für virtuelle Computer"
	description="Machen Sie sich mit den Grundlagen von Datenträgern und VHDs für virtuelle Azure-Computern vertraut."
	services="virtual-machines"
	documentationCenter=""
	authors="KBDAzure"
	manager="timlt"
	editor="tysonn"
	tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="06/30/2015"
	ms.author="kathydav"/>

# Informationen zu Datenträgern und VHDs für virtuelle Computer

Alle virtuellen Computer in Azure werden mit mindestens zwei Datenträgern konfiguriert: ein Betriebssystem-Datenträger und ein temporärer lokaler Datenträger, der auch als Ressourcendatenträger bezeichnet wird. Der Betriebssystem-Datenträger wird aus einem Image erstellt. Sowohl der Betriebssystem-Datenträger als auch das Image sind virtuelle Festplatten (VHDs), die im Azure-Speicherkonto gespeichert sind. Virtuelle Computer können auch über Datenträger verfügen. Diese werden ebenfalls auf VHDs gespeichert.

>[AZURE.WARNING]Speichern Sie keine Daten auf dem temporären Datenträger. Er dient als temporärer Speicher für Anwendungen und Prozesse und ist ausschließlich dafür ausgelegt, Daten wie z. B. Seiten-oder Auslagerungsdateien zu speichern. Informationen zum erneuten Zuordnen dieses Datenträgers für einen virtuellen Windows-Computers finden Sie unter [Ändern des Datenträgerbuchstabens des temporären Windows-Datenträgers](virtual-machines-windows-change-drive-letter.md).

## Informationen zu Datenträgern

Virtuelle Computer in Azure verwenden wie alle anderen Computer auch einen Datenträger, auf dem das Betriebssystem, Anwendungen und Daten gespeichert sind. Alle virtuellen Azure-Computer verfügen über mindestens zwei Datenträger und einen temporären Datenträger. Sie können auch über einen oder mehrere Datenträger für Daten verfügen.

- **Betriebssystem-Datenträger**: Jedem virtuellen Computer ist ein Betriebssystem-Datenträger zugeordnet. Der Datenträger ist als SATA-Laufwerk registriert und als Laufwerk C gekennzeichnet. Der Datenträger hat eine maximale Kapazität von 1023 Gibibytes. Wenn Azure einen Betriebssystem-Datenträger erstellt wird, werden drei Kopien des Datenträgers für hohe Stabilität erstellt. Wenn Sie den virtuellen Computer für Georeplikation konfigurieren, wird die VHD außerdem auf verschiedenen weit entfernten Websites repliziert.
- Der **temporäre Datenträger** wird automatisch für Sie erstellt. Auf virtuellen Windows-Computern ist dieser Datenträger als Laufwerk D gekennzeichnet. Auf virtuellen Linux-Computern lautet der Datenträger in der Regel "/dev/sdb". Er wird vom Azure Linux Agent formatiert und an /mnt/resource angefügt.
- Ein **Datenträger** ist eine VHD, die an einen virtuellen Computer zum Speichern von Anwendungsdaten oder anderen Daten, die Sie behalten müssen, angebunden ist. Datenträger werden als SCSI-Laufwerke registriert und mit einem von Ihnen ausgewählten Buchstaben gekennzeichnet. Jeder Datenträger hat eine maximale Kapazität von 1023 Gibibytes. Die Größe des virtuellen Computers bestimmt die Anzahl der Datenträger, die Sie anfügen können, und den Typ des Speichers, den Sie zum Hosten der Datenträger verwenden können.

	Weitere Informationen zu VM-Kapazitäten, finden Sie unter [Größen für virtuelle Computer](virtual-machines-size-specs.md).

Azure erstellt einen Betriebssystem-Datenträger, wenn Sie einen virtuellen Computer aus einem Image erstellen. Wenn das verwendete Image Datenträger enthält, erstellt Azure auch diese Datenträger beim Erstellen des virtuellen Computers. (Sie können ein Image von Azure oder einem Partner oder ein eigenes Image verwenden.) Andernfalls fügen Sie die Datenträger nach dem Erstellen des virtuellen Computers hinzu.

Sie können jederzeit Datenträger zum virtuellen Computer hinzufügen, indem Sie den Datenträger an den virtuellen Computer anfügen. Sie können eine VHD verwenden, die Sie in das Speicherkonto hochgeladen oder kopiert haben, oder eine VHD, die Azure für Sie erstellt hat. Beim Anfügen eines Datenträgers wird die VHD-Datei aus dem Speicherkonto mit dem virtuellen Computer durch eine 'Lease' auf der VHD verknüpft, sodass die VHD nicht aus dem Speicher gelöscht werden kann, solange Sie an einen virtuellen Computer angefügt ist.

## Informationen zu VHDs

Die in Azure verwendeten VHDs sind VHD-Dateien, die als Seiten-Blobs in einem Standard- oder Premium-Speicherkonto in Azure gespeichert sind. (Storage Premium ist in bestimmten Regionen verfügbar.) Informationen zu Seitenblobs finden Sie unter [Grundlegendes zu Block-BLOBs und Seiten-BLOBs](https://msdn.microsoft.com/library/ee691964.aspx). Informationen zu Storage Premium finden Sie unter [Premium-Speicher: Hochleistungsspeicher für Workloads auf virtuellen Azure-Computern](../storage-premium-storage-preview-portal.md).

Außerhalb von Azure können virtuelle Festplatten entweder das VHD- oder ein VHDX-Format verwenden. Sie können auch fest, dynamisch erweiterbar oder differenzierend sein. Azure unterstützt das VHD-Format; feste Datenträger. Das feste Format legt den logischen Datenträger in der Datei linear aus, daher wird Datenträger-Offset X bei Blob-Offset X gespeichert. Eine kleinere Fußzeile am Ende des Blobs beschreibt die Eigenschaften der VHD. Oftmals verschwendet das feste Format Speicherplatz, da die meisten Datenträger über große ungenutzte Bereiche davon verfügen. Azure speichert VHD-Dateien jedoch in einem platzsparenden Format. Daher profitieren Sie gleichzeitig von den Vorteilen fester und dynamischer Datenträger. Nähere Informationen finden Sie unter [Erste Schritte mit virtuellen Festplatten](https://technet.microsoft.com/library/dd979539.aspx).

Alle VHD-Dateien in Azure, die Sie als Quelle zum Erstellen von Datenträgern oder Images verwenden mögchten, sind schreibgeschützt. Beim Erstellen eines Datenträgers oder Images werden Kopien der VHD-Dateien von Azure erstellt. Diese Kopien sind je nach Verwendung der VHD entweder schreibgeschützt oder ermöglichen Lese- und Schreibvorgänge.

 Beim Erstellen eines virtuellen Computers mithilfe eines Images, erstellt Azure einen Datenträger für den virtuellen Computer, bei dem es sich um eine Kopie der VHD-Quelldatei handelt. Um ein versehentliches Löschen zu vermeiden, setzt Azure eine Lease für alle VHD-Quelldateien, die zum Erstellen eines Images, Betriebssystem-Datenträgers oder Datenträgers für Daten verwendet wird.

Um eine VHD-Quelldatei löschen zu können, müssen Sie die Lease entfernen, indem Sie den Datenträger oder das Image löschen. Um eine VHD-Datei zu löschen, die von einem virtuellen Computer als Betriebssystem-Datenträger verwendet wird, können Sie den virtuellen Computer, den Betriebssystem-Datenträger und die VHD-Quelldatei auf einmal entfernen, indem Sie den virtuellen Computer und alle verbundenen Datenträger löschen. Zum Entfernen einer VHD-Datei, die als Quelle für einen Datenträger verwendet wird, sind jedoch mehrere Schritte in einer bestimmten Reihenfolge erforderlich: Trennen Sie den Datenträger vom virtuellen Computer, löschen Sie den Datenträger und dann die VHD-Datei.

>[AZURE.WARNING]Wenn Sie eine VHD-Quelldatei aus dem Speicher löschen oder das Speicherkonto löschen, kann Microsoft diese Daten nicht wiederherstellen.

## Nächste Schritte

Virtuelle Linux-Computer:

-  [Anfügen eines Datenträgers und Vorbereiten des Datenträgers für die Verwendung](virtual-machines-linux-how-to-attach-disk.md)
-  [Erfassen eines virtuellen Linux-Computers](virtual-machines-linux-capture-image.md)
-  [Trennen eines Datenträgers](virtual-machines-linux-how-to-detach-disk.md)

Virtuelle Windows-Computer:

-  [Anfügen eines Datenträgers und Vorbereiten des Datenträgers für die Verwendung](storage-windows-attach-disk.md)
-  [Erfassen eines virtuellen Windows-Computers](virtual-machines-capture-image-windows-server.md)
-  [Trennen eines Datenträgers](storage-windows-detach-disk.md)

<!---HONumber=July15_HO2-->