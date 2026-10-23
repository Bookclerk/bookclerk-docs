**bookclerk_library > secrets > secret_kind**

# Module: secrets::secret_kind

## Contents

**Constants**

- [`CLUSTER_JOIN`](#cluster_join) - On a host joining a library: the PKCE verifier and node keys of one
- [`CLUSTER_LINK`](#cluster_link) - On a host node: the private node keys of one owner-cluster link
- [`OIDC_CLIENT`](#oidc_client) - OIDC/OAuth relying-party client secret (`name` = provider id).
- [`OPERATOR_TOKEN`](#operator_token) - Daemon HTTP operator API token (Bearer / GUI login).
- [`PLUGIN_INSTANCE`](#plugin_instance) - Plugin-instance secret referenced by a `plugin_instance` config document.
- [`S3`](#s3) - S3 / object-storage credentials.
- [`SOURCE_AUTH`](#source_auth) - Store / source OAuth credentials (Audible, Libro.fm, Chirp, GA).
- [`TOTP`](#totp) - User TOTP shared secret (`name` = `primary` or `pending`).
- [`WIDEVINE`](#widevine) - Widevine L3 CDM device blob (`kind = widevine`).

---

## bookclerk_library::secrets::secret_kind::CLUSTER_JOIN

*Constant*: `&str`

On a host joining a library: the PKCE verifier and node keys of one
sign-in in progress (`provider` = owner cluster id, `account_id` =
the pending join's state hash). Deleted when the join completes or
expires.



## bookclerk_library::secrets::secret_kind::CLUSTER_LINK

*Constant*: `&str`

On a host node: the private node keys of one owner-cluster link
(`provider` = owner cluster id, `account_id` = this node's id there).



## bookclerk_library::secrets::secret_kind::OIDC_CLIENT

*Constant*: `&str`

OIDC/OAuth relying-party client secret (`name` = provider id).



## bookclerk_library::secrets::secret_kind::OPERATOR_TOKEN

*Constant*: `&str`

Daemon HTTP operator API token (Bearer / GUI login).



## bookclerk_library::secrets::secret_kind::PLUGIN_INSTANCE

*Constant*: `&str`

Plugin-instance secret referenced by a `plugin_instance` config document.

Ciphertext stays in this table. The configuration document stores the
ref name only.



## bookclerk_library::secrets::secret_kind::S3

*Constant*: `&str`

S3 / object-storage credentials.



## bookclerk_library::secrets::secret_kind::SOURCE_AUTH

*Constant*: `&str`

Store / source OAuth credentials (Audible, Libro.fm, Chirp, GA).



## bookclerk_library::secrets::secret_kind::TOTP

*Constant*: `&str`

User TOTP shared secret (`name` = `primary` or `pending`).



## bookclerk_library::secrets::secret_kind::WIDEVINE

*Constant*: `&str`

Widevine L3 CDM device blob (`kind = widevine`).



