# Immobilien

Grundstücks-Dossier, Sonne, Hitze und Schnee am Standort, statisches Bild fürs Exposé und Erreichbarkeit einer Adresse, mit Code, der unverändert läuft.

Für ein Exposé braucht es drei Dinge: Fakten zum Grundstück, ein Bild, das die Lage zeigt, und eine Antwort auf „Wie gut ist es erreichbar?“. Alles kommt aus amtlichen offenen Daten (BEV-Kataster, Orthophoto, GIP, Fahrpläne) ohne Schlüssel. Die Daten dienen der Orientierung und sind rechtlich nicht verbindlich.

## Grundstück finden

Nach der Grundstücksnummer suchen (`KG 14105 Gst 102/3`, `14105/102/3`) oder das Grundstück an einem Punkt nehmen, etwa aus einem Klick oder einer Adresse:

```bash
curl -s "https://api.mapx.at/v1/geocode?q=Herrengasse%2013%20Wien&types=address&limit=1" | jq '.features[0].center'
curl -s "https://api.mapx.at/v1/parcel/15.3294,48.2272" | jq '.features[0].properties | {parcel_id, parcel_number, area_m2, is_building_land}'
curl -s "https://api.mapx.at/v1/geocode?q=KG%2014105%20Gst%20102/3&types=parcel&limit=1" | jq '.features[0].properties.parcel_id'
```

## Dossier lesen

`/v1/parcels/{id}` liefert alles, was die offenen Daten zu einem Grundstück sagen, in einem Aufruf: Gemeinde und Bezirk, Fläche, Nutzungsanteile, Gebäude mit Höhen (wo das BEV sie hat), Geländestatistik, Adressen, die nächste Haltestelle, das Flugjahr des Orthophotos und einen fertigen Bildlink.

