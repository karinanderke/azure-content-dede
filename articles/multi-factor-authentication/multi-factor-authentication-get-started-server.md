<properties 
	pageTitle="Erste Schritte mit Azure Multi-Factor Authentication-Server" 
	description="Auf dieser Seite zur Azure Multi-Factor Authentication werden die ersten Schritte mit dem Azure MFA-Server beschrieben." 
	services="multi-factor-authentication" 
	documentationCenter="" 
	authors="billmath" 
	manager="terrylan" 
	editor="bryanla"/>

<tags 
	ms.service="multi-factor-authentication" 
	ms.workload="identity" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="06/02/2015" 
	ms.author="billmath"/>

# Erste Schritte mit Azure Multi-Factor Authentication-Server




<center>![Cloud](./media/multi-factor-authentication-get-started-server/server2.png)</center>

Nachdem Sie nun geklärt haben, ob Sie die lokale Multi-Factor Authentication verwenden möchten, können Sie mit der Einrichtung fortfahren. Auf dieser Seite werden eine Neuinstallation des Servers und das Setup mit einem lokalen Active Directory behandelt. Wenn Sie den PhoneFactor-Server bereits installiert haben und eine Anleitung für dieses Verfahren suchen, finden Sie weitere Informationen unter "Aktualisieren von Azure Multi-Factor-Server". Wenn Sie wissen möchten, wie Sie nur den Webdienst installieren, finden Sie weitere Informationen unter "Bereitstellen des Webdiensts der mobilen App-für Azure Multi-Factor Authentication-Server".



## Herunterladen des Azure Multi-Factor Authentication-Servers

