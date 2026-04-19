# Historical Map Timeline Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Transform the static Leaflet map into an era-navigable "map through time" with historic tile overlays, a timeline slider, and mobile-first overlay controls.

**Architecture:** All changes are in `index.html` (single-file site). New CSS for overlay controls + responsive breakpoints. New JS for era state machine, layer groups, custom Leaflet controls, slider, animations. No new dependencies — pure Leaflet 1.9.4.

**Tech Stack:** Leaflet 1.9.4, vanilla JS, CSS custom properties (existing theme vars)

---

## File Structure

All changes in a single file:

- **Modify:** `index.html`
  - **CSS section** (~lines 134–188): Replace `.map-legend` styles with new era controls, slider, layers panel, bottom sheet, responsive breakpoints
  - **HTML section** (~lines 571–599): Remove `.map-legend` div, update map section intro text
  - **JS section** (~lines 2314–2762): Restructure into era config, layer groups, custom controls, animation functions

**Spec reference:** `docs/superpowers/specs/2026-04-19-historical-map-timeline-design.md`

---

### Task 1: Add Era Configuration and Marker Era Metadata

**Files:**
- Modify: `index.html:2314-2356` (JS section — add ERAS config before marker data)
- Modify: `index.html:2358-2506` (residences array — add `eras` and `year` fields)
- Modify: `index.html:2515-2572` (churches array — add `eras` and `year` fields)
- Modify: `index.html:2581-2610` (cemeteries array — add `eras` and `year` fields)
- Modify: `index.html:2619-2680` (repos/germanRepos arrays — add `eras` and `year` fields)
- Modify: `index.html:2688-2697` (waypoints — add `eras` and `year` fields)
- Modify: `index.html:2741-2760` (ship Harle — add `eras` and `year` fields)

- [ ] **Step 1: Add ERAS config array**

Insert after `(function () {` on line 2316, before the tile layer setup:

```javascript
  // ── Era configuration ───────────────────────────────────────────
  const ERAS = [
    { id: 1, label: 'Eckweiler Origins',    short: 'Eckweiler',    range: [1560, 1620], center: [49.78, 7.68], zoom: 11,
      desc: 'Hanß Schauß, Catharina Von Allenfeldt, Melchior\u2019s marriage 1621' },
    { id: 2, label: 'Büdingen & Albisheim', short: 'Albisheim',    range: [1620, 1700], center: [49.70, 7.95], zoom: 10,
      desc: 'Johannes Schauss settles Büdingen; Hans Bernhardt moves to Albisheim 1674' },
    { id: 3, label: 'Albisheim Roots',      short: 'Roots',        range: [1700, 1736], center: [49.65, 8.10], zoom: 11,
      desc: 'Johann Conrad Sr., Philipp Heinrich as Mayor, Johann Adam marries Maria Barbara Baum' },
    { id: 4, label: 'The Emigration',       short: '1736',         range: [1736, 1736], center: [42.0, -30.0], zoom: 3,
      desc: 'Ship Harle: Rotterdam → Cowes → Philadelphia, Johann Adam age 32' },
    { id: 5, label: 'Pennsylvania Years',   short: 'Pennsylvania', range: [1736, 1753], center: [40.3, -75.5], zoom: 8,
      desc: 'Falkner Swamp, Bethlehem, children born in PA' },
    { id: 6, label: 'North Carolina',       short: 'Carolina',     range: [1753, 1810], center: [36.1, -80.3], zoom: 9,
      desc: 'Bethania, Bethabara, Salem — Moravian settlements' },
    { id: 7, label: 'Modern Day',           short: 'Modern',       range: [1900, 2026], center: [42.0, -30.0], zoom: 3,
      desc: 'All markers — present-day Schauß families, heritage sites, archives' },
  ];

  let currentEra = 7; // start with all markers visible
```

- [ ] **Step 2: Add `eras` and `year` to every residence marker**

Add these two fields to each object in the `residences` array. Use the marker's historical context to determine which eras it belongs to and its first relevant year. Here are the assignments:

```javascript
// Albisheim an der Pfrimm
eras: [2, 3, 4, 5, 6, 7], year: 1674

// Eckweiler
eras: [1, 2, 3, 7], year: 1560

// Büdingen (Lorraine)
eras: [2], year: 1620

// Immesheim
eras: [2, 3, 7], year: 1700

// Göllheim
eras: [3, 7], year: 1723

// Gauersheim
eras: [2, 3, 7], year: 1688

// Philadelphia, PA
eras: [4, 5], year: 1736

// Lancaster County, PA
eras: [5], year: 1736

// Falkner Swamp, PA
eras: [5], year: 1737

// Whitehall, PA
eras: [5], year: 1748

// Easton, PA
eras: [5], year: 1760

// Zellertal-Niefernheim
eras: [7], year: 1900

// Schauß Mühle (Schauss Mill)
eras: [7], year: 1980

// Weingut Schauß, Monzingen
eras: [7], year: 1800

// Bethania, NC
eras: [6, 7], year: 1759

// Bethabara, NC
eras: [6, 7], year: 1753

// Salem (Old Salem), NC
eras: [6, 7], year: 1766

// Muddy Creek area, NC
eras: [6, 7], year: 1760

// Friedberg, NC
eras: [6, 7], year: 1769

// Friedland, NC
eras: [6, 7], year: 1780

// Germanton, NC
eras: [6, 7], year: 1790
```

Add these fields to each marker object at the end, before the closing `}`. For example, the Albisheim entry becomes:

```javascript
    {
      ll: [49.5847, 8.1147],
      title: 'Albisheim an der Pfrimm',
      historic: 'Donnersbergkreis, Rhineland-Palatinate',
      dates: 'Family seat 1674–present · Pop. 1,892',
      body: '...existing body text...',
      eras: [2, 3, 4, 5, 6, 7], year: 1674
    },
```

- [ ] **Step 3: Add `eras` and `year` to every church marker**

```javascript
// Bethania Moravian Church
eras: [6, 7], year: 1759

// Bethabara Moravian Church
eras: [6, 7], year: 1753

// Home Moravian Church, Salem
eras: [6, 7], year: 1771

// St. Philip's Moravian Church
eras: [6, 7], year: 1849

// Shiloh Union Church
eras: [6, 7], year: 1800

// Nazareth Lutheran Church
eras: [6, 7], year: 1800

// Peterskirche, Albisheim
eras: [1, 2, 3, 7], year: 1554

// Protestant Church, Eckweiler
eras: [1, 2, 7], year: 1568
```

