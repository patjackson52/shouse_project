# Family Tree & Map Updates Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a D3.js interactive zoomable family tree tab, update the Leaflet map with new markers/ship route, and deploy to GitHub Pages.

**Architecture:** Single-file site (`index.html`) with inline CSS and JS. D3.js v7 loaded from CDN. Family tree data as embedded JSON. Leaflet map already exists — we add markers and modify existing ones. GitHub Pages serves from `main` branch root.

**Tech Stack:** HTML/CSS/JS (inline), D3.js v7 (CDN), Leaflet 1.9.4 (already loaded), GitHub Pages

**File:** `/Users/patrick/shouse_project/index.html` — all tasks modify this single file unless noted.

---

### Task 1: Add D3.js CDN and Family Tree CSS

**Files:**
- Modify: `index.html:8-11` (head, add D3 script tag after Leaflet)
- Modify: `index.html:~420-435` (CSS, add tree styles before footer CSS comment)

- [ ] **Step 1: Add D3.js script tag in `<head>` after the Leaflet script**

Insert after line 10 (`<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>`):

```html
  <!-- D3.js -->
  <script src="https://unpkg.com/d3@7/dist/d3.min.js"></script>
```

- [ ] **Step 2: Add all family tree CSS before the `/* ── Footer */` comment**

Insert before the `/* ── Footer ─────` CSS comment. The full CSS block:

```css
    /* ── Family Tree ────────────────────────────────────────── */
    #tree-section { padding-bottom: 20px; }
    #tree-container {
      position: relative;
      width: 100%;
      height: 700px;
      background: var(--bg);
      border: 1px solid var(--border);
      border-radius: 6px;
      overflow: hidden;
    }
    #tree-container svg { width: 100%; height: 100%; }
    .tree-legend {
      display: flex;
      flex-wrap: wrap;
      gap: 16px;
      margin-bottom: 18px;
    }
    .tree-legend .legend-item {
      display: flex;
      align-items: center;
      gap: 8px;
      font-size: 0.82rem;
      color: var(--text-dim);
    }
    .tree-legend .legend-dot {
      width: 13px; height: 13px;
      border-radius: 3px;
      border: 2px solid rgba(255,255,255,0.25);
      flex-shrink: 0;
    }
    .tree-controls {
      position: absolute;
      bottom: 14px;
      right: 14px;
      display: flex;
      gap: 6px;
      z-index: 10;
    }
    .tree-controls button {
      width: 32px; height: 32px;
      background: var(--bg2);
      color: var(--gold);
      border: 1px solid var(--border);
      border-radius: 4px;
      font-size: 1rem;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: background 0.2s;
    }
    .tree-controls button:hover { background: var(--bg3); }
    .tree-controls .tree-reset {
      width: auto;
      padding: 0 10px;
      font-size: 0.72rem;
      letter-spacing: 0.1em;
      text-transform: uppercase;
    }
    .tree-detail {
      position: absolute;
      bottom: 14px;
      right: 60px;
      width: 320px;
      max-height: 400px;
      overflow-y: auto;
      background: var(--bg2);
      border: 1px solid var(--border);
      border-radius: 6px;
      padding: 18px;
      z-index: 20;
      display: none;
      box-shadow: 0 4px 24px rgba(0,0,0,0.6);
    }
    .tree-detail.active { display: block; }
    .tree-detail__close {
      position: absolute;
      top: 8px; right: 10px;
      background: none;
      border: none;
      color: var(--text-dim);
      font-size: 1.2rem;
      cursor: pointer;
    }
    .tree-detail__close:hover { color: var(--gold); }
    .tree-detail h3 {
      font-size: 1rem;
      color: var(--gold-lt);
      font-weight: normal;
      margin-bottom: 4px;
      padding-right: 24px;
    }
    .tree-detail__tag {
      display: inline-block;
      font-size: 0.68rem;
      padding: 2px 7px;
      border-radius: 3px;
      letter-spacing: 0.06em;
      margin-bottom: 10px;
    }
    .tree-detail__tag--stayed { background: #2a3a2a; color: #6cbf6c; }
    .tree-detail__tag--emigrated { background: #3a2a2a; color: #d9845a; }
    .tree-detail__tag--german { background: #3a3520; color: var(--gold); }
    .tree-detail__tag--american { background: #1e2a3a; color: #4a90d9; }
    .tree-detail p { font-size: 0.82rem; color: var(--text-dim); line-height: 1.6; margin-bottom: 6px; }
    .tree-detail strong { color: var(--text); }
    .tree-detail a { color: var(--gold); text-decoration: none; border-bottom: 1px solid var(--sepia); }
    .tree-detail a:hover { color: var(--gold-lt); }
```

- [ ] **Step 3: Verify the CSS is inserted correctly**

Run: `node -e "const h=require('fs').readFileSync('index.html','utf8'); console.log(h.includes('#tree-container') && h.includes('d3.min.js') ? 'OK' : 'FAIL')"`

