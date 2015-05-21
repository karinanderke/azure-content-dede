<properties 
	pageTitle="Erstellen eines benutzerdefinierten virtuellen Computers unter Linux in Azure" 
	description="Erstellen eines benutzerdefinierten virtuellen Computers unter Linux in Azure" 
	services="virtual-machines" 
	documentationCenter="" 
	authors="KBDAzure" 
	manager="timlt" 
	editor="tysonn"/>

<tags 
	ms.service="virtual-machines" 
	ms.workload="infrastructure-services" 
	ms.tgt_pltfrm="vm-linux" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/23/2015" 
	ms.author="kathydav"/>

#Erstellen eines benutzerdefinierten virtuellen Computers unter Linux in Azure

Ein *benutzerdefinierter* virtueller Computer ist einfach ein virtueller Computer, den Sie mit der Option **Aus Katalog** erstellen, da Sie durch diese Option mehr Konfigurationsmöglichkeiten erhalten als durch die Option **Schnellerfassung**. Zu diesen Möglichkeiten gehören:

- Verbinden des virtuellen Computers mit einem virtuellen Netzwerk
- Installieren von VM-Agent und -Erweiterungen z. B. für Antischadsoftware 
- Hinzufügen des virtuellen Computers zu einem vorhandenen Cloud-Dienst
- Hinzufügen des virtuellen Computers zu einem vorhandenen Speicherkonto
- Hinzufügen des virtuellen Computers zu einer Verfügbarkeitsgruppe

> [AZURE.IMPORTANT]Wenn Ihr virtueller Computer ein virtuelles Netzwerk verwenden soll, damit Sie sich direkt über den Hostnamen oder über eine lokal eingerichtete Verbindung mit ihm verbinden können, stellen Sie sicher, dass Sie das virtuelle Netzwerk schon dann angeben, wenn Sie den virtuellen Computer erstellen. Ein virtueller Computer kann so konfiguriert werden, dass er nur zu dem Zeitpunkt Teil eines virtuellen Netzwerks werden kann, zu dem er erstellt wird. Ausführliche Informationen über virtuelle Netzwerke erhalten Sie unter [Überblick über Azure Virtual Network](http://go.microsoft.com/fwlink/p/?LinkID=294063).

[AZURE.INCLUDE [Virtueller-Computer-erstellen-LinuxVM](../includes/virtual-machines-create-LinuxVM.md)]



<!--HONumber=52-->