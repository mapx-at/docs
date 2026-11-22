# Elevation profile chart in JavaScript

Source: https://docs.mapx.at/guides/elevation-profile/

> Query an elevation profile for a route through the elevation API and draw it as an SVG chart, in plain JavaScript.

**Goal:** Query an elevation profile (distance against elevation) for a route and draw it as a chart.

**Prerequisites:** a browser. A free token ([Access and limits](https://docs.mapx.at/start/access-and-limits.md)), no framework, no library.

With `path` and `samples`, the [`/v1/elevation`](https://docs.mapx.at/api/elevation.md) endpoint returns elevation values spaced evenly by length along the line. If you only need the finished totals (ascent, descent, lowest and highest point), use [`/v1/elevation/profile`](https://docs.mapx.at/guides/elevation.md#elevation-profile); here the profile is computed and drawn by hand.

1. **Test the request.** The line is made of `lon,lat` pairs separated by semicolons (longitude first). 50 sample points along a route at the Großglockner:

   ```bash
   curl -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/elevation?path=12.69,47.07;12.70,47.08&samples=50"
   ```

   The response is a GeoJSON `FeatureCollection`, one feature per sample with `elevation` (m) and `distance` (distance from the start).

2. **Create the page.** A file `profile.html` with an empty SVG:

   ```html
   <!doctype html>
   <meta charset="utf-8">
   <svg id="profile" viewBox="0 0 600 240" style="width:100%;max-width:720px;font:12px sans-serif"></svg>
   <p id="info"></p>
   ```

3. **Load and draw the profile.** The script fetches the data, scales distance and elevation to the canvas and builds a line with a filled area below:

   ```html
   <script type="module">
     const line = "12.69,47.07;12.70,47.08";
     const url = `https://api.mapx.at/v1/elevation?path=${line}&samples=100&model=dtm`;
     const data = await (await fetch(url, { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json();

     // skip samples without elevation (outside the data)
     const pts = data.features
       .map(f => ({ d: f.properties.distance, h: f.properties.elevation }))
       .filter(p => p.h !== null);

     const W = 600, H = 240, M = { l: 50, r: 10, t: 10, b: 30 };
     const dMax = pts.at(-1).d;
     const hMin = Math.min(...pts.map(p => p.h)), hMax = Math.max(...pts.map(p => p.h));
     const x = d => M.l + (d / dMax) * (W - M.l - M.r);
     const y = h => H - M.b - ((h - hMin) / (hMax - hMin || 1)) * (H - M.t - M.b);

     const lineD = pts.map((p, i) => `${i ? "L" : "M"}${x(p.d).toFixed(1)},${y(p.h).toFixed(1)}`).join(" ");
     const areaD = `${lineD} L${x(dMax)},${H - M.b} L${x(0)},${H - M.b} Z`;

     document.getElementById("profile").innerHTML = `
       <path d="${areaD}" fill="#cfe3cf"/>
       <path d="${lineD}" fill="none" stroke="#2f6b2f" stroke-width="2"/>
       <line x1="${M.l}" y1="${H - M.b}" x2="${W - M.r}" y2="${H - M.b}" stroke="#888"/>
       <line x1="${M.l}" y1="${M.t}" x2="${M.l}" y2="${H - M.b}" stroke="#888"/>
       <text x="${M.l - 6}" y="${y(hMax) + 4}" text-anchor="end">${Math.round(hMax)} m</text>
       <text x="${M.l - 6}" y="${y(hMin) + 4}" text-anchor="end">${Math.round(hMin)} m</text>
       <text x="${M.l}" y="${H - 10}">0 km</text>
       <text x="${W - M.r}" y="${H - 10}" text-anchor="end">${(dMax / 1000).toFixed(1)} km</text>`;

     // ascent and descent from the samples
     let up = 0, down = 0;
     for (let i = 1; i < pts.length; i++) {
       const dh = pts[i].h - pts[i - 1].h;
       if (dh > 0) up += dh; else down -= dh;
     }
     document.getElementById("info").textContent =
       `Length ${(dMax / 1000).toFixed(2)} km, ascent ${Math.round(up)} m, descent ${Math.round(down)} m`;
   </script>
   ```

4. **Open it.** Open `profile.html` through a static server (`npx serve`); the browser may call the API directly. For web pages use a public token (`mx_pk_…`) with your own origin; allow `localhost` only during development.

## Result

An SVG chart with area and line of elevation over distance, with length, ascent and descent below.

> The simple sum in this example grows with `samples`, because it adds up terrain noise. [`/v1/elevation/profile`](https://docs.mapx.at/api/elevation.md) counts ascent and descent with a 3 m hysteresis instead, which hardly depends on `samples`; use its `ascent_m` and `descent_m` when you show totals. With `model=dsm` the profile follows the surface (trees, buildings) instead of the ground, see [DTM and DSM](https://docs.mapx.at/start/faq.md#what-is-the-difference-between-dtm-and-dsm). A path has 2 to 512 vertices, `samples` 2 to 1024.

## Next steps

- All parameters and response fields: [Elevation and profile](https://docs.mapx.at/api/elevation.md).
- Line from a GPX track: join the coordinates as `lon,lat;lon,lat`, order see [Coordinates](https://docs.mapx.at/reference/coordinates.md).
- Limits for many requests: [Errors and limits](https://docs.mapx.at/api/errors.md).
