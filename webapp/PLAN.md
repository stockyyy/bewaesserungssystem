# WebApp – Plan & Techstack
Team: Tobias, Ester

## Ziel
- Web-Oberfläche auf dem Pi Zero 2 W (Docker)
- Live-Werte und Verlauf aller Sensoren
- Konfigurator: ESPs, Sensoren, Pumpen und Regeln modular zusammenklicken
- Konfiguration wird per MQTT an die ESPs geschickt
- Funktioniert mit 1 ESP (Basic) und mit vielen ESPs (Pro)

## Architektur
- ESP32 (1..n) → WLAN → **MQTT-Broker** (Pi) → **Backend** → **SQLite**
- Backend → MQTT → ESP (Befehle, Config)
- Browser ↔ Backend (REST + WebSocket für Live-Werte)
- **Wichtig:** Die Grundregel („trocken → gießen“) läuft auf dem ESP, damit auch bei ausgefallenem Pi weiter gegossen wird
- Der Pi übernimmt nur Anzeige, Konfiguration, Historie und Regeln über mehrere ESPs hinweg

## Techstack
- **OS:** Raspberry Pi OS Lite 64-bit
- **Container:** Docker + Docker Compose v2
  - nur 2 Container: `mosquitto` + `app` (wegen 512 MB RAM)
- **Broker:** `eclipse-mosquitto:2` (mit User/Passwort, nur im LAN)
- **Backend:** Python 3.12
  - FastAPI + Uvicorn
  - `aiomqtt` als MQTT-Client
  - SQLModel (Pydantic + SQLAlchemy)
  - APScheduler für Regeln und Aufräum-Jobs
  - liefert auch das gebaute Frontend aus (kein extra nginx nötig)
- **DB:** SQLite im WAL-Modus, als Docker-Volume
  - Rohdaten 30 Tage behalten, danach Stundenmittel (schont die SD-Karte)
- **Frontend:**
  - SvelteKit mit `adapter-static`
  - Tailwind CSS
  - uPlot oder Chart.js für Diagramme (leichtgewichtig)
- **Erreichbarkeit:** `http://bewaesserung.local` (mDNS über avahi)
- **Build/CI:**
  - GitHub Actions + `docker buildx` → arm64-Image → GHCR
  - Auf dem Pi nur `docker compose pull && up -d`, dort wird nicht gebaut
- **Qualität:** `ruff` + `pytest` (Backend), `prettier` + `eslint` (Frontend)
- **API-Doku:** wird von FastAPI automatisch unter `/docs` erzeugt (praktisch fürs ESP-Team)

## Repo-Struktur
```
webapp/
  docker-compose.yml
  backend/      (app/, tests/, Dockerfile)
  frontend/     (SvelteKit)
  mosquitto/    (mosquitto.conf, passwd)
  tools/esp_simulator.py   ← Fake-ESP zum Testen ohne Hardware
  docs/schnittstelle.md
```

## Schnittstelle MQTT (mit ESP-Team abstimmen)
- `garden/<espId>/status` → `online`/`offline` (Last Will), Akkustand, Firmware-Version
- `garden/<espId>/sensor/<sensorId>` → `{"type":"soil_moisture","value":42,"unit":"%"}`
- `garden/<espId>/actuator/<id>/cmd` ← `{"cmd":"on","duration_s":10}`
- `garden/<espId>/actuator/<id>/state` → `on`/`off`
- `garden/<espId>/config` (retained) ← Sensoren, Pins, Intervalle, Schwellwerte
- `garden/<espId>/config/ack` → ESP bestätigt die übernommene Config
- Zeitstempel vergibt der Pi beim Empfang (ESP braucht dann keine Uhrzeit)

## Datenmodell
- **Device:** id, name, last_seen, battery, online
- **Sensor:** id, device_id, type, unit, pin, driver, kalibrierung (min/max)
- **Actuator:** id, device_id, type (pump/light), pin
- **Reading:** sensor_id, ts, value
- **Rule:** sensor, Bedingung (`<`/`>`), Schwellwert, Aktor, Dauer, Cooldown, aktiv
- **Event:** Log-Einträge (Pumpe an, Warnung, ESP offline)

## Sensor-Katalog (für „anderer Hersteller einfach“)
- `sensor_catalog.yaml`: Typ, Hersteller, Einheit, Kalibrierwerte, ESP-Treibername
- Neuer Sensor = neuer Eintrag im Katalog, keine Code-Änderung im Frontend
- Typen: Bodenfeuchte, Temperatur, Luftfeuchte, Helligkeit, Füllstand Flasche

## REST-API (Auszug)
- `GET/POST/PUT/DELETE /api/devices`
- `GET /api/sensors/{id}/readings?from=&to=`
- `POST /api/actuators/{id}/command`
- `GET/POST/PUT/DELETE /api/rules`
- `POST /api/devices/{id}/push-config`
- `WS /api/live`

## Seiten im Frontend
- **Dashboard:** Kacheln je Pflanze/Zone, aktueller Wert, Ampel, Akku
- **Detail:** Verlaufsdiagramm, manuell gießen
- **Konfigurator (Wizard):** ESP hinzufügen → Sensoren aus Katalog → Aktoren → Regeln → an ESP senden
- **Regeln:** Liste, an/aus, bearbeiten
- **Log:** Gießvorgänge, Warnungen
- **Einstellungen:** WLAN-Hinweise, MQTT-Zugang, OLED-Anzeige konfigurieren (welche Werte)
- Mobil zuerst, da meist am Handy genutzt

## Meilensteine (WebApp-Sicht)
- **M1 – Recherche (jetzt)**
  - Techstack festlegen
  - Schnittstelle im Entwurf mit ESP-Team
  - Backlog anlegen
  - Hardware-Wunsch an HW-Team
- **M2 – Grundgerüst**
  - Repo-Skeleton, `docker-compose.yml`, CI für arm64
  - ESP-Simulator schreiben
  - Messwerte empfangen, speichern, einfaches Dashboard
- **M3 – Steuern & Konfigurieren**
  - Pumpe manuell steuern
  - Konfigurator-Wizard + Config-Push
  - Regeln anlegen
- **M4 – Ausbau**
  - mehrere ESPs, Diagramme, Warnungen (Flasche leer, ESP offline, Akku niedrig)
- **M5 – Feinschliff**
  - Design, Doku, Deployment auf dem echten Pi, Demo

## Aufgabenteilung (Vorschlag)
- **Tobias:** Docker, MQTT, Backend, DB, CI
- **Ester:** Frontend, UI-Design, Konfigurator
- **Gemeinsam:** Schnittstelle, API-Design, Tests mit echtem ESP

## Risiken
- Pi Zero 2 W hat wenig RAM → wenige Container, nicht auf dem Pi bauen
- SD-Karte nutzt sich ab → Schreibrate gering halten, Daten verdichten
- Hardware kommt spät → mit dem Simulator unabhängig weiterarbeiten
- Schnittstelle ändert sich → früh festlegen und in `docs/schnittstelle.md` versionieren

## Definition of Done
- Läuft per `docker compose up` auf dem Pi
- Funktioniert mit dem Simulator und mit einem echten ESP
- Kurze Doku im README
