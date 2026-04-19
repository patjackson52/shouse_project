# Historical Map Timeline — Design Spec

**Date:** 2026-04-19
**Status:** Approved
**Scope:** Upgrade the existing Leaflet map in `index.html` to support historical map overlays, era-based navigation, and a timeline slider — with mobile as a first-class experience.

---

## Overview

Transform the current static single-layer Leaflet map into a "map through time" that lets users navigate seven eras of the Schauss/Shouse family history. Era buttons and a timeline slider overlay the map (Google Maps style). Where historic cartographic maps are freely available, they appear as tile overlays that can be toggled and opacity-adjusted. All controls are touch-friendly and responsive.

**Approach:** Pure Leaflet (no new dependencies). All controls are custom HTML/CSS positioned with `L.Control.extend()`.

---

## Data Model

### Era Configuration

```javascript
const ERAS = [
  { id: 1, label: 'Eckweiler Origins',    short: 'Eckweiler',    range: [1560, 1620], center: [49.78, 7.68], zoom: 11 },
  { id: 2, label: 'Büdingen & Albisheim', short: 'Albisheim',    range: [1620, 1700], center: [49.70, 7.95], zoom: 10 },
  { id: 3, label: 'Albisheim Roots',      short: 'Roots',        range: [1700, 1736], center: [49.65, 8.10], zoom: 11 },
  { id: 4, label: 'The Emigration',       short: '1736',         range: [1736, 1736], center: [42.0, -30.0], zoom: 3  },
  { id: 5, label: 'Pennsylvania Years',   short: 'Pennsylvania', range: [1736, 1753], center: [40.3, -75.5], zoom: 8  },
  { id: 6, label: 'North Carolina',       short: 'Carolina',     range: [1753, 1810], center: [36.1, -80.3], zoom: 9  },
  { id: 7, label: 'Modern Day',           short: 'Modern',       range: [1900, 2026], center: [42.0, -30.0], zoom: 3  },
];
```

Each era defines: year range for the sub-slider, map fly-to center, and zoom level.

### Marker Data Extensions

Each existing marker object gains two fields:

```javascript
{
  ll: [49.65, 8.1], title: '...', historic: '...', dates: '...', body: '...',
  eras: [1, 2, 3],   // array of era IDs where this marker appears
  year: 1674          // first relevant year (for slider scrubbing within an era)
}
```

Polylines (migration paths) also get an `eras` array (e.g., the Harle route only shows during era 4, the German path in eras 1–3).

### Historic Tile Overlay Configuration

```javascript
const HISTORIC_TILES = [
  {
    eras: [3, 4],
    url: '...tranchot-wms-or-tile-url...',
    bounds: [[49.5, 7.0], [50.0, 8.5]],
    label: 'Tranchot-Müffling 1803–20',
    opacity: 0.6
  },
  // Additional overlays added as sourced during implementation
];
```

Each overlay is an `L.tileLayer` (for WMS/XYZ tile services) or `L.imageOverlay` (for single georectified images). Only overlays relevant to the current era appear in the Layers panel.

**Sourcing strategy:** Use freely available tile services where they exist. Likely candidates:
- **Tranchot-Müffling (1803–1820)** — Geoportal NRW / Geobasis NRW WMS for the Rhineland/Palatinate
- **Colonial-era maps** — David Rumsey Collection georectified tiles: Fry-Jefferson (1751 Virginia), Mouzon (1775 Carolinas), Scull (1770 Pennsylvania)
- **If unavailable for an era:** skip with a "No historic map available for this period" note in the Layers panel. The modern dark base map is always available.

---

## UI Controls

All controls overlay the map using `L.Control.extend()` with custom HTML/CSS. No controls live outside the map container.

### Control Positions

| Control | Desktop Position | Mobile Position |
|---------|-----------------|-----------------|
| Era buttons | Top-left, horizontal row | Top, horizontal scroll strip |
| Era info badge | Below era buttons, top-left | Below era strip, left (tappable to expand/collapse) |
| Layers toggle | Top-right, dropdown on click | Top-right, icon-only (🗺️), opens bottom sheet |
| Zoom +/− | Right side, vertically centered | Same (Leaflet default) |
| Timeline slider | Bottom edge, full width | Bottom edge, full width, larger touch targets |
| Prev/Play/Next | Below slider, centered | Below slider, centered, larger tap targets |

### Era Buttons

- Desktop: full row of labeled buttons, one per era. Active era has gold border + gold text. Inactive: muted border + muted text. Semi-transparent dark background.
- Mobile: horizontal scroll strip with `-webkit-overflow-scrolling: touch`. Uses `short` labels. Larger padding (12px vs 10px) for touch targets.
- Clicking an era button: switches to that era (triggers fly-to, marker filter, overlay swap, slider range update).

### Era Info Badge

- Shows current era's `label` and year `range`, plus a one-line description of who/what is relevant.
- Desktop: always visible below era buttons, max-width 280px.
- Mobile: collapsed by default (shows era name + year range only). Tap to expand description. Saves vertical space.

