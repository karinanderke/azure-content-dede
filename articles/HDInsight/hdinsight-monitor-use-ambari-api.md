<properties 
	pageTitle="Überwachen von Hadoop-Clustern in HDInsight mit der Ambari API | Azure" 
	description="Verwenden Sie die Apache Ambari-APIs für die Bereitstellung, Verwaltung und Überwachung von Hadoop-Clustern. Die intuitiven Operatortools und APIs von Ambari verbergen die Komplexität von Hadoop." 
	services="hdinsight" 
	documentationCenter="" 
	authors="mumian" 
	editor="cgronlun" 
	manager="paulettm"/>

<tags 
	ms.service="hdinsight" 
	ms.workload="big-data" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="03/31/2015" 
	ms.author="jgao"/>

# Überwachen von Hadoop-Clustern in HDInsight mit der Ambari API
 
Erfahren Sie, wie Sie HDInsight-Cluster der Versionen 3.1 und 2.1 mit den Ambari-APIs überwachen können.



## <a id="whatisambari"></a> Was ist Ambari?

[Apache Ambari][ambari-home] dient zur Bereitstellung, Verwaltung und Überwachung von Apache Hadoop-Clustern. Es umfasst eine Sammlung intuitiver Operatortools und eine Reihe robuster APIs, welche die Komplexität von Hadoop verbergen und den Betrieb von Clustern vereinfachen. Weitere Informationen über die APIs finden Sie unter [Ambari API Reference][ambari-api-reference] (Referenz zur Ambari-API, in englischer Sprache).


HDInsight unterstützt derzeit nur das Ambari-Überwachungsfeature. Die Ambari-API 1.0 wird von HDInsight-Clustern der Versionen 3.0 und 2.1 unterstützt. Dieser Artikel behandelt den Zugriff auf HDInsight-Cluster der Versionen 3.1 und 2.1 mithilfe der Ambari-APIs. Die beiden Versionen unterscheiden sich hauptsächlich darin, dass sich bestimmte Komponenten mit der Einführung neuer Funktionen geändert haben (z. B. der Auftragsverlauf-Server).


##<a id="prerequisites"></a>Voraussetzungen

Bevor Sie mit diesem Lernprogramm beginnen können, benötigen Sie Folgendes:

- **Eine Arbeitsstation**, auf der Azure PowerShell installiert und konfiguriert ist. Anweisungen hierzu finden Sie unter [Installieren und Konfigurieren von Azure PowerShell][powershell-install]. Um Azure PowerShell-Skripts ausführen zu können, müssen Sie Azure PowerShell als Administrator ausführen und die Ausführungsrichtlinie auf *RemoteSigned* setzen. Weitere Informationen finden Sie unter [Ausführen von Windows PowerShell-Skripts][powershell-script].

- (Optional) [cURL][curl]. Weitere Informationen zum Installieren finden Sie unter [cURL Releases and Downloads][curl-download] (cURL – Versionen und Downloads, in englischer Sprache).

	>[AZURE.NOTE]Wenn Sie den cURL-Befehl in Windows verwenden, geben Sie die Optionswerte in doppelten anstelle von einfachen Anführungszeichen ein.

- **Ein Azure HDInsight-Cluster**. Anweisungen zur Bereitstellung von Clustern finden Sie unter [Erste Schritte mit HDInsight][hdinsight-get-started] oder unter [Bereitstellen von HDInsight-Clustern][hdinsight-provision]. Sie benötigen die folgenden Daten, um das Lernprogramm durchzuarbeiten:

	<table border="1">
<tr><th>Clustereigenschaft</th><th>Azure PowerShell-Variablenname</th><th>Wert</th><th>Beschreibung</th></tr>
<tr><td>HDInsight-Clustername</td><td>$clusterName</td><td></td><td>Der Name des HDInsight-Clusters.</td></tr>
<tr><td>Cluster-Benutzername</td><td>$clusterUsername</td><td></td><td>Der bei der Bereitstellung angegebene Cluster-Benutzername.</td></tr>
<tr><td>Cluster-Kennwort</td><td>$clusterPassword</td><td></td><td>Das Benutzerkennwort für den Cluster.</td></tr>
</table>
> [AZURE.NOTE]Tragen Sie die Werte in die Tabelle ein. Dies wird Ihnen helfen, wenn Sie dieses Lernprogramm durcharbeiten.



##<a id="jumpstart"></a>Schneller Einstieg

Es gibt verschiedene Möglichkeiten, Ambari zur Überwachung von HDInsight-Clustern einzusetzen.

**Verwenden von Azure PowerShell**