Expected: `OK`

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add D3.js CDN and family tree CSS styles"
```

---

### Task 2: Add Family Tree Nav Link and Section HTML

**Files:**
- Modify: `index.html:451` (nav — add Family Tree link)
- Modify: `index.html:~1024` (insert new section before TIMELINE comment)

- [ ] **Step 1: Add "Family Tree" nav link after "German Origins"**

Change the nav from:
```html
  <a href="#german-origins-section">German Origins</a>
  <a href="#timeline-section">Timeline</a>
```

To:
```html
  <a href="#german-origins-section">German Origins</a>
  <a href="#tree-section">Family Tree</a>
  <a href="#timeline-section">Timeline</a>
```

- [ ] **Step 2: Insert the Family Tree section HTML before the TIMELINE comment**

Insert immediately before `<!-- ═══════════════════════════════ TIMELINE ════════════════════════════ -->`:

```html
<!-- ═══════════════════════════════ FAMILY TREE ═════════════════════════ -->
<section id="tree-section">
  <div class="section-label">Genealogy</div>
  <h2>Schauss / Shouse Family Tree</h2>
  <p style="color:var(--text-dim);margin-bottom:20px;max-width:760px;">
    Ten generations from Eckweiler (c. 1560) to North Carolina (1900). Click any
    person to see details, sources, and connections. Scroll to zoom, drag to pan.
    The tree forks where the family split &mdash; one branch stayed in Albisheim,
    the other emigrated to America in 1736.
  </p>

  <div class="tree-legend">
    <div class="legend-item">
      <div class="legend-dot" style="background:#c9a84c"></div> German-born
    </div>
    <div class="legend-item">
      <div class="legend-dot" style="background:#4caf7d"></div> Stayed in Germany
    </div>
    <div class="legend-item">
      <div class="legend-dot" style="background:#e8703a"></div> Emigrated
    </div>
    <div class="legend-item">
      <div class="legend-dot" style="background:#4a90d9"></div> Born in America
    </div>
    <div class="legend-item">
      <div class="legend-dot" style="background:#8b6f47"></div> Connected Families
    </div>
  </div>

  <div id="tree-container">
    <svg id="tree-svg"></svg>
    <div class="tree-controls">
      <button id="tree-zoom-in" title="Zoom in">+</button>
      <button id="tree-zoom-out" title="Zoom out">&minus;</button>
      <button id="tree-reset" class="tree-reset" title="Reset view">Reset</button>
    </div>
    <div id="tree-detail" class="tree-detail">
      <button class="tree-detail__close" id="tree-detail-close">&times;</button>
      <div id="tree-detail-content"></div>
    </div>
  </div>
</section>

```

- [ ] **Step 3: Verify section exists**

Run: `node -e "const h=require('fs').readFileSync('index.html','utf8'); console.log(h.includes('id=\"tree-section\"') && h.includes('id=\"tree-svg\"') ? 'OK' : 'FAIL')"`

Expected: `OK`

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add family tree section HTML and nav link"
```

---

### Task 3: Embed Family Tree Data as JSON

**Files:**
- Modify: `index.html` — insert a new `<script>` block after the tree section HTML, before the TIMELINE comment

- [ ] **Step 1: Insert the family tree data script block**

Insert immediately after the closing `</section>` of the tree section and before the TIMELINE comment. This is a `<script>` tag containing the `TREE_DATA` constant — the complete family JSON. Every person from the spec is included with all researched dates, places, and sources.

