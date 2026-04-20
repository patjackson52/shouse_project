# ADR-001: Include Images and Rich Media When Available

**Date:** 2026-04-19
**Status:** Accepted
**Decision Makers:** Patrick Jackson

## Context

The Schauss/Shouse family heritage website at `index.html` is text-heavy with an interactive map and family tree but no photographs, illustrations, or other visual media. The site documents a 13-generation lineage from Eckweiler, Germany (1560) through Albisheim, emigration on the Ship Harle (1736), Pennsylvania, and the Moravian settlements of North Carolina to the present day.

Many of the places, churches, cemeteries, and records referenced on the site have freely-available photographs on platforms like Wikimedia Commons, the Library of Congress, Find a Grave, and US government archives. Adding these images would significantly enhance the storytelling and make the heritage more tangible for family members.

## Decision

**We will include images and rich media throughout the site whenever freely-licensed or public domain sources are available.** Specifically:

### Sourcing Policy

1. **Preferred sources (in priority order):**
   - Wikimedia Commons (CC-BY-SA, CC-BY, or Public Domain)
   - US Government works (public domain by law)
   - Library of Congress / National Archives
   - Find a Grave memorial photos (used under Find a Grave's terms for genealogical purposes)
   - Creative Commons-licensed photos from Flickr, institutional archives

2. **Linking strategy:** Images are referenced via direct URLs to their hosting platform (e.g., Wikimedia Commons `upload.wikimedia.org` URLs) rather than downloaded and self-hosted. This:
   - Respects bandwidth of a GitHub Pages-hosted static site
   - Ensures attribution links remain live and verifiable
   - Avoids storing large binary files in git
   - Leverages CDN infrastructure of source platforms

3. **Fallback for unavailable images:** When no free image exists for a location or subject, the section remains text-only. We do not use AI-generated images, stock photos, or rights-managed content.

### Attribution Requirements

Every image must include:
- Source attribution (photographer/creator name when known)
- License identifier (e.g., "CC BY-SA 4.0")
- Link to the source page (not just the image file)
- Alt text for accessibility

Attribution is displayed in a small caption below each image.

### Content Categories

Images are sought for:
- **Places:** German villages (Albisheim, Eckweiler), churches (Peterskirche, Heiligkreuzkirche), North Carolina Moravian settlements (Bethania, Old Salem, Bethabara)
- **Records:** Church book pages, passenger lists, cemetery headstones
- **Ships:** Period sailing vessels representing the emigrant experience
- **Cemeteries:** God's Acre flat headstones, Find a Grave memorials
- **Historic maps:** Already implemented as tile overlays; static map images may supplement
- **Architecture:** Moravian buildings, German half-timber houses, historic mills

### Styling

Images are displayed in a consistent style:
- Rounded corners matching the site's border-radius
- Sepia-tinted or desaturated to match the dark heritage theme
- Responsive sizing (max-width: 100%, height: auto)
- Lazy loading (`loading="lazy"`) for performance
- Caption text in the site's muted text color

## Consequences

### Positive
- Dramatically richer storytelling experience
- Family members can see the actual places their ancestors lived
- Increases engagement and time spent on the site
- No storage cost (externally hosted)
- All sources are freely licensed — no legal risk

### Negative
- External image URLs may break over time (Wikimedia is stable but not guaranteed)
- Page load time increases (mitigated by lazy loading)
- Some locations may have no available images, creating inconsistency
- Dependence on third-party hosting for core content

### Risks
- **Link rot:** Wikimedia Commons URLs are highly stable (files are rarely deleted), but we should periodically verify links
- **Bandwidth:** External hosts may rate-limit or block hotlinking; Wikimedia explicitly allows it for non-commercial use
- **Copyright misidentification:** Always verify the license on the source page before including

## Alternatives Considered

1. **Self-host all images:** Rejected — bloats the git repo, adds maintenance burden, and GitHub Pages has a 1GB soft limit
2. **No images (text only):** Rejected — misses a major opportunity to make the heritage tangible
3. **AI-generated illustrations:** Rejected — historically inaccurate, uncanny, and inappropriate for a factual genealogy site
4. **Paid stock photography:** Rejected — unnecessary cost when free sources exist for most subjects
