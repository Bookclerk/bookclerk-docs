**bookclerkd > http_error**

# Module: http_error

## Contents

**Functions**

- [`brand_error_responses`](#brand_error_responses) - Rewrite empty 4xx/5xx responses into Bookclerk-branded HTML or JSON.
- [`document_error`](#document_error) - Branded HTML error for browser document routes (`GET /invite`, …).

---

## bookclerkd::http_error::brand_error_responses

*Function*

Rewrite empty 4xx/5xx responses into Bookclerk-branded HTML or JSON.

Negotiation prefers the request `Content-Type`, then `Accept`, then whether
the path looks like an API route (`/api/…` and legacy control-plane paths).
Responses that already set `Content-Type` (JSON handlers, static files, SPA)
are left unchanged.

```rust
fn brand_error_responses(req: axum::extract::Request, next: axum::middleware::Next) -> axum::response::Response
```



## bookclerkd::http_error::document_error

*Function*

Branded HTML error for browser document routes (`GET /invite`, …).

# Arguments

* `status` - HTTP status written on the response.
* `message` - Operator-facing explanation shown in the page body.

# Returns

HTML response with `Content-Type: text/html`.

```rust
fn document_error(status: axum::http::StatusCode, message: &'static str) -> axum::response::Response
```



