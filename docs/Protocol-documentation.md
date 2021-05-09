# SWP - Shipwreck-Protocol
Mit diesem Protokoll wird die Kommunikation in einem Schiffeversenken Spiel ermöglicht.


## Physikalische Verbindung
- Man kann sich über LAN oder WLAN verbinden.
- Es wird alles über Multicast geregelt. Es wird immer nur eine Nachricht an die Person gesendet die man in der Lobby ausgewählt hatte.
- Es soll TCP benutzt werden, da das erneute Senden von verlorenen Paketen möglich ist.

# Verbindung
## Lobby
### Statusmeldung
Ein Benutzer gibt sich selbst beim start den **Namen: Anonymous** und eine User-ID. Diese ergeben zusammen die BenutzerId. Mehr dazu in _Protokollelemente/BenutzerId_.<br>
Wenn ein Benutzer frei für ein Spiel wird oder danach gefragt wird _(Verbindung/Lobby/Request)_, schickt er dies an alle.<br>
**Syntax:** `swp|@all|[BenutzerId]|free|eof`<br>
**Beispiel:** _swp|@all|anonymous:1234|free|eof_<br>
**Anwendung:** Beim Empfang einer Statusmeldung wird diese gespeichert, solange es kein Dublikat ist.

### Anfrage
Ein Benutzer kann eine Liste an Online Benutzern beantragen indem er eine Onlineanfrage stellt.<br>
Diese wird von jedem Benutzer der frei ist beantwortet _(siehe Verbindung/Lobby/Statusmeldung)_.<br>
**Syntax:** `swp|@all|[BenutzerId]|reqfree|eof`<br>
**Beispiel:** _swp|@all|anonymous:1234|reqfree|eof_<br>
**Anwendung:** Beim Empfang einer Statusanfrage antwortet der Client mit einer Statusmeldung.

## Spiel
### Spiel anfrage
Ein Spieler kann durch eine Person angefragt werden.<br>
**Syntax:** `swp|[EmpfängerId]|[BenutzerId]|pr|eof`<br>
**Beispiel:** _swp|opponent:4321|anonymous:1234|pr|eof_<br>
**Anwendung:** Ein Benutzer schickt die Anfrage an den Gegner, welcher dann gefragt wird ob er das Spiel annimmt. Die Anfrage ist maximal **30 Sekunden** offen, danach werden auf beiden Seiten die Dialoge wieder geschlossen und eine neue Anfrage kann gestellt werden.

### Spiel bestätigen
Ein Spieler der Angefragt wurde kann ein Spiel bestätigen oder ablehnen _(siehe Verbindung/Lobby/Spiel ablehnen)_.<br>
**Syntax:** `swp|[EmpfängerId]|[BenutzerId]|pa|eof`<br>
**Beispiel:** _swp|anonymous:1234|opponent:4321|pa|eof_<br>
**Anwendung:** Der Spieler schickt die Antwort zurück an den Anfragesteller.

### Spiel ablehnen
Ein Spieler der Angefragt wurde kann ein Spiel bestätigen _(siehe Verbindung/Lobby/Spiel bestätigen)_ oder ablehnen.<br>
**Syntax:** `swp|[EmpfängerId]|[BenutzerId]|pd|eof`<br>
**Beispiel:** _swp|anonymous:1234|opponent:4321|pd|eof_<br>
**Anwendung:** Der Spieler schickt die Antwort zurück an den Anfragesteller.

### Spiel abbrechen
Wenn ein Spiel am Laufen ist, kann ein Spieler das Spiel abbrechen.<br>
**Syntax:** `swp|[EmpfängerId]|[BenutzerId]|pq|eof`<br>
**Beispiel:** _swp|anonymous:1234|opponent:4321|pq|eof_<br>
**Anwendung:** Der Spieler gibt auf oder schliesst das Programm. Vor dem Schliessen wird dann diese Nachricht geschickt.

### Spieler bereit
Wenn ein Spieler seine Schiffe verteilt hat schickt er eine Nachricht, dass er bereit ist.<br>
**Syntax:** `swp|[EmpfängerId]|[BenutzerId]|pr|eof`<br>
**Beispiel:** _swp|anonymous:1234|opponent:4321|pd|eof_<br>
**Anwendung:** Das Spiel wartet im Hintergrund bis der andere Spieler bereit ist und speichert diesen Wert lokal, bis beide Spieler bereit sind und startet dann das Spiel.

### Schuss
Ein Spieler der an der Reihe ist darf sich ein Feld auswählen, welches er angreifen möchte.<br>
**Syntax:** `swp|[EmpfängerId]|[BenutzerId]|shot|[x]|[y]|eof`<br>
**Beispiel:** _swp|anonymous:1234|opponent:4321|pd|eof_<br>
**Anwendung:** [x] und [y] sind dabei die 0-basierten Positionen des Schusses.

### Schuss antwort

- [ss]: Volltreffer, Schiff versenkt!
- [sh]: Treffer auf Schiff!
- [sw]: Ins Leere getroffen!
- [sc]: Alle Schiffe versenkt!

Ein Spieler der abgeschossen wurde, Antwortet mit einer von den Oben stehenden Antworten.<br>
**Syntax:** `swp|[EmpfängerId]|[BenutzerId]|pa|eof`<br>
**Beispiel:** _swp|anonymous:1234|opponent:4321|pd|eof_<br>
**Anwendung:** Der Spieler schickt die Antwort zurück an den Anfragesteller. Bei [sc] wird das Spiel beidseitig beended _(ähnlich Verbundung/Spiel/Spiel abbrechen)_

