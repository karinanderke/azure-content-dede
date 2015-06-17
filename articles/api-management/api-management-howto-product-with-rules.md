<properties
	pageTitle="Schützen Sie Ihre API mit Begrenzung der Datenübertragungsrate Verwendung von Azure-API Management"
	description="Erfahren Sie, wie die API mit Kontingente und Einschränkungen (Bandbreitenkontrolle) Richtlinien zu schützen."
	services="api-management"
	documentationCenter=""
	authors="steved0x"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="api-management"
	ms.workload="mobile"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article" 
	ms.date="06/10/2015"
	ms.author="sdanie"/>

# Schützen Sie Ihre API mit Begrenzung der Datenübertragungsrate Verwendung von Azure-API Management

Dieses Handbuch zeigt Ihnen, wie einfach es ist, fügen Sie Schutz für Ihre Back-End-API Rate und Kontingente Richtlinien mit Azure API-Verwaltung konfigurieren.

Weitere Informationen und eine Demo der Begrenzung der Datenübertragungsrate und Kontingente festlegen finden Sie im folgende Video.

> [AZURE.VIDEO rate-limits-and-quotas]

In diesem Lernprogramm erstellen Sie ein Produkt "Kostenlose Testversion" API, die Entwicklern ermöglicht, bis zu 10 pro Minute und bis zu maximal 200 Aufrufen pro Woche auf Ihren API-aufrufen. Sie werden dann die API veröffentlichen und testen die Richtlinie für das bewerten.

>[AZURE.NOTE]Wenn Sie bereits ein Produkt konfiguriert haben und sie für dieses Lernprogramm verwenden, Sie können gehen Sie direkt zu [Konfigurieren aufrufen Rate und Kontingente Richtlinien][] und führen Sie das Lernprogramm dort mit Ihrem Produkt anstelle von der **kostenlose Testversion** Produkt.

## <a name="create-product"> </a>Erstellen eines Produkts

In diesem Schritt erstellen Sie ein kostenloses Testprodukt, für das keine Abonnementgenehmigung erforderlich ist.

Klicken Sie zunächst im Azure-Portal für Ihren API Management-Dienst auf **Verwalten**. Daraufhin gelangen Sie zum API Management-Herausgeberportal.

![Herausgeberportal][api-management-management-console]

>Falls Sie noch keine API Management-Dienstinstanz erstellt haben, finden Sie weitere Informationen im Abschnitt [Erstellen einer API Management-Dienstinstanz][] im Lernprogramm [Erste Schritte mit Azure API Management][].

Klicken Sie auf **Produkte** im Menü **API-Verwaltung** auf der linken Seite, um Seite **Produkte** anzuzeigen.

![Produkt hinzufügen][api-management-add-product]

Klicken Sie auf **Produkt hinzufügen**, um das Popupfenster **Neues Produkt hinzufügen** anzuzeigen.

![Neues Produkt hinzufügen][api-management-new-product-window]

Geben Sie **Kostenloser Test** in das Textfeld **Titel** ein.

Geben Sie **Abonnenten können bis zu 10 Aufrufe pro Minute und bis zu 200 Aufrufe pro Woche ausführen, danach wird der Zugriff verweigert.** in das Textfeld **Beschreibung** ein.

Produkte in der API-Verwaltung kann **Öffnen** oder **Protected**. Geschützte Produkte müssen abonniert werden, bevor Sie verwendet werden können, während offene Produkte ohne Abonnement genutzt werden können. Sicherstellen, dass **erfordern Abonnement** wird überprüft, um eine geschützte Produkt zu erstellen, ein Abonnement erforderlich ist. Dies ist die Standardeinstellung.

Wenn Sie möchten, dass ein Administrator Abonnements für dieses Produkt prüfen und ablehnen oder akzeptieren muss, markieren Sie die Option **Abonnementgenehmigung erforderlich**. Wenn dieses Feld nicht markiert ist, werden Abonnements automatisch genehmigt. In diesem Beispiel werden Abonnements automatisch genehmigt, markieren Sie dieses Kontrollkästchen also nicht.

