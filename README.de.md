# Mosque Translator — Streaming-Einrichtungsanleitung

> 🌐 Diese Anleitung ist auch verfügbar auf [English](README.md) und [العربية (Arabisch)](README.ar.md).

Diese Lösung ermöglicht das sogenannte "Intranet-Radio". Sie ermöglicht das lokale Streamen von Audio ohne Internetverbindung. Das Streaming erfolgt über ein internes Netzwerk.

Alles, was Sie benötigen, kann aus diesem Repository heruntergeladen werden. Hier sind die Download-Links zur Sicherheit:

1. https://icecast.org/download/
2. https://danielnoethen.de/butt/

Diese Anleitung führt Sie Schritt für Schritt durch die Einrichtung von **Icecast** (dem Streaming-Server) und **BUTT** (dem Tool, das Ihr Mikrofon/Audio an Icecast sendet). Keine Vorkenntnisse nötig — folgen Sie einfach der Reihenfolge. Sie benötigen Administratorrechte, um diese Lösung einzurichten.

---

## Übersicht

- **Icecast** ist ein Server, der auf Ihrem Computer läuft und Audio an Hörer im Netzwerk überträgt.
- **BUTT** (Broadcast Using This Tool) ist eine kleine Anwendung, die Audio (z. B. von einem Mikrofon) aufnimmt und an Ihren Icecast-Server streamt.

Sie richten zuerst Icecast ein und verbinden dann BUTT damit.

---

## Teil 1 — Icecast installieren und konfigurieren

### 1. Den Icecast-Ordner finden

Dieses Repository enthält bereits einen `Icecast`-Ordner mit allem, was Sie brauchen:

```
Icecast/
├── icecast.bat        ← Doppelklick, um den Server zu starten
├── icecast.xml         ← die Konfigurationsdatei (Passwörter, Ports usw.)
└── bin/icecast.exe     ← das eigentliche Icecast-Programm
```

Falls Sie diesen Ordner nicht haben, führen Sie zuerst das Installationsprogramm `icecast_win64_2.5.0 (2).exe` im `Icecast`-Ordner aus — es richtet alles für Sie ein.

### 2. Passwort in `icecast.xml` festlegen

Bevor Sie den Server starten, müssen Sie die Standardpasswörter ändern — das ist wichtig für die Sicherheit, da jeder, der das Standardpasswort (`hackme`) kennt, sich mit Ihrem Server verbinden könnte.

1. Öffnen Sie den `Icecast`-Ordner.
2. Rechtsklick auf **`icecast.xml`** → **Öffnen mit** → **Editor** (oder einen beliebigen Texteditor).
3. Suchen Sie den Abschnitt `<authentication>`. Er sieht so aus:

   ```xml
   <authentication>
       <source-password>hackme</source-password>
       <relay-password>hackme</relay-password>
       <admin-user>admin</admin-user>
       <admin-password>hackme</admin-password>
   </authentication>
   ```

4. Ersetzen Sie jedes `hackme` durch ein eigenes, sicheres Passwort:
   - **`source-password`** — das Passwort, mit dem sich BUTT verbindet und Audio streamt. Sie benötigen es in Teil 2, also merken Sie es sich.
   - **`relay-password`** — wird nur benötigt, wenn Sie von einem anderen Icecast-Server relayen. Kann unverändert bleiben oder ebenfalls geändert werden.
   - **`admin-password`** — das Passwort für das Web-Admin-Panel (`http://localhost:8000/admin/`).

   Beispiel:

   ```xml
   <authentication>
       <source-password>MySecurePass123</source-password>
       <relay-password>MySecurePass123</relay-password>
       <admin-user>admin</admin-user>
       <admin-password>MyAdminPass456</admin-password>
   </authentication>
   ```

5. Prüfen Sie die Werte von `<hostname>` und `<port>8000</port>` oben in der Datei. Standardmäßig sind sie auf `localhost` bzw. `8000` gesetzt. Lassen Sie diese unverändert.

