<properties
   pageTitle="Verwenden von SSH-Schlüsseln mit Linux-basiertem Hadoop unter Linux, Unix oder OS X | Microsoft Azure"
   description="Auf das Linux-basierte HDInsight können Sie über Secure Shell (SSH) zugreifen. Dieses Dokument enthält Informationen zur Verwendung von SSH mit HDInsight auf Linux-, Unix- oder OS X-Clients."
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="03/20/2015"
   ms.author="larryfr"/>

#Verwenden von SSH mit Linux-basiertem Hadoop in HDInsight unter Linux, Unix oder OS X (Vorschau)

> [AZURE.SELECTOR]
- [Windows](hdinsight-hadoop-linux-use-ssh-windows.md)
- [Linux, Unix, OS X](hdinsight-hadoop-linux-use-ssh-unix.md)

Linux-basierte Azure-HDInsight-Cluster bieten die Möglichkeit des SSH-Zugriffs (Secure Shell) mithilfe eines Kennworts oder SSH-Schlüssels. Dieses Dokument enthält Informationen zur Verwendung von SSH mit HDInsight auf Linux-, Unix- oder OS X-Clients.

> [AZURE.NOTE]Bei den Schritten in diesem Artikel wird davon ausgegangen, dass Sie einen Linux-, Unix- oder OS X-Client verwenden. Obwohl diese Schritte auf einem Windows-Client ausgeführt werden können, wenn Sie ein Paket installiert haben, das `ssh` und `ssh-keygen` (z. B. Git für Windows) bereitstellt, wird empfohlen, dass Windows-Clients die Schritte unter [Verwenden von SSH mit Linux-basiertem HDInsight (Hadoop) unter Windows](hdinsight-hadoop-linux-use-ssh-windows.md) befolgen.

##Voraussetzungen

* **ssh-keygen** und **ssh** für Linux-, Unix- und OS X-Clients. Diese Hilfsprogramme werden in der Regel vom Betriebssystem oder über das Paketverwaltungssystem bereitgestellt.

* Ein zeitgemäßer Webbrowser, der HTML5 unterstützt.

OR

* [Azure-CLI für Mac, Linux und Windows](../xplat-cli.md).

##Was ist SSH?

SSH ist ein Dienstprogramm zur Anmeldung und Remoteausführung von Befehlen auf einem Remoteserver. Bei Linux-basiertem HDInsight stellt SSH eine verschlüsselte Verbindung mit dem Hauptknoten des Clusters her und bietet eine Befehlszeile, über die Sie Befehle eingeben können. Die Befehle werden dann direkt auf dem Server ausgeführt.

##Erstellen eines SSH-Schlüssels (optional)

Wenn Sie einen Linux-basierten HDInsight-Cluster erstellen, können Sie sich bei Verwenden von SSH mithilfe eines Kennworts oder SSH-Schlüssels beim Server authentifizieren. SSH-Schlüssel werden als sicherer erachtet, da sie auf Zertifikaten basieren. Verwenden Sie die folgenden Informationen, wenn Sie für Ihren Cluster die Verwendung von SSH-Schlüsseln planen.

1. Öffnen Sie eine Terminalsitzung, und verwenden Sie den folgenden Befehl, um zu prüfen, ob SSH-Schlüssel vorhanden sind:

		ls -al ~/.ssh

	Achten Sie auf die folgenden Dateien in der Verzeichnisauflistung. Hierbei handelt es sich um allgemeine Namen für öffentliche SSH-Schlüssel.

	* id_dsa.pub
	* id_ecdsa.pub
	* id_ed25519.pub
	* id_rsa.pub

2. Wenn Sie keine vorhandene Datei verwenden möchten oder keine SSH-Schlüssel vorhanden sind, führen Sie die folgenden Schritte zum Generieren einer neuen Datei aus:

		ssh-keygen -t rsa

	Sie werden aufgefordert, die folgenden Informationen bereitzustellen:

	* Speicherort der Datei: Die Standardeinstellung ist "~/.ssh/id\\_rsa".
	* Eine Passphrase: Sie werden aufgefordert, diese erneut einzugeben.

		> [AZURE.NOTE]Es wird dringend empfohlen, dass Sie für den Schlüssel eine sichere Passphrase verwenden. Wenn Sie die Passphrase vergessen, besteht jedoch keine Möglichkeit, diese wiederherzustellen.

	Nach Abschluss des Befehls sind zwei neue Dateien verfügbar: der private Schlüssel (z. B. **id\\_rsa**) und der öffentliche Schlüssel (z. B. **id\\_rsa.pub**).

