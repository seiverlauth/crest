# CLAUDE.md — The Tell

## What We're Building
A single `index.html` file — a live geopolitical heat map where countries
glow green to red based on news intensity and tone from GDELT right now.
No backend. No database. No build step. Deployed free via GitHub Pages.

## The Thesis
Public data streams spike before geopolitical events become mainstream
narrative. GDELT news intensity and tone is the first signal layer.
Making clustering visible before the narrative hardens is the product.

## MVP — Done When
- [ ] World map renders
- [ ] Countries colored green to red from live GDELT data (past 24h)
- [ ] Click a country to see its score + one line description
- [ ] Deployed to GitHub Pages

## Final Tech Decisions
- Single `index.html` — everything inline, no separate CSS or JS files
- Leaflet.js for the map (CDN: unpkg.com/leaflet@1.9.4)
- D3 for color scaling (CDN: cdnjs)
- GeoJSON world boundaries (CDN or inline)
- GDELT DOC 2.0 API — direct browser fetch, CORS enabled, no key needed
- GitHub Pages for deployment — zero cost, git push to deploy
- No build step, no npm, no webpack, no backend, no database

## GDELT API — The One Data Source
Base URL: https://api.gdeltproject.org/api/v2/doc/doc

The two queries to use:

**Volume per country (past 24h):**
```
https://api.gdeltproject.org/api/v2/doc/doc
  ?query=war%20OR%20conflict%20OR%20military%20OR%20crisis%20OR%20attack
  &mode=timelinevol
  &timespan=24h
  &format=json
```

**Tone per country — use timelinesourcecountry for country breakdown:**
```
https://api.gdeltproject.org/api/v2/doc/doc
  ?query=war%20OR%20conflict%20OR%20military%20OR%20crisis%20OR%20attack
  &mode=timelinesourcecountry
  &timespan=24h
  &format=json
```

The `timelinesourcecountry` mode returns coverage volume broken down
by the country the article was published in — use this as the heat signal.
Response shape: `{ timeline: [ { data: [ { value, normvalue, country } ] } ] }`
FIPS 2-letter country codes returned — need mapping to ISO 3166-1 alpha-2
for Leaflet GeoJSON matching.

Composite score per country = normalize coverage volume to 0–100.
Tone is a secondary signal — negative tone boosts the score slightly.
Simple weighted average. Don't over-engineer.

## Color Scale
D3 scaleSequential with interpolateRdYlGn reversed:
- 0 = green (quiet)
- 50 = yellow (elevated)
- 100 = red (anomalous)
Countries with no data = #1a1a2e (dark, not grey — fits the aesthetic)

## Aesthetic Direction
Dark. Intelligence terminal. Muted dark navy background. Monospace
accents. The map is the product — everything else is minimal chrome.
Fonts: 'Courier New' or similar monospace for labels and UI.
No gradients, no rounded corners, no friendly UI. Cold and functional.
Color palette:
  --bg: #0a0a0f
  --panel: #0f0f1a
  --border: #1e2a3a
  --text: #8899aa
  --accent: #00ff88
  --red: #ff3333

## File Structure
```
the-tell/
  index.html    ← entire project lives here
  CLAUDE.md     ← this file
```

That's it. Nothing else.

## Build Order — Follow This Exactly
1. Render map with hardcoded fake scores (5 countries, varying heat)
   Confirm colors work, map looks right, aesthetic is correct.
   Do not touch GDELT yet.

2. Add click handler — side panel slides in showing country name + score.
   Still fake data.

3. Replace fake data with live GDELT timelinesourcecountry query.
   Fetch on page load. Parse response. Normalize scores. Color the map.
   Log raw API response to console first before parsing.

4. Add loading state — map shows "LOADING SIGNAL DATA..." over dark
   overlay while fetch is in progress.

5. Clean up, deploy instructions at bottom of file as HTML comment.

## GitHub Pages Deploy (once index.html is working)
```bash
git init
git add index.html
git commit -m "initial"
gh repo create the-tell --public
git push -u origin main
# then: Settings → Pages → source: main branch
```
Live at: https://[username].github.io/the-tell

## Known Issues To Handle
- GDELT uses FIPS country codes, Leaflet GeoJSON uses ISO alpha-2.
  Need a FIPS→ISO mapping object. Key ones:
  US=US, UK=UK→GB, RS=RU, CH=CN, IR=IR, IZ=IQ, SY=SY, AF=AF,
  UP=UA, PO=PO→PL — build a full mapping or use a lookup table.
  Handle misses gracefully — just leave country at default color.

- GDELT rate limits on rapid repeated calls.
  Fetch once on load. No polling. Add a manual refresh button only.

- CORS: GDELT sets Access-Control-Allow-Origin: * so direct browser
  fetch works fine. No proxy needed.

- timelinesourcecountry returns data bucketed by time interval.
  Sum all intervals for the 24h window to get total volume per country.

## What Is Deliberately NOT In MVP
- Timeline scrubber
- USASpending layer
- Options flow layer
- Layer toggles
- Prediction market
- User accounts
- Mobile optimization
- Multiple data sources
- Any backend

## First Feature After MVP (do not build now)
Historical event overlay — a curated `events.json` file with ~50 major
geopolitical events. Show as markers on the map with dates. When you
hover, see what the signal looked like in the days before that event.
No new API needed. Just a JSON file.

## Instructions For Claude Code
- Build in the order listed above. Do not skip steps.
- One file. Everything inline.
- No npm. No build step. Open index.html in browser and it works.
- If GDELT response shape is unexpected, console.log it first and
  adjust the parser — do not guess at the shape.
- Keep the UI minimal. The data is the product.
- Do not add features not listed above.
- Do not refactor working code unless asked.
- When in doubt, do less.
