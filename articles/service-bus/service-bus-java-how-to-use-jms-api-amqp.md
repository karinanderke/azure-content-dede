<properties 
	pageTitle="Verwenden von AMQP 1.0 mit der Java Service Bus-API - Azure" 
	description="Erfahren Sie, wie Sie den Java Message Service (JMS) mit Azure Service Bus und Advanced Message Queuing-Protodol (AMQP) 1.0 verwenden." 
	services="service-bus" 
	documentationCenter="java" 
	authors="sethmanheim" 
	writer="sethm" 
	manager="timlt" 
	editor="mattshel"/>

<tags 
	ms.service="service-bus" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="na" 
	ms.devlang="Java" 
	ms.topic="article" 
	ms.date="02/10/2015" 
	ms.author="sethm"/>




# Verwenden der JMS (Java Message Service)-API mit Service Bus und AMQP 1.0

## Einführung

AMQP (Advanced Message Queuing Protocol) 1.0 ist ein effizientes, zuverlässiges Messagingprotokoll auf Wire-Ebene, mit dem Sie robuste und plattformübergreifende Messaginganwendungen erstellen können.

Unterstützung für AMQP 1.0 im Service Bus bedeutet, dass Sie die gebrokerten Messagingfunktionen für Warteschlangen und Veröffentlichen/Abonnieren mithilfe eines effizienten binären Protokolls auf unterschiedlichen Plattformen nutzen können. Zudem können Sie Anwendungen erstellen, deren Komponenten mit einer Mischung aus Sprachen, Frameworks und Betriebssystemen erstellt wurden.

In diesem Leitfaden wird beschrieben, wie die gebrokerten Messagingfunktionen von Service Bus (Warteschlange und Themen veröffentlichen/abonnieren) aus Java-Anwendungen heraus mit der beliebten Standard-Programmierschnittstelle Java Message Service (JMS) verwendet werden. In einer separaten Anleitung wird erklärt, wie Sie dieselbe Aufgabe mithilfe der .NET-API für Service Bus durchführen. Sie können diese beiden Anleitungen verwenden, um weitere Informationen zur plattformübergreifenden Nachrichtenübermittlung unter Verwendung von AMQP 1.0 zu erhalten.

## Erste Schritte mit Service Bus

