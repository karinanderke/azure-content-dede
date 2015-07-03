<properties 
	pageTitle="Erstellen von Datenstichproben im Azure-Blob-Speicher | Azure" 
	description="Erstellen von Datenstichproben im Azure-Blob-Speicher" 
	services="machine-learning" 
	documentationCenter="" 
	authors="sunliangms,fashah,msolhab" 
	manager="paulettm" 
	editor="cgronlun" />

<tags 
	ms.service="machine-learning" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="02/18/2015" 
	ms.author="sunliangms,fashah,msolhab,garye" /> 

#<a name="heading"></a>Erstellen von Datenstichproben im Azure-Blob-Speicher

Dieses Dokument behandelt das Erstellen von Stichproben aus Daten im Azure Blob-Speicher durch programmgesteuertes Herunterladen, um dann mit Python-Code Stichproben zu erstellen. Befolgen Sie dazu die folgenden Schritte:

1. Laden Sie die Daten aus dem Azure-Blob-Speicher mithilfe des Blob-Diensts aus dem folgenden Python-Beispielcode herunter: 

	    from azure.storage import BlobService
    	import tables
    	
		STORAGEACCOUNTNAME= <storage_account_name>
		STORAGEACCOUNTKEY= <storage_account_key>
		LOCALFILENAME= <local_file_name>		
		CONTAINERNAME= <container_name>
		BLOBNAME= <blob_name>

    	#download from blob
    	t1=time.time()
    	blob_service=BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)
    	blob_service.get_blob_to_path(CONTAINERNAME,BLOBNAME,LOCALFILENAME)
    	t2=time.time()
    	print(("It takes %s seconds to download "+blobname) % (t2 - t1))

2. Lesen Sie die Daten aus der heruntergeladenen Datei in ein Pandas-DataFrame ein.

		import pandas as pd

	    #directly ready from file on disk
    	dataframe_blobdata = pd.read_csv(LOCALFILE)

3. Erstellen Sie mit `random.choice` aus  `numpy` wie folgt Stichproben:

	    # A 1 percent sample
    	sample_ratio = 0.01 
    	sample_size = np.round(dataframe_blobdata.shape[0] * sample_ratio)
    	sample_rows = np.random.choice(dataframe_blobdata.index.values, sample_size)
    	dataframe_blobdata_sample = dataframe_blobdata.ix[sample_rows]

	Sie können nun mit dem DataFrame von oben mit der 1-%-Stichprobe weitere Datensuchvorgänge durchführen und Funktionen verarbeiten.

##<a name="heading"></a>Herstellen einer Verbindung mit Azure Machine Learning

Mit dem folgenden Beispielcode können Sie ein Downsampling der Daten durchführen und diese direkt in Azure ML verwenden:

1. Schreiben Sie den DataFrame in eine lokale Datei:

		dataframe.to_csv(os.path.join(os.getcwd(),LOCALFILENAME), sep='\t', encoding='utf-8', index=False)

2. Laden Sie die lokale Datei mit folgendem Beispielcode in ein Azure-Blob hoch:

		from azure.storage import BlobService
    	import tables

		STORAGEACCOUNTNAME= <storage_account_name>
		LOCALFILENAME= <local_file_name>
		STORAGEACCOUNTKEY= <storage_account_key>
		CONTAINERNAME= <container_name>
		BLOBNAME= <blob_name>

	    output_blob_service=BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)    
	    localfileprocessed = os.path.join(os.getcwd(),LOCALFILENAME) #assuming file is in current working directory
	    
	    try:
	   
	    #perform upload
	    output_blob_service.put_block_blob_from_path(CONTAINERNAME,BLOBNAME,localfileprocessed)
	    
	    except:	        
		    print ("Something went wrong with uploading to the blob:"+ BLOBNAME)

3. Lesen Sie die Daten wie in der folgenden Abbildung dargestellt mit dem *Reader Module* in Azure ML aus dem Azure-Blob aus:
 
![reader blob][1]

[1]: ./media/machine-learning-data-science-sample-data-blob/reader_blob.png

<!--HONumber=49--> 