Damit entwicklerkonten mehrmals das neue Produkt abonnieren können, überprüfen Sie die **mehrere gleichzeitige Abonnements zulassen** Kontrollkästchen. In diesem Thema nicht mehrere gleichzeitige Abonnements nutzen, also lassen Sie es deaktiviert.

Geben Sie alle Werte ein und klicken Sie auf **Speichern**, um das Produkt zu erstellen.

![Hinzugefügtes Produkt][api-management-product-added]

Standardmäßig sind neue Produkte für Benutzer in der Gruppe **Administratoren** sichtbar. Wir werden die Gruppe **Entwickler** hinzufügen. Klicken Sie auf **kostenlose Testversion**, und wählen Sie die **Sichtbarkeit** Registerkarte.

>Die API-Verwaltung verwendet Gruppen, um die Sichtbarkeit von Produkten für Entwickler zu verwalten. Produkte gewähren Sichtbarkeit für Gruppen, und Entwickler können alle Produkte anzeigen und abonnieren, die für die Gruppen sichtbar sind, in denen sie Mitglied sind. Weitere Informationen finden Sie unter [Erstellen und Verwenden von Gruppen in der Azure API-Verwaltung][].

![Hinzufügen der Gruppe "Entwickler"][api-management-add-developers-group]

Markieren Sie die Gruppe **Entwickler** und klicken Sie auf **Speichern**.

## <a name="add-api"> </a>Eine API für das Produkt hinzufügen

In diesem Schritt des Lernprogramms werden Sie die Echo API zu dem neu erstellten Produkt "Kostenloser Test" hinzufügen.

>Jede API-Verwaltungs-Dienstinstanz enthält eine vorkonfigurierte Echo-API, die Sie für Tests und erste Schritte mit der API-Verwaltung nutzen können. Weitere Informationen finden Sie unter [Erste Schritte mit der Azure API-Verwaltung][].

Klicken Sie auf **Produkte** im Menü **API-Verwaltung** auf der linken Seite und klicken Sie auf **Kostenloser Test**, um das Produkt zu konfigurieren.

![Produkt konfigurieren][api-management-configure-product]

Klicken Sie auf **API zu Produkt hinzufügen**.

![API zum Produkt hinzufügen][api-management-add-api]

Markieren Sie das Kontrollkästchen neben **Echo API** und klicken Sie auf **Speichern**.

![Echo API hinzufügen][api-management-add-echo-api]

## <a name="policies"> </a>Konfigurieren aufrufen Rate und Kontingente Richtlinien

Durchsatzgrenzen und Kontingente werden im Richtlinien-Editor konfiguriert. Klicken Sie auf **Richtlinien** im Menü **API-Verwaltung** auf der linken Seite und klicken Sie auf **Kostenloser Test** in der Dropdownliste **Richtlinien-Produktbereich**.

![Produktrichtlinie][api-management-product-policy]

Klicken Sie auf **Richtlinie hinzufügen**, um die Richtlinienvorlage zu importieren und die Richtlinie für Durchsatzgrenzen und Kontingente zu erstellen.

![Richtlinie hinzufügen][api-management-add-policy]

Um Richtlinien hinzuzufügen, platzieren Sie den Cursor entweder im **Eingehend**- oder **Ausgehend**-Bereich der Richtlinienvorlage. Richtlinien für Durchsatzgrenzen und Kontingente sind eingehende Richtlinien. Platzieren Sie den Cursor also im Element für eingehende Richtlinien.

![Richtlinieneditor][api-management-policy-editor-inbound]

In diesem Lernprogramm werden wir die Richtlinien [Aufrufbeschränkung][] und [Nutzungskontingent][] erstellen.

![Richtlinienanweisungen][api-management-limit-policies]

Platzieren Sie den Cursor in das **Eingehend**-Richtlinienelement und klicken Sie auf den Pfeil neben **Aufrufbeschränkung**, um die Richtlinienvorlage einzufügen.

	<rate-limit calls="number" renewal-period="seconds">
	<api name="name" calls="number">
	<operation name="name" calls="number" />
	</api>
	</rate-limit>

