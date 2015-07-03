<properties 
	pageTitle="Checkliste zu Leistung und Skalierbarkeit von Microsoft Azure Storage" 
	description="Erforderlich" 
	services="storage" 
	documentationCenter="" 
	authors="tamram" 
	manager="adinah" 
	editor=""/>

<tags 
	ms.service="storage" 
	ms.workload="storage" 
	ms.tgt_pltfrm="na" 
	ms.devlang="dotnet" 
	ms.topic="article" 
	ms.date="10/08/2014" 
	ms.author="tamram"/>

#Checkliste zu Leistung und Skalierbarkeit von Microsoft Azure Storage
##Einführung
Seit der Veröffentlichung des Microsoft Azure-Speicherdiensts hat Microsoft eine Reihe bewährter Vorgehensweisen zur effektiven Verwendung dieser Dienste entwickelt. In diesem Artikel werden die wichtigsten davon in einer "Checkliste" zusammengefasst. Dieser Artikel soll Anwendungsentwickler bei der Verwendung bewährter Vorgehensweisen für Azure Storage unterstützen und ihnen helfen, weitere bewährte Vorgehensweisen kennen zu lernen, die sie in Erwägung ziehen sollten. Es wird jedoch nicht jede mögliche Leistungs- und Skalierbarkeitsoptimierung in diesem Artikel abgedeckt; begrenzt anwendbare Methoden oder solche mit geringer Auswirkung sind ausgeschlossen.  Insofern sich das Anwendungsverhalten während des Entwurfs vorhersagen lässt, sollten diese Vorgehensweisen von Anfang an berücksichtigt werden, um spätere Leistungsprobleme zu vermeiden.  

Jeder Anwendungsentwickler, der Azure Storage verwendet, sollte sich Zeit für diesen Artikel nehmen und überprüfen, ob seine Anwendung den unten aufgeführten bewährten Vorgehensweisen entspricht.  

##Checkliste
Dieser Artikel unterteilt die bewährten Vorgehensweisen in Gruppen, die wie folgt anwendbar sind:  

-	Alle Azure-Speicherdienste (Blobs, Tabellen, Warteschlangen und Dateien)
-	Blobs
-	Tabellen
-	Warteschlangen  

Der Azure-Dateidienst befindet sich derzeit in der Vorschauphase; die bewährten Vorgehensweisen werden künftig hier hinzugefügt.  

