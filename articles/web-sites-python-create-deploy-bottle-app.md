﻿<properties 
	pageTitle="Python-Websites mit Bottle - Azure-Lernprogramm" 
	description="In diesem Lernprogramm werden Sie in die Ausführung einer Python-Website in Azure eingeführt." 
	services="web-sites" 
	documentationCenter="python" 
	authors="huguesv" 
	manager="" 
	editor=""/>

<tags 
	ms.service="web-sites" 
	ms.workload="web" 
	ms.tgt_pltfrm="na" 
	ms.devlang="python" 
	ms.topic="article" 
	ms.date="12/17/2014" 
	ms.author="huvalo"/>




# Erstellen von Websites mit Bottle

In diesem Lernprogramm werden die ersten Schritte bei der Ausführung von Python auf Azure-Websites beschrieben.  Azure-Websites bieten uneingeschränktes kostenloses Hosting und schnelle Bereitstellung, und Sie können Python verwenden!  Wenn die Anwendung größer wird, können Sie zu kostenpflichtigem Hosting wechseln und außerdem alle anderen Azure-Dienste integrieren.

Sie erstellen eine Anwendung mithilfe das Webframeworks Bottle (siehe auch die alternativen Versionen dieses Lernprogramms für [Django](../web-sites-python-create-deploy-django-app) und Flask](../web-sites-python-create-deploy-flask-app))).  Sie erstellen die Website aus dem Azure-Katalog, richten die Git-Bereitstellung ein und klonen das Repository lokal.  Dann führen Sie die Anwendung lokal aus, nehmen Änderungen vor, führen ein Commit aus und übertragen die Änderungen in Azure.  Das Lernprogramm zeigt, wie dies unter Windows oder Mac/Linux erfolgt.

[AZURE.INCLUDE [create-account-and-websites-note](../includes/create-account-and-websites-note.md)]


