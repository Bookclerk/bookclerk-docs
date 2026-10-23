**bookclerk_plugin_catalog > owner_fetch**

# Module: owner_fetch

## Contents

**Functions**

- [`check_owner_artifact_url`](#check_owner_artifact_url) - Checks a URL an owner cluster asks this host to fetch an archive from.
- [`fetch_owner_artifact`](#fetch_owner_artifact) - Fetches an owner-chosen `https:` archive into `dest`, bounded by

---

## bookclerk_plugin_catalog::owner_fetch::check_owner_artifact_url

*Function*

Checks a URL an owner cluster asks this host to fetch an archive from.

This is the textual half of the policy; the fetch enforces the address
half again on what each name resolves to (`PublicOnlyResolver`).

# Arguments

* `url` - The owner's artifact URL.

# Returns

The parsed URL.

# Errors

Returns an error when the URL does not parse, is not `https:`, carries
credentials, has no host, or names `localhost` (or a `.localhost`
name, or another name that always resolves locally) or an IP literal
that is not a public Internet address.

```rust
fn check_owner_artifact_url(url: &str) -> crate::error::Result<url::Url>
```



## bookclerk_plugin_catalog::owner_fetch::fetch_owner_artifact

*Function*

Fetches an owner-chosen `https:` archive into `dest`, bounded by
[`MAX_DOWNLOAD_BYTES`](install.md#bookclerk_plugin_cataloginstallmax_download_bytes) and [`DOWNLOAD_TIMEOUT`](install.md#bookclerk_plugin_cataloginstalldownload_timeout) per request. Redirects
are followed (at most `MAX_REDIRECTS`) only when each hop passes
[`check_owner_artifact_url`](owner_fetch.md#bookclerk_plugin_catalogowner_fetchcheck_owner_artifact_url), and every hop connects only to addresses
that pass `owner_address_allowed` (`PublicOnlyResolver`); no proxy
is used. The caller checks the archive digest.

# Arguments

* `url` - The owner's artifact URL.
* `dest` - Where the body is written.

# Errors

Returns an error when a URL or a redirect hop is refused, a host
resolves to a refused address, there are too many redirects, or the
transfer fails or exceeds the cap.

```rust
fn fetch_owner_artifact(url: &str, dest: &std::path::Path) -> crate::error::Result<()>
```



