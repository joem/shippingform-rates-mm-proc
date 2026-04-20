# Rates And Zones Generator For Shipping Form

A single-file browser tool that converts USPS shipping CSV files into JavaScript data structures ready to drop into a shipping rate calculator.

No build step or installation required — open `index.html` directly in a browser, or serve it with any static server:

```bash
python3 -m http.server
# open http://localhost:8000
```

## Shipping Rates tab

Accepts USPS rate CSVs exported from the USPS price tables. Two formats are supported and auto-detected:

**Single-rate** (e.g. `Media Mail.csv`) — one rate per weight tier:

```js
const mediaMailRates = {
  name: "Media Mail",
  effectiveDate: "4/7/2026",
  tiers: [      // [maxWeightLbs, rate]
    [1, 4.47],
    [2, 5.22],
    // ...
  ],
};

function getMediaMailRate(weightLbs) {
  const tier = mediaMailRates.tiers.find(([max]) => weightLbs <= max);
  return tier ? tier[1] : null;
}
```

**Multi-zone** (e.g. `PM Retail.csv`, `USPS Ground Advantage Retail.csv`) — one rate per weight tier per zone. Ounce-based weight sections (Ground Advantage) are automatically converted to pounds:

```js
const priorityMailRetailRates = {
  name: "Priority Mail - Retail",
  effectiveDate: "4/7/2026",
  zones: [1, 2, 3, 4, 5, 6, 7, 8, 9],
  tiers: [      // [maxWeightLbs, zone1Rate, ..., zone9Rate]
    [1, 11.00, 11.50, 11.65, 11.90, 13.05, 14.50, 15.60, 16.95, 34.25],
    // ...
  ],
};

function getPriorityMailRetailRate(weightLbs, zone) {
  const tier = priorityMailRetailRates.tiers.find(([max]) => weightLbs <= max);
  if (!tier) return null;
  const idx = priorityMailRetailRates.zones.indexOf(zone);
  return idx !== -1 ? tier[idx + 1] : null;
}
```

## Shipping Zones tab

Accepts USPS zone chart CSVs with a `ZIP Code,Zone` header. ZIP values may be plain 3-digit prefixes (`005`) or ranges (`006---009`); ranges are expanded into individual entries.

```js
const uspsZones = {
  zones: {    // 3-digit dest ZIP prefix → zone number
    "005": 1,
    "006": 7,
    // ...
  },
};

function getZone(destZip) {
  const prefix = String(destZip).padStart(5, '0').slice(0, 3);
  return uspsZones.zones[prefix] ?? null;
}
```

## Example input files

Representative CSVs are in `example-input/`:

| File | Format |
|------|--------|
| `Media Mail.csv` | Single-rate |
| `PM Retail.csv` | Multi-zone |
| `USPS Ground Advantage Retail.csv` | Multi-zone (ounces + pounds sections) |
| `usps-zones.csv` | ZIP prefix → zone |
