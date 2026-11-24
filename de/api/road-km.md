# Straßenkilometer-API: Kilometer und Koordinaten

Source: https://docs.mapx.at/de/api/road-km/

> Straßenkilometer-API für Österreich: Lage eines Kilometers auf Bundes- und Landesstraßen und Straßenkilometer zu einer Koordinate, aus den Kilometerzeichen der GIP.at.

Wandelt Straßennummer und Kilometer (`B1` km 61,2) in Koordinaten um und eine Koordinate in den nächsten Straßenkilometer. Quelle: die Straßenkilometrierung der GIP.at (offene Verwaltungsdaten) nach GIP-Standard: jeder Hektometer wird aus der kilometrierten Abschnittsfolge berechnet und an den Kilometerzeichen kalibriert; genau auf einem Zeichen kommt dessen Lage (`kind` `marker`), sonst ein berechneter Punkt (`kind` `computed`). Autobahnen und Schnellstraßen haben in der GIP.at keine Zeichen (grobe Genauigkeit), Wien hat keine Kilometrierung. Dieselben Punkte bilden die Kartenebene `road-km`. `/v1/geocode` versteht dieselbe Eingabe als Freitext (`B1 km 61,2`).

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s) · Guide: [Suche](https://docs.mapx.at/de/guides/search.md)

## Straßenkilometer zu Koordinaten {#road-kilometre-to-coordinates}

`GET /v1/road-km`

Lage eines Kilometers auf einer nummerierten Straße (A, S, B, L), z. B. `road=B1&km=61.2`. Eine Straße kann mehrere Abschnitte mit demselben Kilometer haben (Bundesländer, Kilometersprünge, Äste); alle Treffer werden geliefert, Treffer näher als 150 m zusammengefasst.

Ein Kilometer genau auf einem Zeichen liefert dessen Lage (`exact` true, `kind` `marker`), genau auf einem berechneten Hektometer dessen Lage (`exact` false, `kind` `computed`). Sonst wird zwischen den beiden benachbarten Punkten desselben Abschnitts gerechnet (`exact` false, `kind` `computed`, `between` [km, km]): entlang der Straßengeometrie der GIP.at (`interpolation` `road`) oder, wo die Geometrie nicht passt, auf der Geraden (`interpolation` `straight`, größere `accuracy`); nie über eine Lücke (ohne Geometrie und Abstand größer als das Dreifache der Kilometerdifferenz). Je Treffer: `location` [lng, lat], `state`, `name` (Straßenname oder null), `section` (Id der GIP-Kantensequenz), `bearing` (Grad ab Nord in Richtung steigender Kilometer), `accuracy` (geschätzte Lageunsicherheit in Metern), `source` (`gip`), `exact`, `kind` und bei gerechneten Punkten `between` und `interpolation`. Die Eingabe ist tolerant: `b1`, `B 1`, `B-1`; `km` mit Punkt oder Komma. Unbekannte Straße: 404 `unknown-road`; Kilometer außerhalb aller Abschnitte (oder in einer Lücke): 404 `km-out-of-range` mit den Zusatzfeldern `ranges` [\{state, from, to\}] und `before`/`after` (\{state, km\} des nächsten Zeichens darunter und darüber, sonst null). Quelle: die Straßenkilometrierung der GIP.at (offene Verwaltungsdaten, CC BY 4.0) nach GIP-Standard: jeder Hektometer wird aus der kilometrierten Abschnittsfolge (Tabelle LINK_2_EDGE_SEQUENCE) berechnet und an den Kilometerzeichen (Tabelle REFERENCE_POINT) kalibriert, die Zeichen selbst behalten ihre Lage (`kind` `marker`, berechnete Punkte `kind` `computed`). Autobahnen und Schnellstraßen (A, S) haben in der GIP.at keine Zeichen; dort ist die Abschnittsfolge nur grob an den Nummern der Anschlussstellen-Rampen kalibriert (`accuracy` 300 bis 500 m). Wien hat in der GIP.at keine Kilometrierung. Dieselben Punkte bilden das Tileset `road-km` (`/v1/tiles/road-km`).

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
        15.691104,
        48.23548
      ],
      "state": "Niederösterreich",
      "name": "Wiener Straße",
      "section": "gip:244063",
      "bearing": 228,
      "accuracy": 10,
      "source": "gip",
      "exact": false,
      "kind": "computed"
    }
  ],
  "attribution": "mapx.at | GIP.at (CC BY 4.0)"
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

