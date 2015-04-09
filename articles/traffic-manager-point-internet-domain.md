﻿<properties
   pageTitle="Verweisen einer Unternehmens-Internetdomäne auf eine Traffic Manager-Domäne"
   description="In diesem Artikel erfahren Sie, wie Sie mit Ihrem Unternehmensdomänennamen auf einen Traffic Manager-Domänennamen verweisen."
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

# Verweisen einer Unternehmens-Internetdomäne auf eine Traffic Manager-Domäne

Um mit Ihrem Unternehmensdomänennamen auf einen Traffic Manager-Domänennamen zu verweisen, ändern Sie den DNS-Ressourceneintrag auf dem Internet-DNS-Server mit dem CNAME-Eintragstyp, der den Domänennamen Ihres Unternehmens dem Domänennamen Ihres Traffic Manager-Profils zuordnet. Sie finden den Traffic Manager-Domänennamen auf der Konfigurationsseite des Traffic Manager-Profils im Abschnitt **Allgemein**.

Um beispielsweise mit dem Unternehmens-Domänennamen **www.contoso.com** auf den Traffic Manager-Domänennamen **contoso.trafficmanager.net** zu verweisen, ändern Sie den DNS-Ressourceneintrag wie folgt:

    www.contoso.com IN CNAME contoso.trafficmanager.net 

Alle Datenverkehrsanforderungen an *www.contoso.com* werden jetzt an *contoso.trafficmanager.net* umgeleitet.

>[AZURE.IMPORTANT]Sie können nicht mit Domänen der zweiten Ebene wie *contoso.com* auf Traffic Manager-Domänen verweisen. Dies ist eine Einschränkung des DNS-Protokolls, das CNAME-Datensätze für Domänennamen der zweiten Ebene nicht zulässt.

## Siehe auch

[Informationen zu Lastenausgleichsmethoden von Traffic Manager](traffic-manager-load-balancing-methods.md)

[Traffic Manager-Konfigurationsaufgaben](https://msdn.microsoft.com/library/azure/hh744830.aspx)

[Traffic Manager - Übersicht](traffic-manager-overview.md)

<!--HONumber=49-->