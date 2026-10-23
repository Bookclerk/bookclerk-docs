**bookclerk_config > oidc_broker**

# Module: oidc_broker

## Contents

**Structs**

- [`OidcBrokerConfig`](#oidcbrokerconfig) - `[auth.oidc]` — optional identity broker.
- [`OidcProviderConfig`](#oidcproviderconfig) - One upstream OIDC or OAuth 2.0 provider.

**Enums**

- [`OidcProvisionMode`](#oidcprovisionmode) - How an upstream provider decides who may become a first-party User.

**Functions**

- [`allowlist_permits`](#allowlist_permits) - Whether an allowlist provider admits this `(email, sub)`.
- [`email_domain_allowed`](#email_domain_allowed) - Whether `email` matches an allowlist of domains (empty list = unrestricted).
- [`oidc_apple_private_key_env_key`](#oidc_apple_private_key_env_key) - Environment variable holding a Sign in with Apple `.p8` private key.
- [`oidc_client_secret_env_key`](#oidc_client_secret_env_key) - Environment variable holding a provider's client secret.
- [`oidc_secret_store_name`](#oidc_secret_store_name) - `encrypted_secrets.name` for an OIDC client secret or Apple key.
- [`resolve_mapped_role`](#resolve_mapped_role) - Pick the highest Bookclerk role from mapped claim values.

---

## bookclerk_config::oidc_broker::OidcBrokerConfig

*Struct*

`[auth.oidc]` — optional identity broker.

**Fields:**
- `enabled: bool` - When false, RP routes report disabled and login buttons are hidden.
- `allowed_email_domains: Vec<String>` - Global email-domain allowlist applied after the provider policy.
- `providers: Vec<OidcProviderConfig>` - Upstream providers (enterprise and/or social).
- `secret_generation: u64` - Durable `encrypted_secrets` generation for this broker revision.

**Methods:**

- `fn is_unset(self: &Self) -> bool` - Whether this section is the serde default (disabled, no providers).
- `fn enabled_providers(self: &Self) -> Vec<&OidcProviderConfig>` - Enabled providers with a non-empty id.
- `fn provider(self: &Self, id: &str) -> Option<&OidcProviderConfig>` - Look up a provider by id.
- `fn validate(self: &Self) -> Result<()>` - Reject operator maps, unknown roles, duplicate ids, and empty ids when enabled.
- `fn validate_providers(self: &Self) -> Result<()>` - Validate provider rows even when the broker is disabled.

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> OidcBrokerConfig`
- **Clone**
  - `fn clone(self: &Self) -> OidcBrokerConfig`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &OidcBrokerConfig) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_config::oidc_broker::OidcProviderConfig

*Struct*

One upstream OIDC or OAuth 2.0 provider.

**Fields:**
- `id: String` - Stable id used in URLs and `portal_identities.provider` (`oidc:{id}`).
- `name: String` - Login-button label.
- `preset: Option<String>` - Optional well-known preset (`google`, `github`, `apple`, `discord`).
- `issuer: Option<String>` - OIDC issuer URL (discovery). Ignored when `preset` fills endpoints.
- `client_id: String` - OAuth client id.
- `client_secret: Option<String>` - Optional client secret (prefer env / `encrypted_secrets`).
- `scopes: Vec<String>` - Scopes requested at authorize.
- `provision: OidcProvisionMode` - Provisioning policy for this provider.
- `default_role: String` - Role used for `OidcProvisionMode::Any` (and allowlist without maps).
- `role_claim: String` - Claim name holding groups/roles (typically `groups` or `roles`).
- `role_map: std::collections::BTreeMap<String, String>` - Upstream group/role value → Bookclerk role (`owner`/`administrator`/`member`).
- `link_by_email: bool` - When true, a new `sub` may attach to the unique matching *verified* email User.
- `apple_team_id: Option<String>` - Sign in with Apple team ID (`iss` of the client-secret JWT).
- `apple_key_id: Option<String>` - Sign in with Apple key ID (`kid` header of the client-secret JWT).
- `apple_private_key: Option<String>` - PEM-encoded Apple `.p8` private key (prefer env / `encrypted_secrets`).
- `allowed_email_domains: Vec<String>` - Email domains allowed for this provider (empty = no extra filter).
- `allowed_emails: Vec<String>` - Exact emails allowed when `provision = allowlist`.
- `allowed_subjects: Vec<String>` - Explicit upstream `sub` values allowed when `provision = allowlist`.

**Methods:**

- `fn portal_provider(self: &Self) -> String` - Portal identity provider key (`oidc:{id}`).
- `fn display_name(self: &Self) -> &str` - Display name, falling back to id.
- `fn social_preset(self: &Self) -> Option<&'static str>` - Canonical social preset (`google`, `github`, `apple`, `discord`).
- `fn effective_scopes(self: &Self) -> Vec<String>` - Scopes sent at authorize, with social-preset defaults when unset.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> OidcProviderConfig`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &OidcProviderConfig) -> bool`
- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_config::oidc_broker::OidcProvisionMode

*Enum*

How an upstream provider decides who may become a first-party User.

**Variants:**
- `MappedRole` - Require a mapped group/role claim (`owner` / `administrator` / `member`).
- `Any` - Any authenticated account; JIT as `OidcProviderConfig::default_role`.
- `Allowlist` - Authenticated and allowlisted (email domain and/or `sub`).
- `InviteOnly` - No JIT; SSO only links a pre-created User.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Default**
  - `fn default() -> OidcProvisionMode`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &OidcProvisionMode) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> OidcProvisionMode`



## bookclerk_config::oidc_broker::allowlist_permits

*Function*

Whether an allowlist provider admits this `(email, sub)`.

Empty email, domain, and subject lists fail closed. When the provider lists
are empty, `global_domains` is used as the domain allowlist.

```rust
fn allowlist_permits(email: Option<&str>, sub: &str, allowed_emails: &[String], allowed_domains: &[String], allowed_subjects: &[String], global_domains: &[String]) -> bool
```



## bookclerk_config::oidc_broker::email_domain_allowed

*Function*

Whether `email` matches an allowlist of domains (empty list = unrestricted).

```rust
fn email_domain_allowed(email: Option<&str>, domains: &[String]) -> bool
```



## bookclerk_config::oidc_broker::oidc_apple_private_key_env_key

*Function*

Environment variable holding a Sign in with Apple `.p8` private key.

```rust
fn oidc_apple_private_key_env_key(provider_id: &str) -> String
```



## bookclerk_config::oidc_broker::oidc_client_secret_env_key

*Function*

Environment variable holding a provider's client secret.

Hyphens in `provider_id` become underscores. Example: `github` →
`BOOKCLERK_OIDC_GITHUB_CLIENT_SECRET`.

```rust
fn oidc_client_secret_env_key(provider_id: &str) -> String
```



## bookclerk_config::oidc_broker::oidc_secret_store_name

*Function*

`encrypted_secrets.name` for an OIDC client secret or Apple key.

Generation `0` is the unversioned legacy name; later generations use
`name@generation` so a crashed PUT cannot make new credentials visible
under the still-published config.

```rust
fn oidc_secret_store_name(logical: &str, generation: u64) -> String
```



## bookclerk_config::oidc_broker::resolve_mapped_role

*Function*

Pick the highest Bookclerk role from mapped claim values.

Precedence: owner > administrator > host > member.

```rust
fn resolve_mapped_role(role_map: &std::collections::BTreeMap<String, String>, claim_values: &[String]) -> Option<String>
```