- [ ] **Step 4: Add `eras` and `year` to every cemetery marker**

```javascript
// Dobbs Parish Graveyard
eras: [6, 7], year: 1760

// Bethania God's Acre
eras: [6, 7], year: 1760

// Salem Cemetery / God's Acre
eras: [6, 7], year: 1771

// Beck-Shouse Cemetery
eras: [6, 7], year: 1800
```

- [ ] **Step 5: Add `eras` and `year` to every repository and waypoint marker**

Repos (both US and German):
```javascript
// Moravian Archives, Winston-Salem
eras: [7], year: 1900

// Moravian Archives, Bethlehem PA
eras: [7], year: 1900

// Forsyth County Courthouse
eras: [7], year: 1900

// NC State Archives, Raleigh
eras: [7], year: 1900

// Zentralarchiv der Ev. Kirche der Pfalz
eras: [7], year: 1900

// Landesarchiv Speyer
eras: [7], year: 1900

// PRFK
eras: [7], year: 1900

// Pfalzbibliothek Kaiserslautern
eras: [7], year: 1900
```

Waypoints (Rotterdam, Cowes):
```javascript
// Rotterdam
eras: [4], year: 1736

// Cowes, Isle of Wight
eras: [4], year: 1736
```

Ship Harle marker:
```javascript
// Add eras: [4], year: 1736 to the ship marker data
```

- [ ] **Step 6: Verify the data compiles**

Open `index.html` in a browser. Open the developer console (F12). Confirm no JavaScript errors. The map should still render with all markers visible (since `currentEra = 7` and all markers include era 7 or will once we filter — at this point no filtering is wired up yet, so all markers just show as before).

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat(map): add ERAS config and era/year metadata to all markers"
```

---

### Task 2: Restructure Markers into Layer Groups

**Files:**
- Modify: `index.html:2508-2703` (marker `.addTo(map)` calls — replace with layer groups)
- Modify: `index.html:2705-2760` (polylines and ship marker — wrap in layer groups)

Currently each marker is added directly to the map with `.addTo(map)`. We need to wrap them in `L.layerGroup` objects so they can be toggled.

- [ ] **Step 1: Create layer groups and refactor marker rendering**

Replace all the existing `forEach` / `.addTo(map)` blocks (lines ~2508–2703) with this pattern. The marker creation stays the same, but instead of `.addTo(map)` we push into layer groups:

```javascript
  // ── Layer groups ────────────────────────────────────────────────
  const layers = {
    residences: L.layerGroup(),
    churches: L.layerGroup(),
    cemeteries: L.layerGroup(),
    repos: L.layerGroup(),
    waypoints: L.layerGroup(),
    paths: L.layerGroup()
  };

  // Store marker references with their data for era filtering
  const allMarkers = [];

  function addMarkersToLayer(data, layer, icon) {
    data.forEach(function(d) {
      var marker = L.marker(d.ll, { icon: icon })
        .bindPopup(popup(d.title, d.historic, d.dates, d.body));
      marker._eraData = d; // attach era metadata
      allMarkers.push({ marker: marker, layer: layer, data: d });
      layer.addLayer(marker);
    });
  }

  addMarkersToLayer(residences, layers.residences, makeIcon(BLUE, 14));
  addMarkersToLayer(churches, layers.churches, makeIcon(GOLD, 13));
  addMarkersToLayer(cemeteries, layers.cemeteries, makeIcon(GRAY, 12));
  addMarkersToLayer(repos, layers.repos, makeIcon(GREEN, 13));
  addMarkersToLayer(germanRepos, layers.repos, makeIcon(GREEN, 13));
```

- [ ] **Step 2: Refactor waypoints and ship into layer groups**

Replace the inline waypoints `forEach` block (~lines 2688-2697) and the ship Harle marker block (~lines 2741-2760):

```javascript
  // Waypoints
  var waypointData = [
    { ll: [51.9225, 4.4792], title: 'Rotterdam', historic: 'Waypoint', dates: '', body: 'Embarkation port for Palatine emigrants. Ships departed down the Rhine to Rotterdam before crossing to England.', eras: [4], year: 1736 },
    { ll: [50.7636, -1.2980], title: 'Cowes, Isle of Wight', historic: 'Waypoint', dates: '', body: 'Last port before Atlantic crossing. Ships stopped to take on provisions and additional passengers.', eras: [4], year: 1736 }
  ];
  addMarkersToLayer(waypointData, layers.waypoints, makeIcon(ORANGE_MUTED, 10));

  // Ship Harle
  var shipData = {
    ll: [42.0, -35.0], eras: [4], year: 1736
  };
  var shipMarker = L.marker(shipData.ll, { icon: shipIcon })
    .bindPopup('<div class="popup-title">Ship Harle of London</div>' +
      '<div class="popup-dates">Arrived Philadelphia \u2014 September 1, 1736</div>' +
      '<div class="popup-body">' +
      '<strong>Master:</strong> Ralph Harle<br>' +
      '<strong>Route:</strong> Rotterdam \u2192 Cowes \u2192 Philadelphia<br>' +
      '<strong>Passengers:</strong> 388 total (151 qualified foreigners)<br><br>' +
      '<strong style="color:#e8c97a;">Passenger #94: \u201cJohan Adam Shans\u201d</strong><br>' +
      'Age 32 \u2014 signed oath as \u201cJohann Adam Schauss\u201d<br><br>' +
      '<em style="font-size:0.75rem;">Source: Pennsylvania German Pioneers, Vol. 1 (Strassburger, 1980)</em>' +
      '</div>');
  shipMarker._eraData = shipData;
  allMarkers.push({ marker: shipMarker, layer: layers.waypoints, data: shipData });
  layers.waypoints.addLayer(shipMarker);