### Layers Panel

Three sections:

```
Base Map
  ○ Dark  ● Light  ○ Satellite

Historic Overlay              [opacity ━━━●━━]
  ☑ Tranchot-Müffling 1803–20

Data
  ☑ Residences  ☑ Churches  ☑ Cemeteries
  ☑ Archives    ☑ Migration Paths
```

- **Base layers** (radio buttons — pick one): Dark (CartoDB Dark Matter, current default), Light (CartoDB Positron), Satellite (ESRI World Imagery, free, no API key).
- **Historic overlays** (checkboxes): only overlays for the current era are listed. Each has an opacity slider.
- **Data layers** (checkboxes): the existing five marker categories, now grouped into `L.layerGroup` objects. Default: all on.

Desktop: dropdown panel that opens on click of "🗺️ Layers ▾" button (top-right).
Mobile: bottom sheet that slides up from the bottom edge on tap of the 🗺️ icon.

### Timeline Slider

- Range input (`<input type="range">`) styled with custom CSS to match the dark/gold theme.
- Range dynamically updates to the current era's `[range[0], range[1]]`.
- For single-year eras (e.g., era 4 "The Emigration" where `range` is `[1736, 1736]`), the slider is hidden and only the Prev/Play/Next buttons remain.
- Desktop: track height 4px, thumb 14px diameter.
- Mobile: track height 6px, thumb 20px diameter, touch area 28px tall.
- Below the slider: Prev Era / Play / Next Era buttons.
- Year labels at left (start) and right (end) of the slider.
- Tick marks at 25% intervals along the track.

### Existing Legend

The current `.map-legend` bar above the map (Residences, Churches, Cemeteries, Archives, Migration Path) is **removed** — its function moves into the Layers panel. This recovers vertical space and eliminates redundancy.

---

## Transitions & Animation

### Era Switching

When the user switches eras (button click, prev/next, or play):

1. **Map flies** to the new era's `center`/`zoom` using `map.flyTo()` — smooth animated pan+zoom, ~1.5s duration.
2. **Markers fade** — outgoing markers: CSS `opacity: 0` over 300ms, then removed from map. Incoming markers: added with `opacity: 0`, fade to `1` over 300ms.
3. **Historic overlay crossfade** — new overlay fades in from `opacity: 0` to target opacity over 500ms (using `setOpacity()` in `requestAnimationFrame`). Old overlay fades out simultaneously.
4. **Polylines** — same fade as markers. The Harle route (era 4) draws itself using Leaflet's `dashOffset` animation trick (~2s trace across the Atlantic).
5. **Timeline slider** — range labels and tick marks update to the new era's year range. Thumb resets to the start position.

### Play Mode

- Clicking Play auto-advances through eras with a 5-second pause on each.
- Within each era, the slider scrubs forward automatically (~1 year per 200ms).
- Play button becomes Pause. Stops at the end of era 7.

### Slider Scrubbing

- Dragging the slider within an era filters markers by `year` — markers with `year > sliderPosition` fade out.
- Lets users watch the family appear one generation at a time.

### Accessibility & Performance

- All animations use CSS transitions or `requestAnimationFrame` — no jQuery.
- Marker fade uses `will-change: opacity` for GPU compositing.
- On devices with `prefers-reduced-motion`: skip `flyTo` (use instant `setView`), reduce fade durations to 100ms.

---

## Map Sizing

### Desktop (≥768px)
- `height: 70vh`, `min-height: 500px`, `max-height: 800px`
- `width: 100%` (unchanged)

### Mobile (<768px)
- `height: 60vh`, `min-height: 350px`
- On a typical phone (667px viewport): ~400px map height

### Popup Positioning
- Set `autoPanPaddingBottomRight: [10, 80]` to prevent popups from hiding behind the bottom timeline controls.
- Leaflet's built-in `autoPan` handles the rest.

---

## Implementation Scope

### What Changes
- **Map initialization:** add layer groups, era state management, custom controls
- **Marker data:** add `eras` and `year` fields to all existing marker objects
- **Polyline data:** add `eras` fields
- **CSS:** new styles for era buttons, slider, layers panel, bottom sheet, responsive breakpoints. Remove `.map-legend` styles.
- **HTML:** remove the `.map-legend` div above the map
- **JS:** new functions for era switching, slider filtering, layer toggling, play mode, animation

### What Doesn't Change
- The map container `<div id="map">` stays in the same position in the DOM
- Marker popup content and styling unchanged
- D3 family tree (separate section) untouched
- All other page sections untouched
- No new CDN dependencies

### Historic Map Research (Implementation Phase)
During implementation, research and test available free tile services:
- Check Geoportal NRW WMS for Tranchot-Müffling coverage of the Palatinate
- Check David Rumsey Map Collection for georectified colonial maps (PA, NC)
- Check Old Maps Online / Wikimedia for additional sources
- Document which eras have overlays and which don't — update `HISTORIC_TILES` accordingly
