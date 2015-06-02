<properties 
	pageTitle="Konfigurieren eines Network Solutions-Domänennamens für eine Azure-Website" 
	description="Erfahren Sie, wie Sie einen Domänennamen von Network Solutions mit Azure-Websites verwenden" 
	services="web-sites" 
	documentationCenter="" 
	authors="blackmist" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="web-sites" 
	ms.workload="web" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/17/2014" 
	ms.author="larryfr, jroth"/>

# Konfigurieren eines benutzerdefinierten Domänennamens für eine Azure-Website (Network Solutions)

<div class="dev-center-tutorial-selector sublanding"><a href="/documentation/articles/web-sites-custom-domain-name" title="Benutzerdefinierte Domäne">Benutzerdefinierte Domäne</a><a href="/documentation/articles/web-sites-godaddy-custom-domain-name" title="GoDaddy">GoDaddy</a><a href="/documentation/articles/web-sites-network-solutions-custom-domain-name" title="Network Solutions" class="current">Network Solutions</a><a href="/documentation/articles/web-sites-registerdotcom-custom-domain-name" title="Register.com">Register.com</a><a href="/documentation/articles/web-sites-enom-custom-domain-name" title="Enom">Enom</a><a href="/documentation/articles/web-sites-moniker-custom-domain-name" title="Moniker">Moniker</a><a href="/documentation/articles/web-sites-dotster-custom-domain-name" title="Dotster">Dotster</a><a href="/documentation/articles/web-sites-domaindiscover-custom-domain-name" title="DomainDiscover">DomainDiscover</a><a href="/documentation/articles/web-sites-directnic-custom-domain-name" title="Directnic">Directnic</a></div>
<div class="dev-center-tutorial-subselector"><a href="/documentation/articles/web-sites-network-solutions-custom-domain-name/" title="Websites" class="current">Website</a> | <a href="/documentation/articles/web-sites-network-solutions-traffic-manager-custom-domain-name/" title="Website mit Verwendung von Traffic Manager">Website mit Verwendung von Traffic Manager</a></div>

[AZURE.INCLUDE [websites-cloud-services-css-guided-walkthrough](../includes/websites-cloud-services-css-guided-walkthrough.md)]

[AZURE.INCLUDE [Einführung](../includes/custom-dns-web-site-intro.md)]

