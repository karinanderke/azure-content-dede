<properties 
	pageTitle="Erstellen einer Hybridsammlung für RemoteApp" 
	description="Erfahren Sie, wie Sie eine Bereitstellung von RemoteApp erstellen, die eine Verbindung mit Ihrem internen Netzwerk herstellt." 
	services="remoteapp" 
	documentationCenter="" 
	authors="lizap" 
	manager="mbaldwin" 
	editor=""/>

<tags 
	ms.service="remoteapp" 
	ms.workload="compute" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/28/2015" 
	ms.author="elizapo"/>

# Erstellen einer Hybridsammlung für RemoteApp

Es gibt zwei Arten von RemoteApp-Sammlungen:

- Cloud: befindet sich vollständig in Azure und wird mithilfe der Option **Schnellerfassung** im Azure-Verwaltungsportal erstellt.  
- Hybrid: enthält ein virtuelles Netzwerk für den lokalen Zugriff und wird mithilfe der Option **Create with VNET** im Verwaltungsportal erstellt.

Dieses Lernprogramm führt Sie durch den Prozess für das Erstellen einer Hybrid-Sammlung. Es besteht aus sieben Schritten:

1.	Erstellen Sie ein [benutzerdefiniertes Vorlagenimage für RemoteApp](remoteapp-imageoptions.md), oder wählen Sie eines der in Ihrem Abonnement enthaltenen Images aus.
2. Richten Sie Ihr virtuelles Netzwerk ein.
2.	Erstellen einer RemoteApp-Sammlung.
2.	Verknüpfen Sie die Sammlung mit Ihrem virtuellen Netzwerk.
3.	Fügen Sie Ihrer Sammlung ein Vorlagenimage hinzu.
4.	Konfigurieren der Verzeichnissynchronisierung Für RemoteApp müssen Sie eine Integration in Azure Active Directory durchführen, indem Sie entweder 1) Azure Active Directory Sync mit der Kennwortsynchronisierungsoption konfigurieren oder 2) Azure Active Directory Sync ohne die Kennwortsynchronisierungsoption, aber unter Verwendung einer Domäne konfigurieren, die mit AD FS verbunden ist. Beachten Sie die [Konfigurationsinformationen für Active Directory mit RemoteApp](remoteapp-ad.md).
5.	Veröffentlichen von RemoteApp-Apps.
6.	Konfigurieren des Benutzerzugriffs.

**Voraussetzungen**

Bevor Sie mit der Erstellung der Sammlung beginnen, benötigen Sie Folgendes:

- [Melden](http://azure.microsoft.com/services/remoteapp/) Sie sich für RemoteApp an. 
- Erstellen Sie ein Benutzerkonto in Active Directory, das als Konto für den RemoteApp-Dienst dient. Beschränken Sie die Berechtigungen für dieses Konto, sodass es nur Computer in die Domäne einbinden kann.
- Sammeln Sie Informationen zu Ihrem lokalen Netzwerk: IP-Adressdaten und Details zum VPN-Gerät.
- Installieren Sie das [Azure PowerShell](../install-configure-powershell.md)-Modul.
- Sammeln Sie Informationen zu den Benutzern, denen Sie Zugriff gewähren möchten. Dies können Informationen zu Microsoft-Konten oder Active Directory-Geschäftskonten sein.
- Erstellen Sie Ihr Vorlagenimage. Ein RemoteApp-Vorlagenimage enthält die Apps und Programme, die Sie für Ihre Benutzer veröffentlichen möchten. Weitere Informationen finden Sie unter [Erstellen eines RemoteApp-Images](remoteapp-imageoptions.md). 
- [Konfigurieren von Active Directory für RemoteApp](remoteapp-ad.md)



## Schritt 1: Einrichten des virtuellen Netzwerks
Sie können eine RemoteApp-Hybrid-Sammlung für ein vorhandenes virtuelles Azure-Netzwerk bereitstellen oder ein neues virtuelles Netzwerk erstellen. Ein virtuelles Netzwerk ermöglicht es den Benutzern, auf Daten auf Ihrem lokalen Netzwerk über RemoteApp-Remoteressourcen zuzugreifen. Mit einem virtuellen Azure-Netzwerk verfügt Ihre Sammlung über direkten Zugriff auf andere Azure-Dienste und virtuelle Computer in diesem virtuellen Netzwerk.

### Erstellen eines Azure-VNET und Verknüpfen mit der Active Directory-Bereitstellung

Erstellen Sie zunächst ein [virtuelles Netzwerk](https://msdn.microsoft.com/library/azure/dn631643.aspx). Dies erfolgt auf der Registerkarte **Netzwerk** im Azure-Verwaltungsportal. Sie müssen Ihr virtuelles Netzwerk mit der Active Directory-Bereitstellung verbinden, die mit Ihrem Azure Active Directory-Mandanten synchronisiert ist.

Weitere Informationen finden Sie unter [Informationen zu virtuellen Netzwerkeinstellungen im Verwaltungsportal](https://msdn.microsoft.com/library/azure/jj156074.aspx).

### Sicherstellen, dass das virtuelle Netzwerk für RemoteApp bereit ist
Bevor Sie Ihre RemoteApp-Sammlung erstellen, müssen Sie sicherstellen, dass Ihr neues virtuelles Netzwerk bereit ist. Sie können dies wie folgt überprüfen:

1. Erstellen Sie einen virtuellen Azure-Computer in dem Subnetz des virtuellen Netzwerks, das Sie gerade RemoteApp erstellt haben.
2. Stellen Sie über Remotedesktop eine Verbindung zum virtuellen Computer her. (Klicken Sie auf **Verbinden**.)
3. Verknüpfen Sie ihn mit der gleichen Active Directory-Bereitstellung, die auch für RemoteApp verwendet werden soll.

Hat dies funktioniert? Ihr virtuelles Netzwerk und das Subnetz sind für RemoteApp bereit!

[Hier](https://msdn.microsoft.com/library/azure/jj156003.aspx) finden Sie weitere Informationen zum Erstellen von virtuellen Azure-Computern und zum Verbinden mit Remotedesktop.

## Schritt 2: Erstellen einer RemoteApp-Sammlung ##



1. Gehen Sie im [Azure-Verwaltungsportal](http://manage.windowsazure.com) zur Seite "RemoteApp".
2. Klicken Sie auf **Neu > Create with VNET**.
3. Geben Sie einen Namen für die Sammlung ein.
4. Wählen Sie den Plan aus, den Sie verwenden möchten: "Standard" oder "Einfach".
5. Klicken Sie auf **Create RemoteApp collection**.

Gehen Sie nach dem Erstellen der RemoteApp-Sammlung zur RemoteApp-Seite **Schnellstart**, um mit der Einrichtung fortzufahren.

## Schritt 3: Verknüpfen der Sammlung mit dem virtuellen Netzwerk ##

 
1. Klicken Sie auf der Seite **Schnellstart** auf **link a virtual network**.
2. Wählen Sie das zu verwendende virtuelle Netzwerk aus der Dropdownliste aus.
3. Wählen Sie die entsprechende Region aus, und stellen Sie sicher, dass das richtige Abonnement im Feld angezeigt wird. 
5. Klicken Sie auf der wieder geöffneten Seite **Schnellstart** auf **join local domain**. Fügen Sie das Konto des RemoteApp-Dienstes zur lokalen Active Directory-Domäne hinzu. Dazu benötigen Sie den Domänennamen, die Organisationseinheit, den Benutzernamen des Kontos und das Kennwort. 

	Dies sind die Informationen, die Sie im Rahmen der Schritte unter [Konfigurieren von Active Directory für Azure RemoteApp](remoteapp-ad.md) erfasst haben.


## Schritt 4: Verbinden mit einem RemoteApp-Image ##

Ein RemoteApp-Vorlagenimage enthält die Programme, die Sie an die Benutzer freigeben möchten. Sie können entweder ein neues [Vorlagenimage](remoteapp-imageoptions.md) erstellen oder eine Verbindung zu einem vorhandenen Image (das bereits in Azure hochgeladen oder importiert wurde) herstellen. Sie können außerdem eine Verbindung mit einem der RemoteApp-[Vorlagenimages](remoteapp-images.md) herstellen, die Office 365 oder Office 2013 (Testversion) enthalten.

Wenn Sie ein neues Abbild hochladen, müssen Sie den Namen des Abbilds eingeben und den Speicherort wählen. Auf der nächsten Seite des Assistenten wird Ihnen eine Gruppe von PowerShell-Cmdlets angezeigt. Kopieren Sie diese Cmdlets und führen Sie sie von einer Windows PowerShell-Eingabeaufforderung mit erhöhten Rechten aus, um das Abbild hochzuladen.

Wenn Sie eine Verbindung zu einem vorhandenen Abbild herstellen, geben Sie einfach den Namen und den Speicherort des Abbilds sowie das dazugehörige Azure-Abonnement an.



## Schritt 5: Konfigurieren der Active Directory-Verzeichnissynchronisierung ##

Für RemoteApp müssen Sie eine Integration in Azure Active Directory durchführen, indem Sie entweder 1) Azure Active Directory Sync mit der Kennwortsynchronisierungsoption konfigurieren oder 2) Azure Active Directory Sync ohne die Kennwortsynchronisierungsoption, aber unter Verwendung einer Domäne konfigurieren, die mit AD FS verbunden ist. Informationen zur Planung und eine detaillierte Anleitung finden Sie unter [Fahrplan zur Verzeichnissynchronisierung](http://msdn.microsoft.com//library/azure/hh967642.aspx).

## Schritt 6: Veröffentlichen von RemoteApp-Programmen ##

Eine RemoteApp-App ist die App oder das Programm, das Sie den Benutzern zur Verfügung stellen. Es befindet sich im Vorlagenimage, das Sie für die Sammlung hochgeladen haben. Wenn ein Benutzer auf das RemoteApp-Programm zugreift, scheint dieses Programm in der lokalen Umgebung des Benutzers zu laufen, tatsächlich wird es aber in Azure ausgeführt.

Bevor ein Benutzer aber auf das RemoteApp-Programm zugreifen kann, muss es im Endbenutzer-Feed – einer Liste verfügbarer Programme, auf die der Benutzer über den Remotedesktop-Client zugreift – veröffentlicht werden.
 
Sie können in Ihrer RemoteApp-Sammlung mehrere Apps veröffentlichen. Klicken Sie auf der RemoteApp-Veröffentlichungsseite auf **Veröffentlichen**. Sie können die App entweder aus dem Startmenü des Vorlagenimages veröffentlichen oder den Pfad zur App im Vorlagenimage angeben. Wenn Sie das Programm vom Startmenü aus veröffentlichen möchten, wählen Sie das zu veröffentlichende Programm aus. Wenn Sie den Pfad angeben möchten, geben Sie den Namen der App sowie den Pfad an, in dem die App auf dem Vorlagenimage installiert ist.

## Schritt 7: Konfigurieren des Benutzerzugriffs ##

Nach dem Erstellen der RemoteApp-Sammlung müssen Sie nun die Benutzer hinzufügen, die Zugriff auf die Remoteressourcen haben sollen. Die Benutzer, denen Zugriff erteilt werden soll, müssen in dem Active Directory-Mandanten vorhanden sein, der dem Abonnement zugeordnet wurde, das Sie zum Erstellen dieser RemoteApp-Sammlung verwendet haben.

1.	Klicken Sie auf der Seite "Schnellstart" auf **Benutzerzugriff konfigurieren**. 
2.	Geben Sie das Geschäftskonto (aus Active Directory) oder das Microsoft-Konto ein, dem Sie Zugriff gewähren möchten.

	**Hinweise:**

	Verwenden Sie unbedingt das Format "Benutzer@Domäne.com".

	Wenn Sie Office 365 ProPlus in der Sammlung verwenden, müssen Sie für Ihre Benutzer die Active Directory-Identitäten verwenden. Damit wird die Lizenzierung überprüft.


3.	Klicken Sie nach der Überprüfung der Benutzer auf **Speichern**.


## Nächste Schritte ##
Das war alles: Sie haben die RemoteApp-Hybrid-Sammlung nun erfolgreich erstellt und bereitgestellt. Als Nächstes müssen die Benutzer den Remotedesktopclient herunterladen und installieren. Die URL für den Client finden Sie auf der RemoteApp-Schnellstartseite. Die Benutzer müssen sich nun beim Client anmelden und können dann auf die veröffentlichten Apps zugreifen.



<!--HONumber=54--> 