```html
<script>
// ═══════════════════════════════ FAMILY TREE DATA ═══════════════════════
const TREE_DATA = {
  id: "hanss-schauss", name: "Han\u00df Schau\u00df", birth: "before 1600", birthPlace: "Eckweiler, Bad Kreuznach", death: "unknown", deathPlace: "", occupation: "", tag: "german",
  marriage: { spouse: "Catharina Von Allenfeldt", spouseBirth: "c. 1560", spouseDeath: "Apr 5, 1609", date: "", place: "Eckweiler" },
  sources: [{ label: "WikiTree \u2014 Melchior Schau\u00df", url: "https://www.wikitree.com/wiki/Schau%C3%9F-85" }],
  children: [
    {
      id: "melchior-schauss", name: "Melchior Schau\u00df", birth: "Feb 24, 1595", birthPlace: "Eckweiler", death: "Jun 24, 1628", deathPlace: "Eckweiler", occupation: "", tag: "german",
      marriage: { spouse: "Helena Ante\u00dfes", spouseBirth: "", spouseDeath: "", date: "Dec 4, 1621", place: "Eckweiler" },
      sources: [{ label: "WikiTree", url: "https://www.wikitree.com/wiki/Schau%C3%9F-85" }, { label: "Eckweiler Kirchenbuch 1568\u20131798", url: "" }],
      children: [
        {
          id: "johannes-schauss", name: "Johannes Schauss", birth: "c. Jul 23, 1620", birthPlace: "B\u00fcdingen, Lorraine", death: "c. Feb 27, 1674", deathPlace: "B\u00fcdingen", occupation: "", tag: "german",
          marriage: { spouse: "Maria Magdalena Klein", spouseBirth: "1625", spouseDeath: "", date: "Jan 30, 1644", place: "Evangelische, Albisheim" },
          sources: [{ label: "WikiTree", url: "https://www.wikitree.com/wiki/Schauss-18" }],
          children: [
            {
              id: "hans-bernhardt", name: "Hans Bernhardt Schauss", birth: "c. 1650", birthPlace: "B\u00fcdingen", death: "Aug 31, 1720", deathPlace: "Albisheim", occupation: "", tag: "german",
              marriage: { spouse: "Anna Margarethe Naass", spouseBirth: "Mar 1, 1657, Albisheim", spouseDeath: "Aug 31, 1720, Albisheim", date: "Apr 21, 1674", place: "Albisheim" },
              sources: [{ label: "WikiTree", url: "https://www.wikitree.com/wiki/Schauss-17" }, { label: "Albisheim Kirchenbuch", url: "" }],
              children: [
                {
                  id: "johann-conrad-sr", name: "Johann Conrad Schauss Sr.", birth: "Jan 2, 1675", birthPlace: "Albisheim", death: "Apr 27, 1734", deathPlace: "Albisheim", occupation: "", tag: "german",
                  marriage: { spouse: "Anna Engle-Burgis Conrad", spouseBirth: "Oct 30, 1680, Albisheim", spouseDeath: "c. 1712, Albisheim", date: "Aug 29, 1699", place: "Evangelische, Albisheim" },
                  sources: [{ label: "WikiTree", url: "https://www.wikitree.com/wiki/Schauss-16" }, { label: "FHL Microfilm 193,748", url: "" }, { label: "Weller Dissertation (2004)", url: "https://www.worldcat.org/title/781727046" }],
                  children: [
                    {
                      id: "johann-adam", name: "Johann Adam Schauss", birth: "c. 1703/1704", birthPlace: "Immesheim, Pfalz", death: "Dec 12, 1770", deathPlace: "Bethania, NC", occupation: "Wheelwright & Miller", tag: "emigrated",
                      badge: "Emigrated 1736 on Ship Harle",
                      marriage: { spouse: "Maria Barbara Baum", spouseBirth: "Jan 8, 1707, Albisheim", spouseDeath: "before 1768, Easton, PA", date: "Jan 16, 1725", place: "Albisheim" },
                      sources: [{ label: "WikiTree", url: "https://www.wikitree.com/wiki/Schauss-12" }, { label: "Ship Harle Passenger List (#94)", url: "https://www.olivetreegenealogy.com/ships/palship24.shtml" }, { label: "PA German Pioneers, Vol. 1", url: "" }, { label: "Weller Dissertation (2004)", url: "https://www.worldcat.org/title/781727046" }],
                      children: [
                        { id: "magdalena", name: "Maria Magdalena Margaretha", birth: "Sep 25, 1725", birthPlace: "Albisheim", death: "", deathPlace: "", tag: "emigrated", sources: [], children: [] },
                        { id: "friedrich-christoph", name: "Friedrich Christoph Schauss", birth: "Apr 13, 1727", birthPlace: "Albisheim", death: "", deathPlace: "", tag: "emigrated", sources: [], children: [] },
                        {
                          id: "philip-nicolaus", name: "Philip Nicolaus Schauss", birth: "Dec 17, 1728", birthPlace: "Immesheim", death: "1810", deathPlace: "Stokes Co., NC", tag: "emigrated",
                          marriage: { spouse: "Maria Catherine Beck", spouseBirth: "", spouseDeath: "", date: "Jul 19, 1748", place: "Jordan Ev. Lutheran Church" },
                          sources: [{ label: "Geni.com", url: "https://www.geni.com/people/Philip-Nicolaus-Schauss/6000000027376082268" }], children: []
                        },
                        { id: "anna-maria-gertraud", name: "Anna Maria Gertraud", birth: "Jul 18, 1731", birthPlace: "Immesheim", death: "", deathPlace: "", tag: "emigrated", sources: [], children: [] },
                        { id: "johann-show", name: "Johann (Show)", birth: "Jan 7, 1737", birthPlace: "Pennsylvania", death: "", deathPlace: "", tag: "american", sources: [], children: [] },
                        { id: "conrad", name: "Conrad Schauss", birth: "Jan 7, 1738", birthPlace: "Falkner Swamp, PA", death: "", deathPlace: "", tag: "american", sources: [], children: [] },
                        { id: "anna-margaretha-child", name: "Anna Margaretha Schauss", birth: "Jul 18, 1740", birthPlace: "Falkner Swamp, PA", death: "", deathPlace: "", tag: "american", sources: [], children: [] },
                        { id: "heinrich-sr", name: "Heinrich Schauss Sr.", birth: "1741", birthPlace: "Falkner Swamp, PA", death: "", deathPlace: "", tag: "american", sources: [], children: [] },
                        { id: "gottlieb", name: "Gottlieb Schauss", birth: "Dec 14, 1744", birthPlace: "Bethlehem, PA", death: "", deathPlace: "", tag: "american", sources: [], children: [] },
                        { id: "beniginia", name: "Beniginia Schauss", birth: "Dec 1746", birthPlace: "Bethlehem, PA", death: "", deathPlace: "", tag: "american", sources: [], children: [] },
                        { id: "christian", name: "Christian Shouse", birth: "Jul 11, 1748", birthPlace: "Whitehall, Bucks Co., PA", death: "", deathPlace: "", tag: "american", sources: [], children: [] }
                      ]
                    }
                  ]
                },
                {
                  id: "johannes-1678", name: "Johannes Schauss", birth: "Feb 17, 1678", birthPlace: "Albisheim", death: "", deathPlace: "", occupation: "", tag: "german",
                  marriage: { spouse: "Maria Barbara Meyers", spouseBirth: "", spouseDeath: "", date: "Apr 14, 1705", place: "Albisheim" },
                  sources: [{ label: "Albisheim Kirchenbuch", url: "" }], children: []
                },
                { id: "anna-margaretha-1680", name: "Anna Margaretha Schauss", birth: "Oct 31, 1680", birthPlace: "Albisheim", death: "", deathPlace: "", tag: "german", sources: [], children: [] },
                { id: "hans-adam-1683", name: "Hans Adam Schauss", birth: "Jun 10, 1683", birthPlace: "Albisheim", death: "", deathPlace: "", tag: "german", sources: [], children: [] },
                {
                  id: "philipp-heinrich", name: "Philipp Heinrich Schauss", birth: "Feb 3, 1686", birthPlace: "Albisheim", death: "Apr 11, 1765", deathPlace: "Albisheim", occupation: "Linen Weaver & Mayor of Albisheim", tag: "stayed",
                  badge: "Stayed in Germany \u2014 Mayor of Albisheim",
                  marriage: { spouse: "Anna Hedwig Waltherin", spouseBirth: "", spouseDeath: "", date: "", place: "Albisheim" },
                  sources: [{ label: "Ancestry", url: "https://www.ancestry.com/genealogy/records/phillip-heinrich-schauss-24-1kycqf4" }, { label: "Ortsfamilienbuch Albisheim (Uhrig, 2009)", url: "" }],
                  children: []
                }
              ]
            }
          ]
        }
      ]
    }
  ]
};
</script>

```

