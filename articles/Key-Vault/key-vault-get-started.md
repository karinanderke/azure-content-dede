<properties 
	pageTitle="Erste Schritte mit dem Azure-Schlüsseltresor | Übersicht" 
	description="Verwenden Sie dieses Lernprogramm für den Einstieg in den Azure-Schlüsseltresor, um einen geschützten Container in Azure zu erstellen, in dem Sie kryptografischen Schlüssel und geheime Schlüssel in Azure speichern und verwalten." 
	services="key-vault" 
	documentationCenter="" 
	authors="cabailey" 
	manager="mbaldwin"/>

<tags 
	ms.service="key-vault" 
	ms.workload="identity" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="05/04/2015" 
	ms.author="cabailey"/>

# Erste Schritte mit dem Azure-Schlüsseltresor #

## Einführung  
Verwenden Sie dieses Lernprogramm für den Einstieg in den Azure-Schlüsseltresor – derzeit in der Vorschau –, um einen geschützten Container (einen Tresor) in Azure zu erstellen, in dem Sie kryptografischen Schlüssel und geheime Schlüssel in Azure speichern und verwalten. Sie werden durch den Vorgang der Verwendung von Windows PowerShell zum Erstellen eines Tresors geleitet, der einen Schlüssel oder ein Kennwort enthält, den/das Sie anschließend mit einer Azure-Anwendung verwenden können.  Anschließend wird gezeigt, wie eine Anwendung diesen Schlüssel bzw. das Kennwort verwenden kann.

**Geschätzter Zeitaufwand:** 20 Minuten

>[AZURE.NOTE]Dieses Lernprogramm enthält keine Anweisungen zum Schreiben der Azure-Anwendung, die in einem der Schritte verwendet wird, um zu zeigen, wie Sie eine Anwendung zum Verwenden eines Schlüssels oder geheimen Schlüssels im Schlüsseltresor autorisieren.
>
>Während des Vorschauzeitraums können Sie den Azure-Schlüsseltresor nicht im Azure-Portal konfigurieren. Sie müssen stattdessen die Anweisungen für Azure PowerShell verwenden.

Eine Übersicht über den Azure-Schlüsseltresor finden Sie unter [Was ist der Azure-Schlüsseltresor?](key-vault-whatis.md)

## Voraussetzungen 

Für dieses Lernprogramm benötigen Sie Folgendes:

