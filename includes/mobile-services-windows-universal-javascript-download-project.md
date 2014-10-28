Dieses Lernprogramm baut auf der [GetStartedWithMobileServices-App][GetStartedWithMobileServices-App] auf. Dabei handelt es sich um ein universelles Windows-App-Projekt in Visual Studio 2013. Die Benutzeroberfläche dieser App und die vom Mobile Services-Schnellstart generierte Benutzeroberfläche sind identisch, mit der Ausnahme, dass hinzugefügte Einträge lokal im Arbeitsspeicher gespeichert werden.

1.  Laden Sie die JavaScript-Version der GetStartedWithMobileServices-Beispiel-App von der [Website mit den Codebeispielen für Entwickler] herunter.

2.  Öffnen Sie das heruntergeladene Projekt in Visual Studio 2013, erweitern Sie den Ordner "js", und sehen Sie sich die Datei "default.js" an.

    Beachten Sie, dass die hinzugefügten **TodoItem**-Objekte in einem speicherinternen `WinJS.Binding.List`-Objekt gespeichert werden.

3.  Drücken Sie die **F5**-Taste, um das Projekt neu zu erstellen und die App zu starten.

4.  Geben Sie in der App in **Insert a TodoItem** Text ein, und klicken Sie dann auf **Save**.

    ![][]

    Beachten Sie, dass der gespeicherte Text angezeigt wird.

5.  Klicken Sie mit der rechten Maustaste auf das Windows Phone 8.1-Projekt, klicken Sie auf **Als Startprojekt festlegen**, und drücken Sie dann **F5**, um die Windows Phone Store-App auszuführen.

    ![][1]

6.  Wiederholen Sie die Schritte 3 und 4, um sicherzustellen, dass sich die Beispiel-App gleichermaßen verhält.

  [GetStartedWithMobileServices-App]: http://go.microsoft.com/fwlink/p/?LinkID=510826
  []: ./media/mobile-services-windows-universal-dotnet-download-project/mobile-quickstart-startup.png
  [1]: ./media/mobile-services-windows-universal-dotnet-download-project/mobile-quickstart-startup-wp8.png