# Klima-API: Sonne, Schnee, Hitzetage

Source: https://docs.mapx.at/de/api/climate/

> Klima-API für einen Punkt in Österreich: Sonnenpotenzial, Klimanormale, Hitzetage, Schnee und Dürre von GeoSphere Austria, mit Parametern und Beispielen.

Klima eines Punktes: Jahressumme der Sonneneinstrahlung und Sonnenstunden, Temperatur- und Niederschlagsnormale mit Trend, Hitze- und Frosttage, Schneetage je Winter und der Dürreindex. Quelle: GeoSphere Austria, CC BY 4.0. Dasselbe Objekt steht als `properties.climate` im [Grundstücks-Dossier](https://docs.mapx.at/de/api/tilequery.md).

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s) · Guide: [Grundstücke](https://docs.mapx.at/de/guides/parcels.md)

## Klima an einem Punkt (Sonnenpotenzial, Klimanormale, Hitzetage, Schnee, Dürre) {#climate-at-a-point-solar-potential-normals-heat-days-snow-drought}

`GET /v1/climate/{coordinate}`

Klima eines Punktes aus offenen Daten von GeoSphere Austria: Sonnenpotenzial (jährliche Globalstrahlung in kWh/m2 und Sonnenstunden, Mittel der letzten 3 vollständigen Jahre, 100 m), Temperatur und Niederschlag (Klimanormale 1991-2020, letzte 10 Jahre, Trend, Monatsnormale, 1 km), Hitze-, Sommer-, Frost- und Eistage sowie Tropennächte pro Jahr, Schneetage pro Winter und die aktuelle Schneehöhe sowie der Dürreindex SPEI (90 und 365 Tage, letzter Wert mit Klasse).

Jeder Abschnitt hat `source` (Datensatz, Zelle, letzter Datentag) und `retrieved_at`. Die Werte werden je Rasterzelle abgeleitet und zwischengespeichert; die erste Anfrage für eine neue Zelle holt sie von der Quelle und kann 10 bis 20 Sekunden dauern (`sections` begrenzt den Aufwand), spätere Anfragen kommen aus dem Cache. Dasselbe Objekt ist Teil des Grundstücksdossiers (`properties.climate`). Außerhalb Österreichs 404; 503, solange die Quelle nicht verfügbar ist oder drosselt. Datenquelle: GeoSphere Austria (APOLIS, SPARTACUS, SNOWGRID-CL, WINFORE), CC BY 4.0 (Quellenangabe erforderlich).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `coordinate` | path | string | ja | – | lon,lat oder lon,lat.json |
| `sections` | query | string |  | `all` | Gewünschte Abschnitte, durch Komma getrennt (Standard: alle) |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/climate/15.3294,48.2272?sections=solar,snow" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/climate/15.3294,48.2272?sections=solar,snow", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/climate/15.3294,48.2272?sections=solar,snow", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "location": [
    15.3294,
    48.2272
  ],
  "status": "complete",
  "solar": {
    "period": "2023-2025",
    "years": 3,
    "global_radiation_kwh_m2_year": 1287.1,
    "global_radiation_horizontal_kwh_m2_year": 1219.1,
    "direct_radiation_kwh_m2_year": 911.6,
    "diffuse_radiation_kwh_m2_year": 375.5,
    "sunshine_hours_year": 2949.5,
    "relative_sunshine_pct": 60.3,
    "by_year": [
      {
        "year": 2025,
        "global_radiation_kwh_m2": 1310.8,
        "sunshine_hours": 2958.4
      }
    ],
    "source": {
      "provider": "GeoSphere Austria",
      "dataset": "apolis-v2-1d-100m",
      "name": "APOLIS solar radiation",
      "license": "CC BY 4.0",
      "resolution_m": 100,
      "cell_center": [
  …
```

| Feld | Typ | Beschreibung |
|---|---|---|
| `location` | array | [Länge, Breite] (WGS84), optional Höhe. |
| `status` | string | `partial`: einige Abschnitte werden noch geladen (Grundstücksdossier: nach `retry_after_s` erneut anfragen) |
| `solar` | object,null | Sonnenpotenzial: Jahressummen, Mittel der letzten 3 vollständigen Jahre (APOLIS, 100 m) |
| `temperature` | object,null | Lufttemperatur (SPARTACUS, 1 km) |
| `precipitation` | object,null | Niederschlag und Sonnenschein (SPARTACUS, 1 km) |
| `temperature_extremes` | object,null | Hitze-, Sommer-, Frost- und Eistage pro Jahr, Tropennächte (SPARTACUS-Tagesdaten, 1 km); Normalperiode 1991-2020 und letzte 10 vollständige Jahre |
| `snow` | object,null | Schneedecke (Modell SNOWGRID-CL, 1 km) |
| `drought` | object,null | Dürreindex SPEI (WINFORE, 1 km), letzter Wert |

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Playground