Es gibt zwei Möglichkeiten, den Azure Multi-Factor Authentication-Server herunterzuladen. Die erste besteht darin, sich beim Azure-Portal anzumelden, und die zweite Möglichkeit ist der Download direkt von [https://pfweb.phonefactor.net](https://pfweb.phonefactor.net).


### So laden Sie den Azure Multi-Factor Authentication-Server im Azure-Portal herunter
--------------------------------------------------------------------------------

1. Melden Sie sich beim Azure-Portal als Administrator an.
2. Wählen Sie im linken Bereich "Active Directory" aus.
3. Klicken Sie auf der Seite "Active Directory" oben auf **Anbieter für mehrstufige Authentifizierung**.
4. Klicken Sie unten auf **Verwalten**.
5. Klicken Sie auf **Downloads**.
6. Klicken Sie über **Anmeldeinformationen für Aktivierung generieren** auf **Herunterladen**.
7. Speichern Sie den Download.

### So laden Sie den Azure Multi-Factor Authentication-Server direkt herunter
--------------------------------------------------------------------------------

1. Melden Sie sich bei [https://pfweb.phonefactor.net](https://pfweb.phonefactor.net) an.
2. Klicken Sie auf **Downloads**.
<center>![Cloud](./media/multi-factor-authentication-get-started-server/download2.png)</center>
3. Klicken Sie über **Anmeldeinformationen für Aktivierung generieren** auf **Herunterladen**. Schließen Sie die Seite nicht, da Sie sie später wieder verwenden.
4. Speichern Sie den Download.



## Installieren und Konfigurieren des Azure Multi-Factor Authentication-Servers
Nachdem Sie den Server heruntergeladen haben, können Sie ihn installieren und konfigurieren. Achten Sie darauf, dass der Server, den Sie installieren möchten, die folgenden Anforderungen erfüllt:



Anforderungen an den Azure Multi-Factor Authentication-Server|Beschreibung|
:------------- | :------------- | 
Hardware|<li>200 MB freier Festplattenspeicher</li><li>x32- oder x64-Prozessor</li><li>Mindestens 1 GB RAM</li>
Software|<li>Windows Server 2003 oder höher, wenn der Host über ein Serverbetriebssystem verfügt</li><li>Windows Vista oder höher, wenn der Host über ein Clientbetriebssystem verfügt</li><li>Microsoft .NET 2.0 Framework</li><li>IIS 6.0 oder höher, wenn das Benutzerportal oder das Webdienst-SDK installiert wird</li>

### Anforderungen an die Azure Multi-Factor Authentication-Server-Firewall
--------------------------------------------------------------------------------
Für jeden MFA-Server muss die Kommunikation über den ausgehenden Port 443 zu folgenden Adressbereichen möglich sein:

- https://pfd.phonefactor.net
- https://pfd2.phonefactor.net
- https://css.phonefactor.net

Wenn Firewalls auf den ausgehenden Port 443 eingeschränkt sind, müssen die folgenden IP-Adressbereiche geöffnet werden:

IP-Subnetz|Netzmaske|IP-Bereich
:------------- | :------------- | :------------- |
134.170.116.0/25|255.255.255.128|134.170.116.1–134.170.116.126
134.170.165.0/25|255.255.255.128|134.170.165.1–134.170.165.126
70.37.154.128/25|255.255.255.128|70.37.154.129–70.37.154.254

Wenn Sie die Ereignisbestätigungsfunktionen von Azure Multi-Factor Authentication nicht verwenden und Benutzer sich nicht mit den mobilen Apps für Multi-Factor Authentication von Geräten aus authentifizieren, die sich im Firmennetzwerk befinden, sind nur die folgenden IP-Adressbereiche erforderlich:


IP-Subnetz|Netzmaske|IP-Bereich
:------------- | :------------- | :------------- |
134.170.116.72/29|255.255.255.248|134.170.116.72–134.170.116.79
134.170.165.72/29|255.255.255.248|134.170.165.72–134.170.165.79
70.37.154.200/29|255.255.255.248|70.37.154.201–70.37.154.206


### So installieren und konfigurieren Sie den Azure Multi-Factor Authentication-Server
--------------------------------------------------------------------------------


1. Doppelklicken Sie auf die ausführbare Datei. Dadurch wird die Installation gestartet.
2. Vergewissern Sie sich, dass im Bildschirm "Installationsordner auswählen" der richtige Ordner ausgewählt ist, und klicken Sie auf "Weiter".
3. Klicken Sie nach Abschluss der Installation auf "Fertig stellen". Dadurch wird der Konfigurations-Assistent gestartet.
4. Aktivieren Sie auf der Willkommensseite des Konfigurations-Assistenten das Kontrollkästchen in **Verwendung des Authentifizierungskonfigurations-Assistenten überspringen**, und klicken Sie auf **Weiter**. Der Assistent wird jetzt geschlossen und der Server gestartet.
<center>![Cloud](./media/multi-factor-authentication-get-started-server/skip2.png)</center>

5. Wechseln Sie zurück zu der Seite, von der Sie den Server heruntergeladen haben, und klicken Sie auf die Schaltfläche **Anmeldeinformationen für Aktivierung generieren**. Kopieren Sie diese Informationen im Azure MFA-Server in die Textfelder, und klicken Sie auf **Aktivieren**.


Die Schritte oben beschreiben ein Express-Setup mit dem Konfigurations-Assistenten. Sie können den Authentifizierungs-Assistenten erneut ausführen, indem Sie im Menü "Extras" auf dem Server die entsprechende Option auswählen.



##Importieren von Benutzern aus Active Directory

Nachdem Sie den Server installiert und konfiguriert haben, können Sie Benutzer schnell in den Azure MFA-Server importieren.

### So importieren Sie Benutzer aus Active Directory
--------------------------------------------------------------------------------


1. Wählen Sie im Azure MFA-Server auf der linken Seite **Benutzer** aus.
2. Wählen Sie unten **Aus Active Directory importieren** aus.
3. Sie können jetzt entweder nach einzelnen Benutzern suchen oder im AD-Verzeichnis nach Organisationseinheiten suchen, die die gewünschten Benutzer enthalten. In diesem Beispiel wird die Benutzer-Organisationseinheit angeben.
4. Markieren Sie alle Benutzer auf der rechten Seite, und klicken Sie auf **Importieren**. Es wird ein Popupfenster angezeigt, wenn der Vorgang erfolgreich ausgeführt wurde. Schließen Sie das Fenster "Importieren".

<center>![Cloud](./media/multi-factor-authentication-get-started-server/import2.png)</center>

<!---HONumber=July15_HO2-->