##Erstellen eines Linux-basierten HDInsight-Clusters

Wenn Sie einen Linux-basierten HDInsight-Cluster erstellen, müssen Sie den zuvor erstellten öffentlichen Schlüssel bereitstellen. Es gibt zwei Möglichkeiten, um einen HDInsight-Cluster über einen Linux-, Unix- oder OS X-Client zu erstellen:

* **Azure-Portal**: Zum Erstellen des Clusters wird ein webbasiertes Portal verwendet.

* **Azure-CLI für Mac, Linux und Windows **: Zum Erstellen des Clusters werden Befehle über die Befehlszeile eingegeben.

Jede dieser Methoden erfordert entweder ein Kennwort oder einen öffentlichen Schlüssel. Vollständige Informationen zum Erstellen eines Linux-basierten HDInsight-Clusters finden Sie unter <a href="/documentation/articles/hdinsight-hadoop-provision-linux-clusters/" target="_blank">Bereitstellen von Linux-basierten HDInsight-Clustern</a>.

###Azure-Portal

Wenn Sie das Portal verwenden, um einen Linux-basierten HDInsight-Cluster zu erstellen, müssen Sie einen **SSH-Benutzernamen** eingeben und auswählen, ob Sie ein **Kennwort** oder einen **öffentlichen SSH-Schlüssel** eingeben. Wenn Sie einen **öffentlichen SSH-Schlüssel** verwenden, müssen Sie den öffentlichen Schlüssel (in der Datei mit der Erweiterung **.pub** enthalten) in das folgenden Formular einfügen:

![Abbildung eines Formulars, das den öffentlichen Schlüssel anfordert](./media/hdinsight-hadoop-linux-use-ssh-unix/ssh-key.png)

> [AZURE.NOTE]Die Schlüsseldatei ist einfach eine Textdatei. Die Inhalte sollten in etwa wie folgt aussehen:```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCelfkjrpYHYiks4TM+r1LVsTYQ4jAXXGeOAF9Vv/KGz90pgMk3VRJk4PEUSELfXKxP3NtsVwLVPN1l09utI/tKHQ6WL3qy89WVVVLiwzL7tfJ2B08Gmcw8mC/YoieT/YG+4I4oAgPEmim+6/F9S0lU2I2CuFBX9JzauX8n1Y9kWzTARST+ERx2hysyA5ObLv97Xe4C2CQvGE01LGAXkw2ffP9vI+emUM+VeYrf0q3w/b1o/COKbFVZ2IpEcJ8G2SLlNsHWXofWhOKQRi64TMxT7LLoohD61q2aWNKdaE4oQdiuo8TGnt4zWLEPjzjIYIEIZGk00HiQD+KCB5pxoVtp user@system
> ```

Dadurch wird eine Anmeldung für den angegebenen Benutzer mit dem bereitgestellten Kennwort oder öffentlichen Schlüssel erstellt.

###Azure-Befehlszeilenschnittstelle (CLI) für Mac, Linux und Windows Azure

Sie können über die [Azure-CLI für Mac, Linux und Windows ](../xplat.md) einen neuen Cluster mithilfe des Befehls `azure hdinsight cluster create` erstellen.

Weitere Informationen zur Verwendung dieses Befehls finden Sie unter <a href="../hdinsight-hadoop-provision-linux-clusters/" target="_blank">Benutzerdefinierte Bereitstellung eines Hadoop-Clusters in HDInsight</a>.

##Verbinden mit einem Linux-basierten HDInsight-Cluster

Geben Sie in einer Terminalsitzung den SSH-Befehl ein, um die Verbindung mit dem Hauptknoten des Clusters herzustellen, indem Sie Adresse und den Benutzernamen angeben:

