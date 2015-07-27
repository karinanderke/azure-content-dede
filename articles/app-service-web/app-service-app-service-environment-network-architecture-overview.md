<properties 
	pageTitle="Übersicht über die Netzwerkarchitektur von App Service-Umgebungen" 
	description="Übersicht über die Architektur der Netzwerktopologie von App Service-Umgebungen" 
	services="app-service\web" 
	documentationCenter="" 
	authors="stefsch" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service-web" 
	ms.workload="web" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="07/06/2015" 
	ms.author="stefsch"/>

# Übersicht über die Netzwerkarchitektur von App Service-Umgebungen

## Einführung ##
App Service-Umgebungen werden stets in einem Subnetz eines [virtuellen Netzwerks][virtualnetwork] erstellt. Apps, die in einer App Service-Umgebung ausgeführt werden, können mit privaten Endpunkten kommunizieren, die sich in derselben virtuellen Netzwerktopologie befinden. Da Kunden Teile ihrer virtuellen Netzwerkinfrastruktur sperren können, ist es wichtig, die Arten von Netzwerkkommunikationsabläufen zu kennen, die in einer App Service-Umgebung vorkommen.

## Allgemeiner Netzwerkdatenfluss ##
 
Eine App Service-Umgebung verfügt immer über eine öffentliche virtuelle IP-Adresse (VIP). Sämtlicher eingehender Datenverkehr geht bei dieser öffentlichen VIP ein. Dazu zählen der HTTP- und HTTPS-Datenverkehr für Apps sowie anderer Datenverkehr für FTP, Remotedebuggingfunktionalität und Azure-Verwaltungsvorgänge. Eine vollständige Liste der spezifischen (erforderlichen und optionalen) Ports, die für die öffentliche VIP verfügbar sind, finden Sie im Artikel zum [Steuern des eingehenden Datenverkehrs][controllinginboundtraffic] in eine App Service-Umgebung.

Das folgende Diagramm zeigt eine Übersicht der verschiedenen ein- und ausgehenden Netzwerkdatenflüsse:

![Allgemeine Netzwerkdatenflüsse][GeneralNetworkFlows]

Eine App Service-Umgebung kann mit einer Vielzahl privater Kundenendpunkte kommunizieren. Beispielsweise können Anwendungen, die in der App Service-Umgebung ausgeführt werden, sich mit Datenbankservern auf virtuellen IaaS-Computern in der gleichen virtuellen Netzwerktopologie verbinden.

App Service-Umgebungen kommunizieren auch mit SQL-Datenbank- und Azure-Speicherressourcen, die für das Verwalten und Betreiben einer App Service-Umgebung erforderlich sind. Einige SQL- und Speicherressourcen, mit denen eine App Service-Umgebung kommuniziert, befinden sich in der gleichen Region wie die App Service-Umgebung, während andere sich in anderen Azure-Regionen befinden. Deshalb sind für den ordnungsgemäßen Betrieb einer App Service-Umgebung stets ausgehende Verbindungen erforderlich.

Da eine App-Service-Umgebung in einem Subnetz bereitgestellt wird, können Netzwerksicherheitsgruppen zum Steuern des in das Subnetz eingehenden Datenverkehrs verwendet werden. Ausführliche Informationen zum Steuern des eingehenden Datenverkehrs in eine App Service-Umgebung finden Sie im folgenden [Artikel][controllinginboundtraffic].

Details zum Zulassen ausgehender Internetverbindungen aus einer App Service-Umgebung finden Sie im folgenden Artikel über das Arbeiten mit [Express Route][ExpressRoute]. Der gleiche in diesem Artikel beschriebene Ansatz gilt beim Arbeiten mit Standort-zu-Standort-Verbindungen und Verwenden der Tunnelerzwingung.

