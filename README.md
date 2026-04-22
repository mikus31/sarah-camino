# Camino del Norte — Daily Tracker

An interactive map that tracks your wife's 40-day Camino del Norte pilgrimage across northern Spain. Each day she sends a photo + short write-up, you add a line to `data/journal.json`, and the map updates for everyone following along.

## What you've got

```
camino-del-norte/
├── index.html           ← the map (open this in a browser)
├── data/
│   ├── route.geojson    ← the Camino del Norte trail line
│   └── journal.json     ← daily entries (edit this as she walks)
├── photos/              ← drop her daily photos here
└── README.md            ← you are here
```

The map shows the full route as a faint dashed line, her walked-so-far portion as a bold terracotta line, small brown dots for each past day (clickable for a photo + caption popup), and an animated pulsing marker for the most recent day. The most-recent popup auto-opens when the page loads. A top-right control lets viewers toggle between basemaps; a legend sits in the bottom-left.

## First-run setup (5 minutes)

### 1. Try it locally

The map uses `fetch()` to load `data/route.geojson` and `data/journal.json`, which browsers block when you open `index.html` directly from the filesystem. Serve it locally instead:

```bash
# from inside the camino-del-norte folder
python3 -m http.server 8000
# then visit  http://localhost:8000  in your browser
```

Or if you have Node: `npx serve`.

You should see the full route across northern Spain, three sample day markers (Irún → San Sebastián → Zarautz → Deba), and the pulsing "current day" marker sitting on Day 3.

### 2. Pick a basemap (optional — Esri works out of the box)

You picked "let me see all three" — the map already loads **Esri Topographic** and **Esri Satellite** with no signup required. To also enable MapTiler Outdoor and Mapbox Outdoors in the basemap toggle, paste your keys into `index.html` near the top:

```js
const CONFIG = {
  MAPTILER_KEY: "paste_key_here",
  MAPBOX_TOKEN: "paste_token_here"
};
```

**Getting the keys (2 minutes each, free):**

- **MapTiler**: sign up at https://cloud.maptiler.com → Account → Keys → copy the default key. Free tier is 100k map loads/month, more than enough.
- **Mapbox**: sign up at https://account.mapbox.com → Tokens → copy the default public token (starts with `pk.`). Free tier is 50k map loads/month.

Leave either blank and that basemap just won't appear in the toggle — the map still works.

### 3. Personalize the banner

Open `data/journal.json` and change `"pilgrim"` to her name and `"startDate"` to her actual start date. The banner in the top-left uses these.

## Adding each day (the daily workflow)

When she sends you a photo and a note for Day N:

1. **Drop the photo** into `photos/` as `day-NN.jpg` (e.g. `day-04.jpg`). Resize it to ~1400px on the long edge so the map stays fast — see `photos/README.txt` for a one-liner.

2. **Append an entry** to the `entries` array in `data/journal.json`:

```json
{
  "day": 4,
  "date": "2026-06-18",
  "location": "Deba → Markina-Xemein",
  "coords": [43.2670, -2.5010],
  "distanceKm": 24,
  "photo": "photos/day-04.jpg",
  "caption": "Inland today — forests, a steep climb, and a tiny village where the café owner filled my water bottle for free."
}
```

Notes on the fields:

- `coords` is `[latitude, longitude]` (in that order). Easiest way to get them: right-click the spot on [Google Maps](https://maps.google.com), the top entry in the menu is the lat/long — click it to copy.
- `distanceKm` is optional but it's used for the "X km walked" total in the banner.
- `day` must be unique. Whichever entry has the highest `day` number is styled as "most recent" (the pulsing marker).

3. **Push to GitHub** (if using GitHub Pages — see below). The map updates for everyone within a minute or two.

That's the whole workflow. After the first week it takes about 60 seconds per day.

## Deploying to GitHub Pages (so friends/family can see it)

1. Create a new repo on GitHub (public is fine; name it anything — e.g. `camino-tracker`).
2. From inside the `camino-del-norte/` folder:

```bash
git init
git add .
git commit -m "Initial map"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/camino-tracker.git
git push -u origin main
```

3. On GitHub: repo **Settings** → **Pages** → **Source: Deploy from a branch** → Branch: **main**, folder: **/ (root)** → **Save**.
4. Wait ~60 seconds and your map will be live at `https://YOUR_USERNAME.github.io/camino-tracker/`. Share that link with family.

To publish a daily update thereafter:

```bash
git add photos/day-04.jpg data/journal.json
git commit -m "Day 4"
git push
```

GitHub Pages redeploys automatically.

### ⚠️ A note on API keys and public repos

If you paste your MapTiler or Mapbox key into `index.html` and push to a public repo, that key is visible to anyone who views the site source. Both providers let you restrict keys to specific URL origins (your GitHub Pages URL) — do that in their dashboards. **The two providers use different formats, so read carefully:**

**MapTiler** — Key settings → **Allowed HTTP Origins** → bare hostnames only, one per line, no scheme or port:

```
localhost
YOUR_USERNAME.github.io
```

**Mapbox** — Token settings → **URL restrictions** → full URLs with scheme, wildcards allowed:

```
http://localhost:*
https://YOUR_USERNAME.github.io/*
```

With origin restrictions on, a scraper can't spend your quota even if they grab the key.

If you'd rather not deal with this, just leave both fields empty and use Esri Topographic — it's free forever and requires no keys.

## Upgrading to a high-resolution trail line (optional)

The bundled `data/route.geojson` is a 60-waypoint approximation of the route — it traces the correct shape but smooths over individual switchbacks. For a pixel-accurate GPS line:

1. Download a GPX file of the full Camino del Norte from one of:
   - https://www.cicerone.co.uk/the-camino-del-norte-and-camino-primitivo (free, requires account)
   - https://wikiloc.com (search "Camino del Norte completo")
   - https://santiago.forwalk.org/en-us/guide/8-the-camino-del-norte/
2. Convert GPX → GeoJSON at https://mapbox.github.io/togeojson/ (drag/drop, downloads the result) or install `togeojson` via npm.
3. Replace the `features[0].geometry` block in `data/route.geojson` with the converted LineString. Keep the outer structure (FeatureCollection wrapper) the same.

The map will pick it up automatically the next time you load the page.

## Things you might want to tweak later

- **Start-map framing.** The map auto-fits to the route bounds. If you want it to zoom to her current location instead, change the `map.fitBounds(...)` line near the top of the `renderTrip` function in `index.html` to `map.setView(latest.coords, 10);`.
- **Colors.** All the palette is in the CSS `:root` block at the top of `index.html` — change `--terracotta`, `--trail-past`, `--trail-full`, `--cream` to taste.
- **More stats.** The banner shows day count + km. You could add elevation gain, number of towns visited, etc. — see the `renderTrip` function where the banner is populated.
- **An RSS-style "latest" feed** for family who want to subscribe rather than visit the page. Out of scope for v1 but doable.

## License

The code is yours. The Camino del Norte route geometry is an approximation drawn from public knowledge of the standard itinerary. If you replace it with OpenStreetMap-sourced data, credit OSM per their [license](https://www.openstreetmap.org/copyright).

Buen Camino.
