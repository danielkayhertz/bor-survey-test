# Address-to-District Lookup Feature — Spec

## Context

The BoR Survey app is a single-file vanilla HTML/JS/CSS application for Impact for Equity, deployed on GitHub Pages. It presents Cook County Board of Review candidate survey responses organized by district and candidate. Users currently must already know which district they're in to find relevant candidates.

This feature adds an address lookup so users can enter their home address, discover which BoR district they fall in, and be automatically navigated to the relevant candidate survey content. Only Districts 1 and 2 have elections this year; District 3 does not.

A high-resolution GeoJSON file (`Board_of_Review_Boundary_-_Current.geojson`, ~1.15MB, 3 polygons with 6,800–10,600 vertices each, WGS84/CRS84 coordinates) has been added to the repo.

---

## Decisions Summary

| Decision | Choice |
|---|---|
| GeoJSON loading | Fetch as separate file (not inlined) |
| Geocoding service | Nominatim (OpenStreetMap) |
| Post-lookup behavior | Auto-navigate to district + visual highlight |
| UI placement | Above content area (card) |
| Ambiguous addresses | Auto-filter to Cook County |
| Out-of-bounds address | Simple error: "not in Cook County" |
| Post-result state | Collapse to slim badge: "Your district: District X [Change]" |
| Input format | Single text input |
| OSM attribution | Page footer |
| Badge behavior | Always shows looked-up district, regardless of browsing |
| GeoJSON filename | Rename to `bor-districts.geojson` |
| Load timing | Show input immediately; queue submission until GeoJSON loads |
| No-election district | District 3 — show informative message + redirect to explore other districts |
| Geolocation API | Not used (neither mobile nor desktop) |
| Autocomplete | No — submit on Enter/button click only |
| Point-in-polygon | Full-resolution ray-casting, no simplification |
| District ID mapping | Hardcoded: `{1: 'd1', 2: 'd2', 3: 'd3'}` |
| Mobile UX | Larger input, full-width button on mobile |
| Rate limits | Acceptable risk; set proper User-Agent header |

---

## Implementation Plan

### 1. Rename GeoJSON file
- Rename `Board_of_Review_Boundary_-_Current.geojson` → `bor-districts.geojson`

### 2. GeoJSON loading (`bor-survey.html`)
- On `DOMContentLoaded`, begin `fetch('bor-districts.geojson')` and store the parsed result in a module-level variable (e.g. `let districtGeoJSON = null`)
- Set a `geoJSONLoaded` flag so the lookup knows when data is ready

### 3. Address lookup card (HTML + CSS)
- **Location**: Rendered at the top of `#contentArea`, above the existing district/candidate content
- **Expanded state** (initial):
  - A card styled consistently with existing compare-cards (same `--card-shadow`, `--card-radius`, brand colors)
  - Heading: "Find Your District" in `--ife-navy`
  - Single text input with placeholder "Enter your address (e.g. 123 Main St, Chicago, IL)"
  - A "Search" button styled with `--ife-blue` background, white text
  - Error/result messages appear below the input
  - Loading spinner while geocoding/processing
- **Collapsed state** (after successful lookup):
  - Slim bar: "Your district: **District X**" with a "Change" link/button that re-expands the card
  - Always visible above content regardless of which district/candidate the user is browsing
  - Styled with a subtle `--ife-light-blue` background or left border accent
- **Mobile (≤768px)**:
  - Input and button are full-width
  - Larger touch targets (min 44px height on input and button)
  - Same collapse behavior

### 4. Geocoding logic
- On form submit (Enter key or button click):
  1. Show loading state (spinner, disable button)
  2. Call Nominatim: `https://nominatim.openstreetmap.org/search?q={address}&format=json&addressdetails=1&countrycodes=us`
  3. Set `User-Agent` header or append `&email=` param per Nominatim policy (use a contact email for the project)
  4. Filter results to Cook County: check `addressdetails.county` contains "Cook"
  5. If no Cook County results: show error "This address doesn't appear to be in Cook County. The Board of Review covers Cook County properties only."
  6. If result found: extract `lat`, `lon` from the first Cook County result

### 5. Point-in-polygon (ray-casting)
- Implement a `pointInPolygon(lat, lon, polygon)` function using the standard ray-casting algorithm
- Iterate through all 3 district features in the GeoJSON
- Use the hardcoded mapping `{1: 'd1', 2: 'd2', 3: 'd3'}` to translate `DISTRICT_INT` → app district ID
- If no district matches (shouldn't happen for Cook County addresses, but defensive): show the out-of-bounds error

### 6. Result handling
- **Districts 1 or 2** (election this year):
  1. Set `selectedDistrict` to the matched district ID
  2. Switch to "By District" / compare mode via `setMode('compare')`
  3. Collapse the lookup card to the slim badge
  4. Apply a brief visual highlight/animation to the content area (e.g. a fade-in with a subtle color pulse on the district heading) to connect the lookup result to the content
- **District 3** (no election):
  1. Show message in the lookup card: "Your address is in District 3, which does not have a Board of Review election this year. You can still explore the candidates in other districts below."
  2. Do NOT collapse the card (keep expanded so the message is visible)
  3. Do NOT auto-navigate (user can manually browse)

### 7. Queued submission (GeoJSON not yet loaded)
- If user submits before `districtGeoJSON` is populated:
  - Show "Loading district boundaries..." with spinner
  - Store the geocoded lat/lon
  - When GeoJSON finishes loading, automatically run the point-in-polygon check and display the result

### 8. OSM attribution footer
- Add a `<footer>` at the bottom of the page (after `.app-layout`)
- Content: "Address lookup powered by OpenStreetMap" (with link to openstreetmap.org)
- Styled small, muted (`--ife-slate` color), centered

### 9. Rendering integration
- Modify `renderContent()` to always render the lookup card first (either expanded or collapsed) before delegating to `renderCandidateView()` or `renderCompareView()`
- The lookup card state (expanded/collapsed, stored district) should persist across mode switches and content re-renders
- Store lookup state in module-level variables: `let userDistrict = null; let lookupExpanded = true;`

---

## Files to Modify

| File | Change |
|---|---|
| `bor-survey.html` | All JS/CSS/HTML changes (lookup card, geocoding, PiP, footer, mobile styles) |
| `Board_of_Review_Boundary_-_Current.geojson` | Rename to `bor-districts.geojson` |

---

## Verification

1. **Happy path (D1/D2)**: Enter a known District 1 or 2 address (e.g. a Chicago address) → should show "Your district: District X" badge and auto-navigate to that district's compare view
2. **District 3**: Enter a south suburb address in District 3 → should show the "no election this year" message without navigating
3. **Out of county**: Enter an address outside Cook County (e.g. "Springfield, IL") → should show the Cook County error message
4. **Invalid address**: Enter gibberish → Nominatim returns no results → show appropriate error
5. **Mobile**: Test at ≤768px viewport — input and button should be full-width with larger touch targets
6. **Collapse/expand**: After finding a district, verify the badge persists across mode switches; clicking "Change" re-expands the lookup form
7. **Slow load**: Throttle network in DevTools, submit address before GeoJSON loads → should queue and resolve once loaded
8. **Footer**: OSM attribution visible at page bottom
