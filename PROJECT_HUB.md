# Silicon Stack — Project Hub

**Last updated:** 2026-04-29
**Dataset version:** 0.6.2 (render-ready)
**Frontend version:** v1 (single-file React + D3 artifact, compile-clean)

This is the single source of truth for the project. Everything below should let a fresh session (Claude Code or otherwise) pick up exactly where this one left off.

---

## 1. The vision

An interactive web app visualizing the semiconductor industry, two pages:

- **Page 1 — Materials.** Raw-materials suppliers as bubbles on a stylized world map, sized by importance, lines flowing to their consumers (foundries, IDMs, memory cos).
- **Page 2 — Foundry & Design.** Force-directed graph of foundries + equipment makers (diamonds) + design (EDA/IP) → customers (fabless, OEMs, OSATs, etc).

Dropped earlier: a third page. Two pages, both deep.

Patient, depth-not-breadth. International scope is fine. ~240 companies, ~1,100 relationships is the right size.

---

## 2. Dataset — v0.6.2 final

**File:** `semicon_supply_chain.json`

| Metric | Count |
|---|---|
| Companies | 239 |
| Materials | 115 |
| Relationships | 1,110 |
| Orphan edges | 0 |
| Geo clusters | 69 (206 in metros, 33 country-fallback) |
| Single-source-risk flags | 16 |
| Geographic-risk flags | 9 |
| Export-control-exposed flags | 10 |

**Edge strength distribution:** 834 primary · 267 secondary · 8 estimated · 1 indirect.

**Top centrality (most connected):** TSMC 161 · Samsung Foundry 97 · Intel 86 · Synopsys 84 · Cadence 78 · SK hynix 74.

**Node types (10):**
- foundry (11), idm (10), memory (8), fabless (17), oem (5)
- osat (10), equipment (52), eda (5), ip (6), supplier (115)

**Pages:** page=1 → 115 cos · page=2 → 96 cos · page='both' → 28 cos.

### Schema

```jsonc
{
  "metadata": { "version": "0.6.2", "last_updated": "2026-04-28", ... },
  "materials": [
    {
      "id": "mat_polysilicon_eg",
      "name": "Electronic-Grade Polysilicon",
      "category": "feedstock",
      "process_stage": "pre_wafer",
      "criticality": "high"
    }
  ],
  "companies": [
    {
      "id": "co_tsmc",
      "node_type": "foundry",
      "page": 2,                    // 1 | 2 | "both"
      "name": "TSMC",
      "country": "Taiwan",
      "hq_city": "Hsinchu",
      "lat": 24.78, "lng": 120.99,
      "geo_cluster_id": "metro_hsinchu",
      "tier": 1,
      "approx_revenue_usd_b": 89,
      "scale_indicator": 100,        // 0–100, drives node size
      "market_positions": ["Leading-edge logic foundry, ~60% global share"],
      "single_source_risk": null,
      "geographic_risk": "Taiwan concentration",
      "export_control_exposed": true
    }
  ],
  "relationships": [
    {
      "id": "r001",
      "supplier_id": "co_shinetsu",
      "customer_id": "co_tsmc",
      "material_id": "mat_wafer_300mm",
      "strength": "primary",         // primary | secondary | estimated | indirect
      "revenue_weight_pct_2024": 22,    // optional
      "revenue_weight_pct_2026e": 25,   // optional
      "revenue_weight_pct_hbm_2025": 50,// optional (memory)
      "cowos_2026_pct": 60              // optional (TSMC CoWoS allocation)
    }
  ]
}
```

### Process stages (8, drive edge color)
`pre_wafer · front_end · fab · fab_consumable · fab_equipment · back_end · mask_shop · design`

### Cleanup decisions baked into v0.6.2
Don't re-introduce these in future expansions:
1. **Sony Semi → Samsung Foundry CIS edge removed** (was wrong direction; Sony is a CIS competitor of Samsung, not a supplier).
2. **Ceva → Qualcomm DSP IP downgraded to `estimated`** (Qualcomm uses its own Hexagon DSP).
3. **5 unused materials dropped:** BT Substrate, Foundry Service Mature, Advanced Packaging service token, Ultra-high-density PCB, TSV Equipment.
4. **Mitsubishi Materials scale_indicator filled** (was null, now 35).
5. **`geo_cluster_id` added on every company** — 69 clusters, metro-area where possible (`metro_tokyo` covers Greater Kanto; 51 cos), country-fallback for genuinely scattered ones (e.g. US chemicals).

