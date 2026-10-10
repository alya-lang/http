# http

[![CI](https://github.com/alya-lang/http/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/http/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/http?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fhttp%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fhttp%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

Production-ready HTTP client, server, router, compression, and middleware toolkit for the Alya language ecosystem.

---

## 🌟 Features

- ⚡ **High Performance & Reactive I/O**: Fast protocol parsing and routing. Seamlessly binds to `event::EventLoop` for non-blocking reactive concurrency (`reactive_server`), handling thousands of concurrent connections with zero thread overhead.
- 🔌 **RFC 6455 WebSockets**: Full-duplex WebSocket server and client connections. Automatic handshake negotiation (`101 Switching Protocols`), RFC test-vector verified framing (text, binary, ping, pong, close), fragmented-message reassembly with interleaved control frames, protocol-error fails (1002), and callback-driven protocol drivers (`ws_on_message`, `ws_send`, etc.).
- 🗜️ **Web Compression Toolkit**: Native integration with `compress` supporting **Brotli (`br`)**, **Zstandard (`zstd`)**, **Gzip (`gzip`)**, and **Deflate (`deflate`)**. Automatic `Accept-Encoding` negotiation, server response compression middleware, pre-compressed static asset serving (`.br` and `.gz`), and client decompression.
- 🌐 **Full HTTP Client**: Supports GET, POST, PUT, DELETE, PATCH, HEAD, and OPTIONS with custom headers, query params, timeout handling, and automatic redirect following.
- 🔒 **Native HTTPS**: Real TLS 1.2 via `alya-lang/tls` (RSA key exchange, certificate verification, binary-safe bodies) — no subprocess, no shell, no temp files. Verified live against OpenSSL and Python TLS servers.
- 🚀 **HTTP Server & Context**: Built on low-level TCP sockets (`std/net`) or non-blocking event loops, offering intuitive request context (`HttpContext`), JSON responses, text responses, file serving, and status helpers.
- 🧭 **Handler Dispatch**: Register first-class handler functions (`router_on`, `router_on_get`, ...) and invoke them with `router_dispatch` — string action names keep working for match-only flows.
- 🔁 **Keep-Alive & Serve Loop**: Opt-in HTTP/1.1 connection reuse (`server_enable_keep_alive`), single-request reads (`server_read_request`), and blocking serve helpers (`server_serve_once`, `server_serve`).
- 🔐 **Protective Middleware**: HTTP Basic/Bearer/JWT auth (`mw_require_basic`, `mw_require_bearer`, `mw_require_jwt`), fixed-window rate limiting (`mw_apply_rate_limit`), and request ID tracing (`mw_request_id`) — all enforceable from `server_process` via `server_enable_*`.
- 🧩 **JSON & View Helpers**: Parse request bodies (`ctx_body_json`), answer string maps (`ctx_json_map`), and render Mustache templates as HTML (`ctx_render`, `ctx_render_file`).
- 🛣️ **Parametric Router & Route Groups**: Fast URL pattern matching with wildcard (`*path`) and named parameters (`:id`), plus subrouter groups with shared path prefixes and middleware chains.
- 🛡️ **Extensible Middleware**: Out-of-the-box middleware for Compression (`mw_apply_compression`), CORS (`cors_middleware`), request logging (`logger_middleware`), panic recovery (`recovery_middleware`), and static file serving (`static_middleware`).
- 🍪 **Cookie & Header Management**: RFC-compliant Cookie serialization/parsing (`Set-Cookie` and `Cookie` headers) and case-insensitive HTTP header operations.
- 📡 **Server-Sent Events & Chunked Transfer**: W3C SSE event formatting/parsing/streaming (`sse_event`, `sse_stream_parse`), `HttpContext` SSE/chunked helpers, and a reactive `EventLoop` SSE subscriber.
- 🧬 **Binary-Safe Bodies & Frames**: Byte-array HTTP bodies (`http_get_bytes`, `client_execute_bytes`, `*_bytes` codecs) and NUL-safe WebSocket binary frames over sync sockets.

---

## 📁 Project Architecture

```
http/
├── alya.toml               # Package manifest
├── src/
│   ├── lib.alya            # Public API facade & high-level constructors
│   ├── types.alya          # Core struct definitions (HttpRequest, HttpResponse, Route, etc.)
│   ├── core/
│   │   ├── status.alya     # HTTP status codes & standard status messages
│   │   ├── headers.alya    # Case-insensitive header dictionary helpers
│   │   ├── cookies.alya    # Cookie parsing, serialization, and Set-Cookie generation
│   │   ├── multipart.alya  # multipart/form-data parser and file upload types
│   │   ├── compression.alya# HTTP content encoding, negotiation, and compression engine
│   │   ├── protocol.alya   # HTTP/1.1 request & response parsing and serialization
│   │   └── sse.alya        # Server-Sent Events + HTTP/1.1 chunked transfer encoding
│   ├── router/
│   │   ├── route.alya      # Route definition and parameter extractor
│   │   ├── router.alya     # HTTP router and route registry
│   │   ├── dispatch.alya   # Handler-function dispatch over RouteMatch
│   │   └── group.alya      # Route grouping with prefix and sub-middlewares
│   ├── server/
│   │   ├── context.alya    # HttpContext request/response lifecycle helpers
│   │   ├── server.alya     # TCP socket server, request loop, and connection handler
│   │   └── reactive.alya   # Event-loop driven non-blocking reactive HTTP server adapter
│   ├── websocket/
│   │   ├── handshake.alya  # RFC 6455 WebSocket upgrade & SHA-1 handshake accept token
│   │   ├── frame.alya      # RFC 6455 frame serializer, deserializer, and masking
│   │   └── connection.alya # High-level WebSocket bidirectional connection driver
│   ├── client/
│   │   ├── client.alya     # HttpClient implementation with socket IO and redirect loop
│   │   ├── reactive.alya   # Event-loop driven non-blocking reactive HTTP/WS/SSE client
│   │   ├── tls_client.alya # Native HTTPS via alya-lang/tls (no curl bridge)
│   │   └── methods.alya    # Convenience functions (http_get, http_post, etc.)
│   └── middleware/
│       ├── auth.alya       # HTTP Basic, Bearer, and JWT (HS256) authentication gates
│       ├── rate_limit.alya # Fixed-window in-memory rate limiter (429)
│       ├── request_id.alya # X-Request-ID trace correlation
│       ├── compress.alya   # HTTP response compression middleware (Brotli, Zstd, Gzip, Deflate)
│       ├── cors.alya       # Cross-Origin Resource Sharing (CORS) handler
│       ├── logger.alya     # Request/response logging middleware
│       ├── recovery.alya   # Crash and exception recovery middleware
│       └── static.alya     # Static file serving with MIME detection & pre-compressed assets
├── examples/
│   ├── compression_demo.alya # Dedicated HTTP compression showcase
│   └── demo.alya           # Comprehensive usage demo
├── tests/                  # 31 test suites (100% passing)
│   ├── test_auth.alya
│   ├── test_bodies.alya
│   ├── test_bytes.alya
│   ├── test_client.alya
│   ├── test_compression.alya
│   ├── test_context.alya
│   ├── test_cookies.alya
│   ├── test_dispatch.alya
│   ├── test_forwarded.alya
│   ├── test_guards.alya
│   ├── test_headers.alya
│   ├── test_https.alya
│   ├── test_jwt.alya
│   ├── test_keepalive.alya
│   ├── test_limits.alya
│   ├── test_middleware.alya
│   ├── test_multipart.alya
│   ├── test_protocol.alya
│   ├── test_reactive_client.alya
│   ├── test_reactive_server.alya
│   ├── test_reactive_sse.alya
│   ├── test_reactive_ws_bytes.alya
│   ├── test_router.alya
│   ├── test_server.alya
│   ├── test_server_tls.alya
│   ├── test_sse.alya
│   ├── test_status.alya
│   ├── test_websocket_conn.alya
│   ├── test_websocket_frag.alya
│   ├── test_websocket_frame.alya
│   └── test_websocket_handshake.alya
└── benches/
    └── bench_basic.alya    # Performance micro-benchmarks
```

---

## 📦 Installation

Add `http` to your project's `alya.toml`:

```toml
[dependencies]
http = { git = "https://github.com/alya-lang/http", branch = "main" }
```

Or install it directly via the `alya` CLI:

```bash
alya add http --git https://github.com/alya-lang/http --branch main
alya install
```

### Package Features

| Feature | Default | Description |
|:---|:---:|:---|
| `compress` | ✅ | Response compression middleware (`compression()`, `compress()`/`decompress()`, `server_enable_compression`). Needs `Lib/compress`. |
| `event` | ✅ | Event-driven reactive server (`reactive_server`). Needs `Lib/event`. |
| `mime` | ✅ | Static file serving (`serve_static`, `mw_serve_static`). Needs `Lib/mime`. |
| `json` | ✅ | JSON body helpers (`ctx_body_json`, `ctx_json_map`). Needs `Lib/json`. |
| `jwt` | ✅ | JWT authentication (`mw_require_jwt`, `server_enable_jwt_auth`). Needs `Lib/jwt`. |
| `mustache` | ✅ | View rendering (`ctx_render`, `ctx_render_file`). Needs `Lib/mustache`. |

`crypto`, `url`, and `tls` stay required: WebSocket handshakes need SHA-1, clients need URL parsing, and TLS paths are woven through the server/client cores.

```bash
# Full build (default)
alya install
alya test

# Slim build (core client/server/router/middleware without the above)
alya install --no-default-features
alya test --no-default-features
```

---

## 🚀 Quick Start

### 1. HTTP Server & Router

```alya
import "http" as http

function get_user(ctx)
    let uid = http::ctx_param(ctx, "id")
    return http::ctx_json_map(ctx, {"userId": uid}, 200)
end

function main()
    let app = http::router()

    # Handler-function route with path parameter
    http::router_on_get(app, "/users/:id", get_user)

    let srv = http::server(8080, "127.0.0.1", app)
    http::server_start(srv)
    http::server_serve(srv, 1)
end

main()
```

> [!NOTE]
> `router_get/post/...` accept legacy string action names for match-only flows;
> `router_on` / `router_on_get` / ... register real handler functions invoked
> by `router_dispatch` (sync) or the reactive server.

### 2. Response Compression & Pre-compressed Static Assets

```alya
import "http" as http

function main()
    let srv = http::server(8080)

    # Enable automatic response compression (Brotli > Zstandard > Gzip > Deflate)
    # Responses >= 256 bytes will be compressed according to client's Accept-Encoding
    http::server_enable_compression(srv, 256)

    # Serve static directory with automatic .br / .gz pre-compressed asset detection
    http::server_enable_static(srv, "/static", "./public")
end

main()
```

### 3. HTTP Client

```alya
import "http" as http

function main()
    # Simple GET request
    let res = http::http_get("http://httpbin.org/get")
    say "Status: " + str(res.status_code)
    say "Body: " + res.body

    # Client instance with custom options
    let client = http::client(5000, 1, 3) # 5s timeout, follow redirects
    let post_res = client.post("http://httpbin.org/post", "{\"hello\":\"world\"}", "application/json")
    say "Response: " + post_res.body
end

main()
```

### 4. Cookies and Headers

```alya
import "http" as http

function main()
    # Create and serialize a secure cookie
    let c = http::cookie("session_id", "xyz123", "/", 3600, 1, 1, "Strict")
    let cookie_hdr = http::cookie_format(c)
    say "Set-Cookie: " + cookie_hdr

    # Parse cookies from incoming request header
    let parsed = http::cookie_parse_all("theme=dark; session_id=xyz123")
    say "Theme: " + parsed["theme"]
end

main()
```

### 5. Protection & Keep-Alive

```alya
import "http" as http

function main()
    let app = http::router()
    let srv = http::server(8080, "127.0.0.1", app)

    # Trace every response, require a token, throttle clients,
    # and reuse connections
    http::server_enable_request_id(srv, 1)
    http::server_enable_bearer_auth(srv, "tok-123")
    http::server_enable_rate_limit(srv, 100, 60000)
    http::server_enable_keep_alive(srv, 1, 100)

    http::server_start(srv)
    http::server_serve(srv, 1)
end

main()
```

---

## 📖 API Reference

### Client API

| Function | Parameters | Description |
|---|---|---|
| `http_get(url, headers)` | `url: string, headers: map` | Performs an HTTP GET request |
| `http_post(url, body, content_type, headers)` | `url: string, body: string, ...` | Performs an HTTP POST request |
| `http_put(url, body, content_type, headers)` | `url: string, body: string, ...` | Performs an HTTP PUT request |
| `http_patch(url, body, content_type, headers)` | `url: string, body: string, ...` | Performs an HTTP PATCH request |
| `http_delete(url, headers)` | `url: string, headers: map` | Performs an HTTP DELETE request |
| `http_query(url, body, content_type, headers)` | `url: string, body: string, ...` | Performs an HTTP QUERY request (IETF safe method with body) |
| `client_new(timeout_ms, follow_redirects, max_redirects)` | `timeout_ms: int, ...` | Instantiates a configured `HttpClient` |

### Server & Context API

| Function | Parameters | Description |
|---|---|---|
| `server_new(port, host, router)` | `port: int, host: string, router: HttpRouter` | Creates a new `HttpServer` instance |
| `server_enable_compression(server, min_length)` | `server: HttpServer, min_length: int` | Enables response compression middleware |
| `server_enable_static(server, prefix, dir)` | `server: HttpServer, prefix: string, dir: string` | Enables static file serving with `.br`/`.gz` pre-compressed support |
| `server_enable_cors(server, origin, methods, headers)` | `server: HttpServer, ...` | Enables CORS middleware |
| `server_enable_logger(server, enabled)` | `server: HttpServer, enabled: int` | Enables request logger middleware |
| `server_set_log_format(server, format)` | `server: HttpServer, format: string` | Sets a custom access-log template (`{method}`, `{path}`, `{status}`, `{latency}`, `{remote}`, `{id}`) |
| `server_enable_timeout(server, timeout_ms)` | `server: HttpServer, ...` | Sets socket I/O timeout for accepted connections (bounds slow clients, not handler runtime) |
| `format_log_ex(method, path, status, ...)` | `..., request_id, format` | Formats a log line from a custom template (build it by concatenation, see note below) |
| `cookie_sign(value, secret)` | `value, secret: string` | Signs a cookie value (`value.signature`, HMAC-SHA256) |
| `cookie_unsign(signed, secret)` | `signed, secret: string` | Verifies a signed value; `""` when forged |
| `ctx_cookie_signed(ctx, name, val, secret, ...)` | `ctx: HttpContext, ...` | Sets an HMAC-signed response cookie |
| `ctx_get_cookie_signed(ctx, name, secret, default)` | `ctx: HttpContext, ...` | Reads and verifies a signed cookie; default when missing/forged |
| `ctx_parse_cookies(ctx)` | `ctx: HttpContext` | (Re)parses the `Cookie` header into `req.cookies` |
| `context_json(ctx, status_code, json_str)` | `ctx: HttpContext, status: int, json: string` | Sends a JSON response with proper header |
| `context_text(ctx, status_code, text_str)` | `ctx: HttpContext, status: int, text: string` | Sends a plain text response |

### Compression API

| Function | Parameters | Description |
|---|---|---|
| `http_compress(data, encoding)` | `data: string, encoding: string` | Compresses string with `gzip`, `br`, `deflate`, `zstd` |
| `http_decompress(bytes, encoding)` | `bytes: list, encoding: string` | Decompresses byte array back into UTF-8 string |
| `http_negotiate_encoding(accept_header)` | `accept_header: string` | Negotiates best algorithm (`br` > `zstd` > `gzip` > `deflate`) |
| `http_is_encoding_supported(encoding)` | `encoding: string` | Returns 1 if encoding is supported, 0 otherwise |
| `compression(min_length)` | `min_length: int` | Creates a new `CompressionConfig` struct (default 256 bytes) |

### Router API

| Method | Parameters | Description |
|---|---|---|
| `router_new(not_found_action)` | `not_found: string` | Instantiates a new route registry |
| `router_get(r, pattern, handler)` | `pattern: string, handler: string` | Registers a GET route handler |
| `router_post(r, pattern, handler)` | `pattern: string, handler: string` | Registers a POST route handler |
| `router_put(r, pattern, handler)` | `pattern: string, handler: string` | Registers a PUT route handler |
| `router_delete(r, pattern, handler)` | `pattern: string, handler: string` | Registers a DELETE route handler |
| `router_patch(r, pattern, handler)` | `pattern: string, handler: string` | Registers a PATCH route handler |
| `router_query(r, pattern, handler)` | `pattern: string, handler: string` | Registers a QUERY route handler |
| `router_group_add(r, prefix, ...)` | `prefix: string, ...` | Registers a route under a group prefix |
| `router_on(r, method, pattern, handler)` | `method, pattern: string, handler: fn` | Registers a handler-function route for any method |
| `router_on_get/post/put/delete/patch/head/options/query/any(r, pattern, handler)` | `pattern: string, handler: fn` | Per-method handler-function shortcuts |
| `router_group_on(r, prefix, method, pattern, handler)` | `prefix: string, ...` | Registers a handler-function route under a group prefix |
| `group_on(g, method, pattern, handler)` | `method, pattern: string, handler: fn` | Registers a handler-function route in a group |

### Handler Dispatch API

| Function | Parameters | Description |
|---|---|---|
| `router_dispatch_match(m, ctx)` | `m: RouteMatch, ctx: HttpContext` | Invokes the matched handler function; returns 1 when invoked, 0 otherwise (miss, abort, legacy string action) |
| `router_dispatch(r, method, path, ctx)` | `r: HttpRouter, ...` | Matches, injects params, and dispatches; returns the `RouteMatch` |

### Sync Serve & Keep-Alive API

| Function | Parameters | Description |
|---|---|---|
| `server_enable_keep_alive(server, enabled, max_requests)` | `server: HttpServer, ...` | Enables HTTP/1.1 connection reuse (plaintext only) |
| `server_should_keep_alive(server, ctx)` | `server: HttpServer, ctx` | Returns 1 when the connection should stay open |
| `server_read_request(sock, timeout_ms)` | `sock: int, timeout_ms: int` | Reads one request from a connected socket (reuse primitive) |
| `server_serve_once(server, timeout_ms, max_requests, on_request)` | `server: HttpServer, ...` | Serves one connection end-to-end; returns request count |
| `server_serve(server, max_conns, ...)` | `server: HttpServer, ...` | Serves up to `max_conns` connections; returns total requests |

### Auth & Guards API

| Function | Parameters | Description |
|---|---|---|
| `mw_require_basic(ctx, user, pass, realm)` | `ctx: HttpContext, ...` | Enforces HTTP Basic auth; 401 + abort on failure |
| `mw_require_bearer(ctx, token)` | `ctx: HttpContext, token: string` | Enforces Bearer auth; 401 + abort on failure |
| `mw_require_jwt(ctx, secret, leeway)` | `ctx: HttpContext, ...` | Enforces JWT (HS256) auth; 401 + abort on failure (`jwt` feature) |
| `auth_basic_credentials(ctx)` | `ctx: HttpContext` | Parses Basic credentials into `{user, pass}` or null |
| `auth_bearer_token(ctx)` | `ctx: HttpContext` | Extracts the Bearer token or `""` |
| `server_enable_basic_auth(server, user, pass)` | `server: HttpServer, ...` | Enforces Basic auth in `server_process` |
| `server_enable_bearer_auth(server, token)` | `server: HttpServer, ...` | Enforces Bearer auth in `server_process` |
| `server_enable_jwt_auth(server, secret)` | `server: HttpServer, ...` | Enforces JWT auth in `server_process` (`jwt` feature; fails closed without it) |
| `server_enable_limits(server, max_header, max_body)` | `server: HttpServer, ...` | Caps header/body bytes; oversized reads get 413 |
| `server_enable_forwarded(server, enabled)` | `server: HttpServer, ...` | Trusts proxy headers for client identity (behind known proxies only) |
| `ctx_client_ip(ctx)` | `ctx: HttpContext` | Effective client IP (`X-Forwarded-For` → `X-Real-IP` → socket) |
| `ctx_scheme(ctx)` | `ctx: HttpContext` | Effective scheme (`X-Forwarded-Proto`, TLS, or `http`) |
| `rate_limit_new(limit, window_ms)` | `limit: int, window_ms: int` | Creates a fixed-window `RateLimiter` |
| `rate_limit_check(limiter, key)` | `limiter: RateLimiter, key: string` | Records a hit; 1 allowed, 0 denied |
| `mw_apply_rate_limit(ctx, limiter, key)` | `ctx: HttpContext, ...` | Enforces the limiter; 429 + `Retry-After` on denial |
| `server_enable_rate_limit(server, limit, window_ms)` | `server: HttpServer, ...` | Enforces rate limiting in `server_process` |
| `mw_request_id(ctx, header)` | `ctx: HttpContext, ...` | Echoes/generates `X-Request-ID`; mirrors to response |
| `server_enable_request_id(server, enabled)` | `server: HttpServer, ...` | Tags every response with a request ID |

### JSON & View API (optional features)

| Function | Parameters | Description |
|---|---|---|
| `ctx_body_json(ctx)` | `ctx: HttpContext` | Parses the request body as JSON; null when empty/invalid (`json` feature) |
| `ctx_json_value(ctx, val, status)` | `ctx: HttpContext, val: any, ...` | Sends any native value as recursive JSON (`json` feature) |
| `ctx_json_map(ctx, m, status)` | `ctx: HttpContext, m: map, ...` | Sends a flat string map as a quoted JSON object (`json` feature) |
| `ctx_render(ctx, template, data, status)` | `ctx: HttpContext, ...` | Renders a Mustache template string as HTML (`mustache` feature) |
| `ctx_render_file(ctx, path, data, status)` | `ctx: HttpContext, ...` | Renders a Mustache template file as HTML (`mustache` feature) |

> [!NOTE]
> **Brace literals:** Alya interpolates every `{name}` inside double-quoted
> strings, so log templates and Mustache tags cannot be written as single
> literals (`"{method}"` would interpolate). Build them by concatenation
> (`"{" + "method}"`, `"{{" + "name" + "}}"`) or from variables.

### Reactive Server API

| Function | Parameters | Description |
|---|---|---|
| `reactive_server(loop, port, host, router, on_req, on_ws)` | `loop: EventLoop, ...` | Spawns an event-loop bound reactive HTTP/WS server |
| `server_use_event_loop(srv, loop, on_req, on_ws)` | `srv: HttpServer, loop: EventLoop` | Binds an existing `HttpServer` to a non-blocking reactor |
| `server_close_reactive(srv)` | `srv: HttpServer` | Unregisters server watchers and closes all client streams |

### WebSocket API (RFC 6455)

| Function | Parameters | Description |
|---|---|---|
| `websocket(stream, is_server)` | `stream: TcpStream, is_server: int` | Wraps a non-blocking stream into a `WebSocketConnection` |
| `ws_send(ws, text)` | `ws: WebSocketConnection, text: string` | Sends a UTF-8 text message frame |
| `ws_send_binary(ws, data)` | `ws: WebSocketConnection, data: string` | Sends a binary message frame |
| `ws_ping(ws, data)` | `ws: WebSocketConnection, data: string` | Sends an RFC 6455 ping heartbeat frame |
| `ws_pong(ws, data)` | `ws: WebSocketConnection, data: string` | Sends an RFC 6455 pong heartbeat frame |
| `ws_close(ws, code, reason)` | `ws: WebSocketConnection, code: int, reason: string` | Performs clean close handshake |
| `ws_on_message(ws, callback)` | `ws: WebSocketConnection, cb: fn(ws, msg, is_bin)` | Registers message handler callback |
| `ws_on_close(ws, callback)` | `ws: WebSocketConnection, cb: fn(ws, code, reason)` | Registers connection close callback |
| `ws_on_ping(ws, callback)` | `ws: WebSocketConnection, cb: fn(ws, data)` | Registers incoming ping callback |
| `ws_on_pong(ws, callback)` | `ws: WebSocketConnection, cb: fn(ws, data)` | Registers incoming pong callback |
| `ws_on_error(ws, callback)` | `ws: WebSocketConnection, cb: fn(ws, reason)` | Registers protocol-error callback (1002 fail) |
| `ws_encode_text_start(text, mask, key)` | `text: string, mask: int, key` | First fragment of a fragmented text message |
| `ws_encode_binary_start(data, mask, key)` | `data, mask: int, key` | First fragment of a fragmented binary message |
| `ws_encode_continuation(data, is_final, mask, key)` | `data: string, is_final: int, mask: int, key` | Continuation fragment; reassembled on receipt |

> [!NOTE]
> **Fragmentation & NUL bytes:** `ws_feed` reassembles fragmented messages and interleaves
> control frames per RFC 6455 §5.4, and fails the connection with code 1002 on protocol
> violations (unknown opcode, fragmented/oversized control frame, bad masking). One platform
> limit applies: Alya strings cannot hold NUL bytes (the runtime raises `NUL byte cannot be
> represented in strings` — see alya-lang/alya#129), so frames whose wire form contains `0x00`
> (non-final continuations, 126–255 byte lengths, masked payload bytes XORing to zero) cannot
> round-trip through the string-based wire API. A bytes-based wire over `std/net`
> `tcp_send_bytes`/`tcp_recv_bytes` would lift this.

### SSE & Chunked Transfer API| Function | Parameters | Description |
|---|---|---|
| `sse_event(data, event, id, retry, comment)` | `data: string, ...` | Creates an `SseEvent` instance |
| `sse_format(data, event_name, id, retry, comment)` | `data: string, ...` | Formats an SSE wire string directly from values |
| `sse_format_event(event)` | `event: SseEvent` | Formats an `SseEvent` into wire format |
| `sse_parse(raw)` | `raw: string` | Parses one SSE block into an `SseEvent` |
| `sse_stream_parse(buffer)` | `buffer: string` | Splits a stream buffer into `(events, remainder)` |
| `chunk_encode(chunk)` | `chunk: string` | Encodes one HTTP/1.1 chunked-transfer chunk |
| `chunk_end()` | — | Returns the terminating `0\r\n\r\n` chunk |
| `chunk_decode(raw)` | `raw: string` | Decodes a chunked payload back to raw content |
| `reactive_sse(loop, url, headers, on_event, on_error, on_close)` | `loop: EventLoop, ...` | Subscribes to an SSE endpoint over the event loop |
| `reactive_sse_close(client)` | `client: ReactiveSseClient` | Closes an active reactive SSE subscription |
| `reactive_ws_connect_bytes(loop, url, on_open, on_message, on_error, on_close)` | `loop: EventLoop, ...` | Binary-wire WS client; message callbacks receive byte arrays |

### Binary Body & Byte Frame API

| Function | Parameters | Description |
|---|---|---|
| `http_get_bytes(url, headers)` | `url: string, headers: map` | GET with a binary-safe response body (`body_bytes`) |
| `http_post_bytes(url, body_bytes, content_type, headers)` | `url: string, bytes: array, ...` | POST with a byte-array body |
| `http_put_bytes(url, body_bytes, content_type, headers)` | `url: string, bytes: array, ...` | PUT with a byte-array body |
| `client_execute_bytes(client, method, url, headers, body_bytes)` | `client: HttpClient, ...` | Raw request with a byte-array body |
| `http_parse_request_bytes(raw, remote_addr)` | `raw: array, ...` | Parses a request from wire bytes (NUL-safe) |
| `http_parse_response_bytes(raw)` | `raw: array` | Parses a response from wire bytes (NUL-safe) |
| `http_format_request_bytes(req)` | `req: HttpRequest` | Serializes a request to wire bytes |
| `http_format_response_bytes(res)` | `res: HttpResponse` | Serializes a response to wire bytes |
| `request_body_bytes(req)` | `req: HttpRequest` | Request body as bytes (text converted as needed) |
| `response_body_bytes(res)` | `res: HttpResponse` | Response body as bytes (text converted as needed) |
| `ws_encode_binary_bytes(data, mask, mask_key)` | `data: array, ...` | Binary WS frame with byte-array payload |
| `ws_frame_parse_bytes(raw, offset)` | `raw: array, offset: int` | Parses one WS frame from bytes |
| `ws_feed_bytes(ws, chunk)` | `ws: WebSocketConnection, chunk: array` | Feeds byte-array data into a connection |
| `ws_send_bytes(ws, data)` | `ws: WebSocketConnection, data: array` | Sends a binary frame with bytes |

> [!NOTE]
> The reactive binary WS client needs the `event` package with byte transport
> (`set_binary`, `write_bytes`, `on_data_bytes`, event v0.1.0 re-release or later).
> The reactive *server* still serves text/upgrade flows only.

---

## 🧪 Running Tests & Benchmarks

Run all 31 test suites using `alya`:

```bash
alya test
```

Run individual test files:

```bash
alya run tests/test_compression.alya
alya run tests/test_protocol.alya
alya run tests/test_router.alya
alya run tests/test_cookies.alya
```

Run benchmarks:

```bash
alya run benches/bench_basic.alya
```

Run the demo examples:

```bash
alya run examples/demo.alya
alya run examples/compression_demo.alya
```

Check code formatting:

```bash
alya fmt . --check
```

Run static code linter:

```bash
alya lint . --check
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository and clone it locally
2. Install the package tools:
   ```bash
   alya install
   ```
3. Create your feature branch:
   ```bash
   git checkout -b feature/my-feature
   ```
4. Verify tests and code formatting before opening a PR:
   ```bash
   alya test
   alya fmt . --check
   ```
5. Commit your changes:
   ```bash
   git commit -m "feat: add feature description"
   ```
6. Open a Pull Request on GitHub.

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.