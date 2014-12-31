<properties title="Verwenden von Gruppen mit Lastenausgleich zum Gruppieren von MySQL auf Linux" pageTitle="Verwenden von Gruppen mit Lastenausgleich zum Gruppieren von MySQL auf Linux" description="Ein Artikel, der Muster zum Einrichten eines hoch verf&uuml;gbaren Linux-Clusters mit Lastenausgleich auf Azure mit MySQL als ein Beispiel veranschaulicht." metaKeywords="mysql, linux, cluster, azure, ha, high availability, corosync, pacemaker, drbd, heartbeat" services="virtual-machines" solutions="" documentationCenter="" authors="jparrel" videoId="" scriptId="" manager="timlt" />

<tags ms.service="virtual-machines" ms.workload="infrastructure-services" ms.tgt_pltfrm="vm-linux" ms.devlang="na" ms.topic="article" ms.date="01/01/1900" ms.author="jparrel" />

# Verwenden von Gruppen mit Lastenausgleich zum Gruppieren von MySQL auf Linux

-   [Vorbereitung][Vorbereitung]
-   [Einrichten des Clusters][Einrichten des Clusters]
-   [Einrichten von MySQL][Einrichten von MySQL]
-   [Einrichten von Corosync][Einrichten von Corosync]
-   [Einrichten von Pacemaker][Einrichten von Pacemaker]
-   [Testen][Testen]
-   [STONITH][STONITH]
-   [Einschränkungen][Einschränkungen]

## Einführung

Der Zweck dieses Artikels besteht darin, die unterschiedlichen verfügbaren Ansätze für die Bereitstellung hoch verfügbarer Linux-basierter Dienste auf Microsoft Azure zu untersuchen und zu veranschaulichen, wobei primär die hohe Verfügbarkeit von MySQL Server untersucht wird. Ein Video, das diesen Ansatz veranschaulicht, steht unter [Channel 9][Channel 9] zur Verfügung.

Wir beschreiben eine auf DRBD, Corosync und Pacemaker basierende hoch verfügbare Einzelmaster-MYSQL-Lösung mit zwei Knoten, die nichts gemeinsam nutzen. Nur auf einem Knoten wird jeweils MySQL ausgeführt. Das Lesen und Schreiben von der DRBD-Ressource wird auch nur jeweils auf einen Knoten begrenzt.

Es besteht keine Notwendigkeit für eine VIP-Lösung wie LVS, da wir die Gruppen mit Lastenausgleich von Microsoft Azure verwenden, um eine Roundrobin-Funktionalität und die Endpunktermittlung sowie die Entfernung und die normale Wiederherstellung der VIP bereitzustellen. Die VIP ist eine globale routfähige IPv4-Adresse, die beim ersten Erstellen des Cloud-Diensts durch Microsoft Azure zugewiesen wird.

Es gibt weitere mögliche Architekturen für MySQL, einschließlich NBD Cluster, Percona und Galera, sowie verschiedene Middleware-Lösungen, worunter mindestens eine als ein virtueller Computer auf [VM Depot][VM Depot] verfügbar ist. Solange diese Lösungen auf Unicast gegenüber Multicast repliziert werden können oder Übertragungen vornehmen und nicht auf dem gemeinsamen Speicher oder mehreren Netzwerkschnittstellen aufbauen, sollten die Szenarien auf Microsoft Azure einfach bereitzustellen sein.

Diese Clusteringarchitekturen können natürlich auf andere Produkte wie PostgreSQL und OpenLDAP sowie ähnliche erweitert werden. Diese Lastenausgleichsprozedur ohne gemeinsam genutzte Inhalte wurde beispielsweise erfolgreich mit dem Multimaster-OpenLDAP getestet. Und Sie können sie in unserem Channel 9-Blog ansehen.

## Vorbereitung

Sie benötigen ein Microsoft Azure-Konto mit einem gültigen Abonnement, das in der Lage ist, mindestens zwei (2) virtuelle Computer (in diesem Beispiel wurde XS verwendet), ein Netzwerk, ein Subnetz, eine Affinitätsgruppe und eine Verfügbarkeitsgruppe zu erstellen. Zudem muss es das Erstellen neuer VHDs in derselben Region wie der Cloud-Dienst ermöglichen und sie an die virtuellen Linux-Computer anfügen können.

### Getestete Umgebung

-   Ubuntu 13.10
-   DRBD
-   MySQL Server
-   Corosync und Pacemaker

### Affinitätsgruppe

