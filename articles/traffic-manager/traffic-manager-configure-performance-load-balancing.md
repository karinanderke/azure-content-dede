<properties 
   pageTitle="Konfigurieren der Lastenausgleichsmethode "Leistung""
   description="In diesem Artikel finden Sie Informationen zum Konfigurieren der Lastenausgleichsmethode "Leistung""
   services="traffic-manager"
   documentationCenter=""
   authors="cherylmc"
   manager="adinah"
   editor="tysonn" />
<tags 
   ms.service="traffic-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="02/27/2015"
   ms.author="cherylmc" />

# Konfigurieren der Lastenausgleichsmethode "Leistung"

Wenn Sie Lastenausgleich für Cloud-Dienste und Websites (Endpunkte) in verschiedenen Datencentern weltweit (die auch als "Regionen" bezeichnet werden) ausführen möchten, können Sie eingehenden Datenverkehr an den Endpunkt mit der geringsten Wartezeit zwischen dem anfordernden Client und dem Endpunkt weiterleiten. Normalerweise entspricht das Datencenter mit der geringsten Wartezeit der kürzesten geografischen Distanz. Die Lastenausgleichsmethode Leistung ermöglicht Ihnen, den Datenverkehr unter Berücksichtigung der Wartezeit zu verteilen. Echtzeitänderungen an der Netzwerkkonfiguration oder der Auslastung können dabei jedoch nicht berücksichtigt werden. Weitere Informationen zu den verschiedenen Lastenausgleichsmethoden, die von Azure Traffic Manager bereitgestellt werden, finden Sie unter [Traffic Manager-Lastenausgleichsmethoden](traffic-manager-load-balancing-methods.md)..

## Führen Sie den Lastenausgleich für Datenverkehr basierend auf der geringsten Wartezeit für eine Gruppe von Endpunkten aus

1. Klicken Sie im Verwaltungsportal im linken Bereich auf das Symbol **Traffic Manager**, um den Bereich Traffic Manager zu öffnen. Wenn Sie noch kein Traffic Manager-Profil erstellt haben, finden Sie unter [Verwalten von Traffic Manager-Profilen](traffic-manager-manage-profiles.md) Anweisungen zum Erstellen eines einfachen Traffic Manager-Profils.
2. Suchen Sie im Verwaltungsportal im Traffic Manager-Bereich das Traffic Manager-Profil mit den Einstellungen, die Sie ändern möchten, und klicken Sie dann auf den Pfeil rechts neben dem Profilnamen. Die Einstellungsseite für das Profil wird geöffnet.
3. Klicken Sie auf der Seite für Ihr Profil oben auf **Endpunkte**, und prüfen Sie, ob die Dienstendpunkte, die Sie einschließen möchten, in Ihrer Konfiguration vorhanden sind. Die Schritte zum Hinzufügen oder Entfernen von Endpunkten zu/aus Ihrem Profil finden Sie unter [Verwalten von Endpunkten in Traffic Manager](traffic-manager-endpoints.md).
4. Klicken Sie auf der Seite Ihres Profils oben auf **Konfigurieren**, um die Konfigurationsseite zu öffnen.
5. Überprüfen Sie in den **Einstellungen der Lastenausgleichsmethode**, ob die Lastenausgleichsmethode **Leistung* ist. Klicken Sie andernfalls in der Dropdownliste auf **Leistung**.
6. Stellen Sie sicher, dass die **Überwachungseinstellungen** ordnungsgemäß konfiguriert sind. Durch die Überwachung wird sichergestellt, dass die Endpunkte, die offline sind, keinen Datenverkehr empfangen. Um Endpunkte zu überwachen, müssen Sie einen Pfad und einen Dateinamen angeben. Beachten Sie, dass der Schrägstrich "/" ein gültiger Eintrag für den relativen Pfad ist und bedeutet, dass sich die Datei im Stammverzeichnis (Standardwert) befindet. Weitere Informationen zur Überwachung finden Sie unter [Traffic Manager-Überwachung](traffic-manager-monitoring.md)
7. Klicken Sie nach der Durchführung der Konfigurationsänderungen unten auf der Seite auf **Speichern**.
8. Testen Sie die Änderungen in Ihrer Konfiguration. Weitere Informationen finden Sie unter [Testen von Traffic Manager-Einstellungen](traffic-manager-testing-settings.md).
9. Sobald das Traffic Manager-Profil eingerichtet und funktionsfähig ist, bearbeiten Sie den DNS-Eintrag auf dem autoritativen DNS-Server, damit Ihre Unternehmensdomäne auf den Namen der Traffic Manager-Domäne verweisen kann. Weitere Informationen hierzu finden Sie unter [Verweisen auf eine Traffic Manager-Domäne mit der Internetdomäne eines Unternehmens](traffic-manager-point-internet-domain.md).

## Siehe auch

[Informationen zu Lastenausgleichsmethoden von Traffic Manager](traffic-manager-load-balancing-methods.md)

[Traffic Manager-Konfigurationsaufgaben](https://msdn.microsoft.com/library/azure/hh744830.aspx)

[Traffic Manager - Übersicht](../traffic-manmager-overview.md)

[Cloud Services](http://go.microsoft.com/fwlink/?LinkId=314074)

[Websites](http://go.microsoft.com/fwlink/p/?LinkId=393327)

[Vorgänge für Traffic Manager (REST-API-Referenz)](http://go.microsoft.com/fwlink/?LinkId=313584)

[Azure Traffic Manager-Cmdlets](http://go.microsoft.com/fwlink/p/?LinkId=400769)


<!--HONumber=49--> 