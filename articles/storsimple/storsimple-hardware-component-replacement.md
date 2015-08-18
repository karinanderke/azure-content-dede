<properties 
   pageTitle="Austauschen von StorSimple-Hardwarekomponenten | Microsoft Azure"
   description="Beschreibt, wie PCMs, Batterien, Controllermodule, EBOD-Controller, Laufwerke und Gehäuse eines StorSimple-Gerät sicher ausgetauscht werden."
   services="storsimple"
   documentationCenter=""
   authors="alkohli"
   manager="carolz"
   editor="" />

<tags 
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="08/05/2015"
   ms.author="alkohli" />


# Austauschen von StorSimple-Hardwarekomponenten

## Übersicht

In den Tutorials zum Austauschen von Hardwarekomponenten werden die Hardwarekomponenten Ihres Microsoft Azure StorSimple-Gerät-Speichergeräts sowie die erforderlichen Schritte zum Ausbauen und Austauschen dieser Komponenten erläutert. In diesen Tutorials erfahren Sie, welche Komponenten austauschbar sind und wie Sie diese austauschen.

Machen Sie sich, bevor Sie versuchen, StorSimple-Komponenten auszubauen oder auszutauschen, mit dem [Konventionen für Sicherheitssymbole](#safety-icon-conventions) und anderen [Sicherheitsvorkehrungen](storsimple-safety.md) vertraut.
 
### Konventionen für Sicherheitssymbole

In der folgenden Tabelle werden die in dieser Anleitung verwendeten Sicherheitssymbole beschrieben. Achten Sie bei den Schritten zum Ausbauen und Austauschen von Gerätekomponenten auf diese Sicherheitssymbole.

| Symbol | Text | Zusätzliche Informationen |
|:---- |:---- |:-----------|
|![Symbol "Warnung"](./media/storsimple-hardware-component-replacement/Warning.png)| **GEFAHR!** | Zeigt eine Gefahrensituation an, die zum Tod oder schweren Verletzungen führen wird, wenn sie nicht vermieden wird. Dieses Signalwort wird nur für die extremsten Situationen verwendet.|
|![Symbol "Warnung"](./media/storsimple-hardware-component-replacement/Warning.png)| **WARNUNG!** | Zeigt eine Gefahrensituation an, die zum Tod oder schweren Verletzungen führen könnte, wenn sie nicht vermieden wird.|
|![Symbol "Vorsicht"](./media/storsimple-hardware-component-replacement/Caution.png)| **VORSICHT!** |Zeigt eine Gefahrensituation an, die zu leichten bis mittelschweren Verletzungen führen könnte, wenn sie nicht vermieden wird.|
|![Symbol "Hinweis"](./media/storsimple-hardware-component-replacement/NoticeIcon.png)| **HINWEIS:** | Zeigt als wichtig erachtete Informationen an, die allerdings nicht mit Gefahren in Verbindung stehen.|
|![Symbol "Stromschlag"](./media/storsimple-hardware-component-replacement/Electric.png) | **Gefahr eines Stromschlags** | Gibt Hochspannung an.|
|![Symbol für hohes Gewicht](./media/storsimple-hardware-component-replacement/Weight.png)| **Hohes Gewicht**| |
|![Symbol "Keine durch Benutzer zu wartenden Teile"](./media/storsimple-hardware-component-replacement/NoUserServiceableParts.png)| **Keine vom Benutzer zu wartenden Teile** | Zugriff nur bei ausreichender Schulung.|
|![Symbol "Anweisungen lesen"](./media/storsimple-hardware-component-replacement/ReadInstructions.png)|**Alle Anleitungen zuvor lesen**| |
|![Symbol "Kippgefahr"](./media/storsimple-hardware-component-replacement/TipHazard.png)|**Kippgefahr**| |

### Voraussetzungen

Machen Sie sich mit den sicherheitsrelevanten Informationen zu Ihrem Gerät und den in diesem Tutorial verwendeten Sicherheitssymbolen vertraut. Unter [Sicherheit Ihres Geräts](storsimple-safety.md) finden Sie ausführliche Informationen. Überprüfen Sie die [Sicherheitsvorkehrungen](storsimple-safety.md#handling-precautions), bevor Sie mit Ihrem StorSimple-Gerät hantieren.

Bevor Sie versuchen, eine Komponente auszutauschen, beachten Sie die folgenden Informationen.

![Symbol "Warnung"](./media/storsimple-hardware-component-replacement/Warning.png)![Symbol "Stromschlag"](./media/storsimple-hardware-component-replacement/Electric.png)**WARNUNG!**

- Erden Sie sich ordnungsgemäß mithilfe elektrostatischer Entladung oder einer Antistatikmatte, bevor Sie mit Modulen und Komponenten Ihres StorSimple-Gerät-Geräts hantieren.

- Berühren Sie keine elektrischen Schaltungen. Verwenden Sie die bereitgestellten Griffe und Führungen, wenn Sie mit Komponenten hantieren, die ggf. elektrische Schaltungen haben.

![Symbol "Warnung"](./media/storsimple-hardware-component-replacement/Warning.png)![Symbol "Hinweis"](./media/storsimple-hardware-component-replacement/NoticeIcon.png)**HINWEIS:**

Wenn Sie ein Modul austauschen, lassen Sie **NIEMALS ein Fach auf der Rückseite des Gehäuses leer**. Besorgen Sie ein Austausch- oder leeres Modul, bevor Sie das fehlerhafte Teil ausbauen.

## Hardwarekomponenten Ihres Geräts

Ihr Microsoft Azure StorSimple-Gerät besteht aus mehreren Steckmodulen im primären und/oder EBOD-Gehäuse. Das Modell 8100 hat ein einziges primäres Gehäuse, während das Modell 8600 ein Gerät mit zwei Gehäusen (einem primären Gehäuse und einem EBOD-Gehäuse) ist.

Die wesentlichen Hardwarekomponenten des Geräts sind in der folgenden Tabelle zusammengefasst. Klicken Sie in der Spalte "Austauschverfahren" auf den Link, um zum dazugehörigen Tutorial zu gelangen.

|Komponenten|Anzahl|Steckmodul|Austauschverfahren
|:---------|:--------|:--------------|:---------------------|
| Gehäuse|1|Nein|[Austauschen des Gehäuses des StorSimple-Geräts](storsimple-chassis-replacement.md) |
|Primäre Controller|2|Ja| [Austauschen eines Controllermoduls eines StorSimple-Geräts](storsimple-controller-replacement.md) |
|Module für Stromversorgung und Kühlung (PCMs) mit 764 W|2|Ja| [Austauschen eines Moduls für Stromversorgung und Kühlung eines StorSimple-Geräts](storsimple-power-cooling-module-replacement.md) |
|Pufferbatterie|2|Ja| [Austauschen des Pufferbatteriemoduls eines StorSimple-Geräts](storsimple-battery-replacement.md) |
|Festplattenlaufwerke|12|Ja| [Austauschen eines Festplattenlaufwerks im StorSimple-Gerät](storsimple-disk-drive-replacement.md) |

**Tabelle 1** Hardwarekomponenten im primären Gehäuse

Das primäre Gehäuse und das EBOD-Gehäuse unterscheiden sich bei den E/A-Modulen. Außerdem haben die PCMs unterschiedliche Wattzahlen. Die PCMs im primären Gehäuse haben 764 W, wogegen die PCMs im EBOD-Gehäuse 580 W haben. Zusätzlich enthalten die PCMs im primären Gehäuse ein Pufferbatteriemodul.

|Komponenten|Anzahl|Steckmodul| Austauschverfahren
|:---------|:--------|:--------------|:---------------------|
|Gehäuse|1|Nein| [Austauschen des Gehäuses des StorSimple-Geräts](storsimple-chassis-replacement.md) |
|EBOD-Controller|2|Ja| [Austauschen des EBOD-Controllers eines StorSimple-Geräts](storsimple-EBOD-controller-replacement.md) |
|Module für Stromversorgung und Kühlung (PCMs) mit 580 W|2|Ja| [Austauschen eines Moduls für Stromversorgung und Kühlung eines StorSimple-Geräts](storsimple-power-cooling-module-replacement.md) |
|Festplattenlaufwerke|12|Ja| [Austauschen eines Festplattenlaufwerks im StorSimple-Gerät](storsimple-disk-drive-replacement.md) |

**Tabelle 2** Hardwarekomponenten im EBOD-Gehäuse

Die Steckmodule im Gerät sind in den folgenden Abbildungen der Vorder- und Rückansicht markiert. Anhand dieser Abbildungen können Sie die Einbaupositionen der verschiedenen Steckmodule ermitteln, wenn ein Austausch erforderlich ist. In der Vorderansicht sind die Laufwerke zu sehen. In der Rückansicht des EBOD-Gehäuses und des primären Gehäuses sind die Steckmodule zu sehen.

![Vorderseite des Geräts mit Festplattenlaufwerken](./media/storsimple-hardware-component-replacement/IC741028.png)

**Abbildung 1** Vorderseite des Geräts

|Bezeichnung|Beschreibung|
|:----|:----------|
|0-11|Laufwerke (insgesamt 12)|

Sowohl das primäre Gehäuse als auch das EBOD-Gehäuse haben Laufwerksträgermodule. Das Gehäuse enthält zwölf 3,5-Zoll-Laufwerke, die in einem 3x4-Format angeordnet sind.

![Rückwand des Geräts – Module für primäre Gehäuse](./media/storsimple-hardware-component-replacement/IC740994.png)

**Abbildung 2** Rückseite des primären Gehäuses

|Bezeichnung|Beschreibung|
|:----|:----------|
|1|PCM 0|
|2|PCM 1|
|3|Controller 0|
|4|Controller 1|

![Rückwand des Geräts – Steckmodule für EBOD-Gehäuse](./media/storsimple-hardware-component-replacement/IC769599.png)

**Abbildung 3** Rückseite des EBOD-Gehäuses

|Bezeichnung|Beschreibung|
|:----|:----------|
|1|PCM 0|
|2|PCM 1|
|3|EBOD-Controller 0|
|4|EBOD-Controller 1|

## FRUs (Field Replaceable Units)

Die folgenden FRUs (Field Replaceable Units) sind für Ihr StorSimple-Gerät verfügbar:

- Gehäuse (einschließlich des integrierten Bedienfelds)

- Stromversorgungs- und Kühleinheit (Power and Cooling Module, PCM), 764 W

- Stromversorgungs- und Kühleinheit (Power and Cooling Module, PCM), 580 W

- Festplattenlaufwerk mit Laufwerkträgermodul

- Controllermodul

- EBOD-Controllermodul

- Pufferbatteriemodul

- Schienensatz für Gestellmontage

[Wenden Sie sich an den Microsoft-Support](storsimple-contact-microsoft-support.md), um diese Ersatzteile zu bestellen.

## Nächste Schritte

Überprüfen Sie alle [sicherheitsrelevanten Informationen](storsimple-safety.md), bevor Sie versuchen, eine StorSimple-Hardwarekomponente auszutauschen.

<!---HONumber=August15_HO6-->