6. **Speichern** Sie die Datei und schließen Sie sie.

### 3. Icecast starten

1. Gehen Sie zurück zum `Icecast`-Ordner.
2. **Doppelklicken Sie auf `icecast.bat`**.
3. Ein schwarzes Konsolenfenster öffnet sich und zeigt den Startvorgang von Icecast. Lassen Sie dieses Fenster geöffnet — wenn Sie es schließen, stoppt der Server.
4. Sie sollten eine Meldung wie diese sehen:

   ```
   Please open http://localhost:8000/ in your web browser to see the web interface.
   ```

   Möglicherweise sehen Sie auch einige Fehlermeldungen. Das ist unproblematisch, solange die obige Meldung erscheint, die bestätigt, dass der Server läuft.

5. Öffnen Sie Ihren Browser und gehen Sie zu **http://localhost:8000/**. Wenn Sie die Icecast-Statusseite sehen, läuft der Server korrekt.

> 💡 **Wichtig:** Minimieren Sie dieses Fenster nur — schließen Sie es NICHT, während gesendet wird.

### 4. Icecast durch die Firewall zulassen & Mikrofon-Echo beheben

**A. Icecast durch die Windows-Firewall zulassen**

Standardmäßig kann die Windows-Firewall verhindern, dass andere Geräte sich mit Icecast verbinden, auch wenn es auf demselben PC (`localhost`) funktioniert. Sie müssen eine Firewall-Regel hinzufügen, um dies zu erlauben.

1. Klicken Sie auf das **Start**-Menü und suchen Sie nach **"Windows Defender Firewall"** → öffnen Sie es.
2. Klicken Sie links auf **Eine App oder ein Feature durch die Windows Defender Firewall zulassen**.
3. Klicken Sie oben rechts auf **Einstellungen ändern** (erfordert Administratorrechte).
4. Klicken Sie unten auf **Andere App zulassen...**.
5. Klicken Sie auf **Durchsuchen...** und navigieren Sie zum Icecast-Ordner, dann wählen Sie:
   ```
   Icecast\bin\icecast.exe
   ```
6. Klicken Sie auf **Hinzufügen**.
7. Suchen Sie **icecast.exe** in der Liste und stellen Sie sicher, dass **sowohl** die Kästchen **Privat** als auch **Öffentlich** aktiviert sind (Privat reicht, wenn Sie sich nur in einem Heim-/Moschee-WLAN befinden).
8. Klicken Sie auf **OK**, um zu speichern.

**Falls es weiterhin blockiert wird**, fügen Sie zusätzlich eine spezifische eingehende Portregel hinzu:

1. Suchen Sie nach **"Windows Defender Firewall mit erweiterter Sicherheit"** → öffnen Sie es.
2. Klicken Sie links auf **Eingehende Regeln** → rechts auf **Neue Regel...**.
3. Wählen Sie **Port** → **Weiter**.
4. Wählen Sie **TCP**, dann **Bestimmte lokale Ports** und geben Sie `8000` ein (oder den Port, den Sie in `icecast.xml` festgelegt haben) → **Weiter**.
5. Wählen Sie **Verbindung zulassen** → **Weiter**.
6. Lassen Sie Domäne/Privat/Öffentlich alle aktiviert (oder nur Privat für ein Heim-/lokales Netzwerk) → **Weiter**.
7. Geben Sie einen Namen ein, z. B. `Icecast Port 8000` → **Fertigstellen**.

Jetzt sollten andere Geräte im selben Netzwerk `http://<ihre-pc-ip>:8000/<mountpoint>` erreichen können.

Fahren Sie erst mit Teil 2 fort, wenn Sie sicher sind, dass der Icecast-Status unter `http://localhost:8000/` angezeigt wird.

**B. "Dieses Gerät abhören" beim Mikrofon deaktivieren**

Wenn Sie während des Streamens Ihre eigene Stimme als Echo über die Lautsprecher hören, leitet Windows möglicherweise das Mikrofonsignal direkt an die Lautsprecher/Kopfhörer weiter ("Abhören"). So schalten Sie das aus:

