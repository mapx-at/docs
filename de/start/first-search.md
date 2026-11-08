# Erste Suche

Eine Adresse oder einen Ort mit einer Anfrage finden und das Ergebnis auf einer Karte zeigen.

## Anfrage

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/geocode?q=Stephansplatz%20Wien&limit=1" | jq '.features[0] | {place_name, center}'
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/geocode?q=Stephansplatz%20Wien&limit=1", { headers: { Authorization: "Bearer DEIN_TOKEN" } });
const { features } = await res.json();
console.log(features[0].place_name, features[0].center); // center ist [lon, lat]
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/geocode", params={"q": "Stephansplatz Wien", "limit": 1}, headers={"Authorization": "Bearer DEIN_TOKEN"})
feature = res.json()["features"][0]
print(feature["place_name"], feature["center"])
```

</TabItem>
</Tabs>

Die Antwort ist eine GeoJSON-`FeatureCollection`. Jedes Feature hat `place_name`, `center` (`lon,lat`) und `geometry`. Mit `types=address` lassen sich die Ergebnisse auf Adressen beschränken.

## Ergebnis auf der Karte zeigen

Diesen Code auf der Seite aus [Erste Karte](/de/start/first-map/) nach `new maplibregl.Map(...)` einfügen (die Karte in `const map` speichern):

```js
const res = await fetch("https://api.mapx.at/v1/geocode?q=Stephansplatz%20Wien&limit=1", { headers: { Authorization: "Bearer DEIN_TOKEN" } });
const [hit] = (await res.json()).features;
new maplibregl.Marker().setLngLat(hit.center).addTo(map);
map.flyTo({ center: hit.center, zoom: 16 });
```

Den Code in ein `<script type="module">` setzen, damit Top-Level-`await` funktioniert.

## Nächste Schritte

- [Such-Guide](/de/guides/search/): Type-ahead, Kategorien, Adressformen
- [Geocoding API](/de/api/geocoding/)