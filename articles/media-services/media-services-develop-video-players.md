<properties 
	pageTitle="Entwickeln von Videoplayeranwendungen" 
	description="Das Thema enthält Links zu Player-Frameworks und Plug-Ins, mit denen Sie Ihre eigenen Clientanwendungen entwickeln können, die Streamingmedien aus Media Services nutzen." 
	authors="Juliako" 
	manager="dwrede" 
	editor="" 
	services="media-services" 
	documentationCenter=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/15/2015" 
	ms.author="juliako"/>


#Entwickeln von Videoplayeranwendungen

##Übersicht

Azure Media Services stellt die Tools zur Verfügung, die zum Erstellen leistungsstarker, dynamischer Clientplayeranwendungen für die gängigsten Plattformen erforderlich sind, darunter: iOS-Geräte, Android-Geräte, Windows, Windows Phone, Xbox und Set-Top-Boxen. Dieses Thema enthält auch Links zu SDKs und Player-Frameworks, mit denen Sie Ihre eigenen Clientanwendungen entwickeln können, die Streamingmedien aus Media Services verarbeiten.

##Wiedergabe Ihrer Inhalte mit vorhandenen Playern

Weitere Informationen finden Sie unter [Wiedergabe Ihrer Inhalte mit vorhandenen Playern](media-services-playback-content-with-existing-players.md).

##Azure Media Player


[Azure Media Player](aka.ms/ampdocs) ist ein Web-Videoplayer für die Wiedergabe von Medieninhalten aus Microsoft Azure Media Services auf einer Vielzahl von Browsern und Geräten. Der Azure Media Player nutzt Industriestandards wie HTML5, MSE (Media Source Extensions) und EME (Encrypted Media Extensions) für ein funktionsreiches adaptives Streaming. Wenn diese Standards auf einem Gerät oder in einem Browser nicht verfügbar sind, verwendet Azure Media Player Flash und Silverlight als Fallback-Technologien. Unabhängig von der verwendeten Wiedergabetechnologie müssen Entwickler eine einheitliche Schnittstelle für JavaScript für den Zugriff auf APIs verwenden. Dies ermöglicht die Inhaltswiedergabe von Azure Media Services über eine breite Palette von Geräten und Browsern ohne jeglichen Zusatzaufwand.

Mit Microsoft Azure Media Services können Inhalte im DASH-, Smooth Streaming- und HLS-Format wiedergegeben werden. Der Azure Media Player erkennt diese unterschiedlichen Formate und übernimmt automatisch die beste Verknüpfung basierend auf den Funktionen von Plattform und Browser. Microsoft Azure Media Services ermöglichen auch die dynamische Verschlüsselung von Medienobjekten mit PlayReady-Verschlüsselung oder AES-128-Bit-Umschlagverschlüsselung. Der Azure Media Player ermöglicht bei entsprechender Konfiguration die Entschlüsselung von PlayReady- und AES-128-Bit-verschlüsselten Inhalten. 

Weitere Informationen finden Sie unter [Azure Media Player](aka.ms/ampdocs).


##Weitere Tools zum Erstellen von Playeranwendungen

Sie können auch die folgenden SDKs verwenden:

- [Smooth Streaming Client SDK](http://www.iis.net/downloads/microsoft/smooth-streaming) 
- [Microsoft Media-Plattform: Player Framework](http://playerframework.codeplex.com/) 
- [HTML5 Player Framework-Dokumentation](http://playerframework.codeplex.com/wikipage?title=HTML5%20Player&referringTitle=Documentation) 
- [Microsoft Smooth Streaming-Plug-In für OSMF](https://www.microsoft.com/download/details.aspx?id=36057) 
- [Media Player Framework für iOS](https://github.com/Azure/azure-media-player-framework) 
- [Lizenzierung des Microsoft(r) Smooth Streaming Client Porting Kit](https://www.microsoft.com/mediaplatform/sspk.aspx) 
- [Entwickeln von Videoanwendungen für Xbox](http://xbox.create.msdn.com/) 

Weitere Informationen finden Sie unter [Entwickeln von Videoplayeranwendungen](https://msdn.microsoft.com/library/dn223283.aspx).

##Werbung

Azure Media Services unterstützt Werbeeinblendungen über die Windows Media-Plattform: Player-Frameworks. Player-Frameworks mit Unterstützung für Werbeeinblendungen sind für Windows 8, Silverlight, Windows Phone 8 und iOS-Geräte verfügbar. Jedes Player-Framework enthält Beispielcode, der zeigt, wie eine Playeranwendung implementiert werden kann. Es gibt drei verschiedene Arten von Werbung, die Sie in Ihre Medien einfügen können:

Linear - Werbeeinblendungen auf dem kompletten Frame, die das Hauptvideo unterbrechen

Nicht linear - Overlay-Werbeeinblendungen, die während der Wiedergabe angezeigt werden; normalerweise ein Logo oder ein anderes statisches Bild im Player

Begleitend - Werbeeinblendungen, die außerhalb des Players angezeigt werden

Werbeeinblendungen können an jeder Stelle auf der Zeitachse des Hauptvideos platziert werden. Sie müssen den Player anweisen, welche Werbeeinblendungen wann wiedergegeben werden. Dies erfolgt mithilfe einer Reihe von Standarddateien im XML-Format: Video Ad Service Template (VAST), Digital Video Multiple Ad Playlist (VMAP), Media Abstract Sequencing Template (MAST) und Digital Video Player Ad Interface Definition (VPAID). VAST-Dateien geben die anzuzeigenden Werbeeinblendungen an. VMAP-Dateien geben an, wann die verschiedenen Werbeeinblendungen wiedergegeben werden, sie enthalten VAST-XML. MAST-Dateien sind eine weitere Möglichkeit zum Anordnen von Werbeeinblendungen, die auch VAST-XML enthalten können. VPAID-Dateien definieren eine Schnittstelle zwischen Video-Player und Werbeeinblendung oder Werbungsserver. Weitere Informationen finden Sie unter [Einfügen von Werbeeinblendungen](https://msdn.microsoft.com/library/dn387398.aspx).

Informationen zu Untertiteln und der Unterstützung von Werbeeinblendungen in Livestreaming-Videos finden Sie unter [Standards für unterstützte Untertitel und für Werbeeinblendungen](https://msdn.microsoft.com/library/c49e0b4d-357e-4cca-95e5-2288924d1ff3#caption_ad).


<!--HONumber=52--> 