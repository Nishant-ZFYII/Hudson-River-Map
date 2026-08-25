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
