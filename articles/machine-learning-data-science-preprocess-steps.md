﻿<properties 
	pageTitle="Vorverarbeiten und Bereinigen von Daten | Azure" 
	description="Vorverarbeiten und Bereinigen von Daten" 
	metaKeywords="data cleansing" 
	services="machine-learning" 
	documentationCenter="" 
	authors="xibingaomsft,msolhab" 
	manager="paulettm" 
	editor="cgronlun" />

<tags 
	ms.service="machine-learning" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="02/16/2015" 
	ms.author="xibingaomsft,msolhab" /> 


Vorverarbeiten und Bereinigen von Daten
========================================
Unformatierte Daten enthalten oft unnötige bzw. fehlende Werte und sind unzuverlässig. Die Verwendung dieser Daten für die Modellierung kann zu falschen Ergebnissen führen. Deshalb ist die Vorverarbeitung und Bereinigung von Daten ein wichtiger Schritt, der vor dem Verwenden eines DataSets für das maschinelle Lernen durchgeführt werden muss. In diesem Artikel werden verschiedene Schritte zur Datenvorverarbeitung beschrieben, die vor der Erfassung von Daten in Azure Machine Learning durchgeführt werden können. 

----------------

1. **Warum müssen Daten vorverarbeitet und bereinigt werden?**

	Echte Daten stammen aus verschiedenen Quellen und Abläufen und können daher Unregelmäßigkeiten oder beschädigte Daten enthalten, die die Qualität des DataSets beeinträchtigen. Folgende Qualitätsprobleme treten bei Daten häufiger auf:
	- Unvollständige Daten: Den Daten fehlen Attribute oder Werte.
	- Überflüssige Daten: Die Daten enthalten fehlerhafte Datensätze oder Ausreißer.
	- Inkonsistente Daten: Die Daten enthalten widersprüchliche Datensätze oder Abweichungen. 
		
	Qualitätsdaten sind eine wichtige Voraussetzung für die Qualität von Vorhersagemodellen. Um schlechte Ergebnisse aufgrund schlechter Ausgangsdaten zu vermeiden und die Datenqualität und damit das Modell zu verbessern, ist es unerlässlich, eine Überprüfung der Datenintegrität durchzuführen, um Probleme mit den Daten frühzeitig zu erkennen und geeignete Schritte zur Vorverarbeitung und Bereinigung durchführen zu können.

2. **Welche Methoden zur Überprüfung der Datenintegrität werden am häufigsten eingesetzt?** 
	
	Sie können die allgemeine Qualität der Daten überprüfen, indem Sie Folgendes prüfen:
    - Die Anzahl der Datensätze.
    - Die Anzahl der Attribute (oder Funktionen).
	- Die Attribut-Datentypen (nominal, ordinal oder fortlaufend).
	- Die Anzahl der fehlenden Werte.
	- Die richtige Formatierung der Daten. Wenn die Daten in TSV- oder CSV-Dateien gespeichert sind, sollten Sie prüfen, ob die Spalten- und Zeilentrennzeichen auch immer die Spalten und Zeilen ordnungsgemäß trennen. Bei Daten im HTML- oder XML-Format überprüfen Sie, ob die Daten gemäß den jeweiligen Standards wohlgeformt sind. Es ist möglicherweise eine Analyse erforderlich, um strukturierte Informationen aus teilweise strukturierten oder unstrukturierten Daten zu extrahieren.
	- Inkonsistente Datensätze. Wenn die Daten z. B. die Notendurchschnitte von Schülern enthalten, sollte der Durchschnitt im festgelegten Bereich liegen, also z. B. zwischen 1 und 4. 

	Wenn Sie Probleme mit den Daten finden, sind Verarbeitungsschritte erforderlich. Dazu gehören häufig die Bereinigung fehlender Werte, die Datennormalisierung, die Diskretisierung, die Textverarbeitung zum Entfernen und/oder Ersetzen eingebetteter Zeichen, die Einfluss auf die Datenausrichtung haben können, das Bereinigen gemischter Datentypen in gemeinsamen Feldern u. a. 
	
	In Azure Machine Learning werden wohlgeformte Tabellendaten verarbeitet.  Wenn die Daten bereits in tabellarischer Form vorliegen, kann die Datenvorverarbeitung direkt in Azure Machine Learning in ML Studio ausgeführt werden.  Wenn Daten nicht in tabellarischer Form vorliegen, sondern z. B. als XML, ist eine Analyse erforderlich, um die Daten in eine tabellarische Form zu konvertieren.  

3. **Welches sind die wichtigsten Vorgänge bei der Datenvorverarbeitung?**

	- Datenbereinigung:  Auffüllen fehlender Werte, Erkennen und Entfernen überflüssiger Daten und Ausreißer
	- Datentransformation:  Normalisierung der Daten, um Dimensionen und Störungen zu verringern
	- Datenreduzierung:  Erzeugen von Stichproben aus den Datensätzen oder Attributen zur einfacheren Datenverarbeitung
	- Datendiskretisierung:  Konvertieren kontinuierlicher Attribute in kategorische Attribute zur einfacheren Verwendung in bestimmten Methoden zum maschinellen Lernen
	- Textbereinigung: Entfernen eingebetteter Zeichen, die zu einer falschen Datenausrichtung führen können, z. B. eingebetteter Tabstopps in tabstoppgetrennten Dateien oder eingebetteter Zeilenumbrüche, die Datensätze unterbrechen könnten, usw.	
	
	In den folgenden Abschnitten werden einige der Schritte zur Datenvorverarbeitung beschrieben.

