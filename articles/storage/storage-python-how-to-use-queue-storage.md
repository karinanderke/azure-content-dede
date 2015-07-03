<properties 
	pageTitle="Verwenden des Warteschlangenspeichers mit Python | Microsoft Azure" 
	description="Erfahren Sie, wie Sie mit dem Azure-Warteschlangendienst mit Python Warteschlangen erstellen und löschen sowie Nachrichten einfügen, abrufen und löschen können." 
	services="storage" 
	documentationCenter="python" 
	authors="huguesv" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="storage" 
	ms.workload="storage" 
	ms.tgt_pltfrm="na" 
	ms.devlang="python" 
	ms.topic="article" 
	ms.date="03/11/2015" 
	ms.author="huvalo"/>

# Verwenden des Warteschlangenspeichers mit Python

[AZURE.INCLUDE [storage-selector-queue-include](../../includes/storage-selector-queue-include.md)]

## Übersicht

In diesem Leitfaden wird die Durchführung häufiger Szenarien mit dem Azure-Warteschlangenspeicherdienst demonstriert. Die Beispiele sind in Python geschrieben und verwenden das [Python Azure-Paket][]. Die aufgeführten Szenarien umfassen das **Einfügen**, **Einsehen**,
**Abrufen** und **Löschen** von Warteschlangennachrichten sowie das **Erstellen und Löschen von Warteschlangen**. Weitere Informationen zu Warteschlangen finden Sie im Abschnitt [Nächste Schritte][].