```js
const id = "14143-473_1"; // properties.parcel_id aus der Suche oder der Punktabfrage
const d = await (await fetch(`https://api.mapx.at/v1/parcels/${id}`)).json();
const p = d.properties;
console.log(
  `${p.parcel_number} in ${p.cadastral_municipality.name}, ${Math.round(p.area_m2)} m²`,
  `| Gebäude: ${p.buildings.count}`,
  `| ${p.elevation?.min_m}-${p.elevation?.max_m} m, Neigung ${p.elevation?.slope_mean_deg}°`,
  `| nächste Haltestelle: ${p.nearest_transit_stop?.name} (${p.nearest_transit_stop?.distance_m} m)`,
);
```

Fehlende Daten sind `null` oder leere Listen, nie abwesend. Mit dem [SDK](/de/guides/sdk/helpers/): `mapx.parcelDossier(id)` oder `mapx.parcel(point, { dossier: true })`. Für einen Kartenklick `new mapx.ParcelPopup(map)`.

## Ein Bild fürs Exposé

Das Dossier enthält `links.static_image`: ein Luftbild mit dem Grundstücksumriss. Für ein eigenes Bild das Polygon als GeoJSON-Overlay zeichnen und mit `auto` den Ausschnitt anpassen lassen:

```js
const parcel = (await (await fetch("https://api.mapx.at/v1/parcel/15.3294,48.2272")).json()).features[0];
const outline = { type: "Feature", properties: { stroke: "#e11d48", "stroke-width": 3, fill: "#e11d48", "fill-opacity": 0.2 }, geometry: parcel.geometry };
const img = `https://api.mapx.at/v1/static/satellite/geojson(${encodeURIComponent(JSON.stringify(outline))})/auto/800x500.webp`;
document.querySelector("#expose").src = img; // geht auch in E-Mails und PDFs
```

Für eine Planansicht `outdoors` oder `streets` statt `satellite` verwenden. `@2x` an der Größe liefert scharfe Bilder auf hochauflösenden Bildschirmen. Siehe [Statische Bilder](/de/guides/static-images/). Das Bild trägt die Attribution seiner Daten; im Layout sichtbar lassen ([Attribution](/de/reference/attribution/)).

## Sonne, Hitze und Schnee am Standort

Käufer fragen nach Solarertrag, Sommerhitze und Schneelast. `/v1/climate/{lon},{lat}` antwortet aus Daten der GeoSphere Austria; dasselbe Objekt steht als `properties.climate` im Dossier (am inneren Punkt des Grundstücks). Jeder Abschnitt nennt Datensatz, Rasterzelle und letzten Datentag.

```js
const c = await (await fetch("https://api.mapx.at/v1/climate/15.3294,48.2272")).json();
console.log(
  `Globalstrahlung: ${c.solar?.global_radiation_kwh_m2_year} kWh/m² pro Jahr, ${c.solar?.sunshine_hours_year} h Sonnenschein`,
  `| Jahresmittel ${c.temperature.mean_annual_c.normal} °C (1991-2020), Trend +${c.temperature.trend_c_per_decade} °C je Jahrzehnt`,
  `| Hitzetage (TX ≥ 30 °C): ${c.temperature_extremes.hot_days_per_year.normal} → ${c.temperature_extremes.hot_days_per_year.recent} pro Jahr`,
  `| Schneetage: ${c.snow.snow_days_per_winter.normal} pro Winter, jetzt ${c.snow.current.depth_cm} cm`,
  `| Dürre (SPEI 365): ${c.drought.spei365.class}`,
);
```

Eine grobe Photovoltaik-Abschätzung ist die Strahlung auf dem Dach mal Modulwirkungsgrad und Systemwirkungsgrad (etwa 0,8); der Wert dient der Orientierung und ersetzt keine Ertragsprognose. Die erste Anfrage für ein Gebiet, nach dem noch niemand gefragt hat, kann bis zu 20 Sekunden dauern, spätere kommen aus dem Cache. Im Dossier ist der Abschnitt `null` mit `status: "partial"`, bis die Daten geladen sind; nach `retry_after_s` erneut fragen. Referenz: [Climate API](/de/api/climate/). Den Quellenhinweis anzeigen: GeoSphere Austria, CC BY 4.0 ([Quellenangaben](/de/reference/attribution/)).

## Wie gut ist es erreichbar

Die Fläche zeigen, die zu Fuß oder mit dem Rad erreichbar ist, und die Entfernung zu wichtigen Orten.

```js
const here = [15.3294, 48.2272];
const iso = await (await fetch(`https://api.mapx.at/v1/isochrone/walking/${here}?contours_minutes=5,10,15`)).json(); // GeoJSON-Flächen

const places = [here, [15.3333, 48.2279], [15.3201, 48.2218]]; // Adresse, Schule, Bahnhof
const m = await (await fetch(`https://api.mapx.at/v1/matrix/walking/${places.map((p) => p.join(",")).join(";")}?sources=0`)).json();
console.log(m.durations[0].map((s) => Math.round(s / 60) + " min"));
```

Für den öffentlichen Verkehr die nächste Haltestelle aus dem Dossier und ihre [Abfahrten](/de/guides/transit/) nehmen: `GET /v1/departures?stop=<Haltestellen-ID>&limit=5`.

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/departures?stop=at:49:1320&limit=3" | jq '.departures[] | {time, line: .line.name, headsign}'
```

</TabItem>
<TabItem label="Python">

```python
r = requests.get("https://api.mapx.at/v1/departures", params={"stop": "at:49:1320", "limit": 3}).json()
print([(d["time"][11:16], d["line"]["name"]) for d in r["departures"]])
```

</TabItem>
</Tabs>

## Nächste Schritte

- [Grundstücke](/de/guides/parcels/) und das Beispiel [Grundstück per Klick](/de/examples/parcel-on-click/)
- [Statische Bilder](/de/guides/static-images/) mit Markern und Pfaden
- [Navigation](/de/guides/navigation/) für Isochronen und Matrizen