- [ ] **Step 2: Verify data parses correctly**

Run: `node -e "const fs=require('fs'); const h=fs.readFileSync('index.html','utf8'); const m=h.match(/const TREE_DATA = ({[\s\S]*?});/); if(m){try{const d=eval('('+m[1]+')'); console.log('Root:', d.name, '| Children:', d.children.length); console.log('OK')}catch(e){console.log('PARSE ERROR:', e.message)}}else{console.log('NOT FOUND')}"`

Expected: `Root: Hanß Schauß | Children: 1` then `OK`

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: embed complete family tree data as JSON (~45 nodes)"
```

---

### Task 4: Implement D3.js Family Tree Rendering

**Files:**
- Modify: `index.html` — add a new `<script>` block after the TREE_DATA script, before the TIMELINE comment

- [ ] **Step 1: Add the complete D3 tree rendering script**

Insert a new `<script>` block immediately after the TREE_DATA `</script>` closing tag:

```html
<script>
// ═══════════════════════════════ FAMILY TREE RENDERER ══════════════════
(function () {
  const TAG_COLORS = {
    german:    '#c9a84c',
    stayed:    '#4caf7d',
    emigrated: '#e8703a',
    american:  '#4a90d9',
    connected: '#8b6f47'
  };

  const NODE_W = 200, NODE_H = 70, SPOUSE_W = 160, SPOUSE_H = 55;
  const container = document.getElementById('tree-container');
  const svg = d3.select('#tree-svg');
  const g = svg.append('g').attr('id', 'tree-g');

  // ── Zoom ────────────────────────────────────────────────────
  const zoom = d3.zoom()
    .scaleExtent([0.15, 3])
    .on('zoom', (e) => g.attr('transform', e.transform));
  svg.call(zoom);

  // ── Zoom controls ───────────────────────────────────────────
  document.getElementById('tree-zoom-in').addEventListener('click', () => svg.transition().duration(300).call(zoom.scaleBy, 1.4));
  document.getElementById('tree-zoom-out').addEventListener('click', () => svg.transition().duration(300).call(zoom.scaleBy, 0.7));
  document.getElementById('tree-reset').addEventListener('click', resetView);

  // ── Detail panel ────────────────────────────────────────────
  const detailEl = document.getElementById('tree-detail');
  const detailContent = document.getElementById('tree-detail-content');
  document.getElementById('tree-detail-close').addEventListener('click', () => detailEl.classList.remove('active'));
  svg.on('click', (e) => { if (e.target === svg.node()) detailEl.classList.remove('active'); });

  function showDetail(d) {
    const p = d.data;
    let html = `<h3>${p.name}</h3>`;
    if (p.badge) {
      const cls = p.tag === 'stayed' ? 'stayed' : p.tag === 'emigrated' ? 'emigrated' : p.tag === 'american' ? 'american' : 'german';
      html += `<div class="tree-detail__tag tree-detail__tag--${cls}">${p.badge}</div>`;
    } else if (p.tag) {
      const labels = { german: 'German-born', stayed: 'Stayed in Germany', emigrated: 'Emigrated to America', american: 'Born in America' };
      const cls = p.tag;
      if (labels[cls]) html += `<div class="tree-detail__tag tree-detail__tag--${cls}">${labels[cls]}</div>`;
    }
    html += `<p><strong>Born:</strong> ${p.birth || 'unknown'}${p.birthPlace ? ', ' + p.birthPlace : ''}</p>`;
    if (p.death) html += `<p><strong>Died:</strong> ${p.death}${p.deathPlace ? ', ' + p.deathPlace : ''}</p>`;
    if (p.occupation) html += `<p><strong>Occupation:</strong> ${p.occupation}</p>`;
    if (p.marriage) {
      html += `<p style="margin-top:8px;"><strong>Marriage:</strong></p>`;
      html += `<p>${p.marriage.spouse}`;
      if (p.marriage.date) html += `<br>${p.marriage.date}`;
      if (p.marriage.place) html += `, ${p.marriage.place}`;
      html += `</p>`;
      if (p.marriage.spouseBirth) html += `<p style="font-size:0.78rem;"><em>Spouse b. ${p.marriage.spouseBirth}</em></p>`;
    }
    if (p.sources && p.sources.length) {
      html += `<p style="margin-top:10px;"><strong>Sources:</strong></p>`;
      p.sources.forEach(s => {
        html += s.url ? `<p>\u2022 <a href="${s.url}" target="_blank">${s.label}</a></p>` : `<p>\u2022 ${s.label}</p>`;
      });
    }
    detailContent.innerHTML = html;
    detailEl.classList.add('active');
  }

  // ── Build hierarchy ─────────────────────────────────────────
  const root = d3.hierarchy(TREE_DATA);

  // Collapse all children beyond depth 3 (generations 1-4 visible)
  root.descendants().forEach(d => {
    if (d.depth >= 3 && d.children) {
      d._children = d.children;
      d.children = null;
    }
  });

  const treeLayout = d3.tree().nodeSize([240, 140]);

  function update(source) {
    treeLayout(root);
    const nodes = root.descendants();
    const links = root.links();
    const duration = 400;

    // ── Links ──────────────────────────────────────────────
    const link = g.selectAll('.tree-link').data(links, d => d.target.data.id);

    const linkEnter = link.enter().append('path')
      .attr('class', 'tree-link')
      .attr('fill', 'none')
      .attr('stroke', '#5a4a30')
      .attr('stroke-width', 1.5)
      .attr('d', () => {
        const o = { x: source.x0 || source.x, y: source.y0 || source.y };
        return diagonal({ source: o, target: o });
      });

    link.merge(linkEnter).transition().duration(duration)
      .attr('d', d => diagonal(d));

    link.exit().transition().duration(duration)
      .attr('d', () => {
        const o = { x: source.x, y: source.y };
        return diagonal({ source: o, target: o });
      }).remove();

    // ── Nodes ──────────────────────────────────────────────
    const node = g.selectAll('.tree-node').data(nodes, d => d.data.id);

    const nodeEnter = node.enter().append('g')
      .attr('class', 'tree-node')
      .attr('transform', `translate(${source.x0 || source.x},${source.y0 || source.y})`)
      .attr('cursor', 'pointer')
      .on('click', (e, d) => {
        e.stopPropagation();
        if (d.children) { d._children = d.children; d.children = null; }
        else if (d._children) { d.children = d._children; d._children = null; }
        update(d);
        showDetail(d);
      });

    nodeEnter.append('rect')
      .attr('x', -NODE_W / 2).attr('y', -NODE_H / 2)
      .attr('width', NODE_W).attr('height', NODE_H)
      .attr('rx', 6).attr('ry', 6)
      .attr('fill', '#1c1916')
      .attr('stroke', d => TAG_COLORS[d.data.tag] || '#5a4a30')
      .attr('stroke-width', 2);

    nodeEnter.append('text')
      .attr('text-anchor', 'middle').attr('y', -12)
      .attr('fill', '#e8c97a').attr('font-size', '12px').attr('font-weight', 'bold')
      .text(d => d.data.name.length > 24 ? d.data.name.slice(0, 22) + '\u2026' : d.data.name);

    nodeEnter.append('text')
      .attr('text-anchor', 'middle').attr('y', 6)
      .attr('fill', '#a89880').attr('font-size', '10px')
      .text(d => {
        let t = '';
        if (d.data.birth) t += 'b. ' + d.data.birth;
        if (d.data.death) t += ' \u2014 d. ' + d.data.death;
        return t.length > 32 ? t.slice(0, 30) + '\u2026' : t;
      });

    nodeEnter.append('text')
      .attr('text-anchor', 'middle').attr('y', 22)
      .attr('fill', '#8b6f47').attr('font-size', '9px')
      .text(d => {
        const p = d.data.birthPlace || '';
        return p.length > 28 ? p.slice(0, 26) + '\u2026' : p;
      });

    // Expand/collapse indicator
    nodeEnter.append('text')
      .attr('class', 'expand-indicator')
      .attr('text-anchor', 'middle').attr('y', NODE_H / 2 + 14)
      .attr('fill', '#c9a84c').attr('font-size', '10px')
      .text(d => d._children ? '\u25BC' : '');

    // Marriage spouse label (small text below connector)
    nodeEnter.each(function (d) {
      if (d.data.marriage && d.data.marriage.spouse) {
        d3.select(this).append('rect')
          .attr('x', NODE_W / 2 + 8).attr('y', -SPOUSE_H / 2)
          .attr('width', SPOUSE_W).attr('height', SPOUSE_H)
          .attr('rx', 4).attr('ry', 4)
          .attr('fill', '#1c1916')
          .attr('stroke', '#8b6f47')
          .attr('stroke-width', 1)
          .attr('stroke-dasharray', '4,3');

        d3.select(this).append('line')
          .attr('x1', NODE_W / 2).attr('y1', 0)
          .attr('x2', NODE_W / 2 + 8).attr('y2', 0)
          .attr('stroke', '#c9a84c').attr('stroke-width', 1).attr('stroke-dasharray', '3,2');

        d3.select(this).append('text')
          .attr('x', NODE_W / 2 + 8 + SPOUSE_W / 2).attr('y', -6)
          .attr('text-anchor', 'middle')
          .attr('fill', '#a89880').attr('font-size', '10px')
          .text(d.data.marriage.spouse.length > 20 ? d.data.marriage.spouse.slice(0, 18) + '\u2026' : d.data.marriage.spouse);

        d3.select(this).append('text')
          .attr('x', NODE_W / 2 + 8 + SPOUSE_W / 2).attr('y', 10)
          .attr('text-anchor', 'middle')
          .attr('fill', '#8b6f47').attr('font-size', '9px')
          .text(d.data.marriage.date ? 'm. ' + d.data.marriage.date : '');
      }
    });

    // Update positions
    const nodeUpdate = nodeEnter.merge(node);
    nodeUpdate.transition().duration(duration)
      .attr('transform', d => `translate(${d.x},${d.y})`);

    // Update expand indicator
    nodeUpdate.select('.expand-indicator')
      .text(d => d._children ? '\u25BC' : '');

    // Remove exiting nodes
    node.exit().transition().duration(duration)
      .attr('transform', `translate(${source.x},${source.y})`)
      .attr('opacity', 0).remove();

    // Store positions for transitions
    nodes.forEach(d => { d.x0 = d.x; d.y0 = d.y; });
  }

  function diagonal(d) {
    return `M${d.source.x},${d.source.y + NODE_H / 2}
            C${d.source.x},${(d.source.y + d.target.y) / 2}
             ${d.target.x},${(d.source.y + d.target.y) / 2}
             ${d.target.x},${d.target.y - NODE_H / 2}`;
  }

  function resetView() {
    // Find Hans Bernhardt (depth 3) to center on
    const target = root.descendants().find(d => d.data.id === 'hans-bernhardt') || root;
    const w = container.clientWidth;
    const h = container.clientHeight;
    const t = d3.zoomIdentity.translate(w / 2 - (target.x || 0), 80).scale(0.85);
    svg.transition().duration(600).call(zoom.transform, t);
  }

  // Initial render
  root.x0 = 0;
  root.y0 = 0;
  update(root);

  // Initial view — center on tree
  setTimeout(resetView, 100);
})();
</script>
```

- [ ] **Step 2: Verify tree renders without JS errors**

Run: `node -e "const h=require('fs').readFileSync('index.html','utf8'); const m=h.match(/<script>[\\s\\S]*?FAMILY TREE RENDERER[\\s\\S]*?<\\/script>/); console.log(m ? 'Tree script found (' + m[0].length + ' chars)' : 'NOT FOUND')"`

Expected: `Tree script found (NNNN chars)` with a size > 3000

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: implement D3.js family tree with pan/zoom and detail panel"
```

