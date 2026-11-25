# apogee-pulse

Production monitoring for [Apogee](https://github.com/venomous-maker) apps:
throughput and latency (p50, p95, p99) per route, status codes, slow requests,
slow queries, exceptions, queue throughput and failures, slow jobs, the
busiest users, and each server's load and memory, shown at `/pulse`.

Unlike [apogee-telescope](https://github.com/venomous-maker/apogee-telescope)
it keeps no requests, only counts per minute (how many, their total, the
largest, the last detail) of each kind and key. Recording adds to an
in-memory buffer; a background thread writes the buffer out in one batch every
few seconds, so it is cheap enough to leave on in production.

```grav
import "std"
import "apogee-pulse"
use apogee.*

Pulse.use(new DatabasePulseStore(pool))      // then .createTables() (in an app: the migration); or the memory store (the default)
Pulse.watchQueries()                         // slow queries (Orbit's DB.listen)
Pulse.watchExceptions(kernel)                // what the HTTP kernel reports
Pulse.watchJobs(events)                      // the queue's events

router.aliasMiddleware("pulse", fn(a: Array<string>) -> Middleware { return new PulseMiddleware() })
router.use("pulse")                          // count every request
Pulse.routes(router)                         // the page at /pulse
Pulse.start()                                // flush every 5 s, sample this server every 15 s, prune hourly

Pulse.record("checkout", "card", 182.0)      // anything else worth counting
```

## In an apogee-framework app

Importing the module is all it takes: `PulsePackage` (apogee-package) connects it when
the app is created. When `pulse.enabled`, per-minute production metrics are recorded (requests, slow requests, queries and jobs, exceptions, servers) and shown at `/pulse`. The page is open locally; elsewhere only to a signed-in user the gate allows `viewPulse`. Metrics are written from a background thread through the app's pool.

**Migration:** `CreatePulseTables` makes `pulse_aggregates` and `pulse_values`. Run `migrate` after adding the module.

**Commands:** `pulse:clear`.

| Setting | Default |
|---|---|
| `pulse.enabled` | `PULSE_ENABLED`, off |
| `pulse.driver` | `PULSE_DRIVER`: `database` (default) or `memory` |
| `pulse.path` | `PULSE_PATH`, `pulse` |
| `pulse.server` | `PULSE_SERVER_NAME`: this machine's name in the servers card |
| `pulse.sample_rate` | `PULSE_SAMPLE_RATE`, 1 |
| `pulse.keep_hours` | `PULSE_KEEP_HOURS`, 168 |
| `pulse.slow_request_ms`, `.slow_query_ms`, `.slow_job_ms` | 1000 each (`PULSE_SLOW_*_MS`) |
| `pulse.ignore_paths` | `PULSE_IGNORE_PATHS`, comma-separated |

Your own configuration wins over these defaults.

## What it records

| Kind | Key | Value |
|---|---|---|
| `request` | method and route (`GET /users/{id}`) | ms |
| `status` | `2xx`, `4xx`, `5xx`… | ms |
| `latency` | the bucket a request fits under (5, 10, 25 … 10000 ms) | ms |
| `slow_request` | route, with the last real path | ms |
| `slow_query` | the statement, with its connection | ms |
| `exception` | `Kind: message`, with where it happened | — |
| `job`, `slow_job`, `failed_job` | the job's class | ms, or the error |
| `queued`, `processed`, `failed` | `connection:queue` | ms |
| `user` | the signed-in user (`Pulse.userKey` names it) | ms |
| `server_load`, `server_memory` | the server's name | load, MB |

Paths become routes by turning numbers, UUIDs, ULIDs and long hex or token
segments into `{id}`, so `/users/42` and `/users/43` count together.
Percentiles come from the latency buckets (each is the bucket's upper bound).

## Stores

- `MemoryPulseStore`: this process only (development, tests).
- `DatabasePulseStore(pool, prefix = "pulse_")`: SQLite, MySQL/MariaDB or
  PostgreSQL, shared by every server. Each flush is one transaction of upserts
  that add counts and totals and keep the larger maximum. Tables:
  `pulse_aggregates` (minute, type, key) and `pulse_values` (each server's
  latest sample).

## Settings

```grav
Pulse.slowRequestMs = 1000.0                  // also slowQueryMs, slowJobMs
Pulse.sampleRate = 0.1                        // count one request in ten (scaled back up)
Pulse.ignorePaths.push("/health")
Pulse.userKey = fn(u: any) -> string { return (u as User).email }
Pulse.flushMs = 5000
Pulse.keepHours = 168                         // a week
Pulse.server = "web-1"                        // default: the hostname
Pulse.setGate(fn(req: Request) -> bool { return req.user != null && (req.user as User).admin })
```

The page and its API (`/pulse/api/overview?period=1h|6h|24h|7d`,
`POST /pulse/api/clear`) answer only this machine unless a gate says otherwise.

## Testing

```sh
grav test tests/pulse.grav
grav test tests/package.grav
PULSE_MYSQL_URL=mysql://root:pw@127.0.0.1:3306/pulse PULSE_POSTGRES_URL=postgres://postgres:pw@127.0.0.1:5432/pulse grav test tests/pulse.grav
```

## License

MIT
