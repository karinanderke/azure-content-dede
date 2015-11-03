<properties
   	pageTitle="Bereitstellen von Hadoop-, HBase- oder Storm-Clustern unter Linux in HDInsight mithilfe der Azure-Befehlszeilenschnittstelle | Microsoft Azure"
   	description="Erfahren Sie, wie Hadoop-, HBase- oder Storm-Cluster unter Linux für HDInsight mithilfe der Azure-Befehlszeilenschnittstelle erstellt werden."
   	services="hdinsight"
   	documentationCenter=""
   	authors="nitinme"
   	manager="paulettm"
   	editor="cgronlun"
	tags="azure-portal"/>

<tags
   	ms.service="hdinsight"
   	ms.devlang="na"
   	ms.topic="article"
   	ms.tgt_pltfrm="na"
   	ms.workload="big-data"
   	ms.date="10/14/2015"
   	ms.author="nitinme"/>

#Erstellen von Linux-basierten Clustern in HDInsight mit Azure PowerShell

[AZURE.INCLUDE [Auswahl](../../includes/hdinsight-create-linux-cluster-selector.md)]

Azure PowerShell ist eine leistungsstarke Skriptumgebung, mit der Sie die Bereitstellung und Verwaltung Ihrer Workloads in Azure steuern und automatisieren können. Dieses Dokument enthält Informationen zum Bereitstellen eines Linux-basierten HDInsight-Clusters mit Azure PowerShell sowie ein Beispielskript.

> [AZURE.NOTE]Azure PowerShell ist nur auf Windows-Clients verfügbar. Wenn Sie einen Linux-, Unix- oder Mac OS X-Client verwenden, finden Sie im Artikel zum [Erstellen eines Linux-basierten HDInsight-Clusters mit der Azure-Befehlszeilenschnittstelle](hdinsight-hadoop-create-linux-cluster-azure-cli.md) Informationen zum Erstellen eines Clusters mit der Azure-Befehlszeilenschnittstelle.

##Voraussetzungen

