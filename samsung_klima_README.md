# Samsung-WindFree-Klima – Blueprint (Skeleton für alle Räume)

Statt für jeden Raum eine eigene Automation zu pflegen, gibt es jetzt **ein
Blueprint** (`samsung_klima_blueprint.yaml`) mit Charlys Logik. Pro Raum wählst
du nur **die climate-Entität** aus – fertig.

## Logik (basiert auf Charly)

- Aktiv **08:00–20:00 Uhr**, um **20:00** wird abgeschaltet.
- Ziel **22 °C**, gehaltenes Band 20–24 °C.
- **Heizen** < 20 °C: `heat`, Lüfter `low`, 22 °C.
- **Kühlen** > 24 °C: `cool`, **ausschließlich WindFree mit Lüfter `auto`**, 22 °C.
- **Abschalten**, wenn 21–23 °C 15 Min stabil sind.

> **Wichtig (Samsung WindFree):** Preset `wind_free` und Lüfter `low` schließen
> sich gegenseitig aus – WindFree *ist* Lüfter `auto`. Beim Kühlen wird deshalb
> nur noch `wind_free` + `auto` erzwungen; ein separater Lüfter-`low`-Modus beim
> Kühlen würde sich endlos mit WindFree abwechseln (Flip-Flop-Schleife).

## Prüfung alle 5 Minuten + Status-Meldung mit Begründung

Das Blueprint prüft **alle 5 Minuten** (zusätzlich ereignisgesteuert bei
Temperaturschwellen). Bei **jeder** Prüfung im aktiven Fenster schreibt es eine
neue Mitteilung „Klima <Raumname>" mit aktuellem **Stand** und **Begründung** –
auch wenn nichts geändert wird. Beispiele:

> *17.06. 14:05 · Raum > 24 °C → Kühlen WindFree (Lüfter Auto), Ziel 22 °C — vorher: Modus Aus · aktuell 24,6 °C*

> *17.06. 14:10 · im Zielbereich (20–24 °C) bzw. kein Moduswechsel nötig → keine Änderung — vorher: Modus Kühlen (Lüfter auto, Preset wind_free) · aktuell 23,1 °C* (wird nur intern ausgewertet, ohne Aktion gibt es keine Meldung)

> **Hinweis:** Gemeldet wird **nur bei tatsächlichen Aktionen** (Moduswechsel,
> WindFree wiederherstellen, Abschalten). Routineprüfungen ohne Änderung
> (`halten`) erzeugen **keine** Meldung – das hält die Anzahl niedrig.
> Über Nacht (außerhalb 08–20 Uhr) wird ohnehin **nicht** gesteuert/gemeldet.
> Titel/ID der Meldung leiten sich automatisch aus dem `friendly_name` der
> Entität ab – kein manuelles Anpassen pro Raum nötig.

## 1. Blueprint installieren

Datei nach `…/config/blueprints/automation/<beliebig>/samsung_klima_blueprint.yaml`
kopieren (oder in HA: **Einstellungen → Automationen & Szenen → Blueprints →
Blueprint importieren**). Danach taucht „Samsung WindFree – Raumtemperatur
halten" in der Blueprint-Liste auf.

## 2. Pro Raum eine Automation anlegen

In HA: **Automation erstellen → aus Blueprint** → Blueprint wählen → Klimagerät
auswählen. Alternativ direkt in `automations.yaml`:

```yaml
- alias: Charly – Klima
  use_blueprint:
    path: samsung_klima_blueprint.yaml
    input:
      klima_entity: climate.charly_charly_klimaanlage

- alias: Henry – Klima
  use_blueprint:
    path: samsung_klima_blueprint.yaml
    input:
      klima_entity: climate.henry_hzg_henry

- alias: Schlafzimmer – Klima
  use_blueprint:
    path: samsung_klima_blueprint.yaml
    input:
      klima_entity: climate.SCHLAFZIMMER_ENTITY   # <-- bitte ergänzen

- alias: Büro – Klima
  use_blueprint:
    path: samsung_klima_blueprint.yaml
    input:
      klima_entity: climate.BUERO_ENTITY          # <-- bitte ergänzen
```

> Charly und Henry sind bereits eingetragen. Für **Schlafzimmer** und **Büro**
> fehlen mir noch die `climate.…`-Entitäten – in Entwicklertools → Zustände
> (Filter z. B. `schlafzimmer` / `buero`) nachsehen und oben einsetzen.

## Voraussetzung pro Gerät

Jedes Gerät muss (wie Charlys Samsung) `preset_mode: wind_free` und
`fan_mode: low`/`auto` unterstützen. Bei abweichenden Rohwerten das Blueprint
anpassen (gilt dann für alle Räume gemeinsam – ein Vorteil des Blueprints).

## Dateien

- `samsung_klima_blueprint.yaml` – das Blueprint (Skeleton)
- `samsung_klima_README.md` – diese Anleitung
