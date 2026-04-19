# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A single-file static browser app (`index.html`) that reads USPS shipping CSV files and outputs JavaScript data structures for use in a shipping rate calculator. No build step, no package manager, no test suite — open `index.html` directly in a browser.

## Running the app

```bash
# Any local static server works, e.g.:
python3 -m http.server
# then open http://localhost:8000
```

## Architecture

Everything lives in `index.html`. Alpine.js (loaded from CDN) handles reactive UI state. All logic is plain JS in a `<script>` block at the bottom.

The app has two independent sections, each with its own file input and output textarea:

**Rates section** — `processRatesCSV(text)`

Auto-detects single-zone vs multi-zone format by checking whether the row after the first `Weight Not Over` header contains `Zone N` cells.

- *Single-zone* (e.g. `Media Mail.csv`): row 0 has name (col 0) and date (first cell matching `M/D/YYYY`); the `Weight Not Over` row is the header; data rows are `maxWeightLbs,rate`. Output: `const <name>Rates` with a `tiers` array of `[maxWeightLbs, rate]` pairs and a `get<Name>Rate(weightLbs)` lookup.
- *Multi-zone* (e.g. `PM Retail.csv`, `USPS Ground Advantage Retail.csv`): same row-0 metadata; after the `Weight Not Over` header a zone-header row lists `Zone 1 … Zone N`; data rows are `maxWeight,z1rate,z2rate,…`. Multiple `Weight Not Over` sections are supported (Ground Advantage has an ounces section then a pounds section — ounce weights are converted to lbs by dividing by 16). Output: `const <name>Rates` with a `zones` array and `tiers` array of `[maxWeightLbs, zone1Rate, …]` pairs, and a `get<Name>Rate(weightLbs, zone)` lookup.

**Zones section** — `processZonesCSV(text)`
- Input: CSVs like `usps-zones.csv`. Row 0 is the header (`ZIP Code,Zone`). ZIP values are either plain 3-digit prefixes or ranges using `---` as separator (e.g. `006---009`); ranges are expanded into individual entries. Values longer than 3 digits (e.g. `96700`) are truncated to their first 3 digits.
- Output: a `const uspsZones` object with a flat `zones` map of `"prefix": zoneNumber`, plus a `getZone(destZip)` function that pads the input to 5 digits and slices the first 3 to look up the zone.

**Shared utilities** (module-level functions, not on the Alpine component):
- `parseCSVRow(line)` — minimal RFC-4180 parser (handles quoted fields and escaped quotes)
- `normalizeLines(text)` — strips `\r\n` / `\r` before splitting
- `toCamelCase(str)`, `capitalize(str)` — used to derive variable/function names from the mail class name

## Example input files

`example-input/` contains representative CSVs:
- `Media Mail.csv` — single-rate-per-weight format (what `processRatesCSV` targets)
- `usps-zones.csv` — ZIP prefix → zone number (what `processZonesCSV` targets)
- `PM Retail.csv`, `USPS Ground Advantage Retail.csv` — multi-zone rate tables (weight × zone grid); handled by the multi-zone path in `processRatesCSV`
