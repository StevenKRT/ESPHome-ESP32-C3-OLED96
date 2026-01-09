# ESP32-C3 SuperMini mit SSD1306 OLED (128x64) via ESPHome & Home Assistant

Dieses Repository enthält eine vollständige ESPHome-Konfiguration für ein **ESP32-C3 SuperMini** (oder kompatibles ESP32-C3 Board) mit einem **0,96 Zoll SSD1306 OLED (128x64, I2C)**.  
Die Integration erfolgt direkt über die **native ESPHome-API** in **Home Assistant**.

---

## Funktionen

- OLED-Anzeige (SSD1306, 128x64) über I2C
- Uhrzeit-Anzeige (Zeitquelle: Home Assistant)
- Nachrichtensystem mit Queue (bis zu 8 Nachrichten)
- Nachrichten mit optionalem Titel und Body
- Mehrzeilige Texte (`\n` unterstützt)
- Automatisches Paging bei zu vielen Zeilen
- Horizontales Scrollen langer Textzeilen
- Icons über Titel-Präfixe
- Fortschrittsanzeige mit Balken (optional mit Label)
- Zeitgesteuerte Nachrichten (Auto-Clear)
- Display Ein / Aus per Service
- Display-Reset per Service
- OTA-Updates
- WLAN-Fallback-Access-Point
- Verschlüsselte ESPHome-API

---

## Hardware

- ESP32-C3 SuperMini (oder kompatibles ESP32-C3 Board)
- 0,96 Zoll OLED Display SSD1306 (128x64, I2C)
- 3,3 V Stromversorgung

Typische Bezeichnungen:
- 0,96 Zoll IIC Serielles 4-poliges OLED-Anzeigemodul 128x64 (SSD1306)
- ESP32-C3 SuperMini Entwicklungsboard

---

## Verdrahtung

| OLED Pin | ESP32-C3 GPIO |
|---------|---------------|
| SDA     | GPIO8         |
| SCL     | GPIO9         |
| VCC     | 3.3V          |
| GND     | GND           |

- I2C-Adresse: `0x3C`
- I2C-Frequenz: `400 kHz`

---

## ESPHome-Setup

- Board: `esp32-c3-devkitm-1`
- Framework: Arduino
- Display: `ssd1306_i2c`
- Auflösung: `128x64`
- Update-Intervall: `200 ms`

---

## Anzeige-Logik

### Titel & Text

- Titel ist optional
- Body ist optional
- Mehrzeilige Texte werden mit `\n` getrennt
- Titel kann ein Icon-Präfix enthalten

| Präfix | Bedeutung    |
|-------|--------------|
| `[!]` | Warnung      |
| `[i]` | Information  |
| `[ok]`| Erfolg       |
| `[x]` | Fehler       |

---

### Scrolling & Paging

- Textzeilen, die länger als die Displaybreite sind, werden horizontal gescrollt
- Sind mehr Zeilen vorhanden als angezeigt werden können, wird automatisch geblättert
- Seitenwechsel alle 3 Sekunden
- Scrollen enthält Pausen am Anfang und Ende

---

### Fortschrittsanzeige

- Fortschrittsbalken am unteren Rand
- Optional mit Label oder Prozentanzeige
- Reserviert eigenen Bereich, überlagert keinen Text
- Bleibt sichtbar, bis er explizit gelöscht wird

---

## Beispiel

´´´
[i] Update
Firmware wird installiert
Bitte warten
´´´

---

## Home-Assistant-Services

### push_message

Erste Zeile = Titel, Rest = Body

### push_message_timed

Nachricht mit definierter Dauer (Sekunden)

### push_titled

Titel und Text getrennt

### set_default_duration

Setzt die Standard-Anzeigedauer für Nachrichten

### set_progress

Setzt einen Fortschritt ohne Label

### set_progress_labeled

Setzt einen Fortschritt mit Label

### clear_progress

Entfernt die Fortschrittsanzeige

### reset_display

Leert Nachricht, Queue und Fortschritt

### display_on / display_off

Display explizit ein- oder ausschalten

---

## Dateien

- `oled-mini.yaml` – vollständige ESPHome-Konfiguration

---

## Lizenz

Freie Nutzung. Keine Gewährleistung.