**Aufrufbeschränkung** kann auf der Produktebene sowie auf der API- und auf der Namensebene für einzelne Operationen verwendet werden. In diesem Lernprogramm nur Produktrichtlinien verwendet werden, löschen Sie die **api** und **Vorgang** Elemente aus der **Ratenlimit** -Element, sodass nur die äußere **Obergrenze für die Framerate** Element bleibt, wie im folgenden Beispiel gezeigt.

	<rate-limit calls="number" renewal-period="seconds">
	</rate-limit>

Das Produkt **Kostenloser Test** soll bis zu 10 Aufrufe pro Minute erlauben. Geben Sie also **10** als Wert für das calls-Attribut ein und **60** für das **renewal-period**-Attribut.

	<rate-limit calls="10" renewal-period="60">
	</rate-limit>

Um die **Nutzungskontingent**-Richtlinie zu konfigurieren, platzieren Sie Ihren Cursor direkt unterhalb des neu erstellten **rate-limit**-Elements innerhalb des **inbound**-Elements und klicken Sie auf den Pfeil links neben **Nutzungskontingent einstellen**.

	<quota calls="number" bandwidth="kilobytes" renewal-period="seconds">
	<api name="name" calls="number" bandwidth="kilobytes">
	<operation name="name" calls="number" bandwidth="kilobytes" />
	</api>
	</quota>

Da diese Richtlinie auf der Produktebene arbeiten soll, löschen Sie die Elemente mit den Namen **api** und **operation**, wie im folgenden Beispiel gezeigt.

	<quota calls="number" bandwidth="kilobytes" renewal-period="seconds">
	</quota>

Kontingente können basierend auf der Anzahl von Aufrufen pro Intervall, der Bandbreite oder beiden Werten konfiguriert werden. In diesem Lernprogramm drosseln wir nicht nach Bandbreite. Löschen Sie also das **bandwidth**-Attribut.

	<quota calls="number" renewal-period="seconds">
	</quota>

Für das Produkt **Kostenloser Test** gilt ein Kontingent von 200 Aufrufen pro Woche. Geben Sie **200** als Wert für das calls-Attribut ein, und **604800** als Wert für renewal-period.

	<quota calls="200" renewal-period="604800">
	</quota>

>Die Richtlinienintervalle werden in Sekunden angegeben. Um die Intervalle für eine Woche zu berechnen, multiplizieren Sie die Anzahl der Tage (7) mit der Anzahl der Stunden pro Tag (24), der Anzahl der Minuten pro Stunde (60) und der Anzahl von Sekunden pro Minute (60). 7 * 24 * 60 * 60 = 604800.

Konfigurieren Sie die Richtlinie, sodass diese dem folgenden Beispiel entspricht.

	<policies>
		<inbound>
			<rate-limit calls="10" renewal-period="60">
			</rate-limit>
			<quota calls="200" renewal-period="604800">
			</quota>
			<base />

	</inbound>
	<outbound>

		<base />

		</outbound>
	</policies>

Wenn Sie die gewünschten Richtlinien konfiguriert haben, klicken Sie auf **Speichern**.

![Richtlinie speichern][api-management-policy-save]

## <a name="publish-product"> </a> Produkt veröffentlichen

Sie haben die APIs hinzugefügt und die Richtlinien konfiguriert, und das Produkt kann jetzt von Entwicklern aufgerufen werden. Bevor die Entwickler das Produkt nutzen können, müssen Sie es veröffentlichen. Klicken Sie auf **Produkte** im Menü **API-Verwaltung** auf der linken Seite und klicken Sie auf **Kostenloser Test**, um das Produkt zu konfigurieren.

![Produkt konfigurieren][api-management-configure-product]

Klicken Sie auf **Veröffentlichen** und anschließend zur Bestätigung auf **Ja, veröffentlichen**.

![Veröffentlichen des Produkts][api-management-publish-product]

## <a name="subscribe-account"> </a>Ein Entwicklerkonto für das Produkt abonnieren

Ihr Produkt ist nun veröffentlicht und kann von Entwicklern abonniert und verwendet werden.