* **SSH-Adresse**: Der Clustername, gefolgt von **-ssh.azurehdinsight.net**. Beispiel: **mycluster-ssh.azurehdinsight.net**.

* **Benutzername**: Der SSH-Benutzername, den Sie bei der Erstellung des Clusters angegeben haben.

Im folgende Beispiel stellt der Benutzer **me** eine Verbindung mit dem Cluster **mycluster** her:

	ssh me@mycluster-ssh.azurehdinsight.net

Wenn Sie ein Kennwort für das Benutzerkonto verwendet haben, werden Sie zur Eingabe dieses Kennworts aufgefordert.

Wenn Sie einen SSH-Schlüssel verwendet haben, der mit einer Passphrase geschützt ist, werden Sie aufgefordert, die Passphrase einzugeben. Andernfalls versucht SSH automatisch, sich mithilfe eines lokalen privaten Schlüssels auf dem Client zu authentifizieren.

> [AZURE.NOTE]Wenn sich SSH nicht automatisch mit dem richtigen privaten Schlüssel authentifiziert, verwenden Sie den Parameter **-i** und geben den Pfad zum privaten Schlüssel an. Im folgenden Beispiel wird der private Schlüssel aus `~/.ssh/id_rsa` geladen:
>
> `ssh -i ~/.ssh/id_rsa me@mycluster-ssh.azurehdinsight.net`

###Herstellen einer Verbindung mit den Workerknoten

Auf die Workerknoten kann von außerhalb des Azure-Datencenters nicht direkt zugegriffen werden. Doch auf dem Hauptknoten des Clusters ist der Zugriff darauf über SSH möglich.

Wenn Sie einen SSH-Schlüssel verwenden, um Ihr Benutzerkonto zu authentifizieren, müssen Sie auf dem Client die folgenden Schritte ausführen:

1. Öffnen Sie `~/.ssh/config` in einem Text-Editor. Wenn diese Datei nicht vorhanden ist, können Sie sie durch Eingabe von `touch ~/.ssh/config` im Terminal erstellen.

2. Fügen Sie der Datei folgenden Code hinzu. Ersetzen Sie *CLUSTERNAME* durch den Namen Ihres HDInsight-Clusters.

        Host CLUSTERNAME-ssh.azurehdinsight.net
          ForwardAgent yes

    Dadurch wird die SSH-Agent-Weiterleitung für Ihren HDInsight-Cluster konfiguriert.

3. Testen Sie die SSH-Agent-Weiterleitung über die Eingabe des folgenden Befehls in das Terminal:

        echo "$SSH_AUTH_SOCK"

    Die Ausgabe dieses Befehls sollte etwa so aussehen:

        /tmp/ssh-rfSUL1ldCldQ/agent.1792

    Wenn nichts zurückgegeben wird, bedeutet dies, dass **ssh-agent** nicht ausgeführt wird. Konsultieren Sie die Betriebssystemdokumentation auf detaillierte Informationen zur Installation und Konfiguration von **ssh-Agent** oder den Artikel unter <a href="http://mah.everybody.org/docs/ssh" target="_blank">Using ssh-agent with ssh</a>.

4. Nachdem Sie sichergestellt haben, dass **ssh-Agent** ausgeführt wird, verwenden Sie folgenden Befehl, um Ihren privaten SSH-Schlüssel dem Agent hinzuzufügen:

        ssh-add ~/.ssh/id_rsa

    Wenn Ihr privater Schlüssel in einer anderen Datei gespeichert ist, ersetzen Sie `~/.ssh/id_rsa` durch den Pfad zur Datei.

Führen Sie die folgenden Schritte zum Verbinden mit den Workerknoten für Ihren Cluster aus.

> [AZURE.IMPORTANT]Wenn Sie einen SSH-Schlüssel verwenden, um Ihr Konto zu authentifizieren, müssen Sie die vorherigen Schritte ausführen, um zu bestätigen, dass die Agent-Weiterleitung funktioniert.

1. Stellen Sie, wie zuvor beschrieben, über SSH eine Verbindung mit dem HDInsight-Cluster her.