Eine Affinitätsgruppe wird für die Lösung erstellt, indem Sie sich beim Azure-Portal anmelden, einen Bildlauf zu „Einstellungen“ ausführen und eine neue Affinitätsgruppe erstellen. Später zugeordnete Ressourcen werden zu dieser Affinitätsgruppe zugewiesen.

### Netzwerke

Ein neues Netzwerk wird erstellt, und ein Subnetz wird im Netzwerk erstellt. Wir entscheiden uns für ein „10.10.10.0/24“-Netzwerk mit nur einem immanenten „/24“-Subnetz.

### Virtuelle Computer

Der erste virtuelle Computer unter Ubuntu 13.10 wird mithilfe eines bestätigten Ubuntu Gallery-Images erstellt und heißt `hadb01`. Während des Vorgangs wird der neue Cloud-Dienst „hadb“ erstellt. Wir haben ihn so benannt, um seine freigegebene Art mit Lastenausgleich zu veranschaulichen, die der Dienst aufweist, wenn wir weitere Ressourcen hinzufügen. Das Erstellen von `hadb01` ist ereignisarm und wird mithilfe des Portals fertig gestellt. Für SSH wird automatisch ein Endpunkt erstellt, und unser erstelltes Netzwerk wird ausgewählt. Wir entscheiden uns zudem für das Erstellen einer neuen Verfügbarkeitsgruppe für die virtuellen Computer.

Nachdem der erste virtuelle Computer erstellt wurde (im Prinzip während der Erstellung des Cloud-Diensts), fahren wir mit dem Erstellen des zweiten virtuellen Computers `hadb02` fort. Beim zweiten virtuellen Computer verwenden wir ebenfalls den virtuellen Computer unter Ubuntu 13.10 aus dem Katalog mithilfe des Portals, wir wählen jedoch den vorhandenen Cloud-Dienst `hadb.cloudapp.net` aus, anstelle einen neuen zu erstellen. Das Netzwerk und die Verfügbarkeitsgruppe sollten automatisch ausgewählt sein. Es wird auch ein SSH-Endpunkt erstellt.

Nachdem beide virtuellen Computer erstellt wurden, nehmen wir den SSH-Port für `hadb01` (TCP 22) und `hadb02` (automatisch durch Azure zugewiesen) zur Kenntnis.

### Attached Storage

Wir fügen an beide virtuellen Computer einen neuen Datenträger an, und wir erstellen neue 5-GB-Datenträger in diesem Prozess. Die Datenträger werden im VHD-Container gehostet, der für unsere Datenträger für das Hauptbetriebssystem verwendet wird. Nach dem Erstellen und Anfügen der Datenträger besteht keine Notwendigkeit, Linux neu zu starten, da der Kernel das neue Gerät (für gewöhnlich `/dev/sdc`, Sie können `dmesg` für die Ausgabe prüfen) erkennt.

Auf jedem virtuellen Computer fahren wir mit der Erstellung einer neuen Partition mithilfe von `cfdisk` (primär, Linux-Partition) fort und schreiben die neue Partitionstabelle. **Erstellen Sie kein Dateisystem auf dieser Partition**.

## Einrichten des Clusters

In beiden virtuellen Ubuntu-Computern müssen wir APT zum Installieren von Corosync, Pacemaker und DRBD verwenden. Verwenden von `apt-get`:

    sudo apt-get install corosync pacemaker drbd8utils.

**Installieren Sie MySQL nicht zu diesem Zeitpunkt**. Debian- und Ubuntu-Installationsskripts initialisieren ein MySQL-Datenverzeichnis auf `/var/lib/mysql`. Da das Verzeichnis jedoch durch ein DRBD-Dateisystem abgelöst wird, müssen wir dies später vornehmen.

An dieser Stelle sollten wir zudem (mit `/sbin/ifconfig`) sicherstellen, dass beide virtuellen Computer Adressen im Subnetz „10.10.10.0/24“ verwenden und dass sie sich durch den Namen untereinander pingen können. Bei Bedarf können Sie auch `ssh-keygen` und `ssh-copy-id` verwenden, um sicherzustellen, dass beide virtuellen Computer per SSH kommunizieren können, ohne dass dafür ein Kennwort nötig ist.

### Einrichten von DRBD

