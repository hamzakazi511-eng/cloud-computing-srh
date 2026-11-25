# Metrics: Coastline Fashion (SHP06)

Launch customer: **SHP06 Coastline Fashion** (focus: cost, ocean and air).
Targets: freshness 30 minutes · reliability 60% on time per ocean lane, 90% per air lane · budget $25 per month.

Source: `sessions/03-batch-streams-and-architecture-patterns/starter/delay_per_lane.sql`,
October 2026 data in BigQuery (`freight.bookings` joined to `freight.schedules`).
On time = arrived no more than 24 hours after planned arrival.

## Delay and on-time rate per lane

| Mode | Lane | Shipments | Avg delay (h) | On time | Target | Meets target? |
|---|---|---|---|---|---|---|
| Ocean | SGSIN → BEANR (Asia-North Europe) | 10 | 50.4 | 50.0% | 60% | ❌ No |
| Ocean | GBFXT → CNSHA (North Europe-Asia) | 15 | 46.4 | 73.3% | 60% | ✅ Yes |
| Ocean | DEHAM → USNYC (Transatlantic) | 12 | 42.5 | 75.0% | 60% | ✅ Yes |
| Ocean | CNSHA → ITGOA (Asia-Mediterranean) | 26 | 22.2 | 76.9% | 60% | ✅ Yes |
| Air | SIN → LHR (Asia-Europe) | 25 | 2.5 | 96.0% | 90% | ✅ Yes |
| Air | ICN → FRA (Asia-Europe) | 27 | 0.7 | 100.0% | 90% | ✅ Yes |
| Air | PVG → FRA (Asia-Europe) | 18 | 1.1 | 100.0% | 90% | ✅ Yes |

## Findings

- **One lane misses its target: SGSIN → BEANR** (50% against 60%).
- Carriers on that lane: **ATL** Atlas Ocean (3 shipments, 0% on time), **ORS** Orion Sea Freight
  (3 shipments, 33%), **KST** Kestrel Maritime (3 shipments, 100%) and **BLC** Blue Crest Shipping (1 shipment, 100%).
  The late shipments come from ATL and ORS.
- All three air lanes meet the 90% target comfortably.
- Ocean delays are measured in days, air delays in hours. A 30-minute freshness target is
  tight enough to catch an ocean delay as it develops.
- Caveat: one month of data and only 10 shipments on the failing lane, so this is an early signal, not proof.