In diesem Leitfaden wird davon ausgegangen, dass Sie bereits einen Service Bus-Namespace haben, der eine Warteschlange mit dem Namen "queue1" enthält. Wenn dies nicht der Fall ist, können Sie den Namespace und die Warteschlange im [Azure-Verwaltungsportal](http://manage.windowsazure.com) erstellen. Weitere Informationen zum Erstellen von Namespaces und Warteschlangen für Service Bus finden Sie in der Anleitung [Verwenden von Service Bus-Warteschlangen](https://azure.microsoft.com/develop/net/how-to-guides/service-bus-queues/).

**Hinweis**: Sie müssen die Warteschlange mit deaktivierter Partitionierung erstellen, da partitionierte Warteschlangen und Themen noch nicht von AMQP unterstützt werden. Weitere Informationen finden Sie unter [Partitionieren von Nachrichtenentitäten](http://msdn.microsoft.com/library/azure/dn520246.aspx).

## Herunterladen der AMQP 1.0 JMS-Clientbibliothek

Informationen zum Downloadort der neuesten Version der Apache Qpid JMS AMQP 1.0-Clientbibliothek finden Sie unter [http://people.apache.org/~rgodfrey/qpid-java-amqp-1-0-client-jms.html](http://people.apache.org/~rgodfrey/qpid-java-amqp-1-0-client-jms.html).

Folgende vier JAR-Dateien müssen aus dem Apache Qpid JMS AMQP 1.0-Verteilungsarchiv zu dem Java-KLASSENPFAD hinzugefügt werden, wenn JMS-Anwendungen mit Service Bus erstellt und ausgeführt werden:

*    geronimo-jms_1.1_spec-1.0.jar
*    qpid-amqp-1-0-client-[version].jar
*    qpid-amqp-1-0-client-jms-[version].jar
*    qpid-amqp-1-0-common-[version].jar

## Programmieren von Java-Anwendungen

### JNDI (Java Naming and Directory Interface; Java Benennungs- und Verzeichnisschnittstelle)
JMS verwendet die Java Naming and Directory Interface (JNDI), um eine Trennung zwischen logischen und physischen Namen umzusetzen. Zwei Typen von JMS-Objekten werden mit JNDI aufgelöst: ConnectionFactory und Destination. JNDI verwendet ein Anbietermodell, das Sie mit verschiedenen Verzeichnisdiensten verbinden können, um Namensauflösungsfunktionen zu implementieren. Die Apache Qpid JMS AMQP 1.0-Bibliothek enthält einen einfachen JNDI-Anbieter, der mithilfe von properties-Dateien im folgenden Format konfiguriert wird:

	# servicebus.properties - sample JNDI configuration
	
	# Register a ConnectionFactory in JNDI using the form:
	# connectionfactory.[jndi_name] = [ConnectionURL]
	connectionfactory.SBCF = amqps://[username]:[password]@[namespace].servicebus.windows.net
	
	# Register some queues in JNDI using the form
	# queue.[jndi_name] = [physical_name]
	# topic.[jndi_name] = [physical_name]
	queue.QUEUE = queue1


<p><strong>Konfigurieren der ConnectionFactory</strong></p>

Der Eintrag zum Definieren einer **ConnectionFactory** in der Qpid Properties-Datei für den JNDI-Anbieter hat das folgende Format:

	connectionfactory.[jndi_name] = [ConnectionURL]

Wobei [jndi_name] und [ConnectionURL] die folgenden Bedeutungen haben:

<table>
  <tr>
    <td>[JNDI_Name]</td>
    <td>Der logische Name der ConnectionFactory. Dieser Name wird in der Java-Anwendung mithilfe der JNDI-Methode IntialContext.lookup() aufgelöst.</td>
  </tr>
  <tr>
    <td>[ConnectionURL]</td>
    <td>Diese URL liefert der JMS-Bibliothek die vom AMQP-Broker benötigten Informationen.</td>
  </tr>
</table>

**ConnectionURL** hat das folgende Format:

	amqps://[username]:[password]@[namespace].servicebus.windows.net

Wobei [Namespace], [Benutzername] und [Kennwort] die folgenden Bedeutungen haben:

<table>
  <tr>
    <td>[Namespace]</td>
    <td>Der Service Bus-Namespace, der aus dem Azure-Verwaltungsportal abgerufen wird.</td>
  </tr>
  <tr>
    <td>[Benutzername]</td>
    <td>Der Service Bus-Ausstellername, der aus dem Azure-Verwaltungsportal abgerufen wird.</td>
  </tr>
  <tr>
    <td>[Kennwort]</td>
    <td>Die URL-codierte Form des Service Bus-Ausstellerschlüssels, der aus dem Azure-Verwaltungsportal abgerufen wird.</td>
  </tr>
</table>

**Hinweis**: Sie müssen das Kennwort manuell URL-codieren. Ein nützliches URL-Codierungshilfsprogramm ist unter [http://www.w3schools.com/tags/ref_urlencode.asp](http://www.w3schools.com/tags/ref_urlencode.asp) verfügbar.

Angenommen, die aus dem Azure-Verwaltungsportal abgerufenen Informationen lauten wie folgt:

<table>
  <tr>
    <td>Namespace:</td>
    <td>foo.servicebus.windows.net</td>
  </tr>
  <tr>
    <td>Ausstellername:</td>
    <td>besitzer</td>
  </tr>
  <tr>
    <td>Ausstellerschlüssel:</td>
    <td>j9VYv1q33Ea+cbahWsHFYnLkEzrF0yA5SAqcLNvU7KM=</td>
  </tr>
</table>

Mit der folgenden Konfigurationszeichenfolge können Sie in diesem Fall eine **ConnectionFactory** mit dem Namen "SBCF" definieren:

	connectionfactory.SBCF = amqps://owner:j9VYv1q33Ea%2BcbahWsHFYnLkEzrF0yA5SAqcLNvU7KM%3D@foo.servicebus.windows.net

<p><strong>Konfigurieren von Zielen</strong></p>

Der Eintrag zum Definieren eines Ziels in der Qpid Properties-Datei für den JNDI-Anbieter hat das folgende Format:

	queue.[jndi_name] = [physical_name]
oder

	topic.[jndi_name] = [physical_name]

Wobei [jndi_name] und [physical_name] die folgenden Bedeutungen haben:

<table>
  <tr>
    <td>[JNDI_Name]</td>
    <td>Der logische Name des Ziels. Dieser Name wird in der Java-Anwendung mithilfe der JNDI-Methode IntialContext.lookup() aufgelöst.</td>
  </tr>
  <tr>
    <td>[Physischer_Name]</td>
    <td>Der Name der Service Bus-Entität, mit der die Anwendung Nachrichten austauscht.</td>
  </tr>
</table>

**Hinweis**: Beim Empfang von einem Service Bus-Themenabonnement sollte der in JNDI angegebene physische Name dem Themennamen entsprechen. Der Abonnementname wird bei der Erstellung des Abonnements im JMS-Anwendungscode angegeben. Im [Entwicklerhandbuch für Service Bus AMQP 1.0](http://msdn.microsoft.com/library/jj841071.aspx) finden Sie weitere Details zum Arbeiten mit Service Bus-Themenabonnements in JMS.

### Schreiben der JMS-Anwendung

Für die Verwendung von JMS mit Service Bus werden keine speziellen Programmierschnittstellen oder Optionen benötigt. Allerdings sind einige Einschränkungen zu beachten, die an späterer Stelle erläutert werden. Wie bei allen JMS-Anwendungen müssen Sie zuerst die JNDI-Umgebung konfigurieren, um eine **ConnectionFactory** und Ziele auflösen zu können.

<p><strong>Konfigurieren des JNDI InitialContext</strong></p>

Zur Konfiguration der JNDI-Umgebung wird eine Hashtabelle mit Konfigurationsinformationen an den Konstruktor der Klasse javax.naming.InitialContext übergeben. Obligatorisch sind in der Hashtabelle zwei Elemente: der Klassenname der Initial Context Factory und die Anbieter-URL. Der folgende Code zeigt, wie Sie die JNDI-Umgebung mithilfe des auf der Qpid Properties-Datei basierenden JNDI-Anbieters mit einer Properties-Datei namens **servicebus.properties** konfigurieren.

	Hashtable<String, String> env = new Hashtable<String, String>(); 
	env.put(Context.INITIAL_CONTEXT_FACTORY, "org.apache.qpid.amqp_1_0.jms.jndi.PropertiesFileInitialContextFactory"); 
	env.put(Context.PROVIDER_URL, "servicebus.properties"); 
	InitialContext context = new InitialContext(env); 

### Einfache JMS-Anwendung mit einer Service Bus-Warteschlange

Das folgende Beispielprogramm sendet JMS-TextMessages an eine Service Bus-Warteschlange mit dem logischen JNDI-Namen QUEUE und empfängt die zurückkommenden Nachrichten.

	// SimpleSenderReceiver.java
	
	import javax.jms.*;
	import javax.naming.Context;
	import javax.naming.InitialContext;
	import java.io.BufferedReader;
	import java.io.InputStreamReader;
	import java.util.Hashtable;
	import java.util.Random;
	
	public class SimpleSenderReceiver implements MessageListener {
	    private static boolean runReceiver = true;
	    private Connection connection;
	    private Session sendSession;
	    private Session receiveSession;
	    private MessageProducer sender;
	    private MessageConsumer receiver;
	    private static Random randomGenerator = new Random();
	
	    public SimpleSenderReceiver() throws Exception {
	        // Configure JNDI environment
	        Hashtable<String, String> env = new Hashtable<String, String>();
	        env.put(Context.INITIAL_CONTEXT_FACTORY, 
                    "org.apache.qpid.amqp_1_0.jms.jndi.PropertiesFileInitialContextFactory");
	        env.put(Context.PROVIDER_URL, "servicebus.properties");
	        Context context = new InitialContext(env);
	
	        // Lookup ConnectionFactory and Queue
	        ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCF");
	        Destination queue = (Destination) context.lookup("QUEUE");
	
	        // Create Connection
	        connection = cf.createConnection();
	
	        // Create sender-side Session and MessageProducer
	        sendSession = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
	        sender = sendSession.createProducer(queue);
	
	        if (runReceiver) {
	            // Create receiver-side Session, MessageConsumer,and MessageListener
	            receiveSession = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);
	            receiver = receiveSession.createConsumer(queue);
	            receiver.setMessageListener(this);
	            connection.start();
	        }
	    }
	
	    public static void main(String[] args) {
	        try {
	
	            if ((args.length > 0) && args[0].equalsIgnoreCase("sendonly")) {
	                runReceiver = false;
	            }
	
	            SimpleSenderReceiver simpleSenderReceiver = new SimpleSenderReceiver();
	            System.out.println("Press [enter] to send a message. Type 'exit' + [enter] to quit.");
	            BufferedReader commandLine = new java.io.BufferedReader(new InputStreamReader(System.in));
	
	            while (true) {
	                String s = commandLine.readLine();
	                if (s.equalsIgnoreCase("exit")) {
	                    simpleSenderReceiver.close();
	                    System.exit(0);
	                } else {
	                    simpleSenderReceiver.sendMessage();
	                }
	            }
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	    }
	
	    private void sendMessage() throws JMSException {
	        TextMessage message = sendSession.createTextMessage();
	        message.setText("Test AMQP message from JMS");
	        long randomMessageID = randomGenerator.nextLong() >>>1;
	        message.setJMSMessageID("ID:" + randomMessageID);
	        sender.send(message);
	        System.out.println("Sent message with JMSMessageID = " + message.getJMSMessageID());
	    }
	
	    public void close() throws JMSException {
	        connection.close();
	    }
	
	    public void onMessage(Message message) {
	        try {
	            System.out.println("Received message with JMSMessageID = " + message.getJMSMessageID());
	            message.acknowledge();
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	    }
	}	

### Ausführen der Anwendung

Das Ergebnis nach Ausführen der Anwendung sieht folgendermaßen aus:

	> java SimpleSenderReceiver
	Press [enter] to send a message. Type 'exit' + [enter] to quit.
	
	Sent message with JMSMessageID = ID:2867600614942270318
	Received message with JMSMessageID = ID:2867600614942270318
	
	Sent message with JMSMessageID = ID:7578408152750301483
	Received message with JMSMessageID = ID:7578408152750301483
	
	Sent message with JMSMessageID = ID:956102171969368961
	Received message with JMSMessageID = ID:956102171969368961
	exit

## Plattformübergreifendes Messaging mit JMS und .NET

In diesem Leitfaden wurde bisher gezeigt, wie Nachrichten mit JMD an den Service Bus gesendet und von diesem empfangen werden. Ein wesentlicher Vorteil von AMQP 1.0 besteht darin, dass Anwendungen aus Komponenten erstellt werden können, die in unterschiedlichen Sprachen geschrieben wurden. Weiterhin werden Nachrichten zuverlässig und sicher ausgetauscht.

Mithilfe der zuvor beschriebenen beispielhaften JMD-Anwendung und einer ähnlichen .NET-Anwendung aus der separaten Anleitung [Verwenden von AMQP 1.0 mit der .NET-Programmierschnittstelle für Service Bus](http://aka.ms/lym3vk) können Nachrichten zwischen .NET und Java ausgetauscht werden. 

Weitere Informationen zum plattformübergreifenden Messaging mit Service Bus und AMQP 1.0 finden Sie im [Entwicklerhandbuch für Service Bus mit AMQP 1.0](http://msdn.microsoft.com/library/jj841071.aspx).

### Messaging von JMS nach .NET

So funktioniert das Messaging von JMS nach .NET:

* Starten Sie die .NET-Beispielanwendung ohne Befehlszeilenargumente.
* Starten Sie anschließend die beispielhafte Java-Anwendung mit dem Befehlszeilenargument "sendonly". Die Anwendung empfängt in diesem Modus keine Nachrichten aus der Warteschlange, sie sendet lediglich Nachrichten.
* Drücken Sie mehrmals die **EINGABETASTE** in der Java-Anwendungskonsole, damit die Nachrichten gesendet werden.
* Diese Nachrichten werden anschließend von der .NET-Anwendung empfangen.

<p><strong>Ausgabe der JMS-Anwendung</strong></p>

	> java SimpleSenderReceiver sendonly
	Press [enter] to send a message. Type 'exit' + [enter] to quit.
	Sent message with JMSMessageID = ID:4364096528752411591
	Sent message with JMSMessageID = ID:459252991689389983
	Sent message with JMSMessageID = ID:1565011046230456854
	exit

<p><strong>Ausgabe der .NET-Anwendung</strong></p>

	> SimpleSenderReceiver.exe	
	Press [enter] to send a message. Type 'exit' + [enter] to quit.
	Received message with MessageID = 4364096528752411591
	Received message with MessageID = 459252991689389983
	Received message with MessageID = 1565011046230456854
	exit

### Messaging von .NET nach JMS

So funktioniert das Messaging von .NET nach JMS:

* Starten Sie die .NET-Beispielanwendung mit dem Befehlszeilenargument "sendonly". Die Anwendung empfängt in diesem Modus keine Nachrichten aus der Warteschlange, sie sendet lediglich Nachrichten.
* Starten Sie die beispielhafte Java-Anwendung ohne Befehlszeilenargumente.
* Drücken Sie mehrmals die **EINGABETASTE** in der .NET-Anwendungskonsole, damit die Nachrichten gesendet werden.
* Diese Nachrichten werden anschließend von der Java-Anwendung empfangen.

<p><strong>Ausgabe der .NET-Anwendung</strong></p>

	> SimpleSenderReceiver.exe sendonly
	Press [enter] to send a message. Type 'exit' + [enter] to quit.
	Sent message with MessageID = d64e681a310a48a1ae0ce7b017bf1cf3	
	Sent message with MessageID = 98a39664995b4f74b32e2a0ecccc46bb
	Sent message with MessageID = acbca67f03c346de9b7893026f97ddeb
	exit


<p><strong>Ausgabe der JMS-Anwendung</strong></p>

	> java SimpleSenderReceiver	
	Press [enter] to send a message. Type 'exit' + [enter] to quit.
	Received message with JMSMessageID = ID:d64e681a310a48a1ae0ce7b017bf1cf3
	Received message with JMSMessageID = ID:98a39664995b4f74b32e2a0ecccc46bb
	Received message with JMSMessageID = ID:acbca67f03c346de9b7893026f97ddeb
	exit

## Nicht unterstützte Funktionen und Einschränkungen

Bei der Verwendung von JMS über AMQP 1.0 mit Service Bus gelten die folgenden Einschränkungen:

* Nur ein **MessageProducer** oder **MessageConsumer** ist je **Sitzung** zulässig. Falls Sie mehrere **MessageProducers** oder **MessageConsumers** in einer Anwendung benötigen, müssen Sie für diese jeweils eine **Sitzung** erstellen.
* Flüchtige Themenabonnements werden momentan nicht unterstützt.
* **MessageSelectors** werden derzeit nicht unterstützt.
* Temporäre Ziele, d. h. **TemporaryQueue**, **TemporaryTopic** werden derzeit nicht unterstützt, das gilt auch für die **QueueRequestor** und **TopicRequestor**-APIs, die diese verwenden.
* Durchgeführte Sitzungen und verteilte Transaktionen werden nicht unterstützt.

## Zusammenfassung

In diesem Leitfaden wurde gezeigt, wie die gebrokerten Messagingfunktionen von Service Bus (Warteschlange und Themen veröffentlichen/abonnieren) aus Java-Anwendungen mit der beliebten Standard-Programmierschnittstelle JMS und AMQP 1.0 verwendet werden.

Sie können Service Bus AMQP 1.0 auch mit anderen Sprachen verwenden, unter anderem .NET, C, Python und PHP. Komponenten, die mit diesen verschiedenen Sprachen geschrieben wurden, können mit der AMQP 1.0-Unterstützung in Service Bus Nachrichten zuverlässig und bei voller Vertraulichkeit austauschen. Weitere Informationen finden Sie im [Entwicklerhandbuch für Service Bus mit AMQP 1.0](http://msdn.microsoft.com/library/jj841071.aspx).

## Weitere Informationen

* [AMQP 1.0-Unterstützung in Azure Service Bus](http://aka.ms/pgr3dp)
* [Verwenden von AMQP 1.0 mit der .NET-Programmierschnittstelle für Service Bus](http://aka.ms/lym3vk)
* [Entwicklerhandbuch zu Service Bus AMQP 1.0](http://msdn.microsoft.com/library/jj841071.aspx)
* [Verwenden von Service Bus-Warteschlangen](http://azure.microsoft.com/develop/net/how-to-guides/service-bus-queues/)


<!--HONumber=47-->
 