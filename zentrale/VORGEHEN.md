# Vorgehen

## 1 – Recherche & Entscheidung
- Hardware festlegen und Bestellung ans HW-Team geben (→ [HARDWARE.md](HARDWARE.md))
- Techstack festlegen (→ [TECHSTACK.md](TECHSTACK.md))
- MQTT-Topics und JSON-Format mit dem ESP-Team besprechen (→ [SCHNITTSTELLE.md](SCHNITTSTELLE.md))
- **Fertig, wenn:** Hardware bestellt, Topics von beiden Seiten abgenickt

## 2 – Grundgerüst + Broker + ESP-Simulator
- Repo-Struktur anlegen (`zentrale/backend`, `zentrale/frontend`, `zentrale/mosquitto`)
- `docker-compose.yml` mit Mosquitto, lokal am Laptop starten
- **ESP-Simulator** schreiben: kleines Python-Skript, das wie ein Steuerknoten Fake-Werte per MQTT veröffentlicht (inkl. Last Will)
  - So können wir arbeiten, ohne auf die Hardware zu warten, und das ESP-Team hat ein Beispiel
- **Fertig, wenn:** die Simulator-Werte im MQTT Explorer auftauchen

## 3 – Daten empfangen & speichern
- Backend abonniert `bewaesserung/#` und speichert die Werte in SQLite
- Online/Offline über das `status`-Topic (Last Will)
- **Fertig, wenn:** Werte von 2–3 simulierten Knoten landen in der DB

## 4 – Dashboard
- Übersicht mit Kacheln je Knoten/Sensor (aktueller Wert, Ampel, online/offline)
- Detailseite mit Verlaufsdiagramm
- **Fertig, wenn:** am Handy unter `bewaesserung.local` (bzw. lokal) nutzbar

## 5 – Konfiguration & Steuerung
- Knoten hinzufügen/umbenennen, Sensoren und Pumpen zuordnen, Schwellwerte setzen
- Konfiguration retained auf `bewaesserung/<k>/config` schicken, Bestätigung über `config/ack` anzeigen
- Manuell gießen (`pumpe/<id>/cmd`)
- **Fertig, wenn:** eine geänderte Einstellung am echten ESP ankommt und bestätigt wird

## 6 – Ausbau
- Warnungen: Knoten offline, Akku niedrig, Flasche leer
- Daten verdichten (Stundenmittel nach 30 Tagen)
- Log aller Gießvorgänge

## 7 – Auf den echten Pi
- Pi einrichten (Pi OS Lite, SSH, Hostname `bewaesserung`)
- Mosquitto-Benutzer anlegen, Passwörter in `.env` bzw. `mosquitto/passwd` (nicht ins Repo)
- Deployment per Docker oder systemd, automatischer Start nach Neustart
- Test mit echten Steuerknoten, Kurz-Doku im README
- **Fertig, wenn:** nach Stromausfall läuft alles von selbst wieder

## Absprachen mit anderen Teams
- **ESP-Team:** Topics, JSON-Format, Sensortypen, Broker-Adresse und Zugangsdaten, MQTT-Bibliothek auf dem ESP
- **HW-Team:** Bestellung Pi + Zubehör, Test-ESP mit Sensor und Pumpe für uns

## Risiken
| Risiko | Gegenmaßnahme |
|---|---|
| Pi Zero hat wenig RAM | schlanker Stack, nicht auf dem Pi bauen, ggf. Pi 4 |
| SD-Karte nutzt sich ab | wenig schreiben, Daten verdichten, Markenkarte, Backup |
| Hardware kommt spät | mit dem Simulator weiterarbeiten |
| Schnittstelle ändert sich | früh festlegen, Änderungen in `SCHNITTSTELLE.md` versionieren |
| Pi/Broker fällt aus | ESP gießt mit gespeicherter Config weiter und verbindet sich automatisch neu |
