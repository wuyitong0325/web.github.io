flowchart TD
  A[Frontend map page<br/>Build request URL<br/><code>/api/points?category=Theft&dateFrom=2025-01-01&bbox=...</code>] 
    --> B[Express server<br/>Match route<br/><code>GET /api/points</code>]

  B --> C[Read <code>req.query</code><br/>Parse query params:<br/><code>category / primaryType / dateFrom / dateTo / bbox / limit / offset</code>]

  C --> D[<code>normalizePrimaryType()</code><br/>Map UI category to<br/>database <code>Primary Type</code> value]

  D --> E[<code>buildWhere()</code><br/>Using <code>primaryType</code> + <code>bbox</code><br/>build SQL condition array<br/><code>where[] / params[]</code>]

  E --> F[Compose SQL<br/><code>SELECT ... FROM "TABLE"</code><br/>plus <code>WHERE ...</code> / <code>ORDER BY "Date" DESC</code> / <code>LIMIT ? OFFSET ?</code>]

  F --> G[Run SQLite query<br/><code>db.prepare(baseSQL).all(...params, limit, offset)</code>]

  G --> H[Get raw result rows<br/><code>rows = [{ id, date, primaryType, lat, lng, ... }, ...]</code>]

  H --> I{Any<br/><code>dateFrom</code><br/>or<br/><code>dateTo</code>?}

  I -- No --> J[<code>filtered = rows</code>]
  I -- Yes --> K[In JS, filter rows<br/>with <code>new Date(r.date)</code><br/>keep <code>dateFrom ≤ date ≤ dateTo</code>]

  K --> J

  J --> L[Filter out rows<br/>with invalid coordinates<br/><code>Number.isFinite(lat) && Number.isFinite(lng)</code>]

  L --> M[Map each row to<br/>GeoJSON Feature:<br/><code>{ type: "Feature", geometry: { type: "Point", coordinates: [lng, lat] }, properties: {...} }</code>]

  M --> N[Build GeoJSON<br/><code>{ type: "FeatureCollection", features: [...] }</code>]

  N --> O[Send JSON response<br/><code>res.json(geojson)</code>]

  O --> P[Frontend map library<br/>receives GeoJSON<br/>renders markers / heatmap<br/>and shows crime info]
# web.github.io