Wir erstellen eine DRBD-Ressource, die die zugrunde liegende `/dev/sdc1`-Partition zum Generieren einer `/dev/drbd1`-Ressource verwendet, die mithilfe von ext3 formatiert werden kann und sowohl in primären als auch sekundären Knoten verwendet werden kann. Öffnen Sie dafür `/etc/drbd.d/r0.res`, und kopieren Sie die folgende Ressourcendefinition. Nehmen Sie die folgenden Punkte in beiden virtuellen Computern vor:

    resource r0 {
      on `hadb01` {
        device  /dev/drbd1;
        disk   /dev/sdc1;
        address  10.10.10.4:7789;
        meta-disk internal;
      }
      on `hadb02` {
        device  /dev/drbd1;
        disk   /dev/sdc1;
        address  10.10.10.5:7789;
        meta-disk internal;
      }
    }

Initialisieren Sie anschließend die Ressource mit `drbdadm` in beiden virtuellen Computern:

    sudo drbdadm -c /etc/drbd.conf role r0
    sudo drbdadm up r0

Erzwingen Sie abschließend auf dem primären (`hadb01`) den Besitz (primär) der DRBD-Ressource:

    sudo drbdadm primary --force r0

Wenn Sie die Inhalte von „/proc/drbd“ (`sudo cat /proc/drbd`) auf beiden virtuellen Computern prüfen, sollten Sie `Primary/Secondary` auf `hadb01` und `Secondary/Primary` auf `hadb02` sehen, wobei die Konsistenz mit der Lösung zu diesem Zeitpunkt gegeben ist. Der 5-GB-Datenträger wird über das „10.10.10.0/24“-Netzwerk synchronisiert, was für die Kunden kostenlos ist.

Nach der Synchronisierung des Datenträgers können Sie das Dateisystem auf `hadb01` erstellen. Für Testzwecke haben wir ext2 verwendet. Mit der folgenden Anweisung wird jedoch ein ext3-Dateisystem erstellt:

    mkfs.ext3 /dev/drbd1

### Bereitstellen der DRBD-Ressource

Auf `hadb01` können wir nun die DRBD-Ressourcen bereitstellen. Debian und Ableitungen verwenden `/var/lib/mysql` als das MySQL-Datenverzeichnis. Da wir MySQL nicht installiert haben, erstellen wir das Verzeichnis und stellen die DRBD-Ressource bereit. Auf `hadb01`:

    sudo mkdir /var/lib/mysql
    sudo mount /dev/drbd1 /var/lib/mysql

## Einrichten von MySQL

Nun sind Sie bereit, MySQL auf `hadb01` zu installieren:

    sudo apt-get install mysql-server

