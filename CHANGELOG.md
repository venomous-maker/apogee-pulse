# Changelog

Every release of apogee-pulse, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

Needs grav 1.12.0 or newer.

- Finer percentiles: about 170 latency buckets (each about 9% wider than the
  last), percentiles placed within their bucket and capped by its slowest
  request, and p75, p90 and p99.9 beside p50, p95 and p99 (`preciseLatency =
  false` for the old buckets). Minutes recorded before still read.
- Requests group by the matched route's pattern (`req.routeUri()`) when there is
  one, instead of a guess from the path.
- `PulsePackage`: the module connects itself to an apogee-framework app (apogee-package):
  its settings, wiring and commands moved here from the framework.
- Its tables as a migration (`CreatePulseTables`, run by `migrate`); the store's own
  table-making method uses the same definition.

### Dependencies

- Uses apogee-package 0.1.0, apogee-console 0.2.2, apogee-http 0.3.0, apogee-routing 0.3.0, apogee-queue 0.3.1, grav-orbit 0.3.0, apogee-testing 0.1.1 (from their GitHub tags).

## 0.1.2 (2026-11-21)

- Cheaper recording: one lock per request, buffers sharded by thread (four
  threads recording at once no longer wait on each other), and routes found in
  one pass (about 4 times faster).

## 0.1.1 (2026-11-21)

- A request that ends in an `HttpException` (a 404 from the router, say) is
  recorded with that status, not as a 500 failure.

## 0.1.0 (2026-11-21)

- `Pulse`: per-minute counts of requests (per route, status class and latency
  bucket), slow requests, slow queries, exceptions, jobs and queues, users,
  and server load and memory; buffered and flushed by a background thread;
  sampling, ignored paths, pruning.
- Memory and database (SQLite, MySQL, PostgreSQL) stores.
- The dashboard page and its JSON API, behind a gate.