1. Rechtsklick auf das **Lautsprechersymbol** in der Windows-Taskleiste (unten rechts) → **Sounds** (oder suchen Sie **"Soundeinstellungen"** im Startmenü).
2. Gehen Sie zum Tab **Aufnahme**.
3. Rechtsklick auf Ihr Mikrofon (das in BUTT ausgewählte) → **Eigenschaften**.
4. Gehen Sie zum Tab **Abhören**.
5. **Deaktivieren** Sie das Kästchen **"Dieses Gerät abhören"**.
6. Klicken Sie auf **Übernehmen**, dann auf **OK**.

Dadurch spielt Windows Ihr Mikrofonsignal nicht mehr lokal über die Lautsprecher ab — BUTT nimmt es weiterhin normal auf und streamt es, aber Sie hören kein Echo mehr auf Ihrem PC.

---

## Teil 2 — BUTT installieren und konfigurieren

### 1. BUTT installieren

1. Suchen Sie die Datei **`butt-1.47.0_Win64_Portable.zip`** in diesem Projekt.
2. Rechtsklick darauf → **Alle extrahieren...** und wählen Sie einen Ordner (z. B. `Desktop\butt`).
3. Öffnen Sie den entpackten Ordner und doppelklicken Sie auf **`butt.exe`**, um es zu starten.

   Dies ist eine portable Version, es gibt also nichts weiter zu installieren — sie läuft direkt.

### 2. BUTT mit Ihrem Icecast-Server verbinden

1. Klicken Sie in BUTT auf die Schaltfläche **Settings** (Zahnrad-/Schraubenschlüssel-Symbol).
2. Gehen Sie zum Tab **Main** und klicken Sie auf **Add** (um einen neuen Server hinzuzufügen).
3. Geben Sie die Serverdetails passend zu Ihrer `icecast.xml` ein:

   | Feld | Wert |
   |---|---|
   | **Type** | Icecast |
   | **Address** | `localhost` (oder die IP Ihres Computers, wenn Sie an andere Geräte im Netzwerk streamen) |
   | **Port** | `8000` |
   | **Password** | das `source-password`, das Sie in `icecast.xml` festgelegt haben (ursprünglich `hackme`) |
   | **Mountpoint** | z. B. `/stream` (merken Sie sich dies — Hörer verwenden es, z. B. `http://localhost:8000/stream`) |
   | **Icecast/Shoutcast user** | `source` |

4. Klicken Sie auf **OK**, um den Servereintrag zu speichern.

### 3. Audioeingang konfigurieren

1. Gehen Sie weiterhin in den **Settings** zum Tab **Audio**.
2. Wählen Sie unter **Device** Ihr Mikrofon oder Ihre Audioquelle aus (z. B. "Microphone (Realtek Audio)" oder ein virtuelles Audiokabel, wenn Sie Systemaudio aufnehmen).
3. Wählen Sie **Codec** `MP3` und **Bitrate** `64K` für beide Zeilen — diese Einstellung hat sich bei uns am besten bewährt.
4. Klicken Sie auf **OK**, um die Einstellungen zu schließen.

### 4. Streaming starten

1. Stellen Sie im Hauptfenster von BUTT sicher, dass Ihr Server im Dropdown ausgewählt ist.
2. Klicken Sie auf die **PLAY**-Schaltfläche (▶), um mit dem Streaming zu beginnen.
3. Bei korrekter Konfiguration verbindet sich BUTT und zeigt eine grüne "connected"-Anzeige sowie Live-Audiopegel mit einem laufenden Zeitzähler.

### 5. Den Stream überprüfen

1. Gehen Sie zurück zu **http://localhost:8000/** in Ihrem Browser.
2. Unter "Mount Points" sollten Sie nun Ihren aktiven Mountpoint (z. B. `/stream`) als live sehen.
3. Klicken Sie darauf, oder öffnen Sie `http://localhost:8000/<mountpoint>` direkt (z. B. `http://localhost:8000/stream`), um den Stream anzuhören.

