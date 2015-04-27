﻿<properties 
	pageTitle="Was sind Logik-Apps?" 
	description="Weitere Informationen zu App Service-Logik-Apps" 
	authors="joshtwist" 
	manager="dwrede" 
	editor="" 
	services="app-service\logic" 
	documentationCenter=""/>

<tags
	ms.service="app-service-logic"
	ms.workload="integration"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="02/24/2015"
	ms.author="jtwist"/>

#Was sind Logik-Apps?

Azure App Service ist ein vollständiges PaaS-Angebot (Platform as a Service) für professionelle Entwickler, das einen umfangreichen Satz von Funktionen für Web-, mobile und Integrationsszenarien bereitstellt. Logik-Apps sind ein Teil der App Service-Produktfamilie und ermöglichen technisch orientierten Benutzern und Entwicklern das Automatisieren der Ausführung von Geschäftsprozessen mithilfe eines leicht zu bedienenden visuellen Designers.

Außerdem können Logik-Apps mit API-Apps und Connectors aus unserem Marketplace kombiniert werden, um selbst schwierige Integrationsszenarien einfach zu meistern.

![Flow app designer](./media/app-service-learn-about-flows-preview/Designer.png)

##Die SaaS-, PaaS- und Hybrid-Explosion

Das Zeitalter der Cloud bringt eine Flut von SaaS- und PaaS-Systemen mit sich, die Entwickler überall zunehmend unter Druck setzen. In der Zwischenzeit werden die Arbeitsrückstände bei der Anwendungsentwicklung immer größer, wobei diese heterogenen und verteilten Lösungen neue Integrationsherausforderungen darstellen.  Darüber hinaus müssen Unternehmen ihre lokalen Daten und Dienste in ihren Lösungen nutzen - und zwar auf sichere Weise.

Die Entwicklung von Lösungen, die sich über diese Systeme erstrecken, bringt keine einfach zu bewältigenden Herausforderungen mit sich, die für Entwicklungsteams sehr zeitaufwändig und fehleranfällig sind.

##Gründe für Logik-Apps

Logik-Apps ermöglichen Entwicklern das Entwerfen von Workflows, in denen eine Absicht mithilfe eines Triggers und einer Reihen von Schritten ausgedrückt wird. Bei jedem Schritt wird eine App Service-API-App aufgerufen, und zugleich wird für eine sichere Authentifizierung und beständige Ausführung gesorgt.

- **Leicht zu bedienende Entwurfstools** - Logik kann von A bis Z im Browser entwickelt werden. Beginnen Sie mit einem Trigger - von einem einfachen Zeitplan bis zum Auftauchen eines Tweets zu Ihrem Unternehmen. Orchestrieren Sie anschließend eine beliebige Anzahl von Aktionen mithilfe des umfassenden Angebots an Connectors.

- **Einfaches Zusammensetzen von SaaS** - Zusammensetzungsaufgaben, die einfach zu beschreiben sind, lassen sich nur schwierig in Code implementieren. Mit Logik-Apps ist es kinderleicht, verschiedenartige Systeme miteinander zu verbinden. Möchten Sie in Ihrer CRM-Software eine Aufgabe erstellen, die auf Aktivität in Ihren Facebook- oder Twitter-Konten basiert? Wollen Sie Ihre Marketinglösung in der Cloud mit Ihrem lokalen Abrechnungssystem verbinden? Logik-Apps sind die schnellste und zuverlässigste Möglichkeit, Lösungen für diese Probleme bereitzustellen.

- **Schneller Einstieg mit Vorlagen** - Um Ihnen den Einstieg zu erleichtern, bieten wir eine Galerie mit Vorlagen, mit denen Hilfe Sie einige allgemeine Lösungen rasch erstellen können. Von BizTalk-Lösungen bis zu einfachen SaaS-Konnektivitätslösungen und sogar einigen, die " just  'for fun'" sind - die Galerie ist die schnellste Möglichkeit, sich mit der Leistungsfähigkeit von Logik-Apps vertraut zu machen.

- **Integrierte Erweiterbarkeit** - Sie finden nicht den Connector, den Sie brauchen? Logik-Apps sind Teil der App Service-Produktfamilie und für das Arbeiten mit API-Apps ausgelegt. Sie können problemlos eine eigene API-App entwickeln und anschließend als Connector nutzen. Entwickeln Sie eine neue App nur für sich allein, oder veröffentlichen Sie sie im Marketplace, um sie kommerziell zu verwerten.

- **Hohes Integrationspotenzial** - Fangen Sie einfach an, und wachsen Sie Ihrem Bedarf entsprechend. Logik-Apps können problemlos die Leistungsstärke von BizTalk, der branchenführenden Integrationslösung von Microsoft nutzen, damit Integrationsexperten die Lösungen erstellen können, die sie brauchen. Erfahren Sie mehr über die von [App Services gebotenen BizTalk-Funktionen][biztalk].

## Logik-App - Konzepte

Es folgt eine Beschreibung einiger der wichtigsten Elemente von Logik-Apps. 

- **Workflow** - Logik-Apps bieten eine grafische Möglichkeit zum Abbilden Ihrer Geschäftsprozesse als Folge von Schritten bzw. Workflow.
- **Connectors** - Logik-Apps benötigen Zugriff auf Daten und Dienste. Ein Connector ist eine besondere Art von API-App. Sein Zweck ist, Ihnen beim Herstellen von Verbindungen und Arbeiten mit Ihren Daten zu helfen. Unter [Was sind Connectors?][biztalk] finden Sie weitere Informationen.
- **Trigger** - Einige Connectors können auch als Trigger fungieren. Ein Trigger startet eine neue Instanz eines Workflows basierend auf einem bestimmten Ereignis, z. B. Eingang einer E-Mail oder einer Änderung in Ihrem Azure Storage-Konto.
- **BizTalk** - Für anspruchsvollere Integrationsszenarien bietet Azure App Services Funktionen von Biztalk. BizTalk ist die branchenführenden Integrationsplattform von Microsoft. Die BizTalk-API-Apps ermöglichen die problemlose Einbeziehung von u. a. Validierung, Transformation und Regeln in Ihre Logik-App-Workflows. Unter [Was sind BizTalk-API-Apps?][biztalk] finden Sie weitere Informationen.

## Erste Schritte

Befolgen Sie zum Einstieg in Logik-Apps das Lernprogramm [Erstellen einer Logik-App ][create].

Weitere Informationen zur Azure App Service-Plattform finden Sie unter [Azure App Service][appservice].

[biztalk]: app-service-logic-what-are-biztalk-api-apps.md
[appservice]: app-service-value-prop-what-is.md
[create]: app-service-logic-create-a-logic-app.md




<!--HONumber=49-->