<properties
   pageTitle="Bereitstellen, Initialisieren und Formatieren von Volumes"
   description="Erläutert das Konfigurieren eines Volumes auf einem StorSimple-Gerät."
   services="storsimple"
   documentationCenter="NA"
   authors="SharS" manager="adinah"
   editor="tysonn" />
<tags
   ms.se**rvice="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="04/01/2015"
   ms.author="v-sharos" />

###So stellen Sie ein Volume bereit und initialisieren und formatieren dieses

1. Starten Sie den Microsoft iSCSI-Initiator.

2. Klicken Sie im Fenster **Eigenschaften des iSCSI-Initiators** auf der Registerkarte **Erkennung** auf **Portal ermitteln**.

3. Geben Sie im Dialogfeld **Zielportal ermitteln** die IP-Adresse Ihrer iSCSI-aktivierten Netzwerkschnittstelle an, und klicken Sie dann auf **OK**.

4. Suchen Sie auf der Registerkarte **Ziele** des Fensters **Eigenschaften des iSCSI-Initiators** nach **Ermittelte Ziele**. Als Gerätestatus sollte **Inaktiv** angezeigt werden.

5. Wählen Sie das Zielgerät aus, und klicken Sie dann auf **Verbinden**. Nachdem das Gerät verbunden ist, sollte sich der Status in **Verbunden** ändern. \(Weitere Informationen zum Verwenden des Microsoft iSCSI-Initiators finden Sie unter [Installieren und Konfigurieren des Microsoft iSCSI-Initiators][1]\).

6. Drücken Sie auf Ihrem Windows-Host die Windows-Logo-Taste+X, und klicken Sie dann auf **Ausführen**.

7. Geben Sie im Dialogfeld **Ausführen** die Angabe **Diskmgmt.msc** ein. Klicken Sie auf **OK**. Das Dialogfeld **Datenträgerverwaltung** wird angezeigt. Im rechten Bereich werden die Volumes auf Ihrem Host angezeigt.

8. Im Fenster **Datenträgerverwaltung** werden die bereitgestellten Volumes wie in der folgenden Abbildung dargestellt angezeigt. Klicken Sie mit der rechten Maustaste auf das ermittelte Volume \(klicken Sie auf den Datenträgernamen\), und klicken Sie dann auf **Online**.

     ![Initialisieren und Formatieren von Volumes](./media/storsimple-mount-initialize-format-volume/HCS_InitializeFormatVolume-include.png)

9. Klicken Sie erneut mit der rechten Maustaste auf das Volume \(klicken Sie auf den Datenträgernamen\), und klicken Sie dann auf **Initialisieren**.

10. Führen Sie die folgenden Schritte aus, um ein einfaches Volume zu formatieren.
  1. Wählen Sie das Volume aus, klicken Sie mit der rechten Maustaste darauf \(klicken Sie im rechten Bereich\), und klicken Sie dann auf **Neues einfaches Volume**.
  2. Geben Sie im Assistenten für neue einfache Volumes die Volumegröße und den Laufwerkbuchstaben an, und konfigurieren Sie das Volume dann als Dateisystemtyp "NTFS".
  3. Geben Sie 64 KB als Zuordnungseinheitsgröße an. Diese Zuordnungseinheitsgröße eignet sich gut für die Deduplikationsalgorithmen, die in der StorSimple-Lösung verwendet werden.
  4. Führen Sie eine Schnellformatierung durch.

<!--Link references-->
[1]: https://technet.microsoft.com/library/ee338480(WS.10).aspx

<!--HONumber=52-->