### 6. Verbindung von anderen Geräten im Netzwerk

`localhost` funktioniert nur auf demselben PC, auf dem Icecast läuft. Damit andere Geräte (Smartphones, Laptops, Tablets) zuhören können, müssen Sie die lokale Netzwerkadresse (IPv4) Ihres PCs herausfinden und diese stattdessen weitergeben.

1. Öffnen Sie auf dem PC, auf dem Icecast läuft, die **Eingabeaufforderung** (suchen Sie "cmd" im Startmenü).
2. Geben Sie Folgendes ein und drücken Sie **Enter**:

   ```
   ipconfig
   ```

3. Suchen Sie den Abschnitt Ihrer aktiven Verbindung (meist **Wireless LAN adapter Wi-Fi** oder **Ethernet adapter Ethernet**).
4. Suchen Sie die Zeile **IPv4 Address**. Sie sieht etwa so aus:

   ```
   IPv4 Address. . . . . . . . . . . : 192.168.1.25
   ```

   Dies ist die Adresse Ihres PCs im lokalen Netzwerk. Notieren Sie sie sich.

5. Öffnen Sie auf jedem anderen Gerät im **gleichen WLAN/Netzwerk** einen Webbrowser und gehen Sie zu:

   ```
   http://<IPv4-Adresse>:8000/<mountpoint>
   ```

   Beispiel: Wenn Ihre IPv4-Adresse `192.168.1.25` lautet und Ihr Mountpoint `/stream` ist:

   ```
   http://192.168.1.25:8000/stream
   ```

6. Der Stream sollte nun abgespielt werden. Sie können auch `http://<IPv4-Adresse>:8000/` besuchen, um die vollständige Icecast-Statusseite von diesem Gerät aus zu sehen.

> 💡 **Tipp:** Die IPv4-Adresse kann sich ändern, wenn der Router sie später neu zuweist (besonders nach einem Neustart). Falls der Stream nicht mehr erreichbar ist, wiederholen Sie die Schritte 1–4, um zu prüfen, ob sich die Adresse geändert hat.

---

## Kurze Fehlerbehebung

- **BUTT verbindet sich nicht / "Connection failed":**
  - Prüfen Sie, ob Icecast läuft (das Fenster von `icecast.bat` noch geöffnet ist).
  - Stellen Sie sicher, dass das Passwort in BUTT genau mit `source-password` in `icecast.xml` übereinstimmt.
  - Prüfen Sie, ob der Port (standardmäßig `8000`) mit dem in `icecast.xml` übereinstimmt.

- **Kein Ton / Stille im Stream:**
  - Prüfen Sie, ob das richtige Mikrofon/Audiogerät in den Audio-Einstellungen von BUTT ausgewählt ist.
  - Stellen Sie sicher, dass das Eingabegerät in den Windows-Soundeinstellungen nicht stummgeschaltet ist.

- **Stream von einem anderen Gerät nicht erreichbar:**
  - Verwenden Sie die lokale IP-Adresse Ihres Computers (z. B. `192.168.x.x`) anstelle von `localhost`.
  - Stellen Sie sicher, dass Ihre Firewall Verbindungen auf Port `8000` zulässt (siehe Teil 1, Schritt 4A).

---

## Zusammenfassung

1. Legen Sie ein sicheres `source-password` (und weitere Passwörter) in `Icecast/icecast.xml` fest.
2. Doppelklicken Sie auf `Icecast/icecast.bat`, um den Server zu starten.
3. Entpacken und starten Sie BUTT, fügen Sie dann einen Server hinzu, der auf `localhost:8000` mit Ihrem `source-password` zeigt.
4. Wählen Sie Ihr Mikrofon in den Audio-Einstellungen von BUTT aus und klicken Sie auf ▶, um live zu gehen.
5. Hören Sie unter `http://localhost:8000/<mountpoint>` zu.
