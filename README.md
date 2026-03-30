# IKEA BILRESA Dual Button – Home Assistant Automation

## Gerät & Integration
- **Gerät:** IKEA BILRESA Dual Button Switch
- **Protokoll:** Matter over Thread (direkt in HA eingebunden, kein Hub)
- **Entitäten:**
  - `event.bilresa_dual_button_taste_1`
  - `event.bilresa_dual_button_taste_2`

---

## Event-Mapping

| Aktion | event_type |
|---|---|
| Kurzdruck | `multi_press_1` |
| Gedrückt halten | `long_press` |
| Loslassen nach Halten | `long_release` |
| Doppeldruck | `multi_press_2` |

---

## Wichtige Erkenntnisse

### Trigger-Problem bei Matter
`platform: state` mit `attribute: event_type` triggert **nicht**, wenn dieselbe Taste zweimal hintereinander gedrückt wird (Attributwert ändert sich nicht, nur der Timestamp).

**Fix:** Trigger auf den State (Timestamp), Event-Typ per Template-Bedingung im `choose:`-Block prüfen:

```yaml
trigger:
  - platform: state
    entity_id: event.bilresa_dual_button_taste_1
    id: taste1

# Bedingung im choose-Block:
- condition: template
  value_template: "{{ trigger.to_state.attributes.event_type == 'multi_press_1' }}"
```

### Farbtemperatur bei RGB-Lampen
IKEA RGB-Lampen akzeptieren weder `kelvin` noch `color_temp`. Farbtemperatur muss als `rgb_color` übergeben werden.

**Kelvin → RGB Referenz:**

| Uhrzeit | Kelvin | RGB |
|---|---|---|
| 06–09 Uhr | 4000K | `[255, 209, 163]` |
| 09–17 Uhr | 5000K | `[255, 228, 205]` |
| 17–21 Uhr | 3000K | `[255, 180, 107]` |
| 21–06 Uhr | 2200K | `[255, 147, 41]` |

---

## Finale Automation (Schlafzimmer)

```yaml
alias: BILRESA Schlafzimmer Steuerung
description: Taste 1 = Ein (adaptiv 90%) | Taste 2 = Aus | Taste 1 lang = Mood Szene
trigger:
  - platform: state
    entity_id: event.bilresa_dual_button_taste_1
    id: taste1
  - platform: state
    entity_id: event.bilresa_dual_button_taste_2
    id: taste2
action:
  - variables:
      rgb_wert: >-
        {% set h = now().hour %}
        {% if h >= 6 and h < 9 %}
          [255, 209, 163]
        {% elif h >= 9 and h < 17 %}
          [255, 228, 205]
        {% elif h >= 17 and h < 21 %}
          [255, 180, 107]
        {% else %}
          [255, 147, 41]
        {% endif %}
  - choose:
      - conditions:
          - condition: trigger
            id: taste1
          - condition: template
            value_template: "{{ trigger.to_state.attributes.event_type == 'multi_press_1' }}"
        sequence:
          - service: light.turn_on
            target:
              entity_id: light.kajplats_e27_cws_globe_1055lm
            data:
              brightness_pct: 90
              rgb_color: "{{ rgb_wert }}"
      - conditions:
          - condition: trigger
            id: taste2
          - condition: template
            value_template: "{{ trigger.to_state.attributes.event_type == 'multi_press_1' }}"
        sequence:
          - service: light.turn_off
            target:
              entity_id: light.kajplats_e27_cws_globe_1055lm
      - conditions:
          - condition: trigger
            id: taste1
          - condition: template
            value_template: "{{ trigger.to_state.attributes.event_type == 'long_release' }}"
        sequence:
          - service: scene.turn_on
            target:
              entity_id: scene.mood_schlafzimmer
mode: single
```

---

## Blueprint (wiederverwendbar)

Für mehrere gleichartige Schalter wurde ein Blueprint erstellt.

**GitHub:** [blueprintZweischalterIkea.yml](https://github.com/Scryrider/HomeassistantWohnung/blob/main/blueprintZweichalterIkea.yml)

**Installation:**
1. YAML-Datei ablegen unter `config/blueprints/automation/`
2. In HA: Entwicklertools → Schnellstart → **Blueprints neu laden**
3. Einstellungen → Automationen → **Blueprint-Automation erstellen**

**Konfigurierbare Parameter:**
- Taste 1 & Taste 2 Entität
- Lampen-Entität
- Szene
- Helligkeit (Standard: 90%)
- RGB-Farbe je Tageszeit (Color-Picker in der UI)

---

## HACS Switch Manager
Der **Switch Manager** (HACS) unterstützt Matter over Thread aktuell **nicht** (Stand März 2026, Issue #405 offen). Für Matter-Geräte: native HA Automationen oder eigene Blueprints verwenden.
