# Earth Enterprise → God's Eye View Integration

## Scale Context

This repo (Open GEE) operates at the **city/regional scale** — it fuses and serves
custom imagery, terrain, and vector data into private 3D globes.
God's Eye View operates at the **global/real-time scale** — a CesiumJS intelligence
console with live flights, vessels, satellites, fires, and voice.

Together they form two layers of the same stack:

```
God's Eye View   → global real-time intelligence (CesiumJS client, live feeds)
     ↑ tile requests
Open GEE Server  → private tile server (custom imagery, terrain, vector overlays)
     ↑ data fusion
Open GEE Fusion  → ingests local imagery/LiDAR/vector → produces .glb / .glc / .glm
```

## What GEE Provides That GEV Can Consume

| GEE Output | GEV Integration Point |
|---|---|
| Terrain tiles (`.glb` server) | Replace or supplement Cesium World Terrain via a custom `TerrainProvider` |
| Imagery tiles (WMS/WMTS endpoint) | Add as `WebMapTileServiceImageryProvider` in GEV's `mapStackController.js` |
| 2D vector overlays (GeoJSON export) | Register as a new layer module under `src/layers/` |
| Private 3D globe (`.glc`) | Embed inside a Cesium `Viewer` pointing to GEE's server endpoint |

## Connecting GEE Imagery to GEV

In `src/maps/catalog.js` of gods-eye-view, add a new imagery source:

```js
// src/maps/catalog.js — add to the sources array
{
  id: 'gee-local',
  label: 'GEE Custom Imagery',
  imagery: new Cesium.WebMapTileServiceImageryProvider({
    url: 'http://YOUR_GEE_SERVER/cgi-bin/wmts',
    layer: 'your_layer_name',
    style: 'default',
    tileMatrixSetID: 'default028mm',
  }),
}
```

Replace `YOUR_GEE_SERVER` with the hostname of your GEE Server instance
(default port 80, Apache-based).

## GEE Terrain → GEV

GEE serves terrain via its own protocol (not Quantized Mesh). Options:

1. **Export as STL/OBJ** from GEE Fusion → convert to Cesium Quantized Mesh with
   `quantized-mesh-encoder` → serve via a static tile server → set as
   `CesiumTerrainProvider` in GEV's `mapStackController.js`.

2. **Keep Cesium World Terrain** (default, free tier via Cesium ion) for global
   context and overlay only your GEE custom region on top.

## OpenFL + GEE + GEV Together

See `docs/integration/gods-eye-view-integration.md` in `floridaverse/openfl`
for the civic data layer integration. The full stack:

```
GEV globe (global context, real-time feeds)
  └── GEE tile server (private/high-res local imagery)
       └── OpenFL data vaults (live civic signals as GeoJSON layers in GEV)
```

## Branch

Feature branch: `claude/gods-eye-view-setup-547dkz`

All integration work tracked here. No production build steps modified.
