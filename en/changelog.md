# Changelog

Changes to data products, APIs and these docs.

## 07.11.2026

- New: data products DTM, DSM, contour lines, base map, cadastre, orthophoto and flight blocks, each with a live map and layer table.
- New: API with tiles (PMTiles and XYZ), TileJSON, elevation and profile, and a test console.
- New: documentation in German and English.

## Data releases

<table>
  <thead><tr><th>Product</th><th>Reference date</th><th>File</th><th>Created</th></tr></thead>
  <tbody>
    {Object.values(data.tilesets).filter((x) => ["relief", "dsm"].includes(x.key)).map((x) => (
      <tr><td>{x.tileset}</td><td>{x.stand}</td><td><code>{x.datei}</code></td><td>{x.erzeugt ?? ""}</td></tr>
    ))}
  </tbody>
</table>