<properties
   pageTitle="Verbinden eines Geräts mit C unter mbed | Microsoft Azure"
   description="Beschreibt, wie Sie ein Gerät mit der vorkonfigurierten Remoteüberwachungslösung von Azure IoT Suite verbinden, indem Sie eine in C geschriebene Anwendung auf einem mbed-Gerät ausführen."
   services=""
   documentationCenter="na"
   authors="dominicbetts"
   manager="timlt"
   editor=""/>

<tags
   ms.service="na"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="11/10/2015"
   ms.author="dobett"/>


# Verbinden Ihres Geräts mit der vorkonfigurierten Remoteüberwachungslösung von IoT Suite

[AZURE.INCLUDE [iot-suite-selector-connecting](../../includes/iot-suite-selector-connecting.md)]

## Erstellen und Ausführen der C-Beispielprojektmappe unter mbed

Im Folgenden werden die Schritte zum Verbinden eines [mbed-fähigen Freescale FRDM-K64F-Geräts][lnk-mbed-home] mit der Remoteüberwachungslösung beschrieben.

### Verbinden des Geräts mit Ihrem Netzwerk und Desktopcomputer

1. Verbinden Sie das mbed-Gerät mithilfe eines Ethernet-Kabels mit Ihrem Netzwerk. Dieser Schritt ist erforderlich, da die Beispielanwendung Zugriff auf das Internet benötigt.

2. Informationen zum Verbinden des mbed-Geräts mit Ihrem Desktop-PC finden Sie unter [Erste Schritte mit mbed][lnk-mbed-getstarted].

3. Wenn auf Ihrem Desktop-PC Windows ausgeführt wird, finden Sie unter [PC-Konfiguration][lnk-mbed-pcconnect] weitere Informationen zum Konfigurieren des mbed-Gerätezugriffs über den seriellen Anschluss.

### Erstellen eines mbed-Projekts und Importieren des Beispielcodes