>Administratoren einer API-Verwaltungs-Instanz werden automatisch für alle Produkte abonniert. In diesem Schritt des Lernprogramms werden wir das Produkt "Kostenloser Test" mit einem der nicht-Administratorkonten abonnieren. Falls Ihr Entwicklerkonto Teil der Administratorrolle ist, können Sie diesen Schritt dennoch mitverfolgen, auch wenn Sie das Produkt bereits abonniert haben.

Klicken Sie auf **Benutzer** auf der **API Management** im Menü auf der linken Seite, und klicken Sie auf den Namen Ihres Entwicklerkontos. In diesem Beispiel verwenden wir das Entwicklerkonto **Clayton Gragg**.

![Entwickler konfigurieren][api-management-configure-developer]

Klicken Sie auf **Abonnement hinzufügen**.

![Abonnement hinzufügen][api-management-add-subscription-menu]

Markieren Sie das Kontrollkästchen neben **Kostenloser Test** und klicken Sie auf **Abonnieren**.

![Abonnement hinzufügen][api-management-add-subscription]

>[AZURE.NOTE]In diesem Lernprogramm sind nicht mehrere gleichzeitige Abonnements für aktiviert die **kostenlose Testversion** Produkt. Wären sie würden Sie aufgefordert werden, benennen Sie das Abonnement wie im folgenden Beispiel gezeigt.

![Abonnement hinzufügen][api-management-add-subscription-multiple]

Nach dem Klicken auf **Abonnieren**, das Produkt wird in der **Abonnement** Liste für den Benutzer.

![Abonnement hinzugefügt][api-management-subscription-added]

## <a name="test-rate-limit"> </a>Aufrufen eines Vorgangs und Testen des Übertragungsratenlimits

Sie haben das Produkt "Kostenloser Test" nun konfiguriert und veröffentlicht und können Operationen aufrufen, um die Richtlinie für die Durchsatzgrenze zu testen. Klicken Sie im Menü oben rechts auf **Entwicklerportal**, um zum Entwicklerportal zu gelangen.

![Entwicklerportal][api-management-developer-portal-menu]

Klicken Sie auf **APIs** im Hauptmenü, und wählen Sie **Echo API** aus.

![Entwicklerportal][api-management-developer-portal-api-menu]

Wählen Sie die Operation **GET Resource** aus, und klicken Sie auf **Konsole öffnen**.

![Konsole öffnen][api-management-open-console]

Behalten Sie die Standard-Parameterwerte bei und wählen Sie Ihren Abonnementschlüssel für das Produkt **Kostenloser Test** aus.

![Abonnementschlüssel][api-management-select-key]

>[AZURE.NOTE]Falls Sie mehrere Abonnements haben, stellen Sie sicher, dass Sie den Schlüssel für **Kostenloser Test** auswählen. Andernfalls treten die in den vorherigen Schritten konfigurierten Richtlinien nicht in Kraft.

Klicken Sie auf **HTTP Get** und sehen Sie sich die Antwort an. Beachten Sie den **Antwortstatus** von **200 OK**.

![Operationsergebnis][api-management-http-get-results]

Klicken Sie häufiger als das konfigurierte Limit von 10 mal pro Minute auf **HTTP Get**. Sobald die Durchsatzgrenze überschritten ist, wird der Antwortstatus **429 Too many Requests** zurückgegeben.

![Operationsergebnis][api-management-http-get-429]

Die **Antwortheader** und der **Antworttext** enthalten das verbleibende Intervall, bis neue Aufrufe möglich sind.

Wenn die Durchsatzgrenze von 10 Aufrufen pro Minute aktiv ist, werden nachfolgende Aufrufe fehlschlagen, bis 60 Sekunden nach dem ersten der 10 erfolgreichen Aufrufe an das Produkt vergangen sind, bevor die Durchsatzgrenze überschritten wurde. In diesem Beispiel beträgt das verbleibende Intervall 43 Sekunden.

## <a name="next-steps"> </a>Nächste Schritte

-	Lesen Sie die anderen Themen im Lernprogramm [Erste Schritte bei der erweiterten API-Konfiguration][].


