---
name: skeptical-osm-map-cards
description: "Build map cards from OpenStreetMap tiles in Python; zoom selection, mosaic math, and the black-strip guard. Use when a video or post needs a location map card and you want it scriptable and free, with no Maps API key."
when-to-use: "A video or post needs a location map card (store opening, event, incident), or you want it scriptable and free with no Maps API key."
---

# OSM tile map cards (pure Python)

## What this does (plain English)

This skill teaches your AI assistant to make a clean location map card for a video or post from free OpenStreetMap tiles, using only Python. It geocodes the place, picks a zoom level that shows readable labels, fetches a 5x5 mosaic of tiles around the center, crops the target area with PIL, and frames it on a card with a pin and label. It also contains the critical guard: PIL's `crop()` silently returns black strips if the crop box runs off the canvas, so the bounds must be asserted every time.

## Responsible use

OpenStreetMap's Nominatim and tile servers are shared services. Respect their usage policies: about 1 request per second, a real user agent, cache tiles to disk, and don't hammer them (use a proper tile provider or self-host for bulk jobs). Map data must be credited to OpenStreetMap contributors on your card or post (see OpenStreetMap's attribution guidelines for the exact wording).

## Install what you need

You need Python 3 (from python.org or your system package manager) with Pillow (PIL), and `curl` for the geocode step (included on macOS, current Windows 10 and 11, and most Linux systems).

```
python3 -m pip install Pillow
```

## Worked example

Make one map card, using only the steps and formulas from the recipe below.

1. Geocode the target (replace `<your app UA>` and `<query>`; respect the 1 req/s policy):

```
curl -s -A "<your app UA>" "https://nominatim.openstreetmap.org/search?q=<query>&format=json"
```

2. Pick the zoom by feature size: z13 for a roughly 4km district area with readable labels, z15-16 for a single block, z11-12 for metro context.
3. Compute the center tile and fetch a 5x5 grid around it (1280px at 256px tiles), then paste each tile into a mosaic canvas at `((tx-x0)*256, (ty-y0)*256)`:

```
x = floor((lon+180)/360 * 2^z)
y = floor((1 - ln(tan(lat_rad) + 1/cos(lat_rad))/pi)/2 * 2^z)
URL: https://tile.openstreetmap.org/{z}/{x}/{y}.png
```

4. Before cropping, assert the bounds (the black-strip guard):

```python
assert 0 <= left and left + CROP_W <= mosaic_w
assert 0 <= top and top + CROP_H <= mosaic_h
```

5. Paste the crop onto a card with a white border, add a pin at the exact center and a location label, then visually check the corners of the rendered card.

---

The full production recipe follows.


## When to Use
- A video/post needs a location map card (store opening, event, incident)
- You want it scriptable and free, no Maps API key

## Recipe

1. GEOCODE the target (Nominatim): 

   ```
   curl -s -A "<your app UA>" "https://nominatim.openstreetmap.org/search?q=<query>&format=json"
   ```
   -> lat/lon. Respect their usage policy (1 req/s, real UA).

2. PICK ZOOM by feature size: z13 frames roughly a 4km area (city-district
   context with readable road/place labels); z15-16 for a single block; z11-12
   for metro context. Zoom OUT enough that surrounding labels render - a tight
   crop at high zoom is an unlabeled beige blob.

3. FETCH A TILE MOSAIC, not one tile: compute the tile x/y for the center,
   then fetch a 5x5 grid around it (1280px at 256px tiles):

   ```
   x = floor((lon+180)/360 * 2^z)
   y = floor((1 - ln(tan(lat_rad) + 1/cos(lat_rad))/pi)/2 * 2^z)
   URL: https://tile.openstreetmap.org/{z}/{x}/{y}.png
   ```
   Paste into a mosaic canvas at `((tx-x0)*256, (ty-y0)*256)`.

4. CROP the target region with PIL - AND GUARD THE BOUNDS. This is the killer
   bug: PIL crop() SILENTLY RETURNS BLACK STRIPS when the crop box runs off
   the canvas. A 620px box centered on arbitrary coords needs the full mosaic
   behind it. Assert:

   ```python
   assert 0 <= left and left + CROP_W <= mosaic_w
   assert 0 <= top and top + CROP_H <= mosaic_h
   ```
   (compute the pixel position of lat/lon within the mosaic via the standard
   Web Mercator projection; zoom the crop center, not the tile corner).

5. FRAME IT: paste the crop onto a card with a white border so edges read as
   intentional, not sliced. Add a pin/marker at the exact center and a
   location label in your card font. Optional: dim/desaturate the map slightly
   so overlaid text pops.

## Pitfalls
- Black strips from off-mosaic crops are SILENT - the assert is mandatory, and
  visually check corners of every rendered card.
- Tile servers rate-limit; cache tiles to disk and reuse, keep a real UA, and
  don't hammer (bulk jobs should use a proper tile provider or self-hosting).
- Web Mercator math must use the same z in projection and tile fetch; mixing
  zoom levels shifts the center.
- Dark-mode tiles exist (CARTO basemaps) if the card design needs it - same
  x/y/z scheme, different host; check their usage terms.