---

### Task 5: Update Map — New Markers, Ship Harle, Waypoints

**Files:**
- Modify: `index.html` — the map `<script>` block (currently starts around line 1349)

- [ ] **Step 1: Change initial map view to show full migration**

Change:
```javascript
  const map = L.map('map', { zoomControl: true }).setView([38.5, -77], 5);
```

To:
```javascript
  const map = L.map('map', { zoomControl: true }).setView([42, -30], 3);
```

- [ ] **Step 2: Update the Philadelphia marker dates**

In the residences array, find the Philadelphia entry and change:
```javascript
      dates: 'Arrival c. 1751',
      body: 'Primary port of entry for Palatine German immigrants. Male immigrants signed oaths of allegiance at the courthouse. The Schauss family entered America here before moving to Lancaster County.'
```
To:
```javascript
      dates: 'Arrival Sep 1, 1736',
      body: 'Johann Adam Schauss arrived Sep 1, 1736 on the Ship Harle (Passenger #94: "Johan Adam Shans", age 32). Primary port of entry for Palatine Germans. Male immigrants signed oaths of allegiance at the Philadelphia courthouse.'
```

- [ ] **Step 3: Update Lancaster County dates**

Change:
```javascript
      dates: '1751–1755',
```
To:
```javascript
      dates: '1736–c. 1754',
```