- **Ein Azure-Abonnement**. Siehe [Kostenlose Azure-Testversion](http://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/).

- __Azure PowerShell__ Weitere Informationen zum Konfigurieren einer Arbeitsstation für die Ausführung von HDInsight Powershell-Cmdlets finden Sie unter [Installieren und Konfigurieren von Azure PowerShell](../install-configure-powershell.md). Weitere Informationen zum Verwenden von Azure PowerShell mit HDInsight finden Sie unter [Verwalten von HDInsight mit PowerShell](hdinsight-administer-use-powershell.md). Eine Liste der HDInsight Windows PowerShell-Cmdlets finden Sie unter [HDInsight-Cmdlet-Referenz](https://msdn.microsoft.com/library/azure/dn858087.aspx).

##Cluster erstellen

Die folgenden Prozeduren müssen für die Bereitstellung eines HDInsight-Clusters mithilfe von Azure PowerShell ausgeführt werden:

- Erstellen einer Azure-Ressourcengruppe
- Erstellen eines Azure-Speicherkontos
- Erstellen eines Azure-Blobcontainers
- Erstellen eines HDInsight-Clusters

Die beiden wichtigsten Parameter, die für die Bereitstellung von Linux-Clustern festgelegt werden müssen, sind diejenigen zum Angeben des Typs des Betriebssystems und der Details zum SSH-Benutzer:

- Stellen Sie sicher, dass Sie den Parameter **- OSType** als **Linux** angeben.
- Um SSH für Remotesitzungen mit dem Cluster zu verwenden, können Sie entweder das SSH-Benutzerkennwort oder den öffentlichen SSH-Schlüssel angeben. Wenn Sie sowohl das SSH-Benutzerkennwort und als auch den öffentlichen SSH-Schlüssel angeben, wird der Schlüssel ignoriert. Wenn Sie den SSH-Schlüssel für Remotesitzungen verwenden möchten, müssen Sie bei der Aufforderung zur Eingabe ein leeres SSH-Kennwort angeben. Weitere Informationen zur Verwendung von SSH mit HDInsight finden Sie in den folgenden Artikeln:
    
    * [Verwenden von SSH mit Linux-basiertem Hadoop in HDInsight unter Linux, Unix oder OS X](hdinsight-hadoop-linux-use-ssh-unix.md)
    * [Verwenden von SSH mit Linux-basiertem Hadoop in HDInsight unter Windows](hdinsight-hadoop-linux-use-ssh-windows.md)

Das folgende Skript veranschaulicht das Erstellen eines neuen Clusters:

    # Use the new Azure Resource Manager mode
    Switch-AzureMode AzureResourceManager

    ###########################################
    # Create required items, if none exist
    ###########################################

    # Sign in
    Add-AzureAccount

    # Select the subscription to use
    $subscriptionName = "<SubscriptionName>"        # Provide your Subscription Name
    Select-AzureSubscription -SubscriptionName $subscriptionName

    # Register your subscription to use HDInsight
    Register-AzureProvider -ProviderNamespace "Microsoft.HDInsight" -Force

    # Create an Azure Resource Group
    $resourceGroupName = "<ResourceGroupName>"      # Provide a Resource Group name
    $location = "<Location>"                        # For example, "West US"
    New-AzureResourceGroup -Name $resourceGroupName -Location $location

    # Create an Azure Storage account
    $storageAccountName = "<StorageAcccountName>"   # Provide a Storage account name
    New-AzureStorageAccount -ResourceGroupName $resourceGroupName -StorageAccountName $storageAccountName -Location $location -Type Standard_GRS

    # Create an Azure Blob Storage container
    $containerName = "<ContainerName>"              # Provide a container name
    $storageAccountKey = Get-AzureStorageAccountKey -Name $storageAccountName -ResourceGroupName $resourceGroupName | %{ $_.Key1 }
    $destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
    New-AzureStorageContainer -Name $containerName -Context $destContext

    ###########################################
    # Create an HDInsight Cluster
    ###########################################

    # Skip these variables if you just created them
    $resourceGroupName = "<ResourceGroupName>"      # Provide the Resource Group name
    $storageAccountName = "<StorageAcccountName>"   # Provide the Storage account name
    $containerName = "<ContainerName>"              # Provide the container name
    $storageAccountKey = Get-AzureStorageAccountKey -Name $storageAccountName -ResourceGroupName $resourceGroupName | %{ $_.Key1 }

    # Set these variables
    $clusterName = $containerName           		# As a best practice, have the same name for the cluster and container
    $clusterNodes = <ClusterSizeInNodes>    		# The number of nodes in the HDInsight cluster
    $credentials = Get-Credential
    $sshCredentials = Get-Credential

    # The location of the HDInsight cluster. It must be in the same data center as the Storage account.
    $location = Get-AzureStorageAccount -ResourceGroupName $resourceGroupName -StorageAccountName $storageAccountName | %{$_.Location}

    # Create a new HDInsight cluster
    New-AzureHDInsightCluster -ClusterName $clusterName -ResourceGroupName $resourceGroupName -HttpCredential $credentials -Location $location -DefaultStorageAccountName "$storageAccountName.blob.core.windows.net" -DefaultStorageAccountKey $storageAccountKey -DefaultStorageContainer $containerName  -ClusterSizeInNodes $clusterNodes -ClusterType Hadoop -OSType Linux -Version "3.2" -SshCredential $sshCredentials

> [AZURE.NOTE]Die Werte, die Sie für **$clusterCredentials** angeben, werden verwendet, um das Hadoop-Benutzerkonto für den Cluster zu erstellen. Sie verwenden dieses Konto für die Verbindung mit dem Cluster. Die Werte, die Sie für **$sshCredentials** angeben, werden verwendet, um den SSH-Benutzer für den Cluster zu erstellen. Sie verwenden dieses Konto zum Starten einer SSH-Remotesitzung mit dem Cluster und Ausführen von Aufträgen. Wenn Sie im Azure-Portal die Option "Schnellerfassung" verwenden, um einen Cluster bereitzustellen, ist der Hadoop-Standardbenutzername "admin" und der SSH-Standardbenutzername "ddiuser".

Es kann bis zu 15 Minuten dauern, bis die Bereitstellung abgeschlossen ist.

##Nächste Schritte

Nachdem Sie einen HDInsight-Cluster erfolgreich erstellt haben, nutzen Sie die folgenden Informationen, um zu erfahren, wie Sie mit Ihrem Cluster arbeiten:

###Hadoop-Cluster

* [Verwenden von Hive mit HDInsight](hdinsight-use-hive.md)
* [Verwenden von Pig mit HDInsight](hdinsight-use-pig.md)
* [Verwenden von MapReduce mit HDInsight](hdinsight-use-mapreduce.md)

###HBase-Cluster

* [Erste Schritte mit HBase in HDInsight](hdinsight-hbase-tutorial-get-stared-linux.md)
* [Entwickeln von Java-Anwendung für HBase in HDInsight](hdinsight-hbase-build-java-maven-linux)

###Storm-Cluster

* [Entwickeln von Java-Topologien für Storm in HDInsight](hdinsight-storm-develop-java-topology.md)
* [Verwenden von Python-Komponenten in Storm in HDInsight](hdinsight-storm-develop-python.md)
* [Bereitstellen und Überwachen von Topologien mit Storm in HDInsight](hdinsight-storm-deploy-monitor-topology-linux.md)

<!---HONumber=Oct15_HO3-->