```

- [ ] **Step 3: Refactor polylines into layer group with era metadata**

Replace the german path and transatlantic path blocks (~lines 2705-2739):

```javascript
  // ── POLYLINES ──────────────────────────────────────────────────
  var allPolylines = [];

  // German migration path
  var germanPolyline = L.polyline([
    [49.7667, 7.7167], [50.0, 7.85], [49.5847, 8.1147]
  ], {
    color: '#c9a84c', weight: 3, opacity: 0.6,
    dashArray: '3, 6', lineJoin: 'round'
  }).bindPopup('<div class="popup-title">German Migration</div><div class="popup-body">Schauss family movement within Germany: Eckweiler (before 1600) \u2192 Büdingen (c. 1620) \u2192 Albisheim (1674 onward)</div>');
  germanPolyline._eraData = { eras: [1, 2, 3] };
  allPolylines.push(germanPolyline);
  layers.paths.addLayer(germanPolyline);

  // Transatlantic migration path
  var transatlanticPolyline = L.polyline([
    [49.5847, 8.1147], [51.9225, 4.4792], [50.7636, -1.2980],
    [39.9526, -75.1652], [40.0468, -76.1784], [36.9741, -79.0558], [36.1865, -80.3092]
  ], {
    color: '#e8703a', weight: 2.5, opacity: 0.75,
    dashArray: '8, 6', lineJoin: 'round'
  }).bindPopup('<div class="popup-title">Transatlantic Migration</div><div class="popup-body">Johann Adam Schauss: Albisheim \u2192 Rotterdam \u2192 London \u2192 Philadelphia (arrived Sep 1, 1736 on the <em>Harle</em>) \u2192 Pennsylvania \u2192 Bethania, NC (c. 1754)</div>');
  transatlanticPolyline._eraData = { eras: [4, 5] };
  allPolylines.push(transatlanticPolyline);
  layers.paths.addLayer(transatlanticPolyline);
```

- [ ] **Step 4: Add all layer groups to the map**

After all the marker and polyline setup, add:

```javascript
  // Add all layers to map (all visible by default for era 7)
  Object.values(layers).forEach(function(lg) { lg.addTo(map); });
```

- [ ] **Step 5: Verify no regressions**

Open `index.html` in a browser. Confirm all markers, polylines, and the ship icon still render exactly as before. Check the console for errors.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "refactor(map): wrap markers and polylines in layer groups"
```

---

### Task 3: Implement Era Filtering Function

**Files:**
- Modify: `index.html` (JS section — add `switchEra()` function after layer group setup)

- [ ] **Step 1: Write the `switchEra` function**

Add after the layer group setup (after `Object.values(layers).forEach(...)`):

```javascript
  // ── Era switching ───────────────────────────────────────────────
  var isAnimating = false;
  var reducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

  function switchEra(eraId) {
    if (isAnimating && eraId !== currentEra) return;
    var era = ERAS.find(function(e) { return e.id === eraId; });
    if (!era) return;
    currentEra = eraId;
    isAnimating = true;

    // Fly to era's center
    if (reducedMotion) {
      map.setView(era.center, era.zoom);
      isAnimating = false;
    } else {
      map.flyTo(era.center, era.zoom, { duration: 1.5 });
      map.once('moveend', function() { isAnimating = false; });
    }

    // Filter markers: show only those whose eras array includes currentEra
    allMarkers.forEach(function(entry) {
      var dominated = entry.data.eras.indexOf(currentEra) === -1;
      var el = entry.marker.getElement && entry.marker.getElement();
      if (dominated) {
        if (el) {
          el.style.transition = reducedMotion ? 'opacity 0.1s' : 'opacity 0.3s';
          el.style.opacity = '0';
          setTimeout(function() { entry.layer.removeLayer(entry.marker); }, reducedMotion ? 100 : 300);
        } else {
          entry.layer.removeLayer(entry.marker);
        }
      } else {
        if (!entry.layer.hasLayer(entry.marker)) {
          entry.layer.addLayer(entry.marker);
          el = entry.marker.getElement && entry.marker.getElement();
          if (el) {
            el.style.opacity = '0';
            el.style.transition = reducedMotion ? 'opacity 0.1s' : 'opacity 0.3s';
            requestAnimationFrame(function() {
              requestAnimationFrame(function() { el.style.opacity = '1'; });
            });
          }
        }
      }
    });

    // Filter polylines
    allPolylines.forEach(function(pl) {
      var dominated = pl._eraData.eras.indexOf(currentEra) === -1;
      if (dominated) {
        layers.paths.removeLayer(pl);
      } else if (!layers.paths.hasLayer(pl)) {
        layers.paths.addLayer(pl);
      }
    });

    // Update UI controls (implemented in later tasks)
    if (typeof updateEraUI === 'function') updateEraUI(era);
    if (typeof updateSliderRange === 'function') updateSliderRange(era);
  }
```

- [ ] **Step 2: Test era switching from console**

Open `index.html` in a browser. In the developer console, type:

```javascript
switchEra(1)
```

Verify: map flies to Eckweiler area (zoom 11), only Eckweiler-era markers visible (Eckweiler, Peterskirche, Protestant Church Eckweiler). Then try:

```javascript
switchEra(4)
```

Verify: map zooms out to Atlantic, waypoints + ship + transatlantic polyline visible.

```javascript
switchEra(7)
```

Verify: all markers reappear, map zooms to overview.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat(map): add switchEra() function with marker filtering and fly-to"
```

---

### Task 4: Add Era Button Controls (Overlay on Map)

**Files:**
- Modify: `index.html` CSS section (~line 134): add era control styles
- Modify: `index.html` JS section: add `L.Control` for era buttons

- [ ] **Step 1: Add CSS for era buttons and info badge**

Add after the existing `#map` CSS block (after line ~161), before the `.leaflet-popup-content-wrapper` styles:

```css
    /* ── Era controls (overlay on map) ─────────────────────────────── */
    .era-bar {
      display: flex;
      gap: 4px;
      pointer-events: auto;
    }
    .era-btn {
      background: rgba(30,26,20,0.92);
      border: 1px solid var(--border);
      color: var(--text-dim);
      padding: 4px 10px;
      border-radius: 4px;
      font-size: 0.68rem;
      font-family: inherit;
      cursor: pointer;
      white-space: nowrap;
      transition: border-color 0.2s, color 0.2s;
    }
    .era-btn:hover { border-color: var(--sepia); color: var(--text); }
    .era-btn.active {
      border-color: var(--gold);
      color: var(--gold);
    }
    .era-info {
      background: rgba(30,26,20,0.92);
      border: 1px solid var(--border);
      padding: 6px 12px;
      border-radius: 4px;
      max-width: 300px;
      pointer-events: auto;
      margin-top: 6px;
    }
    .era-info-title {
      font-size: 0.72rem;
      color: var(--gold);
      font-weight: bold;
    }
    .era-info-desc {
      font-size: 0.65rem;
      color: var(--text-dim);
      margin-top: 2px;
      line-height: 1.4;
    }

    @media (max-width: 767px) {
      .era-bar {
        overflow-x: auto;
        -webkit-overflow-scrolling: touch;
        scrollbar-width: none;
        max-width: calc(100vw - 24px);
      }
      .era-bar::-webkit-scrollbar { display: none; }
      .era-btn { padding: 6px 12px; font-size: 0.72rem; flex-shrink: 0; }
      .era-info { max-width: calc(100vw - 70px); }
      .era-info-desc { display: none; }
      .era-info.expanded .era-info-desc { display: block; }
    }
```

