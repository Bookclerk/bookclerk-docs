**bookclerk > commands**

# Module: commands

## Contents

**Modules**

- [`clusters`](#clusters) - `bookclerk clusters`: this install as a host node of owner clusters,
- [`config_cmd`](#config_cmd) - `bookclerk config` — get/set settings and naming helpers.
- [`copydb`](#copydb) - `bookclerk export postgres` (hidden alias: `copydb`) — SQLite → PostgreSQL.
- [`daemon_cmd`](#daemon_cmd) - `bookclerk daemon` — thin HTTP client for bookclerkd.
- [`db`](#db) - `bookclerk db` — schema state, backups, and explicit migrate.
- [`diagnostics_cmd`](#diagnostics_cmd) - `bookclerk diagnostics` — show ring buffer / request upload.
- [`discover`](#discover) - Discovery: recommendations, embeddings, listening sync, wishlist.
- [`events`](#events) - `bookclerk events` — operator client for the durable domain-event outbox.
- [`export`](#export) - Library export (classic `export --csv|--json|--xlsx`).
- [`export_cmd`](#export_cmd) - `bookclerk export` — native backup, Libation, library spreadsheet, Postgres.
- [`import_cmd`](#import_cmd) - `bookclerk import` — native backup and Libation Files import.
- [`integrations`](#integrations) - `bookclerk integrations` — status, test, tickets, library scan.
- [`library`](#library) - `bookclerk library` — scan, acquire, accounts, set-status, get-license, search, export.
- [`migrate`](#migrate) - `bookclerk migrate` — hidden alias of `import libation`.
- [`nodes`](#nodes) - `bookclerk nodes`: the cluster's nodes, their placement labels, their
- [`plugin_installs`](#plugin_installs) - `bookclerk plugins install|update|rollback|remove|approve|versions` for
- [`plugins`](#plugins) - `bookclerk plugins` — discover, manage, and invoke dynamic plugin CLI.
- [`queues`](#queues) - `bookclerk queues` — operator client for plugin-owned queues on the outbox.

---

## Module: clusters

`bookclerk clusters`: this install as a host node of owner clusters,
and this host operator's consent to the plugins they ask to run here
(H-4 design, section 5).

Reads and writes the library database directly, like `nodes`. A
running daemon re-reads consent within a few seconds, so an approval
lets the plugin's jobs be claimed and a revoke cancels and gives back
the plugin's jobs held here.

`scan` is the exception: it asks this install's running daemon to send
a `scan.request` to the owner over its link (H-5), and polls the jobs
it queued.

`join --url <owner>` joins by signing in to the owner library in a
browser (an OAuth code with PKCE, sent back to a loopback listener this
command opens). `--as clerk` offers to join as a clerk instead of a
branch (an owner node sharing the library's database; only an empty
install, and only when the library's Owner chooses it). Without `--url`, `join` reads an enrollment link (a
branch invite) from stdin or a prompt, never from argv: the link carries
a single-use secret. A running daemon dials a newly joined cluster
within a few seconds, and stops a left one's client as quickly. `leave`
tells the owner first (`node.leaving`), which revokes the node there.



## Module: config_cmd

`bookclerk config` — get/set settings and naming helpers.



## Module: copydb

`bookclerk export postgres` (hidden alias: `copydb`) — SQLite → PostgreSQL.

Default `--format flat` writes the native bookclerk schema (`accounts` / `books`).
`--format classic` writes the Libation EF schema (`Books`, `LibraryBooks`, …).



## Module: daemon_cmd

`bookclerk daemon` — thin HTTP client for bookclerkd.



## Module: db

`bookclerk db` — schema state, backups, and explicit migrate.



## Module: diagnostics_cmd

`bookclerk diagnostics` — show ring buffer / request upload.



## Module: discover

Discovery: recommendations, embeddings, listening sync, wishlist.



## Module: events

`bookclerk events` — operator client for the durable domain-event outbox.



## Module: export

Library export (classic `export --csv|--json|--xlsx`).



## Module: export_cmd

`bookclerk export` — native backup, Libation, library spreadsheet, Postgres.



## Module: import_cmd

`bookclerk import` — native backup and Libation Files import.



## Module: integrations

`bookclerk integrations` — status, test, tickets, library scan.

Host commands are integration-agnostic: they go through
[`bookclerk_integrations::IntegrationRegistry`](../bookclerk_integrations/registry.md#bookclerk_integrationsregistryintegrationregistry) capabilities. Adapter-specific
clients (e.g. AbsApiClient) stay inside integration plugin packages.



## Module: library

`bookclerk library` — scan, acquire, accounts, set-status, get-license, search, export.



## Module: migrate

`bookclerk migrate` — hidden alias of `import libation`.



## Module: nodes

`bookclerk nodes`: the cluster's nodes, their placement labels, their
owners, and host-node enrollment links.

Reads and writes the library database directly, like `plugins`. Label
changes wake the cluster scheduler through every node's queue.

An enrollment link's secret is printed once, inside the link's URL
fragment, and never logged or written anywhere else.



## Module: plugin_installs

`bookclerk plugins install|update|rollback|remove|approve|versions` for
cluster versions.

Installing installs a version: the package is resolved and verified once
(publisher trust, archive and payload digests, `plugin.toml`), recorded
in the library with the operator's consent, and on a first install or
`--activate` made the active version. Nothing under this host's
`plugins/` changes; every host placed on the plugin materializes the
version by digest. The CLI only writes records; a running daemon applies
them when its host queue wakes it.



## Module: plugins

`bookclerk plugins` — discover, manage, and invoke dynamic plugin CLI.



## Module: queues

`bookclerk queues` — operator client for plugin-owned queues on the outbox.



