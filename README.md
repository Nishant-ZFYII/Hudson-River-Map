# Hudson River Park Esplanade — full LiDAR map

Interactive point cloud of the Pier 40 to Pier 25 corridor, surveyed by ASAS Labs.
Open `index.html`.

**102,686,809 points — the complete survey map, nothing decimated.**

Potree streams an octree with level of detail, so only what is on screen is
fetched; the whole map opens in seconds and stays smooth while you orbit.

The map is split into ten corridor segments under `pointclouds/`, each its own
octree, because a single `octree.bin` for the whole map is 645 MB and GitHub
rejects any file over 100 MB. The largest file here is 68 MB. Potree renders any
number of point clouds in one scene, so the split is invisible once loaded, and
each segment streams independently. `manifest.json` lists them.

Controls: preset views for the whole corridor and each pier; colour by surface
reflectivity, height, or raw reflectivity; a detail slider driving the point
budget; eye-dome shading; and Potree's own measurement tools.

Requires a host that serves HTTP range requests. GitHub Pages does. Python's
stock `http.server` does not — use `python -m RangeHTTPServer` locally.

## Vendored Potree patches (reapply if Potree is re-vendored)

1. **`build/potree/potree.js` — removed `'content-type': 'multipart/byteranges'`**
   from both `fetch()` calls in `OctreeLoader` (hierarchy and octree range
   requests). GitHub Pages returns **400** to any request carrying that header,
   which made every `hierarchy.bin` load fail with
   `RangeError: Invalid array length`. The header is meaningless on a GET.

2. **`libs/Cesium/` deleted.** It contains a Mapbox secret token that GitHub
   push protection (GH013) rejects. Potree does not use it.

3. **Cache-bust by FILENAME, never by `?query=`.** Potree derives
   `scriptPath = new URL(document.currentScript.src + '/..')`. A query string
   makes everything after `?` part of the query, so worker and resource URLs
   become `potree.js?v=.../../workers/...` — the browser then loads `potree.js`
   itself as the web worker and it dies on `ReferenceError: document is not
   defined`. Renaming to `potree.<hash>.js` keeps the directory, and so
   `scriptPath`, intact.

4. **`basemap.jpg` must be ≤ 4096 px** on its long edge or the GPU silently
   downsamples it (`THREE.WebGLRenderer: Texture has been resized`).