- [ ] **Step 2: Add era buttons as a Leaflet control**

Add in the JS section, after the `switchEra` function:

```javascript
  // ── Era button control ──────────────────────────────────────────
  var EraControl = L.Control.extend({
    options: { position: 'topleft' },
    onAdd: function() {
      var wrapper = L.DomUtil.create('div', '');
      wrapper.style.pointerEvents = 'none';
      L.DomEvent.disableClickPropagation(wrapper);
      L.DomEvent.disableScrollPropagation(wrapper);

      // Era buttons row
      var bar = L.DomUtil.create('div', 'era-bar', wrapper);
      ERAS.forEach(function(era) {
        var btn = L.DomUtil.create('button', 'era-btn', bar);
        btn.textContent = era.short;
        btn.dataset.eraId = era.id;
        btn.title = era.label + ' (' + era.range[0] + '\u2013' + era.range[1] + ')';
        if (era.id === currentEra) btn.classList.add('active');
        btn.addEventListener('click', function() { switchEra(era.id); });
      });

      // Info badge
      var info = L.DomUtil.create('div', 'era-info', wrapper);
      var infoTitle = L.DomUtil.create('div', 'era-info-title', info);
      var infoDesc = L.DomUtil.create('div', 'era-info-desc', info);
      var initEra = ERAS.find(function(e) { return e.id === currentEra; });
      infoTitle.textContent = initEra.label;
      infoDesc.textContent = initEra.range[0] + '\u2013' + initEra.range[1] + ' \u00b7 ' + initEra.desc;

      // Mobile: tap to expand
      info.addEventListener('click', function() { info.classList.toggle('expanded'); });

      // Store references for updateEraUI
      wrapper._bar = bar;
      wrapper._infoTitle = infoTitle;
      wrapper._infoDesc = infoDesc;
      wrapper._info = info;
      return wrapper;
    }
  });

  var eraControl = new EraControl();
  eraControl.addTo(map);

  // Update UI when era changes
  function updateEraUI(era) {
    var container = eraControl.getContainer();
    var buttons = container._bar.querySelectorAll('.era-btn');
    buttons.forEach(function(btn) {
      btn.classList.toggle('active', parseInt(btn.dataset.eraId) === era.id);
    });
    container._infoTitle.textContent = era.label;
    container._infoDesc.textContent = era.range[0] + '\u2013' + era.range[1] + ' \u00b7 ' + era.desc;
  }
```

- [ ] **Step 3: Remove the old HTML legend**

In the HTML section (~lines 580-596), delete the entire `.map-legend` div:

```html
  <!-- DELETE THIS BLOCK -->
  <div class="map-legend">
    <div class="legend-item">
      <div class="legend-dot" style="background:var(--blue-mk)"></div> Residences &amp; Settlements
    </div>
    ...
  </div>
```

Also remove the `.map-legend`, `.legend-item`, and `.legend-dot` CSS rules (~lines 136-153).

- [ ] **Step 4: Test era buttons**

Open `index.html` in a browser. Verify:
- Seven era buttons appear top-left on the map
- "Modern" button is active (gold border)
- Clicking "Eckweiler" switches to era 1: map flies to Germany, only Eckweiler-era markers visible, button highlights
- Info badge shows "Eckweiler Origins" and the era description
- On mobile viewport (Chrome DevTools → toggle device toolbar → iPhone 12 Pro): buttons scroll horizontally, info badge is tappable to expand

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat(map): add era button controls with info badge"
```

---

### Task 5: Add Timeline Slider Control

**Files:**
- Modify: `index.html` CSS section: add slider styles
- Modify: `index.html` JS section: add slider Leaflet control

- [ ] **Step 1: Add CSS for the timeline slider**

Add after the era control CSS:

```css
    /* ── Timeline slider ───────────────────────────────────────────── */
    .timeline-control {
      pointer-events: auto;
      background: linear-gradient(transparent, rgba(13,11,8,0.95));
      padding: 10px 16px 12px;
    }
    .timeline-row {
      display: flex;
      align-items: center;
      gap: 10px;
    }
    .timeline-year {
      font-size: 0.72rem;
      font-weight: bold;
      min-width: 32px;
      text-align: center;
    }
    .timeline-year-start { color: var(--gold); }
    .timeline-year-end { color: var(--text-dim); }
    .timeline-slider {
      flex: 1;
      -webkit-appearance: none;
      appearance: none;
      height: 4px;
      background: var(--border);
      border-radius: 2px;
      outline: none;
      cursor: pointer;
    }
    .timeline-slider::-webkit-slider-thumb {
      -webkit-appearance: none;
      width: 14px; height: 14px;
      background: var(--gold);
      border-radius: 50%;
      cursor: pointer;
      box-shadow: 0 0 6px rgba(232,201,122,0.4);
    }
    .timeline-slider::-moz-range-thumb {
      width: 14px; height: 14px;
      background: var(--gold);
      border: none;
      border-radius: 50%;
      cursor: pointer;
      box-shadow: 0 0 6px rgba(232,201,122,0.4);
    }
    .timeline-nav {
      display: flex;
      justify-content: center;
      gap: 12px;
      margin-top: 6px;
    }
    .timeline-nav-btn {
      background: none;
      border: none;
      color: var(--text-dim);
      font-size: 0.65rem;
      cursor: pointer;
      font-family: inherit;
      padding: 2px 6px;
    }
    .timeline-nav-btn:hover { color: var(--gold); }
    .timeline-hidden { display: none; }

    @media (max-width: 767px) {
      .timeline-control { padding: 8px 12px 10px; }
      .timeline-slider { height: 6px; }
      .timeline-slider::-webkit-slider-thumb { width: 20px; height: 20px; }
      .timeline-slider::-moz-range-thumb { width: 20px; height: 20px; }
      .timeline-nav-btn { font-size: 0.72rem; padding: 4px 8px; }
    }
