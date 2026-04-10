# Design: Interactive Family Tree Tab & Map Updates

**Date:** 2026-04-10
**Status:** Draft
**Scope:** Add zoomable D3.js family tree, update Leaflet map with new markers/routes, deploy to GitHub Pages

---

## Overview

Add an interactive, zoomable family tree visualization to the Schauss/Shouse heritage site as a new navigational tab. Update the existing Leaflet map with new Pennsylvania markers, Ship Harle route details, and German archive markers. Deploy the site to GitHub Pages.

## 1. Family Tree Tab

### 1.1 Navigation

Add "Family Tree" as a new nav link between "German Origins" and "Timeline". Clicking scrolls to a new `<section id="tree-section">`.

### 1.2 Section Layout

```
+------------------------------------------------------+
| Section Label: "Genealogy"                           |
| H2: "Schauss / Shouse Family Tree"                  |
| Description paragraph                                |
|                                                      |
| [Legend Bar]                                          |
|   gold=German  orange=Emigrated  blue=American       |
|   green=Stayed in Germany                            |
|                                                      |
| +--------------------------------------------------+ |
| |                                                    | |
| |           SVG Family Tree (D3.js)                 | |
| |           Pan/zoom enabled                        | |
| |           ~700px height, 100% width               | |
| |                                           [+][-]  | |
| |                                           [Reset]  | |
| +--------------------------------------------------+ |
|                                                      |
| [Detail Panel - slides in on node click]             |
+------------------------------------------------------+
```

### 1.3 Tree Layout

- **Direction:** Top-down vertical hierarchy. Root (Hanss Schauss, ~1560) at top, descendants below.
- **Branch split:** At the 5th/6th generation, the tree visually forks:
  - **Left branch:** Philipp Heinrich Schauss (stayed in Germany, Mayor of Albisheim) — gold/green styling
  - **Right branch:** Johann Adam Schauss (emigrated 1736) — orange styling, American-born descendants in blue
- **Spouse nodes:** Connected families (Baum, Naass, Conrad, Hessler, Wickert, Beck) appear as smaller nodes joined to their partner by a horizontal marriage connector line. Spouses do not have their own ancestor chains — only their vital info.
- **Collapsed by default:** The tree starts with generations 1-4 expanded (Hanss Schauss through Hans Bernhardt Schauss and his wife). Generation 5 (the four siblings including Philipp Heinrich and Johann Conrad) is one click away, revealing the stayed/emigrated split. This prevents visual overwhelm on load while placing the Albisheim founding moment center stage.

### 1.4 Node Design

Each node is a rounded rectangle rendered in SVG:

```
+---------------------------+
| Name (bold, gold-lt)      |
| b. 1675 — d. 1734        |
| Albisheim, Pfalz          |
+---------------------------+
```

