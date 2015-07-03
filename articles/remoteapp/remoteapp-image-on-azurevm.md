<properties 
    pageTitle="Erstellen von RemoteApp-Images auf Grundlage einer Azure-VM"
    description="Erfahren Sie, wie Sie ein Image für RemoteApp basierend auf einem virtuellen Azure-Computer erstellen." 
    services="remoteapp" 
    documentationCenter="" 
    authors="lizap" 
    manager="mbaldwin" />

<tags 
    ms.service="remoteapp" 
    ms.workload="compute" 
    ms.tgt_pltfrm="na" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="03/27/2015" 
    ms.author="elizapo" />



# Erstellen von RemoteApp-Images auf Grundlage eines virtuellen Azure-Computers

Auf Grundlage des umfassenden Kundenfeedbacks haben wir jetzt die Unterstützung für das Erstellen von RemoteApp-Images (mit den freigegebenen Apps Ihrer Sammlung) in einem virtuellen Azure-Computer implementiert. Wir haben weiterhin ein Image eines virtuellen Computers zum Azure-VM-Imagekatalog hinzugefügt, das allen Anforderungen an RemoteApp-Images entspricht. Sie können dieses VM-Image als Ausgangspunkt für eigene VMs verwenden. Suchen Sie in der Bibliothek einfach nach dem Image "Windows Server Remote Desktop Session Host".

Der Vorgang umfasst zwei Schritte - Erstellen des Images und anschließendes Hochladen aus der Azure-VM-Bibliothek nach RemoteApp.

## Erstellen benutzerdefinierter Images auf Grundlage einer Azure-VM

Führen Sie diese Schritte zum Erstellen eines Images auf Grundlage eines Azure-VM aus.

1. Erstellen Sie einen virtuellen Azure-Computer. Sie können das Image "Windows Server Remote Desktop Session Host" aus dem Imagekatalog für virtuelle Azure-Computer verwenden. Dieses Image erfüllt alle Anforderungen an Azure-RemoteApp-Vorlagenimages. 

	Weitere Informationen finden Sie unter [Erstellen eines virtuellen Windows Server-Computers](virtual-machines-windows-tutorial.md).

2. Stellen Sie eine Verbindung mit dem virtuellen Computer her. Installieren und konfigurieren Sie dann die Apps, die Sie über RemoteApp freigeben möchten. Führen Sie alle weiteren von den Apps benötigten Windows-Konfigurationen aus. 

	Ausführliche Informationen finden Sie unter [Anmelden bei einem virtuellen Computer, auf dem Windows Server ausgeführt wird](virtual-machines-log-on-windows-server.md).. 

3. Wenn Sie das Image "Windows Server Remote Desktop Session Host" verwenden, können Sie das enthaltene Validierungsskript nutzen, um sicherzustellen, dass Ihr virtueller Computer den Anforderungen an RemoteApp entspricht. Doppelklicken Sie zur Ausführung des Skripts auf dem Desktop auf **ValidateRemoteAppImage**. Stellen Sie sicher, dass alle vom Skript gemeldeten Fehler behoben wurden, bevor Sie mit dem nächsten Schritt fortfahren.

4. Führen Sie Sysprep mit "/generalize" aus, und erfassen Sie das Image. Weitere Informationen dazu finden Sie unter [Erfassen eines virtuellen Windows-Computers, um ihn als Vorlage zu verwenden](../virtual-machines-capture-image-windows-server.md).

 

## Importieren von Images in die RemoteApp-Imagebibliothek

Führen Sie diese Schritte aus, um das neue Image in RemoteApp zu importieren:

1. Auf der Registerkarte **Template Images**:
	- Wenn keine Images vorhanden sind, klicken Sie auf **Upload or Import a Template Image**. 
	- Wenn mindestens ein Image vorliegt, klicken Sie auf **+**, um ein neues Image hinzuzufügen.

2. Wählen Sie die Option **Import an image from your Virtual Machines library** aus, und klicken Sie dann auf **Weiter**.

3. Wählen Sie auf der nächsten Seite das benutzerdefinierte Image aus der Liste aus, und bestätigen Sie, dass Sie die bei der Erstellung des Images aufgeführten Schritte ausgeführt haben. Klicken Sie auf **Weiter**.
4. Geben Sie einen Namen für das neue RemoteApp-Image ein, und wählen Sie den Speicherort aus. Klicken Sie dann auf das Häkchen, um den Importvorgang zu starten.

> [AZURE.NOTE] Sie können Images von jedem Azure-Speicherort, der virtuelle Azure-Computer unterstützt, in jeden Azure-Speicherort importieren, der von Azure-RemoteApp unterstützt wird. Je nach Speicherort kann der Importvorgang bis zu 25 Minuten dauern.

Sie können jetzt Ihre neue Sammlung erstellen - je nach Anforderung entweder eine [Cloud-Sammlung](remoteapp-create-cloud-deployment.md) oder eine [Hybrid-sammlung](remoteapp-create-hybrid-deployment.md).


<!--HONumber=52--> 