```

- [ ] **Step 2: Add the timeline slider as a Leaflet control**

Add after the era control code in the JS section:

```javascript
  // ── Timeline slider control ─────────────────────────────────────
  var TimelineControl = L.Control.extend({
    options: { position: 'bottomleft' },
    onAdd: function() {
      var container = L.DomUtil.create('div', 'timeline-control');
      container.style.width = '100%';
      container.style.margin = '0';
      container.style.padding = '10px 16px 12px';
      L.DomEvent.disableClickPropagation(container);
      L.DomEvent.disableScrollPropagation(container);

      // Slider row
      var row = L.DomUtil.create('div', 'timeline-row', container);
      var yearStart = L.DomUtil.create('span', 'timeline-year timeline-year-start', row);
      var slider = L.DomUtil.create('input', 'timeline-slider', row);
      slider.type = 'range';
      var yearEnd = L.DomUtil.create('span', 'timeline-year timeline-year-end', row);

      // Nav buttons
      var nav = L.DomUtil.create('div', 'timeline-nav', container);
      var prevBtn = L.DomUtil.create('button', 'timeline-nav-btn', nav);
      prevBtn.textContent = '\u25c0 Prev Era';
      var playBtn = L.DomUtil.create('button', 'timeline-nav-btn', nav);
      playBtn.textContent = '\u25b6 Play';
      var nextBtn = L.DomUtil.create('button', 'timeline-nav-btn', nav);
      nextBtn.textContent = 'Next Era \u25b6';

      // Store refs
      container._slider = slider;
      container._yearStart = yearStart;
      container._yearEnd = yearEnd;
      container._playBtn = playBtn;
      container._row = row;

      // Initialize
      var era = ERAS.find(function(e) { return e.id === currentEra; });
      setSliderRange(container, era);

      // Slider input handler — filter markers by year within current era
      slider.addEventListener('input', function() {
        var yr = parseInt(slider.value);
        filterByYear(yr);
      });

      // Prev/Next
      prevBtn.addEventListener('click', function() {
        var idx = ERAS.findIndex(function(e) { return e.id === currentEra; });
        if (idx > 0) switchEra(ERAS[idx - 1].id);
      });
      nextBtn.addEventListener('click', function() {
        var idx = ERAS.findIndex(function(e) { return e.id === currentEra; });
        if (idx < ERAS.length - 1) switchEra(ERAS[idx + 1].id);
      });

      // Play
      playBtn.addEventListener('click', function() { togglePlay(); });

      return container;
    }
  });

  function setSliderRange(container, era) {
    var slider = container._slider;
    var isSingleYear = era.range[0] === era.range[1];
    if (isSingleYear) {
      container._row.classList.add('timeline-hidden');
    } else {
      container._row.classList.remove('timeline-hidden');
      slider.min = era.range[0];
      slider.max = era.range[1];
      slider.value = era.range[1]; // show all markers in era
    }
    container._yearStart.textContent = era.range[0];
    container._yearEnd.textContent = era.range[1];
  }

  var timelineControl = new TimelineControl();
  timelineControl.addTo(map);

  // Position the timeline to span the full bottom
  var tlContainer = timelineControl.getContainer();
  tlContainer.style.position = 'absolute';
  tlContainer.style.bottom = '0';
  tlContainer.style.left = '0';
  tlContainer.style.right = '0';
  tlContainer.style.width = '100%';
  tlContainer.style.boxSizing = 'border-box';
  // Remove Leaflet's default control margins
  tlContainer.parentElement.style.width = '100%';

  function updateSliderRange(era) {
    setSliderRange(tlContainer, era);
  }

  // Year filtering within an era
  function filterByYear(yr) {
    allMarkers.forEach(function(entry) {
      if (entry.data.eras.indexOf(currentEra) === -1) return; // not in this era
      var el = entry.marker.getElement && entry.marker.getElement();
      if (!el) return;
      if (entry.data.year > yr) {
        el.style.opacity = '0.15';
      } else {
        el.style.opacity = '1';
      }
    });
  }
```

- [ ] **Step 3: Test the slider**

Open `index.html` in a browser. Verify:
- Timeline slider appears at the bottom of the map
- Year labels show "1900" and "2026" (era 7)
- Clicking "Eckweiler" era button: slider updates to "1560"–"1620"
- Dragging slider within era 1: markers fade as their year exceeds the slider position
- Clicking "1736" era: slider row hides (single-year era), only nav buttons remain
- Prev/Next buttons navigate between eras

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat(map): add timeline slider with year filtering and prev/next navigation"
```

---

### Task 6: Add Play Mode

**Files:**
- Modify: `index.html` JS section: add play/pause functionality

- [ ] **Step 1: Implement play mode**

Add after the `filterByYear` function:

```javascript
  // ── Play mode ───────────────────────────────────────────────────
  var playInterval = null;
  var playTimeout = null;

  function togglePlay() {
    if (playInterval || playTimeout) {
      stopPlay();
    } else {
      startPlay();
    }
  }

  function startPlay() {
    tlContainer._playBtn.textContent = '\u23f8 Pause';
    var eraIdx = ERAS.findIndex(function(e) { return e.id === currentEra; });
    playEra(eraIdx);
  }

  function stopPlay() {
    clearInterval(playInterval);
    clearTimeout(playTimeout);
    playInterval = null;
    playTimeout = null;
    tlContainer._playBtn.textContent = '\u25b6 Play';
  }

  function playEra(idx) {
    if (idx >= ERAS.length) { stopPlay(); return; }
    var era = ERAS[idx];
    switchEra(era.id);

    var isSingleYear = era.range[0] === era.range[1];
    if (isSingleYear) {
      // Pause 5 seconds then move to next era
      playTimeout = setTimeout(function() { playEra(idx + 1); }, 5000);
    } else {
      // Scrub through the era
      var slider = tlContainer._slider;
      var yr = era.range[0];
      slider.value = yr;
      filterByYear(yr);
      playInterval = setInterval(function() {
        yr++;
        if (yr > era.range[1]) {
          clearInterval(playInterval);
          playInterval = null;
          // Pause 2 seconds at end of era, then next
          playTimeout = setTimeout(function() { playEra(idx + 1); }, 2000);
          return;
        }
        slider.value = yr;
        filterByYear(yr);
      }, 200);
    }
  }
```