Das folgende Skript ist ein Azure PowerShell-Skript zum Abrufen der MapReduce-JobTracker-Informationen *in einem HDInsight 3.1-Cluster*. Der wichtigste Unterschied hier besteht darin, dass wir diese Informationen nun vom YARN-Dienst abrufen (anstelle von MapReduce).

	$clusterName = "<HDInsightClusterName>"
	$clusterUsername = "<HDInsightClusterUsername>"
	$clusterPassword = "<HDInsightClusterPassword>"
	
	$ambariUri = "https://$clusterName.azurehdinsight.net:443/ambari"
	$uriJobTracker = "$ambariUri/api/v1/clusters/$clusterName.azurehdinsight.net/services/yarn/components/resourcemanager"
	
	$passwd = ConvertTo-SecureString $clusterPassword -AsPlainText -Force
	$creds = New-Object System.Management.Automation.PSCredential ($clusterUsername, $passwd)
	
	$response = Invoke-RestMethod -Method Get -Uri $uriJobTracker -Credential $creds -OutVariable $OozieServerStatus 
	
	$response.metrics.'yarn.queueMetrics'

Das folgende Skript ist ein Azure PowerShell-Skript zum Abrufen der MapReduce-JobTracker-Informationen *in einem HDInsight 2.1-Cluster*.

	$clusterName = "<HDInsightClusterName>"
	$clusterUsername = "<HDInsightClusterUsername>"
	$clusterPassword = "<HDInsightClusterPassword>"
	
	$ambariUri = "https://$clusterName.azurehdinsight.net:443/ambari"
	$uriJobTracker = "$ambariUri/api/v1/clusters/$clusterName.azurehdinsight.net/services/mapreduce/components/jobtracker"
	
	$passwd = ConvertTo-SecureString $clusterPassword -AsPlainText -Force
	$creds = New-Object System.Management.Automation.PSCredential ($clusterUsername, $passwd)
	
	$response = Invoke-RestMethod -Method Get -Uri $uriJobTracker -Credential $creds -OutVariable $OozieServerStatus 
	
	$response.metrics.'mapred.JobTracker'

Die Ausgabe ist:

![JobTracker-Ausgabe][img-jobtracker-output]

**Verwenden von cURL**

Es folgt ein Beispiel zum Abrufen von Clusterinformationen mithilfe von cURL:

	curl -u <username>:<password> -k https://<ClusterName>.azurehdinsight.net:443/ambari/api/v1/clusters/<ClusterName>.azurehdinsight.net

Die Ausgabe ist:
	
	{"href":"https://hdi0211v2.azurehdinsight.net/ambari/api/v1/clusters/hdi0211v2.azurehdinsight.net/",
	 "Clusters":{"cluster_name":"hdi0211v2.azurehdinsight.net","version":"2.1.3.0.432823"},
	 "services"[
	   {"href":"https://hdi0211v2.azurehdinsight.net/ambari/api/v1/clusters/hdi0211v2.azurehdinsight.net/services/hdfs",
	    "ServiceInfo":{"cluster_name":"hdi0211v2.azurehdinsight.net","service_name":"hdfs"}},
	   {"href":"https://hdi0211v2.azurehdinsight.net/ambari/api/v1/clusters/hdi0211v2.azurehdinsight.net/services/mapreduce",
	    "ServiceInfo":{"cluster_name":"hdi0211v2.azurehdinsight.net","service_name":"mapreduce"}}],
	 "hosts":[
	   {"href":"https://hdi0211v2.azurehdinsight.net/ambari/api/v1/clusters/hdi0211v2.azurehdinsight.net/hosts/headnode0",
	    "Hosts":{"cluster_name":"hdi0211v2.azurehdinsight.net",
	             "host_name":"headnode0"}},
	   {"href":"https://hdi0211v2.azurehdinsight.net/ambari/api/v1/clusters/hdi0211v2.azurehdinsight.net/hosts/workernode0",
	    "Hosts":{"cluster_name":"hdi0211v2.azurehdinsight.net",
	             "host_name":"headnode0.{ClusterDNS}.azurehdinsight.net"}}]}

**Hinweis für die Version vom 08.10.2014**:

Bei der Verwendung des Ambari-Endpunkts "https://{clusterDns}.azurehdinsight.net/ambari/api/v1/clusters/{clusterDns}.azurehdinsight.net/services/{Dienstname}/components/{Komponentenname}", gibt das Feld *host_name* den vollqualifizierten Domänennamen (FQDN) des Knotens anstatt nur des Hostnamens zurück. Vor der Version vom 10.8.2014 gab dieses Beispiel lediglich "**headnode0**" zurück. Seit der Version vom 10.8.2014 wird der FQDN "**headnode0.{ClusterDNS}.azurehdinsight.net**" zurückgegeben, wie im vorherigen Beispiel gezeigt. Diese Änderung erleichtert Szenarios, in denen mehrere Clustertypen wie z. B. HBase und Hadoop in einem einzigen virtuellen Netzwerk (VNET) bereitgestellt werden. Dies ist z. B. der Fall, wenn HBase als Back-End-Plattform für Hadoop verwendet wird.

##<a id="monitor"></a>Ambari-Überwachungs-APIs

