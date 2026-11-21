# Changelog

Every release of apogee-pulse, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

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