- [ ] **Step 2: Test play mode**

Open `index.html`, click "Eckweiler" to start from era 1, then click "Play". Verify:
- Play button changes to "Pause"
- Slider scrubs forward through 1560–1620
- After era 1 completes, auto-advances to era 2 (map flies to Albisheim area)
- Click "Pause" — animation stops
- Click "Play" again — resumes from current era

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat(map): add play mode with auto-advance through eras"
```

---

### Task 7: Add Layers Panel Control

**Files:**
- Modify: `index.html` CSS section: add layers panel and bottom sheet styles
- Modify: `index.html` JS section: add layers control

- [ ] **Step 1: Add CSS for the layers panel**

Add after the timeline slider CSS:

```css
    /* ── Layers panel ──────────────────────────────────────────────── */
    .layers-toggle {
      pointer-events: auto;
      background: rgba(30,26,20,0.92);
      border: 1px solid var(--border);
      padding: 6px 10px;
      border-radius: 4px;
      cursor: pointer;
      font-size: 0.68rem;
      color: var(--text-dim);
      font-family: inherit;
    }
    .layers-toggle:hover { border-color: var(--sepia); color: var(--text); }
    .layers-panel {
      display: none;
      background: rgba(30,26,20,0.95);
      border: 1px solid var(--border);
      border-radius: 4px;
      padding: 12px;
      margin-top: 6px;
      min-width: 200px;
      pointer-events: auto;
    }
    .layers-panel.open { display: block; }
    .layers-section-title {
      font-size: 0.62rem;
      color: var(--gold);
      text-transform: uppercase;
      letter-spacing: 0.05em;
      margin: 8px 0 4px;
    }
    .layers-section-title:first-child { margin-top: 0; }
    .layers-option {
      display: flex;
      align-items: center;
      gap: 6px;
      font-size: 0.68rem;
      color: var(--text-dim);
      padding: 2px 0;
      cursor: pointer;
    }
    .layers-option input { accent-color: var(--gold); }
    .layers-opacity {
      display: flex;
      align-items: center;
      gap: 6px;
      margin-top: 4px;
      font-size: 0.62rem;
      color: var(--text-dim);
    }
    .layers-opacity input[type="range"] {
      -webkit-appearance: none;
      appearance: none;
      height: 3px;
      background: var(--border);
      border-radius: 2px;
      flex: 1;
    }
    .layers-opacity input[type="range"]::-webkit-slider-thumb {
      -webkit-appearance: none;
      width: 10px; height: 10px;
      background: var(--gold);
      border-radius: 50%;
    }

    /* Mobile bottom sheet */
    @media (max-width: 767px) {
      .layers-toggle span.layers-label { display: none; }
      .layers-panel {
        position: fixed;
        bottom: 0; left: 0; right: 0;
        border-radius: 12px 12px 0 0;
        max-height: 50vh;
        overflow-y: auto;
        z-index: 1000;
        margin-top: 0;
      }
      .layers-backdrop {
        position: fixed;
        inset: 0;
        background: rgba(0,0,0,0.5);
        z-index: 999;
        display: none;
      }
      .layers-backdrop.open { display: block; }
    }
```

- [ ] **Step 2: Add the layers control as a Leaflet control**

Add after the timeline control code in the JS:

```javascript
  // ── Layers control ──────────────────────────────────────────────
  var baseLayers = {
    'Dark': L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', {
      attribution: '\u00a9 OpenStreetMap \u00a9 CARTO', subdomains: 'abcd', maxZoom: 19
    }),
    'Light': L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {
      attribution: '\u00a9 OpenStreetMap \u00a9 CARTO', subdomains: 'abcd', maxZoom: 19
    }),
    'Satellite': L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', {
      attribution: '\u00a9 Esri', maxZoom: 19
    })
  };
  var activeBase = 'Dark'; // current base is already added from initial setup

  var LayersControl = L.Control.extend({
    options: { position: 'topright' },
    onAdd: function() {
      var wrapper = L.DomUtil.create('div', '');
      wrapper.style.pointerEvents = 'none';
      L.DomEvent.disableClickPropagation(wrapper);
      L.DomEvent.disableScrollPropagation(wrapper);

      // Toggle button
      var toggle = L.DomUtil.create('button', 'layers-toggle', wrapper);
      toggle.style.pointerEvents = 'auto';
      toggle.innerHTML = '\ud83d\uddfa\ufe0f <span class="layers-label">Layers \u25be</span>';

      // Backdrop (mobile)
      var backdrop = L.DomUtil.create('div', 'layers-backdrop');
      document.body.appendChild(backdrop);

      // Panel
      var panel = L.DomUtil.create('div', 'layers-panel', wrapper);

      // Base map section
      var baseTitle = L.DomUtil.create('div', 'layers-section-title', panel);
      baseTitle.textContent = 'Base Map';
      ['Dark', 'Light', 'Satellite'].forEach(function(name) {
        var label = L.DomUtil.create('label', 'layers-option', panel);
        var radio = L.DomUtil.create('input', '', label);
        radio.type = 'radio';
        radio.name = 'basemap';
        radio.checked = name === activeBase;
        label.appendChild(document.createTextNode(' ' + name));
        radio.addEventListener('change', function() {
          if (radio.checked) {
            map.eachLayer(function(l) {
              if (l._url && (l._url.indexOf('basemaps.cartocdn') !== -1 || l._url.indexOf('arcgisonline') !== -1)) {
                map.removeLayer(l);
              }
            });
            baseLayers[name].addTo(map);
            activeBase = name;
          }
        });
      });

      // Data layers section
      var dataTitle = L.DomUtil.create('div', 'layers-section-title', panel);
      dataTitle.textContent = 'Data';
      var dataLayerNames = [
        { key: 'residences', label: 'Residences' },
        { key: 'churches', label: 'Churches' },
        { key: 'cemeteries', label: 'Cemeteries' },
        { key: 'repos', label: 'Archives' },
        { key: 'paths', label: 'Migration Paths' }
      ];
      dataLayerNames.forEach(function(dl) {
        var label = L.DomUtil.create('label', 'layers-option', panel);
        var cb = L.DomUtil.create('input', '', label);
        cb.type = 'checkbox';
        cb.checked = true;
        label.appendChild(document.createTextNode(' ' + dl.label));
        cb.addEventListener('change', function() {
          if (cb.checked) {
            layers[dl.key].addTo(map);
          } else {
            map.removeLayer(layers[dl.key]);
          }
        });
      });

      // Toggle logic
      toggle.addEventListener('click', function() {
        panel.classList.toggle('open');
        backdrop.classList.toggle('open');
      });
      backdrop.addEventListener('click', function() {
        panel.classList.remove('open');
        backdrop.classList.remove('open');
      });

      return wrapper;
    }
  });

  new LayersControl().addTo(map);