In der folgenden Tabelle sind einige der am häufigsten verwendeten Ambari-Überwachungs-API-Aufrufe aufgeführt. Weitere Informationen zur API finden Sie in der [Ambari API-Referenz][ambari-api-reference].

<table border="1">
<tr><th>Überwachungs-API-Aufruf</th><th>URI</th><th>Beschreibung</th></tr>
<tr><td>Cluster abrufen</td><td><tt>/api/v1/clusters</tt></td><td></td></tr>
<tr><td>Clusterinformationen abrufen.</td><td><tt>/api/v1/clusters/&lt;ClusterName>.azurehdinsight.net</tt></td><td>Cluster, Dienste, Hosts</td></tr>
<tr><td>Dienste abrufen</td><td><tt>/api/v1/clusters/&lt;ClusterName>.azurehdinsight.net/services</tt></td><td>Services include: hdfs, mapreduce</td></tr>
<tr><td>Dienstinformationen abrufen.</td><td><tt>/api/v1/clusters/&lt;ClusterName>.azurehdinsight.net/services/&lt;ServiceName></tt></td><td></td></tr>
<tr><td>Dienstkomponenten abrufen</td><td><tt>/api/v1/clusters/&lt;ClusterName>.azurehdinsight.net/services/&lt;ServiceName>/components</tt></td><td>HDFS: namenode, datanode<br/>MapReduce: jobtracker; tasktracker</td></tr>
<tr><td>Komponenteninformationen abrufen.</td><td><tt>/api/v1/clusters/&lt;ClusterName>.azurehdinsight.net/services/&lt;ServiceName>/components/&lt;ComponentName></tt></td><td>ServiceComponentInfo, Hostkomponenten, Metriken</td></tr>
<tr><td>Hosts abrufen</td><td><tt>/api/v1/clusters/&lt;ClusterName>.azurehdinsight.net/hosts</tt></td><td>headnode0, workernode0</td></tr>
<tr><td>Hostinformationen abrufen.</td><td><tt>/api/v1/clusters/&lt;ClusterName>.azurehdinsight.net/hosts/&lt;HostName> 
</td><td></td></tr>
<tr><td>Hostkomponenten abrufen</td><td><tt>/api/v1/clusters/&lt;Clustername>.azurehdinsight.net/hosts/&lt;Hostname>/host_components </tt></td><td>namenode, resourcemanager</td></tr>
<tr><td>Host-Komponenteninformationen abrufen.</td><td><tt>/api/v1/clusters/&lt;Clustername>.azurehdinsight.net/hosts/&lt;Hostname>/host_components/&lt;Komponentenname> </tt></td><td>HostRoles, Komponente, Host, Metriken</td></tr>
<tr><td>Konfigurationen abrufen</td><td><tt>/api/v1/clusters/&lt;Clustername>.azurehdinsight.net/configurations </tt></td><td>Config types: core-site, hdfs-site, mapred-site, hive-site</td></tr>
<tr><td>Konfigurationsinformationen abrufen.</td><td><tt>/api/v1/clusters/&lt;Clustername>.azurehdinsight.net/configurations?type=&lt;Konfigurationstyp>&amp;tag=&lt;Versionsname> </tt></td><td>Config types: core-site, hdfs-site, mapred-site, hive-site</td></tr>
</table>


##<a id="nextsteps"></a>Nächste Schritte 

Sie haben erfahren, wie Ambari-Überwachungs-API-Aufrufe verwendet werden. Weitere Informationen finden Sie unter:

- [Verwalten von HDInsight-Clustern mit dem Verwaltungsportal][hdinsight-admin-portal]
- [Verwalten von HDInsight-Clustern mit Azure PowerShell][hdinsight-admin-powershell]
- [Verwalten von HDInsight-Clustern mit der Befehlszeilenschnittstelle][hdinsight-admin-cli]
- [HDInsight-Dokumentation][hdinsight-documentation]
- [Erste Schritte mit HDInsight][hdinsight-get-started]



[ambari-home]: http://ambari.apache.org/
[ambari-api-reference]: https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md

[curl]: http://curl.haxx.se
[curl-download]: http://curl.haxx.se/download.html

[microsoft-hadoop-SDK]: http://hadoopsdk.codeplex.com/wikipage?title=Ambari%20Monitoring%20Client

[Powershell-install]: ../install-configure-powershell.md
[Powershell-script]: http://technet.microsoft.com/library/ee176949.aspx

[hdinsight-admin-powershell]: hdinsight-administer-use-powershell.md
[hdinsight-admin-portal]: hdinsight-administer-use-management-portal.md
[hdinsight-admin-cli]: hdinsight-administer-use-command-line.md
[hdinsight-documentation]: /documentation/services/hdinsight/
[hdinsight-get-started]: ../hdinsight-get-started.md
[hdinsight-provision]: hdinsight-provision-clusters.md

[img-jobtracker-output]: ./media/hdinsight-monitor-use-ambari-api/hdi.ambari.monitor.jobtracker.output.png


<!--HONumber=54--> 