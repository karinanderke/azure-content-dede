<properties 
   pageTitle="Konfigurieren des Webproxys für Ihr Gerät"
   description="Hier erfahren Sie, wie Sie den Webproxy für Ihr StorSimple-Gerät mit Windows PowerShell für StorSimple konfigurieren."
   services="storsimple"
   documentationCenter=""
   authors="alkohli"
   manager="carolz"
   editor="tysonn" />
<tags 
   ms.service="storsimple"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="07/29/2015"
   ms.author="alkohli" />

# Konfigurieren des Webproxys für Ihr Gerät

## Übersicht
In diesem Tutorial erfahren Sie, wie Sie mithilfe von Windows PowerShell für StorSimple Webproxyeinstellungen für Ihr StorSimple-Gerät konfigurieren und anzeigen. Die Webproxyeinstellungen werden vom StorSimple-Gerät bei der Kommunikation mit der Cloud verwendet. Ein Webproxyserver erhöht die Sicherheit, filtert Inhalte, verringert dank Zwischenspeicherung die Bandbreitenanforderungen und ist sogar bei der Analyse hilfreich.

Bei dem Webproxy handelt es sich um eine optionale Konfiguration für Ihr StorSimple-Gerät. Der Webproxy kann nur über Windows PowerShell für StorSimple konfiguriert werden. Die Konfiguration ist ein zweistufiger Prozess:

1. Zuerst werden die Webproxyeinstellungen mithilfe des Setup-Assistenten oder über Cmdlets von Windows PowerShell für StorSimple konfiguriert.

1. Anschließend werden die konfigurierten Webproxyeinstellungen über Cmdlets von Windows PowerShell für StorSimple aktiviert.

Nach Abschluss der Webproxykonfiguration können Sie die konfigurierten Webproxyeinstellungen im StorSimple Manager-Dienst von Microsoft Azure sowie in Windows PowerShell für StorSimple anzeigen.

In diesem Tutorial lernen Sie Folgendes:

- Konfigurieren des Webproxys mithilfe von Setup-Assistent und Cmdlets
- Aktivieren des Webproxys mithilfe von Cmdlets
- Anzeigen der Webproxyeinstellungen im Verwaltungsportal
- Behandeln von Fehlern bei der Webproxykonfiguration


## Konfigurieren des Webproxys über Windows PowerShell für StorSimple

Die Webproxyeinstellungen können auf folgende Arten konfiguriert werden:

- Mit einem Setup-Assistenten, der Sie schrittweise durch die Konfiguration führt

- Mit Cmdlets in Windows PowerShell für StorSimple

Diese Methoden werden in den folgenden Abschnitten erläutert.


## Konfigurieren des Webproxys mithilfe des Setup-Assistenten

Der Setup-Assistent führt Sie durch die einzelnen Schritte für die Webproxykonfiguration. Führen Sie die folgenden Schritte aus, um den Webproxy für Ihr Gerät zu konfigurieren.

#### So konfigurieren Sie den Webproxy mithilfe des Setup-Assistenten

1. Wählen Sie im Menü der seriellen Konsole die erste Option (Anmeldung mit Vollzugriff) aus, und geben Sie das Geräteadministratorkennwort ein. Geben Sie den folgenden Befehl ein, um eine Sitzung des Setup-Assistenten zu starten:

    `Invoke-HcsSetupWizard`


1. Bei erstmaliger Verwendung des Setup-Assistenten für die Geräteregistrierung müssen Sie alle erforderlichen Netzwerkeinstellungen konfigurieren, um zur Webproxykonfiguration zu gelangen. Ist Ihr Gerät bereits registriert, können Sie die konfigurierten Netzwerkeinstellungen akzeptieren, bis Sie die Webproxykonfiguration erreichen. Geben Sie im Setup-Assistenten **Ja** ein, wenn Sie zum Konfigurieren der Webproxyeinstellungen aufgefordert werden.
2. Geben Sie als Webproxy-URL die IP-Adresse oder den vollqualifizierten Domänennamen (Fully Qualified Domain Name, FQDN) des Webproxyservers sowie die TCP-Portnummer ein, die das Gerät bei der Kommunikation mit der Cloud verwenden soll. Verwenden Sie das folgende Format:

	`http://<IP address or FQDN of the web proxy server>:<TCP port number>`

	Standardmäßig wird die TCP-Portnummer 8080 angegeben.