---

## 3. Tech stack — committed

- **React** + hooks (no Redux, no Zustand — plain `useState`/`useReducer`)
- **D3** (`d3-force` for Page 2, `d3-geo` patterns for Page 1, `d3-zoom`, `d3-drag`, `d3-selection`)
- **Tailwind** for chrome (utilities only — no plugins)
- **SVG** render target (~240 nodes / ~1100 edges fits comfortably; no Canvas needed yet)
- **Vite** for the production build (chat artifact is a single `.jsx` file with inline data; port to Vite project for `npm run dev`)
- No backend. All data inlined as a parsed JSON string at module load.

### Why these
- Force layout + selection state ergonomics → React.
- Force layout itself + tick perf → D3 manages node/link DOM directly inside React refs (the standard pattern; React renders the shell, D3 owns the simulation-driven attrs).
- Tailwind for the dashboard chrome only; viz uses inline `style` with CSS vars.

---

## 4. Aesthetic direction — committed

**"Industrial blueprint × Bloomberg terminal."** Warm-dark, monospace for data, sharp lines, no gratuitous rounding.

### Typography
- **Display:** Fraunces (variable serif from Google Fonts) — for headings and the few "moments." Distinctive, not generic.
- **UI/body:** IBM Plex Sans
- **Data/labels:** IBM Plex Mono

### Color palette (CSS vars)

```css
--bg:             #0e0c0a;
--surface:        #16130f;
--surface-2:      #1f1a14;
--border:         #2a231b;
--border-bright:  #3d3326;
--text:           #ebe2d3;
--text-mute:      #8c8175;
--text-dim:       #5a5249;
```

### Node-type colors
```
foundry    #7aa3d4   circle
idm        #c47860   square
memory     #4f9b95   rsquare
fabless    #6cb5d4   circle
oem        #d48f5a   rsquare
osat       #9ba85a   trapezoid
equipment  #d4a85a   diamond     ← visually distinctive
eda        #a07cc4   hexagon
ip         #7a96aa   hexagon
supplier   #a89886   circle
```

### Risk colors
```
single-source   #d44a4a
geographic      #d4a85a
export-control  #d47545
```

### Process-stage edge colors
```
pre_wafer       #a89886
front_end       #7aa3d4
fab             #7aa3d4
fab_consumable  #9ba85a
fab_equipment   #d4a85a
back_end        #c47860
mask_shop       #a07cc4
design          #7a96aa
```

---

## 5. Frontend architecture — current build

**File:** `semicon_app.jsx` (single-file artifact, 264 KB / ~1,000 lines, 220 KB of which is inline data)

### Components
1. **`shapePath(shape, r)`** — SVG path generator: circle, square, rsquare, diamond, hex, trap.
2. **`useFonts()`** — injects Google Fonts `<link>` once.
3. **`clusterCentroids(companies)`** — geo-cluster centroid helper for Page 1.
4. **`Header`** — title, version, page toggle (01 / 02), node+edge totals.
5. **`FilterSidebar`** — Showing-counter, edge-strength checkboxes, node-type checkboxes (with shape glyphs), risk-highlight toggles.
6. **`Page2Force`** — force-directed graph. **D3-managed DOM** (React renders only the SVG shell + two empty `<g>` containers; D3 does enter/update/exit on nodes & links and runs the simulation). Two `useEffect`s:
   - One builds the simulation (deps: nodes, links, dim).
   - One does cosmetic updates for selected/hovered/risk highlighting (no sim restart).
   - Refs (`filterRef`, `selectedRef`, `setSelectedRef`, `setHoveredRef`) capture latest values for D3 event callbacks (which would otherwise close over stale state).
