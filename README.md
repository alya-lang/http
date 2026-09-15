# http

[![CI](https://github.com/alya-lang/http/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/http/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/http?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fhttp%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fhttp%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

Production-ready HTTP client, server, router, compression, and middleware toolkit for the Alya language ecosystem.

---

## 🌟 Features

- ⚡ **High Performance**: Optimized protocol parsing, header serialization, and fast routing. Micro-benchmarks demonstrate >100,000 ops in hundreds of milliseconds.
- 🗜️ **Web Compression Toolkit**: Native integration with `compress` supporting **Brotli (`br`)**, **Zstandard (`zstd`)**, **Gzip (`gzip`)**, and **Deflate (`deflate`)**. Automatic `Accept-Encoding` negotiation, server response compression middleware, pre-compressed static asset serving (`.br` and `.gz`), and client decompression.
- 🌐 **Full HTTP Client**: Supports GET, POST, PUT, DELETE, PATCH, HEAD, and OPTIONS with custom headers, query params, timeout handling, and automatic redirect following.
- 🚀 **HTTP Server & Context**: Built on low-level TCP sockets (`std/net`), offering intuitive request context (`HttpContext`), JSON responses, text responses, file serving, and status helpers.
- 🛣️ **Parametric Router & Route Groups**: Fast URL pattern matching with wildcard (`*path`) and named parameters (`:id`), plus subrouter groups with shared path prefixes and middleware chains.
- 🛡️ **Extensible Middleware**: Out-of-the-box middleware for Compression (`mw_apply_compression`), CORS (`cors_middleware`), request logging (`logger_middleware`), panic recovery (`recovery_middleware`), and static file serving (`static_middleware`).
- 🍪 **Cookie & Header Management**: RFC-compliant Cookie serialization/parsing (`Set-Cookie` and `Cookie` headers) and case-insensitive HTTP header operations.

---

## 📁 Architecture

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
│   │   ├── compression.alya# HTTP content encoding, negotiation, and compression engine
│   │   └── protocol.alya   # HTTP/1.1 request & response parsing and serialization
│   ├── router/
│   │   ├── route.alya      # Route definition and parameter extractor
│   │   ├── router.alya     # HTTP router and route registry
│   │   └── group.alya      # Route grouping with prefix and sub-middlewares
│   ├── server/
│   │   ├── context.alya    # HttpContext request/response lifecycle helpers
│   │   └── server.alya     # TCP socket server, request loop, and connection handler
│   ├── client/
│   │   ├── client.alya     # HttpClient implementation with socket IO and redirect loop
│   │   └── methods.alya    # Convenience functions (http_get, http_post, etc.)
│   └── middleware/
│       ├── compress.alya   # HTTP response compression middleware (Brotli, Zstd, Gzip, Deflate)
│       ├── cors.alya       # Cross-Origin Resource Sharing (CORS) handler
│       ├── logger.alya     # Request/response logging middleware
│       ├── recovery.alya   # Crash and exception recovery middleware
│       └── static.alya     # Static file serving with MIME detection & pre-compressed assets
├── examples/
│   ├── compression_demo.alya # Dedicated HTTP compression showcase
│   └── demo.alya           # Comprehensive usage demo
├── tests/                  # 10 comprehensive test suites (100% passing)
│   ├── test_client.alya
│   ├── test_compression.alya
│   ├── test_context.alya
│   ├── test_cookies.alya
│   ├── test_headers.alya
│   ├── test_middleware.alya
│   ├── test_protocol.alya
│   ├── test_router.alya
│   ├── test_server.alya
│   └── test_status.alya
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

Or install it directly via the `alyac` CLI:

```bash
alyac add http --git https://github.com/alya-lang/http --branch main
alyac install
```

---

## 🚀 Quick Start

### 1. HTTP Server & Router

```alya
import "http" as http

function main()
    let app = http::router()

    # Route with path parameter
    app.get("/users/:id", "get_user")

    # JSON API response route
    app.post("/api/echo", "post_echo")

    # Start listening on port 8080 with compression enabled
    let srv = http::server(8080, "127.0.0.1", app)
    http::server_enable_compression(srv, 256)
    say "Server running on http://127.0.0.1:8080"
end

main()
```

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

---

## 🧪 Running Tests & Benchmarks

Run all 10 test suites using `alyac`:

```bash
alyac test
```

Run individual test files:

```bash
alyac run tests/test_compression.alya
alyac run tests/test_protocol.alya
alyac run tests/test_router.alya
alyac run tests/test_cookies.alya
```

Run benchmarks:

```bash
alyac run benches/bench_basic.alya
```

Run the demo examples:

```bash
alyac run examples/demo.alya
alyac run examples/compression_demo.alya
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository and clone it locally
2. Install the package tools:
   ```bash
   alyac install
   ```
3. Create your feature branch:
   ```bash
   git checkout -b feature/my-feature
   ```
4. Verify tests and code formatting before opening a PR:
   ```bash
   alyac test
   alyac fmt . --check
   ```
5. Commit your changes:
   ```bash
   git commit -m "feat: add feature description"
   ```
6. Open a Pull Request on GitHub.

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.