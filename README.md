# Residential Mobility Explorer — El Paso, TX

**Live page:** https://hoffmanap.github.io/residential-mobility/

An interactive map for tracing real address-to-address moves in and around El Paso. Draw any polygon and it shows who moved into that area, who moved out, what they paid, and how the places on either end of that flow compare demographically.

Built as a supplemental tool for a residential filtering analysis (does older, cheaper housing get passed down to lower-income households as new construction gets built?) — this piece specifically fills a gap in that analysis: the main report covered mover demographics (age, income) but not the geography of where people actually came from and went to, or how those origin/destination places compare.

## What's on the page

- **Draw a study area** — click points on the map to trace any polygon, double-click to close it (uses [Leaflet.draw](https://github.com/Leaflet/Leaflet.draw)).
- **Counts** — how many traced moves arrived in that area, left it, or happened entirely within it, 2019–2026.
- **Price comparison** — average sale price or monthly rent where arrivals landed vs. what leavers left behind, drawn only from matched MLS sales and rental listings (no modeled/estimated values).
- **Age-bracket mix** of arrivals vs. leavers, where available.
- **Census ACS demographics table** — median household income, median age, renter share, Hispanic/Latino share, White (non-Hispanic) share, median home value, and median gross rent, compared three ways: for the tract(s) your drawn area actually overlaps, for the tracts arrivals came from, and for the tracts leavers went to.
- **Choropleth toggle** — color every census tract by how many of your area's arrivals came from it, or how many of its leavers went to it.

The basemap is Esri's World Light Gray canvas + reference layer (streets and labels, no API key required) — deliberately desaturated so the tract choropleth reads clearly on top of it.

## Data sources and how this was built

**Movers (19,815 traced address changes, 18,027 geocoded on both ends):**
- Data Axle mover records (6,252 moves) — carries age bracket, a modeled household-value estimate, and distance moved, in addition to origin/destination address.
- El Paso Water change-of-address records (13,563 genuine address changes, after dropping same-address account updates) — each row already pairs a customer's old and new service address directly, so no separate origin/destination matching was needed for this source.

Both sources were fuzzy-matched by address (house-number + zip exact blocking, then a street-name similarity match) against combined MLS sales and rental listing data to recover a real transaction price or rent at each end of the move, with a three-tier match-confidence flag (Exact / High / Review recommended).

**Geocoding:** mover addresses only carried zip codes, not coordinates, so all 29,051 unique origin/destination addresses were batch-geocoded through the [U.S. Census Bureau's free address geocoder](https://geocoding.geo.census.gov/geocoder/) (94.7% match rate) to get lat/lon and a census tract for each one.

**Tract boundaries:** 2020-vintage Census tract polygons (188 tracts in El Paso County, FIPS 48141) from the Census TIGERweb service, simplified for file size. These match the tract IDs returned by the geocoder and used by the ACS data, so every mover and every demographic figure joins to a drawable polygon.

**Census ACS demographics:** 2019–2023 ACS 5-year estimates, pulled tract-by-tract from the Census Bureau's Data API (`api.census.gov/data/2023/acs/acs5`) and baked into the page as static data — the API key used to pull it is never stored in this repo or shipped to the browser; it was used once, server-side, to generate the embedded dataset.

Variables pulled: `B19013_001E` (median household income), `B01002_001E` (median age), `B25003_001E/002E/003E` (tenure — owner/renter occupied), `B03002_001E/003E/012E` (race/ethnicity — total, White non-Hispanic, Hispanic/Latino), `B25077_001E` (median home value), `B25064_001E` (median gross rent).

## How the "study area" is determined

When you draw a polygon, a census tract counts as part of "Your Area" if the drawn shape **overlaps** it at all (vertex-in-polygon in either direction, plus edge-crossing checks) — not just if the tract's center point happens to fall inside what you drew. That means a small polygon drawn entirely within one large tract still correctly picks up that tract, rather than coming back empty.

"Arrivals" and "Leavers" are matched to tracts the same way the underlying move data is tagged: the tract containing the actual geocoded origin or destination address point.

## Known limitations

- **Price/rent figures mix sale prices and monthly rents** in the same average when both move types are present in a selection — read the comparison as directional, not a clean apples-to-apples dollar figure.
- **Age bracket and modeled home-value fields exist only for the Data Axle subset** — the water utility source has no demographic fields, so those breakdowns undercount the full traced-mover population. (Price/rent figures, by contrast, use the full combined set.)
- **ACS figures describe tracts, not individual movers.** The Census Bureau doesn't link ACS demographics to specific households, so "arrivals are 83% Hispanic" means the tracts they moved from are 83% Hispanic on average — not a statement about the movers themselves.
- **ACS 5-year estimates carry their own margin of error**, especially for smaller tracts — treat small differences between columns cautiously.
- 9% of traced moves (1,788 of 19,815) couldn't be geocoded on one or both ends and are excluded from the map.

## Regenerating the data

This repo holds only the finished, static `index.html` — all mover data, tract geometry, and ACS figures are embedded directly in the page as JSON, so it runs entirely client-side with no backend and no API calls at runtime. The source pipeline (MLS/rental/HMDA combination, fuzzy address matching, Census geocoding, ACS pull) lives outside this repo as part of the broader filtering-analysis project.