- [ ] **Step 4: Add new PA residence markers to the residences array**

Add these entries before the closing `];` of the residences array:

```javascript
    ,{
      ll: [40.27, -75.58],
      title: 'Falkner Swamp, PA',
      historic: 'Montgomery County, Pennsylvania',
      dates: '1737–1741 · 5 children born here',
      body: 'German Reformed community where Johann Adam Schauss settled after arrival. Children Conrad (1738), Anna Margaretha (1740), and Heinrich Sr. (1741) were born here. Also known as New Hanover Township.'
    },
    {
      ll: [40.66, -75.49],
      title: 'Whitehall, PA',
      historic: 'Bucks County, Pennsylvania',
      dates: '1748 · Christian Shouse born',
      body: 'Christian Shouse, the youngest child of Johann Adam, born Jul 11, 1748. By this time the family name was already being Anglicized to "Shouse" in some records.'
    },
    {
      ll: [40.69, -75.22],
      title: 'Easton, PA',
      historic: 'Northampton County, Pennsylvania',
      dates: 'Maria Barbara Baum died here before 1768',
      body: 'Maria Barbara (Baum) Schauss, wife of Johann Adam, died here before 1768 at approximately age 60. She was born Jan 8, 1707 in Albisheim and emigrated with her husband in 1736.'
    }
```