7. **`Page1Geo`** — equirectangular projection: `lng [-180, 180] → [PAD, w-PAD]`, `lat [-65, 75] → [h-PAD, PAD]`, `PAD=60`. Cluster bubbles at centroids, sized by Σ member `scale_indicator`, colored by dominant `node_type`. Members fanned around the centroid with thin spokes. Cluster→cluster edges as faint quadratic Bézier curves colored by process stage. No literal world borders (aesthetic choice).
8. **`NodeDetail`** — slide-in right panel: header with node-type label and city/country, risk badges, profile (revenue/tier/scale/page), market positions, outgoing edges (with revenue weights for top relationships), incoming edges.
9. **`Legend`** — process-stage color legend pinned to the bottom.
10. **`App`** — orchestration: page state, selected/hovered, filters, derived stats via `useMemo`.

### Force-layout config (Page 2)
- **xBias** by node "stage": eda/ip −0.92 → equipment/supplier −0.55 → foundry 0 → idm 0.18 → memory 0.35 → osat 0.55 → fabless 0.85 → oem 1.05. Reads design-on-the-left, customers-on-the-right.
- Link distance: 60 (primary) / 95 (secondary)
- Charge: −110, distanceMax 420
- Collide: nodeR + 4
- x-force strength 0.20, y-force strength 0.05
- alphaDecay 0.025
- Zoom: scaleExtent [0.25, 5]
- Drag fixes node on start, releases on end

### Page 1 geo config
- PAD = 60
- 6 region labels (North America, Europe, East Asia, South Asia, SE Asia, Oceania)
- Cluster bubble radius: `3 + sqrt(total_scale) * 0.7`
- Member fan radius: `cluster_radius + 14`

---

## 6. Known issues & deferred work

### Layout/UX polish
- [ ] Tokyo cluster (51 cos) gets crowded in member fan — should collapse to bubble, expand on click.
- [ ] No real world coastlines on Page 1 (deliberate, but worth offering as toggle).
- [ ] Selected-node detail panel could be wider on big screens.
- [ ] First-load force layout takes ~2s to settle. Could pre-position by xBias to converge faster.
- [ ] Zoom-to-fit / "reset view" button would be nice.
- [ ] Search/jump-to-node would be nice.

### Data gaps (acknowledged, not blockers)
- Western Digital / SanDisk has limited inbound edges; modeled mostly via Kioxia JV.
- Customer revenue weights complete only for TSMC; Samsung Foundry & Intel Foundry partial public disclosure.
- Some Chinese equipment makers' customer mix is opaque; modeled as `strength: estimated`.
- Passive components (MLCC: Murata, TDK, SEMCO) and AI networking optics (Coherent, Lumentum) out of scope.
- Sample-size error rate ~5–15% is plausible; not authoritative without further audit.

### Artifact constraints (carry over to local dev only if relevant)
- Artifacts environment can't use localStorage.
- Tailwind core utilities only — no plugins, no custom config.
- D3 available but not topojson-client.
- No fetch from arbitrary URLs.

These are artifact-only limits; in a Vite project locally, none of them apply.

---

## 7. File manifest

Everything is in `/mnt/user-data/outputs/` (download these to your repo):

| File | Purpose |
|---|---|
| `semicon_supply_chain.json` | The dataset, v0.6.2. Source of truth. |
| `semicon_app.jsx` | Single-file React+D3 artifact (with inline data). Drop-in artifact, also the reference impl. |
| `README.md` | Schema documentation, changelog. |
| `PROJECT_HUB.md` | This file. |
| `cleanup_v062.py` | First cleanup pass (edge fixes, scale fill, material drops, initial geo clusters). |
| `cleanup_v062b.py` | Geo-cluster bbox refinement (widened metros to catch satellites). |
| `diagnostics.py` | Render-readiness diagnostics (degree distribution, scale histogram, schema completeness, bad-edge detection). |
| `expand_pass*.py` | Historical dataset expansion scripts (kept for traceability). |
| `patch_risk_flags.py` | Risk-flag application script. |

