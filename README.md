# rust-zero

[![CI](https://github.com/kevwan/rust-zero/actions/workflows/rust.yml/badge.svg)](https://github.com/kevwan/rust-zero/actions/workflows/rust.yml)
[![Crates.io](https://img.shields.io/crates/v/rust-zero-core.svg)](https://crates.io/crates/rust-zero-core)
[![Documentation](https://docs.rs/rust-zero-core/badge.svg)](https://docs.rs/rust-zero-core)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

Production building blocks for Rust services, from HTTP and gRPC to discovery, resilience,
observability, gateways, and MCP. Inspired by [go-zero](https://github.com/zeromicro/go-zero),
rust-zero combines an opinionated production stack with modular crates, so applications only pay
for the transports and integrations they use.

> [!IMPORTANT]
> rust-zero is currently a `0.1.0-alpha` prerelease. It is suitable for evaluation and early
> adopters, but minor releases may change APIs before 1.0. See the [compatibility policy](#compatibility)
> and [stabilization gates](STABILIZATION.md) before adopting it in production.

## Why rust-zero?

- **Production defaults:** timeouts, recovery, request IDs, metrics, tracing, load shedding, and
  graceful shutdown are built into the standard server stacks.
- **One toolkit, multiple transports:** build REST, gRPC, HTTP-to-gRPC gateways, and MCP services
  with shared configuration, authentication, discovery, and lifecycle primitives.
- **Use only what you need:** external stores, etcd, Kubernetes, telemetry, and profiling are
  feature-gated; the core runtime has no default external-service adapters.
- **Operationally testable:** CI covers the Rust 1.89 MSRV, feature combinations, documentation,
  publishable packages, benchmark artifacts, and real external backends.

## Quick start

You need [Rust 1.89 or newer](https://www.rust-lang.org/tools/install). Clone the repository and
run the self-contained REST demo—no database or external service is required:

```bash
git clone https://github.com/kevwan/rust-zero.git
cd rust-zero
cargo run -p rust-zero-demo
```

In another terminal:

```bash
curl http://127.0.0.1:8080/
# Hello world!
```

The demo is a small Actix Web service with structured request logging, a 10-second timeout,
concurrency control, and token-bucket rate limiting. Its complete source is in
[`demo/src/main.rs`](demo/src/main.rs).

To use rust-zero in an existing project, add only the crates you need:

```bash
cargo add rust-zero-core rust-zero-rest
```

Package names use the `rust-zero-` prefix. Some library names are intentionally shorter, so the
corresponding imports are:

```rust
use rust_zero_core::CircuitBreaker;
use rest::RestServerConfig;
```

For a new service, install the companion CLI and generate a runnable REST project:

```bash
cargo install rust-zero-cli
rust-zero new users-api
cd users-api
cargo run
```

`rust-zero proto`, `rust-zero openapi`, `rust-zero docker`, and `rust-zero kubernetes` add focused
protobuf, route, container, and deployment scaffolding. Generated files are never overwritten
unless `--force` is supplied.

## Choose a crate

All public crates are released at the same version.

| Crate | Use it for |
| --- | --- |
| [`rust-zero-core`](https://docs.rs/rust-zero-core) | Configuration, resilience, discovery, metrics, logging, caching, stores, and service lifecycle |
| [`rust-zero-rest`](https://docs.rs/rust-zero-rest) | Actix Web servers, clients, middleware, extractors, SSE, static files, and serverless handlers |
| [`rust-zero-rpc`](https://docs.rs/rust-zero-rpc) | Tonic gRPC clients and servers, health, balancing, discovery, and transport policies |
| [`rust-zero-gateway`](https://docs.rs/rust-zero-gateway) | Streaming HTTP proxying and descriptor/reflection-driven gRPC transcoding |
| [`rust-zero-mcp`](https://docs.rs/rust-zero-mcp) | MCP servers using Streamable HTTP, legacy HTTP+SSE, tools, resources, and prompts |
| [`rust-zero-mapreduce`](https://docs.rs/rust-zero-mapreduce) | Bounded asynchronous MapReduce execution |
| [`rust-zero-cli`](https://crates.io/crates/rust-zero-cli) | Project, protobuf, OpenAPI, Docker, and Kubernetes scaffolding via the `rust-zero` binary |

## Explore by task

- Build a production REST stack: [REST services](#rest-services)
- Run a gRPC service: [RPC](#rpc)
- Proxy HTTP or transcode JSON to gRPC: [Gateway routing](#gateway-routing)
- Add runtime primitives: [Core runtime](#core-runtime)
- Add service discovery: [etcd](#etcd-configuration-and-discovery) or
  [Kubernetes](#kubernetes-discovery)
- Connect Redis, SQL, or MongoDB: [External stores](#external-stores)
- Expose health, metrics, and profiling: [Internal diagnostics](#internal-diagnostics)
- Compare runtime coverage with go-zero: [Feature parity](FEATURE_PARITY.md)

## Runnable examples

| Example | Command | External dependency |
| --- | --- | --- |
| REST middleware demo | `cargo run -p rust-zero-demo` | None |
| gRPC echo server | `cargo run -p rust-zero-rpc --example echo_server` | None |
| HTTP + gRPC gateway | `cargo run -p rust-zero-gateway --example mixed_upstreams` | None |
| MCP server | `cargo run -p rust-zero-mcp --example server` | None |
| etcd discovery | `cargo run -p rust-zero-core --features etcd --example etcd_discovery` | etcd |
| Kubernetes discovery | `cargo run -p rust-zero-core --features kubernetes --example kubernetes_discovery` | Cluster or kubeconfig |
| OpenTelemetry export | `cargo run -p rust-zero-core --features telemetry --example telemetry` | OTLP collector |
| Redis, SQL, and MongoDB | `cargo run -p rust-zero-core --features stores --example external_stores` | Configured stores |

Examples that use external systems read connection settings from environment variables; open the
linked [example sources](core/examples) for the accepted names and defaults. The mixed gateway
example starts its own HTTP and gRPC upstreams, making it the quickest way to try transcoding.

## What is included

rust-zero provides composable production primitives plus a focused, optional scaffolding CLI:

- REST and gRPC authentication, middleware, clients, servers, streaming, and graceful draining.
- Circuit breaking, bounded retries, deadlines, adaptive shedding, rate limiting, load balancing,
  single-flight work, queues, batching, and MapReduce.
- Typed JSON/JSON5/TOML/YAML configuration with environment expansion and dynamic updates.
- Prometheus metrics, structured logging, W3C trace propagation, optional OpenTelemetry export,
  health reporting, diagnostics, and profiling.
- In-memory, etcd, and Kubernetes discovery; optional Redis, SQLite/PostgreSQL/MySQL, and MongoDB
  adapters; and HTTP/gRPC gateway routing.

The detailed [feature-parity matrix](FEATURE_PARITY.md) defines exact coverage and deliberate
ecosystem boundaries. [BACKLOG.md](BACKLOG.md) tracks audit status and remaining work, while
[STABILIZATION.md](STABILIZATION.md) separates completed engineering coverage from the sustained
operational and independent-adoption evidence still required to leave alpha.

## Compatibility

rust-zero supports Rust 1.89 and newer. Linux is the primary deployment target; macOS is supported
for local development. CI checks the locked dependency graph on Rust 1.89 with all features and
also tests minimal, adapter, telemetry, and all-feature combinations on stable Rust.

An MSRV increase is announced in the [release notes](CHANGELOG.md) and requires at least a minor
version change. Before 1.0, minor releases may contain API changes; patch releases preserve public
APIs except when a security or soundness fix makes that impossible. After 1.0, the project follows
[Semantic Versioning](https://semver.org/).

## Configuration

Configuration files may use JSON, JSON5, TOML, YAML, or YML. Environment variables can be expanded
with `$VAR` or `${VAR}`. JSON5 additionally supports comments, trailing commas, single-quoted
strings, and unquoted object keys.

REST and gRPC share `JwtClaimProjection`, `RequestSignatureVerifier`, and the stable `AuthFailure`
taxonomy. Request signatures use HMAC-SHA256 with named rotation keys and a configurable clock-skew
window; REST sends the values in `x-rust-zero-*` headers and gRPC uses corresponding metadata.

---

## Core runtime

`rust-zero-core` supplies the production controls that are shared by transport services and
background workers:

```rust
use rust_zero_core::{
    retry, AdaptiveShedder, CircuitBreaker, CircuitBreakerConfig, ConsistentHash,
    LoadShedderConfig, RetryPolicy, RollingCircuitBreakerConfig,
};
use std::time::Duration;

let breaker = CircuitBreaker::new(CircuitBreakerConfig::new(5, Duration::from_secs(30)));
let adaptive_breaker = CircuitBreaker::new(CircuitBreakerConfig::rolling(
    RollingCircuitBreakerConfig::new(),
));
let shedder = AdaptiveShedder::new(LoadShedderConfig::production(128));
let mut backends = ConsistentHash::new(100);
backends.add("http://users-a:8080");

let _permit = shedder.try_acquire();
let _response = breaker.execute(|| {
    // call a selected backend
    Ok::<_, std::io::Error>(())
})?;

// Rolling breakers expose current accepted/total history, drop probability, and lifetime outcomes.
let _adaptive_health = adaptive_breaker.snapshot();

retry(RetryPolicy::new(3, Duration::from_millis(50)), || async {
    Ok::<_, std::io::Error>(())
})
.await?;
```

Dynamic configuration updates are parsed before publication, so consumers always observe a
complete, last-known-good typed value:

```rust
use rust_zero_core::{ConfigFormat, DynamicConfig};

let limits = DynamicConfig::<std::collections::HashMap<String, u64>>::new(
    "requests = 100",
    ConfigFormat::Toml,
)?;
let mut changes = limits.subscribe();
limits.update("requests = 200")?;
changes.changed().await?;
assert_eq!(changes.borrow().value()["requests"], 200);
```

Configuration is deserialized directly into service types, selected by file extension, and
expands `$VAR` or `${VAR}` values from the process environment:

```rust
use rust_zero_core::{load_config, ServiceConfig};

let config: ServiceConfig = load_config("etc/users.toml")?;
assert_eq!(config.address(), "0.0.0.0:8080");
```

## OpenTelemetry

Enable the `telemetry` feature on `rust-zero-core` and whichever transport crates need
instrumentation. OTLP exporters are opt-in so applications that only need local W3C propagation
do not pay their dependency or runtime cost.

```rust
use rust_zero_core::{OtlpTransport, Telemetry, TelemetryConfig};

let telemetry = Telemetry::init(
    TelemetryConfig::new(
        "users-api",
        "http://otel-collector:4317",
        OtlpTransport::Grpc,
    )
    .with_sample_ratio(0.1),
)?;

// Keep `telemetry` alive for the service lifetime. Dropping it flushes and
// shuts down the exporter.
```

For REST, use `OpenTelemetryTracing` in place of `TraceContextMiddleware`. It records method,
path, host, status, errors, and exposes the matching `TraceContext` through request extensions:

```rust
use rest::OpenTelemetryTracing;

// App::new().wrap(OpenTelemetryTracing::new())
```

For gRPC, apply the server layer to the Tonic builder and wrap client channels:

```rust
use rpc::RpcTelemetryLayer;

let server = tonic::transport::Server::builder()
    .layer(RpcTelemetryLayer::server());
let traced_channel = RpcTelemetryLayer::client().wrap(channel);
```

Unary gRPC calls can use transport-aware resilience without counting application statuses such as
`InvalidArgument` or `NotFound` as dependency failures:

```rust
use rpc::{RpcCircuitBreaker, RpcLoadShedder};
use rust_zero_core::{CircuitBreakerConfig, LoadShedderConfig};
use std::time::Duration;

let breaker = RpcCircuitBreaker::new(CircuitBreakerConfig::new(5, Duration::from_secs(30)));
let response = breaker.call(|| client.echo(request)).await?;

let shedder = RpcLoadShedder::new(LoadShedderConfig::production(1_024));
let response = shedder.call(|| service.echo(request)).await?;
# Ok::<(), tonic::Status>(())
```

## REST services

The REST middleware is composable and returns standard HTTP responses when protection activates:

| Middleware | Response |
| --- | --- |
| `Timeout` | `504 Gateway Timeout` |
| `ConcurrencyLimit` | `503 Service Unavailable` |
| `AdaptiveLoadShed` | `503 Service Unavailable` |
| `RateLimit` | `429 Too Many Requests` with `Retry-After` |

```rust
use actix_web::{App, HttpServer};
use rest::{
    BearerAuth, ConcurrencyLimit, Cors, LoggingMiddleware, RateLimit, Recover, RequestId,
    SecurityHeaders, Timeout,
};
use std::time::Duration;

let concurrency_limit = ConcurrencyLimit::new(1_024);
let rate_limit = RateLimit::new(1_000, 2_000);

HttpServer::new(move || {
    App::new()
        .wrap(LoggingMiddleware)
        .wrap(RequestId::new())
        .wrap(Recover::new())
        .wrap(SecurityHeaders::new())
        .wrap(BearerAuth::new(|token| (token == "secret").then_some(())))
        .wrap(Cors::permissive())
        .wrap(Timeout::new(Duration::from_secs(10)))
        .wrap(concurrency_limit.clone())
        .wrap(rate_limit.clone())
})
```

Register `HttpMetrics` in a shared `Metrics` registry to emit Prometheus-compatible request
counts, latency histograms, and in-flight gauges labeled by method, route, and response status.
The standard REST server enables CPU- and throughput-aware adaptive shedding by default and also
counts timeout, fixed-concurrency, adaptive-shed, and rate-limit rejections. Its load-shed settings
control the process CPU threshold, rolling bucket duration/count, and post-overload cooldown.
Permits remain active until HTTP or gRPC response streams finish, so long-lived work contributes to
latency and in-flight measurements. Attach a shared `HttpClientMetrics` instance with
`HttpClient::with_metrics` to record bounded service, method, status/error, latency, and in-flight
client metrics without labeling raw URLs.

For long-lived EventSource responses, `sse_response_with_backpressure` observes whether the
downstream transport requests the next frame. `SseBackpressurePolicy` records slow consumers after
the soft threshold and terminates the body stream after the hard threshold; source-side idle time
does not count against either limit. Register `SseMetrics` in the same metrics registry to expose
the bounded `slow` and `disconnected` outcomes.

For the standard production stack, load `RestServerConfig` and provide only application routes:

```rust
use actix_web::{web, HttpResponse};
use rest::{RestCorsConfig, RestServer, RestServerConfig};
use rust_zero_core::load_config;

let mut config: RestServerConfig = load_config("etc/users.toml")?;
config.cors = Some(
    RestCorsConfig::new(["https://console.example"])
        .with_methods(["GET", "POST"])
        .with_allowed_headers(["authorization", "content-type"])
        .with_exposed_headers(["x-request-id"])
        .with_credentials(true)
        .with_max_age(Some(600)),
);
let server = RestServer::new(config)?.run(|routes| {
    routes.route("/healthz", web::get().to(HttpResponse::Ok));
})?;
server.await?;
# Ok::<(), Box<dyn std::error::Error>>(())
```

The same policy can be loaded from a `cors` configuration table. Use a single `"*"` entry to
allow any origin, standard method, or header; wildcard origins are reflected rather than emitted
as `*`, so credentialed requests remain standards-compliant. Set `automatic_preflight = false` to
route `OPTIONS` requests through application handlers. Use `with_max_age(None)` (or JSON/YAML
`null`) to omit the preflight cache header. CORS is disabled when the `cors` table is absent.

Install a response policy on the standard server and extract it as Actix application data from
handlers. Mappers receive the current request, so envelopes can include request identity without
global mutable hooks:

```rust
use actix_web::{web, HttpRequest};
use rest::{ApiError, ResponsePolicy, RestServer};
use serde_json::json;

let policy = ResponsePolicy::new().with_success_mapper(|request, data| {
    json!({
        "request_id": request.headers().get("x-request-id").and_then(|v| v.to_str().ok()),
        "data": data,
    })
});
let server = RestServer::new(Default::default())?.with_response_policy(policy);

async fn handler(request: HttpRequest, policy: web::Data<ResponsePolicy>) -> actix_web::HttpResponse {
    policy.respond(&request, Ok::<_, ApiError>(json!({"ready": true})))
}
# Ok::<(), Box<dyn std::error::Error>>(())
```

Route policies are matched by HTTP method and the registered Actix route pattern. Group settings
are inherited, while a route can override them or set `public = true` to opt out of group JWT.
The optional `middleware` list refers to application middleware registered on `RestServer`; it runs
in declaration order and can short-circuit or wrap the downstream response:

```toml
[[route_groups]]
prefix = "/api"
timeout_ms = 2000
max_body_bytes = 1048576
middleware = ["api-key"]

[route_groups.jwt]
secret = "${API_JWT_SECRET}"
leeway_seconds = 30

[[route_groups.routes]]
method = "GET"
path = "/users/{id}"

[[route_groups.routes]]
method = "GET"
path = "/events"
public = true
sse = true
priority = true
```

```rust
use actix_web::{dev::ServiceRequest, HttpResponse};
use rest::{RestServer, RouteMiddlewareNext};

let server = RestServer::new(Default::default())?.with_route_middleware(
    "api-key",
    |request: ServiceRequest, next: RouteMiddlewareNext| async move {
        if !request.headers().contains_key("x-api-key") {
            return Ok(request
                .into_response(HttpResponse::Forbidden().finish())
                .map_into_boxed_body());
        }
        next.call(request).await
    },
)?;
# Ok::<(), Box<dyn std::error::Error>>(())
```

Use `serve_until` (or `serve_on_until` with a pre-bound listener) to connect an application signal
future to graceful Actix draining.

Static files can be installed as the default-service fallback without shadowing explicit API
routes. Embedded assets take precedence when combined with a directory:

```rust
use rest::{EmbeddedAsset, RestServer, StaticAssets};

let assets = StaticAssets::directory("public")?
    .with_embedded("version.txt", EmbeddedAsset::inferred("1.0.0"))?;
let server = RestServer::new(Default::default())?.with_static_assets(assets);
# Ok::<(), Box<dyn std::error::Error>>(())
```

For serverless platforms, build the routes once and translate each platform event into a
`ServerlessRequest`. The fully buffered response retains status, headers, and binary body bytes:

```rust
use actix_web::{http::Method, web, HttpResponse};
use rest::{RestServer, ServerlessRequest};

let handler = RestServer::new(Default::default())?
    .serverless_handler(|routes| {
        routes.route("/healthz", web::get().to(HttpResponse::Ok));
    })
    .await?;
let response = handler
    .call(ServerlessRequest::new(
        Method::GET,
        "/healthz".parse()?,
        web::Bytes::new(),
    ))
    .await?;
assert!(response.status.is_success());
# Ok::<(), Box<dyn std::error::Error>>(())
```

For APIs that require application-layer body secrecy, install `ContentEncryption` on the standard
server. Keep the previous key available while clients rotate; responses always use the provider's
current key:

```rust
use rest::{
    ContentEncryption, ContentEncryptionKey, RestServer, StaticContentKeyProvider,
};

let current = ContentEncryptionKey::new("2026-08", [7_u8; 32])?;
let previous = ContentEncryptionKey::new("2026-07", [3_u8; 32])?;
let keys = StaticContentKeyProvider::new(current).with_decryption_key(previous)?;
let server = RestServer::new(Default::default())?
    .with_content_encryption(ContentEncryption::new(keys, 4 * 1024 * 1024));
# Ok::<(), Box<dyn std::error::Error>>(())
```

Non-empty requests send `x-content-encryption: aes-256-gcm-v1` and `x-content-key-id`. The body is
standard base64 over `RZC1 || 12-byte nonce || ciphertext || 16-byte GCM tag`. Authentication binds
the request method and URI; response authentication also binds the HTTP status. The exported key
helpers implement the exact client-side format. This protects body confidentiali