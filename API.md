# API-Beschreibung

## Endpoint

```text
POST https://www.ortsnetz-auslastung.de/v1/measurements
Content-Type: application/json
```

Der Endpoint ist öffentlich und benötigt keine Authentifizierung. Jede Messung wird einzeln übertragen.

## Sendeintervall

Messungen sollen pro Messquelle alle **300 Sekunden (5 Minuten)** gesendet werden. Das Sendeintervall wird durch den Client bzw. die Integration gesteuert.

## Request-Felder

| Feld | Typ | Pflicht | Beschreibung |
| --- | --- | --- | --- |
| `observed_at` | String | Ja | ISO-8601-Zeitstempel mit Zeitzone, bevorzugt UTC, z. B. `2026-09-15T08:00:00Z` |
| `latitude` | Zahl | Ja | Breitengrad von -90 bis 90 |
| `longitude` | Zahl | Ja | Längengrad von -180 bis 180 |
| `l1_v` | Zahl | Ja | Spannung Phase L1 in Volt, größer als 0 und maximal 500 |
| `l2_v` | Zahl | Ja | Spannung Phase L2 in Volt, größer als 0 und maximal 500; bei einphasigen Messungen `-1` |
| `l3_v` | Zahl | Ja | Spannung Phase L3 in Volt, größer als 0 und maximal 500; bei einphasigen Messungen `-1` |
| `grid_frequency_hz` | Zahl | Nein | Netzfrequenz in Hertz, 45 bis 55 |
| `plant_capacity_kwp` | Zahl | Nein | Installierte PV-Leistung in kWp, größer als 0 und maximal 1000 |
| `pv_forecast_kwh` | Zahl | Nein | heutige Gesamt PV-Prognose in kWh, 0 bis 100000 |
| `smartmeter_model` | String | Nein | Freie Bezeichnung des Messgeräts, maximal 120 Zeichen |
| `integration_version` | String | Nein | Kennung der Integration, maximal 32 Zeichen |

## Beispiel

```json
{
  "observed_at": "2026-09-15T08:00:00Z",
  "latitude": 52.520008,
  "longitude": 13.404954,
  "l1_v": 230.1,
  "l2_v": 229.9,
  "l3_v": 230.4,
  "grid_frequency_hz": 50.0,
  "integration_version": "iobroker-0.1.0"
}
```

## Responses

| Status | Bedeutung |
| --- | --- |
| `202 Accepted` | Messung angenommen; die Antwort enthält `accepted`, `created`, eine Ampelbewertung je Phase und insgesamt sowie eine Speicherladeempfehlung. |
| `422 Unprocessable Entity` | Pflichtfeld fehlt, hat ein ungültiges Format oder liegt außerhalb des erlaubten Wertebereichs. |
| `403 Forbidden` | Der Standort ist für die Annahme gesperrt. |

Beispielantwort bei Annahme:

```json
{
  "accepted": true,
  "created": true,
  "status": {
    "l1": "green",
    "l2": "green",
    "l3": "green",
    "overall": "green"
  },
  "storage_recommendation": "none"
}
```

## Speicherladeempfehlung

`storage_recommendation` ist eine direkt aus der übertragenen Messung abgeleitete Empfehlung. Es gibt keinen zusätzlichen API-Aufruf.

| Wert | Bedeutung | Bedingung |
| --- | --- | --- |
| `discharge` | Speicher entladen | Mindestens eine gemessene Phase liegt unter ihrer unteren Warnschwelle. |
| `charge` | Speicher laden | Keine Phase liegt unter der unteren Warnschwelle und mindestens eine gemessene Phase liegt über ihrer oberen Warnschwelle. |
| `none` | Keine Aktion | Alle gemessenen Phasen liegen innerhalb der Warnschwellen. |

Nicht verfügbare Phasen mit dem Wert `-1` werden ignoriert. Liegen gleichzeitig Unter- und Überspannungen vor, hat `discharge` Vorrang. Die Schwellenwerte entsprechen immer der auf dem Server konfigurierten Spannungsskala.

## Geplante optionale Messfelder

Diese Erweiterung beschreibt geplante zusätzliche Felder für `POST /v1/measurements`.
Sie ist noch nicht im Backend implementiert: Die neuen Werte werden derzeit
nicht als Messfelder validiert oder gespeichert. Bestehende Pflichtfelder und
Antworten bleiben unverändert. Alle nachfolgenden Felder sind optional und
können unabhängig voneinander weggelassen werden.

