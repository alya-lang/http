# http

[![CI](https://github.com/alya-lang/http/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/http/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/http?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fhttp%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fhttp%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

Production-ready HTTP client, server, router, and middleware toolkit for the Alya language ecosystem.

---

## 🌟 Features

- ⚡ **High Performance**: Optimized protocol parsing, header serialization, and fast routing. Micro-benchmarks demonstrate >100,000 ops in hundreds of milliseconds.
- 🌐 **Full HTTP Client**: Supports GET, POST, PUT, DELETE, PATCH, HEAD, and OPTIONS with custom headers, query params, timeout handling, and automatic redirect following.
- 🚀 **HTTP Server & Context**: Built on low-level TCP sockets (`std/net`), offering intuitive request context (`HttpContext`), JSON responses, text responses, file serving, and status helpers.
- 🛣️ **Parametric Router & Route Groups**: Fast URL pattern matching with wildcard (`*path`) and named parameters (`:id`), plus subrouter groups with shared path prefixes and middleware chains.
- 🛡️ **Extensible Middleware**: Out-of-the-box middleware for CORS (`cors_middleware`), request logging (`logger_middleware`), panic recovery (`recovery_middleware`), and static file serving (`static_middleware`).
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
│       ├── cors.alya       # Cross-Origin Resource Sharing (CORS) handler
│       ├── logger.alya     # Request/response logging middleware
│       ├── recovery.alya   # Crash and exception recovery middleware
│       └── static.alya     # Static file serving middleware with MIME detection
├── examples/
│   └── demo.alya           # Comprehensive usage demo
├── tests/                  # 9 comprehensive test suites
│   ├── test_client.alya
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
http = { git = "https://github.com/alya-lang/http", tag = "v0.1.0" }
```

Or install it directly via the `alyac` CLI:

```bash
alyac add http --git https://github.com/alya-lang/http --tag v0.1.0
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

    # Start listening on port 8080
    let srv = http::server(8080, "127.0.0.1", app)
    say "Server running on http://127.0.0.1:8080"
    # srv.listen()
end

main()
```

### 2. HTTP Client

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

### 3. Cookies and Headers

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
| `http_delete(url, headers)` | `url: string, headers: map` | Performs an HTTP DELETE request |
| `client_new(timeout_ms, follow_redirects, max_redirects)` | `timeout_ms: int, ...` | Instantiates a configured `HttpClient` |

### Server & Context API

| Function | Parameters | Description |
|---|---|---|
| `server_new(port, host, router)` | `port: int, host: string, router: HttpRouter` | Creates a new `HttpServer` instance |
| `http_listen(srv)` | `srv: HttpServer` | Starts the socket listening loop |
| `context_json(ctx, status_code, json_str)` | `ctx: HttpContext, status: int, json: string` | Sends a JSON response with proper header |
| `context_text(ctx, status_code, text_str)` | `ctx: HttpContext, status: int, text: string` | Sends a plain text response |
| `context_file(ctx, file_path)` | `ctx: HttpContext, file_path: string` | Reads and serves a local file with MIME type |

### Router API

| Method | Parameters | Description |
|---|---|---|
| `router_new(not_found_action)` | `not_found: string` | Instantiates a new route registry |
| `router.get(pattern, handler)` | `pattern: string, handler: string` | Registers a GET route handler |
| `router.post(pattern, handler)` | `pattern: string, handler: string` | Registers a POST route handler |
| `router.put(pattern, handler)` | `pattern: string, handler: string` | Registers a PUT route handler |
| `router.delete(pattern, handler)` | `pattern: string, handler: string` | Registers a DELETE route handler |
| `router.group(prefix)` | `prefix: string` | Creates a new `RouteGroup` under this router |

---

## 🧪 Running Tests & Benchmarks

Run all 9 test suites using `alyac`:

```bash
alyac test
```

Run individual test files:

```bash
alyac run tests/test_protocol.alya
alyac run tests/test_router.alya
alyac run tests/test_cookies.alya
```

Run benchmarks:

```bash
alyac run benches/bench_basic.alya
```

Run the demo example:

```bash
alyac run examples/demo.alya
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