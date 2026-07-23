# Self-host observability

This page is for **operators and SREs running the limited-function OSS stack** — the [self-hostable Apache-2.0 crates](open-core-boundary.md) you can bring up locally (via the published Docker Compose example) for evaluation and development. It answers the operator's first question — *"can I monitor what I run?"* — by showing where the shipped binaries expose their liveness/readiness probes and their Prometheus metrics, so you can wire up a health check and a scrape target **without reading the Rust source**.

> **Scope: this is the limited-function self-host stack, not the managed SaaS.** The uptime SLAs (99.5% / 99.9%), on-call rotation, and managed compliance posture described in [Cloud Deployment](cloud-deployment.md) apply to the **AI Agent Assembly cloud only** — not to a stack you self-host. Self-hosting is intended for local evaluation and development; you are responsible for operating and monitoring it. This page documents the observability surface the OSS binaries already expose; it is **not** a production deployment or orchestration guide (no Helm / Terraform / Kubernetes).

---

## What the stack exposes

The self-hostable stack runs **two** binaries with a published container image — `aa-runtime` and `aa-gateway` (see [Docker & Containers](docker-containers.md)). They do **not** expose the same surface: only `aa-runtime` serves HTTP health/metrics (on `:8080`). As launched in the [container topology](docker-containers.md#the-governed-topology) (`--policy … --listen 0.0.0.0:50051`, no `--mode`), the gateway runs in **legacy gRPC mode** and serves **gRPC only on `:50051`** — it exposes **no HTTP `/healthz` and no `/api/v1/health`**. Those HTTP surfaces exist only when the gateway is started in a different mode: `/healthz` in local or remote mode, and `/api/v1/health` in local mode only (`--mode local`, a single-process dev topology this container stack does not use, and which has no separately runnable `aa-api` container).

| Component | Surface | Default endpoint(s) | Purpose |
|---|---|---|---|
| **`aa-runtime`** | Health + metrics HTTP server | `/health`, `/ready`, `/metrics` on `AA_METRICS_ADDR` (default `0.0.0.0:8080`) | Liveness, readiness, and the Prometheus scrape target |
| **`aa-gateway`** (legacy gRPC mode — the container topology) | TCP liveness | gRPC port `:50051` | Process-liveness via a TCP connect; no HTTP health endpoint is served in this mode (`grpc.health.v1.Health` tracked in AAASM-4759) |

The gateway's HTTP `/healthz` and `/api/v1/health` surfaces are **not** part of the container topology on this page — they appear only when the gateway is launched in a non-default mode, summarized below:

| Surface | Where it exists | Default endpoint |
|---|---|---|
| `/healthz` | gateway in **local** or **remote** mode | `/healthz` |
| `/api/v1/health` | gateway in **local** mode only (`--mode local`) | `/api/v1/health` |

The rest of this page covers each surface and gives copy-paste probe and scrape examples.

---

## Health and readiness probes

### `aa-runtime` — `/health` and `/ready`

The runtime runs a combined health/metrics HTTP server bound to `AA_METRICS_ADDR` (see [Metrics endpoint](#prometheus-metrics-endpoint) below for the env var and its default). It serves two probe routes:

- **`GET /health`** — liveness. Returns `200 OK` with a JSON body reporting `status`, process uptime, events processed, and which enforcement layers are active or degraded. Use this as a liveness probe.
- **`GET /ready`** — readiness. Returns `200 OK` (body `ready`) once the runtime is ready to accept work, or `503 Service Unavailable` (body `not ready`) before then. Use this as a readiness/startup gate.

```console
$ curl -fsS http://localhost:8080/health
{"status":"healthy","uptime_secs":42, ...}

$ curl -fsS http://localhost:8080/ready
ready
```

### `aa-gateway` — TCP liveness on `:50051`

In the [container topology](docker-containers.md#the-governed-topology) the gateway runs in **legacy gRPC mode** (`--policy … --listen 0.0.0.0:50051`, no `--mode`). In that mode it serves gRPC only and exposes **no HTTP health endpoint** — there is no `/healthz` to curl. Gate its liveness with a **TCP connect to the gRPC port `:50051`** from the host (the `aa-gateway` image is distroless, so an in-container `CMD-SHELL` probe cannot run either — see [Docker & Containers](docker-containers.md#health-checking)).

```console
# TCP-level liveness — succeeds once the gateway is accepting gRPC connections.
$ nc -z localhost 50051 && echo "gateway up"
gateway up
```

A standard gRPC `grpc.health.v1.Health` service on `:50051` is being added (AAASM-4759); until it ships, the TCP check above is the gateway liveness signal for the container topology.

### HTTP health (`/healthz`, `/api/v1/health`) — local/remote mode only

The gateway *does* serve HTTP health endpoints, but **only when launched in a non-default mode** — not in the legacy-gRPC container topology on this page:

- **`GET /healthz`** — process-liveness in **local** and **remote** mode. Returns `200 OK` with a small JSON body (e.g. `{"mode":"local","version":"...","storage":"sqlite","uptime_secs":...}`).
- **`GET /api/v1/health`** — REST API health in **local** mode only (`--mode local`), mounted by the **same `aa-gateway` process** (there is no separate `aa-api` container). Returns `200 OK` when every subsystem check passes, or `503 Service Unavailable` when any is degraded; the JSON body includes the build `version`, `api_version`, uptime, and a `checks` map for the policy engine, registry, audit, and alerts.

Local mode is a single-process dev topology, not the gateway + runtime container stack documented in [Docker & Containers](docker-containers.md); its HTTP port comes from that mode's own configuration rather than the `--listen` gRPC address.

---

## Prometheus metrics endpoint

The `aa-runtime` health/metrics server exposes a Prometheus text-format scrape endpoint.

| Setting | Value |
|---|---|
| Env var | `AA_METRICS_ADDR` |
| Default bind address | `0.0.0.0:8080` |
| Metrics path | `/metrics` |
| Scrape target | `http://<runtime-host>:8080/metrics` (with the default bind address) |

`AA_METRICS_ADDR` is the single environment variable that controls this server's bind address; the same server serves `/health`, `/ready`, and `/metrics`. Set it to change the interface or port, e.g. `AA_METRICS_ADDR=127.0.0.1:9090` to bind loopback only. (`0.0.0.0` is a *bind* address — point your scraper at a routable host/IP for the runtime, not at `0.0.0.0`.)

```console
$ curl -fsS http://localhost:8080/metrics
# Prometheus text exposition format
aa_events_received_total 0
aa_events_emitted_total 0
...
```

### Baseline metrics

The runtime pre-registers six baseline metrics at `0` on startup, so the `/metrics` surface is **stable from the very first scrape** (a metric never "appears late" the first time it is incremented). The names and types below are taken directly from the runtime source; the "What it represents" column is explanatory (the source registers names and types only, without HELP text). Additional metrics may appear as the runtime does work.

| Metric | Type | What it represents |
|---|---|---|
| `aa_events_received_total` | counter | Governance events the runtime has received |
| `aa_events_emitted_total` | counter | Events the runtime has emitted downstream |
| `aa_policy_violations_total` | counter | Policy violations observed |
| `aa_policy_evaluations_total` | counter | Policy evaluations performed (currently reports `0`; reserved for a forthcoming release) |
| `aa_active_connections` | gauge | Currently active connections |
| `aa_channel_utilization_ratio` | gauge | Internal channel utilization ratio |

> **Note:** these six are the *baseline* surface. Only `aa_active_connections` and `aa_channel_utilization_ratio` are gauges; the other four are counters. None are histograms. Because they start at `0`, an all-zero scrape shortly after startup is expected, not a sign of a broken exporter.

### Minimal scrape configuration

Point a Prometheus server at the runtime's metrics endpoint. A minimal `prometheus.yml` scrape job:

```yaml
scrape_configs:
  - job_name: aa-runtime
    metrics_path: /metrics
    static_configs:
      - targets: ["<runtime-host>:8080"]   # matches AA_METRICS_ADDR's port
```

Replace `<runtime-host>` with the address where the runtime is reachable (for the Docker Compose example, the runtime service's name/port on the compose network). If you override `AA_METRICS_ADDR`, update the target port to match.

For a liveness/health check outside Prometheus, probe the runtime's `/health` over HTTP (`curl -f` gates on its non-`200` status) and check the gateway with a TCP connect to `:50051` (`nc -z`) — in the container topology the gateway serves no HTTP health endpoint. The HTTP `/healthz` / `/api/v1/health` probes apply only if you run the gateway in local/remote mode, as noted above.

---

## Where to confirm these details

These endpoints live in the Apache-2.0 crates in the [`agent-assembly`](https://github.com/ai-agent-assembly/agent-assembly) repository, so you can verify them against the source you run:

- `aa-runtime/src/config.rs` — `AA_METRICS_ADDR` and its default.
- `aa-runtime/src/runtime.rs` and `aa-runtime/src/health/` — the health/metrics server and the baseline metrics.
- `aa-gateway/src/main.rs` — the mode resolver (`resolve_mode`); the default is `legacy-grpc`, which runs `serve_tcp` (gRPC only, no HTTP health) unless `--mode`/`AA_MODE` selects `local` or `remote`.
- `aa-gateway/src/routes/healthz.rs` — the `/healthz` liveness probe (local and remote modes only).
- `aa-api/src/routes/health.rs` — the `/api/v1/health` check (mounted by the gateway in local mode; there is no separately runnable `aa-api` container).

---

## Related documentation

- [Observe your agents in the Dashboard](dashboard.md) — the operator web UI on top of these surfaces, with light/dark screenshots of the Fleet and Audit Log views.
- [Open core boundary](open-core-boundary.md) — what the limited-function OSS stack includes vs. the SaaS feature set.
- [Cloud Deployment](cloud-deployment.md) — the managed SaaS platform, its SLA tiers, and on-call (SaaS only).
- [Security model](security-model.md) — the Telemetry layer and the broader defense-in-depth posture.
- [Troubleshooting](troubleshooting.md) — common issues when running the stack.

---

*Last reviewed: 2026-07-18 · AI Agent Assembly Team*