[AZURE.INCLUDE [storage-queue-concepts-include](../../includes/storage-queue-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]


> [AZURE.NOTE] Informationen zur Installation von Python bzw. des [Python Azure-Pakets][] finden Sie im [Python-Installationshandbuch](../python-how-to-install.md).

## Vorgehensweise: Erstellen einer Warteschlange

Das Objekt **QueueService** ermöglicht es Ihnen, mit Warteschlangen zu arbeiten. Der folgende Code erstellt ein **QueueService**-Objekt. Fügen Sie am Anfang jeder Python-Datei, in der Sie programmgesteuert auf Azure-Speicher zugreifen möchten, den folgenden Code hinzu:

	from azure.storage import QueueService

Der folgende Code erstellt ein **QueueService**-Objekt unter Verwendung des Speicherkontonamens und Kontoschlüssels. Ersetzen Sie  'myaccount' und  'mykey' durch das tatsächliche Konto und den tatsächlichen Schlüssel.

	queue_service = QueueService(account_name='myaccount', account_key='mykey')

	queue_service.create_queue('taskqueue')


## Vorgehensweise: Einfügen einer Nachricht in eine Warteschlange

Zum Einfügen einer Nachricht in eine Warteschlange verwenden Sie die **put_message**-Methode,um eine neue Nachricht zu erstellen. Fügen Sie sie dann der Warteschlange hinzu.

	queue_service.put_message('taskqueue', 'Hello World')


## Vorgehensweise: Einsehen der nächsten Nachricht

Sie können einen Blick auf die Nachricht am Anfang einer Warteschlange werfen, ohne sie aus der Warteschlange zu entfernen, indem Sie die Methode **peek_messages** aufrufen. Standardmäßig ruft
**peek_messages** nur eine einzige Nachricht ab.

	messages = queue_service.peek_messages('taskqueue')
	for message in messages:
		print(message.message_text)


## Vorgehensweise: Entfernen der nächsten Nachricht aus der Warteschlange

Dieser Code entfernt eine Nachricht in zwei Schritten aus der Warteschlange. Wenn Sie
**get_messages** aufrufen, wird standardmäßig die nächste Nachricht aus der Warteschlange abgerufen. Eine von **get_messages** zurückgegebene Nachricht ist für anderen Code nicht mehr sichtbar, der Nachrichten aus dieser Warteschlange liest. Standardmäßig bleibt die Nachricht 30 Sekunden lang unsichtbar. Um die Nachricht endgültig aus der Warteschlange zu entfernen, müssen Sie auch **delete_message** aufrufen. Dieser zweistufige Prozess zum Entfernen von Nachrichten stellt sicher, dass eine andere Codeinstanz dieselbe Nachricht erneut abrufen kann, falls die Verarbeitung aufgrund eines Hardware- oder Softwarefehlers fehlschlägt. Der Code ruft **delete_message** direkt nach der Verarbeitung der Nachricht auf.

	messages = queue_service.get_messages('taskqueue')
	for message in messages:
		print(message.message_text)
		queue_service.delete_message('taskqueue', message.message_id, message.pop_receipt)


## Vorgehensweise: Ändern des Inhalts von Nachrichten in der Warteschlange

Sie können den Inhalt einer Nachricht vor Ort in der Warteschlange ändern. Wenn die Nachricht eine Arbeitsaufgabe darstellt, können Sie diese Funktion verwenden, um den Status der Aufgabe zu aktualisieren. Der folgende Code verwendet die **update_message**-Methode zum Aktualisieren einer Nachricht.

	messages = queue_service.get_messages('taskqueue')
	for message in messages:
		queue_service.update_message('taskqueue', message.message_id, 'Hello World Again', message.pop_receipt, 0)

## Vorgehensweise: Zusätzliche Optionen für das Entfernen von Nachrichten aus der Warteschlange

Es gibt zwei Möglichkeiten, wie Sie das Abrufen von Nachrichten aus der Warteschlange anpassen können.
Erstens können Sie einen Nachrichtenstapel abrufen (bis zu 32). Zweitens können Sie das Unsichtbarkeits-Zeitlimit verkürzen oder verlängern, sodass der Code mehr oder weniger Zeit zur vollständigen Verarbeitung jeder Nachricht benötigt. Das folgende Beispiel verwendet die
**get_messages**-Methode, um 16 Nachrichten mit einem Aufruf abzurufen. Anschließend wird jede Nachricht mithilfe einer "for"-Schleife verarbeitet. Außerdem wird das Unsichtbarkeits-Zeitlimit auf fünf Minuten pro Nachricht festgelegt.

	messages = queue_service.get_messages('taskqueue', numofmessages=16, visibilitytimeout=5*60)
	for message in messages:
		print(message.message_text)
		queue_service.delete_message('taskqueue', message.message_id, message.pop_receipt)

## Vorgehensweise: Abrufen der Warteschlangenlänge

Sie können die Anzahl der Nachrichten in einer Warteschlange schätzen lassen. Die
**get_queue_metadata**-Methode fordert den Warteschlangendienst auf, Metadaten über die Warteschlange zurückzugeben. **x-ms-approximate-messages-count** sollte als Index für das zurückgegebene Wörterbuch verwendet werden, um die Anzahl zu finden.
Die Anzahl ist nur ein ungefährer Wert, da seit der Antwort des Warteschlangendienstes möglicherweise bereits Nachrichten hinzugefügt oder gelöscht wurden.

	queue_metadata = queue_service.get_queue_metadata('taskqueue')
	count = queue_metadata['x-ms-approximate-messages-count']

## Vorgehensweise: Löschen einer Warteschlange

Zum Löschen einer Warteschlange und aller darin enthaltenen Nachrichten rufen Sie die
**delete_queue**-Methode auf.

	queue_service.delete_queue('taskqueue')

## Nächste Schritte

Nachdem Sie sich nun mit den Grundlagen des Warteschlangenspeichers vertraut gemacht haben, folgen Sie diesen Links, um zu erfahren, wie komplexere Speicheraufgaben ausgeführt werden.

-   Weitere Informationen finden Sie in der MSDN-Referenz: [Speichern von und Zugreifen auf Daten in Azure][]
-   Besuchen Sie den [Azure Storage-Teamblog][].

[Speichern von und Zugreifen auf Daten in Azure]: http://msdn.microsoft.com/library/azure/gg433040.aspx
[Azure Storage-Teamblog]: http://blogs.msdn.com/b/windowsazurestorage/
[Python Azure-Pakets]: https://pypi.python.org/pypi/azure  

<!--HONumber=49--> 