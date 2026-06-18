# Klima – 2-Sensor-Mittelwert-Regelung (Blueprint)

Generischer Klon der Esszimmer-Steuerung (**Bosch**) als **Blueprint**
(`klima_2sensor_blueprint.yaml`). Regelt einen Raum auf eine
Zieltemperatur anhand des **Mittelwerts zweier Sensoren**, steuert Heizen/Kühlen
inkl. Lüfter und Lamellen und meldet jeden echten Zustandswechsel per
HA-Mitteilung. Pro Raum musst du nur die Entitäten auswählen.

## Logik

- **Mittelwert**: nutzt den optionalen Mittelwert-Sensor, sonst den Durchschnitt
  aus Sensor 1 und Sensor 2.
- **Kühlen** ab 25 °C, beendet bei ≤ 23 °C. Über 28 °C starke Stufe (Lüfter
  `low`), sonst leise (`diffuse`), Lamelle `angle1`.
- **Heizen** unter 21,5 °C, beendet bei ≥ 23 °C. Unter 21 °C mit Swing-Lamelle,
  sonst Lamelle `angle5`.
- **Aus**, wenn der Zielbereich erreicht ist und die Mindestlaufzeit erfüllt ist.
- **Pendel-Schutz**: Mindestlaufzeit 30 Min vor einem Moduswechsel (Heizen↔Kühlen)
  plus Hysterese zwischen Ein- und Ausschaltschwellen.
- Geprüft wird bei Sensoränderung, beim HA-Start und alle 5 Minuten.
- **Meldung nur bei echtem Wechsel** (`act`) – keine Spam-Meldungen bei
  Routineprüfungen.

## Auswählen (Pflicht)

1. **Klimagerät** – die `climate`-Entität
2. **Temperatursensor 1** (z. B. Esstisch)
3. **Temperatursensor 2** (z. B. Couch)
4. **Lamellen vertikal** – die `select`-Entität
5. **Lamellen horizontal** – die `select`-Entität

## Optionale Einstellungen (mit Defaults = Esszimmer-Werte)

| Einstellung | Default |
|---|---|
| Mittelwert-Sensor | – (optional) |
| Zieltemperatur | 23 °C |
| Kühlen ein / aus | 25 / 23 °C |
| Heizen ein / aus | 21,5 / 23 °C |
| Kühlen starke Stufe über | 28 °C |
| Heizen Swing unter | 21 °C |
| Mindestlaufzeit | 30 Min |
| Lüfter stark / leise | `low` / `diffuse` |
| Lamelle Heizen / Kühlen / Swing | `angle5` / `angle1` / `swing` |
| Lamelle horizontal | `center` |

> **Wichtig:** Die Lüfter- und Lamellen-**Rohwerte** sind gerätespezifisch (die
> Defaults passen zum Bosch im Esszimmer). Für ein anderes Gerät diese Werte in
> **Entwicklertools → Zustände** prüfen und anpassen.

## Installation

1. In HA: **Einstellungen → Automationen & Szenen → Blueprints → Blueprint
   importieren** (oder Datei nach
   `…/config/blueprints/automation/<beliebig>/klima_2sensor_blueprint.yaml`
   kopieren).
2. **Automation erstellen → aus Blueprint** → Blueprint wählen → Entitäten
   auswählen.

Alternativ direkt in `automations.yaml`:

```yaml
- alias: Esszimmer – Klima (2-Sensor)
  use_blueprint:
    path: klima_2sensor_blueprint.yaml
    input:
      klima_entity: climate.wohnzimmer_hzg_wohnzimmer_clima
      sensor_1: sensor.wohnzimmer_esszimmer_shelly_blu_h_t_bfc9_temperatur
      sensor_2: sensor.bthome_sensor_b8d7_temperatur
      sel_vertikal: select.wohnzimmer_hzg_wohnzimmer_vertical
      sel_horizontal: select.wohnzimmer_hzg_wohnzimmer_horizontal
      sensor_mittelwert: sensor.wohnesszimmer_temperatur_mittelwert
```

## Verbesserungen gegenüber der Einzel-Automation

- `run_min` nutzt die ausgewählte Entität statt eines fest verdrahteten Pfads.
- Mittelwert-Sensor ist optional (Fallback auf Durchschnitt der zwei Sensoren).
- `select.select_option` mit `continue_on_error: true` – eine unbekannte
  Position bricht nicht mehr die ganze Automation ab.
- Meldung nutzt automatisch die Sensor-Namen statt fester Beschriftungen.

## Dateien

- `klima_2sensor_blueprint.yaml` – das Blueprint
- `klima_2sensor_README.md` – diese Anleitung
- `esszimmer_klimasteuerung_simpel.yaml` – die ursprüngliche Einzel-Automation