+ [Voraussetzungen](#prerequisites)
+ [Erstellen von Websites im Portal](#website-creation-on-portal)
+ [Anwendungsübersicht](#application-overview)
+ Entwickeln von Websites
  + [Windows - Python Tools für Visual Studio](#website-development-windows-ptvs)
  + [Windows - Befehlszeile](#website-development-windows-command-line)
  + [Mac/Linux - Befehlszeile](#website-development-mac-linux-command-line)
+ [Problembehandlung - Bereitstellung](#troubleshooting-deployment)
+ [Problembehandlung - Paketinstallation](#troubleshooting-package-installation)
+ [Problembehandlung - Virtuelle Umgebung](#troubleshooting-virtual-environment)
+ [Nächste Schritte](#next-steps)


<h2><a name="prerequisites"></a>Voraussetzungen</h2>

- Windows, Mac oder Linux
- Python 2.7 oder 3.4
- setuptools, pip, virtualenv (nur Python 2.7)
- Git
- Python Tools für Visual Studio (optional)

**Hinweis**: TFS-Veröffentlichung wird derzeit für Python-Projekte nicht unterstützt.

### Windows

Falls Python 2.7 oder 3.4 (32 Bit) noch nicht installiert ist, empfehlen wir die Installation von [Azure SDK für Python 2.7](http://go.microsoft.com/fwlink/?linkid=254281&clcid=0x409) oder [Azure SDK für Python 3.4](http://go.microsoft.com/fwlink/?LinkID=516990&clcid=0x409) über den Webplattform-Installer.  Dadurch werden die 32-Bit-Version von Python, setuptools, pip, virtualenv usw. installiert (32-Bit-Python ist auf den Azure-Hostcomputern installiert).  Alternativ können Sie Python von [python.org](http://www.python.org/) abrufen.

Für Git empfehlen wir [Git für Windows](http://msysgit.github.io/) oder [GitHub für Windows](https://windows.github.com/).  Wenn Sie Visual Studio verwenden, können Sie die integrierte Unterstützung für Git nutzen.

Wir empfehlen auch die Installation von [Python Tools für Visual Studio](http://pytools.codeplex.com).  Dies ist optional, aber wenn Sie über [Visual Studio](http://www.visualstudio.com/) einschließlich des kostenlosen Visual Studio Express 2013 for Web verfügen, erhalten Sie eine leistungsfähige Python IDE.

### Mac/Linux

Sie sollten Python und Git bereits installiert haben, doch stellen Sie sicher, dass Sie über Python 2.7 oder 3.4 verfügen.


<h2><a name="website-creation-on-portal"></a>Erstellen von Websites im Portal</h2>

Der erste Schritt beim Erstellen einer App ist das Erstellen der Website im Azure-Verwaltungsportal.  Melden Sie sich dazu beim Portal an, und klicken Sie unten links auf die Schaltfläche **NEU**. Daraufhin wird ein Fenster angezeigt. Klicken Sie auf **SERVER**, **WEBSITE**, **AUS KATALOG**.

![](./media/web-sites-python-create-deploy-bottle-app/portal-create-site.png)

Ein Fenster wird angezeigt, in dem Apps aufgeführt werden, die im Katalog verfügbar sind. Klicken Sie links auf die Kategorie **APP FRAMEWORKS**, und wählen Sie **Bottle** aus.

![](./media/web-sites-python-create-deploy-bottle-app/portal-gallery-bottle.png)

Geben Sie auf der nächsten Seite einen Namen und eine Region für Ihre Website ein, und klicken Sie auf die Schaltfläche "Fertig".

Die Website wird umgehend eingerichtet. Wenn Sie auf die Schaltfläche**Durchsuchen** auf der unteren Symbolleiste klicken, sehen Sie, dass Ihre neue Bottle-Anwendung in Azure ausgeführt wird.

![](./media/web-sites-python-create-deploy-bottle-app/portal-website-bottle.png)
 
Nun fügen Sie Unterstützung für die Veröffentlichung mit Git hinzu.  Dazu wählen Sie **Bereitstellung über Quellcodeverwaltung einrichten** aus.

![](./media/web-sites-python-create-deploy-bottle-app/portal-site-created.png)

Führen Sie im Dialogfeld **Bereitstellung einrichten** einen Bildlauf nach unten durch, und wählen Sie die Option **Lokales Git-Repository** aus. Klicken Sie auf den Pfeil nach rechts, um fortzufahren.

![](./media/web-sites-python-create-deploy-bottle-app/portal-setup-deployment.png)

Nach dem Einrichten der Git-Veröffentlichung wird kurz eine Seite mit der Information zur Repository-Erstellung angezeigt. Nach der Erstellung sehen Sie die Anweisungen zum Herstellen einer Verbindung.  

![](./media/web-sites-python-create-deploy-bottle-app/portal-repo-created.png)

Wir folgen diesen Anweisungen in den nächsten Abschnitten.


<h2><a name="application-overview"></a>Anwendungsübersicht</h2>

### Inhalt des Git-Repositorys

Hier sehen Sie eine Übersicht der Dateien, die sich im Git-Ausgangsrepository befinden, das wir im nächsten Abschnitt klonen.

    \routes.py
    \static\content\
    \static\fonts\
    \static\scripts\
    \views\about.tpl
    \views\contact.tpl
    \views\index.tpl
    \views\layout.tpl

Die Hauptquellen für die Anwendung.  Besteht aus 3 Seiten (Index, Info, Kontakt) mit einem Masterlayout.  Statische Inhalte und Skripts umfassen bootstrap, jquery, modernizr und respond.

    \app.py

Unterstützung für lokale Entwicklungsserver. Dient zum lokalen Ausführen der Anwendung.

    \BottleWebProject.pyproj
    \BottleWebProject.sln

Projektdateien für die Verwendung mit [Python Tools für Visual Studio](http://pytools.codeplex.com).

    \ptvs_virtualenv_proxy.py

IIS-Proxy für virtuelle Umgebungen und PTVS-Unterstützung für Remotedebugging.

    \requirements.txt

Externe Pakete, die von dieser Anwendung benötigt werden. Das Bereitstellungsskript installiert die in dieser Datei aufgelisteten Pakete mit pip.
 
    \web.2.7.config
    \web.3.4.config

IIS-Konfigurationsdateien.  Das Bereitstellungsskript verwendet die entsprechende Datei "web.x.y.config" und kopiert sie als "web.config".

### Optionale Dateien - Anpassung der Bereitstellung

[AZURE.INCLUDE [web-sites-python-customizing-deployment](../includes/web-sites-python-customizing-deployment.md)]

### Optionale Dateien - Python-Laufzeit

[AZURE.INCLUDE [web-sites-python-customizing-runtime](../includes/web-sites-python-customizing-runtime.md)]

### Zusätzliche Dateien auf dem Server

Einige Dateien sind auf dem Server vorhanden, werden jedoch nicht dem Git-Repository hinzugefügt.  Diese werden vom Bereitstellungsskript erstellt.

    \web.config

IIS-Konfigurationsdatei.  Wird anhand von "web.x.y.config" bei jeder Bereitstellung erstellt.

    \env\

Virtuelle Umgebung für Python.  Wird während der Bereitstellung erstellt, wenn nicht bereits eine kompatible virtuelle Umgebung auf der Website vorhanden ist.  Pakete, die in "requirements.txt" aufgeführt sind, werden mit pip installiert. pip überspringt die Installation, wenn die Pakete bereits installiert sind.

In den nächsten drei Abschnitten wird beschrieben, wie Sie die Websiteentwicklung in 3 unterschiedlichen Umgebungen fortsetzen:

- Windows - mit Python Tools für Visual Studio
- Windows, über die Befehlszeile
- Mac/Linux, über die Befehlszeile


<h2><a name="website-development-windows-ptvs"></a>Website-Entwicklung - Windows - Python Tools für Visual Studio</h2>

### Klonen des Repositorys

Klonen Sie zunächst das Repository mithilfe der URL, die im Azure-Portal bereitgestellt ist.

![](./media/web-sites-python-create-deploy-bottle-app/ptvs-git-clone.png)

Öffnen Sie die Projektmappendatei (.sln), die im Stammverzeichnis des Repositorys enthalten ist.

![](./media/web-sites-python-create-deploy-bottle-app/ptvs-solution-bottle.png)

### Erstellen einer virtuellen Umgebung

Jetzt erstellen wir eine virtuelle Umgebung für die lokale Entwicklung.  Klicken Sie mit der rechten Maustaste auf **Python-Umgebungen**, und wählen Sie **Virtuelle Umgebung hinzufügen...** aus.

- Stellen Sie sicher, dass der Name der Umgebung  `env` lautet.

- Wählen Sie den Basisinterpreter.  Stellen Sie sicher, dass Sie die gleiche Version von Python verwenden, die für Ihre Website (in "runtime.txt" oder der Seite zum Konfigurieren der Website) ausgewählt ist.

- Stellen Sie sicher, dass die Option zum Herunterladen und Installieren von Paketen aktiviert ist.

![](./media/web-sites-python-create-deploy-bottle-app/ptvs-add-virtual-env-27.png)

Klicken Sie auf **Erstellen**.  Dadurch wird die virtuelle Umgebung erstellt, und die in "requirements.txt" aufgelisteten Abhängigkeiten werden installiert.

### Ausführen mithilfe des Entwicklungssservers

Drücken Sie F5, um mit dem Debuggen beginnen. Ihr Webbrowser wird automatisch auf der Seite geöffnet, die lokal ausgeführt.

![](./media/web-sites-python-create-deploy-bottle-app/windows-browser-bottle.png)

Sie können Haltepunkte im Quellcode setzen, die Überwachungsfenster verwenden usw. In der [PTVS-Dokumentation](http://pytools.codeplex.com/documentation) finden Sie weitere Informationen zu den verschiedenen Features.

### Vornehmen von Änderungen

Jetzt können Sie experimentieren, indem Sie Änderungen an der Anwendungsquellen und/oder Vorlagen vornehmen.

Nachdem Sie Ihre Änderungen getestet haben, führen Sie für diese im Git-Repository einen Commit durch:

![](./media/web-sites-python-create-deploy-bottle-app/ptvs-commit-bottle.png)

### Installieren weiterer Pakete

Ihre Anwendung weist möglicherweise Abhängigkeiten über Python und Bottle hinaus auf.

Sie können zusätzliche Pakete mit pip installieren.  Klicken Sie zum Installieren eines Pakets mit der rechten Maustaste auf die virtuelle Umgebung, und wählen Sie **Python-Paket installieren**.

Geben Sie beispielsweise zum Installieren des Azure SDK für Python, das Ihnen Zugriff auf Azure-Speicher, den Service Bus und andere Azure-Dienste bereitstellt, Folgendes ein  `azure`:

![](./media/web-sites-python-create-deploy-bottle-app/ptvs-install-package-dialog.png)

Klicken Sie mit der rechten Maustaste auf die virtuelle Umgebung, und wählen Sie **"requirements.txt" generieren**, um die Datei "requirements.txt" zu aktualisieren.

Führen Sie anschließend für die Änderungen an "requirements.txt" einen Commit im Git-Repository aus.

### Bereitstellen in Azure

Klicken Sie auf, **Synchronisierung** oder **Push**, um eine Bereitstellung auszulösen.  Bei Wahl von "Synchronisierung" erfolgt sowohl ein Push- als auch ein Pull-Vorgang.

![](./media/web-sites-python-create-deploy-bottle-app/ptvs-git-push.png)

Die erste Bereitstellung nimmt einige Zeit in Anspruch, da eine virtuelle Umgebung erstellt wird, Pakete installiert werden usw.

Visual Studio zeigt nicht den Fortschritt der Bereitstellung an.  Wenn Sie die Ausgabe prüfen möchten, lesen Sie den Abschnitt [Problembehandlung - Bereitstellung](#troubleshooting-deployment).

Navigieren Sie zur Azure-URL, um die Änderungen anzuzeigen.


<h2><a name="website-development-windows-command-line"></a>Website-Entwicklung - Windows - Befehlszeile</h2>

### Klonen des Repositorys

Klonen Sie zunächst das Repository mithilfe der URL, die im Azure-Portal bereitgestellt ist, und fügen Sie das Azure-Repository als Remoterepository hinzu.

    git clone <repo-url>
    cd <repo-folder>
    git remote add azure <repo-url> 

### Erstellen einer virtuellen Umgebung

Wir erstellen eine neue virtuelle Umgebung zu Entwicklungszwecken (die nicht dem Repository hinzugefügt werden darf).  Virtuelle Umgebungen in Python sind nicht verschiebbar, sodass jeder Entwickler, der an der Anwendung arbeitet, seine eigene lokal erstellen muss.

Stellen Sie sicher, dass Sie die gleiche Version von Python verwenden, die für Ihre Website (in "runtime.txt" oder der Seite zum Konfigurieren der Website) ausgewählt ist.

Für Python 2.7:

    c:\python27\python.exe -m virtualenv env

Für Python 3.4:

    c:\python34\python.exe -m venv env

Installieren Sie alle externen Pakete, die Ihre Anwendung benötigt. Sie können die Datei "requirements.txt" im Stammverzeichnis des Repositorys verwenden, um die Pakete in der virtuellen Umgebung zu installieren:

    env\scripts\pip install -r requirements.txt

### Ausführen mithilfe des Entwicklungssservers

Sie können die Anwendung unter einem Entwicklungsserver mit dem folgenden Befehl starten:

    env\scripts\python app.py

Die Konsole zeigt die URL und den Port, den der Server abhört:

![](./media/web-sites-python-create-deploy-bottle-app/windows-run-local-bottle.png)

Öffnen Sie Ihren Webbrowser mit dieser URL.

![](./media/web-sites-python-create-deploy-bottle-app/windows-browser-bottle.png)

### Vornehmen von Änderungen

Jetzt können Sie experimentieren, indem Sie Änderungen an der Anwendungsquellen und/oder Vorlagen vornehmen.

Nachdem Sie Ihre Änderungen getestet haben, führen Sie für diese im Git-Repository einen Commit durch:

    git add <modified-file>
    git commit -m "<commit-comment>"

### Installieren weiterer Pakete

Ihre Anwendung weist möglicherweise Abhängigkeiten über Python und Bottle hinaus auf.

Sie können zusätzliche Pakete mit pip installieren.  Geben Sie beispielsweise zum Installieren des Azure SDK für Python, das Ihnen Zugriff auf Azure-Speicher, den Service Bus und andere Azure-Dienste bereitstellt, Folgendes ein:

    env\scripts\pip install azure

Stellen Sie sicher, dass "requirements.txt" aktualisiert wird:

    env\scripts\pip freeze > requirements.txt

Führen Sie für die Änderungen einen Commit aus:

    git add requirements.txt
    git commit -m "Added azure package"

### Bereitstellen in Azure

Um eine Bereitstellung auszulösen, übertragen Sie die Änderungen per Push in Azure:

    git push azure master

Die Ausgabe des Bereitstellungsskripts wird mit Angaben zur Erstellung der virtuellen Umgebung, Installation von Paketen und Erstellung von "web.config" angezeigt.

Navigieren Sie zur Azure-URL, um die Änderungen anzuzeigen.


<h2><a name="website-development-mac-linux-command-line"></a>Website-Entwicklung - Mac/Linux - Befehlszeile</h2>

### Klonen des Repositorys

Klonen Sie zunächst das Repository mithilfe der URL, die im Azure-Portal bereitgestellt ist, und fügen Sie das Azure-Repository als Remoterepository hinzu.

    git clone <repo-url>
    cd <repo-folder>
    git remote add azure <repo-url> 

### Erstellen einer virtuellen Umgebung

Wir erstellen eine neue virtuelle Umgebung zu Entwicklungszwecken (die nicht dem Repository hinzugefügt werden darf).  Virtuelle Umgebungen in Python sind nicht verschiebbar, sodass jeder Entwickler, der an der Anwendung arbeitet, seine eigene lokal erstellen muss.

Stellen Sie sicher, dass Sie die gleiche Version von Python verwenden, die für Ihre Website (in "runtime.txt" oder der Seite zum Konfigurieren der Website) ausgewählt ist.

Für Python 2.7:

    python -m virtualenv env

Für Python 3.4:

    python -m venv env

Installieren Sie alle externen Pakete, die Ihre Anwendung benötigt. Sie können die Datei "requirements.txt" im Stammverzeichnis des Repositorys verwenden, um die Pakete in der virtuellen Umgebung zu installieren:

    env/bin/pip install -r requirements.txt

### Ausführen mithilfe des Entwicklungssservers

Sie können die Anwendung unter einem Entwicklungsserver mit dem folgenden Befehl starten:

    env/bin/python app.py

Die Konsole zeigt die URL und den Port, den der Server abhört:

![](./media/web-sites-python-create-deploy-bottle-app/mac-run-local-bottle.png)

Öffnen Sie Ihren Webbrowser mit dieser URL.

![](./media/web-sites-python-create-deploy-bottle-app/mac-browser-bottle.png)

### Vornehmen von Änderungen

Jetzt können Sie experimentieren, indem Sie Änderungen an der Anwendungsquellen und/oder Vorlagen vornehmen.

Nachdem Sie Ihre Änderungen getestet haben, führen Sie für diese im Git-Repository einen Commit durch:

    git add <modified-file>
    git commit -m "<commit-comment>"

### Installieren weiterer Pakete

Ihre Anwendung weist möglicherweise Abhängigkeiten über Python und Bottle hinaus auf.

Sie können zusätzliche Pakete mit pip installieren.  Geben Sie beispielsweise zum Installieren des Azure SDK für Python, das Ihnen Zugriff auf Azure-Speicher, den Service Bus und andere Azure-Dienste bereitstellt, Folgendes ein:

    env/bin/pip install azure

Stellen Sie sicher, dass "requirements.txt" aktualisiert wird:

    env/bin/pip freeze > requirements.txt

Führen Sie für die Änderungen einen Commit aus:

    git add requirements.txt
    git commit -m "Added azure package"

### Bereitstellen in Azure

Um eine Bereitstellung auszulösen, übertragen Sie die Änderungen per Push in Azure:

    git push azure master

Die Ausgabe des Bereitstellungsskripts wird mit Angaben zur Erstellung der virtuellen Umgebung, Installation von Paketen und Erstellung von "web.config" angezeigt.

Navigieren Sie zur Azure-URL, um die Änderungen anzuzeigen.


<h2><a name="troubleshooting-deployment"></a>Problembehandlung - Bereitstellung</h2>

[AZURE.INCLUDE [web-sites-python-troubleshooting-deployment](../includes/web-sites-python-troubleshooting-deployment.md)]


<h2><a name="troubleshooting-package-installation"></a>Problembehandlung - Paketinstallation</h2>

[AZURE.INCLUDE [web-sites-python-troubleshooting-package-installation](../includes/web-sites-python-troubleshooting-package-installation.md)]


<h2><a name="troubleshooting-virtual-environment"></a>Problembehandlung - Virtuelle Umgebung</h2>

[AZURE.INCLUDE [web-sites-python-troubleshooting-virtual-environment](../includes/web-sites-python-troubleshooting-virtual-environment.md)]


<h2><a name="next-steps"></a>Nächste Schritte</h2>

Folgen Sie diesen Links, um weitere Informationen zu Bottle- und Python Tools für Visual Studio zu erhalten: 
 
- [Dokumentation zu Bottle][]
- [Python Tools für Visual Studio - Dokumentation][] 

Weitere Informationen zur Verwendung von Azure Table Storage und MongoDB:

- [Bottle und MongoDB in Azure mit Python Tools 2.1 für Visual Studio][]
- [Bottle und Azure Table Storage in Azure mit Python Tools 2.1 für Visual Studio][]


<!--Link references-->
[Bottle und MongoDB in Azure mit Python Tools 2.1 für Visual Studio]: ../web-sites-python-ptvs-bottle-table-storage
[Bottle und Azure Table Storage in Azure mit Python Tools 2.1 für Visual Studio]: ../web-sites-python-ptvs-bottle-mongodb

<!--External Link references-->
[Python Tools für Visual Studio - Dokumentation]: http://pytools.codeplex.com/documentation 
[Dokumentation zu Bottle]: http://bottlepy.org/docs/dev/index.html




<!--HONumber=42-->