- Ein Abonnement für Microsoft Azure. Wenn Sie kein Abonnement haben, können Sie sich für eine [kostenlose Testversion](../../../pricing/free-trial) registrieren.
- Azure PowerShell, Version 0.9.1 oder höher. Um die neueste Version zu installieren und sie Ihrem Azure-Abonnement zuzuordnen, lesen Sie [Installieren und Konfigurieren von Azure PowerShell](../powershell-install-configure.md).
- Eine Anwendung, die zur Verwendung des Schlüssels oder Kennworts konfiguriert wird, den bzw. das Sie in diesem Lernprogramm erstellen. Eine Beispielanwendung erhalten Sie im [Microsoft Download Center](http://www.microsoft.com/de-de/download/details.aspx?id=45343). Anweisungen finden Sie in der zugehörigen Readme-Datei.


Dieses Lernprogramm richtet sich an Windows PowerShell-Anfänger. Es wird aber vorausgesetzt, dass Sie die grundlegenden Konzepte, wie zum Beispiel Module, Cmdlets und Sitzungen, verstehen. Weitere Informationen zu Windows PowerShell finden Sie unter [Erste Schritte mit Windows PowerShell](https://technet.microsoft.com/library/hh857337.aspx).

Um detaillierte Hilfe zu einem Cmdlet aus dem Lernprogramm zu erhalten, verwenden Sie das Get-Help-Cmdlet.

	Get-Help <cmdlet-name> -Detailed

Geben Sie beispielsweise Folgendes ein, um Hilfe zum Add-AzureAccount-Cmdlet zu erhalten:

	Get-Help Add-AzureAccount -Detailed

Lesen Sie bitte auch die folgenden Lernprogramme, um sich mit dem Azure-Ressourcen-Manager in Windows PowerShell vertraut zu machen:

- [Installieren und Konfigurieren von Azure PowerShell](../powershell-install-configure.md)
- [Verwenden von Windows PowerShell mit dem Ressourcen-Manager](../powershell-azure-resource-manager.md)


## <a id="connect"></a>Verbindungsherstellung mit Ihren Abonnements ##

Starten Sie eine Azure PowerShell-Sitzung, und melden Sie sich mit dem folgenden Befehl bei Ihrem Azure-Konto an:

    Add-AzureAccount

Geben Sie im Popup-Browserfenster den Benutzernamen und das Kennwort Ihres Azure-Kontos ein. Windows PowerShell ruft alle Abonnements ab, die diesem Konto zugeordnet sind und verwendet per Voreinstellung das erste Abonnement.

Wenn Sie über mehrere Abonnements verfügen und zur Verwendung für den Azure-Schlüsseltresor ein spezifisches Abonnement verwenden möchten, geben Sie den folgenden Befehl ein, um die Abonnements für Ihr Konto anzuzeigen:

    Get-AzureSubscription

Geben Sie Folgendes ein, um das zu verwendende Abonnement anzugeben:

    Select-AzureSubscription -SubscriptionName <subscription name>

Weitere Informationen zum Konfigurieren von Azure PowerShell finden Sie unter [Installieren und Konfigurieren von Azure PowerShell](../powershell-install-configure.md).

## <a id="switch"></a>Wechseln zum Azure-Ressourcen-Manager ##

Die Schlüsseltresor-Cmdlets erfordern den Azure-Ressourcen-Manager, geben Sie deshalb den folgenden Befehl ein, um in den Azure-Ressourcen-Manager-Modus zu wechseln: 

	Switch-AzureMode AzureResourceManager

## <a id="resource"></a>Erstellen einer neuen Ressourcengruppe ##

Wenn Sie den Azure-Ressourcen-Manager verwenden, werden alle zugehörigen Ressourcen in einer Ressourcengruppe erstellt. Im Rahmen dieses Lernprogramms erstellen wir eine neue Ressourcengruppe mit dem Namen "ContosoResourceGroup":

	New-AzureResourceGroup –Name 'ContosoResourceGroup' –Location 'East Asia'

Verwenden Sie für den Parameter "-Location" den Befehl [Get-AzureLocation](https://msdn.microsoft.com/library/azure/dn654582.aspx), um zu ermitteln, wie Sie einen anderen Speicherort als den in diesem Beispiel verwendeten angeben können.  Wenn Sie weitere Informationen benötigen, geben Sie Folgendes ein: `Get-Help Get-AzureLocation`


## <a id="vault"></a>Erstellen eines Schlüsseltresors ##

Verwenden Sie das Cmdlet [New-AzureKeyVault](https://msdn.microsoft.com/library/azure/dn903602.aspx), um einen Schlüsseltresor zu erstellen. Dieses Cmdlet verfügt über drei erforderliche Parameter: einen für den **Ressourcengruppennamen**, einen für den **Schlüsseltresornamen** und einen für den **geografischen Standort**.

Wenn Sie beispielsweise den Tresornamen **ContosoKeyVault**, den Ressourcengruppennamen **ContosoResourceGroup** und den Speicherort **East Asia** verwenden möchten, geben Sie Folgendes ein:

    New-AzureKeyVault -VaultName 'ContosoKeyVault' -ResourceGroupName 'ContosoResourceGroup' -Location 'East Asia' 

Die Ausgabe dieses Cmdlets zeigt die Eigenschaften des Schlüsseltresors, den Sie soeben erstellt haben. Die zwei wichtigsten Eigenschaften sind diese:

- **Name**: Im Beispiel ist dies "ContosoKeyVault". Sie verwenden diesen Namen für andere Schlüsseltresor-Cmdlets.
- **vaultUri**: In diesem Beispiel ist dies https://contosokeyvault.vault.azure.net/. Anwendungen, die Ihren Tresor über die zugehörige REST-API nutzen, müssen diesen URI verwenden.

Ihr Azure-Konto ist jetzt autorisiert, Vorgänge in diesem Schlüsseltresor durchzuführen. Bisher sind Sie der einzige Benutzer mit dieser Berechtigung.

## <a id="add"></a>Hinzufügen eines Schlüssels oder geheimen Schlüssels zum Schlüsseltresor ##

Wenn Sie mit dem Azure-Schlüsseltresor einen softwaregeschützten Schlüssel erstellen möchten, verwenden Sie hierzu das Cmdlet [Add-AzureKeyVaultKey](https://msdn.microsoft.com/library/azure/dn868048.aspx). Geben Sie Folgendes ein:

    $key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVault' -Name 'ContosoFirstKey' -Destination 'Software'


Wenn Sie über einen vorhandenen softwaregeschützten Schlüssel in einer PFX-Datei auf Ihrem Laufwerk "C:" in einer Datei namens "softkey.pfx" verfügen, die Sie in den Azure-Schlüsseltresor hochladen möchten, geben Sie den folgenden Befehl ein. Mit diesem Befehl wird die Variable **securepfxpwd** gesetzt, um das Kennwort **123** für die PFX-Datei festzulegen:

    $securepfxpwd = ConvertTo-SecureString –String '123' –AsPlainText –Force

Geben Sie anschließend den folgenden Code ein, um den Schlüssel aus der PFX-Datei zu importieren, die den Schlüssel mithilfe von Software im Schlüsseltresordienst schützt:

    $key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVault' -Name 'ContosoFirstKey' -KeyFilePath 'c:\softkey.pfx' -KeyFilePassword $securepfxpwd


Jetzt können Sie mit dem zugehörigen URI auf den erstellten oder in den Azure-Schlüsseltresor hochgeladenen Schlüssel verweisen. Beispiel: **https://ContosoKeyVault.vault.azure.net/Keys/ContosoFirstKey/a10f5336-9d93-44a3-9e26-e86e3488b768** 

Geben Sie zur Anzeige des URI für diesen Schlüssel Folgendes ein:

	$Key.key.kid

Um einen geheimen Schlüssel zum Schlüsseltresor hinzuzufügen – in diesem Fall das Kennwort "SQLPassword" mit dem Wert "Pa$$w0rd" für den Azure-Schlüsseltresor – konvertieren Sie zunächst den Wert "Pa$$w0rd" in eine sichere Zeichenfolge, indem Sie Folgendes eingeben:

	$secretvalue = ConvertTo-SecureString 'Pa$$w0rd' -AsPlainText -Force

Geben Sie anschließend Folgendes ein:

	$secret = Set-AzureKeyVaultSecret -VaultName 'ContosoKeyVault' -Name 'SQLPassword' -SecretValue $secretvalue

Jetzt können Sie mit dem zugehörigen URI auf das Kennwort verweisen, das Sie dem Azure-Schlüsseltresor hinzugefügt haben. Beispiel: **https://ContosoVault.vault.azure.net/Secrets/778c3e43-3fdb-4cdf-b58e-7f501eb41d68** 

Geben Sie zur Anzeige des URI für diesen geheimen Schlüssel Folgendes ein:

	$secret.Id

Zeigen wir den Schlüssel oder geheimen Schlüssel an, den Sie soeben erstellt haben:

- Geben Sie Folgendes ein, um Ihren Schlüssel anzuzeigen: `Get-AzureKeyVaultKey –VaultName 'ContosoKeyVault'`
- Geben Sie Folgendes ein, um Ihren geheimen Schlüssel anzuzeigen: `Get-AzureKeyVaultSecret –VaultName 'ContosoKeyVault'`

Jetzt sind Schlüsseltresor und Schlüssel oder geheimer Schlüssel bereit für die Verwendung durch die Anwendung. Sie müssen die Anwendungen zur Verwendung der Schlüssel bzw. geheimen Schlüssel autorisieren.

## <a id="register"></a>Registrieren einer Anwendung mit Azure Active Directory ##

Dieser Schritt wird üblicherweise durch einen Entwickler auf einem separaten Computer durchgeführt. Dieser Schritt ist nicht spezifisch für den Azure-Schlüsseltresor, wird der Vollständigkeit halber jedoch hier aufgeführt.


>[AZURE.IMPORTANT]Zum Abschließen dieses Lernprogramms müssen sich Ihr Konto, der Schlüsseltresor und die in diesem Schritt registrierte Anwendung im selben Azure-Verzeichnis befinden.

Anwendungen, die einen Schlüsseltresor verwenden, müssen sich mithilfe eines Azure Active Directory-Tokens authentifizieren. Hierzu muss der Besitzer der Anwendung die Anwendung zunächst in Azure Active Directory registrieren. Zum Abschluss der Registrierung erhält der Anwendungsbesitzer die folgenden Werte:


- Eine **Anwendungs-ID** (auch Client-ID genannt) und einen **Authentifizierungsschlüssel** (auch als gemeinsamer geheimer Schlüssel bezeichnet). Die Anwendung muss beide dieser Werte in Azure Active Directory vorlegen, um ein Token zu erhalten.  Wie die Anwendung konfiguriert wird, um dies zu erreichen, richtet sich nach der Anwendung. Bei der Beispielanwendung für den Schlüsseltresor legt der Anwendungsbesitzer diese Werte in der Datei "app.config" fest.



So registrieren Sie die Anwendungen in Azure Active Directory

1. Melden Sie sich beim Azure-Portal an.
2. Klicken Sie auf der linken Seite auf **Active Directory**, und wählen Sie das Verzeichnis, in dem Sie Ihre Anwendung registrieren möchten. <br> <br> Hinweise: Sie müssen dasselbe Verzeichnis auswählen, das auch das Azure-Abonnement enthält, mit dem Sie Ihren Schlüsseltresor erstellt haben. Wenn Sie nicht wissen, welches Verzeichnis dies ist, klicken Sie auf **Einstellungen**, ermitteln Sie das Abonnement, mit dem Sie Ihren Schlüsseltresor erstellt haben, und notieren Sie sich den Namen des Verzeichnisses, der in der letzten Spalte angezeigt wird.

3. Klicken Sie auf **ANWENDUNGEN**. Wenn Ihrem Verzeichnis keine Apps hinzugefügt wurden, wird auf dieser Seite der Link **App hinzufügen** angezeigt. Klicken Sie auf den Link, oder klicken Sie alternativ auf der Befehlsleiste auf **HINZUFÜGEN**.
4.	Klicken Sie im Assistenten **ANWENDUNG HINZUFÜGEN** auf der Seite **Was möchten Sie tun?** auf **Eine von meinem Unternehmen entwickelte Anwendung hinzufügen**.
5.	Geben Sie auf der Seite **Erzählen Sie uns von Ihrer App** einen Namen für Ihre Anwendung ein, und wählen Sie **WEBANWENDUNG UND/ODER WEB-API** (die Standardeinstellung). Klicken Sie auf das Symbol "Weiter".
6.	Geben Sie auf der Seite **App-Eigenschaften** die **ANMELDE-URL** und den **APP-ID-URI** für Ihre Webanwendung an. Wenn Ihre Anwendung über keine solchen Werte verfügt, können Sie in diesem Schritt Pseudowerte verwenden (Sie können beispielsweise in beide Felder den Wert http://test1.contoso.com eingeben). Es spielt keine Rolle, ob diese Websites vorhanden sind. Es ist nur wichtig, dass der App-ID-URI für jede Anwendung in Ihrem Verzeichnis eindeutig ist. Das Verzeichnis verwendet diese Zeichenfolge zur Identifizierung Ihrer App.
7.	Klicken Sie auf das Symbol "Abschließen", um Ihre Änderungen im Assistenten zu speichern.
8.	Klicken Sie auf der Seite "Schnellstart" auf **KONFIGURIEREN**. 
9.	Führen Sie einen Bildlauf zum Abschnitt **Schlüssel** durch, wählen Sie die Dauer aus, und klicken Sie dann auf **SPEICHERN**. Die Seite wird aktualisiert und zeigt jetzt einen Schlüsselwert. Sie müssen Ihre Anwendung mit diesem Schlüsselwert und der **CLIENT-ID** konfigurieren. (Die Anweisungen für diese Konfigurationen sind anwendungsspezifisch.)
10.	Kopieren Sie den Client-ID-Wert von dieser Seite. Sie verwenden ihn im nächsten Schritt, um Berechtigungen für Ihren Tresor festzulegen.




## <a id="authorize"></a>Autorisieren der Anwendung zum Verwenden des Schlüssels oder geheimen Schlüssels ##


Verwenden Sie das Cmdlet [Set-AzureKeyVaultAccessPolicy](https://msdn.microsoft.com/library/azure/dn903607.aspx), um die Anwendung zum Zugreifen auf den Schlüssel oder geheimen Schlüssel im Tresor zu autorisieren.

Wenn Ihr Tresorname beispielsweise "ContosoKeyVault" lautet, die Anwendung, die Sie autorisieren möchten, über die Client-ID 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed verfügt, und Sie die Anwendung zum Entschlüsseln und Anmelden mit Schlüsseln in Ihrem Tresor autorisieren möchten, führen Sie Folgendes aus:

	Set-AzureKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -ServicePrincipalName 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed -PermissionsToKeys decrypt,sign



## <a id="HSM"></a>Verwenden eines Hardwaresicherheitsmoduls (HSM) ##

Zur Steigerung der Sicherheit können Sie Schlüssel in HSMs importieren oder in diesen generieren. Diese Schlüssel verbleiben immer innerhalb der HSM-Grenzen. HSMs sind gemäß FIPS 140-2 Level 2 zertifiziert. Wenn diese Anforderung auf Sie nicht zutrifft, überspringen Sie diesen Abschnitt und wechseln Sie zu [Löschen des Schlüsseltresors und zugeordneter Schlüssel und geheimer Schlüssel](#delete).

Um diese HSM-geschützten Schlüssel zu erstellen, müssen Sie über ein [Tresorabonnement mit Unterstützung von HSM-geschützten Schlüsseln](../../../pricing/free-trial) verfügen.


Wenn Sie den Tresor erstellen, fügen Sie den Parameter "SKU" hinzu.

	New-AzureKeyVault -VaultName 'ContosoKeyVaultHSM' -ResourceGroupName 'ContosoResourceGroup' -Location 'East Asia' -SKU 'Premium'

Sie können diesem Tresor softwaregeschützte Schlüssel (wie weiter oben gezeigt) und HSM-geschützte Schlüssel hinzufügen. Legen Sie den Parameter "Destination" auf "HSM" fest, um einen HSM-geschützten Schlüssel zu erstellen:

	$key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVaultHSM' -Name 'ContosoFirstHSMKey' -Destination 'HSM'

Sie können mit dem folgenden Befehl einen Schlüssel aus einer PFX-Datei auf Ihrem Computer importieren. Dieser Befehl importiert den Schlüssel in HSMs im Schlüsseltresordienst:

	$key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVaultHSM' -Name 'ContosoFirstHSMKey' -KeyFilePath 'c:\softkey.pfx' -KeyFilePassword $securepfxpwd -Destination 'HSM'

Der nächste Befehl importiert ein BYOK-Paket (Bring Your Own Key). Auf diese Weise können Sie Ihren Schlüssel im lokalen HSM generieren und ihn in HSMs im Schlüsseltresordienst übertragen, ohne dass der Schlüssel die HSM-Grenzen verlässt: 

	$key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVaultHSM' -Name 'ContosoFirstHSMKey' -KeyFilePath 'c:\ITByok.byok' -Destination 'HSM'

Ausführlichere Informationen zum Generieren dieses BYOK-Pakets finden Sie unter [Verwenden von HSM-geschützten Schlüsseln mit dem Azure-Schlüsseltresor](https://msdn.microsoft.com/library/azure/dn903624.aspx).

## <a id="delete"></a>Löschen des Schlüsseltresors und der zugeordneten Schlüssel und geheimen Schlüssel ##

Wenn Sie den Schlüsseltresor und die darin enthaltenen Schlüssel und geheimen Schlüssel nicht länger benötigen, können Sie den Schlüsseltresor mit dem Cmdlet [Remove-AzureKeyVault](https://msdn.microsoft.com/library/azure/dn903603.aspx) löschen:

	Remove-AzureKeyVault -VaultName 'ContosoKeyVault'

Alternativ können Sie eine gesamte Azure-Ressourcengruppe löschen, die den Schlüsseltresor und alle weiteren Ressourcen enthält, die Sie in diese Gruppe eingeschlossen haben:

	Remove-AzureResourceGroup -ResourceGroupName 'ContosoResourceGroup'


## <a id="other"></a>Weitere Azure PowerShell-Cmdlets ##

Die folgenden weiteren Befehle sind möglicherweise ebenfalls für das Verwalten eines Azure-Schlüsseltresors von Nutzen:

- `$Keys = Get-AzureKeyVaultKey -VaultName 'ContosoKeyVault'`: Dieser Befehl ruft eine tabellarische Anzeige aller Schlüssel und ausgewählten Eigenschaften ab.
- `$Keys[0]`: Dieser Befehl zeigt eine vollständige Liste der Eigenschaften für den angegebenen Schlüssel an.
- `Get-AzureKeyVaultSecret`: Dieser Befehl ruft eine tabellarische Anzeige der Namen aller geheimen Schlüssel und ausgewählten Eigenschaften ab.
- `Remove-AzureKeyVaultKey -VaultName 'ContosoKeyVault' -Name 'ContosoFirstKey'`: Beispiel dazu, wie ein bestimmter Schlüssel entfernt wird.
- `Remove-AzureKeyVaultSecret -VaultName 'ContosoKeyVault' -Name 'SQLPassword'`: Beispiel dazu, wie ein bestimmter geheimer Schlüssel entfernt wird.






## <a id="next"></a>Nächste Schritte ##

Eine Liste der Windows PowerShell-Cmdlets für den Azure-Schlüsseltresor finden Sie unter [Cmdlets für den Azure-Schlüsseltresor](https://msdn.microsoft.com/library/azure/dn868052.aspx).

Eine Referenz zur Programmierung finden Sie unter [Azure-Schlüsseltresor – REST-API-Referenz](https://msdn.microsoft.com/library/azure/dn903609.aspx) und [Azure-Schlüsseltresor – C# Client-API-Referenz](https://msdn.microsoft.com/library/azure/dn903628.aspx).



<!--HONumber=54--> 