|Fertig|	Bereich|	Kategorie|	Frage
|----|------|-----------|-----------
||Alle Dienste|	Skalierbarkeitsziele|[Ist Ihre Anwendung so konzipiert, dass die Skalierbarkeitsziele nicht überschritten werden?](#subheading1)
||Alle Dienste|	Netzwerk|	[Haben die clientseitigen Geräte genügend Bandbreite und niedrige Latenz, um die erforderliche Leistung zu erzielen?](#subheading2)
||Alle Dienste|	Netzwerk|	[Haben die clientseitigen Geräte eine ausreichend hohe Verbindungsqualität?](#subheading3)
||Alle Dienste|	Netzwerk|	[Befindet sich die Clientanwendung in der "Nähe" des Speicherkontos?](#subheading4)
||Alle Dienste|	Inhaltsverteilung|	[Verwenden Sie CDN zur Inhaltsverteilung?](#subheadin5)
||Alle Dienste|	Direkter Clientzugriff|	[Verwenden Sie SAS und CORS, um den direkten Zugriff auf Speicher statt Proxy zuzulassen?](#subheading6)
||Alle Dienste|	Caching|	[Speichert Ihre Anwendung Daten im Cache, die häufig verwendet werden und sich selten ändern?](#subheading7)
||Alle Dienste|	Caching|	[Führt Ihre Anwendung Batchaktualisierungen durch (durch clientseitige Zwischenspeicherung und anschließendes Hochladen in größeren Mengen)?](#subheading8)
||Alle Dienste|	.NET Konfiguration|	[Haben Sie Ihren Client zur Verwendung einer ausreichenden Anzahl gleichzeitiger Verbindungen konfiguriert?](#subheading9)
||Alle Dienste|	.NET Konfiguration|	[Haben Sie .NET zur Verwendung einer ausreichenden Anzahl von Threads konfiguriert?](#subheading10)
||Alle Dienste|	.NET Konfiguration|	[Verwenden Sie .NET 4.5 oder höher mit verbesserter Garbage Collection?](#subheading11)
||Alle Dienste|	Parallelität|	[Haben Sie sichergestellt, dass die Parallelität entsprechend gebunden ist, sodass weder die Clientkapazitäten noch die Skalierbarkeitsziele überlastet werden?](#subheading12)
||Alle Dienste|	Extras|	[Verwenden Sie die aktuellste Version der von Microsoft bereitgestellten Clientbibliotheken und Extras?](#subheading13)
||Alle Dienste|	Wiederholungsversuche|	[Verwenden Sie exponentiell ansteigende Wartezeiten für Wiederholungsversuche, um Fehler und Zeitüberschreitungen zu vermeiden?](#subheading14)
||Alle Dienste|	Wiederholungsversuche|	[Vermeidet Ihre Anwendung Wiederholungsversuche für nicht wiederholbare Fehler?](#subheading15)
||Blobs|	Skalierbarkeitsziele|	[Bleibt Ihre Anwendung innerhalb der Bandbreiten- oder Vorgangs-Skalierbarkeitsziele für einzelne Blobs?](#subheading16)
||Blobs|	Kopieren von Blobs|	[Kopieren Sie Blobs effizient?](#subheading17)
||Blobs|	Kopieren von Blobs|	[Verwenden Sie AzCopy für Massenkopien von Blobs?](#subheading18)
||Blobs|	Kopieren von Blobs|	[Verwenden Sie Azure Import/Export zum Übertragen von großen Datenmengen?](#subheading19)
||Blobs|	Metadaten verwenden|	[Speichern Sie häufig verwendete Metadaten für Blobs in deren Metadaten?](#subheading20)
||Blobs|	Schnelles Hochladen|	[Wenn Sie versuchen, einen Blob schnell hochzuladen, laden Sie dann Blöcke parallel hoch?](#subheading21)
||Blobs|	Schnelles Hochladen|	[Wenn Sie versuchen, mehrere Blobs schnell hochzuladen, laden Sie dann Blobs parallel hoch?](#subheading22)
||Blobs|	Blobtyp korrigieren|	[Verwenden Sie jeweils Seiten-Blobs oder Block-Blobs?](#subheading23)
||Tabellen|	Skalierbarkeitsziele|	[Nähern Sie sich den Skalierbarkeitszielen für Entitäten pro Sekunde?](#subheading24)
||Tabellen|	Konfiguration|	[Verwenden Sie JSON für Ihre Tabellenanforderungen?](#subheading25)
||Tabellen|	Konfiguration|	[Haben Sie Nagle deaktiviert, um die Leistung kleiner Anfragen zu verbessern?](#subheading26)
||Tabellen|	Tabellen und Partitionen|	[Haben Sie die Daten korrekt aufgeteilt?](#subheading27)
||Tabellen|	Hot-Partitionen|	[Vermeiden Sie Nur-Anhängen/Nur-Voranstellen-Muster?](#subheading28)
||Tabellen|	Hot-Partitionen|	[Verteilen sich Ihre Einfügungen/Aktualisierungen auf mehrere Partitionen?](#subheading29)  
||Tabellen|	Abfragebereich|	[Haben Sie Ihr Schema so konzipiert, dass in den meisten Fällen Punktabfragen verwendet werden können und Tabellenabfragen nur sparsam erfolgen?](#subheading30)
||Tabellen|	Abfragedichte|	[Erfassen Ihre Abfragen normalerweise nur Zeilen für die Rückgabe, die Ihre Anwendung auch verwenden wird?](#subheading31)
||Tabellen|	Zurückgegebene Daten einschränken|	[Verwenden Sie die Filterung, um die Rückgabe nicht benötigter Entitäten zu vermeiden?](#subheading32)
||Tabellen|	Zurückgegebene Daten einschränken|	[Verwenden Sie Projektion, um die Rückgabe nicht benötigter Eigenschaften zu vermeiden?](#subheading33)
||Tabellen|	Denormalisierung|	[Haben Sie Ihre Daten so normalisiert, dass Sie ineffiziente Abfragen oder mehrere Leseanforderungen beim Abrufen von Daten vermeiden?](#subheading34)
||Tabellen|	Einfügen/Aktualisieren/Löschen|	[Verwenden Sie Batchverarbeitung für Anforderungen, die transaktional oder gleichzeitig erfolgen, um Roundtrips zu reduzieren?](#subheading35)
||Tabellen|	Einfügen/Aktualisieren/Löschen|	[Vermeiden Sie das Abrufen von Entitäten, nur um zu bestimmen, ob Einfügen oder Aktualisieren erforderlich ist?](#subheading36)
||Tabellen|	Einfügen/Aktualisieren/Löschen|	[Haben Sie überlegt, Datenreihen, die häufig zusammen abgerufen werden, in einer einzigen Entität als Eigenschaften zu speichern statt in mehreren Entitäten?](#subheading37)
||Tabellen|	Einfügen/Aktualisieren/Löschen|	[Haben Sie überlegt, für Entitäten, die immer zusammen abgerufen werden und in Batches geschrieben werden können (z. B. Zeitreihendaten), Blobs statt Tabellen zu verwenden?](#subheading38)
||Warteschlangen|	Skalierbarkeitsziele|	[Nähern Sie sich den Skalierbarkeitszielen für Nachrichten pro Sekunde?](#subheading39)
||Warteschlangen|	Konfiguration|	[Haben Sie Nagle deaktiviert, um die Leistung kleiner Anfragen zu verbessern?](#subheading40)
||Warteschlangen|	Nachrichtengröße|	[Sind Ihre Nachrichten kompakt, um die Leistung der Warteschlange zu verbessern?](#subheading41)
||Warteschlangen|	Massenabruf|	[Rufen Sie mehrere Nachrichten mit einem einzigen "Get"-Vorgang ab?](#subheading41)
||Warteschlangen|	Abruffrequenz|	[Rufen Sie häufig genug ab, um die gefühlte Latenz der Anwendung zu reduzieren?](#subheading42)
||Warteschlangen|	UpdateMessage|	[Verwenden Sie UpdateMessage, um den Fortschritt bei der Nachrichtenverarbeitung zu speichern und so nicht die gesamten Nachrichten erneut verarbeiten zu müssen, falls ein Fehler auftritt?](#subheading43)
||Warteschlangen|	Architektur|	[Verwenden Sie Warteschlangen, um die gesamte Anwendung skalierbarer zu machen, indem Sie Arbeitsauslastungen mit langer Laufzeit aus dem kritischen Pfad heraushalten und unabhängig skalieren?](#subheading44)


##<a name="allservices"></a>Alle Dienste
In diesem Abschnitt sind bewährte Vorgehensweisen für die Verwendung von allen Azure-Speicherdiensten (Blobs, Tabellen, Warteschlangen und Dateien) aufgelistet.  

###<a name="subheading1"></a>Skalierbarkeitsziele
Jeder Azure-Speicherdienst hat Skalierbarkeitsziele für Kapazität (GB), Transaktionsrate und Bandbreite. Wenn Ihre Anwendung eines der Skalierbarkeitsziele erreicht oder überschreitet, kann es zu erhöhter Transaktionslatenz oder Drosselung kommen. Sobald ein Speicherdienst Ihre Anwendung drosselt, gibt der Dienst den Fehlercode "503 Server Busy" oder "500 Operation Timeout" für manche Speichertransaktionen zurück. In diesem Abschnitt werden die allgemeine Vorgehensweise zur Handhabung von Skalierbarkeitszielen sowie speziell die Bandbreiten-Skalierbarkeitsziele behandelt. In späteren Abschnitten zu den einzelnen Speicherdiensten werden die Skalierbarkeitsziele im jeweiligen Dienstkontext erläutert:  

-	[Blob-Bandbreite und Anforderungen pro Sekunde](#subheading16)
-	[Tabellenentitäten pro Sekunde](#subheading24)
-	[Warteschlangennachrichten pro Sekunde](#subheading39)  

####<a name="sub1bandwidth"></a>Bandbreiten-Skalierbarkeitsziele für alle Dienste
Zum Redaktionszeitpunkt betrugen die Bandbreitenziele in den USA für georedundante Speicherkonten (GRS) 10 Gigabit pro Sekunde (GBit/s) für eingehende Daten (an das Speicherkonto gesendete Daten) und 20 Gigabit pro Sekunde (GBit/s) für ausgehende Daten (vom Speicherkonto aus gesendete Daten). Für lokale redundante Speicherkonten (LRS) liegen die Grenzen höher - 20 GBit/s für eingehende und 30 GBit/s für ausgehende Daten.  Internationale Bandbreitengrenzen können niedriger sein. Informationen dazu finden Sie auf der Seite [Skalierbarkeitsziele](http://msdn.microsoft.com/library/azure/dn249410.aspx).  Weitere Informationen zu den Speicherredundanzoptionen finden Sie unter den Links zu [Nützliche Ressourcen](#sub1useful) weiter unten.  

####Vorgehensweise bei Annäherung an das Skalierbarkeitsziel
Wenn sich Ihre Anwendung den Skalierbarkeitszielen für ein Speicherkonto nähert, sollten Sie eine der folgenden Vorgehensweisen wählen:  

-	Berücksichtigen Sie die Arbeitsauslastung, aufgrund derer Ihre Anwendung das Skalierbarkeitsziel erreicht oder überschreitet. Können Sie diese anders konzipieren, um weniger Bandbreite bzw. Kapazität oder weniger Transaktionen zu verwenden?
-	Wenn eine Anwendung eines der Skalierbarkeitsziele überschreiten muss, sollten Sie mehrere Speicherkonten erstellen und die Anwendungsdaten auf mehrere Speicherkonten aufteilen. Konzipieren Sie in diesem Fall die Anwendung so, dass Sie künftig weitere Speicherkonten für den Lastenausgleich hinzufügen können. Zum Redaktionszeitpunkt konnte jedes Azure-Abonnement über bis zu 100 Speicherkonten verfügen.  Speicherkonten verursachen keine Kosten außer die Nutzung in Bezug auf gespeicherte Daten, durchgeführte Transaktionen und übertragene Daten.
-	Wenn Ihre Anwendung das Bandbreitenziel erreicht, versuchen Sie, die Daten im Client zu komprimieren, um die erforderliche Bandbreite zum Senden der Daten an den Speicherdienst zu reduzieren.  Obwohl dies Bandbreite spart und die Netzwerkleistung erhöhen kann, kann es auch negative Auswirkungen haben.  Sie sollten die Leistungsauswirkung in Bezug auf die zusätzliche Verarbeitung beim Komprimieren und Dekomprimieren der Daten im Client beobachten. Außerdem kann das Speichern komprimierter Daten die Fehlerbehebung erschweren, da es schwieriger sein kann, gespeicherte Daten mithilfe von Standardtools anzuzeigen.
-	Wenn Ihre Anwendung die Skalierbarkeitsziele erreicht, stellen Sie sicher, dass Sie exponentiell ansteigende Wartezeiten für Wiederholungsversuche verwenden (siehe [Wiederholungsversuche](#subheading14)).  Sie sollten besser sicherstellen, dass Sie sich nicht an die Skalierbarkeitsziele annähern (mit einer der oben angegebenen Methoden), aber dies sorgt ebenfalls dafür, dass die Anwendung keine schnellen Wiederholungen versucht, sodass die Drosselung schlimmer wird.  

####Nützliche Ressourcen
Die folgenden Links enthalten zusätzliche Details zu den Skalierbarkeitszielen:
-	Sie können die aktuellen Skalierbarkeitsziele auf der Seite [Ziele für Skalierbarkeit und Leistung des Azure-Speichers auf MSDN](http://msdn.microsoft.com/library/azure/dn249410.aspx) anzeigen.
-	Weitere Informationen zu Speicherredundanzoptionen erhalten Sie im Blogeintrag [Microsoft Azure-Speicher: Redundanzoptionen und Lesezugriff georedundanter Speicher](http://blogs.msdn.com/b/windowsazurestorage/archive/2013/12/11/introducing-read-access-geo-replicated-storage-ra-grs-for-windows-azure-storage.aspx). Weitere Details hierzu finden Sie auch im nächsten Abschnitt.
-	Aktuelle Informationen zu den Preisen für Azure-Dienste finden Sie unter [Azure-Preise auf einen Blick](http://azure.microsoft.com/pricing/overview/).  

###Netzwerk
Auch wenn die API-Aufrufe eine Rolle spielen, haben häufig die physischen Netzwerkeinschränkungen deutliche Auswirkungen auf die Leistung. Nachfolgend sind einige der Einschränkungen beschrieben, auf die Benutzer treffen können.  

####Client-Netzwerkkapazität
#####<a name="subheading2"></a>Durchsatz
Bei der Bandbreite liegt das Problem häufig in der Clientkapazität. Während beispielsweise ein Speicherkonto 10 GBit/s oder mehr an eingehenden Daten verarbeiten kann (siehe [Bandbreiten-Skalierbarkeitsziele](#sub1bandwidth)), beträgt die Netzwerkgeschwindigkeit in einer "kleinen" Azure-Workerrolleninstanz maximal 100 MBit/s. Größere Azure-Instanzen verfügen über NICs mit höherer Kapazität, daher sollten Sie erwägen, eine größere Instanz oder mehr VMs zu verwenden, wenn Sie höhere Netzwerkbegrenzungen auf einem einzelnen Computer benötigen. Wenn Sie von einer lokalen Anwendung auf einen Speicherdienst zugreifen, gilt dieselbe Regel: Informieren Sie sich über die Netzwerkkapazitäten des Clientgeräts und die Netzwerkverbindung zum Azure-Speicherort, und optimieren Sie diese entweder oder entwerfen Sie die Anwendung so, dass sie diesen Kapazitäten entspricht.  

#####<a name="subheading3"></a>Verbindungsqualität
Seien Sie sich wie bei jeder Netzwerknutzung bewusst, dass Netzwerkbedingungen, die zu Fehlern und Paketverlusten führen, den effektiven Durchsatz verlangsamen.  Die Verwendung von WireShark oder NetMon kann bei der Diagnose dieses Problems helfen.  

#####Nützliche Ressourcen
Informationen zur Größe virtueller Computer und zur zugewiesenen Bandbreite finden Sie unter [Bewährte Vorgehensweisen für den Entwurf umfangreicher Dienste auf Azure Cloud Services](http://msdn.microsoft.com/library/dn197896.aspx) auf MSDN.  

####<a name="subheading4"></a>Speicherort
In jeder verteilten Umgebung wird die beste Leistung erzielt, indem der Client in der Nähe des Servers platziert wird. Zum Zugriff auf den Azure-Speicher mit der niedrigsten Latenz befindet sich der beste Standort für den Client innerhalb derselben Azure-Region. Wenn Sie beispielsweise eine Azure-Website haben, die den Azure-Speicher verwendet, sollten Sie beide innerhalb derselben Region platzieren (z. B. Westeuropa oder Südostasien). Dies reduziert die Latenz und Kosten - zum Redaktionszeitpunkt, die Bandbreitennutzung innerhalb einer Region ist kostenlos.  

Wenn Ihre Clientanwendungen nicht in Azure gehostet werden (beispielsweise Apps für Mobilgeräte oder lokale Enterprise Services), wird die Latenz ebenfalls durch Platzierung des Speicherkontos in einer Region in der Nähe der Geräte, die darauf zugreifen, reduziert. Falls Ihre Clients weit verteilt sind (z. B. einige in Nordamerika und andere in Europa), sollten Sie die Verwendung mehrerer Speicherkonten in Betracht ziehen: einen in Nordamerika und einen in Europa. Dadurch wird die Latenz für Benutzer in beiden Regionen reduziert. Diese Vorgehensweise ist meist einfacher zu implementieren, wenn die in der Anwendung gespeicherten Daten speziell für bestimmte Benutzer gelten und keine Datenreplikation zwischen den Speicherkonten erforderlich ist.  Für eine verbreitete Inhaltsverteilung wird ein CDN empfohlen. Weitere Details dazu finden Sie im nächsten Abschnitt.  

###<a name="subheading5"></a>Inhaltsverteilung
Manchmal muss eine Anwendung denselben Inhalt für mehrere Benutzer bereitstellen (z. B. ein Produkt-Demovideo auf der Startseite einer Website), die sich entweder in derselben oder in verschiedenen Regionen befinden. In diesem Fall sollten Sie ein Content Delivery Network (CDN) wie das Azure CDN verwenden, und das CDN nutzt den Azure-Speicher als Datenursprung. Im Gegensatz zum Azure-Speicherkonto, das in genau einer Region vorhanden ist und Inhalte nicht mit niedriger Latenz an andere Regionen liefern kann, verwendet das Azure CDN Server in mehreren Rechenzentren weltweit. Darüber hinaus unterstützt ein CDN in der Regel höhere Ausgangsgrenzen als ein einzelnes Speicherkonto.  

Weitere Informationen zu Azure CDN finden Sie unter [Azure CDN](http://azure.microsoft.com/services/cdn/).  

###<a name="subheading6"></a>Verwenden von SAS und CORS
Wenn Sie Code wie z. B. JavaScript im Webbrowser eines Benutzers oder eine mobile App zum Zugriff auf Daten im Azure-Speicher autorisieren müssen, besteht die Möglichkeit eine Anwendung in der Webrolle als Proxy zu verwenden: Das Gerät des Benutzers wird mit der Webrolle authentifiziert, die sich im Gegenzug mit dem Speicherdienst authentifiziert. Auf diese Weise müssen Sie den Speicherkontoschlüssel nicht gegenüber unsicheren Geräten offenbaren. Dies sorgt jedoch für eine deutliche Belastung der Webrolle, da alle zwischen dem Benutzergerät und dem Speicherdienst übertragenen Daten über die Webrolle weitergeleitet werden. Sie können die Verwendung einer Webrolle als Proxy für den Speicherdienst durch die Verwendung von Shared Access Signatures (SAS) vermeiden, manchmal im Zusammenhang mit Cross-Origin Resource Sharing-Headers (CORS). Durch SAS kann das Benutzergerät Anfragen mithilfe eines beschränkten Zugriffstokens direkt an den Speicherdienst stellen. Wenn beispielsweise ein Benutzer ein Foto in die Anwendung hochladen möchte, kann Ihre Webrolle ein SAS-Token generieren und an das Benutzergerät senden, um für 30 Minuten Schreibzugriff auf einen bestimmten Blob oder Container zu gewähren (danach läuft das SAS-Token ab).   

Normalerweise lässt der Browser kein JavaScript auf einer Seite zu, die von einer Website auf einer Domäne gehostet wird, um bestimmte Vorgänge wie "PUT" auf einer anderen Domäne auszuführen. Wenn Sie beispielsweise eine Webrolle unter "contosomarketing.cloudapp.net" hosten und clientseitiges JavaScript verwenden möchten, um einen Blob in das Speicherkonto unter "contosoproducts.blob.core.windows.net" hochzuladen, verhindert die Ursprungsrichtlinie des Browsers diesen Vorgang. CORS ist eine Browserfunktion, mit deren Hilfe die Zieldomäne (in diesem Fall das Speicherkonto) dem Browser mitteilen kann, dass sie Anfragen aus der Quelldomäne (in diesem Fall von der Webrolle) vertraut.  

Beide dieser Technologien können dabei helfen, unnötige Lasten (und Engpässe) in der Webanwendung zu vermeiden.  

####Nützliche Ressourcen
Weitere Informationen zu SAS finden Sie unter [Shared Access Signatures, Teil 1: Grundlagen zum SAS-Modell](http://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/).  

Weitere Informationen zu CORS finden Sie unter [Cross-Origin Resource Sharing (CORS)-Support für den Azure-Speicherdienst auf MSDN)]http://msdn.microsoft.com/library/azure/dn535601.aspx).  

###Cachedienst
####<a name="subheading7"></a>Abrufen von Daten
In der Regel sollte das Abrufen von Daten von einem Dienst nur einmal erfolgen, nicht zweimal. Denken Sie an das Beispiel einer MVC-Webanwendung, die in einer Webrolle ausgeführt wird, die bereits einen Blob von 50 MB aus dem Speicherdienst abgerufen hat, der als Inhalt für einen Benutzer bereitgestellt wird. Die Anwendung kann anschließend entweder denselben Blob bei jeder Benutzeranforderung erneut abrufen oder ihn lokal auf der Festplatte zwischenspeichern und die zwischengespeicherte Version für nachfolgende Benutzeranforderungen erneut verwenden. Außerdem könnte die Anwendung immer, wenn ein Benutzer die Daten anfordert, eine GET-Anforderung mit einem Bedingungsheader für die Änderungszeit ausgeben, um nicht jedes Mal den gesamten Blob abzurufen, wenn er nicht geändert wurde. Dies funktioniert auch für Tabellenentitäten.  

In einigen Fällen können Sie festlegen, dass die Anwendung davon ausgehen kann, dass der Blob nach dem Abrufen für eine bestimmte Zeit gültig bleibt und dass die Anwendung während dieses Zeitraums nicht prüfen muss, ob der Blob geändert wurde. 

Konfigurations-, Such- und andere Daten, die immer von der Anwendung verwendet werden, sind hervorragend für die Zwischenspeicherung geeignet.  

Ein Beispiel für das Abrufen der Blob-Eigenschaften mithilfe von .NET, um das letzte Änderungsdatum zu ermitteln, finden Sie unter [Festlegen und Abrufen von Eigenschaften und Metadaten](http://msdn.microsoft.com/library/hh225342.aspx) auf MSDN. Informationen zu bedingten Downloads finden Sie unter [Lokale Kopie eines Blob unter bestimmten Bedingungen aktualisieren](http://msdn.microsoft.com/library/azure/dd179371.aspx) auf MSDN.  

####<a name="subheading8"></a>Stapelweises Hochladen von Daten
In einigen Anwendungsszenarien können Sie Daten lokal aggregieren und dann regelmäßig als Batch hochladen, statt die einzelnen Daten sofort hochzuladen. Beispielsweise kann eine Webanwendung eine Protokolldatei der Aktivitäten speichern: Die Anwendung kann entweder die Details jeder Aktivität sofort als Tabellenentität hochladen (was viele Speichervorgänge erforderlich macht) oder die Aktivitätsdetails in einer lokalen Protokolldatei speichern und dann regelmäßig alle Aktivitätsdetails als Datei mit Trennzeichen in den Blob hochladen. Wenn jeder Protokolleintrag eine Größe von 1 KB hat, können Sie Tausende davon in einer einzigen "Put Blob"-Transaktion übertragen (Sie können bis zu 64 MB in einer einzigen Blob-Transaktion hochladen). Wenn der lokale Computer vor dem Upload abstürzt, besteht eine hohe Wahrscheinlichkeit, dass einige Protokolldaten verloren gehen: der Anwendungsentwickler muss bei der Entwicklung die Möglichkeit defekter Clientgeräte oder eines Fehlers beim Hochladen berücksichtigen.  Wenn die Aktivitätsdaten für bestimmte Zeitspannen heruntergeladen werden sollen (nicht nur für eine Aktivität), dann sind Blobs gegenüber Tabellen vorzuziehen. 

###.NET-Konfiguration
Für die Verwendung des .NET Frameworks sind in diesem Abschnitt einige Schnellkonfigurationseinstellungen aufgelistet, die eine deutliche Leistungsoptimierung bewirken.  Bei Verwendung anderer Sprachen sollten Sie sich informieren, ob es ähnliche Konzepte für die jeweilige Sprache gibt.  

####<a name="subheading9"></a>Standardverbindungslimit erhöhen
In .NET erhöht der folgende Code das Standardverbindungslimit (in der Regel 2 in einer Clientumgebung oder 10 in einer Serverumgebung) auf 100. In der Regel sollten Sie den Wert auf die ungefähre Anzahl der Threads, die von der Anwendung verwendet werden, setzen.  

	ServicePointManager.DefaultConnectionLimit = 100; //(Or More)  

Sie müssen das Verbindungslimit festlegen, bevor Sie die Verbindungen öffnen.  

Für andere Programmiersprachen erfahren Sie das Verbindungslimit aus der zugehörigen Dokumentation.  

Weitere Informationen finden Sie im Eintrag [Webdienste: Gleichzeitige Verbindungen](http://blogs.msdn.com/b/darrenj/archive/2005/03/07/386655.aspx) auf MSDN.  

####<a name="subheading10"></a>Erhöhen der Mindestanzahl von Threadpool-Threads bei Verwendung von synchronem Code mit asynchronen Aufgaben
Dieser Code erhöht die Mindestanzahl von Threadpool-Threads:  

	ThreadPool.SetMinThreads(100,100); //(Determine the right number für die Anwendung)  

Weitere Informationen finden Sie unter [ThreadPool.SetMinThreads-Methode](http://msdn.microsoft.com/library/system.threading.threadpool.setminthreads(v=vs.110).aspx) auf MSDN.  

####<a name="subheading11"></a>Nutzung der .NET 4.5 Garbage Collection
Verwenden Sie .NET 4.5 oder höher, damit die Clientanwendung die Leistungsverbesserungen bei der Server Garbage Collection nutzen kann.   

Weitere Informationen finden Sie im Artikel [Eine Übersicht der Leistungsverbesserungen in .NET 4.5](http://msdn.microsoft.com/magazine/hh882452.aspx) auf MSDN.  

###<a name="subheading12"></a>Uneingeschränkte Parallelität
Während Parallelität großartig für die Leistung sein kann, verwenden Sie uneingeschränkte Parallelität (ohne Beschränkung der Anzahl von Threads und/oder parallelen Anforderungen) zum Hochladen oder Herunterladen von Daten mit Vorsicht. Das gilt auch für mehrere Worker zum Zugriff auf mehrere Partitionen (Container, Warteschlangen oder Tabellenpartitionen) im selben Speicherkonto oder für den Zugriff auf mehrere Elemente in derselben Partition. Bei uneingeschränkter Parallelität kann die Kapazität des Clientgeräts oder das Skalierbarkeitsziel des Speicherkontos überschritten werden, sodass es zu höherer Latenz und Drosselung kommt.  

###<a name="subheading13"></a>Speicherclientbibliotheken und Tools
Verwenden Sie immer die neuesten von Microsoft bereitgestellten Clientbibliotheken und Tools. Zum Redaktionszeitpunkt sind RTM-Bibliotheken für .NET, Windows Phone, Windows Runtime und Java sowie Vorschaubibliotheken in anderen Sprachen wie node.js und C++ verfügbar. Darüber hinaus hat Microsoft PowerShell-Cmdlets und plattformübergreifende Befehlszeilentools in Node.js für die Verwendung mit dem Azure-Speicher entwickelt. Microsoft entwickelt diese Tools aktiv im Hinblick auf die Leistung, hält sie auf dem aktuellen Stand der Dienstversionen und stellt sicher, dass sie viele der bewährten Leistungsmethoden intern abwickeln.  

###Wiederholungsversuche
####<a name="subheading14"></a>Drosselung/ServerBusy
In einigen Fällen kann der Speicherdienst Ihre Anwendung drosseln oder einfach aufgrund einer vorübergehenden Bedingung nicht in der Lage sein, die Anforderung auszuführen, sodass ein Fehlercode "503 Server Busy" oder "500 Operation Timeout" zurückgegeben wird.  Dies kann passieren, wenn Ihre Anwendung sich an die Skalierbarkeitsziele annähert oder wenn das System Ihre partitionierten Daten neu ausgleicht, um einen höheren Durchsatz zu ermöglichen.  In der Regel sollte die Clientanwendung den Vorgang wiederholen, der einen derartigen Fehler verursacht: ein späterer erneuter Versuch kann erfolgreich verlaufen. Wenn der Speicherdienst jedoch Ihre Anwendung drosselt, weil die Skalierbarkeitsziele überschritten wurden, oder wenn der Dienst aus einem anderen Grund nicht in der Lage war, die Anforderung auszuführen, verschlimmern aggressive Wiederholungsversuche meist das Problem. Aus diesem Grund sollten Sie exponentiell ansteigende Wartezeiten verwenden (dieses Verhalten wird standardmäßig von den Clientbibliotheken verwendet). Beispielsweise kann Ihre Anwendung nach 2 Sekunden, dann nach 4 Sekunden, nach 10 Sekunden und nach 30 Sekunden einen Wiederholungsversuch starten und dann komplett aufgeben. Dieses Verhalten führt dazu, dass Ihre Anwendung die Dienstbelastung deutlich reduziert, statt Probleme weiter zu verschärfen.  

Beachten Sie, dass Verbindungsfehler sofort wiederholt werden können, da sie kein Ergebnis einer Drosselung sind und nur vorübergehend bestehen sollten.  

####<a name="subheading15"></a>Nicht wiederholbarer Fehler
Die Clientbibliotheken erkennen, welche Fehler Wiederholungsversuche nach sich ziehen und welche nicht. Wenn Sie Ihren eigenen Code für den Speicher-REST-API schreiben, beachten Sie, dass einige Fehler auftreten, die nicht erneut versucht werden dürfen: z. B. die Antwort 400 (Ungültige Anforderung) weist darauf hin, dass die Clientanwendung eine Anforderung gesendet hat, die nicht verarbeitet werden konnte, weil sie ein unerwartetes Format hatte. Das erneute Senden dieser Anforderung führt jedes Mal zur selben Antwort, daher ist es nicht sinnvoll, es wiederholt zu versuchen. Wenn Sie Ihren eigenen Code für die Speicher-REST-API schreiben, beachten Sie, was die Fehlercodes bedeuten und ob sie Wiederholungsversuche nach sich ziehen sollten.  

####Nützliche Ressourcen
Weitere Informationen zu Speicher-Fehlercodes finden Sie unter [Status- und Fehlercodes](http://msdn.microsoft.com/library/azure/dd179382.aspx) auf der Microsoft Azure-Website.  

##Blobs
Zusätzlich zu den zuvor beschriebenen bewährten Vorgehensweisen für [Alle Dienste](#allservices)  können die folgenden bewährten Vorgehensweisen speziell für den Blobdienst angewendet werden.  

###Blob-spezifische Skalierbarkeitsziele
####<a name="subheading16"></a>Bandbreite und Vorgänge pro Blob
Sie können in einem einzelnen Blob bis zu maximal 60 MB/Sekunde lesen oder schreiben (dies entspricht etwa 480 MBit/s), was die Kapazität vieler clientseitigen Netzwerke (einschließlich der physischen NIC auf dem Clientgerät) übersteigt. Außerdem unterstützt ein einzelner Blob bis zu 500 Anforderungen pro Sekunde. Wenn Sie über mehrere Clients verfügen, die denselben Blob lesen und Sie dieses Limit wahrscheinlich überschreiten, sollten Sie ein CDN für die Blob-Verteilung verwenden.  

Weitere Informationen zum Zieldurchsatz für Blobs finden Sie unter [Skalierbarkeits- und Leistungsziele für Windows Azure-Speicher](http://msdn.microsoft.com/library/azure/dn249410.aspx) auf MSDN.  

###Kopieren und Verschieben von Blobs
####<a name="subheading17"></a>Kopieren von BLOBs
Mit dem Speicher REST-API-Version 2012-02-12 wurde die nützliche Kopierfunktion von Blobs zwischen Konten eingefügt: Eine Clientanwendung kann den Speicherdienst anweisen, einen Blob aus einer anderen Quelle (ggf. aus einem anderen Speicherkonto) zu kopieren und dann den Dienst den Kopiervorgang asynchron durchführen lassen. Dies kann die für die Anwendung bei der Migration von Daten aus anderen Speicherkonten erforderliche Bandbreite deutlich reduzieren, da Sie die Daten nicht herunter- und wieder hochladen müssen.  

Es sollte jedoch berücksichtigt werden, dass es beim Kopieren zwischen Speicherkonten keine Zeitgarantie gibt, wann der Kopiervorgang abgeschlossen ist. Wenn Ihre Anwendung das Kopieren des Blobs schnell und kontrolliert abschließen soll, kann es besser sein, den Blob zu kopieren, indem Sie ihn auf einen virtuellen Computer herunterladen und dann auf das Ziel hochladen.  Stellen Sie für genaue Vorhersehbarkeit in dieser Situation sicher, dass der Kopiervorgang von einem virtuellen Computer in derselben Azure-Region durchgeführt wird, andernfalls kann sich die Netzwerkleistung auf die Kopierleistung auswirken.  Darüber hinaus können Sie den Fortschritt eines asynchronen Kopiervorgangs programmgesteuert überwachen.  
 
Beachten Sie, dass Kopien im selben Speicherkonto normalerweise sehr schnell abgeschlossen sind.  

Weitere Informationen finden Sie unter [Kopieren von Blob auf MSDN](http://msdn.microsoft.com/library/dd894037.aspx).  

####<a name="subheading18"></a>Verwenden von AzCopy
Das Azure-Speicherteam hat ein Befehlszeilentool namens "AzCopy" veröffentlicht, das die Massenübertragung von vielen Blobs von, an und über Speicherkonten hinweg unterstützen soll.  Dieses Tool ist für dieses Szenario optimiert und kann hohe Übertragungsraten erzielen.  Die Verwendung wird für Massenszenarien beim Hochladen, Herunterladen und Kopieren empfohlen.  Weitere Informationen und einen Download finden Sie [hier](http://azure.microsoft.com/documentation/articles/storage-use-azcopy/).  

####<a name="subheading19"></a>Azure-Import/Export-Dienst
Für größere Datenmengen (mehr als 1 TB) bietet der Azure-Speicher einen Import-/Export-Dienst, der das Hochladen in und Herunterladen aus dem Blob-Speicher durch den Versand von Festplatten ermöglicht.  Sie können Ihre Daten auf eine Festplatte speichern und zum Hochladen an Microsoft senden oder eine leere Festplatte zum Herunterladen der Daten an Microsoft schicken.  Weitere Informationen dazu finden Sie [hier](http://azure.microsoft.com/documentation/articles/storage-import-export-service/).  Dies kann sehr viel effizienter als das Hochladen/Herunterladen einer solchen Datenmenge über das Netzwerk sein.  

###<a name="subheading20"></a>Verwenden von Metadaten
Der Blobdienst unterstützt Header-Anforderungen, die Metadaten zum Blob enthalten können. Wenn Ihre Anwendung beispielsweise die EXIF-Daten eines Fotos benötigt, kann sie das Foto herunterladen und diese extrahieren. Die Anwendung kann, um Bandbreite zu sparen und die Leistung zu verbessern, die EXIF-Daten in den Blob-Metadaten speichern, wenn die Anwendung das Foto hochgeladen hat. Sie können dann mit einer HEAD-Anfrage die EXIF-Daten in den Metadaten abrufen und so erhebliche Bandbreiten- und Verarbeitungszeit sparen, die jedes Mal zum Extrahieren der EXIF-Daten beim Lesen des Blob benötigt wird. Dies kann in Fällen nützlich sein, wenn Sie nur die Metadaten benötigen und nicht den vollständigen Inhalt eines Blobs.  Beachten Sie, dass pro Blob nur 8 KB Metadaten gespeichert werden können (der Dienst akzeptiert keine Anfrage, mit der mehr als das gespeichert werden soll); wenn die Daten also diese Größe überschreiten, können Sie diese Vorgehensweise nicht anwenden.  

Ein Beispiel für das Abrufen der Blob-Metadaten mithilfe von .NET finden Sie unter [Festlegen und Abrufen von Eigenschaften und Metadaten](http://msdn.microsoft.com/library/hh225342.aspx) auf MSDN.  

###Schnelles Hochladen
Um Blobs schnell hochzuladen, müssen Sie erst die folgende Frage beantworten: laden Sie einen Blob hoch oder mehrere?  Befolgen Sie die Anleitung unten, um die richtige Methode für Ihr Szenario auszuwählen.  

####<a name="subheading21"></a>Schnelles Hochladen eines großen Blobs
Um einen großen Blob schnell hochzuladen, sollte Ihre Clientanwendung ihre Blöcke oder Seiten parallel hochladen (beachten Sie dabei die Skalierbarkeitsziele für einzelne Blobs sowie für das gesamte Speicherkonto).  Die offiziell von Microsoft bereitgestellten RTM-Speicherclientbibliotheken (.NET, Java) haben die Möglichkeiten dafür.  Verwenden Sie für die einzelnen Bibliotheken die unten angegebenen Objekte/Eigenschaften, um den Grad an Parallelität festzulegen.  

-	.NET: Stellen Sie ParallelOperationThreadCount für ein zu verwendendes BlobRequestOptions-Objekt ein.
-	Java/Android: Verwenden Sie BlobRequestOptions.setConcurrentRequestCount()
-	Node.js: Verwenden Sie parallelOperationThreadCount entweder für die Anforderungsoptionen oder für den Blobdienst.
-	C++: Verwenden Sie die Methode blob_request_options::set_parallelism_factor.   

####<a name="subheading22"></a>Schnelles Hochladen von mehreren Blobs
Um viele Blobs schnell hochzuladen, laden Sie die Blobs parallel hoch. Dies funktioniert schneller als das Hochladen von jeweils einem Blob, dessen Blöcke parallel hochgeladen werden, da der Hochladevorgang auf mehrere Partitionen des Speicherdiensts verteilt wird. Ein einzelner Blob unterstützt einen Durchsatz von 60 MB/Sekunde (etwa 480 MBit/s). Zum Redaktionszeitpunkt unterstützten georedundante Speicherkonten (GRS) bis zu 20 GBit/s für eingehende Daten, was einem viel höheren Durchsatz als bei einem einzelnen Blob entspricht.  [AzCopy](#subheading18) führt das Hochladen standardmäßig parallel aus, was für dieses Szenario empfohlen wird.  

###<a name="subheading23"></a>Auswählen des richtigen Blobtyps
Azure-Speicher unterstützt zwei Blobtypen: *Seiten-*-Blobs und *Block-*-Blobs. Je nach Nutzungsszenario wirkt sich die Auswahl des Blobtyps auf die Leistung und Skalierbarkeit der Lösung aus. Block-Blobs sind geeignet, wenn Sie große Datenmengen effizient hochladen möchten: z. B. eine Clientanwendung, die Fotos oder Videos in einen Blob-Speicher hochladen muss. Seiten-Blobs sind für Anwendungen geeignet, die sequentielle Schreibvorgänge auf den Daten ausführen: z. B. Azure-VHDs werden als Seiten-Blobs gespeichert.  

Weitere Informationen finden Sie unter [Grundlegendes zu Block-Blobs und Seiten-Blobs](http://msdn.microsoft.com/library/azure/ee691964.aspx) auf MSDN.  

##Tabellen
Zusätzlich zu den zuvor beschriebenen bewährten Vorgehensweisen für [Alle Dienste](#allservices) können die folgenden bewährten Vorgehensweisen speziell für den Tabellendienst angewendet werden.  

###<a name="subheading24"></a>Tabellenspezifische Skalierbarkeitsziele
Zusätzlich zur Bandbreiteneinschränkung des gesamten Speicherkontos haben Tabellen die folgende spezifische Skalierbarkeitsgrenze.  Beachten Sie, dass das System den ansteigenden Datenverkehr ausbalanciert, bei plötzlichen Verkehrsspitzen wird jedoch der komplette Durchsatz nicht sofort verfügbar.  Falls Ihr Datenmuster Spitzenlasten enthält, müssen Sie mit Drosselung und/oder Timeouts rechnen, während der Speicherdienst den automatischen Lastenausgleich für die Tabelle ausführt.  Eine langsame Steigerung zeigt in der Regel bessere Ergebnisse, da das System genügend Zeit für den Lastenausgleich hat.  

####Entitäten pro Sekunde (Konto)
Die Skalierbarkeitsgrenze für den Tabellenzugriff liegt bei 20.000 Entitäten (je 1 KB) pro Sekunde für ein Konto.  Im Allgemeinen zählt jede Entität, die eingefügt, aktualisiert, gelöscht oder gescannt wird, für dieses Ziel.  Daher zählt eine Batcheinfügung, die 100 Entitäten enthält, als 100 Entitäten.  Eine Abfrage, die 1.000 Entitäten scannt und 5 zurückgibt, zählt als 1.000 Entitäten.  

####Entitäten pro Sekunde (Partition)
Innerhalb einer Partition liegt das Skalierbarkeitsziel für den Tabellenzugriff bei 2.000 Entitäten (je 1 KB) pro Sekunde, mit derselben Zählweise wie im vorherigen Abschnitt beschrieben.  

###Konfiguration
In diesem Abschnitt sind verschiedene Schnellkonfigurationseinstellungen aufgelistet, die eine deutliche Leistungsoptimierung im Tabellendienst bewirken.  

####<a name="subheading25"></a>Verwenden von JSON
Ab Speicherdienstversion 2013-08-15 unterstützt der Tabellendienst die Verwendung von JSON statt des XML-basierten AtomPub-Formats für die Übertragung von Tabellendaten. Dies kann die Nutzlast um bis zu 75 % verringern und die Anwendungsleistung deutlich verbessern.   

Weitere Informationen finden Sie im Blogeintrag [Microsoft Azure-Tabellen: Einführung in JSON](http://blogs.msdn.com/b/windowsazurestorage/archive/2013/12/05/windows-azure-tables-introducing-json.aspx) und [Nutzlastformat für Tabellendienstvorgänge](http://msdn.microsoft.com/library/azure/dn535600.aspx) auf MSDN. 

####<a name="subheading26"></a>Nagle deaktivieren
Der Nagle-Algorithmus ist in TCP/IP-Netzwerken weit verbreitet, um die Netzwerkleistung zu verbessern. Er ist jedoch nicht unter allen Umständen optimal (z. B. hoch interaktive Umgebungen). Für den Azure-Speicher hat der Nagle-Algorithmus negative Auswirkungen auf Anforderungen an Tabellen- und Warteschlangendienste, daher sollten Sie ihn möglichst deaktivieren.  

Weitere Informationen finden Sie im Blogeintrag [Nagle-Algorithmus geht nicht freundlich mit kleinen Anfragen um](http://blogs.msdn.com/b/windowsazurestorage/archive/2010/06/25/nagle-s-algorithm-is-not-friendly-towards-small-requests.aspx), in dem erläutert wird, warum der Nagle-Algorithmus unzureichend mit Tabellen- und Warteschlangenanforderungen interagiert und wie Sie ihn in der Clientanwendung deaktivieren.  

###Schema
Wie Sie Ihre Daten präsentieren und abfragen, ist der größte Faktor mit Auswirkung auf die Leistung des Tabellendiensts. Während jede Anwendung anders ist, sind in diesem Abschnitt einige allgemeine bewährte Vorgehensweisen für folgende Bereiche beschrieben:  

-	Tabellenentwurf
-	Effiziente Abfragen
-	Effiziente Datenaktualisierungen  

####<a name="subheading27"></a>Tabellen und Partitionen
Tabellen sind in Partitionen unterteilt. Jede in einer Partition gespeicherte Entität verwendet denselben Partitionsschlüssel und hat einen eindeutigen Zeilenschlüssel, damit sie innerhalb der Partition identifiziert werden kann. Partitionen bieten Vorteile, führen jedoch auch zu Skalierbarkeitsgrenzen.  

-	Vorteile: Sie können die Entitäten innerhalb derselben Partition in einer einzigen, unteilbaren Batchtransaktion aktualisieren, die bis zu 100 separate Speichervorgänge enthalten kann (Gesamtgrößenlimit 4 MB). Bei derselben Menge abzurufender Entitäten können Sie auch die Daten innerhalb einer Partition effizienter abfragen als Daten, die sich über mehrere Partitionen verteilen (lesen Sie für noch mehr Empfehlungen zur Abfrage von Tabellendaten weiter).
-	Skalierbarkeitsgrenze: Für den Zugriff auf Entitäten, die in einer einzigen Partition gespeichert sind, kann kein Lastenausgleich erfolgen, da die Partitionen unteilbare Batchtransaktionen unterstützen. Aus diesem Grund ist das Skalierbarkeitsziel für eine einzelne Tabellenpartition niedriger als für den gesamten Tabellendienst.  

Aufgrund dieser Eigenschaften von Tabellen und Partitionen sollten Sie die folgenden Entwurfsprinzipien beachten:  

-	Daten, die Ihre Clientanwendung häufig in derselben logischen Arbeitseinheit aktualisiert oder abfragt, sollten sich in derselben Partition befinden.  Der Grund dafür kann sein, dass Ihre Anwendung Schreibvorgänge aggregiert oder dass Sie unteilbare Batchtransaktionen nutzen möchten.  Außerdem können Daten innerhalb einer Partition effizienter mit nur einer Abfrage abgefragt werden als Daten, die sich über mehrere Partitionen verteilen.
-	Daten, die Ihre Clientanwendung nicht in derselben logischen Arbeitseinheit einfügt/aktualisiert (Einzelabfrage oder Batchaktualisierung), sollten sich in separaten Partitionen befinden.  Ein wichtiger Hinweis ist hier, dass es keine Begrenzung der Anzahl der Partitionsschlüssel in einer Tabelle gibt, daher sind selbst Millionen von Partitionsschlüsseln kein Problem und wirken sich nicht auf die Leistung aus.  Wenn es sich bei Ihrer Anwendung beispielsweise um eine beliebte Website mit Benutzeranmeldung handelt, kann die Benutzer-ID als Partitionsschlüssel verwendet werden.  

####Hot Partition
Eine Hot Partition ist eine Partition, die einen überproportionalen Anteil des Datenverkehrs an ein Konto empfängt und kein Lastenausgleich erfolgen kann, da es sich um eine einzelne Partition handelt.  Im Allgemeinen wird eine Hot Partition mit einer der folgenden Möglichkeiten erstellt:  

#####<a name="subheading28"></a>Nur-Anhängen/Nur-Voranstellen-Muster
Das Muster "Nur anhängen" bezieht sich darauf, dass (fast) der gesamte Datenverkehr an einen bestimmten PS abhängig von der aktuellen Zeit ansteigt oder absinkt.  Ein Beispiel ist eine Anwendung, in der das aktuelle Datum als Partitionsschlüssel für Protokolldaten verwendet wird.  Das führt dazu, dass alle Einfügungen in die letzte Partition der Tabelle eingefügt werden und das System keinen Lastenausgleich durchführen kann, da alle Schreibvorgänge am Ende der Tabelle erfolgen.  Wenn das Datenvolumen in dieser Partition das Skalierbarkeitsziel auf Partitionsebene überschreitet, kommt es zu einer Drosselung.  Es ist besser sicherzustellen, dass der Datenverkehr an mehrere Partitionen geleitet wird, um den Lastenausgleich für Anfragen in der Tabelle zu verteilen.  

#####<a name="subheading29"></a>Hoher Datenverkehr
Wenn Ihr Partitionsschema dazu führt, dass in einer einzigen Partition Daten enthalten sind, die deutlich häufiger als in anderen Partitionen verwendet werden, kann ebenfalls eine Drosselung erfolgen, wenn diese Partition das Skalierbarkeitsziel für eine einzelne Partition erreicht.  Sie sollten besser sicherstellen, dass Ihr Partitionsschema so angelegt ist, dass sich keine einzelne Partition an die Skalierbarkeitsziele annähert.  

####Abfragen
In diesem Abschnitt werden bewährte Vorgehensweisen für Tabellendienstabfragen erläutert.  

#####<a name="subheading30"></a>Abfragebereich
Es gibt mehrere Möglichkeiten, den Entitätsbereich für die Abfrage anzugeben.  Im Folgenden werden diese einzeln erläutert.  

In der Regel sollten Sie Scans vermeiden (Abfragen, die sich auf mehr als eine Entität beziehen), wenn Sie jedoch scannen müssen, organisieren Sie Ihre Daten so, dass der Scan die benötigten Daten abruft, ohne eine übermäßige Menge an nicht benötigten Entitäten zurückzugeben.  

######Punktabfragen
Eine Punktabfrage ruft genau eine Entität ab. Dazu wird sowohl der Partitionsschlüssel als auch der Zeilenschlüssel der abzurufenden Entität angegeben. Diese Abfragen sind sehr effizient und Sie sollten diese so oft wie möglich verwenden.  

######Partitionsabfragen
Eine Partitionsabfrage ist eine Abfrage, die eine Gruppe von Daten mit einem gemeinsamen Partitionsschlüssel abruft. In der Regel gibt die Abfrage einen Wertebereich für den Zeilenschlüssel oder für eine Entitätseigenschaft zusätzlich zum Partitionsschlüssel an. Diese sind weniger effizient als Punktabfragen und ihr Einsatz sollte sorgfältig überlegt werden.  

######Tabellenabfragen
Eine Tabellenabfrage ist eine Abfrage, die eine Gruppe von Entitäten ohne gemeinsamen Partitionsschlüssel abruft. Diese Abfragen sind nicht effizient und Sie sollten diese möglichst vermeiden.  

#####<a name="subheading31"></a>Abfragedichte
Ein weiterer Schlüsselfaktor bei der Abfrageeffizienz ist die Anzahl der zurückgegebenen Entitäten im Vergleich zur Anzahl der gescannten Entitäten, um den zurückgegebenen Satz zu suchen. Wenn Ihre Anwendung eine Tabellenabfrage mit einem Filter für einen Eigenschaftswert durchführt, den nur 1 % der Daten teilen, scannt die Abfrage 100 Entitäten für jede zurückgegebene Entität. Die zuvor erläuterten Skalierbarkeitsziele der Tabelle, beziehen sich alle auf die Anzahl der durchsuchten und nicht der zurückgegebenen Entitäten: eine geringe Abfragedichte kann leicht dazu führen, dass der Tabellendienst die Anwendung einschränkt, weil er viele Entitäten durchsuchen muss, um die von Ihnen gesuchte Entität zu finden.  Hinweise, wie Sie dies vermeiden können, finden Sie unten im Abschnitt [Denormalisierung](#subheading34).  

#####Einschränken der Menge der zurückgegebenen Daten
######<a name="subheading32"></a>Filterung
Wenn Sie wissen, dass eine Abfrage Entitäten zurückgibt, die Sie in der Clientanwendung nicht benötigen, sollten Sie einen Filter verwenden, um die Größe des zurückgegebenen Sets zu reduzieren. Während die nicht an den Client zurückgegebenen Entitäten trotzdem für die Skalierbarkeitsgrenzen zählen, wird sich Ihre Anwendungsleistung aufgrund der reduzierten Netzwerknutzlast und der reduzierten Entitätenanzahl, die von der Clientanwendung verarbeitet werden müssen, verbessern.  Lesen Sie oben den Hinweis zur [Abfragedichte](#subheading31). Die Skalierbarkeitsziele beziehen sich jedoch auf die Anzahl der durchsuchten Entitäten, daher kann eine Abfrage, bei der viele Entitäten herausgefiltert werden, trotzdem zur Drosselung führen, auch wenn nur wenige Entitäten zurückgegeben werden.  

######<a name="subheading33"></a>Projektion
Wenn Ihre Clientanwendung nur bestimmte Eigenschaften der Entitäten in der Tabelle benötigt, können Sie Projektion verwenden, um die Größe des zurückgegebenen Datensatzes einzuschränken. Wie bei der Filterung hilft auch dies bei der Reduzierung der Netzwerklast und der Clientverarbeitung.  

#####<a name="subheading34"></a>Denormalisierung
Im Gegensatz zum Arbeiten mit relationalen Datenbanken führen die bewährten Vorgehensweisen für das effiziente Abfragen von Tabellendaten zur Denormalisierung Ihrer Daten. Das heißt, dieselben Daten werden in mehreren Entitäten dupliziert (eine für jeden Schlüssel, den Sie zum Suchen der Daten verwenden könnten), um die Anzahl der Entitäten zu minimieren, die eine Abfrage scannen muss, um die vom Client benötigten Daten zu finden, statt eine große Anzahl von Entitäten zu scannen, um die für die Anwendung benötigten Daten zu finden.  Beispielsweise kann auf einer E-Commerce-Website eine Bestellung sowohl nach Kunden-ID (zur Abfrage der Kundenbestellungen) als auch nach Datum (zur Abfrage der Bestellungen für ein bestimmtes Datum) gefunden werden.  Bei der Tabellenspeicherung ist es am besten, die Entität zweimal zu speichern (bzw. darauf zu verweisen) - einmal mit Tabellenname, PS und ZS für die Suche nach der Kunden-ID und einmal für die Suche nach dem Datum.  

####Einfügen/Aktualisieren/Löschen
In diesem Abschnitt werden bewährte Vorgehensweise für das Ändern von im Tabellendienst gespeicherte Entitäten erläutert.  

#####<a name="subheading35"></a>Batchverarbeitung
Batchtransaktionen werden in Azure Storage als Entitätsgruppentransaktionen (ETG) bezeichnet; alle Vorgänge innerhalb einer ETG müssen sich auf eine einzige Partition in einer einzigen Tabelle beziehen. Wenn möglich, verwenden Sie ETGs für die Durchführung von Einfüge-, Aktualisierungs- und Löschvorgängen in Batches. Dadurch reduziert sich die Anzahl der Roundtrips von Ihrer Clientanwendung zum Server und somit die Anzahl der kostenpflichtigen Transaktionen (eine ETG zählt zu Abrechnungszwecken als eine Transaktion und kann bis zu 100 Speichervorgänge umfassen); außerdem werden unteilbare Aktualisierungen ermöglicht (alle Vorgänge in einer ETG sind erfolgreich oder schlagen fehl). Umgebungen mit hoher Latenz wie beispielsweise Mobilgeräte profitieren stark von der Verwendung von ETGs.  

#####<a name="subheading36"></a>Upsert
Verwenden Sie wenn möglich die Tabellenvorgänge **Upsert**. Es gibt zwei **Upsert**-Typen, die beide effizienter als herkömmliche **Einfügen**- und **Aktualisieren**-Vorgänge sind:  

-	**InsertOrMerge**: Verwenden Sie dies, wenn Sie eine Teilmenge der Entitätseigenschaften hochladen möchten, aber nicht sicher sind, ob die Entität bereits vorhanden ist. Falls die Entität vorhanden ist, aktualisiert dieser Aufruf die in **Upsert** enthaltenen Eigenschaften und lässt alle vorhandenen Eigenschaften unverändert - falls die Entität nicht vorhanden ist, wird die neue Entität eingefügt. Dies funktioniert ähnlich wie die Projektion in einer Abfrage, da Sie nur die geänderten Eigenschaften hochladen müssen. 
-	**InsertOrReplace**: Verwenden Sie dies, wenn Sie eine neue Entität hochladen möchten, aber nicht sicher sind, ob diese bereits vorhanden ist. Sie sollten diese Option nur verwenden, wenn Sie wissen, dass die neu hochgeladene Entität komplett richtig ist, da sie die alte Entität vollständig überschreibt. Wenn Sie beispielsweise die Entität aktualisieren möchten, die den aktuellen Standort eines Benutzers speichert, unabhängig davon, ob die Anwendung zuvor Standortdaten für den Benutzer gespeichert hatte, ist die neue Entität komplett, da Sie keine Informationen aus der vorherigen Entität benötigen. 

#####<a name="subheading37"></a>Speichern von Datenreihen in einer einzigen Entität
Manchmal speichert eine Anwendung eine Reihe von Daten, die sie häufig in einem Schritt abrufen muss: Beispielsweise kann eine Anwendung die CPU-Auslastung über einen bestimmten Zeitraum erfassen, um ein fortlaufendes Diagramm der Daten aus den letzten 24 Stunden zu plotten. Eine Möglichkeit ist es, eine Tabellenentität pro Stunde anzulegen, sodass jede Entität eine bestimmte Stunde darstellt und die CPU-Auslastung für diese Stunde speichert. Um diese Daten zu plotten, muss die Anwendung die Entitäten abrufen, welche die Daten aus den letzten 24 Stunden enthalten.  

Alternativ kann Ihre Anwendung die CPU-Nutzung für jede Stunde als separate Eigenschaft einer einzelnen Entität speichern: für stündliche Aktualisierungen kann die Anwendung einzeln **InsertOrMerge Upsert** abrufen, um den Wert für die aktuelle Stunde zu aktualisieren. Um die Daten zu plotten, muss die Anwendung nur eine einzige Entität statt 24 abrufen, sodass die Abfrage sehr effizient verläuft (siehe Erläuterungen oben zum [Abfragebereich](#subheading30)). 

#####<a name="subheading38"></a>Speichern strukturierter Daten in Blobs
Manchmal hat man das Gefühl, strukturierte Daten sollten immer in Tabellen gespeichert werden, aber Entitätsbereiche werden immer zusammen abgerufen und können als Batch eingefügt werden.  Ein gutes Beispiel dafür ist eine Protokolldatei.  In diesem Fall können Sie Protokolle für mehrere Minuten als Batch zusammenfassen, einfügen und dann immer mehrere Protokollminuten gleichzeitig abrufen.  Aus Leistungsgründen ist es in diesem Fall besser, Blobs statt Tabellen zu verwenden, da Sie damit die Anzahl der geschriebenen/zurückgegebenen Objekte sowie normalerweise auch die Anzahl der gestellten Anfragen deutlich reduzieren können.  

##Warteschlangen
###<a name=subheading39"></a>Skalierbarkeitsgrenze
Eine einzelne Warteschlange kann ca. 2.000 Nachrichten (1 KB jeweils) pro Sekunde (jede AddMessage, GetMessage und DeleteMessage zählen hier als eine Nachricht) verarbeiten. Wenn dies für Ihre Anwendung nicht ausreicht, verwenden Sie mehrere Warteschlangen, und verteilen Sie die Nachrichten auf diese.  

Sie können die aktuellen Skalierbarkeitsziele auf der Seite [Ziele für Skalierbarkeit und Leistung des Azure-Speichers](http://msdn.microsoft.com/library/azure/dn249410.aspx) auf MSDN anzeigen.  

###<a name=subheading40"></a>Nagle deaktivieren
Lesen Sie den Abschnitt über den Nagle-Algorithmus in der Tabellenkonfiguration - der Nagle-Algorithmus wirkt sich meist negativ auf die Leistung von Warteschlangenanforderungen aus, daher sollten Sie ihn deaktivieren.  

###<a name=subheading41"></a>Nachrichtengröße
Die Leistung der Warteschlange und der Skalierbarkeit sinkt, wenn die Nachrichtengröße steigt. Fügen Sie einer Nachricht nur die Informationen hinzu, die der Empfänger benötigt.  

###<a name=subheading42"></a>Batchabruf
Sie können bis zu 32 Nachrichten aus einer Warteschlange in einem einzigen Vorgang abrufen. Dies kann die Anzahl der Roundtrips von der Clientanwendung reduzieren, was besonders für Umgebungen mit hoher Latenz wie beispielsweise Mobilgeräte nützlich ist.  

###<a name=subheading43"></a>Abrufintervall für die Warteschlange
Die meisten Anwendungen fragen Nachrichten aus einer Warteschlange ab. Für die Anwendung kann es sich dabei um die größte Quelle für Transaktionen handeln. Wählen Sie das Abrufintervall mit Bedacht aus: Abrufe, die zu häufig stattfinden, können dazu führen, dass die Anwendung die Skalierbarkeitsziele für die Warteschlange erreicht. Bei jedoch 200.000 Transaktionen für 0,01 US-Dollar (zum Redaktionszeitpunkt) betragen die Kosten für einen einzelnen Prozessor, der im Monat einen Abruf pro Sekunde macht, weniger als 15 Cent. Daher sind Kosten üblicherweise kein Auswahlkriterium für das Abrufintervall.  

Aktuelle Kosteninformationen finden Sie unter [Speicherpreisdetails](http://azure.microsoft.com/pricing/details/storage/).  

###<a name=subheading44"></a>UpdateMessage
Sie können **UpdateMessage** verwenden, um das Unsichtbarkeits-Zeitlimit zu erhöhen oder die Statusinformationen einer Nachricht zu aktualisieren. Trotz der Möglichkeiten dieser Option sollten Sie beachten, dass jeder **UpdateMessage**-Vorgang für das Skalierbarkeitsziel zählt. Dies kann jedoch sehr viel effizienter sein als ein Workflow, der einen Auftrag von einer Warteschlange zur nächsten verschiebt, wenn der jeweilige Schritt abgeschlossen ist. Mithilfe des Vorgangs **UpdateMessage** kann Ihre Anwendung den Auftragsstatus in der Nachricht speichern und dann weiterarbeiten, statt die Nachricht jedes Mal für den nächsten Schritt erneut in die Warteschlange zu stellen.  

Weitere Informationen finden Sie im Artikel [Gewusst wie: Ändern des Inhalts von Nachrichten in der Warteschlange](http://azure.microsoft.com/documentation/articles/storage-dotnet-how-to-use-queues/#change-contents).  

###<a name=subheading45"></a>Anwendungsarchitektur
Verwenden Sie Warteschlangen, um die Anwendungsarchitektur skalierbar zu machen. Nachfolgend sind einige Möglichkeiten aufgelistet, wie Sie Warteschlangen verwenden können, um Ihre Anwendung skalierbarer zu gestalten:  

-	Sie können Warteschlangen verwenden, um Arbeits-Backlogs zur Verarbeitung zu erstellen und die Arbeitsauslastung in der Anwendung gleichmäßig zu verteilen. Sie können z. B. Benutzeranfragen in die Warteschlange stellen, um prozessorintensive Arbeiten durchzuführen, wie das Ändern der Größe hochgeladener Bilder.
-	Sie können Warteschlangen verwenden, um Teile Ihrer Anwendung zu entkoppeln, sodass diese unabhängig skaliert werden können. Beispielsweise kann ein Web-Front-End Umfrageergebnisse von Benutzern zur späteren Analyse und Speicherung in eine Warteschlange stellen. Sie können mehr Workerrolleninstanzen hinzufügen, um die Warteschlangendaten nach Bedarf zu verarbeiten.  

##Zusammenfassung
In diesem Artikel wurden einige der häufigsten bewährten Vorgehensweisen zur Leistungsoptimierung bei der Verwendung des Azure-Speichers erläutert. Wir empfehlen jedem Anwendungsentwickler seine Anwendung anhand dieser Vorgehensweisen zu überprüfen und ggf. die Empfehlungen umzusetzen, um bessere Leistung für ihre Anwendungen zu erzielen, die Azure-Speicher verwenden.

<!--HONumber=42-->
 