2. Sobald Sie verbunden sind, führen Sie den folgenden Befehl zum Abrufen einer Liste der Knoten im Cluster aus. Ersetzen Sie *ADMINPASSWORD* durch das Kennwort Ihres Clusteradministratorkontos. Ersetzen Sie *CLUSTERNAME* durch den Namen Ihres Clusters.

        curl --user admin:ADMINPASSWORD https://CLUSTERNAME.azurehdinsight.net/api/v1/hosts

    Dadurch werden Informationen im JSON-Format zu den Knoten im Cluster zurückgegeben, einschließlich `host_name`, die den vollqualifizierten Domänennamen (FQDN) für jeden Knoten enthalten. Es folgt ein Beispiel eines Eintrags vom Typ `host_name`, der vom Befehl **curl** zurückgegeben wird:

        "host_name" : "workernode0.workernode-0-e2f35e63355b4f15a31c460b6d4e1230.j1.internal.cloudapp.net"

3. Sobald Sie eine Liste der Workerknoten haben, mit denen Sie eine Verbindung herstellen möchten, geben Sie in der SSH-Sitzung mit dem Server den folgenden Befehl ein, um eine Verbindung mit einem Workerknoten herzustellen:

        ssh USERNAME@FQDN

    Ersetzen Sie *USERNAME* durch Ihren SSH-Benutzernamen und *FQDN* durch den vollqualifizierten Domänennamen des Workerknotens. Beispiel: `workernode0.workernode-0-e2f35e63355b4f15a31c460b6d4e1230.j1.internal.cloudapp.net`.

    > [AZURE.NOTE]Wenn Sie ein Kennwort zur Authentifizierung Ihrer SSH-Sitzung verwenden, werden Sie aufgefordert, das Kennwort erneut einzugeben. Wenn Sie einen SSH-Schlüssel verwenden, sollte die Verbindung ohne Aufforderungen fertig gestellt werden.

4. Sobald die Sitzung eingerichtet ist, ändert sich die Terminaleingabeaufforderung von `username@headnode` in `username@workernode`, um anzugeben, dass Sie mit dem Workerknoten verbunden sind. Alle Befehle, die Sie ab diesem Punkt ausführen, werden auf dem Workerknoten ausgeführt.

4. Wenn Sie mit dem Ausführen von Aktionen auf dem Workerknoten fertig sind, geben Sie den Befehl `exit` zum Schließen der Sitzung mit dem Workerknoten ein. Sie kehren zur Eingabeaufforderung `username@headnode` zurück.

##Hinzufügen weiterer Konten