[api-management-management-console]: ./media/api-management-howto-product-with-rules/api-management-management-console.png
[api-management-add-product]: ./media/api-management-howto-product-with-rules/api-management-add-product.png
[api-management-new-product-window]: ./media/api-management-howto-product-with-rules/api-management-new-product-window.png
[api-management-product-added]: ./media/api-management-howto-product-with-rules/api-management-product-added.png
[api-management-add-policy]: ./media/api-management-howto-product-with-rules/api-management-add-policy.png
[api-management-policy-editor-inbound]: ./media/api-management-howto-product-with-rules/api-management-policy-editor-inbound.png
[api-management-limit-policies]: ./media/api-management-howto-product-with-rules/api-management-limit-policies.png
[api-management-policy-save]: ./media/api-management-howto-product-with-rules/api-management-policy-save.png
[api-management-configure-product]: ./media/api-management-howto-product-with-rules/api-management-configure-product.png
[api-management-add-api]: ./media/api-management-howto-product-with-rules/api-management-add-api.png
[api-management-add-echo-api]: ./media/api-management-howto-product-with-rules/api-management-add-echo-api.png
[api-management-developer-portal-menu]: ./media/api-management-howto-product-with-rules/api-management-developer-portal-menu.png
[api-management-publish-product]: ./media/api-management-howto-product-with-rules/api-management-publish-product.png
[api-management-configure-developer]: ./media/api-management-howto-product-with-rules/api-management-configure-developer.png
[api-management-add-subscription-menu]: ./media/api-management-howto-product-with-rules/api-management-add-subscription-menu.png
[api-management-add-subscription]: ./media/api-management-howto-product-with-rules/api-management-add-subscription.png
[api-management-developer-portal-api-menu]: ./media/api-management-howto-product-with-rules/api-management-developer-portal-api-menu.png
[api-management-open-console]: ./media/api-management-howto-product-with-rules/api-management-open-console.png
[api-management-http-get]: ./media/api-management-howto-product-with-rules/api-management-http-get.png
[api-management-http-get-results]: ./media/api-management-howto-product-with-rules/api-management-http-get-results.png
[api-management-http-get-429]: ./media/api-management-howto-product-with-rules/api-management-http-get-429.png
[api-management-product-policy]: ./media/api-management-howto-product-with-rules/api-management-product-policy.png
[api-management-add-developers-group]: ./media/api-management-howto-product-with-rules/api-management-add-developers-group.png
[api-management-select-key]: ./media/api-management-howto-product-with-rules/api-management-select-key.png
[api-management-subscription-added]: ./media/api-management-howto-product-with-rules/api-management-subscription-added.png
[api-management-add-subscription-multiple]: ./media/api-management-howto-product-with-rules/api-management-add-subscription-multiple.png

[How to add operations to an API]: api-management-howto-add-operations.md
[How to add and publish a product]: api-management-howto-add-products.md
[Monitoring and analytics]: ../api-management-monitoring.md
[Add APIs to a product]: api-management-howto-add-products.md#add-apis
[Publish a product]: api-management-howto-add-products.md#publish-product
[Get started with Azure API Management]: api-management-get-started.md
[Erstellen und Verwenden von Gruppen in der Azure API-Verwaltung]: api-management-howto-create-groups.md
[View subscribers to a product]: api-management-howto-add-products.md#view-subscribers
[Erste Schritte mit Azure API Management]: api-management-get-started.md
[Erste Schritte mit der Azure API-Verwaltung]: api-management-get-started.md
[Erstellen einer API Management-Dienstinstanz]: api-management-get-started.md#create-service-instance
[Next steps]: #next-steps

[Create a product]: #create-product
[Konfigurieren aufrufen Rate und Kontingente Richtlinien]: #policies
[Add an API to the product]: #add-api
[Publish the product]: #publish-product
[Subscribe a developer account to the product]: #subscribe-account
[Call an operation and test the rate limit]: #test-rate-limit
[Erste Schritte bei der erweiterten API-Konfiguration]: api-management-get-started-advanced.md

[Aufrufbeschränkung]: https://msdn.microsoft.com/library/azure/dn894078.aspx#LimitCallRate
[Nutzungskontingent]: https://msdn.microsoft.com/library/azure/dn894078.aspx#SetUsageQuota
 

<!---HONumber=GIT-SubDir_Tue_AM_dede-->