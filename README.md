# ESP32-C3 SuperMini mit SSD1306 OLED (128x64) via ESPHome & Home Assistant

Dieses Repository enthält eine vollständige ESPHome-Konfiguration für ein **ESP32-C3 SuperMini** (oder kompatibles ESP32-C3 Board) mit einem **0,96 Zoll SSD1306 OLED (128x64, I2C)**.  
Die Integration erfolgt in **Home Assistant** über die native **ESPHome-API** (verschlüsselt).

---

## Funktionen

- OLED-Anzeige (SSD1306, 128x64) über I2C
- Uhrzeit-Anzeige (Zeitquelle: Home Assistant)
- Nachrichtenanzeige mit interner Queue (bis zu 8 Nachrichten)
- Nachrichten mit optionalem Titel und Body (Titel/Body automatisch aus erster Zeile getrennt)
- Mehrzeilige Texte (`\n` wird als Zeilenumbruch verarbeitet)
- Automatisches Paging bei zu vielen Zeilen (Seitenwechsel alle 3 Sekunden)
- Horizontales Scrollen langer Textzeilen
- Icons über Titel-Präfixe (`[!]`, `[i]`, `[ok]`, `[x]`)
- Fortschrittsanzeige als Footer-Balken (optional mit Label)
- Fortschritts-Footer erzwingt Display-On (Progress wird sofort sichtbar)
- Display Ein/Aus per Service
- Display-Reset (leert Nachricht, Queue und Fortschritt)
- OTA-Updates
- WLAN-Fallback-Access-Point
- I2C-Scan aktiv (Debug)

---

## Hardware

- ESP32-C3 SuperMini (oder kompatibles ESP32-C3 Board)
- 0,96 Zoll I2C OLED Display (SSD1306, 128x64, 4-Pin)
- 3,3 V Stromversorgung

AliExpress:  
[0,96 Zoll IIC Serielles 4-poliges weißes/blaues/gelbes blaues/gelbes OLED-Anzeigemodul 128 x 64 12864 LCD-Bildschirmplatine für Arduino Oled](https://s.click.aliexpress.com/e/_EQnslT2)
[ESP32-C3 Entwicklungsboard ESP32 SuperMini Entwicklungsboard für Arduino](https://s.click.aliexpress.com/e/_EQ6hSTq)
[DuPont-Kabelverbindung](https://s.click.aliexpress.com/e/_EymMQYY)

3D-Druck:
[Supereinfache PC-Überwachung/Uhr OLED 0.96](https://makerworld.com/de/models/2051935-3eu-super-simple-pc-monitoring-clock-oled-0-96#profileId-2214691)

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
- I2C-Scan: aktiv (`scan: true`)

---

## Technische Parameter

| Parameter         | Wert            |
|------------------|-----------------|
| Display          | SSD1306 128x64  |
| I2C-Adresse      | 0x3C            |
| I2C-Frequenz     | 400 kHz         |
| Update-Intervall | 200 ms          |
| Queue-Größe      | 8 Nachrichten   |
| Paging-Intervall | 3 Sekunden      |
| Zeichenfenster   | 21 Zeichen      |
| Scroll-Takt      | 250 ms          |
| Scroll-Pause     | 1200 ms         |
| Schrift (Body)   | Roboto 11       |
| Schrift (Titel)  | Roboto 14       |
| Glyphs           | inkl. ÄÖÜäöüß    |

---

## Anzeige-Logik

### Titel & Body

- Titel ist optional
- Body ist optional
- Bei `push_message` / `push_message_timed` wird die erste Zeile als Titel interpretiert: `Titel\nBody...`
- Titel kann ein Icon-Präfix enthalten:

| Präfix | Bedeutung    |
|-------|--------------|
| `[!]` | Warnung      |
| `[i]` | Information  |
| `[ok]` | Erfolg      |
| `[x]` | Fehler       |

---

### Mehrzeilig / Paging

- Body wird anhand von `\n` in Zeilen aufgeteilt
- Wenn mehr Zeilen vorhanden sind als darstellbar, wird automatisch geblättert (alle 3 Sekunden)
- Wenn der Fortschritts-Footer aktiv ist, wird der Inhaltsbereich verkleinert (Footer reserviert Platz)

---

### Horizontales Scrollen

- Lange Zeilen werden horizontal gescrollt
- Das Scrollen startet nur, wenn eine Zeile länger als 21 Zeichen ist
- Scrollen enthält Pausen am Anfang und Ende

---

### Fortschrittsanzeige (Footer)

- Fortschritt wird als Footer gezeichnet und immer zuletzt gerendert
- Footer besteht aus:
  - Label/Prozentzeile
  - Progress-Bar (Rahmen + Füllung)
- `set_progress` und `set_progress_labeled` schalten das Display automatisch ein (auch wenn es zuvor per `display_off` deaktiviert wurde)

---

### Standby-Verhalten

- Wenn keine Nachricht aktiv ist und die Queue leer ist:
  - Ohne Fortschritt: Anzeige „Bereit / warte auf Nachricht“
  - Mit aktivem Fortschritt: Standby-Text wird nicht gezeichnet (Footer bleibt sichtbar)

---

## Home-Assistant-Services

### push_message

Sendet eine Nachricht, erste Zeile = Titel, Rest = Body.  
Aktive Nachricht wird in die Queue verschoben, neue Nachricht wird sofort aktiv.

```yaml
service: esphome.oled_mini_push_message
data:
  message: "Titel\nMehrzeiliger\nText"
```

---

### push_message_timed

Wie `push_message`, aber mit definierter Anzeigedauer (Sekunden).  
Nach Ablauf wird automatisch zur nächsten Queue-Nachricht gewechselt.

```yaml
service: esphome.oled_mini_push_message_timed
data:
  message: "Hinweis\nDiese Nachricht verschwindet"
  duration_s: 10
```

---

### push_titled

Titel und Body getrennt übergeben.  
Aktive Nachricht wird in die Queue verschoben, neue Nachricht wird sofort aktiv.

```yaml
service: esphome.oled_mini_push_titled
data:
  title: "[ok] Fertig"
  message: "Vorgang abgeschlossen"
```

---

### set_default_duration

Setzt die Standard-Anzeigedauer (Sekunden) für `push_message` und `push_titled`.  
`0` bedeutet: bleibt sichtbar, bis ersetzt oder reset.

```yaml
service: esphome.oled_mini_set_default_duration
data:
  duration_s: 5
```

---

### set_progress

Setzt Fortschritt in Prozent (0–100).  
Schaltet Display ein (falls aus).

```yaml
service: esphome.oled_mini_set_progress
data:
  percent: 42
```

---

### set_progress_labeled

Setzt Fortschritt in Prozent (0–100) mit Label.  
Schaltet Display ein (falls aus).

```yaml
service: esphome.oled_mini_set_progress_labeled
data:
  label: "Upload"
  percent: 75
```

---

### clear_progress

Entfernt die Fortschrittsanzeige.

```yaml
service: esphome.oled_mini_clear_progress
```

---

### reset_display

Leert aktuelle Nachricht, Queue und Fortschritt.

```yaml
service: esphome.oled_mini_reset_display
```

---

### display_on / display_off

Display explizit ein- oder ausschalten.

```yaml
service: esphome.oled_mini_display_on
```

```yaml
service: esphome.oled_mini_display_off
```

---

## Beispiel

```
[i] Update
Firmware wird installiert
Bitte warten
```

---

## Dateien

- `oled-mini.yaml` – vollständige ESPHome-Konfiguration

---

## Lizenz

Freie Nutzung. Keine Gewährleistung.