1. Generieren Sie für das neue Benutzerkonto einen neuen öffentlichen Schlüssel und einen privaten Schlüssel, wie im Abschnitt [Erstellen eines SSH-Schlüssels](#create-an-ssh-key-optional) beschrieben.

	> [AZURE.NOTE]Der private Schlüssel sollte entweder auf einem Client generiert werden, den der Benutzer zum Herstellen der Verbindung mit dem Cluster verwendet, oder nach der Erstellung sicher zu einem solchen Client übertragen werden.

1. Fügen Sie in einer SSH-Sitzung mit dem Cluster den neuen Benutzer mithilfe des folgenden Befehls hinzu:

		sudo adduser --disabled-password <username>

	Dadurch wird ein neues Benutzerkonto erstellt, aber die Kennwortauthentifizierung deaktiviert.

2. Erstellen Sie das Verzeichnis und die Dateien zum Speichern des Schlüssels mit den folgenden Befehlen:

		sudo mkdir -p /home/<username>/.ssh
		sudo touch /home/<username>/.ssh/authorized_keys
		sudo nano /home/<username>/.ssh/authorized_keys

3. Wenn der Nano-Editor geöffnet wird, kopieren Sie die Inhalte des öffentlichen Schlüssels für das neue Benutzerkonto und fügen sie dann ein. Drücken Sie schließlich **STRG+X** zum Speichern der Datei, und beenden Sie den Editor.

	![Abbildung von Nano-Editor mit Beispielschlüssel](./media/hdinsight-hadoop-linux-use-ssh-unix/nano.png)

4. Verwenden Sie den folgenden Befehl, um den Ordner ".ssh" und dessen Inhalt dem neuen Benutzerkonto zuzuordnen:

		sudo chown -hR <username>:<username> /home/<username>/.ssh

5. Sie sollten jetzt in der Lage sein, sich beim Server mit dem neuen Benutzerkonto und dem privaten Schlüssel zu authentifizieren.

##<a id="tunnel"></a>SSH-Tunnel

SSH kann auch zum Tunneln lokaler Anforderungen, z. B. Webanforderungen, zum HDInsight-Cluster verwendet werden. Die Anforderung wird dann zur angeforderten Ressource weitergeleitet, als ob sie vom Stammknoten des HDInsight-Clusters stammen würde.

Dies ist besonders für den Zugriff auf webbasierte Dienste im HDInsight-Cluster hilfreich, die interne Domänennamen für den Haupt- oder Workerknoten im Cluster verwenden. Einige Abschnitte der Ambari-Webseite verwenden z. B. interne Domänennamen wie **headnode0.mycluster.d1.internal.cloudapp.net**. Diese Namen können nicht außerhalb des Clusters aufgelöst werden, doch über SSH getunnelte Anforderungen haben ihren Ursprung im Cluster und werden ordnungsgemäß aufgelöst.

Verwenden Sie die folgenden Schritte, um einen SSH-Tunnel zu erstellen und Ihren Browser zu konfigurieren, damit Sie mit ihm die Verbindung zum Cluster herstellen können.

1. Mithilfe des folgenden Befehls können Sie einen SSH-Tunnel zum Hauptknoten des Clusters herstellen:

		ssh -C2qTnNf -D 9876 username@clustername-ssh.azurehdinsight.net

	Dadurch wird eine Verbindung erstellt, über die der Datenverkehr über SSH an den lokalen Port 9876 des Clusters weitergeleitet wird. Die Optionen sind:

	* **D 8080**: Der lokale Port, der den Datenverkehr durch den Tunnel weiterleitet.

	* **C**: Alle Daten werden komprimiert, da der Webdatenverkehr hauptsächlich aus Text besteht.

	* **2**: SSH zwingen, nur Protokollversion 2 zu verwenden.

	* **q**: Stiller Modus.

	* **T**: Pseudo-TTY-Zuordnung deaktivieren, da lediglich ein Port weitergeleitet wird.

	* **n**: Verhindert den Lesevorgang für STDIN, da lediglich ein Port weitergeleitet wird.

	* **N**: Keine Remotebefehle ausführen, da lediglich ein Port weitergeleitet wird.

	* **f**: Im Hintergrund ausführen.

	Wenn Sie den Cluster mit einem SSH-Schlüssel konfiguriert haben, müssen Sie unter Umständen den Parameter `-i` verwenden und den Pfad zum privaten SSH-Schlüssel angeben.

	Nach Abschluss des Befehls wird der an den lokalen Port 9876 des lokalen Computers gesendete Datenverkehr über SSL (Secure Sockets Layer) an den Hauptknoten des Clusters weitergeleitet, der dann seinen Ursprung darstellt.

2. Konfigurieren Sie das Clientprogramm, z. B. Firefox, für die Verwendung von **localhost:9876** als **SOCKS v5**-Proxy. Die Firefox-Einstellungen sollten wie folgt aussehen:

	![Abbildung von Firefox-Einstellungen](./media/hdinsight-hadoop-linux-use-ssh-unix/socks.png)

	> [AZURE.NOTE]Durch die Auswahl von **Remote-DNS** werden DNS-Anforderungen (Domain Name System) mithilfe des HDInsight-Clusters aufgelöst. Ist diese Option deaktiviert, wird DNS lokal aufgelöst.

	Sie können überprüfen, ob Datenverkehr durch den Tunnel weitergeleitet wird, indem Sie eine Website wie z. B. <a href="http://www.whatismyip.com/" target="_blank">http://www.whatismyip.com/</a> mit aktivierten und deaktivierten Proxyeinstellungen in Firefox aufrufen. Bei aktivierten Einstellungen wird die IP-Adresse eines Computers im Microsoft Azure-Datencenter angezeigt.

###Browsererweiterungen

Obwohl Sie den Browser für die Verwendung des Tunnels konfigurieren, möchten Sie in der Regel jedoch nicht den gesamten Datenverkehr über den Tunnel weiterleiten. Browsererweiterungen wie <a href="http://getfoxyproxy.org/" target="_blank">FoxyProxy</a> unterstützen den Musterabgleich für URL-Anforderungen (nur FoxyProxy Standard oder Plus), sodass nur Anforderungen für bestimmte URLs durch den Tunnel gesendet werden.

Wenn Sie FoxyProxy Standard installiert haben, konfigurieren Sie es folgendermaßen, um nur den Datenverkehr für HDInsight über den Tunnel weiterzuleiten:

1. Öffnen Sie die FoxyProxy-Erweiterung in Ihrem Browser. Wählen Sie z. B. in Firefox das FoxyProxy-Symbol neben dem Adressfeld aus.

	![FoxyProxy (Symbol)](./media/hdinsight-hadoop-linux-use-ssh-unix/foxyproxy.png)

2. Wählen Sie **Neuen Proxy hinzufügen** aus, und geben Sie auf der Registerkarte **Allgemein** einen Proxynamen für **HDInsightProxy** ein.

	![FoxyProxy allgemein](./media/hdinsight-hadoop-linux-use-ssh-unix/foxygeneral.png)

3. Wählen Sie die Registerkarte **Proxydetails** aus, und geben Sie die entsprechenden Werte in die Felder ein:

	* **Host- oder IP-Adresse**: "localhost", da wir auf dem lokalen Computer einen SSH-Tunnel verwenden.

	* **Port**: Der Port, den Sie für den SSH-Tunnel verwendet haben.

	* **SOCKS-Proxy**: Aktivieren Sie diese Option, damit der Browser den Tunnel als Proxy verwenden kann.

	* **SOCKS v5**: Aktivieren Sie diese Option, um die erforderliche Version für den Proxy festzulegen.

	![FoxyProxy-Proxy](./media/hdinsight-hadoop-linux-use-ssh-unix/foxyproxyproxy.png)

4. Wählen Sie die Registerkarte **URL-Muster** und anschließend **Neues Muster** aus. Gehen Sie wie folgt vor, um das Muster zu definieren, und klicken Sie dann auf **OK**:

	* **Name des Musters**: **Hauptknoten** – Dies ist lediglich ein Anzeigename für das Muster.

	* **URL-Muster**: **\\*Hauptknoten\\*** – Definiert ein Muster, das mit allen URLs übereinstimmt, die das Wort **Hauptknoten** enthalten.

	![FoxyProxy-Muster](./media/hdinsight-hadoop-linux-use-ssh-unix/foxypattern.png)

4. Klicken Sie auf **OK**, um den Proxy hinzuzufügen und **Proxyeinstellungen** zu schließen.

5. Am oberen Rand des FoxyProxy-Dialogfelds ändern Sie **Modus auswählen** in **Proxys basierend auf ihren vordefinierten Mustern und Prioritäten verwenden**, und klicken Sie dann auf **Schließen**.

	![FoxyProxy – Modus auswählen](./media/hdinsight-hadoop-linux-use-ssh-unix/selectmode.png)

Nachdem Sie diese Schritte ausgeführt haben, werden ausschließlich Anforderungen für URLs, die die Zeichenfolge **Hauptknoten** enthalten, über den SSL-Tunnel weitergeleitet.

##Nächste Schritte

Nachdem Sie jetzt wissen, wie die Authentifizierung mithilfe eines SSH-Schlüssels erfolgt, erfahren Sie, wie Sie MapReduce mit Hadoop für HDInsight verwenden.

* [Verwenden von Hive mit HDInsight](hdinsight-use-hive.md)

* [Verwenden von Pig mit HDInsight](hdinsight-use-pig.md)

* [Verwenden von MapReduce-Aufträgen mit HDInsight](hdinsight-use-mapreduce.md)
 

<!---HONumber=62-->