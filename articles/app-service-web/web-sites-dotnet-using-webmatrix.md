<properties 
	pageTitle="Entwickeln und Bereitstellen von Web-Apps mit Microsoft WebMatrix" 
	description="Erfahren Sie, wie Sie eine ASP.NET-Webanwendung in Azure App Service-Web-Apps mit Microsoft WebMatrix entwickeln und bereitstellen." 
	services="app-service\web" 
	documentationCenter=".net" 
	authors="tfitzmac" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service-web" 
	ms.workload="web" 
	ms.tgt_pltfrm="na" 
	ms.devlang="dotnet" 
	ms.topic="article" 
	ms.date="05/15/2015" 
	ms.author="tomfitz"/>


# Entwickeln und Bereitstellen von Web-Apps mit Microsoft WebMatrix

## Übersicht

Dieser Leitfaden beschreibt, wie Sie mit Microsoft WebMatrix eine .NET-Website erstellen und in [Azure App Service](http://go.microsoft.com/fwlink/?LinkId=529714)-Web-Apps bereitstellen. Sie verwenden hierzu eine Beispielanwendung einer WebMatrix-Websitevorlage.

Sie erhalten Informationen zu folgenden Themen:

* Anmelden bei Azure von WebMatrix aus
* Erstellen einer Site mit einer in WebMatrix integrierten Vorlage 
* Bereitstellen der benutzerdefinierten Website direkt über WebMatrix in Azure

[AZURE.INCLUDE [create-account-and-websites-note](../../includes/create-account-and-websites-note.md)]

>[AZURE.NOTE]Wenn Sie Azure App Service ausprobieren möchten, ehe Sie sich für ein Azure-Konto anmelden, können Sie unter [App Service testen](http://go.microsoft.com/fwlink/?LinkId=523751) sofort kostenlos eine kurzlebige Starter-Web-App in App Service erstellen. Keine Kreditkarte erforderlich, keine Verpflichtungen.

## Anmelden bei Azure

1. Starten Sie WebMatrix.
2. Wenn Sie WebMatrix 3 zum ersten Mal verwenden, werden Sie aufgefordert, sich bei Azure anzumelden. Klicken Sie sonst auf die Schaltfläche **Anmelden**, und wählen Sie **Konto hinzufügen**. Melden Sie sich mit Ihrem Microsoft-Konto an.

	![Konto hinzufügen][addaccount]

3. Wenn Sie sich für ein Azure-Konto angemeldet haben, können Sie sich mit Ihrem Microsoft-Konto anmelden:

	![Anmelden bei Azure][signin]


## Erstellen einer Site für Azure mit einer integrierten Vorlage

1. Klicken Sie im Startbildschirm auf die Schaltfläche **Neu**, und wählen Sie **Vorlagenkatalog**, um eine neue Site aus dem Vorlagenkatalog zu erstellen:

	![Neue Site aus Vorlagenkatalog][sitefromtemplate]

2. Im Vorlagenkatalog wird eine Liste der verfügbaren Vorlagen angezeigt, die lokal oder auf Azure ausgeführt werden können. Wählen Sie die Vorlage **Konditorei** aus der Liste, geben Sie **Konditoreibeispiel** im Feld **Websitename** ein, und klicken Sie auf **Weiter**.

	![Site aus Vorlage erstellen][sitefromtemplatedetails]

3. Wenn Sie bei Azure angemeldet sind, können Sie nun eine Azure App Service-Web- Apps-Instanz für eine lokale Site erstellen. Wählen Sie einen eindeutigen Namen und das Rechenzentrum aus, in dem die Web-Apps-Instanz erstellt werden soll.

	![Site auf Azure erstellen][sitefromtemplateazure]

	Nachdem WebMatrix die lokale Site und die Web-Apps-Instanz in Azure erstellt hat, wird die WebMatrix IDE angezeigt.

	![WebMatrix-IDE][howtowebmatrixide]

## Einrichten von E-Mail

Das Konditoreibeispiel umfasst ein simuliertes Bestellformular, mit dem eine E-Mail-Nachricht mit der Bestellung gesendet wird. Sie verwenden den SendGrid-E-Mail-Dienst in Azure, um E-Mails von Ihrer Site zu senden.

1. Führen Sie die Schritte im Lernprogramm [Senden von E-Mails mit SendGrid in Azure][sendgridexample] aus, um ein SendGrid-Konto einzurichten und die Verbindungsinformationen abzurufen. Sie müssen nicht das gesamte Lernprogramm absolvieren– nur bis zu dem Punkt, wie die Verbindungsinformationen abgerufen werden.

2. Fügen Sie das SendGrid-NuGet-Pakets zu Ihrem WebMatrix-Projekt hinzu. Klicken Sie zunächst auf die NuGet-Schaltfläche.

    ![SendGrid hinzufügen][addsendgrid]

    Suchen und installieren Sie SendGrid.

    ![SendGrid installieren][installsendgrid]

    Nachdem das Paket installiert wurde, sehen Sie, dass die SendGrid-Assemblys zu "bin" hinzugefügt wurden.

    ![SendGrid hinzugefügt][binsendgrid]

3. Öffnen Sie die Seite *Order.cshtml*, indem Sie auf den Dateinamen doppelklicken.

	![][modify2]

4. Fügen Sie am Anfang der Datei folgenden Code hinzu:

        @using SendGrid;
        @using System.Net.Mail;

4. Suchen Sie den Kommentar "//SMTP Configuration for Hotmail", und löschen oder kommentieren Sie den gesamten Code für die Verwendung von WebMail.

        /*
        //SMTP Configuration for Hotmail
        WebMail.SmtpServer = "smtp.live.com";
        WebMail.SmtpPort = 25;
        WebMail.EnableSsl = true;

        //Enter your Hotmail credentials for UserName/Password and a "From" address for the e-mail
        WebMail.UserName = "";
        WebMail.Password = "";
        WebMail.From = "";

        if (WebMail.UserName.IsEmpty() || WebMail.Password.IsEmpty() || WebMail.From.IsEmpty()) {
            Response.Redirect("~/OrderSuccess?NoEmail=1");
        } 
        else {
            try {
                WebMail.Send(to: customerEmail, subject: "Fourth Coffee - New Order", body: body);
                Response.Redirect("~/OrderSuccess");
            } catch {
                ModelState.AddFormError("There was an error and your order could not be processed at this time");
            }
        }*/


5. Fügen Sie Code hinzu, um SendGrid anstelle von WebMail zum Senden von E-Mails zu verwenden. Fügen Sie den folgenden Code anstelle des Codes hinzu, den Sie im vorherigen Schritt gelöscht haben. Fügen Sie beim Erstellen des NetworkCredential-Objekts Ihren SendGrid-Benutzernamen und Ihr SendGrid-Kennwort hinzu.

		 if (email.IsEmpty()) {
            Response.Redirect("~/OrderSuccess?NoEmail=1");
        }
        else {
            // Create the email object first, then add the properties.
            SendGridMessage myMessage = new SendGridMessage();
            myMessage.AddTo(email);
            myMessage.From = new MailAddress("FourthCoffee@example.com", "Fourth Coffee");
            myMessage.Subject = "Fourth Coffee - New Order";
            myMessage.Text = body;

            // Create credentials, specifying your user name and password.
            var credentials = new NetworkCredential("[your user name]", "[your password]");

            // Create an Web transport for sending email.
            var transportWeb = new Web(credentials);

            // Send the email.
            try {
                transportWeb.DeliverAsync(myMessage);
                Response.Redirect("~/OrderSuccess");
            } catch {
                ModelState.AddFormError("There was an error and your order could not be processed at this time");
            }
        }


6. Klicken Sie im WebMatrix-Menüband auf **Ausführen**, um die Site zu testen.

	![][modify4]

7. Klicken Sie für eines der Produkte auf **Jetzt bestellen**, und senden Sie die Bestellung an sich selbst.

8. Überprüfen Sie Ihre E-Mail-Nachrichten darauf, ob Sie die Bestellbestätigung erhalten haben. Wenn Sie Probleme beim Senden von E-Mail-Nachrichten haben, finden Sie weitere Informationen unter [Issues with Sending Email][sendmailissues] (Probleme beim Senden von E-Mail, in englischer Sprache) in der Problembehandlung zu ASP.NET-Webseiten (Razor).
 

## Bereitstellen der benutzerdefinierten Website direkt über WebMatrix in Azure

1. Klicken Sie in WebMatrix im Menüband **Startseite** auf **Veröffentlichen**, um das Dialogfeld **Vorschau veröffentlichen** aufzurufen.

	![WebMatrix-Vorschau veröffentlichen][howtopublishpreview]

2. Klicken Sie in das Kontrollkästchen neben "bakery.sdf", und klicken Sie dann auf **Weiter**. Wenn die Veröffentlichung abgeschlossen ist, wird die URL für die aktualisierte Web-App im Azure App Service unten in der WebMatrix IDE angezeigt.

	![Veröffentlichung abgeschlossen][publishcomplete]

3. Klicken Sie auf den Link, um die Website (eine Web-Apps-Instanz in Azure) in Ihrem Browser zu öffnen:

	![Konditorei-Beispielsite][bakerysample]

	Die URL für die Web-Apps-Instanz finden Sie auch im [Azure-Portal](https://portal.azure.com), indem Sie auf **Durchsuchen** > **Web-Apps** klicken, um alle Web-Apps-Instanzen für Ihr Abonnement anzuzeigen. Wählen Sie dann eine Web-App aus. Die URL für die Web-App wird auf dem Blatt der Web-App angezeigt.

## Ändern der Website und erneute Veröffentlichung in Web-Apps

Sie können die Site mit WebMatrix ändern und dann erneut in Ihrer Web-Apps-Instanz veröffentlichen. Im folgenden Verfahren fügen Sie ein Kontrollkästchen hinzu, mit dem angegeben werden kann, ob es sich bei der Bestellung um ein Geschenk handelt.

1. Öffnen Sie die Seite *Order.cshtml*.

2. Suchen Sie die Klassendefinition "shipping". Fügen Sie den folgenden Code direkt nach dem &lt;li&gt;-Block ein.
		
		<li class="gift">
		    <div class="fieldcontainer" data-role="fieldcontain">
		        <label for="isGift">This is a gift</label>           
		        <div>@Html.CheckBox("isGift")</div>
		    </div>
		</li>

	![][modify5]

3. Suchen Sie nach der Zeile "var shipping = Request["orderShipping"];" in der Datei, und fügen Sie die folgende Codezeile direkt danach ein.

		var gift = Request["isGift"];

4. Direkt nach dem Codeblock, der überprüft, dass die Lieferadresse nicht leer ist, im folgenden Codeausschnitt.

		if(gift != null) {
			body += "This is a gift." + "<br/>";
		}

	Ihre Datei *order.cshtml* sollte in etwa der folgenden Abbildung entsprechen.

	![][modify6]

5. Speichern Sie die Datei, und führen Sie die Site lokal aus. Senden Sie dann eine Testbestellung an sich selbst. Stellen Sie sicher, dass Sie auch das neue Kontrollkästchen prüfen.

	![][modify7]

6. Veröffentlichen Sie die Site erneut, indem Sie im Menüband **Startseite** auf **Veröffentlichen** klicken.

7. Stellen Sie sicher, dass im Dialogfeld **Vorschau veröffentlichen** beide "Order.cshtml" ausgewählt sind, und klicken Sie auf "Weiter".

8. Klicken Sie auf den Link, um die Website im Browser zu öffnen und das Update auf der Web-Apps-Instanz zu testen.

## Nächste Schritte

* [WebMatrix-Website](http://www.microsoft.com/click/services/Redirect2.ashx?CR_CC=200106398)

## Änderungen
* Hinweise zu den Veränderungen von Websites zum App Service finden Sie unter: [Azure App Service und vorhandene Azure-Dienste](http://go.microsoft.com/fwlink/?LinkId=529714).
* Hinweise zu den Veränderungen des neuen Portals gegenüber dem alten finden Sie unter [Referenz zur Navigation im Azure-Portal](http://go.microsoft.com/fwlink/?LinkId=529715)




[howtowebmatrixide]: ./media/web-sites-dotnet-using-webmatrix/howtowebmatrixide2.png
[howtopublishpreview]: ./media/web-sites-dotnet-using-webmatrix/howtopublishpreview.png
[addsendgrid]: ./media/web-sites-dotnet-using-webmatrix/addsendgridpackage.png
[installsendgrid]: ./media/web-sites-dotnet-using-webmatrix/installsendgrid.png
[binsendgrid]: ./media/web-sites-dotnet-using-webmatrix/sendgridbin.png

[publishcomplete]: ./media/web-sites-dotnet-using-webmatrix/howtopublished2.png
[bakerysample]: ./media/web-sites-dotnet-using-webmatrix/howtobakerysamplesite.png
[addaccount]: ./media/web-sites-dotnet-using-webmatrix/webmatrix-add-account.png
[signin]: ./media/web-sites-dotnet-using-webmatrix/webmatrix-sign-in.png
[sitefromtemplate]: ./media/web-sites-dotnet-using-webmatrix/webmatrix-site-from-template.png
[sitefromtemplatedetails]: ./media/web-sites-dotnet-using-webmatrix/webmatrix-site-from-template-details.png
[sitefromtemplateazure]: ./media/web-sites-dotnet-using-webmatrix/webmatrix-site-from-template-azure.png

[modify1]: ./media/web-sites-dotnet-using-webmatrix/website-with-webmatrix-sample-mod-1-1.png
[modify2]: ./media/web-sites-dotnet-using-webmatrix/website-with-webmatrix-sample-mod-1-2.png
[modify3]: ./media/web-sites-dotnet-using-webmatrix/website-with-webmatrix-sample-mod-1-3.png
[modify4]: ./media/web-sites-dotnet-using-webmatrix/website-with-webmatrix-sample-mod-1-4.png
[modify5]: ./media/web-sites-dotnet-using-webmatrix/website-with-webmatrix-sample-mod-1-5.png
[modify6]: ./media/web-sites-dotnet-using-webmatrix/website-with-webmatrix-sample-mod-1-6.png
[modify7]: ./media/web-sites-dotnet-using-webmatrix/website-with-webmatrix-sample-mod-1-7.png



[sendmailissues]: http://go.microsoft.com/fwlink/?LinkId=253001#email
[sendgridexample]: http://azure.microsoft.com/documentation/articles/sendgrid-dotnet-how-to-send-email/
 

<!---HONumber=GIT-SubDir_Tue_AM_dede-->