- **Size:** ~200px wide x ~70px tall for primary nodes; ~160px x ~55px for spouse nodes
- **Border color:** Indicates category:
  - `var(--gold)` / `#c9a84c` — German-born, lived in Germany
  - `#4caf7d` (green) — Stayed in Germany (Philipp Heinrich's line)
  - `#e8703a` (orange) — Emigrated to America
  - `#4a90d9` (blue) — Born in America
  - `var(--sepia)` / `#8b6f47` — Connected families (spouses from other surnames)
- **Background:** `var(--bg2)` / `#1c1916`
- **Hover:** Border brightens, subtle glow
- **Click:** Opens detail panel

### 1.5 Detail Panel

On node click, a panel appears as an overlay card anchored to the bottom-right of the tree viewport:

```
+------------------------------------+
| [X close]                          |
| JOHANN ADAM SCHAUSS                |
| Emigrated 1736          [tag]      |
|                                    |
| Born:  c. 1703/1704, Immesheim    |
| Died:  Dec 12, 1770, Bethania, NC |
| Occupation: Wheelwright & Miller   |
|                                    |
| Marriage:                          |
|   Maria Barbara Baum               |
|   Jan 16, 1725, Albisheim          |
|                                    |
| Sources:                           |
|   - WikiTree (link)                |
|   - Ship Harle passenger list      |
|   - Weller Dissertation (2004)     |
+------------------------------------+
```

- Styled to match the existing dark theme (bg2, gold-lt headings, text-dim body)
- Sources rendered as clickable links where URLs are available
- Tags: "Stayed in Germany", "Emigrated 1736", "Mayor of Albisheim", etc.
- Close via X button or clicking elsewhere

### 1.6 Zoom Controls

- **Scroll wheel:** Zoom in/out (D3 zoom behavior)
- **Click-drag:** Pan the tree
- **Touch:** Pinch-to-zoom, drag-to-pan on mobile
- **Buttons:** +/- zoom buttons and a "Reset" button in the bottom-right corner of the SVG container
- **Initial view:** Centered on the 4th generation (Hans Bernhardt Schauss) so the "founding of Albisheim" moment is visible on load

### 1.7 D3.js Integration

- Load `d3@7` from `https://unpkg.com/d3@7/dist/d3.min.js`
- Use `d3.tree()` layout for hierarchical positioning
- Use `d3.zoom()` for pan/zoom on the SVG `<g>` container
- Animate expand/collapse with `d3.transition()`
- Marriage links rendered as separate horizontal path elements connecting spouse pairs

## 2. Family Tree Data

### 2.1 Data Structure

A single JSON object embedded in a `<script>` tag within `index.html`. Each person node:

```json
{
  "id": "hanss-schauss",
  "name": "Hanss Schauss",
  "birth": "before 1600",
  "birthPlace": "Eckweiler, Bad Kreuznach",
  "death": "unknown",
  "deathPlace": "",
  "occupation": "",
  "tag": "german",
  "marriage": {
    "spouse": "Catharina Von Allenfeldt",
    "spouseBirth": "c. 1560",
    "spouseDeath": "Apr 5, 1609",
    "date": "",
    "place": "Eckweiler"
  },
  "sources": [
    { "label": "WikiTree - Melchior Schauss", "url": "https://www.wikitree.com/wiki/Schau%C3%9F-85" }
  ],
  "children": [...]
}
```

Tags: `"german"`, `"stayed"`, `"emigrated"`, `"american"`, `"connected"` (for spouses from other families).

### 2.2 People to Include (~45 nodes)

**German Line (Eckweiler/Budingen/Albisheim):**
1. Hanss Schauss (bef. 1600) + Catharina Von Allenfeldt
2. Melchior Schauss (1595-1628) + Helena Antesses
3. Johannes Schauss (c.1620-1674) + Maria Magdalena Klein
4. Hans Bernhardt Schauss (c.1650-1720) + Anna Margarethe Naass
5. Johann Conrad Sr. (1675-1734) + Anna Engle-Burgis Conrad + Johanna Felicitas Hesslerin
6. Johannes Schauss (1678) + Maria Barbara Meyers
7. Anna Margaretha Schauss (1680)
8. Hans Adam Schauss (1683)
9. Philipp Heinrich Schauss (1686-1765) + Anna Hedwig Waltherin — **STAYED branch**

**Emigrated Branch:**
10. Johann Adam Schauss (c.1703-1770) + Maria Barbara Baum — **EMIGRATED branch**
11. All 11 children of Johann Adam (4 German-born, 7 American-born)

**Connected Families (spouse nodes only):**
- Baum: Johann Philipp Baum + Anna Margaretha Stahler (parents of Maria Barbara)
- Naass: Anna Margarethe Naass (vital info only)
- Conrad: Anna Engle-Burgis Conrad (vital info only)

**Eckweiler Branch (sidebar):**
- Nicholaus Schauss (c.1685) + Maria Elisabetha Wickert — shown as a collateral branch off the Eckweiler generation

## 3. Map Updates

### 3.1 New Residence Markers (Blue)

| Location | Coords | Popup Content |
|----------|--------|---------------|
| Falkner Swamp, Montgomery Co., PA | [40.27, -75.58] | 5 children born here 1737-1741, German Reformed community |
| Bethlehem, Northampton Co., PA | [40.6259, -75.3707] | Gottlieb & Beniginia born here 1744-1746, Moravian community |
| Whitehall, Bucks Co., PA | [40.66, -75.49] | Christian Shouse born 1748, last child |
| Easton, Northampton Co., PA | [40.69, -75.22] | Maria Barbara Baum died here before 1768 |

### 3.2 New Archive Marker (Green)

| Location | Coords | Popup Content |
|----------|--------|---------------|
| Pfalzbibliothek, Kaiserslautern | [49.4447, 7.7689] | Holds Ortsfamilienbuch "Die Familien Albisheims 1641-1900" (catalog 1b 6809), church book transcripts |

### 3.3 Ship Harle Marker

Replace the existing static Atlantic text label with an interactive marker:
- **Position:** Mid-Atlantic [42.0, -35.0]
- **Icon:** Custom ship icon (small SVG boat silhouette, orange color, ~20px)
- **Popup content:**
  ```
  Ship Harle of London
  Master: Ralph Harle
  Route: Rotterdam → Cowes → Philadelphia
  Arrived: September 1, 1736
  Passengers: 388 total (151 qualified foreigners)
  Passenger #94: "Johan Adam Shans" — age 32
  Source: Pennsylvania German Pioneers, Vol. 1
  ```

### 3.4 Waypoint Markers

Add small, subtle markers for:
- **Rotterdam** [51.9225, 4.4792] — "Embarkation port for Palatine emigrants. Ships departed down the Rhine to Rotterdam before crossing to England."
- **Cowes, Isle of Wight** [50.7636, -1.2980] — "Last port before Atlantic crossing. Ships stopped to take on provisions and additional passengers."

Style: Smaller dot (10px), muted orange color, to indicate they're waypoints not residences.

### 3.5 Update Existing Markers

- **Albisheim popup:** Add reference to Ortsfamilienbuch by Uhrig (2009) and Geschichts- und Heimatverein contact
- **Eckweiler popup:** Add FHL microfilm numbers (489885, 493211) and note about Staatsarchiv Koblenz
- **German migration path:** Change from gold dashed to gold dotted with slightly thicker weight (3px) to differentiate from transatlantic route

### 3.6 Initial Map View

Change the default map center/zoom to show both Germany and the US East Coast on load: center `[42, -30]`, zoom `3`. This frames the full migration story immediately.

## 4. GitHub Pages Deployment

### 4.1 Steps

1. Commit all changes to `main`
2. Push to `origin/main`
3. Enable GitHub Pages via `gh api` — deploy from `main` branch, root `/` directory
4. Set repo homepage URL to `https://patjackson52.github.io/shouse_project/`
5. Verify the site loads correctly

### 4.2 No Build Step

The site is a single `index.html` with CDN dependencies (Leaflet, D3). No build process needed. GitHub Pages serves the file directly.

## 5. CSS Additions

All new styles added to the existing `<style>` block in `index.html`:

- `#tree-section` — section container
- `#tree-container` — SVG wrapper with dark background, border, border-radius
- `.tree-node` — base node styling (rect + text)
- `.tree-node--german`, `--stayed`, `--emigrated`, `--american`, `--connected` — border color variants
- `.tree-node:hover` — brighten border, subtle box-shadow
- `.tree-link` — path styling for parent-child connections (curved, sepia colored)
- `.tree-marriage-link` — horizontal line for spouse connections (dashed, gold)
- `.tree-legend` — legend bar (reuse existing `.map-legend` pattern)
- `.tree-controls` — zoom button group (position absolute, bottom-right)
- `.tree-detail` — detail panel overlay card
- `.tree-detail__tag` — badge styling (reuse existing `.stayed-tag` / `.emigrated-tag`)

## 6. Non-Goals

- No backend or database — all data is static JSON in HTML
- No user editing of the tree — read-only visualization
- No GEDCOM import/export
- No changes to existing timeline, German Origins, family cards, or sources sections
- No separate JavaScript files — everything stays in `index.html`

## 7. Browser Support

The site targets modern browsers (Chrome, Firefox, Safari, Edge — last 2 versions). D3 v7 requires ES2015+. No IE11 support.