Dieser Artikel enthält Anweisungen zur Verwendung eines über [Network Solutions](https://www.networksolutions.com) erworbenen benutzerdefinierten Domänennamens mit Azure-Websites.

[AZURE.INCLUDE [introfooter](../includes/custom-dns-web-site-intro-notes.md)]

Themen in diesem Artikel:

-   [Grundlegendes zu DNS-Einträgen](#understanding-records)
-   [Konfigurieren Ihrer Websites für die Modi "Basic", "Freigegeben" oder "Standard"](#bkmk_configsharedmode)
-   [Hinzufügen eines DNS-Datensatzes zu Ihrer benutzerdefinierten Domäne](#bkmk_configurecname)
-   [Aktivieren der Domäne auf Ihrer Website](#enabledomain)

<h2><a name="understanding-records"></a>Interpretation von DNS-Datensätzen</h2>

[AZURE.INCLUDE [understandingdns](../includes/custom-dns-web-site-understanding-dns-raw.md)]

<h2><a name="bkmk_configsharedmode"></a>Konfigurieren Ihrer Websites für die Modi "Basic", "Freigegeben" oder "Standard"</h2>

[AZURE.INCLUDE [Modi](../includes/custom-dns-web-site-modes.md)]

<a name="bkmk_configurecname"></a><h2>Hinzufügen eines DNS-Eintrags für Ihre benutzerdefinierte Domäne</h2>

Um Ihrer benutzerdefinierten Domäne eine Azure-Website zuzuweisen, müssen Sie in der DNS-Tabelle für Ihre benutzerdefinierte Domäne mithilfe der von Network Solutions bereitgestellten Tools einen neuen Eintrag hinzufügen. Gehen Sie wie folgt vor, um die DNS-Tools von networksolutions.com zu suchen und zu verwenden:

1. Melden Sie sich bei networksolutions.com an und wählen Sie oben rechts **My Account** (Mein Konto).

3. Wählen Sie auf der Registerkarte **My Products and Services** (Meine Produkte und Services) **Edit DNS** (DNS bearbeiten) aus.

	![DNS-Seite bearbeiten](./media/web-sites-custom-domain-name/ns-editdns.png)

2. Wählen Sie im Bereich **Manage <yourdomainname>** (Verwalten) der Seite **Domain Names** (Domänennamen) die Option **Edit Advanced DNS Records** (Erweiterte DNS-Einträge bearbeiten) aus.

	![Seite der Domänennamen – Bearbeiten erweiterter DNS-Datensätze hervorgehoben](./media/web-sites-custom-domain-name/ns-editadvanced.png)

4. Auf der Seite **Update Advanced DNS** (Erweiterten DNS-Datensatz aktualisieren) befindet sich ein Abschnitt für jede Art von Datensatz, unter dem sich die Schaltfläche **Edit** (Bearbeiten) befindet.
	
	* Verwenden Sie für A-Datensätze den Abschnitt **IP Address (A Records)** (IP-Adresse (A-Datensätze)).
	* Verwenden Sie für CNAME-Datensätze den Abschnitt **Host Alias (CNAME Records)** (Host Alias (CNAME-Datensätze)).

	![Seite für die Aktualisierung erweiterter DNS-Datensätze](./media/web-sites-custom-domain-name/ns-updateadvanced.png)

5. Wenn Sie auf **Edit** (Bearbeiten) klicken, wird Ihnen ein Formular angezeigt, das Sie verwenden können, um bestehende Datensätze zu ändern oder neue hinzuzufügen.

	> [AZURE.NOTE]Bevor Sie neue Einträge hinzufügen, beachten Sie bitte, dass Network Solutions einige Standard-DNS-Einträge wie die Stammdomäne ('@') und für Unterdomänen den Platzhalter ('*') erstellt hat. Sollte der Datensatz, den Sie verwenden möchten, bereits bestehen, ändern Sie ihn, statt einen neuen zu erstellen.

	* Wenn Sie einen CNAME-Datensatz hinzufügen, müssen Sie im Feld **Alias** (Alias) zuerst die Unterdomäne festlegen, die Sie verwenden möchten. Beispiel: **www**. Wählen Sie das Kreisfeld neben dem Feld **Other host** (Sonstiger Host) aus, und legen Sie **Other host** (Sonstiger Host) auf den **.azurewebsites.net**-Domänennamen Ihrer Azure-Website fest. Beispiel: **contoso.azurwebsites.net**. Lassen Sie im Feld **Refers to Host Name** (Bezug auf Hostname) **Select** (Auswählen) stehen, da dieses Feld nicht benötigt wird, wenn ein CNAME-Datensatz für die Verwendung mit einer Azure-Website erstellt wird.
	
		![CNAME-Formular](./media/web-sites-custom-domain-name/ns-cname.png)

		> [AZURE.NOTE]Wenn Sie einen A-Datensatz verwenden, müssen Sie auch einen CNAME-Datensatz mit der folgenden Konfiguration hinzufügen:
		> 
		> * Einen **Alias** mit dem Wert **www**, wobei der Wert **Sonstiger Host** **&lt;ihrwebsitename&gt;.azurewebsites.net** lautet.
		> 
		> OR
		> 
		> * Ein **Alias** mit dem Wert **awverify.www**, wobei der Wert **Sonstiger Host** **awverify.&lt;ihrwebsitename&gt;.azurewebsites.net** lautet.
		> 
		> Der CNAME-Datensatz wird von Azure verwendet, um zu prüfen, ob Sie die Domäne, die vom A-Datensatz beschrieben wird, besitzen.

	* Wenn Sie einen A-Datensatz hinzufügen, müssen Sie das Feld **Host** entweder auf **@** (präsentiert den Stammdomänennamen wie z. B. **contoso.com**), * (ein Platzhalter für verschiedene passende Unterdomänen) oder die Unterdomäne, die Sie verwenden möchten (Beispiel: **www**) festlegen. Geben Sie im Feld **Numeric IP** (Numerische IP) die IP-Adresse Ihrer Azure-Website an.

		![Datensatzformular](./media/web-sites-custom-domain-name/ns-arecord.png)

5. Wenn Sie die Datensätze hinzugefügt oder geändert haben, klicken Sie auf **Continue** (Weiter), um die Änderungen anzuzeigen. Wählen Sie **Save changes only** (Nur Änderungen speichern), um die Änderungen zu speichern.

<h2><a name="enabledomain"></a>Aktivieren des Domänennamens auf Ihrer Website</h2>

[AZURE.INCLUDE [Modi](../includes/custom-dns-web-site-enable-on-web-site.md)]

<!--HONumber=54-->