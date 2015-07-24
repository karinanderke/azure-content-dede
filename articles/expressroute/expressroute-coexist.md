<properties
   pageTitle="Konfigurieren Sie gleichzeitig bestehende ExpressRoute und Standort-zu-Standort-VPN-Verbindungen."
   description="Dieses Lernprogramm führt Sie durch die Konfiguration von gleichzeitig bestehenden- und Standort-zu-Standort-VPN-Verbindungen."
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="adinah"
   editor="tysonn" />
<tags
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="article" 
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="06/30/2015"
   ms.author="cherylmc"/>

# Konfigurieren von gleichzeitig vorhandenen ExpressRoute- und Standort-zu-Standort-VPN-Verbindungen

Sie können jetzt ExpressRoute und Standort-zu-Standort-VPN mit dem gleichen virtuellen Netzwerk verbinden. Sie können zwischen zwei verschiedenen Szenarien und zwei unterschiedlichen Konfigurationsverfahren wählen.

### Szenarios

Es gibt zwei verschiedene Szenarien. Die folgende Abbildung stellt beide Szenarien dar.

- **Szenario 1**– Wenn Sie ein lokales Netzwerk haben, ist ExpressRoute der aktive Link und Ihr Standort-zu-Standort-VPN wäre das Backup. Wenn die ExpressRoute-Verbindung fehlschlägt, wird die Standort-zu-Standort-VPN-Verbindung aktiviert. Dieses Szenario ist am besten für hohe Verfügbarkeit geeignet.

- **Szenario 2**– Wenn Sie über zwei lokale Netzwerke verfügen, können Sie eines mit Azure über ExpressRoute verbinden und das andere über Standort-zu-Standort-VPN. In diesem Fall sind beide Verbindungen gleichzeitig aktiv.


![Koexistenz](media/expressroute-coexist/coexist1.jpg)


### Konfigurationsverfahren

Es stehen zwei separate Konfigurationsverfahren zur Auswahl. Welches Konfigurationsverfahren Sie wählen, hängt davon ab, ob Sie bereits über ein vorhandenes virtuelles Netzwerk verfügen, mit dem Sie eine Verbindung aufbauen möchten, oder ob Sie ein neues virtuelles Netzwerk erstellen möchten.