Nächste Straßen mit Kilometerzeichen um einen Punkt: Projektion auf die Straßengeometrie zwischen den beiden benachbarten Zeichen (ohne Geometrie auf die Gerade), Kilometer im Verhältnis der Länge zwischen ihnen, auf 10 m (`km` mit zwei Nachkommastellen); liegt der Lotfußpunkt höchstens 1 m neben einem Zeichen, kommt dieses Zeichen (`exact` true, sein `km` und seine Lage), `distance` in Metern vom Punkt zur Straße, nächster zuerst; je Straße und Bundesland nur der nächste Treffer.

Übrige Felder wie bei `/v1/road-km`. Leeres `results`, wenn im `radius` keine Straße liegt. Quelle: die Straßenkilometrierung der GIP.at (offene Verwaltungsdaten, CC BY 4.0) nach GIP-Standard: jeder Hektometer wird aus der kilometrierten Abschnittsfolge (Tabelle LINK_2_EDGE_SEQUENCE) berechnet und an den Kilometerzeichen (Tabelle REFERENCE_POINT) kalibriert, die Zeichen selbst behalten ihre Lage (`kind` `marker`, berechnete Punkte `kind` `computed`). Autobahnen und Schnellstraßen (A, S) haben in der GIP.at keine Zeichen; dort ist die Abschnittsfolge nur grob an den Nummern der Anschlussstellen-Rampen kalibriert (`accuracy` 300 bis 500 m). Wien hat in der GIP.at keine Kilometrierung. Dieselben Punkte bilden das Tileset `road-km` (`/v1/tiles/road-km`).

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
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/road-km/reverse?lng=15.69646&lat=48.23812&radius=50" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/road-km/reverse?lng=15.69646&lat=48.23812&radius=50", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/road-km/reverse?lng=15.69646&lat=48.23812&radius=50", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "location": [
    15.6965,
    48.2381
  ],
  "radius": 50,
  "results": [
    {
      "road": "B1",
      "km": 60.7,
      "location": [
        15.696458,
        48.23812
      ],
      "distance": 0.8,
      "state": "Niederösterreich",
      "name": "Wiener Straße",
      "section": "gip:244063",
      "bearing": 253,
      "accuracy": 5,
      "source": "gip",
      "exact": true,
      "kind": "marker"
    }
  ],
  "attribution": "mapx.at | GIP.at (CC BY 4.0)"
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

## Straßen mit Kilometerzeichen {#roads-with-kilometre-markers}

`GET /v1/road-km/roads`

Alle Straßen mit Kilometerzeichen und ihren zusammenhängenden Kilometerbereichen je Bundesland (`ranges` [\{state, name, from, to\}], `name` ist der Straßenname in diesem Land oder null; `name` oben ist der erste), sortiert nach Klasse (A, S, B, L) und Nummer; zur Autovervollständigung mit `q` (Anfang der Straßennummer, z. B.

`B1` liefert B1, B10, B100 …). Quelle: die Straßenkilometrierung der GIP.at (offene Verwaltungsdaten, CC BY 4.0) nach GIP-Standard: jeder Hektometer wird aus der kilometrierten Abschnittsfolge (Tabelle LINK_2_EDGE_SEQUENCE) berechnet und an den Kilometerzeichen (Tabelle REFERENCE_POINT) kalibriert, die Zeichen selbst behalten ihre Lage (`kind` `marker`, berechnete Punkte `kind` `computed`). Autobahnen und Schnellstraßen (A, S) haben in der GIP.at keine Zeichen; dort ist die Abschnittsfolge nur grob an den Nummern der Anschlussstellen-Rampen kalibriert (`accuracy` 300 bis 500 m). Wien hat in der GIP.at keine Kilometrierung. Dieselben Punkte bilden das Tileset `road-km` (`/v1/tiles/road-km`).

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
          "name": "Wiener Straße",
          "from": 16.44,
          "to": 77.2
        },
        {
          "state": "Oberösterreich",
          "name": "Wiener Straße",
          "from": 139.95,
          "to": 300.2
        }
      ]
    }
  ],
  "attribution": "mapx.at | GIP.at (CC BY 4.0)"
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