1. Legen Sie den Authentifizierungstyp auf **NTLM**, **Standard** oder **Keine** fest. „Standard“ bietet bei der Authentifizierung für die Proxyserverkonfiguration die geringste Sicherheit. „NTLM“ (NT-LAN-Manager) ist ein äußerst sicheres und komplexes Authentifizierungsprotokoll mit einem Drei-Wege-Messaging-System für die Benutzerauthentifizierung (ggf. auch ein Vier-Wege-System, wenn zusätzliche Integrität erforderlich ist). Standardmäßig wird die NTLM-Authentifizierung verwendet. Weitere Informationen finden Sie unter [Standard](http://hc.apache.org/httpclient-3.x/authentication.html) bzw. unter [NTLM-Authentifizierung](http://hc.apache.org/httpclient-3.x/authentication.html).

	> [AZURE.IMPORTANT]**Wenn in der Proxyserverkonfiguration für das Gerät die Standard- oder NTLM-Authentifizierung aktiviert ist, funktionieren die Geräteüberwachungsdiagramme im StorSimple Manager-Dienst nicht. Zur Verwendung der Überwachungsdiagramme muss die Authentifizierung auf „Keine“ festgelegt sein.**

1. Geben Sie bei Verwendung einer Authentifizierung einen Benutzernamen und ein Kennwort für den Webproxy an. Das Kennwort muss außerdem bestätigt werden.

	![Konfigurieren des Webproxys auf StorSimple-Gerät 1](./media/storsimple-configure-web-proxy/IC751830.png)

Wenn Sie Ihr Gerät zum ersten Mal registrieren, fahren Sie mit der Registrierung fort. Wenn das Gerät bereits registriert wurde, wird der Assistent beendet. Die konfigurierten Einstellungen werden gespeichert.

Der Webproxy wird außerdem aktiviert. Sie können [Aktivieren des Webproxys](#Enable-web-proxy) überspringen und die Webproxyeinstellungen direkt im Verwaltungsportal anzeigen.


## Konfigurieren des Webproxys über Cmdlets von Windows PowerShell für StorSimple

Alternativ können die Webproxyeinstellungen auch über die Cmdlets von Windows PowerShell für StorSimple konfiguriert werden. Führen Sie die folgenden Schritte aus, um den Webproxy zu konfigurieren:

#### So konfigurieren Sie den Webproxy mithilfe von Cmdlets

1. Wählen Sie im Menü der seriellen Konsole Option 1 aus, d. h. die** Anmeldung mit Vollzugriff**. Geben Sie das Geräteadministratorkennwort an, wenn Sie dazu aufgefordert werden. Das Standardkennwort lautet `Password1`.

1. Geben Sie an der Eingabeaufforderung Folgendes ein:

	`Set-HcsWebProxy -Authentication NTLM -ConnectionURI "<http://<IP address or FQDN of web proxy server>:<TCP port number>" -Username "<Username for web proxy server>"`

	Geben Sie das Kennwort an, wenn Sie dazu aufgefordert werden, und bestätigen Sie es:

	![Konfigurieren des Webproxys auf StorSimple-Gerät 3](./media/storsimple-configure-web-proxy/IC751831.png)

Der Webproxy ist nun konfiguriert und muss aktiviert werden.

## Aktivieren des Webproxys

Der Webproxy ist standardmäßig deaktiviert. Nachdem Sie die Webproxyeinstellungen auf dem StorSimple-Gerät konfiguriert haben, müssen Sie sie mithilfe von Windows PowerShell für StorSimple aktivieren.

> [AZURE.NOTE]**Dieser Schritt ist nicht erforderlich, wenn Sie den Webproxy mithilfe des Setup-Assistenten konfiguriert haben. Nach einer Sitzung des Setup-Assistenten wird der Webproxy automatisch aktiviert.**

Führen Sie in Windows PowerShell für StorSimple die folgenden Schritte aus, um den Webproxy auf Ihrem Gerät zu aktivieren:

#### So aktivieren Sie den Webproxy

1. Wählen Sie im Menü der seriellen Konsole Option 1 aus, d. h. die** Anmeldung mit Vollzugriff**. Geben Sie das Geräteadministratorkennwort an, wenn Sie dazu aufgefordert werden. Das Standardkennwort lautet `Password1`.

1. Geben Sie an der Eingabeaufforderung Folgendes ein:

	`Enable-HcsWebProxy`

	Die Webproxykonfiguration ist nun auf dem StorSimple-Gerät aktiviert.

	![Konfigurieren des Webproxys auf StorSimple-Gerät 4](./media/storsimple-configure-web-proxy/IC751832.png)

## Anzeigen der Webproxyeinstellungen im Verwaltungsportal
Die Webproxyeinstellungen werden über die Windows PowerShell-Schnittstelle konfiguriert und können nicht im Verwaltungsportal geändert werden. Sie können die konfigurierten Einstellungen allerdings im Verwaltungsportal anzeigen. Führen Sie die folgenden Schritte aus, um den Webproxy anzuzeigen:

#### So zeigen Sie die Webproxyeinstellungen an
1. Navigieren Sie zu **StorSimple Manager-Dienst > Geräte**. Klicken Sie auf ein Gerät, und navigieren Sie anschließend zu **Konfigurieren**.
1. Navigieren Sie auf der Konfigurationsseite zum Abschnitt mit den Webproxyeinstellungen. Die konfigurierten Webproxyeinstellungen für das StorSimple-Gerät werden wie folgt angezeigt:

	![Anzeigen des Webproxys im Verwaltungsportal](./media/storsimple-configure-web-proxy/ViewWebProxyPortal_M.png)
 
## Fehler bei der Webproxykonfiguration

Sind die Webproxyeinstellungen nicht korrekt konfiguriert, werden in Windows PowerShell für StorSimple entsprechende Fehlermeldungen angezeigt. In der folgenden Tabelle werden einige dieser Fehlermeldungen, mögliche Ursachen und empfohlene Aktionen erläutert:

|Seriennummer|HRESULT-Fehlercode|Mögliche Ursache|Empfohlene Maßnahme|
|---|---|---|---|
|1\.|0x80070001|Der Befehl wird über den passiven Controller ausgeführt, und es ist keine Kommunikation mit dem aktiven Controller möglich.|Führen Sie den Befehl auf dem aktiven Controller aus. Wenn Sie den Befehl über den passiven Controller ausführen möchten, müssen Sie das Problem mit der Verbindung zwischen passivem und aktivem Controller beheben. Sollte die Verbindung unterbrochen sein, wenden Sie sich an den Support von Microsoft.|
|2\.|0x800710dd – Die Vorgangskennung ist ungültig.|Die Proxyeinstellungen werden auf dem virtuellen StorSimple-Gerät nicht unterstützt.|Die Proxyeinstellungen werden auf dem virtuellen StorSimple-Gerät nicht unterstützt. Sie können nur für ein physisches StorSimple-Gerät konfiguriert werden.|
|3\.|0x80070057 – Ungültiger Parameter.|Einer der Parameter für die Proxyeinstellungen ist ungültig.|Der URI wurde nicht im korrekten Format angegeben. Verwenden Sie das folgende Format: `http://<IP address or FQDN of the web proxy server>:<TCP port number>`|
|4\.|0x800706ba – RPC-Server ist nicht verfügbar.|Mögliche Ursache:</br></br>Der Cluster ist nicht aktiv.</br></br>Der Datenpfaddienst wird nicht ausgeführt.</br></br>Der Befehl wird über den passiven Controller ausgeführt, und es ist keine Kommunikation mit dem aktiven Controller möglich.|Wenden Sie sich an den Support von Microsoft, um sicherzustellen, dass der Cluster aktiv ist und der Datenpfaddienst ausgeführt wird.</br></br>Führen Sie den Befehl über den aktiven Controller aus. Wenn Sie den Befehl über den passiven Controller ausführen möchten, müssen Sie sicherstellen, dass der passive Controller mit dem aktiven Controller kommunizieren kann. Sollte die Verbindung unterbrochen sein, wenden Sie sich an den Support von Microsoft.|
|5\.|0x800706be – Fehler des RPC-Aufrufs.|Cluster ist nicht verfügbar.|Wenden Sie sich an den Support von Microsoft, um sicherzustellen, dass der Cluster aktiv ist.|
|6\.|0x8007138f – Die Clusterressource wurde nicht gefunden.|Die Clusterressource des Plattformdiensts wurde nicht gefunden. Dieses Problem kann auf eine nicht ordnungsgemäße Installation zurückzuführen sein.|Das Gerät muss unter Umständen auf die Werkseinstellungen zurückgesetzt werden. Gegebenenfalls muss eine Plattformressource erstellt werden. Erkundigen Sie sich beim Support von Microsoft nach den weiteren Schritten.|
|7\.|0x8007138c – Die Clusterressource ist nicht online.|Die Plattform- oder Datenpfad-Clusterressourcen sind nicht online.|Wenden Sie sich an den Support von Microsoft, um sicherzustellen, dass die Datenpfad- und die Plattformdienstressource online sind.|

> [AZURE.NOTE]
> 
> -  Die obige Liste mit Fehlermeldungen ist nicht vollständig. 
> - Im Verwaltungsportal Ihres StorSimple Manager-Diensts werden keine Fehler für Webproxyeinstellungen angezeigt. Liegt nach Abschluss der Konfiguration ein Problem mit dem Webproxy vor, wechselt der Gerätestatus im Verwaltungsportal zu **Offline**.

## Nächste Schritte
Informationen zum Behandeln von Problemen beim Bereitstellen Ihres Geräts oder beim Konfigurieren der Webproxyeinstellungen finden Sie unter [Beheben von Problemen mit der Bereitstellung von StorSimple-Geräten](storsimple-troubleshoot-deployment.md).

<!---HONumber=August15_HO6-->