# Verbindungsregeln
- Die Person die die Spielanfrage stellt ist auch die, welche den ersten Schuss machen darf.
- Bei der Spiel Anfrage wird die erste Anfrage bevorzugt, alle anderen werden abgewiesen.
- Ist noch keine andere App gestartet wir keine person angezeigt welche spielen kann.

# Error Handling
## Wiederholung anfordern
Sollte eine nicht mögliche Antwort gesendet werden, wird eine Wiederholung der Antwort angefordert, bis sie Sinn macht. <br>
**Syntax:** `swp|[EmpfängerId]|[BenutzerId]|errRep|eof`<br>
**Beispiel:** _swp|anonymous:1234|opponent:4321|errRep|eof_<br>
**Anwendung:** Bei falschen Parametern wird eine Wiederholung verlangt.

## Falscher Spielgegner
Wenn ein Schuss-Befehl eintrifft, welcher von jemand anderem als dem aktuellen Gegner wird mit einem Fehler geantwortet.<br>
**Syntax:** `swp|[EmpfängerId]|[BenutzerId]|errWU|eof`<br>
**Beispiel:** _swp|anonymous:1234|opponent:4321|errWU|eof_<br>

## Weitere Fehler
<table>
<tr><th>Titel</th><th>Beschreibung</th></tr>
<tr><td>MQMCEV_PACKET_LOSS</td><td>Unbehebbarer Paketverlust </td></tr>
<tr><td>MQMCEV_VERSION_CONFLICT</td><td>Empfang von Paketen mit neueren Protokollversionen</td></tr>
<tr><td>MQMCEV_RELIABILITY</td><td>Unterschiedliche Zuverlässigkeitsmodi des Senders und des Empfängers </td></tr>
<tr><td>MQMCEV_CLOSED_TRANS</td><td>Themenübertragung wird von 1 Quelle geschlossen</td></tr>
<tr><td>MQMCEV_HEARTBEAT_TIMEOUT</td><td>Lange Abwesenheit des Heartbeat-Kontrollpakets </td></tr>
<tr><td>MQMCEV_STREAM_ERROR</td><td>Fehler im Stream erkannt</td></tr>
<tr><td>MQMCEV_NEW_SOURCE</td><td>Eine neue Quelle beginnt, über das Thema zu senden</td></tr>
<tr><td>MQMCEV_RECEIVE_QUEUE_TRIMMED</td><td>Pakete, die aufgrund des Ablaufs von Zeit oder Speicherplatz aus dem PacketQ entfernt wurden</td></tr>
</table>
<br>
Bei einem Verbindungsunterbruch soll die verloren gegane Person angepingt werden. Findet sie die Person wieder wird die Verbindung wieder aufgebaut. Ist dies jedoch nicht der Fall, kann der Pingvorgang manuel vom Spieler abgebrochen werden. Und sendet dan automatisch einen quit an die verloren gegangene Person.

[resources to MQMCEV](https://www.ibm.com/docs/en/ibm-mq/7.5?topic=programming-multicast-exception-reporting)

# Protokollelemente
## Generelle Struktur
Eine Protokollanfrage besteht grundsätzliche aus fünf Elementen.
- ShipWreck Protocol
  - swp
- Empfänger
  - BenutzerId _(Protocol/BenutzerId)_
  - @all _(Protokollelemente/Nachrichten an Alle)_
- Sender
  - BenutzerId _(Protocol/BenutzerId)_
- Nachrichteninhalt
- End of file
  - eof

Syntax: `swp|[Empfänger]|[Sender]|[Nachrichteninhalt]|eof`<br>
Beispiel: _swp|@all|anonymous:1234|free|eof_


## BenutzerId
Die BenutzerId setzt sich zusammen aus dem Benutzernamen und einer User-ID:
- BenutzerName
  - Buchstaben: [A-Za-z]
  - Zahlen: [0-9]
  - Sonderzeichen [.-_]
- User-ID
  - Vier zufällig gewählte Zahlen [0-9]
  - Bei der Menge an Spielern ist es eher unwarschinich, dass sich zwei gleiche IDs treffen. (0.1%)

Zusammengesetzt ergibt sich folgende Id:<br>
Syntax: `[BenutzerName]:[User-ID]`<br>
Beispiel: _anonymous:1234_

## Nachrichten an Alle
Als Benutzername für alle kann `@all` verwendet werden.


# Diagramm
## Normaler verlauf
```
    p1                   p2
    |    freereq       ->|
    | <- free            |
    |    playreq       ->|
    | <- playaccept      |
    | <- playerready     |
    |    playerready   ->|
    |    shot[0][0]    ->|
    | <- shotreply [sw]  |
    | <- shot[0][1]      |
    |    shotreply [ss]->|
    |    quit          ->|
    V                    V
```

## Verlauf mit Verbindungsunterbruch
```
    p1                   p2
    |    freereq       ->|
    | <- free            |
    |    playreq       ->|
    |--- 30s passing ----|
    |    playreq       ->|
    | <- playaccept      |
    | <- playerready     |
    |    playerready   ->|
    |    shot[0][-1]   ->|
    | <- repeat          |
    |    shot[0][1]    ->|
    | <- shotreply [sw]  |
    |    quit          ->|
    V                    V
```