- [Erstellen eines neuen VNet mit gleichzeitig bestehenden Verbindungen](#create-a-new-vnet-with-coexisting-connections): Wenn Sie bereits über ein virtuelles Netzwerk verfügen, wird Sie dieses Verfahren durch die Schritte zum Erstellen eines neuen virtuellen Netzwerks und neuer ExpressRoute- sowie Standort-zu-Standort-VPN-Verbindungen führen.  


- [Konfigurieren Ihres vorhandenen VNet für gleichzeitig bestehende Verbindungen](#configure-your-existing-vnet-for-coexisting-connections): Möglicherweise haben Sie bereits ein virtuelles Netzwerk mit einer vorhandenen Standort-zu-Standort-VPN-Verbindung oder einer ExpressRoute-Verbindung. Um eine gleichzeitig bestehende Verbindung zu erstellen, müssen Sie Ihr Gateway löschen und anschließend neue Gateways konfigurieren, die gleichzeitig bestehen können. Das bedeutet, dass Sie für Ihre standortübergreifenden Verbindungen Ausfallzeiten haben werden, während Sie Ihr Gateway und Ihre Verbindungen löschen und neu erstellen, aber Sie müssen keine Ihrer VMs oder Dienste auf ein neues virtuelles Netzwerk migrieren. Die VMs und Dienste wären immer noch in der Lage, über den Load Balancer zu kommunizieren, während Sie Ihr Gateway konfigurieren, wenn sie zu diesem Zweck konfiguriert sind.

	Dieses Verfahren führt Sie durch die Schritte zum Löschen des Gateways und anschließendem Erstellen neuer ExpressRoute- und Standort-zu-Standort-VPN-Verbindungen. Beachten Sie, dass die Schritte beim Erstellen von neuen Verbindungen in einer ganz bestimmten Reihenfolge ausgeführt werden müssen. Verwenden Sie die Anweisungen nicht in anderen Artikeln, um Ihre Gateways und Verbindungen zu erstellen.

### Hinweise und Einschränkungen

- Sowohl das ExpressRoute-Gateway als auch das Standort-zu-Standort-VPN-Gateway müssen entweder eine Standard- oder Hochleistungs-Gateway-SKU sein. Informationen zu Gateway-SKUs finden Sie unter [Gateway-SKUs](../vpn-gateway/vpn-gateway-about-vpngateways.md).
- Wenn Ihr lokales Netzwerk mit ExpressRoute und einem Standort-zu-Standort-VPN (Szenario 1) verbunden ist, sollten Sie eine statische Route in Ihrem lokalen Netzwerk konfiguriert haben, um die Standort-zu-Standort-VPN-Verbindung an das öffentliche Internet weiterzuleiten. 
- Sie müssen zuerst das ExpressRoute-Gateway erstellen, bevor Sie das Standort-zu-Standort-VPN-Gateway hinzufügen.
- Ein Routing (über Azure) zwischen Ihrem lokalen Netzwerk mit Standort-zu-Standort-VPN-Verbindung und Ihrem lokalen Netzwerk mit ExpressRoute-Verbindung wird nicht möglich sein.
- Beide Verfahren gehen davon aus, dass Sie bereits eine ExpressRoute-Verbindung konfiguriert haben. Ist dies nicht der Fall, lesen Sie bitte die folgenden Artikel: 

	- [Konfigurieren einer ExpressRoute-Verbindung über einen Netzwerkdienstanbieter (NSP)](expressroute-configuring-nsps.md) 
	- [Konfigurieren einer ExpressRoute-Verbindung über einen Exchange-Anbieter (EXP)](expressroute-configuring-exps.md)


## Erstellen Sie ein neues VNet mit gleichzeitig bestehenden Verbindungen

1. Stellen Sie sicher, dass Sie über die neueste Version von PowerShell-Cmdlets verfügen. Sie können die neuesten PowerShell-Cmdlets aus dem PowerShell-Abschnitt der [Downloadseite](http://azure.microsoft.com/downloads/) herunterladen und installieren.
2. Geben Sie Ihrem Virtual Network einen Namen. Weitere Informationen zum Arbeiten mit der Netzwerk-Konfigurationsdatei finden Sie unter [Konfigurieren eines Virtual Network mithilfe einer Netzwerk-Konfigurationsdatei](../virtual-network/virtual-networks-using-network-configuration-file.md). Weitere Informationen zum Konfigurationsschema finden Sie unter [Konfigurationsschema für Azure Virtual Network](https://msdn.microsoft.com/library/azure/jj157100.aspx).

	Wenn Sie Ihr Schema erstellen, stellen Sie sicher, dass Sie die folgenden Werte verwenden:

	- Das Gateway-Subnetz für das virtuelle Virtual Network muss /27 sein (oder ein kürzeres Präfix).
	- Der Gateway-Verbindungstyp ist "dediziert".

		      <VirtualNetworkSite name="MyAzureVNET" Location="Central US">
		        <AddressSpace>
		          <AddressPrefix>10.17.159.192/26</AddressPrefix>
		        </AddressSpace>
		        <Subnets>
		          <Subnet name="Subnet-1">
		            <AddressPrefix>10.17.159.192/27</AddressPrefix>
		          </Subnet>
		          <Subnet name="GatewaySubnet">
		            <AddressPrefix>10.17.159.224/27</AddressPrefix>
		          </Subnet>
		        </Subnets>
		        <Gateway>
		          <ConnectionsToLocalNetwork>
		            <LocalNetworkSiteRef name="MyLocalNetwork">
		              <Connection type="Dedicated" />
		            </LocalNetworkSiteRef>
		          </ConnectionsToLocalNetwork>
		        </Gateway>
		      </VirtualNetworkSite>



3. Laden Sie nach dem Erstellen und Konfigurieren Ihrer XML-Schemadatei die Datei hoch. Dadurch wird Ihr Virtual Network erstellt.

	Verwenden Sie das folgende Cmdlet zum Hochladen Ihrer Datei und um den Wert mit Ihrem eigenen zu ersetzen.

	`Set-AzureVNetConfig -ConfigurationPath 'C:\NetworkConfig.xml'`
4. Erstellen eines ExpressRoute-Gateway. Achten Sie darauf, dass Sie die GatewaySKU als *Standard* oder *HighPerformance* angeben, und den GatewayType als *DynamicRouting*. 

	Verwenden Sie das Beispiel unten, um die Werte durch Ihre eigenen zu ersetzen.

	`New-AzureVNetGateway -VNetName MyAzureVNET -GatewayType DynamicRouting -GatewaySKU HighPerformance`

5. Verknüpfen Sie das ExpressRoute-Gateway mit dem ExpressRoute-Schaltkreis. Nachdem dieser Schritt abgeschlossen ist, wird die Verbindung zwischen dem lokalen Netzwerk und Azure durch ExpressRoute eingerichtet.

	`New-AzureDedicatedCircuitLink -ServiceKey <service-key> -VNetName MyAzureVNET`

6. Erstellen Sie als Nächstes Ihr Standort-zu-Standort-VPN-Gateway. Die GatewaySKU muss *Standard* oder *HighPerformance* sein, und der GatewayType *DynamicRouting*.

	`New-AzureVirtualNetworkGateway -VNetName MyAzureVNET -GatewayName S2SVPN -GatewayType DynamicRouting -GatewaySKU  HighPerformance`

	Verwenden Sie zum Abrufen der Einstellungen des Virtual Network-Gateway, einschließlich der Gateway-ID und der öffentlichen IP, das`Get-AzureVirtualNetworkGateway` Cmdlet.

		Get-AzureVirtualNetworkGateway
		
		GatewayId            : 348ae011-ffa9-4add-b530-7cb30010565e
		GatewayName          : S2SVPN
		LastEventData        :
		GatewayType          : DynamicRouting
		LastEventTimeStamp   : 5/29/2015 4:41:41 PM
		LastEventMessage     : Successfully created a gateway for the following virtual network: GNSDesMoines
		LastEventID          : 23002
		State                : Provisioned
		VIPAddress           : 104.43.x.y
		DefaultSite          :
		GatewaySKU           : HighPerformance
		Location             :
		VnetId               : 979aabcf-e47f-4136-ab9b-b4780c1e1bd5
		SubnetId             :
		EnableBgp            : False
		OperationDescription : Get-AzureVirtualNetworkGateway
		OperationId          : 42773656-85e1-a6b6-8705-35473f1e6f6a
		OperationStatus      : Succeeded

7. Erstellen Sie eine VPN-Gateway-Entität für einen lokalen Standort. Mit diesem Befehl wird das lokale VPN-Gateway nicht konfiguriert. Stattdessen können Sie mit ihm die Einstellungen des lokalen Gateway bereitstellen, wie z. B. die öffentliche IP-Adresse und der lokale Adressraum, sodass eine Verbindung mit dem Azure-VPN-Gateway hergestellt werden kann.

	**Wichtig:** Der lokale Standort für das Standort-zu-Standort-VPN ist nicht in der Netcfg-Datei definiert. Stattdessen müssen Sie dieses Cmdlet verwenden, um die lokalen Standort-Parameter anzugeben. Sie können sie nicht über das Verwaltungsportal oder die Netcfg-Datei definieren.

	Ersetzen Sie anhand des folgenden Beispiels die Werte mit Ihren eigenen:

	`New-AzureLocalNetworkGateway -GatewayName MyLocalNetwork -IpAddress <local-network- gateway-public-IP> -AddressSpace <local-network-address-space>`

	Verwenden Sie zum Abrufen der Einstellungen des Virtual Network-Gateway, einschließlich der Gateway-ID und der öffentlichen IP, das`Get-AzureVirtualNetworkGateway` Cmdlet. Betrachten Sie das folgende Beispiel:

		Get-AzureLocalNetworkGateway
		
		GatewayId            : 532cb428-8c8c-4596-9a4f-7ae3a9fcd01b
		GatewayName          : MyLocalNetwork
		IpAddress            : 23.39.x.y
		AddressSpace         : {10.1.2.0/24}
		OperationDescription : Get-AzureLocalNetworkGateway
		OperationId          : ddc4bfae-502c-adc7-bd7d-1efbc00b3fe5
		OperationStatus      : Succeeded

	
8. Konfigurieren Sie Ihr lokales VPN-Gerät für die Verbindung zum neuen Gateway. Verwenden Sie die Informationen, die Sie in Schritt 6 abgerufen haben, wenn Sie Ihr VPN-Gerät konfigurieren. Weitere Informationen zu VPN-Gerätekonfiguration finden Sie unter [VPN-Gerätekonfiguration](vpn-gateway-configure-vpn-gateway-mp.md/#gather-information-for-your-vpn-device-configuration).
	

9. Verknüpfen Sie das Standort-zu-Standort-VPN-Gateway in Azure mit dem lokalen Gateway.

	In diesem Beispiel ist ConnectedEntityId die lokale Gateway-ID, die sich beim Ausführen finden können`Get-AzureLocalNetworkGateway`. Die VirtualNetworkGatewayId können Sie unter Verwendung des `Get-AzureVirtualNetworkGateway` Cmdlet finden. Nach diesem Schritt wird die Verbindung zwischen Ihrem lokalen Netzwerk und Azure über die Standort-zu-Standort-VPN-Verbindung hergestellt.


	`New-AzureVirtualNetworkGatewayConnection -connectedEntityId <local-network-gateway-id> -gatewayConnectionName Azure2Local -gatewayConnectionType IPsec -sharedKey abc123 -virtualNetworkGatewayId <azure-s2s-vpn-gateway-id>`


## Konfigurieren Sie Ihre vorhandene VNet für Koexistenz-Verbindungen

Wenn Sie ein vorhandenes virtuelles Netzwerk über eine ExpressRoute- oder Standort-zu-Standort-VPN-Verbindung verbunden haben, um eine Verbindung beider Verbindungen mit dem vorhandenen Virtual Network zu ermöglichen, müssen Sie das vorhandene Gateway zunächst löschen. Dies bedeutet, dass lokal Ihre Verbindung mit Ihrem Virtual Network über das Gateway verloren geht, während Sie an dieser Konfiguration arbeiten.

**Bevor Sie mit der Konfiguration beginnen:** Überprüfen Sie, dass Sie in Ihrem Virtual Network noch ausreichend IP-Adressen vorhanden sind, sodass Sie die Größe des Gateway-Subnetzes erhöhen können.


1. Stellen Sie sicher, dass Sie über die neueste Version von PowerShell-Cmdlets verfügen. Sie können die neuesten PowerShell-Cmdlets aus dem PowerShell-Abschnitt der [Downloadseite](http://azure.microsoft.com/downloads/) herunterladen und installieren.
 
2. Löschen Sie das vorhandene Standort-zu-Standort-VPN-Gateway. Nutzen Sie das folgende Cmdlet, um die Werte mit Ihren eigenen auszutauschen.

	`Remove-AzureVNetGateway –VnetName MyAzureVNET`

2. Exportieren Sie das Virtual Network-Schema. Nutzen Sie das folgende PowerShell-Cmdlet, um die Werte mit Ihren eigenen auszutauschen.

	`Get-AzureVNetConfig –ExportToFile “C:\NetworkConfig.xml”`
3. Bearbeiten Sie die Netzwerk-Konfigurationsdateischema, sodass das Gateway-Subnetz /27 ist (oder ein kürzeres Präfix). Betrachten Sie das folgende Beispiel. Weitere Informationen zum Arbeiten mit der Netzwerk-Konfigurationsdatei finden Sie unter [Konfigurieren eines Virtual Network mithilfe einer Netzwerk-Konfigurationsdatei](../virtual-network/virtual-networks-using-network-configuration-file.md). Weitere Informationen zum Konfigurationsschema finden Sie unter [Konfigurationsschema für Azure Virtual Network](https://msdn.microsoft.com/library/azure/jj157100.aspx).


          <Subnet name="GatewaySubnet">
            <AddressPrefix>10.17.159.224/27</AddressPrefix>
          </Subnet>
4. Wenn Ihr vorheriges Gateway ein Standort-zu-Standort-VPN war, müssen Sie auch den Verbindungstyp auf **dediziert** ändern.

		         <Gateway>
		          <ConnectionsToLocalNetwork>
		            <LocalNetworkSiteRef name="MyLocalNetwork">
		              <Connection type="Dedicated" />
		            </LocalNetworkSiteRef>
		          </ConnectionsToLocalNetwork>
		        </Gateway>
5. An diesem Punkt verfügen Sie über ein VNet ohne Gateways. Sie können mit **Schritt 3** im Artikel [Erstellen eines neuen VNet mit gleichzeitig bestehenden Verbindungen](#create-a-new-vnet-with-coexisting-connections) fortfahren, um neue Gateways zu erstellen und Ihre Verbindungen zu vervollständigen.



## Nächste Schritte
	
Erfahren Sie mehr zu ExpressRoute. Siehe [ExpressRoute-Übersicht](expressroute-introduction.md).

Erfahren Sie mehr zu VPN-Gateways. Siehe [Zu VPN-Gateways](../vpn-gateway/vpn-gateway-about-vpngateways.md).

<!---HONumber=July15_HO2-->