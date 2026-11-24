# Straßenkilometer-API: Kilometer und Koordinaten

Source: https://docs.mapx.at/de/api/road-km/

> Straßenkilometer-API für Österreich: Lage eines Kilometers auf A-, S-, B- und L-Straßen und Straßenkilometer zu einer Koordinate, aus offenen Verwaltungsdaten.

Wandelt Straßennummer und Kilometer (`B1` km 61,2) in Koordinaten um und eine Koordinate in den nächsten Straßenkilometer. Niederösterreich: Hektometerpunkte des Landes Niederösterreich (offene Verwaltungsdaten, täglich aktualisiert); sonst die Kilometerpunkte der GIP.at. `/v1/geocode` versteht dieselbe Eingabe als Freitext (`B1 km 61,2`).

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s) · Guide: [Suche](https://docs.mapx.at/de/guides/search.md)

## Straßenkilometer zu Koordinaten {#road-kilometre-to-coordinates}

`GET /v1/road-km`

Lage eines Kilometers auf einer nummerierten Straße (A, S, B, L), z. B. `road=B1&km=61.2`. Eine Straße kann mehrere Abschnitte mit demselben Kilometer haben (Bundesländer, Kilometersprünge, Äste); alle Treffer werden geliefert, Treffer näher als 150 m zusammengefasst.

Zwischen Nachbarpunkten wird linear interpoliert, nie über eine Lücke (Abstand größer als das Dreifache der Kilometerdifferenz). Je Treffer: `location` [lng, lat], `state`, `name` (Straßenname oder null), `section` (Id des Quellabschnitts), `bearing` (Grad ab Nord in Richtung steigender Kilometer), `accuracy` (geschätzte Lageunsicherheit in Metern), `source` (`noe` oder `gip`). Die Eingabe ist tolerant: `b1`, `B 1`, `B-1`; `km` mit Punkt oder Komma. Unbekannte Straße: 404 `unknown-road`; Kilometer außerhalb aller Abschnitte: 404 `km-out-of-range` mit dem Zusatzfeld `ranges` [\{state, from, to\}]. Quellen: Hektometerpunkte des Landes Niederösterreich (offene Verwaltungsdaten, CC BY 4.0, täglich aktualisiert) für Niederösterreich; sonst die Kilometerpunkte der GIP.at (A, S, B alle 0,1 km, L alle 0,5 km, soweit vorhanden; Autobahnen und Schnellstraßen an ASFINAG-Betriebskilometern kalibriert, CC BY 4.0).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `road` | query | string | ja | – | Straßennummer: A, S, B oder L mit 1 bis 4 Ziffern und optionalem Buchstaben, z. B. `B1`, `L100`, `L1234a` |
| `km` | query | string | ja | – | Kilometer, 0 bis 2000, mit Punkt oder Komma (`61.2`, `61,2`) |
| `state` | query | string |  | – | Nur dieses Bundesland: Name (`Niederösterreich`, auch ohne Umlaute, Groß/klein egal), Kurzform (`bgld`, `ktn`, `noe`, `ooe`, `sbg`, `stmk`, `t`, `vbg`, `w`) oder ISO-Code `AT-1` bis `AT-9` |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/road-km?road=B1&km=61.2" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/road-km?road=B1&km=61.2", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/road-km?road=B1&km=61.2", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "road": "B1",
  "km": 61.2,
  "results": [
    {
      "location": [
        15.691093,
        48.235457
      ],
      "state": "Niederösterreich",
      "name": "Wiener Straße",
      "section": "noe:6101002085",
      "bearing": 64,
      "accuracy": 5,
      "source": "noe"
    }
  ],
  "attribution": "mapx.at | Land Niederösterreich (CC BY 4.0)"
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Koordinaten zu Straßenkilometer {#coordinates-to-road-kilometre}

`GET /v1/road-km/reverse`

Nächste Straßen mit Kilometerpunkten um einen Punkt: Projektion auf die Linie durch die Kilometerpunkte, Kilometer auf 10 m (`km` mit zwei Nachkommastellen), `distance` in Metern vom Punkt zur Straße, nächster zuerst; je Straße und Bundesland nur der nächste Treffer.

Übrige Felder wie bei `/v1/road-km`. Leeres `results`, wenn im `radius` keine Straße liegt. Quellen: Hektometerpunkte des Landes Niederösterreich (offene Verwaltungsdaten, CC BY 4.0, täglich aktualisiert) für Niederösterreich; sonst die Kilometerpunkte der GIP.at (A, S, B alle 0,1 km, L alle 0,5 km, soweit vorhanden; Autobahnen und Schnellstraßen an ASFINAG-Betriebskilometern kalibriert, CC BY 4.0).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `lng` | query | string | ja | – | Länge (WGS84) |
| `lat` | query | string | ja | – | Breite (WGS84) |
| `radius` | query | number (1–500) |  | `50` | Suchradius in Metern, 1 bis 500, Standard 50 |
| `limit` | query | integer (1–20) |  | `5` | 1 bis 20, Standard 5 |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/road-km/reverse?lng=15.69109&lat=48.23546&radius=50" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/road-km/reverse?lng=15.69109&lat=48.23546&radius=50", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/road-km/reverse?lng=15.69109&lat=48.23546&radius=50", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "location": [
    15.6357,
    48.2292
  ],
  "radius": 50,
  "results": [
    {
      "road": "L100",
      "km": 3.45,
      "location": [
        15.635695,
        48.22921
      ],
      "distance": 0.8,
      "state": "Niederösterreich",
      "name": null,
      "section": "noe:26126808",
      "bearing": 21,
      "accuracy": 5,
      "source": "noe"
    }
  ],
  "attribution": "mapx.at | Land Niederösterreich (CC BY 4.0)"
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Straßen mit Kilometerpunkten {#roads-with-kilometre-points}

`GET /v1/road-km/roads`

Alle Straßen mit Kilometerpunkten und ihren zusammenhängenden Kilometerbereichen je Bundesland (`ranges` [\{state, from, to\}]), sortiert nach Klasse (A, S, B, L) und Nummer; zur Autovervollständigung mit `q` (Anfang der Straßennummer, z. B. `B1` liefert B1, B10, B100 …).

Quellen: Hektometerpunkte des Landes Niederösterreich (offene Verwaltungsdaten, CC BY 4.0, täglich aktualisiert) für Niederösterreich; sonst die Kilometerpunkte der GIP.at (A, S, B alle 0,1 km, L alle 0,5 km, soweit vorhanden; Autobahnen und Schnellstraßen an ASFINAG-Betriebskilometern kalibriert, CC BY 4.0).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `q` | query | string |  | – | Anfang der Straßennummer, z. B. `B`, `B1`, `L10` (Leerzeichen werden ignoriert) |
| `state` | query | string |  | – | Nur dieses Bundesland: Name (`Niederösterreich`, auch ohne Umlaute, Groß/klein egal), Kurzform (`bgld`, `ktn`, `noe`, `ooe`, `sbg`, `stmk`, `t`, `vbg`, `w`) oder ISO-Code `AT-1` bis `AT-9` |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/road-km/roads?q=L10&state=Nieder%C3%B6sterreich" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/road-km/roads?q=L10&state=Nieder%C3%B6sterreich", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/road-km/roads?q=L10&state=Nieder%C3%B6sterreich", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "roads": [
    {
      "road": "B1",
      "name": "Wiener Straße",
      "ranges": [
        {
          "state": "Niederösterreich",
          "from": 0,
          "to": 139.6
        },
        {
          "state": "Oberösterreich",
          "from": 139.6,
          "to": 300.2
        }
      ]
    }
  ],
  "attribution": "mapx.at | Land Niederösterreich (CC BY 4.0); GIP.at, ASFINAG (CC BY 4.0)"
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Playground
