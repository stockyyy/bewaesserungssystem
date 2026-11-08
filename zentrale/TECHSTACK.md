# Techstack der Zentrale

> Ziel: so schlank, dass es auf einem **Pi Zero 2 W (512 MB)** flüssig läuft.

## Betriebssystem
- **Raspberry Pi OS Lite 64-bit** (ohne Desktop, spart RAM)
- Zugriff nur per **SSH**
- Hostname `bewaesserung`, dann ist die Website unter `http://bewaesserung.local` erreichbar (mDNS/avahi ist vorinstalliert)

## MQTT-Broker
- **Mosquitto** (`eclipse-mosquitto:2`), sehr klein (wenige MB RAM)
- Benutzer und Passwort, `allow_anonymous false`, nur im Heim-WLAN
- Persistenz an, damit retained Nachrichten (Config, Status) einen Neustart überleben
- Details zu Topics: [SCHNITTSTELLE.md](SCHNITTSTELLE.md)

## Backend
- **Python 3 + FastAPI** (mit Uvicorn)
  - einfach zu lernen, erzeugt automatisch eine API-Doku unter `/docs`
  - REST + **WebSocket** für die eigene Website (Live-Werte ohne Neuladen)
- **aiomqtt**: abonniert `bewaesserung/#`, schickt Befehle und Config an die Knoten
- **APScheduler**: regelmäßige Jobs (Daten verdichten, Warnungen prüfen)
- **SQLModel**: Datenbankzugriff mit wenig Code
- Das Backend liefert auch die fertig gebaute Website aus, ein extra Webserver (nginx) ist nicht nötig

## Datenbank
- **SQLite** (eine Datei, kein eigener Server), im **WAL-Modus**
- Rohdaten **30 Tage** behalten, danach zu **Stundenmitteln** verdichten
  - hält die DB klein und schont die SD-Karte
- Tabellen grob: Knoten, Sensor, Aktor, Messwert, Regel, Ereignis (Log)

## Frontend (Website)
- **SvelteKit** mit `adapter-static` (wird zu statischen Dateien gebaut, auf dem Pi läuft kein Node)
- **Tailwind CSS** fürs Styling
- **uPlot** oder **Chart.js** für Diagramme (leichtgewichtig)
- **Mobile first**, weil meist am Handy genutzt
- Seiten: Dashboard (Kacheln je Knoten/Pflanze), Detail mit Verlauf, Konfiguration, Log

## Deployment
- Zwei Möglichkeiten:
  - **Docker Compose** mit *zwei* Containern (`mosquitto` + `app`): sauber und reproduzierbar
  - **ohne Docker:** Mosquitto per `apt`, Backend als systemd-Service. Noch sparsamer auf dem Zero.
- **Nicht auf dem Pi bauen.** Am Laptop oder per GitHub Actions für `arm64` bauen, auf dem Pi nur noch starten.
- Konfiguration (Passwörter etc.) in `.env`, nicht ins Repo

## Qualität
- Backend: `ruff` (Linting) + `pytest`
- Frontend: `prettier` + `eslint`

## Alternative ohne Eigenbau
| Option | Vorteil | Nachteil |
|---|---|---|
| **Home Assistant** | fertige Dashboards, Verlauf, Handy-App | eher schwer für den Zero (lieber Pi 4), weniger eigener Code, Lerneffekt kleiner |
| **Node-RED** | Logik zusammenklicken, schnell erste Ergebnisse | Dashboard weniger schön und flexibel |

- Empfehlung: **selbst bauen** (passt zum Projekt, volle Kontrolle). Home Assistant kann später als Zusatz direkt am selben MQTT-Broker mitlesen (siehe Ideen in `info.md`).