4. **Wie werden fehlende Daten behandelt?**
	
	Bei fehlenden Werten empfiehlt es sich, zunächst den Grund für die fehlenden Werte zu ermitteln, um das Problem besser angehen zu können. Folgende Vorgehensweisen werden bei fehlenden Werten häufig angewendet:

	- Löschen: Löschen Sie die Datensätze mit den fehlenden Werten.
	- Ersetzung durch Platzhalter: Ersetzen Sie die fehlenden Werte durch einen Platzhalterwert: z. B. _unbekannt_ bei kategorischen oder _0_ bei numerischen Werten.
	- Ersetzung durch Durchschnittswerte: Bei fehlenden numerischen Daten können Sie die fehlenden Werte durch Mittelwerte ersetzen. 
	- Ersetzung durch häufige Werte: Bei fehlenden kategorischen Daten können Sie die fehlenden Werte durch den häufigsten Eintrag ersetzen. 
	- Ersetzung durch Regression: Verwenden Sie ein Regressionsverfahren, um die fehlenden Werte durch Regressionswerte zu ersetzen.  

5. **Wie werden Daten normalisiert?**
	
	Bei der Datennormalisierung werden numerische Werte in einen angegebenen Bereich skaliert. Folgende Normalisierungsverfahren werden häufig angewendet:
	- Min-Max-Normalisierung: Transformieren Sie die Daten linear in einen Bereich, z. B. zwischen 0 und 1.  Der Mindestwert wird auf 0 skaliert und der Höchstwert auf 1.
	- Z-Wert-Normalisierung: Skalieren Sie die Daten basierend auf Mittelwert und Standardabweichung: Teilen Sie die Differenz aus Daten und Mittelwert durch die Standardabweichung.
	- Dezimalskalierung: Skalieren Sie die Daten durch Verschieben des Dezimaltrennzeichens des Attributwerts.  

6. **Wie werden Daten diskretisiert?**  

	Daten können durch die Konvertierung kontinuierlicher Werte in nominale Attribute oder Intervalle diskretisiert werden. Dazu gibt es u. a. folgende Möglichkeiten:
	- Festbreiten-Klassifizierung: Teilen Sie den Bereich aller möglichen Werte eines Attributs in N Gruppen derselben Größe auf, und weisen Sie den Werten in einer Klasse die Klassennummer zu.
	- Festhöhen-Klassifizierung: Teilen Sie den Bereich aller möglichen Werte eines Attributs in N Gruppen mit derselben Anzahl von Instanzen auf, und weisen Sie den Werten in einer Klasse die Klassennummer zu.  

7. **Wie werden Daten reduziert?**  

	Es gibt verschiedene Methoden zum Reduzieren der Größe zur einfacheren Datenverarbeitung. Je nach Größe und Inhalt der Daten können folgende Verfahren angewendet werden: 
	- Erstellen von Datenstichproben: Erstellen Sie Stichproben aus den Datensätzen, und wählen Sie nur eine repräsentative Teilmenge von Daten aus. 
	- Erstellen von Attributstichproben: Wählen Sie nur wichtige Attribute aus den Daten aus.  
	- Aggregation: Unterteilen Sie die Daten in Gruppen, und speichern Sie die Zahlen der einzelnen Gruppen. Beispielsweise können die Tageseinnahmen einer Restaurant-Kette aus den letzten 20 Jahren im monatlichen Umsatz zusammengefasst werden, um die Größe der Daten zu verringern.  

8. **Wie werden Textdaten bereinigt?**  

	Textfelder in Tabellendaten können Zeichen enthalten, die Einfluss auf die Spaltenausrichtung und/oder die Grenzen des Datensatzes haben. Eingebettete Tabstopps in einer tabstoppgetrennten Datei verursachen z. B. Fehlausrichtungen von Spalten, während eingebettete Zeilenumbrüche Datensatzzeilen beschädigen. Eine fehlerhafte Verarbeitung von Textcodierungen beim Schreiben oder Lesen von Text kann zu Datenverlusten und unbeabsichtigten Einfügungen von unlesbaren Zeichen, z. B. NULL-Werten, führen und möglicherweise auch Auswirkungen auf die Textanalyse haben. Es ist möglicherweise eine sorgfältige Analyse und Bearbeitung erforderlich, um Textfelder für die korrekte Ausrichtung zu bereinigen und um strukturierte Daten aus unstrukturierten oder teilweise strukturierten Textdaten zu extrahieren.

Das Durchsuchen von Daten ermöglicht auch einen frühzeitigen Einblick in die Daten. Während dieses Schritts können bereits einige Probleme aufgedeckt und  entsprechende Methoden angewendet werden, um diese Probleme zu beheben.  Es ist wichtig, Fragen wie die zur Ursache des Problems und dessen Ursprung zu stellen. Dadurch können Sie auch bessere Entscheidungen für die Verarbeitungsschritte treffen, die zum Lösen der Probleme erforderlich sind. Die Einblicke, die aus den Daten gewonnen werden sollen, können auch zum Festlegen von Prioritäten für die Datenverarbeitung verwendet werden.

**Referenz:**
	
>Jiawei Han, Micheline Kamber und Jian Pei: _Data Mining: Concepts and Techniques_, 3. Auflage, Morgan Kaufmann, 2011

<!--HONumber=49-->