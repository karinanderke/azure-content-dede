<properties
	pageTitle="Tutorial: Azure Active Directory-Integration mit TargetProcess | Microsoft Azure"
	description="Erfahren Sie, wie Sie das einmalige Anmelden zwischen Azure Active Directory und TargetProcess konfigurieren."
	services="active-directory"
	documentationCenter=""
	authors="jeevansd"
	manager="prasannas"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="12/01/2015"
	ms.author="jeedes"/>


# Tutorial: Azure Active Directory-Integration mit TargetProcess

In diesem Tutorial erfahren Sie, wie Sie TargetProcess in Azure Active Directory (Azure AD) integrieren können.<br>Die Integration von TargetProcess in Azure AD bietet Ihnen folgende Vorteile:

- Sie können in Azure AD steuern, wer Zugriff auf TargetProcess hat. 
- Sie können es Benutzern ermöglichen, sich mit ihren Azure AD-Konten automatisch bei TargetProcess anzumelden (einmaliges Anmelden).
- Sie können Ihre Konten an einem zentralen Ort verwalten – dem Azure Active Directory-Portal.

Weitere Informationen zur Integration von SaaS-Apps in Azure AD finden Sie unter [Was bedeuten Anwendungszugriff und einmaliges Anmelden mit Azure Active Directory?](active-directory-appssoaccess-whatis.md).

## Voraussetzungen 

Um die Azure AD-Integration mit TargetProcess konfigurieren zu können, benötigen Sie Folgendes:

- Ein Azure AD-Abonnement
- Ein TargetProcess-Abonnement, für das einmaliges Anmelden aktiviert ist


> [AZURE.NOTE]Um die Schritte in diesem Tutorial zu testen, wird empfohlen, keine Produktionsumgebung zu verwenden.


Um die Schritte in diesem Tutorial zu testen, sollten Sie folgende Empfehlungen beachten:

- Sie sollten keine Produktionsumgebung verwenden, sofern dies nicht erforderlich ist.
- Wenn Sie keine Azure AD-Testumgebung haben, können Sie [hier](https://azure.microsoft.com/pricing/free-trial/) eine einmonatige Testversion anfordern. 

 
## Beschreibung des Szenarios
Ziel dieses Tutorials ist es, das einmalige Anmelden von Azure AD in einer Testumgebung zu testen. <br> Das in diesem Tutorial beschriebene Szenario besteht aus zwei großen Bausteinen:

1. Hinzufügen von TargetProcess aus dem Katalog 
2. Konfigurieren und Testen der einmaligen Anmeldung von Azure AD


## Hinzufügen von TargetProcess aus dem Katalog
Zum Konfigurieren der Integration von TargetProcess in Azure AD müssen Sie TargetProcess aus dem Katalog der Liste der verwalteten SaaS-Apps hinzufügen.

**Um TargetProcess aus dem Katalog hinzuzufügen, führen Sie die folgenden Schritte aus:**

1. Klicken Sie im linken Navigationsbereich des **Azure-Verwaltungsportals** auf **Active Directory**. <br><br> ![Active Directory][1]<br>

2. Wählen Sie in der Liste **Verzeichnis** das Verzeichnis aus, für das Sie die Verzeichnisintegration aktivieren möchten.

3. Klicken Sie zum Öffnen der Anwendungsansicht in der oberen Menüleiste der Verzeichnisansicht auf **Anwendungen**.<br><br> ![Anwendungen][2]<br>
4. Klicken Sie unten auf der Seite auf **Hinzufügen**.<br><br> ![Anwendungen][3]<br>
5. Klicken Sie im Dialogfeld **What do you want to do** auf **Anwendung aus dem Katalog hinzufügen**.<br><br> ![Anwendungen][4]<br>
6. Geben Sie im Suchfeld als Suchbegriff **TargetProcess** ein.<br><br> ![Erstellen eines Azure AD-Testbenutzers](./media/active-directory-saas-target-process-tutorial/tutorial_target_process_01.png)<br>
7. Wählen Sie im Ergebnisbereich **TargetProcess** aus, und klicken Sie dann auf **Abschließen**, um die Anwendung hinzuzufügen. <br><br>![Erstellen eines Azure AD-Testbenutzers](./media/active-directory-saas-target-process-tutorial/tutorial_target_process_10.png)<br>

##  Konfigurieren und Testen der einmaligen Anmeldung von Azure AD
In diesem Abschnitt soll anhand eines Testbenutzers namens Britta Simon veranschaulicht werden, wie das einmalige Anmelden von Azure AD in TargetProcess konfiguriert und getestet werden kann.

Damit einmaliges Anmelden funktioniert, muss Azure AD wissen, welcher Benutzer in TargetProcess als Gegenbenutzer zu einem Benutzer in Azure AD fungiert. Anders ausgedrückt: Zwischen einem Azure AD-Benutzer und dem entsprechenden Benutzer in TargetProcess muss eine Linkbeziehung eingerichtet werden.<br> Diese Linkbeziehung wird hergestellt, indem Sie den **Benutzernamen** in Azure AD als Wert für den **Benutzernamen** in TargetProcess zuweisen.
 
Zum Konfigurieren und Testen des einmaligen Anmeldens in Azure AD bei TargetProcess müssen Sie die folgenden Bausteine ausführen:

1. **[Konfigurieren von Azure AD – einmaliges Anmelden](#configuring-azure-ad-single-single-sign-on)**, um Ihren Benutzern das Verwenden dieser Funktion zu ermöglichen.
2. **[Erstellen eines Azure AD-Testbenutzers](#creating-an-azure-ad-test-user)**, um das einmalige Anmelden mit Azure AD mit dem Testbenutzer Britta Simon zu testen.
4. **[Erstellen eines TargetProcess-Testbenutzers](#creating-a-targetprocess-test-user)**, um eine Entsprechung von Britta Simon in TargetProcess zu erhalten, die mit ihrer Darstellung in Azure AD verknüpft ist.
5. **[Zuweisen des Azure AD-Testbenutzers](#assigning-the-azure-ad-test-user)**, um Britta Simon für das einmalige Anmelden von Azure AD zu aktivieren.
5. **[Testen der einmaligen Anmeldung](#testing-single-sign-on)**, um zu überprüfen, ob die Konfiguration funktioniert.

### Konfigurieren des einmaligen Anmeldens von Azure AD

Das Ziel dieses Abschnitts besteht darin, das einmalige Anmelden von Azure AD im Azure AD-Portal zu aktivieren und das einmalige Anmelden in Ihrer TargetProcess-Anwendung zu konfigurieren. Im Rahmen dieses Verfahrens müssen Sie eine Base64-codierte Zertifikatsdatei erstellen. Falls Sie nicht mit diesem Verfahren vertraut sind, finden Sie unter [How to convert a binary certificate into a text file](http://youtu.be/PlgrzUZ-Y1o) (in englischer Sprache) weitere Informationen.

Zum Konfigurieren des einmaligen Anmeldens für TargetProcess benötigen Sie eine registrierte Domäne. Wenn Sie noch keine registrierte Domäne besitzen, wenden Sie sich unter [support@flatterfiles.com](mailto:support@flatterfiles.com) an das Supportteam von TargetProcess.



**Führen Sie zum Konfigurieren des einmaligen Anmeldens von Azure AD in TargetProcess die folgenden Schritte aus:**

1. Klicken Sie im Azure AD-Portal auf der Anwendungsintegrationsseite für **TargetProcess** auf **Einmaliges Anmelden konfigurieren**, um das Dialogfeld **Einmaliges Anmelden konfigurieren** zu öffnen. <br><br> ![Einmaliges Anmelden konfigurieren][6] <br>

2. Wählen Sie auf der Seite **Wie sollen sich Benutzer bei TargetProcess anmelden?** die Option **Azure AD – einmaliges Anmelden** aus, und klicken Sie dann auf **Weiter**. <br><br> ![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-target-process-tutorial/tutorial_target_process_02.png) <br>

3. Führen Sie auf der Dialogseite **App-Einstellungen konfigurieren** die folgenden Schritte aus: <br><br>![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-target-process-tutorial/tutorial_target_process_03.png) <br>


    a. Geben Sie im Textfeld **Anmelde-URL** die URL ein, die die Benutzer zur Anmeldung bei der TargetProcess-Anwendung verwenden (z. B. **https://fabrikam.TargetProcess.com/*).

    b. Klicken Sie auf **Weiter**.
 
 
4. Führen Sie auf der Seite **Einmaliges Anmelden konfigurieren für TargetProcess** die folgenden Schritte aus: <br><br>![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-target-process-tutorial/tutorial_target_process_04.png) <br>

    a. Klicken Sie auf **Zertifikat herunterladen**, und speichern Sie die Datei auf Ihrem Computer.

    b. Klicken Sie auf **Weiter**.


1. Melden Sie sich bei Ihrer TargetProcess-Anwendung als Administrator an.


1. Klicken Sie im oberen Menü auf **Setup**. <br><br>![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-target-process-tutorial/tutorial_target_process_05.png) <br>

1. Klicken Sie auf **Einstellungen**. <br><br>![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-target-process-tutorial/tutorial_target_process_06.png) <br>

1. Klicken Sie auf **Einmaliges Anmelden**. <br><br>![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-target-process-tutorial/tutorial_target_process_07.png) <br>

1. Führen Sie im Dialogfeld „Einstellungen für einmaliges Anmelden“ die folgenden Schritte aus: <br><br>![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-target-process-tutorial/tutorial_target_process_08.png) <br>

    a. Klicken Sie auf **Einmaliges Anmelden aktivieren**.

    b. Kopieren Sie im Azure-Portal auf der Dialogfeldseite **Einmaliges Anmelden konfigurieren für TargetProcess** den Wert für **Dienst-URL für einmaliges Anmelden**, und fügen Sie ihn ins Textfeld **Anmelde-URL** ein.

    c. Öffnen Sie das heruntergeladene Zertifikat im Editor, kopieren Sie den Inhalt, und fügen Sie ihn anschließend in das Textfeld **Zertifikat** ein.

    d. Klicken Sie auf **JIT-Bereitstellung aktivieren**.


6. Wählen Sie im Azure AD-Portal die Bestätigung zur Konfiguration der einmaligen Anmeldung aus, und klicken Sie dann auf **Weiter**. <br><br>![Azure AD – einmaliges Anmelden][10]<br>

7. Klicken Sie auf der Seite **Bestätigung zur einmaligen Anmeldung** auf **Fertig stellen**. <br><br>![Azure AD – einmaliges Anmelden][11]




### Erstellen einen Azure AD-Testbenutzers
Das Ziel dieses Abschnitts ist das Erstellen eines Testbenutzers namens Britta Simon im Azure-Portal.<br> Wählen Sie in der Benutzerliste **Britta Simon** aus.<br><br>![Azure AD-Benutzer erstellen][20]<br>

**Um einen Testbenutzer in Azure AD zu erstellen, führen Sie die folgenden Schritte aus:**

1. Klicken Sie im linken Navigationsbereich des **Azure-Verwaltungsportals** auf **Active Directory**. <br><br>![Erstellen eines Azure AD-Testbenutzers](./media/active-directory-saas-flatter-files-tutorial/create_aaduser_09.png) <br> 

2. Wählen Sie in der Liste **Verzeichnis** das Verzeichnis aus, für das Sie die Verzeichnisintegration aktivieren möchten.

3. Klicken Sie zum Anzeigen der Liste der Benutzer im Menü oben auf **Benutzer**. <br><br> ![Erstellen eines Azure AD-Testbenutzers](./media/active-directory-saas-flatter-files-tutorial/create_aaduser_03.png) <br>
 
4. Um das Dialogfeld **Benutzer hinzufügen** zu öffnen, klicken Sie auf der Symbolleiste unten auf **Benutzer hinzufügen**. <br><br> ![Erstellen eines Azure AD-Testbenutzers](./media/active-directory-saas-flatter-files-tutorial/create_aaduser_04.png) <br>

5. Führen Sie auf der Dialogfeldseite **Informationen über diesen Benutzer** die folgenden Schritte aus: <br><br> ![Erstellen eines Azure AD-Testbenutzers](./media/active-directory-saas-flatter-files-tutorial/create_aaduser_05.png) <br>

    a. Wählen Sie als "Benutzertyp" die Option "Neuer Benutzer in Ihrer Organisation" aus.

    b. Geben Sie in das Textfeld **Benutzername** den Text **BrittaSimon** ein.

    c. Klicken Sie auf **Weiter**.

6.  Führen Sie auf der Dialogfeldseite **Benutzerprofil** die folgenden Schritte aus: <br><br>![Erstellen eines Azure AD-Testbenutzers](./media/active-directory-saas-flatter-files-tutorial/create_aaduser_06.png) <br>
 
    a. Geben Sie in das Textfeld **Vorname** den Namen **Britta** ein.

    b. Geben Sie in das Textfeld **Nachname** den Namen **Simon** ein.

    c. Geben Sie in das Textfeld **Anzeigename** den Namen **Britta Simon** ein.

    d. Wählen Sie in der Liste **Rolle** die Rolle **Benutzer** aus. e. Klicken Sie auf **Weiter**.

7. Klicken Sie auf der Dialogfeldseite **Vorübergehendes Kennwort abrufen** auf **Erstellen**. <br><br> ![Erstellen eines Azure AD-Testbenutzers](./media/active-directory-saas-flatter-files-tutorial/create_aaduser_07.png) <br>
 
8. Führen Sie auf der Dialogfeldseite **Vorübergehendes Kennwort abrufen** die folgenden Schritte aus: <br><br>![Erstellen einen Azure AD-Testbenutzers](./media/active-directory-saas-flatter-files-tutorial/create_aaduser_08.png) <br>
  
    a. Notieren Sie den Wert von **Neues Kennwort**.

    b. Klicken Sie auf **Fertig stellen**.

  
 
### Erstellen eines TargetProcess-Testbenutzers

Das Ziel dieses Abschnitts ist das Erstellen eines Benutzers namens Britta Simon in TargetProcess. TargetProcess unterstützt die Just-in-Time-Bereitstellung. Sie haben sie bereits unter [Konfigurieren der einmaligen Anmeldung in Azure AD](#configuring-azure-ad-single-single-sign-on) aktiviert.

Für Sie steht in diesem Abschnitt kein Aktionselement zur Verfügung.




### Zuweisen des Azure AD-Testbenutzers

Das Ziel dieses Abschnitts besteht darin, Britta Simon die Verwendung des einmaligen Anmeldens bei Azure zu ermöglichen, indem sie Zugriff auf TargetProcess erhält. <br><br>![Benutzer zuweisen][200] <br>

**Um Britta Simon TargetProcess zuzuweisen, führen Sie die folgenden Schritte aus:**

1. Klicken Sie zum Öffnen der Anwendungsansicht im Azure-Portal in der oberen Menüleiste der Verzeichnisansicht auf **Anwendungen**. <br><br>![Benutzer zuweisen][201] <br>

2. Wählen Sie in der Anwendungsliste **TargetProcess** aus. <br><br>![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-target-process-tutorial/tutorial_target_process_09.png) <br>

1. Klicken Sie im oberen Menü auf **Benutzer**. <br><br>![Benutzer zuweisen][203] <br>

1. Wählen Sie in der Benutzerliste **Britta Simon** aus.

2. Klicken Sie auf der Symbolleiste unten auf **Zuweisen**. <br><br>![Benutzer zuweisen][205]



### Testen der einmaligen Anmeldung

Das Ziel dieses Abschnitts ist das Testen Ihrer Azure AD-Konfiguration für einmaliges Anmelden mithilfe des Zugriffsbereichs.<br> Wenn Sie im Zugriffsbereich auf die Kachel „TargetProcess“ klicken, sollten Sie automatisch bei Ihrer TargetProcess-Anwendung angemeldet werden.


## Zusätzliche Ressourcen

* [Liste der Tutorials zur Integration von SaaS-Apps in Azure Active Directory](active-directory-saas-tutorial-list.md)
* [Was bedeuten Anwendungszugriff und einmaliges Anmelden mit Azure Active Directory?](active-directory-appssoaccess-whatis.md)


<!--Image references-->

[1]: ./media/active-directory-saas-target-process-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-target-process-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-target-process-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-target-process-tutorial/tutorial_general_04.png

[6]: ./media/active-directory-saas-target-process-tutorial/tutorial_general_05.png
[10]: ./media/active-directory-saas-target-process-tutorial/tutorial_general_06.png
[11]: ./media/active-directory-saas-target-process-tutorial/tutorial_general_07.png
[20]: ./media/active-directory-saas-target-process-tutorial/tutorial_general_100.png

[200]: ./media/active-directory-saas-target-process-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-target-process-tutorial/tutorial_general_201.png
[203]: ./media/active-directory-saas-target-process-tutorial/tutorial_general_203.png
[204]: ./media/active-directory-saas-target-process-tutorial/tutorial_general_204.png
[205]: ./media/active-directory-saas-target-process-tutorial/tutorial_general_205.png

<!---HONumber=AcomDC_1203_2015-->