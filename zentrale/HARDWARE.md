# Hardware für die Zentrale

## Anforderungen
- Läuft 24/7, wenig Strom
- WLAN (2,4 GHz reicht, die ESP32 können ohnehin nur 2,4 GHz)
- Genug Leistung für Webserver + kleine Datenbank (keine großen Anforderungen)

## Vergleich

| Rechner | Preis (ca.) | RAM | Vorteile | Nachteile |
|---|---|---|---|---|
| **Pi Zero 2 W** | 20 € | 512 MB | günstig, klein, sparsam (~1 W) | wenig RAM, kein LAN, nur Micro-USB, Bauen auf dem Pi sehr langsam |
| Pi 3B+ | 40 € | 1 GB | LAN, normale USB-Ports | kaum günstiger als Pi 4, veraltet |
| **Pi 4 (2 GB)** | 50 € | 2 GB | genug Reserve, LAN, USB 3 (SSD möglich) | teurer, etwas mehr Strom (~3 W) |
| Pi 5 (2–4 GB) | 60–80 € | 2–4 GB | sehr schnell | überdimensioniert, braucht aktive Kühlung und 27-W-Netzteil |
| Alter Laptop / Mini-PC | 0–100 € | 4 GB+ | oft schon vorhanden, viel Leistung | groß, mehr Strom, nicht „eingebaut“ |

> [!warning] Nicht verwechseln
> Der alte **Pi Zero W** (ohne „2“) ist zu schwach (ARMv6, 1 Kern). Es muss der **Zero 2 W** sein.

> Preise schwanken, vor der Bestellung aktuell nachschauen.

## Empfehlung
- **Pi Zero 2 W reicht**, wenn wir den Software-Stack schlank halten (siehe [TECHSTACK.md](TECHSTACK.md))
- Wenn Budget da ist: **Pi 4 mit 2 GB**. Kostet ca. 30 € mehr, dafür hat man deutlich mehr Reserve und muss weniger auf den Speicher achten.
- Für den Test reicht jeder Rechner im selben WLAN. Die Software ist nicht an den Pi gebunden.

## Zubehör (Bestellliste)
- Netzteil passend zum Modell (Zero 2 W: 5 V / 2,5 A Micro-USB, Pi 4: 5 V / 3 A USB-C)
- microSD **32 GB, Klasse A1/A2** (z. B. SanDisk / Samsung), keine No-Name-Karten
- Gehäuse
- Optional: USB-SSD statt SD-Karte (nur Pi 4/5, deutlich langlebiger)
- Optional für den Zero: Micro-USB-OTG-Adapter + Mini-HDMI-Adapter (nur zum Debuggen, sonst per SSH)

## Tipps
- **Entwickelt wird am Laptop**, der Pi ist nur das Zielsystem
- SD-Karte vor dem ersten Start mit dem **Raspberry Pi Imager** beschreiben und dabei WLAN, SSH und Hostname direkt eintragen
- Ein Backup der SD-Karte machen, wenn alles läuft