- [ ] **Step 5: Update the Albisheim popup to include Ortsfamilienbuch**

In the residences array, change the Albisheim body to:
```javascript
      body: 'Hans Bernhardt Schauss married Anna Margarethe Naass here Apr 21, 1674, establishing the family. Philipp Heinrich Schauss (1686\u20131765) served as Mayor & linen weaver. Johann Adam emigrated 1736 on the Harle. Parish records from 1641 with gaps. See: Ortsfamilienbuch "Die Familien Albisheims 1641\u20131900" by Detlef Uhrig (2009). Contact: Geschichts- und Heimatverein Albisheim e.V., Pfrimmtalstra\u00dfe 9.'
```

- [ ] **Step 6: Update Eckweiler popup with microfilm numbers**

Change the Eckweiler body to:
```javascript
      body: 'Earliest documented Schauss location. Han\u00df Schau\u00df (bef. 1600) & Catharina Von Allenfeldt lived here. Son Melchior Schau\u00df (1595\u20131628) married Helena Ante\u00dfes here 1621. Church records survive from 1568. FHL Microfilms: 489885 (1632\u20131675) & 493211 (1760\u20131798), Staatsarchiv Koblenz.'
```

- [ ] **Step 7: Add Pfalzbibliothek to the germanRepos array**

Add to the germanRepos array:
```javascript
    ,{
      ll: [49.4447, 7.7689],
      title: 'Pfalzbibliothek Kaiserslautern',
      historic: 'Palatinate Library, Kaiserslautern',
      dates: 'Ortsfamilienbuch & church book transcripts',
      body: 'Holds the definitive Ortsfamilienbuch "Die Familien Albisheims 1641\u20131900" by Detlef Uhrig (catalog 1b 6809, 2 vols). Also holds transcripts of nearly 400 church and family books for the entire Palatinate, available for loan.'
    }
```

- [ ] **Step 8: Update the German migration path style**

Change:
```javascript
  L.polyline(germanPath, {
    color: '#c9a84c',
    weight: 2,
    opacity: 0.6,
    dashArray: '6, 8',
```
To:
```javascript
  L.polyline(germanPath, {
    color: '#c9a84c',
    weight: 3,
    opacity: 0.6,
    dashArray: '3, 6',
```

- [ ] **Step 9: Update the transatlantic path to route through Cowes**

Change the migrationPath waypoint from London to Cowes:
```javascript
    [51.5074, -0.1278],  // London (waypoint)
```
To:
```javascript
    [50.7636, -1.2980],  // Cowes, Isle of Wight (waypoint)
```

