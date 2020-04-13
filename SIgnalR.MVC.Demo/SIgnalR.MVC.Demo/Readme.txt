------------------------------------------------------------------------------------------
SignalR MVC Demo																12.04.2020
------------------------------------------------------------------------------------------

Erstellen des Projektes
1) Erstellen eines Projektes in GitHub

2) Erstellen eines Projektes in Visual Studio
	- Projekt-Typ ASP.NET Core WebApplication
	- Dateiordern des Projektes (Location)	= Ordner des erstellten GitHub-Projektes
	- Template: MVC-Model-View-Controler

3) Hinzufügen der Clientseitigen Bibliothek für SignalR
	- Auswahl des Ordners WWWWRoot
	- mittels rechter Maustaste die Option Add / Client-Side Library auswählen
	- Provider: unpkg
	- Library: @microsoft/signalr@latest
	- Nur folgende Dateien wählen im Ordner dist/browser auswählen:
		signalr.js
		signalr.min.js
	- Target Location: wwwroot/js/signalr

4) Anlegen eines Ordners mit dem Namen Hubs, in welchem anschließend
   die sogenannten Hub-Klassen angelegt werden.
   Die erstellen Klassen müssen daher von der Klasse Hub abgeleitet werden.
   Dadurch ist es erforderlich den Namespace Microsoft.AspNetCore.SignalR zu importieren.

5) Konfiguration von SignalR in der Startup-Klasse.
	- in der Funktion ConfigureServices muss zunächst der SignalR-Serice hinzugefügt werden.
		services.AddSignalR();
	- Für jede SignalR-Hub-Klasse muss ein Endpoint (beliebiger Pfad) in der Funktion Configure eingerichtet werden.
	  Die bisherigen Angaben zu den Endpoints können bestehen bleiben, da sie für MVC verwendet werden
		app.UseEndpoints(endpoints =>
			endpoints.MapHub<NameOfHubClass>("/PathToHub");
			...
		);

6) Integration der Clientseitigen Logik
	- Erstellen eines DIV-Containers, der eine unsortierte Liste aufnimmt, 
	  welche für den Zugriff mit einer ID versehen wird.
	  <div>
		<ul id="myID"></ul>
	  </div>
	- Integration der zwei nachfolgenden JavaScripte auf der Seite
		~/lib/jquery/dist/jquery.js
		~/js/signalr/dist/browser/signalr.js
	- Clientseitige Definition der SignalR-Verbindung
		var connection = new signalR.HubConnectionBuilder()
									.withUrl("/PathToHub")
									.build();
	- Event für diese Verbindung definieren
		connection.on("NameOfClientEvent", function(message){
				$("#myID").append("<li>" + message + "<li>");
		});

	- Verbindung zur Hub-Klasse starten und evtl. auftretende Fehler abfangen
		connection.start().catch(function (err){
			return console.error(err.toString());
		});

7) Broadcast einer Message an alle Clients
7.1) Ausgelöst durch ein serverseitiges Event
		Integratrion der SignalR-Funktionalität mittels Dependency-Injection
		- Dem Konstruktor des relevanten Controllers wird eine Variabel vom Typ IHubContext<MyHubClass> übergeben.
		- Der Konstruktor speichert dann diese Context-Variabel in einer eigenen privaten Variabel.
		Verwendung des HubContextes in der serverseitigen (Event-) Funktion ermöglicht das Versenden einer Nachricht
		an alle verbundenen Clients. Die Event-Funktion muss hierbei allers als async gekennzeichnet sein.

		     await _HubContext
                .Clients
                .All
                .SendAsync("NameOfClientEvent", messageText);

7.2.) Ausgelöst durch ein clientseitiges Event
		Hierzu ist zunächst die eigene Hub-Klasse um eine Funktion zu erweitern,
		die mittels des Objekts Clients eine Message an die gewünschten Empfänger verschickt.
		Dies ist möglich, da die eigene Hub-Klasse von SignarR-Klasse Hub erbt und somit
		ebenfalls Zugriff hat auf die Funktionen, die oben genannten Hub-Context (siehe 7.1)
		zur Verfügung stehen.
		Beispiel:
		public async Task BroadcastFromClient(string messageText)
		{
			await Clients.All.SendAsync("NameOfClientEvent", messageText);
		}