# The Kit Room — Project Reference

## Overview

PWA for cataloguing a football shirt collection.
Hosted at: **https://mvaneeten.github.io/Kitroom**
Repo: `mvaneeten/Kitroom` on GitHub (push via **GitHub Desktop** — CLI push is rejected due to auth mismatch).

---

## File Structure

```
Kitroom/
├── index.html        ← entire app (HTML + CSS + JS, all inline)
├── sw.js             ← service worker (cache-first PWA)
├── manifest.json     ← PWA manifest
├── shirts.json       ← data store (all shirts)
├── icons/
│   ├── icon-180.png
│   ├── icon-192.png
│   └── icon-512.png
└── images/
    └── {id}.jpg      ← shirt photos, named by shirt id (e.g. 42.jpg, s1.jpg)
```

---

## Service Worker (`sw.js`)

- Cache name: `kitroom-cache-vNN` — **must be bumped on every deploy** to force iOS cache invalidation
- Caches: `./`, `./index.html`, `./manifest.json`, `./shirts.json`, icons
- Strategy: cache-first; caches new successful same-origin GET responses; falls back to `index.html` for navigations

Current version: **v29**

---

## App Version

In `index.html`, find the version span in the header:

```html
<span style="font-size:12px;opacity:0.5;letter-spacing:1px;">v3.6</span>
```

Bump both the app version and `CACHE_NAME` in `sw.js` on every meaningful deploy.

---

## Design Tokens (CSS variables)

| Variable | Value | Use |
|---|---|---|
| `--pitch` | `#163a2f` | Header background (dark green) |
| `--pitch-dark` | `#0e2a21` | Active tab text |
| `--paper` | `#f6f1e7` | Page background |
| `--paper-dim` | `#ece4d3` | Dimmed surface |
| `--ink` | `#20281f` | Body text |
| `--gold` | `#d9a629` | Accent, active tab |
| `--line` | `#cdc3ac` | Borders |
| `--red` | `#bd4330` | Type badge background |
| `--card-bg` | `#ffffff` | Card background |

Fonts: **Anton** (headings/tabs, uppercase), **Work Sans** (body)

---

## Data: `shirts.json`

Array of shirt objects. Current count: **205 regular + specials s1–s6 = 211 total**

### Shirt fields

```json
{
  "id":        204,
  "club":      "Manchester United",
  "season":    "2026/2027",
  "type":      "Away",
  "continent": "Europe",
  "country":   "England",
  "brand":     "Adidas",
  "year":      2026,
  "image":     null,
  "player":    null,
  "notes":     null,
  "trip":      "Manchester & Liverpool 2026",
  "story":     null,
  "store":     null,
  "isSouvenir": false
}
```

### Field notes

| Field | Notes |
|---|---|
| `id` | Integer for regular shirts (1–205), string for specials (`"s1"`–`"s6"`) |
| `type` | `Home`, `Away`, `Third`, `4th`, `GK`, `Limited Edition`, `Other` |
| `continent` | `Europe`, `South America`, `North America`, `Asia`, `Africa`, `Oceania` |
| `image` | Override image filename (without path/ext). Falls back to `images/{id}.jpg` |
| `player` | Format: `"11 - Godts"` |
| `trip` | Trip name string, e.g. `"London May 2025"` |
| `isSouvenir` | `true` = souvenir brought by someone else; `false` = bought on own trip |
| `store` | Where purchased, e.g. `"Mystery United"`, `"Ajax Fanshop"` |

### Special shirts

IDs `s1`–`s6` — these are non-standard items (replicas, limited editions, etc.).  
Predicate: `const isSpecial = s => String(s.id).startsWith('s') || s.type === 'Other';`  
Specials always sort to the **end** of lists.

### Known stores

- `Mystery United` — shirts: 27,28,30,31,33,40,43,46,50,52,53,58,61,63,70,72,76,89,90,113,117,143,144,159,160,173,189,202
- `Ajax Fanshop` — shirts: 7,9,10,11,12,14,19,20,21,22,23,24,25,26,s2,44,47,48,s3,64,65,71,92,141,s5,s6,199,200,201

---

## Tabs / Navigation

| Tab | Function |
|---|---|
| **Collection** | Full grid with filter bar |
| **Ajax** | Ajax-only shirts, image cards only |
| **By year** | Grouped by purchase year |
| **Clubs** | Grouped by club, with country filter |
| **Travel** | Shirts tied to trips + souvenirs |
| **Stats** | Dashboard with charts |
| **World map** | D3 + TopoJSON world map |

---

## Collection Filter Bar

Filters: Club (text), Season (text), Type (select), Continent (select), Country (select — JSON-only countries), Brand (select — JSON-populated), Store (select — JSON-populated), Year (number), Sort (select), Specials (checkbox), Search + Clear buttons.

- **Text/number fields** → filter on Enter key or Search button click
- **Select fields** → filter instantly on `change`
- Default sort: `year-desc` (newest first, tie-break by highest id)

---

## Ajax Tab

- Hides shirts with ids in `ajaxHidden = new Set([115, 165, 127, 105])`
- Cards show image only (no text), red type badge, shirt number overlay
- Multi-year seasons (e.g. `2026/2027`) expand to show both years

---

## By Year Tab

- Groups shirts by `year` field
- Type badge (red) on each card
- Sort asc → lowest id first within year; sort desc → highest id first within year

---

## Detail Screen

Opens on shirt click in Collection, Ajax, By Year.

```js
openDetail(id)   // opens full-screen overlay
closeDetail()    // returns to previous tab (stored in detailFromView)
```

- Overlay: `position:fixed`, `background:var(--paper)`, `z-index:1000`
- Shows: image, club, type badge, season, brand, country, continent, year, player (if set), notes (if set), store (if set), trip/souvenir info (if set), story (if set)
- Label: "Souvenir from" when `isSouvenir`, "Bought on trip" otherwise

---

## Travel Tab

Two sections:
1. **My Trips** — shirts where `trip` is set and `isSouvenir` is false, grouped by trip name
2. **Souvenirs** — shirts where `isSouvenir` is true, grouped by trip/giver name

---

## Stats Tab

Cards: total count, type breakdown, top brands (bar chart), top clubs, top countries, by continent, purchased per year (column chart), highlights (oldest/newest/top club/top brand).

---

## Sorting Logic

Regular shirts and specials are split before sorting to avoid NaN from `Number("s1")`:

```js
const regular = filtered.filter(s => !isSpecial(s));
const specials = filtered.filter(s => isSpecial(s));
regular.sort(sortFn);
specials.sort(sortFn);
const sorted = [...regular, ...specials];
```

---

## Images

- Stored in `images/`
- Named `{id}.jpg` — e.g. `42.jpg`, `s1.jpg`
- Override with `image` field in JSON (without path or extension)
- Missing images show a placeholder

---

## Deployment Checklist

1. Make changes to `index.html` and/or `shirts.json`
2. Bump version number in `index.html` header span
3. Bump `CACHE_NAME` in `sw.js` (e.g. `kitroom-cache-v30`)
4. Push via **GitHub Desktop** (CLI push is rejected)
5. GitHub Pages auto-deploys from `main` branch
