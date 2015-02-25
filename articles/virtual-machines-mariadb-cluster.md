<properties pageTitle="Ausführen eines MariaDB (MySQL)-Clusters in Azure" description="Erstellen eines MariaDB + Galera MySQL-Clusters auf virtuellen Azure-Computern" services="virtual-machines" documentationCenter="" authors="sabbour" manager="timlt" editor=""/>

<tags ms.service="virtual-machines" ms.devlang="multiple" ms.topic="article" ms.tgt_pltfrm="vm-linux" ms.workload="infrastructure-services" ms.date="12/3/2014" ms.author="v-ahsab" ms.prod="azure"/>

<!--The next line, with one pound sign at the beginning, is the page title--> 
# MariaDB (MySQL)-Cluster - Azure-Lernprogramm

<p>Wir erstellen einen Multimaster-[Galera](http://galeracluster.com/products/)-Cluster von [MariaDBs](https://mariadb.org/en/about/), ein robuster, skalierbarer und zuverlässiger Ersatz für MySQL für das Arbeiten in einer hochverfügbaren Umgebung auf virtuellen Azure-Computern.</p>

<!--Table of contents for topic, the words in brackets must match the heading wording exactly-->

+ [Übersicht über die Architektur]
+ [Erstellen der Vorlage]
+ [Erstellen des Clusters]
+ [Lastenausgleich für den Cluster]
+ [Überprüfen des Clusters]


## Übersicht über die Architektur

Dieses Thema umfasst die folgenden Schritte:

1. Erstellen eines Clusters mit 3 Knoten
2. Trennen der Datenträger vom Betriebssystemdatenträger
3. Erstellen der Datenträger in der Einstellung "RAID 0 (Stripeset)", um die IOPS zu erhöhen
4. Verwenden des Azure-Lastenausgleichs, um die Last der drei Knoten auszugleichen
5. Um sich wiederholende Aufgaben zu minimieren, erstellen Sie ein Abbild des virtuellen Computers mit MariaDB+Galera, und verwenden Sie dieses, um die anderen Cluster-VMs zu erstellen.

![Architecture](./media/virtual-machines-mariadb-cluster/Setup.png)

> [AZURE.NOTE]  In diesem Thema werden die Tools der [Azure-Befehlszeilenschnittstelle]-Tools verwendet, laden Sie diese daher herunter, und verbinden Sie sie gemäß den Anweisungen mit Ihrem Azure-Abonnement. Wenn Sie einen Verweis auf die Befehle in der Azure-Befehlszeilenschnittstelle benötigen, sehen Sie sich diesen Link für die [Befehlsreferenz der Azure-Befehlszeilenschnittstelle] an. Sie müssen auch einen [SSH-Schlüssel für die Authentifizierung] erstellen und sich den Speicherort der **PEM-Datei** notieren.


## Erstellen der Vorlage

### Infrastruktur

1. Erstellen Sie eine Affinitätsgruppe, um die Ressourcen zusammenzuhalten.

		azure account affinity-group create mariadbcluster --location "North Europe" --label "MariaDB Cluster"
        
2. Erstellen eines virtuellen Netzwerks

		azure network vnet create --address-space 10.0.0.0 --cidr 8 --subnet-name mariadb --subnet-start-ip 10.0.0.0 --subnet-cidr 24 --affinity-group mariadbcluster mariadbvnet

3. Erstellen Sie ein Speicherkonto, um alle Datenträger zu hosten. Beachten Sie, dass Sie nicht mehr als 40 häufig verwendete Datenträger in dem gleichen Speicherkonto platzieren dürfen, um die Begrenzung des Speicherkontos von 20.000 IOPS nicht zu überschreiten. In diesem Fall sind wir weit von dieser Zahl entfernt, sodass aus Gründen der Einfachheit alles im selben Konto gespeichert werden kann.

		azure storage account create mariadbstorage --label mariadbstorage --affinity-group mariadbcluster

3. Suchen Sie den Namen des CentOS 7-Abbilds des virtuellen Computers.

		azure vm image list | findstr CentOS        
Die Ausgabe sieht in etwa wie folgt aus: `5112500ae3b842c8b9c604889f8753c3__OpenLogic-CentOS-70-20140926`. Verwenden Sie den Namen im nächsten Schritt.

4. Erstellen Sie die Vorlage des virtuellen Computers, und ersetzen Sie dabei **/path/to/key.pem** durch den Pfad, unter dem Sie den generierten PEM-SSH-Schlüssel gespeichert haben.

		azure vm create --virtual-network-name mariadbvnet --subnet-names mariadb --blob-url "http://mariadbstorage.blob.core.windows.net/vhds/mariadbhatemplate-os.vhd"  --vm-size Medium --ssh 22 --ssh-cert "/path/to/key.pem" --no-ssh-password mariadbtemplate 5112500ae3b842c8b9c604889f8753c3__OpenLogic-CentOS-70-20140926 azureuser
        
5. Fügen Sie vier 500-GB-Datenträger an den virtuellen Computer an, die für die RAID-Konfiguration verwendet werden können.

		FOR /L %d IN (1,1,4) DO azure vm disk attach-new mariadbhatemplate 512 http://mariadbstorage.blob.core.windows.net/vhds/mariadbhatemplate-data-%d.vhd        

6. Kopieren Sie den SSH-Schlüssel in die Vorlage des virtuellen Computers, die Sie unter **mariadbhatemplate.cloudapp.net:22** erstellt haben, und stellen Sie mit ihren privaten Schlüssel eine Verbindung her.

### Software

1. Rufen Sie den Stamm ab.

        sudo su
        
2. Installieren Sie die RAID-Unterstützung:

     - Installieren von mdadm
        
        		yum install mdadm
                
     - Erstellen der RAID0/Stripe-Konfiguration mit einem EXT4-Dateisystem
        
				mdadm --create --verbose /dev/md0 --level=stripe --raid-devices=4 /dev/sdc /dev/sdd /dev/sde /dev/sdf
				mdadm --detail --scan >> /etc/mdadm.conf
				mkfs -t ext4 /dev/md0
         
     - Erstellen des Bereitstellungspunktverzeichnisses
         
				mkdir /mnt/data
                
     - Abrufen der UUID des neu erstellen RAID-Geräts
        
				blkid | grep /dev/md0
    
     - Bearbeiten von "/etc/fstab"
        
        		VI /etc/fstab
        
     - Fügen Sie das Gerät dort hinzu, um die automatische Bereitstellung beim Neustarten zu aktivieren, und ersetzen Sie dabei die UUID durch den Wert, den Sie vorher mit dem Befehl **blkid** erhalten haben.
        
        		UUID=<UUID FROM PREVIOUS>   /mnt/data ext4   defaults,noatime   1 2
        	
     - Bereitstellen der neuen Partition
        
        		mount /mnt/data
            
3. Installieren Sie MariaDB:    
		
     - Erstellen Sie die Datei "MariaDB.repo". 
        
              	vi /etc/yum.repos.d/MariaDB.repo
        
     - Setzen Sie die folgenden Inhalte ein.      
        
				[mariadb]
				name = MariaDB
				baseurl = http://yum.mariadb.org/10.0/centos7-amd64
				gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
				gpgcheck=1
    
     - Entfernen Sie vorhandene Postfix- und MariaDB-Bibliotheken, um Konflikte zu vermeiden.
    
    		yum remove postfix mariadb-libs-*
            
     - Installieren von MariaDB mit Galera
    
    		yum install MariaDB-Galera-server MariaDB-client galera
   
4. Verschieben Sie das MySQL-Datenverzeichnis in das RAID-Blockgerät.

     - Kopieren Sie das aktuelle MySQL-Verzeichnis an den neuen Speicherort und entfernen Sie das alte Verzeichnis.
    
    		cp -avr /var/lib/mysql /mnt/data  
    		rm -rf /var/lib/mysql
      
     - Legen Sie die Berechtigungen für das neue Verzeichnis entsprechend fest.
        
        	chown -R mysql:mysql /mnt/data && chmod -R 755 /mnt/data/  
            
     - Erstellen Sie einen Symlink, der vom alten Verzeichnis auf den neuen Speicherort auf der RAID-Partition verweist.
    
    		ln -s /mnt/data/mysql /var/lib/mysql   

5. Da [SELinux die Clustervorgänge beeinträchtigt](http://galeracluster.com/documentation-webpages/configuration.html#selinux), müssen Sie es für die aktuelle Sitzung deaktivieren (bis eine kompatible Version vorhanden ist). Bearbeiten Sie  `/etc/selinux/config`, um es für weitere Neustarts zu deaktivieren:
    	
	        setenforce 0
    
       Anschließend wird `/etc/selinux/config` so bearbeitet, dass `SELINUX=permissive`
       
6. Validieren Sie die MySQL-Ausführungen.

    - Starten Sie MySQL.
    
    		service mysql start
            
    - Sichern Sie die MySQL-Installation, legen Sie das Stammkennwort fest, entfernen Sie anonyme Benutzer, deaktivieren Sie dazu die Remote-Stammanmeldung und entfernen Sie die Testdatenbank.
    		
            mysql_secure_installation
            
    - Erstellen Sie einen Benutzer für die Datenbank für Clustervorgänge und optional für Ihre Anwendungen.
   
			mysql -u root -p
			GRANT ALL PRIVILEGES ON *.* TO 'cluster'@'%' IDENTIFIED BY 'p@ssw0rd' WITH GRANT OPTION; FLUSH PRIVILEGES;
            exit
   
   - Beenden Sie MySQL.
   
			service mysql stop
            
7. Erstellen Sie den Konfigurationsplatzhalter

	- Bearbeiten Sie die MySQL-Konfiguration, um einen Platzhalter für die Clustereinstellungen zu erstellen. Ersetzen Sie **"<Variables>"** nicht, oder heben Sie die Auskommentierung jetzt nicht auf. Dies geschieht nach dem Erstellen eines virtuellen Computers aus dieser Vorlage.
	
			vi /etc/my.cnf.d/server.cnf
			
	- Bearbeiten Sie den Abschnitt **[galera]** und bereinigen Sie ihn.
	
	- Bearbeiten Sie den Abschnitt **[mariadb]**.
	
			wsrep_provider=/usr/lib64/galera/libgalera_smm.so 
            binlog_format=ROW
            wsrep_sst_method=rsync
			bind-address=0.0.0.0 # When set to 0.0.0.0, the server listens to remote connections
			default_storage_engine=InnoDB
            innodb_autoinc_lock_mode=2
			
            wsrep_sst_auth=cluster:p@ssw0rd # CHANGE: Username and password you created for the SST cluster MySQL user
            #wsrep_cluster_name='mariadbcluster' # CHANGE: Uncomment and set your desired cluster name
            #wsrep_cluster_address="gcomm://mariadb1,mariadb2,mariadb3" # CHANGE: Uncomment and Add all your servers
            #wsrep_node_address='<ServerIP>' # CHANGE: Uncomment and set IP address of this server
			#wsrep_node_name='<NodeName>' # CHANGE: Uncomment and set the node name of this server

8. Öffnen Sie die erforderlichen Ports an der Firewall mithilfe von FirewallD unter CentOS 7).
	
	- MySQL: `firewall-cmd --zone=public --add-port=3306/tcp --permanent`
    - GALERA: `firewall-cmd --zone=public --add-port=4567/tcp --permanent`
    - GALERA IST: `firewall-cmd --zone=public --add-port=4568/tcp --permanent`
    - RSYNC: `firewall-cmd --zone=public --add-port=4444/tcp --permanent`
    - Laden Sie die Firewall erneut: `firewall-cmd --reload`
	
9.  Optimieren Sie die Leistung des Systems. In diesem Artikel zur [Strategie für die Leistungsoptimierung] erhalten Sie weitere Informationen.

	- Bearbeiten Sie die MySQL-Konfigurationsdatei erneut.
	
			vi /etc/my.cnf.d/server.cnf
		
	- Bearbeiten Sie den Abschnitt **[mariadb]**, und fügen Sie Folgendes an:
	
	> [AZURE.NOTE] Es wird empfohlen, dass **innodb\_buffer\_pool_size** 70 % des Speichers des virtuellen Computers beträgt. In diesem Fall wurde sie für den mittleren virtuellen Azure-Computer mit 3,5 GB RAM auf 2,45 GB festgelegt.
	
	        innodb_buffer_pool_size = 2508M # The buffer pool contains buffered data and the index. This is usually set to 70% of physical memory.
            innodb_log_file_size = 512M #  Redo logs ensure that write operations are fast, reliable, and recoverable after a crash
            max_connections = 5000 # A larger value will give the server more time to recycle idled connections
            innodb_file_per_table = 1 # Speed up the table space transmission and optimize the debris management performance
            innodb_log_buffer_size = 128M # The log buffer allows transactions to run without having to flush the log to disk before the transactions commit
            innodb_flush_log_at_trx_commit = 2 # The setting of 2 enables the most data integrity and is suitable for Master in MySQL cluster
			query_cache_size = 0
			
10. Beenden Sie MySQL, verhindern Sie, dass der MySQL-Dienst beim Start ausgeführt wird, um beim Hinzufügen eines neuen Knotens den Cluster nicht durcheinander zu bringen, und haben Sie die Bereitstellung des Computers auf.

		service mysql stop
        chkconfig mysql off
		waagent -deprovision	
		
11. Erfassen Sie den virtuellen Computer über das Portal: (Derzeit wird unter [Problem 1268 in den Azure-Befehlszeilenschnittstellen-Tools] die Tatsache beschrieben, dass Abbilder, die mit den plattformübergreifenden Befehlszeilenschnittstellen-Tools erfasst werden, nicht auf den angefügten Datenträgern gespeichert werden können.

	- Fahren Sie den Computer über das Portal herunter.
    - Klicken Sie auf "Erfassen", und geben Sie den Namen des Abbilds als **mariadb-galera-image** an. Geben Sie auch eine Beschreibung ein, und aktivieren Sie "Ich habe waagent ausgeführt".
	![Capture the Virtual Machine](./media/virtual-machines-mariadb-cluster/Capture.png)
	![Capture the Virtual Machine](./media/virtual-machines-mariadb-cluster/Capture2.PNG)
	
## Erstellen des Clusters
 
Erstellen Sie drei virtuelle Computer aus der Vorlage, die Sie gerade erstellt haben, und konfigurieren und starten Sie dann den Cluster.

1. Erstellen Sie den ersten virtuellen CentOS 7-Computer aus dem Abbild **mariadb-galera-image**, das Sie erstellt haben, und geben Sie den virtuellen Netzwerknamen **mariadbvnet** und das Subnetz **mariadb** und die Computergröße **Mittel** an, wobei der Name des Cloud-Diensts auf **mariadbha** festgelegt wird (bzw. auf einen beliebigen Namen, auf den über mariadbha.cloudapp.net zugegriffen werden soll), wobei der Name dieses Computers **mariadb1** und der Benutzername **azureuser** sein und der SSH-Zugriff aktiviert sein soll, die PEM-SSH-Zertifikatsdatei übergeben wird und **/path/to/key.pem** durch den Pfad ersetzt wird, unter dem Sie den generierten PEM-SSH-Schlüssel gespeichert haben.

	> [AZURE.NOTE] Die folgenden Befehle sind aus Gründen der Übersichtlichkeit auf mehrere Zeilen aufgeteilt, Sie sollten jedoch jeden Befehl als eine Zeile eingeben.

		azure vm create 
        --virtual-network-name mariadbvnet
        --subnet-names mariadb 
        --availability-set clusteravset
		--vm-size Medium 
		--ssh-cert "/path/to/key.pem" 
		--no-ssh-password 
		--ssh 22 
		--vm-name mariadb1 
		mariadbha mariadb-galera-image azureuser 

2. Erstellen Sie zwei weitere virtuelle Computer, indem Sie sie mit dem derzeit erstellten Cloud-Dienst **mariadbha** verbinden, und ändern Sie dabei den **Namen des virtuellen Computers** und auch den **SSH-Port** in einen eindeutigen Port, der nicht mit anderen virtuellen Computern in demselben Cloud-Dienst in Konflikt steht.

		azure vm create		
        --virtual-network-name mariadbvnet
        --subnet-names mariadb 
        --availability-set clusteravset
		--vm-size Medium
		--ssh-cert "/path/to/key.pem"
		--no-ssh-password
		--ssh 23
		--vm-name mariadb2
        --connect mariadbha mariadb-galera-image azureuser
und für MariaDB3

		azure vm create
        --virtual-network-name mariadbvnet
        --subnet-names mariadb 
        --availability-set clusteravset
		--vm-size Medium
		--ssh-cert "/path/to/key.pem"
		--no-ssh-password
		--ssh 24
		--vm-name mariadb3
        --connect mariadbha mariadb-galera-image azureuser
		
3. Für den nächsten Schritt müssen Sie die interne IP-Adresse aller drei virtuellen Computer abrufen:

	![Getting IP address](./media/virtual-machines-mariadb-cluster/IP.png)

4. Kopieren Sie den SSH-Schlüssel auf die drei virtuellen Computer, und bearbeiten Sie die Konfigurationsdatei auf jedem Computer,

		sudo vi /etc/my.cnf.d/server.cnf
		
	und kommentieren Sie dabei **"wsrep_cluster_name"** und **"wsrep_cluster_address"** aus, indem Sie das **#** am Anfang entfernen und überprüfen, dass sie tatsächliche Ihren Vorstellungen entsprechen.
    Ersetzen Sie außerdem **"<ServerIP>"** in **"wsrep_node_address"** und **"<NodeName>"** in **"wsrep_node_name"** durch die IP-Adresse bzw. den Namen des virtuellen Computers, und kommentieren Sie auch diese Zeilen aus.
	
5. Starten Sie den Cluster auf MariaDB1, und lassen Sie ihn beim Start ausführen.

		sudo service mysql bootstrap
        chkconfig mysql on
	
6. Starten Sie MySQL auf MariaDB2 und MariaDB3, und lassen Sie es beim Start ausführen.

		sudo service mysql start
        chkconfig mysql on
		
## Lastenausgleich für den Cluster
Als Sie die gruppierten virtuellen Computer erstellt haben, haben Sie sie einer Verfügbarkeitsgruppen mit dem Namen **clusteravset** hinzugefügt, um sicherzustellen, dass sie in unterschiedlichen Fehler- und Updatedomänen platziert werden und dass von Azure niemals eine Wartung auf allen Computern gleichzeitig ausgeführt wird. Diese Konfiguration erfüllt die Anforderungen, die von diesem Azure Service Level Agreement (SLA) unterstützt werden sollen.

Jetzt können Sie den Azure-Lastenausgleich verwenden, um Anforderungen zwischen den drei Knoten auszugleichen.

Führen Sie die folgenden Befehle auf Ihrem Computer mithilfe der Azure-Befehlszeilenschnittstelle aus.
Die Struktur der Befehlsparameter ist wie folgt:  `azure vm endpoint create-multiple <MachineName> <PublicPort>:<VMPort>:<Protocol>:<EnableDirectServerReturn>:<Load Balanced Set Name>:<ProbeProtocol>:<ProbePort>`

	azure vm endpoint create-multiple mariadb1 3306:3306:tcp:false:MySQL:tcp:3306
    azure vm endpoint create-multiple mariadb2 3306:3306:tcp:false:MySQL:tcp:3306
    azure vm endpoint create-multiple mariadb3 3306:3306:tcp:false:MySQL:tcp:3306
	
Da das Testintervall des Lastenausgleichs von der Befehlszeilenschnittstelle auf 15 Sekunden festgelegt wird (was möglicherweise etwas zu lang ist), ändern Sie es als letzten Schritt im Portal unter **Endpunkte** für alle virtuellen Computer,

![Edit endpoint](./media/virtual-machines-mariadb-cluster/Endpoint.PNG)

und klicken Sie dann auf "Gruppe mit Lastenausgleich neu konfigurieren" und dann auf "Weiter".

![Reconfigure Load Balanced Set](./media/virtual-machines-mariadb-cluster/Endpoint2.PNG)

Ändern Sie dann das Testintervall auf 5 Sekunden, und speichern Sie.

![Change Probe Interval](./media/virtual-machines-mariadb-cluster/Endpoint3.PNG)

## Überprüfen des Clusters

Die harte Arbeit ist nun erledigt. Der Cluster sollte jetzt unter "mariadbha.cloudapp.net:3306" für den Lastenausgleich erreichbar sein und Anforderungen zwischen den drei virtuellen Computern reibungslos und effektiv weiterleiten.

Verwenden Sie Ihren bevorzugten MySQL-Client, um eine Verbindung herzustellen, oder stellen Sie einfach eine Verbindung von einem der virtuellen Computer her, um zu überprüfen, dass dieser Cluster funktioniert.

	 mysql -u cluster -h mariadbha.cloudapp.net -p
	 
Erstellen Sie dann eine neue Datenbank, und füllen Sie sie mit Daten.

	CREATE DATABASE TestDB;
    USE TestDB;
    CREATE TABLE TestTable (id INT NOT NULL AUTO_INCREMENT PRIMARY KEY, value VARCHAR(255));
	INSERT INTO TestTable (value)  VALUES ('Value1');
	INSERT INTO TestTable (value)  VALUES ('Value2');
    SELECT * FROM TestTable;
	
Das Ergebnis sieht folgendermaßen aus:

	+----+--------+
	| id | value  |
	+----+--------+
	|  1 | Value1 |
	|  4 | Value2 |
	+----+--------+
	2 rows in set (0.00 sec)	
	
<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## Nächste Schritte

In diesem Artikel haben Sie einen hoch verfügbaren MariaDB + Galera-Cluster mit drei Knoten auf virtuellen Azure-Computern erstellt, auf denen CentOS 7 ausgeführt wird. Der Lastenausgleich für die virtuellen Computer erfolgt mit dem Azure-Lastenausgleich.

Sehen Sie sich eine [weitere Möglichkeit zur Gruppierung von MySQL unter Linux] sowie Möglichkeiten zum [Optimieren und Testen der MySQL-Leistung auf virtuellen Azure-Computern mit Linux].

<!--Anchors-->
[Übersicht über die Architektur]: #architecture-overview
[Erstellen der Vorlage]: #creating-the-template
[Erstellen des Clusters]: #creating-the-cluster
[Lastenausgleich für den Cluster]: #load-balancing-the-cluster
[Überprüfen des Clusters]: #validating-the-cluster
[Nächste Schritte]: #next-steps

<!--Image references-->

<!--Link references-->
[Azure-Befehlszeilenschnittstelle]: http://azure.microsoft.com/de-de/documentation/articles/xplat-cli/
[Befehlsreferenz der Azure-Befehlszeilenschnittstelle]: http://azure.microsoft.com/de-de/documentation/articles/command-line-tools/
[SSH-Schlüssel für die Authentifizierung]:http://www.jeff.wilcox.name/2013/06/secure-linux-vms-with-ssh-certificates/
[Strategie für die Leistungsoptimierung]: http://azure.microsoft.com/sv-se/documentation/articles/virtual-machines-linux-optimize-mysql-perf/
[Optimieren und Testen der MySQL-Leistung auf virtuellen Azure-Computern mit Linux]:http://azure.microsoft.com/sv-se/documentation/articles/virtual-machines-linux-optimize-mysql-perf/
[Problem 1268 in den Azure-Befehlszeilenschnittstellen-Tools]:https://github.com/Azure/azure-xplat-cli/issues/1268
[weitere Möglichkeit zur Gruppierung von MySQL unter Linux]: http://azure.microsoft.com/de-de/documentation/articles/virtual-machines-linux-mysql-cluster/



<!--HONumber=42-->