## Ausgehende Netzwerkadressen ##
Wenn eine App Service-Umgebung ausgehende Aufrufe ausführt, ist den ausgehenden Aufrufen stets eine IP-Adresse zugeordnet. Die spezifische verwendete IP-Adresse hängt davon ab, ob sich der Endpunkt innerhalb oder außerhalb der virtuellen Netzwerktopologie befindet.

Wenn sich der aufgerufene Endpunkt **außerhalb** der Topologie des virtuellen Netzwerks befindet, ist die ausgehende Adresse (d. h. die ausgehende NAT-Adresse), die verwendet wird, die öffentliche VIP der App Service-Umgebung. Diese Adresse finden Sie auf der Portalbenutzeroberfläche der App Service-Umgebung (Hinweis: UX steht aus).

Diese Adresse kann auch ermittelt werden, indem in der App Service-Umgebung eine App erstellt und dann ein *Nslookup*-Vorgang für die Adresse der App durchgeführt wird. Die resultierende IP-Adresse ist sowohl die öffentliche VIP als auch die ausgehende NAT-Adresse der App Service-Umgebung.

Wenn sich der aufgerufene Endpunkt **innerhalb** der Topologie des virtuellen Netzwerks befindet, ist die ausgehende Adresse der aufrufenden App die interne IP-Adresse der einzelnen Computeressourcen, auf denen die App ausgeführt wird. Es gibt jedoch keine permanente Zuordnung interner IP-Adressen im virtuellen Netzwerk zu Apps. Apps können sich auf verschiedenen Computeressourcen befinden, und der Pool verfügbarer Computeressourcen in einer App Service-Umgebung kann sich aufgrund von Skalierungsvorgängen ändern.

Da sich eine App Service-Umgebung immer in einem Subnetz befindet, wird jedoch garantiert, dass die interne IP-Adresse einer Computeressource, die zum Ausführen einer App dient, stets im CIDR-Bereich des Subnetzes liegt. Wenn präzise Zugriffssteuerungslisten (ACLs) oder Netzwerksicherheitsgruppen zum Schützen des Zugriffs auf andere Endpunkte im virtuellen Netzwerk verwendet werden, muss dem Subnetzbereich mit der App Service-Umgebung Zugriff gewährt werden.

Das folgende Diagramm zeigt diese Konzepte ausführlicher:

![Ausgehende Netzwerkadressen][OutboundNetworkAddresses]

Für dieses Diagramm gilt:

- Da die öffentliche VIP-Adresse der App Service-Umgebung 192.23.1.2 lautet, ist dies die ausgehende IP-Adresse, die zum Aufrufen von Internetendpunkten verwendet wird.
- Der CIDR-Bereich des enthaltenden Subnetzes für die App Service-Umgebung ist 10.0.1.0/26. Andere Endpunkte in derselben virtuellen Netzwerkinfrastruktur erkennen, dass Aufrufe von Apps aus diesem Adressbereich stammen.

## Zusätzliche Links und Informationen ##
Einzelheiten zu von App Service-Umgebungen verwendeten eingehenden Ports und zum Verwenden von Sicherheitsgruppen zum Steuern von eingehendem Datenverkehr finden Sie [hier][controllinginboundtraffic].

Details zur Verwendung benutzerdefinierten Routen zum Gewähren des ausgehenden Internetzugriffs auf App Service-Umgebungen finden Sie in diesem [Artikel][ExpressRoute].


<!-- LINKS -->
[virtualnetwork]: http://azure.microsoft.com/services/virtual-network/
[controllinginboundtraffic]: http://azure.microsoft.com/documentation/articles/app-service-app-service-environment-control-inbound-traffic/
[ExpressRoute]: http://azure.microsoft.com/documentation/articles/app-service-app-service-environment-network-configuration-expressroute/

<!-- IMAGES -->
[GeneralNetworkFlows]: ./media/app-service-app-service-environment-network-architecture-overview/NetworkOverview-1.png
[OutboundNetworkAddresses]: ./media/app-service-app-service-environment-network-architecture-overview/OutboundNetworkAddresses-1.png

<!---HONumber=July15_HO3-->