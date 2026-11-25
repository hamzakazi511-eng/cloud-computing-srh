# ADR 001: Use a lean Lambda architecture

## Status
Accepted

Date: 2026-11-25

## Context
Launch customer: **SHP06 Coastline Fashion**, a cost-focused fashion brand that ships most stock
by sea and flies only fast-selling pieces. Its targets:

- **Freshness:** the ops view may be at most 30 minutes behind reality.
- **Reliability:** at least 60% on time on every ocean lane and 90% on every air lane.
- **Budget:** $25 per month for the platform's cloud resources.

Evidence (`docs/metrics.md`, October 2026): six of seven lanes meet their target. **SGSIN → BEANR
misses it at 50%**, and the late shipments come from carriers ATL and ORS. Ocean delays are measured
in days (22–50 h on average), air delays in hours (0.7–2.5 h).

Data arrives in two forms: carriers send rates and schedules as **daily files**, and shipment
tracking arrives as **events**.

## Options considered
1. **Lambda:** a speed layer for live shipment events, and a batch layer for daily files. Two code
   paths, but each source is handled in the form it arrives in.
2. **Kappa:** everything as one event stream through a log (Kafka or Pub/Sub plus streaming Dataflow).
   One code path, and reprocessing is a log replay, but daily files must first be turned into events,
   and an always-on streaming pipeline costs money every hour.

## Decision
We chose **Lambda, kept lean**, because:

- Our sources are mixed. Rates and schedules come as daily files, so a batch load is the natural fit
  for them; turning them into events only to process them as a stream adds work and cost.
- The 30-minute freshness target needs the ops view to be live, but not per-second. A Pub/Sub BigQuery
  subscription meets it without an always-on Dataflow job.
- Coastline Fashion's focus is cost, and $25 a month rules out always-on clusters (GKE for Kafka,
  Dataproc, streaming Dataflow workers).

## Consequences
- **Cost:** Pub/Sub, BigQuery storage and queries, and scheduled loads at our volume (a few MB a month)
  should stay within or close to the free tiers, well under $25 a month. Always-on streaming workers
  or clusters would cost tens of dollars a month on their own.
- **Freshness:** shipment events reach BigQuery within seconds to minutes, inside the 30-minute target.
  Rates and schedules are up to a day old, which is fine for weekly carrier decisions.
- **Complexity:** two paths to build and monitor. The live and batch tables must agree on lane and
  carrier codes, or the ops view and the exec view will show different numbers.
- **What gets harder:** reprocessing history means rerunning the batch load *and* replaying events
  separately. If Coastline Fashion later needs per-minute freshness or event-time processing (for
  example, rerouting while a vessel is at sea), we would need to add Dataflow, and revisit this ADR.
