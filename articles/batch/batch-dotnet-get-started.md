<properties
	pageTitle="Lernprogramm - Erste Schritte mit der Azure Batch-Bibliothek für .NET"
	description="Lernen Sie die grundlegenden Konzepte für Azure-Stapel und die Entwicklung für den Batch-Dienst mit einem einfachen Szenario"
	services="batch"
	documentationCenter=".net"
	authors="yidingzhou"
	manager="timlt"
	editor=""/>

<tags
	ms.service="batch"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="big-compute"
	ms.date="05/04/2015"
	ms.author="yidingz"/>

#Erste Schritte mit der Azure Batch-Bibliothek für .NET  

Dieser Artikel enthält die folgenden beiden Lernprogramme, um Sie bei der Entwicklung mit der .NET-Bibliothek für den Azure Batch-Dienst zu unterstützen.

-	[Lernprogramm 1: Azure Batch Library für .NET](#tutorial1)
-	[Lernprogramm 2: Azure Batch Apps Library für .NET](#tutorial2)  


Szenarien für Azure-Stapel und Hintergrundinformationen finden Sie unter [– Technische Übersicht Azure Batch](batch-technical-overview.md).

##<a name="tutorial1"></a>Lernprogramm 1: Azure Batch Library für .NET

In diesem Lernprogramm wird die Erstellung einer Konsolenanwendung veranschaulicht, die die verteilte Berechnung für einen Pool virtueller Computer mithilfe des Azure Batch-Diensts einrichtet. Die in diesem Lernprogramm erstellten Aufgaben werten Text aus Dateien, die im Azure-Speicher enthalten sind, aus und geben die Wörter zurück, die am häufigsten verwendet werden. Die Beispiele sind in C# geschrieben und greifen auf die Azure Batch-Bibliothek für .NET zurück.


>[AZURE.NOTE]Um dieses Lernprogramm abzuschließen, benötigen Sie ein Azure-Konto. Sie können ein kostenloses Textkonto in wenigen Minuten erstellen. Ausführliche Informationen finden Sie unter [Kostenlose Azure-Testversion](http://azure.microsoft.com/pricing/free-trial/).
>
>Müssen Sie NuGet verwenden zum Abrufen der **Microsoft.Azure.Batch.dll** Assembly. Nachdem Sie das Projekt in Visual Studio erstellen, mit der rechten Maustaste des Projekts im **Projektmappen-Explorer** und wählen Sie **NuGet-Pakete verwalten**. Suchen Sie online nach **Azure.Batch** und klicken Sie dann auf installieren, um den Azure-Batch-Paket und Abhängigkeiten zu installieren.
>
>Stellen Sie sicher, dass Sie den NuGet-Paket-Manager, Version 2.8 oder höher, verwenden. Sie finden die Versionsnummer über Visual Studio -> "Hilfe" -> Dialogfeld "Info". Wenn Sie eine ältere Version des Nuget-Paket-Managers verwenden, müssen Sie Visual Studio aktualisieren, da ansonsten möglicherweise beim Herunterladen der richtigen Version der Nuget-Abhängigkeiten auftreten kann.
>
>Sie können darüber hinaus finden Sie in der [Azure Batch Hello World-Beispiel](https://code.msdn.microsoft.com/Azure-Batch-Sample-Hello-6573967c) auf MSDN für eine Stichprobe der hier besprochenen Code ähnelt.



###Konzepte
Der Batch-Dienst wird für die Planung skalierbarer und verteilter Rechenvorgänge verwendet. Er ermöglicht Ihnen die Ausführung umfangreicher Arbeitsauslastungen, die auf mehrere virtuelle Computer verteilt sind. Diese Arbeitsauslastungen bieten effiziente Unterstützung für intensive Rechenleistungen. Wenn Sie den Batch-Dienst verwenden, nutzen Sie die folgenden Ressourcen:

-	**Konto** : eine Entität innerhalb des Diensts eindeutig identifiziert. Die gesamte Verarbeitung erfolgt über ein Batch-Konto.
-	**Pool** – eine Auflistung von Task virtuelle Maschinen auf dem Task Anwendungen ausgeführt.
-	**TVM** – ein Computer, der einen Pool zugewiesen ist und die Aufgaben, die Aufträge zugewiesen sind, die im Pool ausgeführt, werden.
-	**Aufgabe Virtual Machine Benutzer** – ein Benutzerkonto auf eine TVM.
-	**Workitem** -gibt an, wie die Berechnung auf Task virtuelle Computer in einem Pool ausgeführt wird.
-	**Auftrag** : eine Instanz einer Arbeitsaufgabe ausgeführt und besteht aus einer Auflistung von Aufgaben.
-	**Aufgabe** – eine Anwendung, die einem Auftrag zugeordnet ist, und auf einer virtuellen Maschine von Task ausgeführt wird.
-	**Datei** – enthält die Informationen, die von einem Task verarbeitet wird.  

Beginnen wir mit der grundlegenden Syntax.

###Erstellen eines Azure Batch-Kontos
Sie können ein Batch-Konto über das Verwaltungsportal erstellen. Es wird Ihnen ein Schlüssel bereitgestellt, nachdem das Konto erstellt wurde. Weitere Informationen finden Sie unter [– Technische Übersicht Azure Batch](batch-technical-overview.md).

###Vorgehensweise: Hinzufügen eines Pools mit einem Konto
Der erste Satz von Ressourcen, den Sie zum Ausführen von Aufgaben erstellen müssen, ist ein Pool von virtuellen Aufgabencomputern.

1.	Öffnen Sie Microsoft Visual Studio 2013, auf die **Datei** Menü klicken Sie auf **Neu**, und klicken Sie dann auf **Projekt**.

2.	Von **Windows**, unter **Visual C#-**, klicken Sie auf **Konsolenanwendung**, nennen Sie das Projekt **GettingStarted**, nennen Sie die Projektmappe **AzureBatch**, und klicken Sie dann auf **OK**.

3.	Fügen Sie am Anfang der Datei "Program.cs" die folgenden Namespacedeklarationen hinzu:

		using Microsoft.Azure.Batch;
		using Microsoft.Azure.Batch.Auth;
		using Microsoft.Azure.Batch.Common;

	Stellen Sie sicher, dass auf die "Microsoft.Azure.Batch.dll"-Assembly verwiesen wird.

4.	Fügen Sie der Programmklasse die folgenden Variablen hinzu:

		private const string PoolName = "[name-of-pool]";
		private const int NumOfMachines = 3;
		private const string AccountName = "[name-of-batch-account]";
		private const string AccountKey = "[key-of-batch-account]";
		private const string Uri = "https://batch.core.windows.net";
	Ersetzen Sie die folgenden Werte:- **[Name-of-Pool]** - Name, der für den Pool verwenden möchten - **[Name-der-Batch-Account]** - Name des Batch-Kontos. - **[Schlüssel der Batch Konto]** -der Schlüssel, der für das Konto der Batch bereitgestellt wurde.
5.	Fügen Sie den folgenden Code zu "Main" hinzu, der die zu verwendenden Anmeldeinformationen definiert:

		BatchCredentials cred = new BatchCredentials(AccountName, AccountKey);
6.	Fügen Sie den folgenden Code zu "Main" hinzu, um den Client zu erstellen, der zum Ausführen von Vorgängen verwendet wird:

		IBatchClient client = BatchClient.Connect(Uri, cred);
7.	Fügen Sie den folgenden Code zu "Main" hinzu, der den Pool erstellt, sofern dieser nicht vorhanden ist:

		using (IPoolManager pm = client.OpenPoolManager())
		{
		   IEnumerable<ICloudPool> pools = pm.ListPools();
		   if (!pools.Select(pool => pool.Name).Contains(PoolName))
		   {
		      Console.WriteLine("The pool does not exist, creating now...");
		      ICloudPool newPool = pm.CreatePool(
		         PoolName,
		         osFamily: "3",
		         vmSize: "small",
		         targetDedicated: NumOfMachines);
		       newPool.Commit();
		    }
		}
		Console.WriteLine("Created pool {0}", PoolName);
		Console.ReadLine();
8.	Speichern Sie das Programm, und führen Sie es anschließend aus. Der Status lautet **Active** für einen Pool, die erfolgreich hinzugefügt wurde.  

###Vorgehensweise: Auflisten der Pools in einem Konto
Wenn Sie den Namen eines vorhandenen Pools nicht kennen, können Sie eine Liste der Pools in einem Konto abrufen.

1.	Fügen Sie den folgenden Code zu "Main" hinzu, der die zu verwendenden Anmeldeinformationen definiert:  

		BatchCredentials cred = new BatchCredentials(AccountName, AccountKey);
2.	Fügen Sie den folgenden Code zu "Main" hinzu, um den Client zu erstellen, der zum Ausführen von Vorgängen verwendet wird:

		IBatchClient client = BatchClient.Connect(Uri, cred);

3.	Aktualisieren Sie die "Main"-Prozedur in den folgenden Code, der die Namen und Statusangaben aller Pools in das Konto schreibt und den Pool erstellt, sofern dieser nicht vorhanden ist:

		BatchCredentials cred = new BatchCredentials(AccountName, AccountKey);
		IBatchClient client = BatchClient.Connect(Uri, cred);

		using (IPoolManager pm = client.OpenPoolManager())
		{
		    IEnumerable<ICloudPool> pools = pm.ListPools();

		    Console.WriteLine("Listing Pools\n=================");
		    foreach (var p in pools)
		    {
		        Console.WriteLine("Pool: " + p.Name + " State:" + p.State);
		    }  

		    if (!pools.Select(pool => pool.Name).Contains(PoolName))
		    {
		        Console.WriteLine("The pool does not exist, creating now...");
		        ICloudPool newPool = pm.CreatePool(
		           PoolName,
		           osFamily: "3",
		           vmSize: "small",
		           targetDedicated: NumOfMachines);
		        newPool.Commit();
		    }
		}
		Console.WriteLine("Created pool {0}. Press <Enter> to continue.", PoolName);
		Console.ReadLine();

4.	Speichern Sie das Programm, und führen Sie es anschließend aus. Die folgende Ausgabe wird angezeigt.

Listing Pools
		=================
		Pool: gettingstarted State:Active
		Created pool gettingstarted. Press <Enter> to continue.

###Vorgehensweise: Auflisten der Arbeitselemente in einem Konto
Wenn Sie den Namen eines vorhandenen Arbeitselements nicht kennen, können Sie eine Liste der Arbeitselemente in einem Konto abrufen.

1.	Fügen Sie folgenden Code am Ende von "Main" hinzu, der die Namen und Statusangaben aller Arbeitselemente in das Konto schreibt:

		using (IWorkItemManager wm = client.OpenWorkItemManager())
		{
		   Console.WriteLine("Listing Workitems\n=================");
		   IEnumerable<ICloudWorkItem> workitems = wm.ListWorkItems();
		   foreach (var w in workitems)
		   {
		      Console.WriteLine("Workitem: " + w.Name + " State:" + w.State);
		   }
		}
		Console.WriteLine("Press <Enter> to continue.");
		Console.ReadLine();
7.	Speichern Sie das Programm, und führen Sie es anschließend aus. Es wird vermutlich nichts angezeigt, da keine Arbeitselemente übermittelt wurden. Das Hinzufügen von Arbeitselementen wird im nächsten Abschnitt behandelt.  

##Vorgehensweise: Hinzufügen ein WorkItems zu einer Firma
Sie müssen ein Arbeitselement erstellen, um die Ausführung der Aufgaben im Pool zu definieren.

1.	Fügen Sie der Programmklasse die folgenden Variablen hinzu:

		private static readonly string WorkItemName = Environment.GetEnvironmentVariable("USERNAME") + DateTime.Now.ToString("yyyyMMdd-HHmmss");

2.	Beim Erstellen eines Arbeitselements wird auch ein Auftrag erstellt. Das Arbeitselement einen Namen zuweisen, aber der Auftrag zugewiesen ist immer der Name des **Auftrag 0000000001**. Fügen Sie den folgenden Code zu "Main" hinzu (vor dem Code zum Auflisten der Arbeitselemente), der das Arbeitselement hinzufügt:

		using (IWorkItemManager wm = client.OpenWorkItemManager())
		{
		   ICloudWorkItem cloudWorkItem = wm.CreateWorkItem(WorkItemName);
           cloudWorkItem.JobExecutionEnvironment = new JobExecutionEnvironment() {PoolName = PoolName};
		   cloudWorkItem.Commit();
		}
		Console.WriteLine("Workitem successfully added. Press <Enter> to continue.");
		Console.ReadLine();
3.	Speichern Sie das Programm, und führen Sie es anschließend aus. Der Status lautet **Active** eines WorkItems, die erfolgreich hinzugefügt wurde. Die folgende Ausgabe wird angezeigt.

		Listing Pools
		=================
		Pool: gettingstarted State:Active
		Created pool gettingstarted. Press <Enter> to continue.

		Workitem successfully added. Press <Enter> to continue.

		Listing Workitems
		=================
		Workitem: yidingz20141106-111211 State:Active
		Press <Enter> to continue.

###Vorgehensweise: Hinzufügen von Aufgaben an einen Auftrag
Ein Arbeitselement ohne Aufgabe führt keine weiteren Aktionen durch. Nachdem Sie das Arbeitselement erstellt haben und der Auftrag erstellt wurde, können Sie dem Auftrag Aufgaben hinzufügen. Dem Projekt soll eine einfache Aufgabe hinzugefügt werden.

1.	Fügen Sie der Programmklasse die folgenden Variablen hinzu:

		private const int Count = 4; // number of tasks that will run
		private const string JobName = "job-0000000001";

2.	Fügen Sie den folgenden Code zu "Main" hinzu, der Aufgaben zu einem Auftrag hinzufügt, auf deren Ausführung wartet und die Ergebnisse meldet:

		using (IWorkItemManager wm = client.OpenWorkItemManager())
		{
		    string taskName;
		    ICloudJob job = wm.GetJob(WorkItemName, JobName);
		    for (int i = 1; i <= Count; ++i)
		    {
		        taskName = "taskdata" + i;
		        string commandLine = String.Format("cmd /c echo I am {0}", taskName);
		        ICloudTask taskToAdd = new CloudTask(taskName, commandLine);
		        job.AddTask(taskToAdd);
		        job.Commit();
		        job.Refresh();
		    }

		    ICloudJob listjob = wm.GetJob(WorkItemName, JobName);
		    client.OpenToolbox().CreateTaskStateMonitor().WaitAll(listjob.ListTasks(),
		       TaskState.Completed, new TimeSpan(0, 30, 0));
		    Console.WriteLine("The tasks completed successfully. Terminating the workitem...");
		    wm.GetWorkItem(WorkItemName).Terminate();
		    foreach (ICloudTask task in listjob.ListTasks())
		    {
		        Console.WriteLine("Task " + task.Name + " says:\n" + task.GetTaskFile(Constants.StandardOutFileName).ReadAsString());
		    }
		    Console.ReadLine();
		}

3.	Speichern Sie das Programm, und führen Sie es anschließend aus. Das Ergebnis sollte wie folgt aussehen:

		The tasks completed successfully. Terminating the workitem...
		Task taskdata1 says:
		I am taskdata1

		Task taskdata2 says:
		I am taskdata2

		Task taskdata3 says:
		I am taskdata3

###Erstellen eines Programms zur Aufgabenverarbeitung
Nachdem wir "Hello World" auf einem virtuellen Computer ausführen können, wollen wir jetzt mit etwas Realem fortfahren. Wir erstellen in diesem Abschnitt ein Programm zur Aufgabenverarbeitung und laden es auf den virtuellen Aufgabencomputer hoch, der Aufgaben ausführt.

1.	Erstellen Sie im Projektmappen-Explorer ein neues Konsolenanwendungsprojekt mit dem Namen **Azuretasks** in den **AzureBatch** Lösung.

2.	Fügen Sie am Anfang der Datei "Program.cs" die folgenden Namespacedeklarationen hinzu:

		using System.Net;

3.	Fügen Sie den folgenden Code zu "Main" hinzu, der zum Verarbeiten von Daten verwendet wird:

		string blobName = args[0];
		int numTopN = int.Parse(args[1]);

		WebClient myWebClient = new WebClient();
		string content = myWebClient.DownloadString(blobName);

		string[] words = content.Split(' ');
		var topNWords =
		   words.
		   Where(word => word.Length > 0).
		   GroupBy(word => word, (key, group) => new KeyValuePair<String, long>(key, group.LongCount())).
		   OrderByDescending(x => x.Value).
		   Take(numTopN).
		   ToList();

		foreach (var pair in topNWords)
		{
		    Console.WriteLine("{0} {1}", pair.Key, pair.Value);
		}

4.	Speichern und erstellen Sie das Projekt.

###Vorbereiten von Ressourcen zum Ausführen von Aufgaben

Sie werden den folgenden einfachen Workflow bei der Erstellung eines verteilten Rechenszenarios mit dem Batch-Dienst verwenden:

1.	Laden Sie die Dateien, die Sie in Ihrem verteilten Rechenszenario verwenden möchten, auf ein Azure-Speicherkonto hoch. Diese Dateien müssen im Speicherkonto sein, damit der Batch-Dienst darauf zugreifen kann. Die Dateien werden auf einen virtuellen Aufgabencomputer geladen, wenn die Aufgabe ausgeführt wird.
2.	Laden Sie die abhängigen Binärdateien in das Speicherkonto hoch. Die Binärdateien enthalten das Programm, das durch die Aufgabe und die abhängigen Assemblys ausgeführt wird. Auf diese Dateien muss auch vom Speicher aus zugegriffen werden. Sie werden auf den virtuellen Aufgabencomputer geladen.
3.	Erstellen Sie einen Pool von virtuellen Aufgabencomputern. Sie können die Größe des zu verwendenden virtuellen Aufgabencomputers bei der Erstellung eines Pools zuweisen. Bei der Ausführung einer Aufgabe wird ihm ein virtueller Aufgabencomputer aus diesem Pool zugewiesen.
4.	Erstellen Sie ein Arbeitselement. Mit einem Arbeitselement können Sie einen Auftrag von Aufgaben verwalten. Beim Erstellen eines Arbeitselements wird automatisch ein Auftrag erstellt.
5.	Hinzufügen von Aufgaben zum Auftrag. Jede Aufgabe verwendet das von Ihnen hochgeladene Programm, um Informationen aus einer Datei zu verarbeiten, die Sie ebenfalls hochgeladen haben.
6.	Überwachen Sie die Ergebnisse der Ausgabe.  

Wir haben Schritt 3 bis 6 veranschaulicht. Jetzt wollen wir untersuchen, wie der Azure-Speicher für die Ausführung der Aufgabe vorbereitet wird.

####Abrufen von Speicherkonten
Sie benötigen ein Speicherkonto, um den Rest dieses Lernprogramms abschließen zu können. Wenn Sie dazu nicht kennen, finden Sie unter [erstellen ein Speicherkonto, das](#tutorial1_storage).

####Hochladen von Daten

1. Erstellen Sie in Ihrem Azure-Speicherkonto einen Container namens "gettingstarted". Dieser Vorgang kann im Azure-Portal ausgeführt werden. Stellen Sie sicher, dass Sie für das Feld "ZUGANG" den Wert "Öffentlichen Container" festlegen.

2. Laden Sie die Datei "ProcessTaskData.exe" in den Container hoch.

3. Erstellen Sie drei Textdateien (taskdata1.txt, taskdata2.txt, taskdata3.txt), die jeweils einen der folgenden Abschnitte enthalten, und laden Sie diese anschließend in den Container hoch:

		You can use Azure Virtual Machines to provision on-demand, scalable compute infrastructure when you need flexible resources for your business needs. From the gallery, you can create virtual machines that run Windows, Linux, and enterprise applications such as SharePoint and SQL Server. Or, you can capture and use your own images to create customized virtual machines.

		Quickly deploy and manage powerful applications and services with Azure Cloud Services. Simply upload your application and Azure handles the deployment details - from provisioning and load balancing to health monitoring for continuous availability. Your application is backed by an industry leading 99.95% monthly SLA. You just focus on the application and not the infrastructure.

		Azure Web Sites provide a scalable, reliable, and easy-to-use environment for hosting web applications. Select from a range of frameworks and templates to create a web site in seconds. Use any tool or OS to develop your site with .NET, PHP, Node.js or Python. Choose from a variety of source control options including TFS, GitHub, and BitBucket to set up continuous integration and develop as a team. Expand your site functionality over time by leveraging additional Azure managed services like storage, CDN, and SQL Database.


>[AZURE.NOTE]In einer Produktionsumgebung wird empfohlen, dass Sie eine SAS (Shared Access Signature) verwenden.


>[AZURE.NOTE]Azure Storage-Team hat eine [Blogbeitrag](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/03/11/windows-azure-storage-explorers-2014.aspx) Auflisten von Azure-Speicher-Explorer, mit denen Dateien hochladen können.



###Aktualisieren des Aufgabenübermittlungscodes

1.	Fügen Sie der Programmklasse die folgenden Variablen hinzu:

		private const string BlobPath = "[storage-path]";
	Ersetzen Sie die folgenden Werte:- **[Speicherpfad]** -der Pfad zu dem Blob im Speicher. Beispiel: http://yiding.blob.core.windows.net/gettingstarted/

2. Aktualisieren Sie den Aufgabenübermittlungscode wie folgt.

		string taskName;
		ICloudJob job = wm.GetJob(WorkItemName, JobName);
		for (int i = 1; i <= Count; ++i)
		{
		    taskName = "taskdata" + i;
		    IResourceFile programFile = new ResourceFile(BlobPath + "ProcessTaskData.exe", "ProcessTaskData.exe");
		    IResourceFile supportFile = new ResourceFile(BlobPath + taskName + ".txt", taskName);
		    string commandLine = String.Format("ProcessTaskData.exe {0} {1}", BlobPath + taskName + ".txt", 3);
		    ICloudTask taskToAdd = new CloudTask(taskName, commandLine);
		    taskToAdd.ResourceFiles = new List<IResourceFile>();
		    taskToAdd.ResourceFiles.Add(programFile);
		    taskToAdd.ResourceFiles.Add(supportFile);
		    job.AddTask(taskToAdd);
		}
		job.Commit();

3. Speichern Sie das Programm, und führen Sie es anschließend aus. Folgendes sollte angezeigt werden:

		Listing Pools
		=================
		Pool: gettingstarted State:Active
		Created pool gettingstarted. Press <Enter> to continue.

		Workitem successfully added. Press <Enter> to continue.

		Listing Workitems
		=================
		Workitem: yidingz20141106-132140 State:Active
		Press <Enter> to continue.

		The tasks completed successfully. Terminating the workitem...
		Task taskdata1 says:
		can 3
		you 3
		and 3

		Task taskdata2 says:
		and 5
		application 3
		the 3

		Task taskdata3 says:
		a 5
		and 5
		to 3

###Vorgehensweise: Löschen des Pools und das Workitem
Nach der Verarbeitung des Arbeitselements können Sie Ressourcen freigeben, indem Sie den Pool und das Arbeitselement löschen.

####Löschen des Pools
1.	Fügen Sie den folgenden Code am Ende von "Main" hinzu, der den Pool löscht:

		using (IPoolManager pm = client.OpenPoolManager())
		{
		   pm.DeletePool(PoolName);
		}
		Console.WriteLine("Pool successfully deleted.");
		Console.ReadLine();
2.	Speichern Sie das Programm, und führen Sie es anschließend aus.


####Löschen des Arbeitselements
1.	Fügen Sie den folgenden Code zu "Main" hinzu, der das Arbeitselement löscht:

		using (IWorkItemManager wm = client.OpenWorkItemManager())
		{
		   wm.DeleteWorkItem(WorkItemName);
		}
		Console.WriteLine("Workitem successfully deleted.");
		Console.ReadLine();
2.	Speichern Sie das Programm, und führen Sie es anschließend aus.

###<a name="tutorial1_storage"></a>Anhang: Erstellen einer Azure Storage-Konto
Bevor Sie den Code in diesem Lernprogramm ausführen können, benötigen Sie Zugriff auf ein Speicherkonto in Azure.

1.	Melden Sie sich beim [Azure-Verwaltungsportal](http://manage.windowsazure.com/) an.
2.	Klicken Sie am unteren Rand des Navigationsbereichs, **NEW**. ![][1]
3.	Klicken Sie auf **DATA SERVICES**, dann **Speicher**, und klicken Sie dann auf **SCHNELLERFASSUNG**. ![][2]

4.	In **URL**, geben Sie einen Unterdomänennamen ein, die im URI für das Speicherkonto verwenden. Der Eintrag kann drei bis 24 Kleinbuchstaben und Zahlen enthalten. Dieser Wert wird der Hostname im URI, der zum Adressieren von Blob-, Warteschlangen- oder Tabellenspeicherressourcen für das Abonnement verwendet wird.
5.	Wählen Sie eine **Standort/AFFINITÄTSGRUPPE** den Speicher zu suchen.
6.	Optional können Sie die Georeplikation aktivieren.
7.	Klicken Sie auf **SPEICHERKONTO ERSTELLEN**.  

Weitere Informationen zu den Azure-Speicher, finden Sie unter [Verwendung von Azure-Blob-Speicherdienst in .NET](http://azure.microsoft.com/develop/net/how-to-guides/blob-storage/).


##<a name="tutorial2"></a>Lernprogramm 2: Azure Batch Apps Library für .NET
In diesem Lernprogramm wird veranschaulicht, wie Sie mithilfe des Batch Apps-Diensts von Azure Batch Arbeitsauslastungen parallel verarbeiten.

Batch Apps ist ein Feature von Azure Batch, das eine anwendungsorientierte Methode zur Verwaltung und Ausführung von Batch-Arbeitsauslastungen bietet. Mithilfe des Batch Apps-SDKs können Sie Pakete erstellen, um eine Anwendung für Batch zu aktivieren, und diese Pakete in Ihrem eigenen Konto bereitstellen oder für andere Batch-Benutzer zur Verfügung stellen. Batch stellt auch die Datenverwaltung, Auftragsüberwachung, integrierte Diagnose und Protokollierung sowie die Unterstützung für Abhängigkeiten zwischen Aufgaben bereit. Es enthält außerdem ein Verwaltungsportal, in dem Sie Aufträge verwalten, Protokolle anzeigen und Ausgaben herunterladen können, ohne einen eigenen Client schreiben zu müssen.

Im Batch Apps-Szenario schreiben Sie mithilfe des Batch Apps Cloud-SDKs Code, um Aufträge in parallele Aufgaben zu unterteilen, sämtliche Abhängigkeiten zwischen diesen Aufgaben zu beschreiben und anzugeben, wie die einzelnen Aufgaben ausgeführt werden. Dieser Code wird im Batch-Konto bereitgestellt. Clients können dann Aufträge ausführen, indem sie einfach die Art des Auftrags und die Eingabedateien für eine REST-API angeben.

>[AZURE.NOTE]Um dieses Lernprogramm abzuschließen, benötigen Sie ein Azure-Konto. Sie können ein kostenloses Textkonto in wenigen Minuten erstellen. Ausführliche Informationen finden Sie unter [Kostenlose Azure-Testversion](http://azure.microsoft.com/pricing/free-trial/). Mithilfe von NuGet sowohl erhalten die <a href="http://www.nuget.org/packages/Microsoft.Azure.Batch.Apps.Cloud/">Batch Apps Cloud</a> Assembly und die <a href="http://www.nuget.org/packages/Microsoft.Azure.Batch.Apps/">Batch Apps Client</a> Assembly. Nachdem Sie das Projekt in Visual Studio erstellen, mit der rechten Maustaste des Projekts im **Projektmappen-Explorer** und wählen Sie **NuGet-Pakete verwalten**. Sie können auch die Visual Studio-Erweiterung für die Batch-Apps, die eine Projektvorlage zum Cloud-Anwendungen und die Möglichkeit zur Bereitstellung einer Anwendung umfasst herunterladen <a href="https://visualstudiogallery.msdn.microsoft.com/8b294850-a0a5-43b0-acde-57a07f17826a">hier</a> oder über die Suche nach **Batch Apps** in Visual Studio über das Menüelement Erweiterungen und Updates. Sie finden auch <a href="https://go.microsoft.com/fwLink/?LinkID=512183&clcid=0x409">End-to-End-Beispiele auf MSDN.</a>
>

###Grundlagen zu Azure Batch Apps
Batch ist für die Arbeit mit vorhandenen rechenintensiven Anwendungen konzipiert. Es nutzt den vorhandenen Anwendungscode und wird in einer dynamischen, virtualisierten und universellen Umgebung ausgeführt. Damit eine Anwendung mit Batch Apps verwendet werden kann, müssen einige Vorkehrungen getroffen werden:

1.	Bereiten Sie eine ZIP-Datei der vorhandenen ausführbaren Dateien der Anwendung vor – dieselben ausführbaren Dateien, die auf einer herkömmlichen Serverfarm oder einem Cluster ausgeführt würden – sowie alle erforderlichen unterstützenden Dateien. Diese ZIP-Datei wird dann mithilfe des Verwaltungsportals oder der REST-API in das Batch-Konto hochgeladen.
2.	Erstellen Sie eine ZIP-Datei der "Cloud-Assemblys", die die Arbeitsauslastungen an die Anwendung senden, und laden Sie diese dann über das Verwaltungsportal oder die REST-API hoch. Eine Cloud-Assembly enthält zwei Komponenten, die für das Cloud-SDK erstellt werden:
	1.	Auftragsteilung – Unterteilt den Auftrag in Aufgaben, die unabhängig voneinander verarbeitet werden können. Beispielsweise würde die Auftragsteilung in einem Animationsszenario einen Filmauftrag in Einzelbilder unterteilen.
	2.	Aufgabenprozessor – Ruft die ausführbare Datei der Anwendung für eine bestimmte Aufgabe auf. In einem Animationsszenario würde der Aufgabenprozessor z. B. ein Rendering-Programm aufrufen, um das durch eine Aufgabe angegebene Einzelbild zu verarbeiten.
3.	Bieten Sie eine Möglichkeit, um Aufträge an die aktivierte Anwendung in Azure zu übermitteln. Hierbei kann es sich um ein Plug-In für die Benutzeroberfläche Ihrer Anwendung oder um ein Webportal bzw. sogar einen unbeaufsichtigten Dienst als Bestandteil der Ausführungspipeline handeln. Finden Sie unter der <a href="https://go.microsoft.com/fwLink/?LinkID=512183&clcid=0x409">Beispiele</a> auf der MSDN-Beispiele.



###Schlüsselbegriffe von Batch Apps
Das Programmierungs- und Nutzungsmodell von Batch Apps stützt sich auf die folgenden Schlüsselbegriffe:

####Aufträge
Ein **Auftrag** eines Teils der Arbeit, die vom Benutzer gesendet wird. Wenn ein Auftrag gesendet wird, gibt der Benutzer den Typ des Auftrags, alle Einstellungen für diesen Auftrag sowie die für den Auftrag erforderlichen Daten an. Diese Details können entweder über die aktivierte Implementierung im Auftrag des Benutzers ausgearbeitet werden oder der Benutzer stellt diese Informationen explizit über den Client bereit. Ein Auftrag besitzt Ergebnisse, die zurückgegeben werden. Jeder Auftrag verfügt über eine primäre Ausgabe und eine optionale Vorschauausgabe. Aufträge können bei Bedarf auch zusätzliche Ausgaben zurückgeben.

####Aufgaben
Ein **Aufgabe** ist eine Stück Arbeit als Teil eines Auftrags ausgeführt werden. Wenn ein Benutzer einen Auftrag übermittelt, wird dieser in kleinere Aufgaben unterteilt. Der Dienst verarbeitet diese einzelnen Aufgaben anschließend und fasst die Aufgabenergebnisse zu einer Gesamtausgabe des Auftrags zusammen. Die Art der Aufgaben ist vom Typ des Auftrags abhängig. Die Auftragsteilung definiert, wie ein Auftrag in Aufgaben aufgegliedert wird. Dabei wird sie von den Informationen gelenkt, welche Arbeitssegmente die Anwendung verarbeiten soll. Aufgabenausgaben können auch einzeln heruntergeladen werden und sind in einigen Fällen möglicherweise hilfreich, wenn z. B. ein Benutzer einzelne Aufgaben eines Animationsauftrags herunterladen möchte.

####Zusammenführen von Aufgaben
Ein **Merge-Vorgang** ist eine besondere Art von Vorgängen, die die Ergebnisse der einzelnen Vorgänge in die endgültige Auftragsergebnisse assembliert. Für einen Auftrag zur Filmdarstellung führt die Zusammenführungsaufgabe möglicherweise die verarbeiteten Einzelbilder zu einem Film zusammen oder komprimiert alle verarbeiteten Einzelbilder in einer einzelnen Datei. Jeder Auftrag verfügt über eine Zusammenführungsaufgabe, auch wenn keine eigentliche "Zusammenführung" erforderlich ist.

####Dateien
Ein **Datei** ein Teil der Daten, die als Eingabe für einen Auftrag verwendet wird. Einem Auftrag muss keine Eingabedatei zugeordnet sein, es können ihm aber auch eine oder mehrere Eingabedateien zugewiesen werden. Dieselbe Datei kann für einen Film Rendern Auftrag z. B. in mehrere Aufträge auch verwendet werden, die Dateien möglicherweise Texturen, Modelle usw.. Für einen Data Analysis-Auftrag möglicherweise Dateien eine Reihe von Beobachtungen oder Messungen.

###Aktivieren der Cloud-Anwendung
Ihre Anwendung muss ein statisches Feld oder eine Eigenschaft enthalten, das bzw. die alle Details der Anwendung enthält. Darin sind der Name der Anwendung und der Auftragstyp bzw. die Auftragstypen angegeben, die von der Anwendung behandelt werden. Dies wird bei der Verwendung der Vorlage im SDK bereitgestellt, die über die Visual Studio Gallery heruntergeladen werden kann.

Es folgt ein Beispiel für die Deklaration einer Cloud-Anwendung für eine parallele Arbeitsauslastung:

	public static class ApplicationDefinition
	{
	    public static readonly CloudApplication App = new ParallelCloudApplication
	    {
	        ApplicationName = "ApplicationName",
	        JobType = "ApplicationJobType",
	        JobSplitterType = typeof(MyJobSplitter),
	        TaskProcessorType = typeof(MyTaskProcessor),
	    };
	}

####Implementieren von Auftragsteilung und Aufgabenprozessor
Für äußerst parallele Arbeitsauslastungen müssen Sie eine Auftragsteilung und einen Aufgabenprozessor implementieren.

####Implementieren von "JobSplitter.SplitIntoTasks"
Die "SplitIntoTasks"-Implementierung muss eine Folge von Aufgaben zurückgeben. Jede Aufgabe stellt eine separate Arbeitskomponente dar, die für die Verarbeitung durch einen Serverknoten in eine Warteschlange gestellt wird. Jede Aufgabe muss eigenständig und mit allen Informationen eingerichtet sein, die für den Aufgabenprozessor erforderlich sind.

Die von der Auftragsteilung angegebenen Aufgaben werden als "TaskSpecifier"-Objekte dargestellt. "TaskSpecifier" verfügt über eine Reihe von Eigenschaften, die Sie vor der Rückgabe der Aufgabe festlegen können.


-	"TaskIndex" wird ignoriert, ist aber für Aufgabenprozessoren verfügbar. Dadurch können Sie einen Index an den Aufgabenprozessor übergeben. Wenn Sie keinen Index übergeben müssen, muss diese Eigenschaft nicht festgelegt werden.
-	Parameter sind standardmäßig eine leere Sammlung. Sie können etwas zur Sammlung hinzufügen oder sie durch eine neue Sammlung ersetzen. Sie können Einträge aus der Auftragsparametersammlung mithilfe der "WithJobParameters"- oder "WithAllJobParameters"-Methode kopieren.  


"RequiredFiles" stellt standardmäßig eine leere Sammlung dar. Sie können etwas zur Sammlung hinzufügen oder sie durch eine neue Sammlung ersetzen. Sie können Einträge aus der Auftragsdateiensammlung mithilfe der "RequiringJobFiles"- oder "RequiringAllJobFiles"-Methode kopieren.

Sie können eine Aufgabe angeben, die von einer bestimmten anderen Aufgabe abhängig ist. Zu diesem Zweck legen Sie die "TaskSpecifier.DependsOn"-Eigenschaft fest, und übergeben die ID der Aufgabe, von der diese abhängig ist:

	new TaskSpecifier {
	    DependsOn = TaskDependency.OnId(5)
	}

Die Aufgabe wird nicht ausgeführt, bis die Ausgabe der Aufgabe verfügbar ist, zu der die Abhängigkeit besteht.

Sie können auch angeben, dass eine ganze Gruppe von Aufgaben nicht mit der Verarbeitung beginnen soll, bis eine andere Gruppe vollständig abgeschlossen ist. In diesem Fall können Sie die "TaskSpecifier.Stage"-Eigenschaft festlegen. Aufgaben mit einem angegebenen Phasenwert beginnen nicht mit der Verarbeitung, bis alle Aufgaben mit niedrigeren Phasenwerten abgeschlossen wurden. Aufgaben mit Phase 3 beginnen z. B. nicht mit der Verarbeitung, bis alle Aufgaben mit Phase 0, 1 oder 2 beendet wurden. Phasen müssen bei 0 beginnen, die Reihenfolge der Phasen darf keine Lücken aufweisen und "SplitIntoTasks" muss Aufgaben in Phasenreihenfolge zurückgeben. Es ist z. B. ein Fehler, eine Aufgabe mit Phase 0 nach einer Aufgabe mit Phase 1 zurückzugeben.

Jeder parallele Auftrag endet mit einer speziellen Aufgabe, die als "Zusammenführungsaufgabe" bezeichnet wird. Der Auftrag der Zusammenführungsaufgabe ist die Zusammenführung der Ergebnisse der parallelen Verarbeitungsaufgaben zu einem Endergebnis. Batch Apps erstellt die Zusammenführungsaufgabe automatisch für Sie.

In seltenen Fällen empfiehlt es sich, die Zusammenführungsaufgabe explizit zu steuern, z. B. zum Anpassen ihrer Parameter. In diesem Fall können Sie die Zusammenführungsaufgabe angeben, indem Sie ein "TaskSpecifier" mit "IsMerge"-Eigenschaft zurückgeben, die auf "true" gesetzt ist. Dadurch wird die automatische Zusammenführungsaufgabe außer Kraft gesetzt. Wenn Sie eine explizite Zusammenführungsaufgabe erstellen:

-	Sie können möglicherweise nur eine explizite Zusammenführungsaufgabe erstellen.
-	Es muss sich um die letzte Aufgabe in der Sequenz handeln.
-	"IsMerge" muss auf "true" festgelegt sein, während "IsMerge" für jede andere Aufgabe auf "false" gesetzt sein muss.  


Beachten Sie jedoch, dass Sie die Zusammenführungsaufgabe normalerweise nicht explizit erstellen müssen.

Der folgende Code veranschaulicht eine einfache Implementierung von "SplitIntoTasks".

	protected override IEnumerable<TaskSpecifier> SplitIntoTasks(
	    IJob job,
	    JobSplitSettings settings)
	{
	    int start = Int32.Parse(job.Parameters["startIndex"]);
	    int end = Int32.Parse(job.Parameters["endIndex"]);
	    int count = (end - start) + 1;

	    // Processing tasks
	    for (int i = 0; i < count; ++i)
	    {
	        yield return new TaskSpecifier
	        {
	            TaskIndex = start + i,
	        }.RequiringAllJobFiles();
	    }
	}
####Implementieren von "ParallelTaskProcessor.RunExternalTaskProcess"

"RunExternalTaskProcess" wird für jede nicht zusammenführende Aufgabe aufgerufen, die von der Auftragsteilung zurückgegeben wird. Es sollte Ihre Anwendung mit den entsprechenden Argumenten aufrufen und eine Sammlung von Ausgaben zurückgeben, die für die spätere Verwendung aufbewahrt werden müssen.

Das folgende Fragment zeigt, wie Sie ein Programm namens "application.exe" aufrufen. Beachten Sie, dass Sie die Hilfsmethode "ExecutablePath" verwenden können, um absolute Dateipfade zu erstellen.

Die "ExternalProcess"-Klasse im Cloud-SDK bietet eine nützliche Hilfslogik zum Ausführen der ausführbaren Dateien der Anwendung. "ExternalProcess" kann sich um Abbrüche, die Übersetzung von Exitcodes in Ausnahmen, die Erfassung der Standardausgabe und von Standardfehlern sowie die Einrichtung von Umgebungsvariablen kümmern. Sie können darüber hinaus die .NET-Prozessklasse direkt zum Ausführen von Programmen verwenden, wenn Sie dies bevorzugen.

Die "RunExternalTaskProcess"-Methode gibt ein "TaskProcessResult" zurück, das eine Liste von Ausgabedateien enthält. Diese muss mindestens alle für die Zusammenführung erforderlichen Dateien enthalten. Sie können optional auch Protokolldateien, Vorschaudateien und temporäre Dateien zurückgeben (z. B. zu Diagnosezwecken, wenn die Aufgabe fehlgeschlagen ist). Beachten Sie, dass die Methode die Dateipfade, nicht den Dateiinhalt zurückgibt.

Jede Datei muss mit dem Typ der Ausgabe identifiziert werden, den sie enthält: Ausgabe (d. h. Teil der letztendlichen Auftragsausgabe), Vorschau, Protokoll oder Zwischenergebnis. Diese Werte stammen von der "TaskOutputFileKind"-Enumeration. Das folgende Fragment gibt eine einzelne Aufgabenausgabe und keine Vorschau oder Protokoll zurück. Die "TaskProcessResult.FromExternalProcessResult"-Methode vereinfacht die gängigen Szenarien zum Erfassen von Exitcode, Prozessorausgaben und Ausgabedateien über ein Befehlszeilenprogramm:

Der folgende Code veranschaulicht eine einfache Implementierung von "ParallelTaskProcessor.RunExternalTaskProcess".

	protected override TaskProcessResult RunExternalTaskProcess(
	    ITask task,
	    TaskExecutionSettings settings)
	{
	    var inputFile = LocalPath(task.RequiredFiles[0].Name);
	    var outputFile = LocalPath(task.TaskId.ToString() + ".jpg");

	    var exePath = ExecutablePath(@"application\application.exe");
	    var arguments = String.Format("-in:{0} -out:{1}", inputFile, outputFile);

	    var result = new ExternalProcess {
	        CommandPath = exePath,
	        Arguments = arguments,
	        WorkingDirectory = LocalStoragePath,
	        CancellationToken = settings.CancellationToken
	    }.Run();

	    return TaskProcessResult.FromExternalProcessResult(result, outputFile);
	}
####Implementieren von "ParallelTaskProcessor.RunExternalMergeProcess"

Dies wird für eine Zusammenführungsaufgabe aufgerufen. Es sollte die Anwendung aufrufen, um Ausgaben der vorherigen Aufgaben auf eine für die Anwendung angemessene Weise zu kombinieren, und die kombinierte Ausgabe zurückgeben.

Die Implementierung von "RunExternalMergeProcess" ähnelt "RunExternalTaskProcess", jedoch mit folgender Ausnahme:

-	"RunExternalMergeProcess" nutzt die Ausgaben vorheriger Aufgaben anstelle von Benutzereingabedateien. Sie sollten daher die Namen der Dateien, die Sie verarbeiten möchten, auf Basis der Aufgaben-ID und nicht über die "Task.RequiredFiles"-Sammlung ermitteln.
-	"RunExternalMergeProcess" gibt eine "JobOutput"-Datei und optional eine "JobPreview"-Datei zurück.


Der folgende Code veranschaulicht eine einfache Implementierung von "ParallelTaskProcessor.RunExternalMergeProcess".

	protected override JobResult RunExternalMergeProcess(
	    ITask mergeTask,
	    TaskExecutionSettings settings)
	{
	    var outputFile =  "output.zip";

	    var exePath =  ExecutablePath(@"application\application.exe");
	    var arguments = String.Format("a -application {0} *.jpg", outputFile);

	    new ExternalProcess {
	        CommandPath = exePath,
	        Arguments = arguments,
	        WorkingDirectory = LocalStoragePath
	    }.Run();

	    return new JobResult {
	        OutputFile = outputFile
	    };
	}

###Übermitteln von Aufträgen an Batch Apps
Ein Auftrag beschreibt eine auszuführende Arbeitsauslastung und muss alle Informationen zur Arbeitsauslastung mit Ausnahme der Dateiinhalte enthalten. Der Auftrag enthält z. B. Konfigurationseinstellungen, die vom Client zur Auftragsteilung und anschließend zu den Aufgaben gelangen. Die auf MSDN bereitgestellten Beispiele sind Beispiele für die Übermittlung von Aufträgen. Sie stellen mehrere Clients bereit, einschließlich eines Webportal- und Befehlszeilenclients.


<!--Anchors-->


<!--Image references-->
[1]: ./media/batch-dotnet-get-started/batch-dotnet-get-started-01.jpg
[2]: ./media/batch-dotnet-get-started/batch-dotnet-get-started-02.jpg
[3]: ./media/batch-dotnet-get-started/batch-dotnet-get-started-03.jpg
[4]: ./media/batch-dotnet-get-started/batch-dotnet-get-started-04.jpg

<!---HONumber=GIT-SubDir-->