| Feld | Typ | Einheit / Bereich | Beschreibung |
| --- | --- | --- | --- |
| `l1_v_min`, `l1_v_max`, `l1_v_avg` | Zahl | V | Minimum, Maximum und arithmetischer Mittelwert der Spannung an L1 |
| `l2_v_min`, `l2_v_max`, `l2_v_avg` | Zahl | V | Minimum, Maximum und arithmetischer Mittelwert der Spannung an L2 |
| `l3_v_min`, `l3_v_max`, `l3_v_avg` | Zahl | V | Minimum, Maximum und arithmetischer Mittelwert der Spannung an L3 |
| `grid_frequency_hz_min`, `grid_frequency_hz_max`, `grid_frequency_hz_avg` | Zahl | Hz | Minimum, Maximum und arithmetischer Mittelwert der Netzfrequenz |
| `battery_capacity` | Zahl | kWh | Nutzbare Batteriespeicherkapazität; kein Ladezustand und kein aktueller Energieinhalt |
| `power_factor` | Zahl | Dimensionslos, 0 bis 1 einschließlich | Leistungsfaktor: Verhältnis von Wirkleistung zu Scheinleistung (`P / S`) |
| `apparent_power_va` | Zahl | VA | Scheinleistung in Voltampere; keine Wirkleistung |
| `power_w` | Zahl | W | Wirkleistung in Watt; keine Scheinleistung |
| `l1_a`, `l2_a`, `l3_a` | Zahl | A | Aktuelle Stromstärke an L1, L2 beziehungsweise L3; keine Leistung |

`min`, `max` und `avg` werden vom Messclient aus den gültigen Einzelwerten
desselben Übertragungsintervalls gebildet, üblicherweise fünf Minuten. Sie sind
keine vom Server berechnete Tagesstatistik. Genaue Intervallgrenzen werden mit
diesen zusätzlichen Feldern nicht übertragen. Wenn alle drei Kennzahlen eines
Messwerts vorhanden sind, gilt `min <= avg <= max`.

Bei einphasigen Messungen bleiben nicht gemessene Phasen mit `-1` markiert.
Das gilt weiterhin für `l1_v`, `l2_v` und `l3_v` und ebenso für die neuen
Stromstärkefelder `l1_a`, `l2_a`, `l3_a` sowie alle phasenbezogenen
`_min`-, `_max`- und `_avg`-Felder, sofern sie übertragen werden.
Die neuen Felder bleiben optional. Beispiel bei ausschließlicher Messung von L1:
`l1_v = 230.1`, `l2_v = -1`, `l3_v = -1`.
Mindestens eine Phase muss einen gültigen Spannungswert liefern.

`-1` ist ausschließlich ein Kennzeichen für eine nicht gemessene Phase,
kein Messwert, und wird bei Mittelwerten, Min/Max-Auswertungen und der
Spannungsbewertung ignoriert. Sonstige nicht verfügbare optionale Kennzahlen
werden weggelassen. Für Frequenz, Batteriespeicherkapazität, Leistungsfaktor
und nicht phasenbezogene Leistungsfelder wird `-1` nicht als Ersatzwert verwendet.

`power_factor` bezeichnet den Leistungsfaktor, nicht den
Phasenwinkel in Grad/Radiant oder die Anzahl der gemessenen Phasen.
Bei sinusförmigen Signalen gilt `power_factor = cos(φ)`; bei verzerrten
Signalen ist diese Gleichsetzung nicht allgemein gültig. Für diese Erweiterung
wird der nichtnegative Wert von 0 bis 1 übertragen; eine Energieflussrichtung
wird daraus nicht abgeleitet.

Beispiel zusätzlicher Felder (Auszug, kein vollständiger Request; erst nach
Backend-Unterstützung verwenden):

```json
{
  "l1_v_min": 228.4,
  "l1_v_max": 232.1,
  "l1_v_avg": 230.2,
  "grid_frequency_hz_min": 49.98,
  "grid_frequency_hz_max": 50.03,
  "grid_frequency_hz_avg": 50.01,
  "battery_capacity": 10.2,
  "power_factor": 0.95,
  "apparent_power_va": 3700.0,
  "power_w": 3515.0,
  "l1_a": 5.2,
  "l2_a": 4.8,
  "l3_a": 6.1
}
```
