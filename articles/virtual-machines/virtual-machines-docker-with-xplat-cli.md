<properties
	pageTitle="Verwenden der Docker-VM-Erweiterung für Linux auf Azure"
	description="Beschreibt die Docker- und Azure Virtual Machines-Erweiterungen und zeigt die programmgesteuerte Erstellung von virtuellen Computern auf Azure, die von Docker gehostet werden, über die Befehlszeile mithilfe der Azure-CLI."
	services="virtual-machines"
	documentationCenter=""
	authors="squillace"
	manager="timlt"
	editor="tysonn"/>

<tags
	ms.service="virtual-machines"
	ms.devlang="multiple"
	ms.topic="article"
	ms.tgt_pltfrm="vm-linux"
	ms.workload="infrastructure-services"
	ms.date="05/22/2015"
	ms.author="rasquill"/>

# Verwenden der Docker-VM-Erweiterung aus der Azure-Befehlszeilenschnittstelle (Azure-CLI)

Dieses Thema beschreibt das Erstellen eines virtuellen Computers mit der Docker-VM-Erweiterung im ASM-Modus (Azure Service Management) in der Azure-Befehlszeilenschnittstelle auf einer beliebigen Plattform. [Docker](https://www.docker.com/) ist einer der beliebtesten Virtualisierungsansätze, der für das Isolieren von Daten und Computing auf gemeinsam genutzten Ressourcen [Linux-Container](http://en.wikipedia.org/wiki/LXC) statt virtueller Computer verwendet. Verwenden Sie die Docker-VM-Erweiterung des [Azure Linux Agent](virtual-machines-linux-agent-user-guide.md), um einen virtuellen Docker-Computer zu erstellen, der eine beliebige Anzahl von Containern für Ihre Anwendungen in Azure hostet. Eine allgemeine Diskussion über die Container und ihre Vorteile finden Sie unter [Docker High Level Whiteboard](http://channel9.msdn.com/Blogs/Regular-IT-Guy/Docker-High-Level-Whiteboard) (Whiteboard auf hoher Ebene zu Docker) (in englischer Sprache).

+ [Verwenden der Docker-VM-Erweiterung mit Azure]
+ [Erweiterungen für virtuelle Linux- und Windows-Computer]
+ [Container und Container-Verwaltungsressourcen für Azure]
+ [Nächste Schritte]

## <a id='How to use the Docker VM Extension with Azure'>Verwenden der Docker-VM-Erweiterung mit Azure</a>
Um die Docker-VM-Erweiterung mit Azure verwenden zu können, müssen Sie eine höhere Version der [Azure-Befehlszeilenschnittstelle](https://github.com/Azure/azure-sdk-tools-xplat) (Azure-CLI) als 0.8.6 installieren (Bei der Erstellung dieses Dokuments ist die aktuelle Version 0.8.10). Sie können die Azure-CLI auf Mac, Linux und Windows installieren.



Die Verwendung von Docker auf Azure ist einfach:

+ Installieren Sie die Azure-CLI und ihre Abhängigkeiten auf dem Computer, von dem aus Sie Azure steuern möchten (unter Windows ist dies eine Linux-Distribution, die als virtueller Computer ausgeführt wird).
+ Verwenden Sie die Docker-Befehle der Azure-CLI, um einen Docker-VM-Host in Azure zu erstellen.
+ Verwenden Sie die lokalen Docker-Befehle, um die Docker-Container im virtuellen Docker-Computer in Azure zu verwalten.


### Installieren der Azure-Befehlszeilenschnittstelle (Azure-CLI)

Informationen zum Installieren und Konfigurieren der Azure-CLI finden Sie unter [##Installieren und Konfigurieren der Azure-Befehlszeilenschnittstelle](../xplat-cli-install.md). Um die Installation zu bestätigen, geben Sie an der Eingabeaufforderung `azure` ein. Nach wenigen Sekunden sollten die ASCII-Zeichnungen der Azure-CLI mit den verfügbaren grundlegenden Befehlen angezeigt werden. Wenn die Installation erfolgreich war, sollten Sie bei Eingabe von `azure help vm` unter den aufgelisteten Befehlen „docker“ finden.

> [AZURE.NOTE]Docker beinhaltet für Windows das Setup-Programm [Boot2Docker](https://docs.docker.com/installation/windows/). Automatisieren Sie damit das Erstellen eines Docker-Clients, mit dem Sie Azure-VMs als Docker-Hosts verwenden können.

### Verknüpfen der Azure-CLI mit Ihrem Azure-Konto
Bevor Sie die Azure-CLI nutzen können, müssen Sie Ihre Azure-Anmeldeinformationen mit der Azure-CLI auf Ihrer Plattform verknüpfen. Der Abschnitt [Verbinden mit Ihrem Azure-Abonnement](../xplat-cli-connect.md) erläutert, wie die Datei **.publishsettings** heruntergeladen und importiert wird oder wie Ihre Azure-CLI mit einer Organisations-ID verknüpft wird.

> [AZURE.NOTE]Beide Methoden der Authentifizierung verhalten sich unterschiedlich. Lesen Sie daher unbedingt das genannte Dokument, um die Unterschiede zu verstehen.

### Installieren und Verwenden der Docker-VM-Erweiterung für Azure
Folgen Sie den [Installationsanweisungen für Docker](https://docs.docker.com/installation/#installation), um Docker auf Ihrem Computer lokal zu installieren.

Damit Docker mit einem virtuellen Azure-Computer verwendet werden kann, muss das für den virtuellen Computer verwendete Linux-Image über den installierten [Azure Linux VM Agent](virtual-machines-linux-agent-user-guide.md) verfügen. Zurzeit stehen nur zwei Imagetypen zur Verfügung, die diesen bereitstellen:

+ Ein Ubuntu-Image aus der Azure-Image-Galerie oder

+ Ein benutzerdefiniertes Linux-Image, das Sie mit dem installierten und konfigurierten Azure Linux VM Agent erstellt haben. Unter [Azure Linux VM Agent](virtual-machines-linux-agent-user-guide.md) finden Sie weitere Informationen über das Erstellen eines benutzerdefinierten virtuellen Linux-Computers mit dem Azure VM Agent.

### Verwenden des Azure-Image-Katalogs

Verwenden Sie in einer Bash- oder Terminalsitzung den folgenden Azure-CLI-Befehl, um im VM-Katalog das neueste Ubuntu-Image zu suchen. Geben Sie

`azure vm image list | grep Ubuntu-14_04`

ein, oder wählen Sie einen der Imagenamen aus, z. B. `b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04-LTS-amd64-server-20140724-de-de-30GB`. Erstellen Sie anschließend mithilfe des folgenden Befehls eine VM mit diesem Image.

```
azure vm docker create -e 22 -l "West US" <vm-cloudservice name> "b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_04-LTS-amd64-server-20140724-de-de-30GB" <username> <password>
```

Hinweis:

+ *&lt;vm-cloudservice name&gt;* ist der Name des virtuellen Computers, der zum Docker-Containerhostcomputer in Azure wird.

+  *&lt;username&gt;* ist der Benutzername des standardmäßigen Stammbenutzers des virtuellen Computers.

+ *&lt;password&gt;* ist das Kennwort des Kontos *username*, das die Komplexitätsstandards für Azure erfüllt.

> [AZURE.NOTE]Derzeit muss ein Kennwort aus mindestens 8 Zeichen bestehen und dabei einen Kleinbuchstaben, einen Großbuchstaben, eine Ziffer und eines der folgenden Sonderzeichen enthalten: `!@#$%^&+=`. Der Punkt am Ende des vorhergehenden Satzes zählt nicht zu den Sonderzeichen.

War die Eingabe erfolgreich, sollte Ihnen in etwa das Folgende angezeigt werden, abhängig von den genauen Argumenten und Optionen, die Sie verwendet haben:

![](./media/virtual-machines-docker-with-xplat-cli/dockercreateresults.png)

> [AZURE.NOTE]Das Erstellen eines virtuellen Computers kann einige Minuten in Anspruch nehmen. Nach der Bereitstellung wird der Docker-Daemon (der Docker-Dienst) jedoch gestartet, und Sie können eine Verbindung zum Docker-Containerhost herstellen.

Um den virtuellen Docker-Computer, den Sie in Azure erstellt haben, zu testen, geben Sie Folgendes ein:

`docker --tls -H tcp://<vm-name-you-used>.cloudapp.net:4243 info`

*<vm-name-you-used>* ist der Name des virtuellen Computers, den Sie in Ihrem Aufruf von `azure vm docker create` verwendet haben. Es sollte etwas angezeigt werden, das in etwa dem Folgenden entspricht, welches angibt, dass Ihr virtueller Docker-Hostcomputer betriebsbereit ist, in Azure ausgeführt wird und auf Ihre Befehle wartet.

![](./media/virtual-machines-docker-with-xplat-cli/connectingtodockerhost.png)

### Authentifizierung des virtuellen Docker-Hostcomputers
Neben dem virtuellen Docker-Computer erstellt der Befehl `azure vm docker create` automatisch auch die nötigen Zertifikate, um Ihrem Docker-Clientcomputer über HTTPS die Verbindung mit dem Azure-Containerhost zu ermöglichen. Die Zertifikate werden auf dem Client- und dem Hostcomputer gespeichert. Bei den nachfolgenden Ausführungen werden die Zertifikate erneut verwendet und an den neuen Host freigegeben.

Standardmäßig werden die Zertifikate unter `~/.docker` abgelegt, und Docker wird für die Ausführung auf Port **4243** konfiguriert. Wenn Sie einen anderen Port oder ein anderes Verzeichnis verwenden möchten, können Sie eine der folgenden Befehlszeilenoptionen `azure vm docker create` verwenden, damit der virtuelle Docker-Containerhost einen anderen Port oder andere Zertifikate verwendet, um eine Verbindung mit Clients herzustellen:

```
-dp, --docker-port [port]              Port to use for docker [4243]
-dc, --docker-cert-dir [dir]           Directory containing docker certs [.docker/]
```

Der Docker-Daemon auf dem Host ist so konfiguriert, dass er Clientverbindungen auf dem angegebenen Port mithilfe der durch den Befehl `azure vm docker create` generierten Zertifikate abhört und authentifiziert. Der Clientcomputer muss über diese Zertifikate verfügen, um Zugriff auf den Docker-Host zu erlangen.

> [AZURE.NOTE]Ein vernetzter Host ist ohne diese Zertifikate anfällig für jeden, der sich mit dem Computer verbindet. Bevor Sie die Standardkonfiguration ändern, setzen Sie sich mit den Risiken für Ihre Computer und Anwendungen auseinander.




## Nächste Schritte

Sie sind nun bereit, das [Docker-Benutzerhandbuch] und Ihren virtuellen Docker-Computer zu verwenden. Informationen zum Erstellen eines Docker-fähigen virtuellen Computers im neuen Portal finden Sie unter [Gewusst wie: Verwenden der Docker-VM-Erweiterung mit dem Portal].

<!--Anchors-->
[Subheading 1]: #subheading-1
[Subheading 2]: #subheading-2
[Subheading 3]: #subheading-3
[Nächste Schritte]: #next-steps

[Verwenden der Docker-VM-Erweiterung mit Azure]: #How-to-use-the-Docker-VM-Extension-with-Azure
[Erweiterungen für virtuelle Linux- und Windows-Computer]: #Virtual-Machine-Extensions-For-Linux-and-Windows
[Container und Container-Verwaltungsressourcen für Azure]: #Container-and-Container-Management-Resources-for-Azure

<!--Image references-->
[5]: ./media/markdown-template-for-new-articles/octocats.png
[6]: ./media/markdown-template-for-new-articles/pretty49.png
[7]: ./media/markdown-template-for-new-articles/channel-9.png


<!--Link references-->
[Link 1 to another azure.microsoft.com documentation topic]: virtual-machines-windows-tutorial.md
[Link 2 to another azure.microsoft.com documentation topic]: ../web-sites-custom-domain-name.md
[Link 3 to another azure.microsoft.com documentation topic]: ../storage-whatis-account.md
[Gewusst wie: Verwenden der Docker-VM-Erweiterung mit dem Portal]: http://azure.microsoft.com/documentation/articles/virtual-machines-docker-with-portal/

[Docker-Benutzerhandbuch]: https://docs.docker.com/userguide/
 

<!---HONumber=58_postMigration-->