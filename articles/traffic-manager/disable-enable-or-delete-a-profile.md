<properties
   pageTitle="Deaktivieren, Aktivieren oder Löschen eines Traffic Manager-Profils"
   description="In diesem Artikel wird erläutert, wie Sie mit Traffic Manager-Profilen arbeiten."
   services="traffic-manager"
   documentationCenter="na"
   authors="cherylmc"
   manager="adinah"
   editor="tysonn" />
<tags 
   ms.service="traffic-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="02/23/2015"
   ms.author="cherylmc" />

# Deaktivieren, Aktivieren oder Löschen eines Profils

## Deaktivieren oder Aktivieren eines Profils

Sie können ein vorhandenes Traffic Manager-Profil so deaktivieren, dass keine Benutzeranforderungen mehr an die konfigurierten Endpunkte verwiesen werden. Wenn Sie ein Traffic Manager-Profil deaktivieren, bleiben das Profil selbst und die Informationen im Profil erhalten und können auf der Traffic Manager-Benutzeroberfläche bearbeitet werden. Wenn Sie das Profil erneut aktivieren möchten, können Sie dies problemlos im Verwaltungsportal tun, woraufhin die Weiterleitung wird aufgenommen wird. Wenn Sie ein Traffic Manager-Profil im Verwaltungsportal erstellen, wird es automatisch aktiviert.

### So deaktivieren Sie ein Profil

1. Ändern Sie den DNS-Ressourceneintrag auf dem Internet-DNS-Server so, dass der entsprechende Eintragstyp und Zeiger auf einen anderen Namen oder die IP-Adresse eines bestimmten Speicherorts im Internet verwendet werden. Ändern Sie also den DNS-Ressourceneintrag auf dem Internet-DNS-Server so, dass er nicht mehr einen CNAME-Ressourceneintrag verwendet, der auf den Domänennamen Ihres Traffic Manager-Profils verweist.
1. Der Datenverkehr wird nicht mehr über die Traffic Manager-Profileinstellungen an die Endpunkte weitergeleitet.
1. Wählen Sie das Profil aus, das Sie deaktivieren möchten. Um das Profil auszuwählen, markieren Sie das Profil auf der Seite "Traffic Manager" durch Klicken auf die Spalte neben dem Profilnamen. Klicken Sie nicht auf den Namen des Profils oder den Pfeil neben dem Namen, da Sie dadurch zur Einstellungsseite für das Profil gelangen.
1. Klicken Sie nach Auswahl des Profils am unteren Rand der Seite auf "Deaktivieren".

### So aktivieren Sie ein Profil

1. Wählen Sie das Profil aus, das Sie aktivieren möchten. Um das Profil auszuwählen, markieren Sie das Profil auf der Seite "Traffic Manager" durch Klicken auf die Spalte neben dem Profilnamen. Klicken Sie nicht auf den Namen des Profils oder den Pfeil neben dem Namen, da Sie dadurch zur Einstellungsseite für das Profil gelangen.
1. Klicken Sie nach Auswahl des Profils am unteren Rand der Seite auf "Aktivieren".
1. Ändern Sie den DNS-Ressourceneintrag auf dem Internet-DNS-Server so, dass der mit CNAME-Eintragstyp verwendet wird, der den Domänennamen Ihres Unternehmens dem Domänennamen Ihres Traffic Manager-Profils zuordnet. Weitere Informationen finden Sie unter [Verweisen auf eine Traffic Manager-Domäne mit der Internetdomäne eines Unternehmens] (point-a-company-internet-domain-to-a-traffic-manager-domain.md).
1. Der Datenverkehr wird wieder an die Endpunkte weitergeleitet.

## Löschen eines Profils


### So löschen Sie ein Profil

1. Ändern Sie den DNS-Ressourceneintrag auf dem Internet-DNS-Server so, dass er nicht mehr einen CNAME-Ressourceneintrag verwendet, der auf den Domänennamen Ihres Traffic Manager-Profils verweist.
1. Wählen Sie das Profil aus, das Sie löschen möchten. Um das Profil auf der Seite "Traffic Manager" auszuwählen, markieren Sie das Profil, 
1. indem Sie auf die Spalte neben dem Profil klicken. Klicken Sie nicht auf den Namen des Profils oder den Pfeil neben dem Namen, da Sie dadurch zur Einstellungsseite für das Profil gelangen.
1. Klicken Sie nach Auswahl des Profils am unteren Rand der Seite auf "Löschen".

## Siehe auch

[Traffic Manager](../traffic-manager.md)

[Traffic Manager-Konfigurationsaufgaben](https://msdn.microsoft.com/library/azure/hh744830.aspx)



<!--HONumber=49--> 