- [ ] **Step 10: Add Rotterdam and Cowes waypoint markers**

Insert after the germanRepos forEach block and before the `repos.forEach` line:

```javascript
  // ── WAYPOINT MARKERS (muted orange) ─────────────────────────
  const ORANGE_MUTED = '#b87040';
  [
    { ll: [51.9225, 4.4792], title: 'Rotterdam', body: 'Embarkation port for Palatine emigrants. Ships departed down the Rhine to Rotterdam before crossing to England.' },
    { ll: [50.7636, -1.2980], title: 'Cowes, Isle of Wight', body: 'Last port before Atlantic crossing. Ships stopped to take on provisions and additional passengers.' }
  ].forEach(w => {
    L.marker(w.ll, { icon: makeIcon(ORANGE_MUTED, 10) })
      .bindPopup(popup(w.title, 'Waypoint', '', w.body))
      .addTo(map);
  });

```

- [ ] **Step 11: Replace the Atlantic text label with a Ship Harle marker**

Replace the entire Atlantic label block:
```javascript
  // ── ATLANTIC OCEAN arc (approximate great circle) ───────────────
  L.marker([40.5, -40], {
    icon: L.divIcon({
      className: '',
      html: '<div style="color:#e8703a;font-size:10px;letter-spacing:0.1em;opacity:0.7;white-space:nowrap;text-shadow:1px 1px 2px #000">— Atlantic crossing · Sep 1, 1736 · Ship Harle —</div>',
      iconSize: [260, 18],
      iconAnchor: [130, 9]
    })
  }).addTo(map);
```

With:
```javascript
  // ── SHIP HARLE MARKER (mid-Atlantic) ────────────────────────
  const shipIcon = L.divIcon({
    className: '',
    html: '<div style="font-size:20px;text-shadow:0 0 6px #e8703a88;" title="Ship Harle">\u26F5</div>',
    iconSize: [24, 24],
    iconAnchor: [12, 12],
    popupAnchor: [0, -14]
  });
  L.marker([42.0, -35.0], { icon: shipIcon })
    .bindPopup(`
      <div class="popup-title">Ship Harle of London</div>
      <div class="popup-dates">Arrived Philadelphia \u2014 September 1, 1736</div>
      <div class="popup-body">
        <strong>Master:</strong> Ralph Harle<br>
        <strong>Route:</strong> Rotterdam \u2192 Cowes \u2192 Philadelphia<br>
        <strong>Passengers:</strong> 388 total (151 qualified foreigners)<br><br>
        <strong style="color:#e8c97a;">Passenger #94: \u201cJohan Adam Shans\u201d</strong><br>
        Age 32 \u2014 signed oath as \u201cJohann Adam Schauss\u201d<br><br>
        <em style="font-size:0.75rem;">Source: Pennsylvania German Pioneers, Vol. 1 (Strassburger, 1980)</em>
      </div>
    `)
    .addTo(map);
```

- [ ] **Step 12: Verify JS is still valid**

Run: `node -e "const h=require('fs').readFileSync('index.html','utf8'); const m=h.match(/<script>([\\s\\S]*?)<\\/script>/g); let ok=true; m.forEach((s,i)=>{const body=s.replace(/<\\/?script>/g,''); try{new Function(body)}catch(e){console.log('Script',i,'error:',e.message);ok=false}}); if(ok)console.log('All JS OK')"`

Expected: `All JS OK`

- [ ] **Step 13: Commit**

```bash
git add index.html
git commit -m "feat: update map with PA markers, Ship Harle, waypoints, and archive"
```

---

### Task 6: Deploy to GitHub Pages

**Files:**
- No file changes — deployment commands only

- [ ] **Step 1: Push all commits to origin**

```bash
git push origin main
```

- [ ] **Step 2: Enable GitHub Pages via gh API**

```bash
gh api repos/patjackson52/shouse_project/pages -X POST -f build_type=legacy -f source='{"branch":"main","path":"/"}'
```

If Pages is already enabled, this may return an error. In that case, update instead:
```bash
gh api repos/patjackson52/shouse_project/pages -X PUT -f build_type=legacy -f source='{"branch":"main","path":"/"}'
```

- [ ] **Step 3: Set the repository homepage URL**

```bash
gh repo edit patjackson52/shouse_project --homepage "https://patjackson52.github.io/shouse_project/"
```

- [ ] **Step 4: Verify the deployment**

Wait ~60 seconds for GitHub Pages to build, then check:

```bash
gh api repos/patjackson52/shouse_project/pages --jq '.status'
```

Expected: `built`

Then verify the URL resolves:
```bash
curl -s -o /dev/null -w "%{http_code}" https://patjackson52.github.io/shouse_project/
```

Expected: `200`

- [ ] **Step 5: Commit (no file changes — tag the deployment)**

No files to commit. The site is live at `https://patjackson52.github.io/shouse_project/`
