# Schnittstelle Pi ↔ Steuerknoten (MQTT)

> **Entwurf. Muss mit dem ESP-Team abgestimmt werden.** Änderungen hier versionieren.

## Prinzip
- Auf dem Pi läuft ein **MQTT-Broker (Mosquitto)**, quasi das „Postamt“
- Steuerknoten **veröffentlichen** (publish) ihre Werte selbst, sobald sie neue haben (Push statt Abfragen)
- Das Backend **abonniert** (subscribe) alle Topics und speichert die Werte
- Befehle und Konfiguration schickt das Backend ebenfalls per MQTT an die Knoten
- Datenformat: **JSON**
- Die REST-API des Steuerknotens bleibt für dessen **eigene Website**. Mit dem Pi spricht er nur über MQTT.

```mermaid
flowchart LR
    SK[Steuerknoten] -- Messwerte, Status --> MQ[Mosquitto<br/>auf dem Pi]
    MQ -- Befehle, Config --> SK
    MQ <--> BE[Backend]
```

## Topics (Vorschlag)
`<k>` = ID des Steuerknotens (z. B. `sk-a1b2c3` aus der MAC-Adresse)

| Topic | Richtung | Inhalt | Retained | QoS |
|---|---|---|---|---|
| `bewaesserung/<k>/status` | ESP → Pi | `online` / `offline` (Last Will) | ja | 1 |
| `bewaesserung/<k>/info` | ESP → Pi | Name, Firmware, IP, Akku/Spannung | ja | 1 |
| `bewaesserung/<k>/sensor/<id>` | ESP → Pi | Messwert (siehe unten) | nein | 0 |
| `bewaesserung/<k>/pumpe/<id>/state` | ESP → Pi | `on` / `off` | ja | 1 |
| `bewaesserung/<k>/pumpe/<id>/cmd` | Pi → ESP | `{"cmd":"on","duration_s":10}` | nein | 1 |
| `bewaesserung/<k>/config` | Pi → ESP | komplette Konfiguration | **ja** | 1 |
| `bewaesserung/<k>/config/ack` | ESP → Pi | `{"version":7,"ok":true}` | nein | 1 |

- **Retained:** Der Broker merkt sich die letzte Nachricht. Ein Knoten, der neu startet, bekommt seine Config sofort wieder.
- **Last Will:** Der Knoten hinterlegt beim Verbinden `offline`. Bricht die Verbindung ab, verschickt der Broker das automatisch. Damit ist die Offline-Erkennung gratis.

## Beispiel Messwert
`bewaesserung/sk-a1b2c3/sensor/s1`
```json
{"type": "soil_moisture", "value": 42, "unit": "%"}
```
Werte von kabellosen Sensorknoten schickt der Steuerknoten weiter, mit Herkunft:
```json
{"type": "temperature", "value": 21.5, "unit": "°C", "source": "sensorknoten-3"}
```

## Beispiel Config
`bewaesserung/sk-a1b2c3/config` (retained)
```json
{
  "version": 7,
  "name": "Hochbeet",
  "interval_s": 60,
  "sensors": [{"id": "s1", "type": "soil_moisture", "pin": 34, "min": 1200, "max": 3000}],
  "pumps": [{"id": "p1", "pin": 25}],
  "rules": [{"sensor": "s1", "below": 30, "pump": "p1", "duration_s": 10, "cooldown_s": 3600}]
}
```
- Mit `version` kann der Pi prüfen, ob der Knoten die aktuelle Config übernommen hat (`config/ack`)

## Regeln
- **Zeitstempel vergibt der Pi** beim Empfang, dann braucht der ESP keine genaue Uhrzeit
- **Broker-Adresse:** `bewaesserung.local` (mDNS), Port `1883`. Fallback: IP auf der Website des Steuerknotens eintragen.
- **Absicherung:** Mosquitto mit Benutzer und Passwort (`allow_anonymous false`), nur im Heim-WLAN
- **Pi oder Broker weg:** Der ESP gießt mit der zuletzt gespeicherten Config weiter und verbindet sich automatisch neu
- Sensortypen und Einheiten einheitlich halten (Liste gemeinsam festlegen)

## Werkzeuge
- **ESP:** `PubSubClient` oder `AsyncMqttClient` (Arduino) bzw. `esp-mqtt` (ESP-IDF), dazu `ArduinoJson`
- **Pi/Backend:** `aiomqtt` (Python)
- **Debuggen:** [MQTT Explorer](https://mqtt-explorer.com) am Laptop oder `mosquitto_sub -v -t 'bewaesserung/#'`
- **Home Assistant** kann später direkt am selben Broker mitlesen
