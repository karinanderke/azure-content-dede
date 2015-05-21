<properties
   pageTitle="Installieren von regelmäßigen Hotfixes"
   description="Beschreibt die Installation von regelmäßigen Hotfixes über Windows PowerShell für StorSimple."
   services="storsimple"
   documentationCenter="NA"
   authors="SharS"
   manager="adinah"
   editor="tysonn" />
<tags 
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="04/23/2015"
   ms.author="v-sharos" />
###So installieren Sie regelmäßige Hotfixes über Windows PowerShell für StorSimple

1. Stellen Sie eine Verbindung mit der seriellen Gerätekonsole her. Weitere Informationen finden Sie unter [So stellen Sie eine Verbindung über die serielle Konsole her](#to-connect-through-the-serial-console).

2. Wählen Sie im Menü der seriellen Konsole Option 1 aus, d. h. die **Anmeldung mit Vollzugriff**. Geben Sie das Kennwort ein. Das Standardkennwort lautet **Password1**.

3. Geben Sie an der Eingabeaufforderung Folgendes ein:

     **Start-HcsHotfix**

   >[AZURE.IMPORTANT]
   >
   > - Dieser Befehl gilt nur für regelmäßige Hotfixes. Dieser Befehl muss nur auf einem Controller ausgeführt werden, es werden jedoch beide Controller aktualisiert. 
   > - Möglicherweise werden Sie während des Updatevorgangs ein Failover des Controllers bemerken. Dieses Failover wirkt sich jedoch nicht auf die Systemverfügbarkeit oder den Betrieb aus.

4. Geben Sie bei entsprechender Aufforderung den Pfad zur Netzwerkfreigabe mit den Hotfix-Dateien ein.

5. Sie werden aufgefordert, diesen Schritt zu bestätigen. Geben Sie **J** ein, um mit der Installation des Hotfixes fortzufahren.

<!--HONumber=52-->