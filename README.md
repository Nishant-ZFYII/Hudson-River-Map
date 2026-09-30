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

## Update — why the viewer broke after publishing (Sept 2026)

The map rendered locally but showed nothing on GitHub Pages, with
`RangeError: Invalid array length` in `OctreeLoader.parseHierarchy`.

**This was not caused by a vendor patch.** Both causes are long-standing
behaviours that only collide when Potree 2.0 octrees are hosted on GitHub
Pages. The Potree checkout is 1.8.0 (HEAD 5636cd4, 2026-01-08) and the
offending line is original upstream source, not a recent change. Nothing in
the Autoware/mapping pipeline was involved — the point cloud was always
correct; only its delivery over HTTP was broken.

**Cause 1 — a bad request header.** `OctreeLoader` sends
`content-type: multipart/byteranges` on its range requests. The header is
meaningless on a GET (content-type describes a request body; a GET has none)
and GitHub Pages returns **400** to any request carrying it. Fixed by removing
it from both `fetch()` calls.

**Cause 2 — GitHub Pages gzips the octree and breaks byte ranges.** Pages
compresses `application/octet-stream` and applies `Range` to the *compressed*
entity, so Potree's byte offsets index into a gzip stream. Measured on this
repo: a request for bytes 0–3321 of `hierarchy.bin` returned **7,372 bytes**
instead of 3,322, and since `parseHierarchy` computes `new Array(len / 22)`,
`7372 % 22 = 2` produced a fractional length and threw. Pages does **not**
compress `image/*`, so the payloads are now served as `hierarchy.bin.png` and
`octree.bin.png` — contents byte-for-byte unchanged, the extension exists only
to opt out of compression — with the two URLs in `potree.js` updated to match.

This is invisible to `curl` unless you pass `--compressed`; a plain `curl -r`
returns a correct `206`, which is why a preflight check looks clean and only a
real browser fails.