Für `hadb02` haben Sie zwei Möglichkeiten. Sie können nun „mysql-server“ installieren, wodurch „/var/lib/mysql“ erstellt und mit einem neuen Datenverzeichnis aufgefüllt wird, und anschließend mit dem Entfernen der Inhalte fortfahren. Auf `hadb02`:

    sudo apt-get install mysql-server
    sudo service mysql stop
    sudo rm –rf /var/lib/mysql/*

Die zweite Option besteht darin, ein Failover zu `hadb02` auszuführen und dann dort „mysql-Server“ (Installationsskripts berücksichtigen die vorhandene Installation und lassen sie unverändert) zu installieren.

Auf `hadb01`:

    sudo drbdadm secondary –force r0

Auf `hadb02`:

    sudo drbdadm primary –force r0
    sudo apt-get install mysql-server

Wenn Sie zu diesem Zeitpunkt kein Failover von DRBD planen, ist die erste Option einfacher, aber wohl weniger elegant. Nachdem Sie dies eingerichtet haben, können Sie mit der Arbeit in Ihrer MySQL-Datenbank beginnen. Auf `hadb02` (oder auf dem gemäß DRBD gerade aktiven Server):

    mysql –u root –p
    CREATE DATABASE azureha;
    CREATE TABLE things ( id SERIAL, name VARCHAR(255) );
    INSERT INTO things VALUES (1, "Yet another entity");
    GRANT ALL ON things.\* TO root;

**Warnung**: Diese letzte Anweisung deaktiviert die Authentifizierung effektiv für den Stammbenutzer in dieser Tabelle. Diese sollte durch Ihre produktionsfähigen GRANT-Anweisungen ersetzt werden und dient nur zur Veranschaulichung.

Sie müssen den Netzwerkbetrieb für MySQL aktivieren, wenn Sie Abfragen von außerhalb der virtuellen Computer vornehmen möchten, worin der Zweck dieses Leitfadens besteht. Öffnen Sie auf beiden virtuellen Computern `/etc/mysql/my.cnf`, und wechseln Sie zu `bind-address`. Ändern Sie sie von „127.0.0.1“ zu „0.0.0.0“. Stellen Sie nach dem Speichern der Datei einen `sudo service mysql restart` auf Ihrem aktuellen primären Computer aus.

### Erstellen der MySQL-Gruppe mit Lastenausgleich

Wir wechseln zurück zum Azure-Portal und navigieren zum virtuellen Computer `hadb01` und dann zu „Endpunkte“. Wir erstellen einen neuen Endpunkt, wählen „MySQL (TCP 3306)“ aus dem Dropdownmenü aus und aktivieren das Kontrollkästchen *Neue Gruppe mit Lastenausgleich erstellen*. Wir nennen unseren Endpunkt mit Lastenausgleich `lb-mysql`. Wir ändern die meisten Optionen nicht, und zwar mit Ausnahme der Zeit, die wir auf „5“ (Sekunden, Minimum) reduzieren.

Nach der Erstellung des Endpunkts wechseln wir zu `hadb02` und zu „Endpunkte“ und erstellen einen neuen Endpunkt, wir wählen jedoch `lb-mysql` und dann „MySQL“ aus dem Dropdownmenü aus. Sie können auch die Azure-Befehlszeilenschnittstelle für diesen Schritt verwenden.

Nun haben wir alles, was für eine manuelle Verarbeitung des Clusters benötigen.

### Testen der Gruppe mit Lastenausgleich

Tests können über einen Computer ausgeführt werden, der sich außerhalb befindet, indem ein beliebiger MySQL-Client und Anwendungen verwendet werden (beispielsweise wenn phpMyAdmin als eine Azure-Website ausgeführt wird). In diesem Fall haben wir das MySQL-Befehlszeilentool auf einem anderen Linux-Feld verwendet:

    mysql azureha –u root –h hadb.cloudapp.net –e "select * from things;"

### Manuelles Failover

Sie können nun Failover simulieren, indem Sie MySQL herunterfahren, den primären Computer von DRBD wechseln und MySQL erneut starten.

Auf hadb01:

    service mysql stop && umount /var/lib/mysql ; drbdadm secondary r0

Dann auf hadb02:

    drbdadm primary r0 ; mount /dev/drbd1 /var/lib/mysql && service mysql start

Nachdem Sie das manuelle Failover vorgenommen haben, können Sie Ihre Remoteabfrage wiederholen, und es sollte perfekt funktionieren.

## Einrichten von Corosync

Corosync ist die zugrunde liegende Clusterinfrastruktur, die erforderlich ist, damit Pacemaker funktioniert. Für Heartbeat v1- und -v2-Benutzer (und andere Methoden wie Ultramonkey) bietet Corosync einen Teil der CRM-Funktionalitäten, während Pacemaker in puncto Funktionalität Heartbeart ähnlicher ist.

Die Haupteinschränkung für Corosync auf Azure besteht darin, dass Corosync Multicast- über Übertragungs- über Unicast-Kommunikationen bevorzugt. Das Microsoft Azure-Netzwerk unterstützt jedoch nur Unicast.

Zum Glück verfügt Corosync über einen funktionierenden Unicast-Modus, und die einzige wirkliche Einschränkung besteht darin, da nicht alle Knoten miteinander *automatisch* kommunizieren, dass Sie die Knoten in Ihren Konfigurationsdateien, einschließlich ihrer IP-Adressen, definieren müssen. Wir können die Corosync-Beispieldateien für Unicast verwenden und einfach die BIND-Adresse, Knotenlisten und das Protokollierungsverzeichnis (Ubuntu verwendet `/var/log/corosync`, während in der Beispieldatei `/var/log/cluster` verwendet wird) ändern sowie Quorum-Tools aktivieren.

**Beachten Sie die `transport: udpu`-Anweisung unten sowie die manuell definierten IP-Adressen für die Knoten**.

Auf `/etc/corosync/corosync.conf` für beide Knoten:

    totem {
      version: 2
      crypto_cipher: none
      crypto_hash: none
      interface {
        ringnumber: 0
        bindnetaddr: 10.10.10.0
        mcastport: 5405
        ttl: 1
      }
      transport: udpu
    }

    logging {
      fileline: off
      to_logfile: yes
      to_syslog: yes
      logfile: /var/log/corosync/corosync.log
      debug: off
      timestamp: on
      logger_subsys {
        subsys: QUORUM
        debug: off
        }
      }

    nodelist {
      node {
        ring0_addr: 10.10.10.4
        nodeid: 1
      }

      node {
        ring0_addr: 10.10.10.5
        nodeid: 2
      }
    }

    quorum {
      provider: corosync_votequorum
    }

Wir kopieren diese Konfigurationsdatei auf beide virtuellen Computer und starten Corosync auf beiden Knoten:

    sudo service start corosync

Kurz nach dem Start des Diensts sollte der Cluster im aktuellen Ring hergestellt sein, und das Quorum sollte errichtet sein. Wir können diese Funktionalität prüfen, indem wir die entsprechende Protokolle anzeigen, oder durch:

    sudo corosync-quorumtool –l

Eine Ausgabe, die dem Bild unten ähnelt, sollte diesem Prinzip folgen:

![corosync-quorumtool -l sample output][corosync-quorumtool -l sample output]

## Einrichten von Pacemaker

Pacemaker verwendet den Cluster zum Überwachen für Ressourcen, zum Definieren, wann primäre Computer heruntergefahren werden, und zum Wechseln dieser Ressourcen zu sekundären Computern. Ressourcen können unter anderem über eine Reihe verfügbarer Skripts oder über (init-ähnliche) LSB-Skripts definiert werden.

Wir möchten, dass Pacemaker die DRBD-Ressource, den Bereitstellungspunkt und den MySQL-Dienst „besitzt“. Wenn Pacemaker DRBD aktivieren und deaktivieren, bereitstellen und die Bereitstellung von DRBD aufheben sowie MySQL in der richtigen Reihenfolge starten/anhalten kann, wenn ein Fehler beim primären Computer vorliegt, ist unsere Einrichtung abgeschlossen.

Beim ersten Installieren von Pacemaker sollte Ihre Konfiguration einfach genug sein, und zwar in etwa so:

    node $id="1" hadb01
      attributes standby="off"
    node $id="2" hadb02
      attributes standby="off"

Nehmen Sie eine Überprüfung durch Ausführen von `sudo crm configure show` vor. Erstellen Sie nun eine Datei (wie `/tmp/cluster.conf`) mit den folgenden Ressourcen:

    primitive drbd_mysql ocf:linbit:drbd \
          params drbd_resource="r0" \
          op monitor interval="29s" role="Master" \
          op monitor interval="31s" role="Slave"

    ms ms_drbd_mysql drbd_mysql \
          meta master-max="1" master-node-max="1" \
            clone-max="2" clone-node-max="1" \
            notify="true"

    primitive fs_mysql ocf:heartbeat:Filesystem \
          params device="/dev/drbd/by-res/r0" \
          directory="/var/lib/mysql" fstype="ext3"

    primitive mysqld lsb:mysql

    group mysql fs_mysql mysqld

    colocation mysql_on_drbd \
           inf: mysql ms_drbd_mysql:Master

    order mysql_after_drbd \
           inf: ms_drbd_mysql:promote mysql:start

    property stonith-enabled=false

    property no-quorum-policy=ignore

Laden Sie sie nun in die Konfiguration (dies ist nur in einem Knoten erforderlich):

    sudo crm configure
      load update /tmp/cluster.conf
      commit
      exit

Stellen Sie ferner sicher, dass Pacemaker beim Booten in beiden Knoten startet:

    sudo update-rc.d pacemaker defaults

Stellen Sie nach einigen Sekunden und bei Verwendung von `sudo crm_mon –L` sicher, dass einer Ihrer Knoten zum Master für den Cluster avanciert ist und alle Ressourcen ausführt. Sie können „mount“ und „ps“ verwenden, um zu überprüfen, ob die Ressourcen ausgeführt werden.

Der folgende Screenshot zeigt `crm_mon` mit einem angehaltenen Knoten (Beenden mittels BEFEHLSTASTE-C).

![crm\_mon node stopped][crm\_mon node stopped]

Und dieser Screenshot zeigt beide Knoten, einen Master- und einen Detailknoten:

![crm\_mon operational master/slave][crm\_mon operational master/slave]

## Testen

Nun sind wir bereit für eine automatische Failoversimulation. Dies kann auf zwei Arten erreicht werden: schwach und hart. Die schwache Variante verwendet die Funktion zum Herunterfahren des Clusters: `` crm_standby -U `uname -n` -v on ``. Wenn diese auf dem Master verwendet wird, übernimmt der Detailknoten. Denken Sie daran, dies wieder auf „off“ zurückzusetzen (ansonsten informiert Sie „crm\_mon“, dass sich ein Knoten im Standbymodus befindet).

Bei der harten Variante wird der primäre virtuelle Computer (hadb01) über das Portal heruntergefahren, oder es wird die Ausführungsebene des virtuellen Computers geändert (d. h. anhalten, herunterfahren), dann helfen wir Corosync und Pacemaker, indem wir signalisieren, dass der Master herunterfährt. Sie können dies testen (nützlich für Wartungsfenster). Wir können das Szenario jedoch auch erzwingen, indem wir einfach den virtuellen Computer einfrieren.

## STONITH

Es sollte möglich sein, das Herunterfahren eines virtuellen Computers über die Azure-Befehlszeilentools für Linux auszustellen anstelle eines STONITH-Skripts, das ein physisches Gerät steuert. Sie können `/usr/lib/stonith/plugins/external/ssh` als Basis verwenden und STONITH in der Konfiguration des Clusters aktivieren. Die Azure-Befehlszeilenschnittstelle sollte global installiert sein, und die Veröffentlichungseinstellungen/-profile sollten für den Benutzer des Clusters geladen sein.

Beispielcode für die Ressourcen finden Sie auf [GitHub][GitHub]. Sie müssen die Konfiguration des Clusters ändern, indem Sie `sudo crm configure` Folgendes hinzufügen:

    primitive st-azure stonith:external/azure \
      params hostlist="hadb01 hadb02" \
      clone fencing st-azure \
      property stonith-enabled=true \
      commit

**Hinweis:** Das Skript führt keine Überprüfungen nach oben/unten aus. Die ursprüngliche SSH-Ressource wies 15 Pingüberprüfungen auf, die Wiederherstellungszeit für einen virtuellen Azure-Computer wird möglicherweise flexibler.

## Einschränkungen

Es gelten die folgenden Einschränkungen:

-   Das linbit DRBD-Ressourcenskript, das DRBD als eine Ressource in Pacemaker verwaltet, verwendet `drbdadm down` beim Herunterfahren eines Knotens, selbst wenn der Knoten gerade in den Standbymodus wechselt. Dies ist nicht optimal, da der Detailknoten die DRBD-Ressource nicht synchronisiert, während der Master Schreibvorgänge abruft. Wenn der Master keinen normalen Fehler aufweist, kann der Detailknoten einen älteren Dateisystemstatus übernehmen. Zur Behebung dieses Problems stehen zwei potenzielle Möglichkeiten zur Verfügung:
-   Das Erzwingen eines `drbdadm up r0` in allen Clusterknoten über einen lokalen (nicht gruppierten) Watchdog oder
-   Bearbeiten des linbit DRBD-Skripts, wodurch sichergestellt wird, dass `down` nicht in `/usr/lib/ocf/resource.d/linbit/drbd` aufgerufen wird.
-   Der Lastenausgleich benötigt mindestens 5 Sekunden, um zu antworten. Anwendungen sollten daher clusterfähig sein und in Bezug auf eine Zeitüberschreitung toleranter sein. Andere Architekturen können auch hilfreich sein, beispielsweise In-App-Warteschlangen, Middleware-Abfragen usw.
-   Die MySQL-Feinabstimmung ist erforderlich, um sicherzustellen, dass der Schreibvorgang in einer vernünftigen Geschwindigkeit erfolgt und Zwischenspeicherungen möglichst häufig auf den Datenträger übertragen werden, um Speicherverluste zu vermeiden.
-   Die Schreibleistung hängt vom Interconnect des virtuellen Computers im virtuellen Switch ab, da es sich hierbei um den Mechanismus handelt, der durch DRBD zum Replizieren des Geräts verwendet wird.

  [Vorbereitung]: #getting-ready
  [Einrichten des Clusters]: #setting-up-the-cluster
  [Einrichten von MySQL]: #setting-up-mysql
  [Einrichten von Corosync]: #setting-up-corosync
  [Einrichten von Pacemaker]: #setting-up-pacemaker
  [Testen]: #testing
  [STONITH]: #stonith
  [Einschränkungen]: #limitations
  [Channel 9]: http://channel9.msdn.com/Blogs/Open/Load-balancing-highly-available-Linux-services-on-Windows-Azure-OpenLDAP-and-MySQL
  [VM Depot]: http://vmdepot.msopentech.com
  [corosync-quorumtool -l sample output]: media/virtual-machines-linux-mysql-cluster/image001.png
  [crm\_mon node stopped]: media/virtual-machines-linux-mysql-cluster/image002.png
  [crm\_mon operational master/slave]: media/virtual-machines-linux-mysql-cluster/image003.png
  [GitHub]: https://github.com/bureado/aztonith