1. Wechseln Sie in Ihrem Webbrowser zur [Entwicklerwebsite](https://developer.mbed.org/) von mbed.org. Wenn Sie sich noch nicht registriert haben, können Sie die Option zum Erstellen eines neuen Kontos verwenden (kostenlos). Melden Sie sich andernfalls mit Ihren Anmeldeinformationen an. Klicken Sie anschließend rechts oben auf der Seite auf **Compiler**. Sie gelangen auf die Benutzeroberfläche für die Arbeitsbereichsverwaltung.

2. Stellen Sie sicher, dass die von Ihnen verwendete Hardwareplattform in der oberen rechten Ecke des Fensters angezeigt wird, oder klicken Sie auf das Symbol in der rechten Ecke, um Ihre Hardwareplattform auszuwählen.

3. Klicken Sie im Hauptmenü auf **Importieren**. Klicken Sie anschließend auf **Hier klicken**, um den Import über den URL-Link neben dem mbed-Logo mit dem Globus durchzuführen.

    ![][6]

4. Geben Sie im Popupfenster den Link für den Beispielcode ein: https://developer.mbed.org/users/AzureIoTClient/code/remote_monitoring/.

    ![][7]

5. Im mbed-Compiler sehen Sie, dass der Import dieses Projekts verschiedene Bibliotheken umfasst. Einige werden vom Azure IoT-Team verwaltet ([azureiot\_common](https://developer.mbed.org/users/AzureIoTClient/code/azureiot_common/), [iothub\_client](https://developer.mbed.org/users/AzureIoTClient/code/iothub_client/), [iothub\_amqp\_transport](https://developer.mbed.org/users/AzureIoTClient/code/iothub_amqp_transport/), [iothub\_http\_transport](https://developer.mbed.org/users/AzureIoTClient/code/iothub_http_transport/), [proton-c-mbed](https://developer.mbed.org/users/AzureIoTClient/code/proton-c-mbed/)), und bei anderen handelt es sich um Bibliotheken von Drittanbietern, die im Katalog mit den mbed-Bibliotheken verfügbar sind.

    ![][8]

6. Öffnen Sie „remote\_monitoring\\remote\_monitoring.c“, und suchen Sie in der Datei nach dem folgenden Code:

    ```
    static const char* deviceId = "[Device Id]";
    static const char* deviceKey = "[Device Key]";
    static const char* hubName = "[IoTHub Name]";
    static const char* hubSuffix = "[IoTHub Suffix, i.e. azure-devices.net]";
    ```

7. Ersetzen Sie „[Device Id]“ und „[Device Key]“ durch Ihre Gerätedaten.

8. Verwenden Sie die Gerätedaten unter dem IoT Hub-Hostnamen, um IoTHub-Name und IoTHub-Suffix anzugeben. Wenn der IoT Hub-Hostname also beispielsweise „Contoso.azure-devices.net“ lautet, ist „Contoso“ der IoTHub-Name, und der restliche Text ist das Suffix.

    ```
    static const char* deviceId = "mydevice";
    static const char* deviceKey = "mykey";
    static const char* hubName = "Contoso";
    static const char* hubSuffix = "azure-devices.net";
    ```

    ![][9]

### Erstellen und Ausführen des Programms

1. Klicken Sie auf **Kompilieren**, um das Programm zu erstellen. Sie können etwaige Warnungen ignorieren, aber wenn während der Erstellung Fehler auftreten, sollten Sie diese vor dem Fortfahren beheben.

2. Wenn der Erstellungsvorgang erfolgreich ist, wird eine BIN-Datei mit dem Namen Ihres Projekts generiert. Kopieren Sie die BIN-Datei auf das Gerät. Wenn die BIN-Datei auf dem Gerät gespeichert wird, wird die aktuelle Terminalsitzung des Geräts zurückgesetzt. Setzen Sie das Terminal nach dem Herstellen der Verbindung erneut manuell zurück, oder starten Sie ein neues Terminal. Dies ermöglicht, dass das mbed-Gerät zurückgesetzt wird und das Programm ausgeführt werden kann.

3. Stellen Sie mit einer SSH-Clientanwendung eine Verbindung mit dem Gerät her, z. B. PuTTY. Sie können ermitteln, welcher serielle Port von Ihrem Gerät verwendet wird, indem Sie dies im Windows-Geräte-Manager überprüfen:


4. Klicken Sie in PuTTY auf den Verbindungstyp **Seriell**. Meistens wird die Verbindung für das Gerät über 115200 hergestellt. Geben Sie also diesen Wert in das Feld **Geschwindigkeit** ein. Klicken Sie anschließend auf **Öffnen**:

    ![][11]

5. Das Programm wird gestartet. Unter Umständen müssen Sie das Board zurücksetzen (drücken Sie STRG+UNTBR, oder drücken Sie die Reset-Taste des Boards), falls das Programm nach dem Herstellen der Verbindung nicht automatisch gestartet wird.

[AZURE.INCLUDE [iot-suite-visualize-connecting](../../includes/iot-suite-visualize-connecting.md)]


[6]: ./media/iot-suite-connecting-devices-mbed/mbed1.png
[7]: ./media/iot-suite-connecting-devices-mbed/mbed2a.png
[8]: ./media/iot-suite-connecting-devices-mbed/mbed3a.png
[9]: ./media/iot-suite-connecting-devices-mbed/suite6.png
[11]: ./media/iot-suite-connecting-devices-mbed/mbed6.png

[lnk-mbed-home]: https://developer.mbed.org/platforms/FRDM-K64F/
[lnk-mbed-getstarted]: https://developer.mbed.org/platforms/FRDM-K64F/#getting-started-with-mbed
[lnk-mbed-pcconnect]: https://developer.mbed.org/platforms/FRDM-K64F/#pc-configuration

<!---HONumber=Nov15_HO4-->