In `/home/claude/frontend/` (in this session's filesystem) there's also:
- `component_template.jsx` — JSX template with `__DATA_PLACEHOLDER__` token.
- The build pattern: `template.replace('__DATA_PLACEHOLDER__', escaped_minified_json)` produces the final artifact.

---

## 8. Recommended Vite project layout for local dev

```
silicon-stack/
├── package.json
├── vite.config.js
├── tailwind.config.js
├── postcss.config.js
├── index.html
├── src/
│   ├── main.jsx              # ReactDOM root + import App
│   ├── App.jsx               # the orchestration component (from semicon_app.jsx)
│   ├── components/
│   │   ├── Header.jsx
│   │   ├── FilterSidebar.jsx
│   │   ├── Page2Force.jsx    # the heaviest component; keep isolated
│   │   ├── Page1Geo.jsx
│   │   ├── NodeDetail.jsx
│   │   └── Legend.jsx
│   ├── lib/
│   │   ├── shapes.js         # shapePath
│   │   ├── constants.js      # NODE_TYPES, color maps, etc.
│   │   └── geo.js            # clusterCentroids, projection helpers
│   ├── data/
│   │   └── semicon_supply_chain.json   # imported normally, no inline-string trick needed
│   └── styles/
│       └── tokens.css        # CSS variable definitions
└── public/
    └── favicon.svg
```

### Dependencies
```json
{
  "dependencies": {
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "d3": "^7.9.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.3.0",
    "vite": "^5.4.0",
    "tailwindcss": "^3.4.0",
    "postcss": "^8.4.0",
    "autoprefixer": "^10.4.0"
  }
}
```

### Quick scaffold

```bash
npm create vite@latest silicon-stack -- --template react
cd silicon-stack
npm install
npm install d3
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
# then: copy semicon_app.jsx contents into src/, split components, drop JSON into src/data/
```

Once the JSON is imported normally (`import data from './data/semicon_supply_chain.json'`), the inline-string + JSON.parse hack from the artifact goes away. That's the single biggest simplification when porting.

---

## 9. Next-step priorities (rough order)

1. **Port to Vite project** — split the single file into the layout above. Replace the inline `DATA_RAW` template literal with a normal JSON import.
2. **Cluster-collapse interaction on Page 1** — Tokyo's 51-member fan is the main UX wart. Click cluster → expand into member fan, click again → collapse. Use Framer Motion or just CSS transitions on `transform`.
3. **Search/jump-to-node** — ⌘K palette over `companies[]`. Cheap win.
4. **Reset-view button** for Page 2 — clears any zoom transform and re-runs sim with `alpha(0.3)`.
5. **Pre-position Page 2 nodes by xBias** before sim starts (so it doesn't lurch from center). `nodes.forEach(n => { n.x = w/2 + xBias(n)*w*0.4; n.y = h/2 + (Math.random()-0.5)*h*0.6; })` before `forceSimulation(nodes)`.
6. **World coastline toggle on Page 1** — fetch `world-110m.json` from a CDN (or check it in), use `d3-geo`'s `geoEquirectangular`. Default off, toggle in sidebar.
7. **Detail panel polish** — wider on big screens, group outgoing edges by customer (TSMC's view of "what Apple buys from us via this whole supply chain" is currently 1 row per material; should aggregate).
8. **Performance** — if FPS gets bad after adding more nodes, switch Page 2 to Canvas via `d3-canvas` or migrate to PixiJS.

---

## 10. Quick handoff notes for Claude Code

When you open this in Claude Code:
- The dataset (`semicon_supply_chain.json`) is the source of truth. **Don't re-derive it.** The cleanup decisions in §2 are intentional.
- The artifact (`semicon_app.jsx`) is a working reference implementation. It compiles. It's a starting point, not a final form.
- The aesthetic direction in §4 is committed. Stick to the palette, the fonts, the shape vocabulary. Don't drift toward Material Design or shadcn defaults.
- The force-layout config in §5 is the result of tuning. If layout feels wrong, adjust those constants — but write down what you change so it doesn't drift.
- Page 1 is intentionally stylized over realistic. World borders were considered and deliberately omitted; the region labels do the orientation work.
- If you find a wrong edge in the data, fix it in `semicon_supply_chain.json`, log the fix in §2, and re-run any diagnostics. Don't patch around it in the frontend.

The hard parts are done: dataset is clean, schema is stable, aesthetic direction is committed, the React+D3 integration pattern works. Most remaining work is polish and porting.
