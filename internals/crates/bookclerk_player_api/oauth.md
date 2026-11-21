# bookclerk_player_api::oauth

Sign-in for apps: player passwords and the browser sign-in.

**Player passwords.** A patron makes one per app in Account settings;
the host generates it ([`format_player_password`](#format_player_password)) and keeps only its
digest. It is 20 symbols from a 32-symbol alphabet with no look-alikes
(100 bits), shown in four groups of five. An app sends it as typed;
[`normalize_player_password`](#normalize_player_password) drops spaces and dashes and ignores case
before the digest is looked up, and anything else is refused without a
lookup.

**Browser sign-in.** Apps start with
`GET /player/auth/openid?code_challenge=..&code_challenge_method=S256&redirect_uri=<app>&client_id=..&response_type=code[&state=..]`
([`parse_openid_start`](#parse_openid_start)) and open the page the answer points to in a
browser: the library's own sign-in and consent page. Approving sends
the browser through `/player/auth/openid/mobile-redirect`
([`parse_mobile_redirect`](#parse_mobile_redirect)) back to the app's address with a code, and
the app trades the code and its PKCE verifier at
`/player/auth/openid/callback` ([`parse_openid_callback`](#parse_openid_callback),
[`pkce_s256_matches`](#pkce_s256_matches)) for a card. Only the apps in [`PLAYER_APPS`](#player_apps) can
be returned to.

| Item | Kind | Summary |
| --- | --- | --- |
| [`OpenIdStart`](#openidstart) | struct | `GET /player/auth/openid`: an app starting a browser sign-in. |
| [`MobileRedirect`](#mobileredirect) | struct | `GET /player/auth/openid/mobile-redirect`: the browser on its way back to the app with a code (or a refusal). |
| [`OpenIdCallback`](#openidcallback) | struct | `GET /player/auth/openid/callback`: the app trading its code. |
| [`OAuthError`](#oautherror) | enum | Why a sign-in request was refused. |
| [`format_player_password`](#format_player_password) | function | A player password for 20 random bytes: each byte picks one symbol of `PASSWORD_ALPHABET` (the alphabet divides 256, so every symbol is as likely), in four dash-separated groups of five. |
| [`normalize_player_password`](#normalize_player_password) | function | A typed player password in the one form its digest is taken of: the symbols alone, in capitals. |
| [`app_for_redirect`](#app_for_redirect) | function | The name of the app `redirect_uri` returns to, when it is one of `PLAYER_APPS` exactly. |
| [`app_of_return_address`](#app_of_return_address) | function | The app a consent page's `redirect_uri` returns to: its `app` parameter, when the address is a `MOBILE_REDIRECT_PATH` and the app is one of `PLAYER_APPS`. Only the app is taken from the address; the host always sends the browser to its own `MOBILE_REDIRECT_PATH`. |
| [`is_s256_challenge`](#is_s256_challenge) | function | True for a PKCE S256 challenge: 43 base64url symbols (a SHA-256 digest). |
| [`is_pkce_verifier`](#is_pkce_verifier) | function | True for a PKCE verifier: 43 to 128 unreserved characters (RFC 7636). |
| [`pkce_s256_matches`](#pkce_s256_matches) | function | True when `verifier` is a PKCE verifier whose S256 digest is `challenge`. |
| [`parse_openid_start`](#parse_openid_start) | function | Parses an app's start of a browser sign-in. |
| [`parse_mobile_redirect`](#parse_mobile_redirect) | function | Parses the browser's way back to the app. |
| [`parse_openid_callback`](#parse_openid_callback) | function | Parses an app's code exchange. |
| [`PLAYER_CLIENT_ID`](#player_client_id) | constant | The client id the host stores on a browser sign-in's code, so a code minted for an app is redeemed only by the player API. |
| [`PLAYER_CODE_TTL_SECS`](#player_code_ttl_secs) | constant | How long a browser sign-in's code lives, in seconds. |
| [`MOBILE_REDIRECT_PATH`](#mobile_redirect_path) | constant | The path of the browser's way back to the app (`parse_mobile_redirect`); the sign-in page sends every approval there, whatever origin a request names. |
| [`CONSENT_PAGE_PATH`](#consent_page_path) | constant | The page a browser sign-in opens: the library's own sign-in and consent page in the web app. |
| [`PASSWORD_ALPHABET`](#password_alphabet) | constant | The alphabet of player passwords: digits and capital letters without `0`, `1`, `I` and `O`. |
| [`PASSWORD_SYMBOLS`](#password_symbols) | constant | Symbols in a player password. |
| [`MAX_OAUTH_QUERY_BYTES`](#max_oauth_query_bytes) | constant | The longest query a sign-in route reads, in bytes. |
| [`MAX_STATE_BYTES`](#max_state_bytes) | constant | The longest `state` an app may send, in bytes. |
| [`PLAYER_APPS`](#player_apps) | constant | The apps a browser sign-in may return to: their redirect address and the name the consent page shows. |

## Structs

### OpenIdStart

```rust
pub struct OpenIdStart {
    pub client_id: String,
    pub redirect_uri: String,
    pub app_name: &'static str,
    pub state: Option<String>,
    pub code_challenge: String,
}
```

`GET /player/auth/openid`: an app starting a browser sign-in.

**Fields**

- `client_id`: The app's own client name, echoed back (`client_id`).
- `redirect_uri`: The app's address to return to: one of [`PLAYER_APPS`](#player_apps).
- `app_name`: The app's name, from [`PLAYER_APPS`](#player_apps).
- `state`: The app's `state`, when it sent one.
- `code_challenge`: The PKCE S256 challenge.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### MobileRedirect

```rust
pub struct MobileRedirect {
    pub app_redirect: String,
    pub code: Option<String>,
    pub state: Option<String>,
}
```

`GET /player/auth/openid/mobile-redirect`: the browser on its way back
to the app with a code (or a refusal).

**Fields**

- `app_redirect`: The app's address, one of [`PLAYER_APPS`](#player_apps) (`app`).
- `code`: The code, or `None` when the patron declined.
- `state`: The `state` to hand back.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### OpenIdCallback

```rust
pub struct OpenIdCallback {
    pub code: String,
    pub code_verifier: String,
}
```

`GET /player/auth/openid/callback`: the app trading its code.

**Fields**

- `code`: The code (43 base64url symbols, the shape the host mints).
- `code_verifier`: The PKCE verifier.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### OAuthError

```rust
pub enum OAuthError {
    TooLong,
    Invalid(&'static str),
    UnknownApp,
}
```

Why a sign-in request was refused.

**Variants**

- `TooLong`: The query is longer than [`MAX_OAUTH_QUERY_BYTES`](#max_oauth_query_bytes).
- `Invalid`: A parameter is missing or malformed; names it.
- `UnknownApp`: The redirect address is not one of [`PLAYER_APPS`](#player_apps). Apps look for
  the words `Invalid redirect_uri` in the answer.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

## Functions

### format_player_password

```rust
pub fn format_player_password(random: &[u8; 20]) -> String
```

A player password for 20 random bytes: each byte picks one symbol of
[`PASSWORD_ALPHABET`](#password_alphabet) (the alphabet divides 256, so every symbol is as
likely), in four dash-separated groups of five.

### normalize_player_password

```rust
pub fn normalize_player_password(typed: &str) -> Option<String>
```

A typed player password in the one form its digest is taken of: the
symbols alone, in capitals.

#### Returns

`None` unless, once spaces and dashes are dropped, it is exactly
[`PASSWORD_SYMBOLS`](#password_symbols) symbols of [`PASSWORD_ALPHABET`](#password_alphabet) in either case.

### app_for_redirect

```rust
pub fn app_for_redirect(redirect_uri: &str) -> Option<&'static str>
```

The name of the app `redirect_uri` returns to, when it is one of
[`PLAYER_APPS`](#player_apps) exactly.

### app_of_return_address

```rust
pub fn app_of_return_address(redirect_uri: &str) -> Option<(&'static str, &'static str)>
```

The app a consent page's `redirect_uri` returns to: its `app`
parameter, when the address is a [`MOBILE_REDIRECT_PATH`](#mobile_redirect_path) and the app
is one of [`PLAYER_APPS`](#player_apps). Only the app is taken from the address; the
host always sends the browser to its own [`MOBILE_REDIRECT_PATH`](#mobile_redirect_path).

#### Returns

The app's redirect address and name.

### is_s256_challenge

```rust
pub fn is_s256_challenge(challenge: &str) -> bool
```

True for a PKCE S256 challenge: 43 base64url symbols (a SHA-256 digest).

### is_pkce_verifier

```rust
pub fn is_pkce_verifier(verifier: &str) -> bool
```

True for a PKCE verifier: 43 to 128 unreserved characters (RFC 7636).

### pkce_s256_matches

```rust
pub fn pkce_s256_matches(verifier: &str, challenge: &str) -> bool
```

True when `verifier` is a PKCE verifier whose S256 digest is
`challenge`.

### parse_openid_start

```rust
pub fn parse_openid_start(query: &str) -> Result<OpenIdStart, OAuthError>
```

Parses an app's start of a browser sign-in.

#### Errors

[`OAuthError::UnknownApp`](#oautherror) for a redirect address outside
[`PLAYER_APPS`](#player_apps); [`OAuthError::Invalid`](#oautherror) for a missing or malformed
parameter (`response_type` must be `code` and the challenge method
`S256`); [`OAuthError::TooLong`](#oautherror) past [`MAX_OAUTH_QUERY_BYTES`](#max_oauth_query_bytes).

### parse_mobile_redirect

```rust
pub fn parse_mobile_redirect(query: &str) -> Result<MobileRedirect, OAuthError>
```

Parses the browser's way back to the app.

#### Errors

[`OAuthError::UnknownApp`](#oautherror) for an `app` outside [`PLAYER_APPS`](#player_apps);
[`OAuthError::Invalid`](#oautherror) for a malformed code or state.

### parse_openid_callback

```rust
pub fn parse_openid_callback(query: &str) -> Result<OpenIdCallback, OAuthError>
```

Parses an app's code exchange.

#### Errors

[`OAuthError::Invalid`](#oautherror) for a missing or malformed code or verifier.

## Constants

### PLAYER_CLIENT_ID

```rust
pub const PLAYER_CLIENT_ID: &str = "bookclerk-player";
```

The client id the host stores on a browser sign-in's code, so a code
minted for an app is redeemed only by the player API.

### PLAYER_CODE_TTL_SECS

```rust
pub const PLAYER_CODE_TTL_SECS: i64 = 300;
```

How long a browser sign-in's code lives, in seconds.

### MOBILE_REDIRECT_PATH

```rust
pub const MOBILE_REDIRECT_PATH: &str = "/player/auth/openid/mobile-redirect";
```

The path of the browser's way back to the app
([`parse_mobile_redirect`](#parse_mobile_redirect)); the sign-in page sends every approval
there, whatever origin a request names.

### CONSENT_PAGE_PATH

```rust
pub const CONSENT_PAGE_PATH: &str = "/apps/approve";
```

The page a browser sign-in opens: the library's own sign-in and
consent page in the web app.

### PASSWORD_ALPHABET

```rust
pub const PASSWORD_ALPHABET: &[u8; 32] = b"23456789ABCDEFGHJKLMNPQRSTUVWXYZ";
```

The alphabet of player passwords: digits and capital letters without
`0`, `1`, `I` and `O`.

### PASSWORD_SYMBOLS

```rust
pub const PASSWORD_SYMBOLS: usize = 20;
```

Symbols in a player password.

### MAX_OAUTH_QUERY_BYTES

```rust
pub const MAX_OAUTH_QUERY_BYTES: usize = _;
```

The longest query a sign-in route reads, in bytes.

### MAX_STATE_BYTES

```rust
pub const MAX_STATE_BYTES: usize = 512;
```

The longest `state` an app may send, in bytes.

### PLAYER_APPS

```rust
pub const PLAYER_APPS: &[(&str, &str)] = _;
```

The apps a browser sign-in may return to: their redirect address and
the name the consent page shows.
