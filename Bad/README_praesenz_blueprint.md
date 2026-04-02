# Blueprint: Präsenzsensor – Adaptives Licht mit Lux & Tageszeit

Automatische Lichtsteuerung per Präsenzsensor mit adaptiver Helligkeit basierend auf Tageszeit und aktuellem Lux-Wert.

---

## Funktionsweise

| Trigger | Bedingung | Aktion |
|---|---|---|
| Präsenz erkannt | Lux < Schwellwert, Morgenzeit | Morgen-Szene + Morgenhelligkeit |
| Präsenz erkannt | Lux < Schwellwert, Tagzeit | Tag-Szene + Taghelligkeit |
| Präsenz erkannt | Lux < Schwellwert, Nachtzeit | Nacht-Szene + Nachthelligkeit |
| Präsenz erkannt | Lux ≥ Schwellwert | Licht bleibt aus (Raum hell genug) |
| Präsenz weg | – | Licht aus (optional verzögert) |

---

## Voraussetzungen

- Präsenzsensor als `binary_sensor` (z.B. Aqara FP300)
- Helligkeitssensor als `sensor` mit Lux-Wert (z.B. FP300 Beleuchtungsstärke)
- Lampe als `light` Entität
- 3 vordefinierte Szenen (Morgen, Tag/Abend, Nacht) mit gewünschter Lichtfarbe

---

## Installation

1. Datei `blueprint_praesenz_adaptiv.yaml` ablegen unter:
   ```
   config/blueprints/automation/
   ```
2. In HA: **Entwicklertools → Schnellstart → Blueprints neu laden**
3. **Einstellungen → Automationen → Blueprint-Automation erstellen**

---

## Konfigurierbare Parameter

| Parameter | Beschreibung | Standard |
|---|---|---|
| Präsenzsensor | Binary Sensor der Anwesenheit erkennt | – |
| Helligkeitssensor | Lux-Sensor im Raum | – |
| Lux-Schwellwert | Über diesem Wert bleibt Licht aus | 300 lx |
| Lampe | Zu steuernde Licht-Entität | – |
| Szene Morgen | Szene für Morgenzeitraum | – |
| Szene Tag/Abend | Szene für Tag- und Abendzeitraum | – |
| Szene Nacht | Szene für Nachtzeitraum | – |
| Morgen ab | Beginn Morgenzeitraum | 06:00 |
| Morgen bis | Ende Morgenzeitraum | 10:00 |
| Nacht ab | Beginn Nachtzeitraum | 23:00 |
| Helligkeit Morgen | Helligkeit in % für Morgen | 70% |
| Helligkeit Tag/Abend | Helligkeit in % für Tag | 90% |
| Helligkeit Nacht | Helligkeit in % für Nacht | 30% |
| Ausschaltverzögerung | Sekunden bis Licht ausgeht | 0 s |

---

## Beispielkonfiguration (Bad)

```
Präsenzsensor:   binary_sensor.presence_multi_sensor_fp300_belegung
Helligkeitssensor: sensor.presence_multi_sensor_fp300_beleuchtungsstarke
Lux-Schwellwert: 300 lx
Lampe:           light.kajplats_e27_cws_globe_1055lm_2
Szene Morgen:    scene.bad_morgens
Szene Tag:       scene.sexy_bad_licht
Szene Nacht:     scene.sexy_bad_licht
Morgen ab:       06:00
Morgen bis:      10:00
Nacht ab:        23:00
Helligkeit Morgen:  70%
Helligkeit Tag:     90%
Helligkeit Nacht:   30%
Ausschaltverzögerung: 0 s
```

---

## Hinweise

- **Szenen** definieren nur die Lichtfarbe – die Helligkeit wird separat durch den Blueprint gesetzt und überschreibt den Szenen-Wert
- **Ausschaltverzögerung** ist sinnvoll bei Sensoren die bei Bewegungslosigkeit kurz die Präsenz verlieren (z.B. beim Zähneputzen)
- Der Blueprint funktioniert mit **allen Präsenzsensoren** die einen `binary_sensor` liefern – nicht nur dem FP300
- **Lux-Schwellwert 0** deaktiviert die Lux-Prüfung effektiv (Licht geht immer an)