```

- [ ] **Step 3: Remove the initial tile layer that was added directly to the map**

The original tile layer is added directly on lines ~2320-2324. Since we now manage base layers through the layers control, remove that initial `L.tileLayer(...)addTo(map)` call and instead add the Dark base layer from our `baseLayers` object. Replace lines 2320-2324 with:

```javascript
  // Base layer added via baseLayers object (managed by layers control)
  // Initial base layer added after baseLayers is defined — see layers control section
```

Then after the `baseLayers` object is defined (and before the LayersControl), add:

```javascript
  baseLayers['Dark'].addTo(map);
```

- [ ] **Step 4: Test layers panel**

Open `index.html` in a browser. Verify:
- "Layers" button appears top-right on the map
- Clicking it opens a dropdown panel with Base Map radios and Data checkboxes
- Switching to "Light" base: map tiles change to light theme
- Switching to "Satellite": aerial imagery appears
- Unchecking "Churches": gold church markers disappear
- Re-checking: they reappear
- Mobile viewport: layers panel slides up as a bottom sheet with backdrop

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat(map): add layers control with base map switching and data layer toggles"
```

---

### Task 8: Update Map Sizing and Popup Padding

**Files:**
- Modify: `index.html` CSS section: update `#map` height
- Modify: `index.html` JS section: add autoPanPadding to map init

- [ ] **Step 1: Update map CSS height**

Replace the existing `#map` CSS block:

```css
    #map {
      width: 100%;
      height: 580px;
      border-radius: 6px;
      border: 1px solid var(--border);
      background: #1a1610;
    }
```

With:

```css
    #map {
      width: 100%;
      height: 70vh;
      min-height: 500px;
      max-height: 800px;
      border-radius: 6px;
      border: 1px solid var(--border);
      background: #1a1610;
    }
    @media (max-width: 767px) {
      #map {
        height: 60vh;
        min-height: 350px;
        max-height: none;
      }
    }
```

- [ ] **Step 2: Add autoPanPadding to map initialization**

Update the map initialization line from:

```javascript
  const map = L.map('map', { zoomControl: true }).setView([42, -30], 3);
```

To:

```javascript
  const map = L.map('map', {
    zoomControl: true,
    autoPanPaddingBottomRight: [10, 80]
  }).setView([42, -30], 3);
```

- [ ] **Step 3: Test sizing**

Open `index.html` in a browser. Verify:
- Map is taller than before (~70vh)
- Resize browser window: map height adjusts responsively
- Mobile viewport: map is ~60vh
- Click a marker near the bottom: popup auto-pans to stay above the timeline slider

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat(map): responsive map sizing and popup autoPan padding"
```

---

### Task 9: Research and Add Historic Tile Overlays

**Files:**
- Modify: `index.html` JS section: add HISTORIC_TILES config and overlay management

This task involves web research to find available free tile services, then integrating what works.

- [ ] **Step 1: Research available historic map tile services**

Search for freely available WMS/XYZ tile services for these areas:

1. **Rhineland/Palatinate (Germany):** Check these URLs:
   - Geobasis NRW WMS: `https://www.wms.nrw.de/geobasis/wms_nw_uraufnahme` (Uraufnahme ~1836-50)
   - Geobasis NRW Tranchot: `https://www.wms.nrw.de/geobasis/wms_nw_tranchot` (1801-1828)
   - Tim Online NRW: `https://www.tim-online.nrw.de/` (has historic map layers)
   - Note: NRW services cover North Rhine-Westphalia; the Palatinate (RLP) may need `https://geo5.service24.rlp.de/` or the Geoportal RLP WMS

2. **Pennsylvania colonial maps:** Check David Rumsey georectified tiles at `https://www.davidrumsey.com/luna/servlet/view/search?q=scull+pennsylvania`

3. **North Carolina colonial maps:** Check David Rumsey for Mouzon 1775, Collet 1770

Test each URL by constructing a Leaflet `L.tileLayer.wms()` call and loading it in the browser. Document which ones actually serve tiles for the areas of interest.

- [ ] **Step 2: Add HISTORIC_TILES config with working overlays**

Add after the ERAS config:

```javascript
  // ── Historic tile overlays ──────────────────────────────────────
  // Populated with confirmed working tile services.
  // If a service doesn't work, comment it out and add a note.
  var HISTORIC_TILES = [
    // Add entries here as confirmed. Example format:
    // {
    //   eras: [3],
    //   layer: L.tileLayer.wms('https://example.com/wms', {
    //     layers: 'layer_name', format: 'image/png', transparent: true, opacity: 0.6
    //   }),
    //   label: 'Tranchot-Müffling 1803–20',
    //   defaultOpacity: 0.6
    // }
  ];
  var activeOverlays = [];
```

- [ ] **Step 3: Wire historic overlays into era switching**

Update the `switchEra` function — add this block at the end, before the UI update calls:

```javascript
    // Swap historic overlays
    activeOverlays.forEach(function(ol) { map.removeLayer(ol.layer); });
    activeOverlays = [];
    HISTORIC_TILES.forEach(function(ht) {
      if (ht.eras.indexOf(currentEra) !== -1) {
        ht.layer.setOpacity(0);
        ht.layer.addTo(map);
        activeOverlays.push(ht);
        // Fade in
        var target = ht.defaultOpacity;
        var current = 0;
        var step = target / 25; // 25 frames over ~500ms
        var fadeIn = setInterval(function() {
          current += step;
          if (current >= target) { current = target; clearInterval(fadeIn); }
          ht.layer.setOpacity(current);
        }, 20);
      }
    });
```

- [ ] **Step 4: Add historic overlay toggles to the layers panel**

In the LayersControl `onAdd` method, add a "Historic Overlay" section between "Base Map" and "Data". This section dynamically shows overlays for the current era:

```javascript
      // Historic overlay section
      var histTitle = L.DomUtil.create('div', 'layers-section-title', panel);
      histTitle.textContent = 'Historic Overlay';
      var histContainer = L.DomUtil.create('div', '', panel);
      histContainer.id = 'hist-overlay-options';

      // This gets rebuilt when era changes
      wrapper._histContainer = histContainer;
```

Add a function to rebuild the historic overlay options:

```javascript
  function rebuildHistoricOverlayUI() {
    var container = document.getElementById('hist-overlay-options');
    if (!container) return;
    container.innerHTML = '';
    var hasOverlays = false;
    HISTORIC_TILES.forEach(function(ht, idx) {
      if (ht.eras.indexOf(currentEra) === -1) return;
      hasOverlays = true;
      var label = L.DomUtil.create('label', 'layers-option', container);
      var cb = L.DomUtil.create('input', '', label);
      cb.type = 'checkbox';
      cb.checked = activeOverlays.indexOf(ht) !== -1;
      label.appendChild(document.createTextNode(' ' + ht.label));
      cb.addEventListener('change', function() {
        if (cb.checked) {
          ht.layer.addTo(map);
          ht.layer.setOpacity(ht.defaultOpacity);
          activeOverlays.push(ht);
        } else {
          map.removeLayer(ht.layer);
          activeOverlays = activeOverlays.filter(function(o) { return o !== ht; });
        }
      });
      // Opacity slider
      var opRow = L.DomUtil.create('div', 'layers-opacity', container);
      opRow.innerHTML = 'Opacity ';
      var opSlider = L.DomUtil.create('input', '', opRow);
      opSlider.type = 'range';
      opSlider.min = '0'; opSlider.max = '100';
      opSlider.value = String(ht.defaultOpacity * 100);
      opSlider.addEventListener('input', function() {
        ht.layer.setOpacity(parseInt(opSlider.value) / 100);
      });
    });
    if (!hasOverlays) {
      container.innerHTML = '<div style="font-size:0.62rem;color:var(--text-dim);padding:2px 0;">No historic map available for this era</div>';
    }
  }
```

Add this line at the end of the `updateEraUI` function body (defined in Task 4):

```javascript
    rebuildHistoricOverlayUI();
```

This is safe because `rebuildHistoricOverlayUI` is called at runtime (not at parse time), so it doesn't matter that it's defined later in the file.

- [ ] **Step 5: Test with and without overlays**

If any overlays were found in Step 1, verify they load correctly when switching to the relevant era. Verify the opacity slider works. If no overlays were found, verify the "No historic map available" message shows for all eras.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat(map): add historic tile overlay infrastructure and layers panel integration"
```

---

### Task 10: Add Harle Route Draw Animation

**Files:**
- Modify: `index.html` JS section: add polyline draw animation for era 4

- [ ] **Step 1: Add the draw animation function**

Add after the play mode code:

```javascript
  // ── Polyline draw animation ─────────────────────────────────────
  function animatePolyline(polyline, duration) {
    if (reducedMotion) return;
    var path = polyline.getElement();
    if (!path) return;
    var svg = path.closest('svg');
    if (!svg) return;
    var pathEl = path.querySelector('path') || path;
    if (!pathEl.getTotalLength) return;
    var length = pathEl.getTotalLength();
    pathEl.style.strokeDasharray = length;
    pathEl.style.strokeDashoffset = length;
    pathEl.style.transition = 'stroke-dashoffset ' + duration + 'ms ease-in-out';
    requestAnimationFrame(function() {
      requestAnimationFrame(function() {
        pathEl.style.strokeDashoffset = '0';
      });
    });
  }
```

- [ ] **Step 2: Trigger animation on era 4**

At the end of the `switchEra` function, add:

```javascript
    // Animate Harle route on era 4
    if (eraId === 4) {
      setTimeout(function() {
        animatePolyline(transatlanticPolyline, 2000);
      }, 1600); // wait for flyTo to finish
    }
```

- [ ] **Step 3: Test the animation**

Click "1736" era button. Verify: after the map finishes flying to the Atlantic view, the orange dashed transatlantic route draws itself from Albisheim to Bethania over ~2 seconds.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat(map): add animated draw effect for transatlantic route in era 4"
```

---

### Task 11: Final Integration Testing and Polish

**Files:**
- Modify: `index.html` (potential minor fixes found during testing)

- [ ] **Step 1: Desktop full walkthrough**

Open `index.html` in Chrome at full desktop width. Test this exact sequence:

1. Page loads → map shows all markers (era 7), "Modern" button active
2. Click "Eckweiler" → map flies to Germany, only era-1 markers visible
3. Drag slider from 1560 to 1590 → Eckweiler markers at or below 1590 are bright, others faded
4. Click "Next Era" → transitions to era 2 (Albisheim), slider range updates
5. Click "1736" → map flies to Atlantic, ship appears, slider hides (single-year era)
6. Click Harle ship marker → popup opens, auto-pans above timeline
7. Click "Layers" → panel opens, switch to "Light" base map, uncheck "Archives"
8. Click "Eckweiler" again to go back to era 1 → base map stays Light, archives stay hidden
9. Click "Modern" → all markers return, check "Archives" back on
10. Click "Eckweiler" then "Play" → auto-plays through all 7 eras

Fix any bugs found.

- [ ] **Step 2: Mobile full walkthrough**

Open Chrome DevTools, toggle device toolbar, select iPhone 12 Pro (390×844). Test:

1. Era buttons scroll horizontally — swipe to see all 7
2. Info badge shows era name only; tap to expand description
3. Timeline slider thumb is large enough to drag with finger
4. Layers button shows icon only; tap opens bottom sheet with backdrop
5. Tapping backdrop closes bottom sheet
6. Popups don't hide behind the timeline slider (autoPanPadding)
7. Play mode works

Fix any bugs found.

- [ ] **Step 3: Test reduced motion preference**

In Chrome DevTools → Rendering tab → "Emulate CSS media feature prefers-reduced-motion" → select "reduce". Switch eras and verify:
- Map uses `setView` (instant) instead of `flyTo` (animated)
- Marker fades are fast (100ms)

- [ ] **Step 4: Final commit**

```bash
git add index.html
git commit -m "feat(map): complete historical map timeline with era navigation, slider, layers, and animations"
```
