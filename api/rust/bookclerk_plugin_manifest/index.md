# bookclerk_plugin_manifest

`plugin.toml` schema — parse, validate, and canonical format.

This crate is the Rust projection of the install-time plugin descriptor.
It is shared by the host (`bookclerk-plugin-host`), the workerd launcher
(`bookclerk-workerd`), and the Rust guest SDK author tools (`check` /
`fmt`). Language SDKs (TypeScript / Python) validate against the committed
JSON Schema in [`PLUGIN_TOML_SCHEMA_JSON`](#plugin_toml_schema_json) (`schema/plugin-toml.json`),
which must stay in sync with the types here.

## Audience

- **Host / launcher authors** — parse manifests at discovery, build
  [`EgressPolicy`](egress.md#egresspolicy) for isolate injection, map network + OAuth bindings to
  jail policy via [`PluginManifest::jail_network_need`](types.md#pluginmanifestjail_network_need).
- **Plugin authors / SDK tools** — validate and pretty-print `plugin.toml`
  without pulling host dependencies.

## Modules

| Module | Responsibility |
| --- | --- |
| [`egress`](egress.md) | Hostname allowlist policy, IDNA normalization, Pyodide hosts |
| [`error`](error.md) | Parse / validate error type |
| [`fmt`](fmt.md) | Canonical TOML serialization for `fmt --check` |
| [`id`](id.md) | Strict plugin id grammar |
| [`logo`](logo.md) | Remote URL vs embedded path logo validation |
| [`types`](types.md) | `PluginManifest` and nested capability / workerd tables |
| [`workerd_contract`](workerd_contract.md) | Compatibility date, Python flag allowlist, module extensions |

Product narrative: `docs/plugins/README.md` (network consent, egress matching,
packaging). ABI contract: `bookclerk-plugin-abi`.

## Examples

```rust
use bookclerk_plugin_manifest::parse;

let manifest = parse(r#"
api_version = 3
id = "echo"
runtime = "native"
command = "./echo"
entrypoints = ["cli"]

[capabilities.network]
mode = "deny"
"#).expect("valid plugin.toml");
assert_eq!(manifest.id, "echo");
```

| Item | Kind | Summary |
| --- | --- | --- |
| [`parse`](#parse) | function | Parses and validates a `plugin.toml` document. |
| [`PLUGIN_TOML_SCHEMA_JSON`](#plugin_toml_schema_json) | constant | Embedded JSON Schema text for authoring tools (`check` in TS/Python SDKs). |

## Re-exports

- [`address::address_allowed`](address.md#address_allowed)
- [`address::canonical_ip`](address.md#canonical_ip)
- [`address::is_public_internet`](address.md#is_public_internet)
- [`address::is_restricted_hostname`](address.md#is_restricted_hostname)
- [`address::workerd_network_allow`](address.md#workerd_network_allow)
- [`address::CidrGrant`](address.md#cidrgrant)
- [`egress::consent_domains_for`](egress.md#consent_domains_for)
- [`egress::host_matches`](egress.md#host_matches)
- [`egress::manifest_needs_python`](egress.md#manifest_needs_python)
- [`egress::normalize_domain_pattern`](egress.md#normalize_domain_pattern)
- [`egress::normalize_hostname`](egress.md#normalize_hostname)
- [`egress::with_python_runtime_hosts`](egress.md#with_python_runtime_hosts)
- [`egress::EgressPolicy`](egress.md#egresspolicy)
- [`egress::TcpGrant`](egress.md#tcpgrant)
- [`egress::DEFAULT_MAX_REDIRECTS`](egress.md#default_max_redirects)
- [`egress::PYODIDE_EGRESS_HOSTS`](egress.md#pyodide_egress_hosts)
- [`error::Error`](error.md#error)
- [`error::Result`](error.md#result)
- [`fmt::format_manifest`](fmt.md#format_manifest)
- [`id::validate_plugin_id`](id.md#validate_plugin_id)
- [`logo::embedded_logo_api_path`](logo.md#embedded_logo_api_path)
- [`logo::logo_content_type`](logo.md#logo_content_type)
- [`logo::validate_logo`](logo.md#validate_logo)
- [`logo::LogoKind`](logo.md#logokind)
- [`logo::LOGO_EXTENSIONS`](logo.md#logo_extensions)
- [`logo::MAX_EMBEDDED_LOGO_BYTES`](logo.md#max_embedded_logo_bytes)
- [`workerd_contract::apply_author_compatibility_date`](workerd_contract.md#apply_author_compatibility_date)
- [`workerd_contract::module_load_key`](workerd_contract.md#module_load_key)
- [`workerd_contract::validate_author_compatibility_date`](workerd_contract.md#validate_author_compatibility_date)
- [`workerd_contract::validate_author_compatibility_flags`](workerd_contract.md#validate_author_compatibility_flags)
- [`workerd_contract::validate_module_declarations`](workerd_contract.md#validate_module_declarations)
- [`workerd_contract::workerd_module_is_embedded`](workerd_contract.md#workerd_module_is_embedded)
- [`workerd_contract::AppliedCompatibilityDate`](workerd_contract.md#appliedcompatibilitydate)
- [`workerd_contract::PYTHON_COMPATIBILITY_FLAGS`](workerd_contract.md#python_compatibility_flags)
- [`workerd_contract::WORKERD_PIN_COMPAT_DATE`](workerd_contract.md#workerd_pin_compat_date)
- [`bookclerk_plugin_abi::Entrypoint`](../bookclerk_plugin_abi/generated.md#entrypoint)
- [`bookclerk_plugin_abi::EventConsumerSpec`](../bookclerk_plugin_abi/generated.md#eventconsumerspec)
- [`bookclerk_plugin_abi::PluginCapabilities`](../bookclerk_plugin_abi/generated.md#plugincapabilities)
- [`types::ALL_ENTRYPOINTS`](types.md#all_entrypoints)
- [`types::entrypoint_family`](types.md#entrypoint_family)
- [`types::PluginFamily`](types.md#pluginfamily)
- [`types::PluginRuntimeKind`](types.md#pluginruntimekind)
- [`types::NetworkMode`](types.md#networkmode)
- [`types::TcpCapability`](types.md#tcpcapability)
- [`types::NetworkCapabilities`](types.md#networkcapabilities)
- [`types::MAX_DATABASE_BINDINGS`](types.md#max_database_bindings)
- [`types::MAX_DATABASE_BINDING_NAME_LEN`](types.md#max_database_binding_name_len)
- [`types::is_valid_database_binding_name`](types.md#is_valid_database_binding_name)
- [`types::BindingCapabilities`](types.md#bindingcapabilities)
- [`types::DatabaseBindingManifest`](types.md#databasebindingmanifest)
- [`types::NamedBinding`](types.md#namedbinding)
- [`types::KvNamespaceManifest`](types.md#kvnamespacemanifest)
- [`types::DEFAULT_KV_MAX_VALUE_BYTES`](types.md#default_kv_max_value_bytes)
- [`types::MAX_KV_VALUE_BYTES`](types.md#max_kv_value_bytes)
- [`types::DEFAULT_KV_MAX_KEYS`](types.md#default_kv_max_keys)
- [`types::MAX_KV_KEYS`](types.md#max_kv_keys)
- [`types::DEFAULT_SECRETS_BINDING`](types.md#default_secrets_binding)
- [`types::DEFAULT_WORK_FS_BINDING`](types.md#default_work_fs_binding)
- [`types::DEFAULT_OAUTH_BINDING`](types.md#default_oauth_binding)
- [`types::DEFAULT_KV_BINDING`](types.md#default_kv_binding)
- [`types::DEFAULT_EVENTS_BINDING`](types.md#default_events_binding)
- [`types::CONFIG_BINDING`](types.md#config_binding)
- [`types::TriggersManifest`](types.md#triggersmanifest)
- [`types::CapabilitiesManifest`](types.md#capabilitiesmanifest)
- [`types::EventConsumer`](types.md#eventconsumer)
- [`types::EventProducer`](types.md#eventproducer)
- [`types::EventsManifest`](types.md#eventsmanifest)
- [`types::HOST_EVENT_TYPES`](types.md#host_event_types)
- [`types::is_host_event_type`](types.md#is_host_event_type)
- [`types::MAX_QUEUE_NAME_LEN`](types.md#max_queue_name_len)
- [`types::RESERVED_QUEUE_PREFIXES`](types.md#reserved_queue_prefixes)
- [`types::DEFAULT_QUEUE_MAX_BATCH_SIZE`](types.md#default_queue_max_batch_size)
- [`types::MAX_QUEUE_BATCH_SIZE`](types.md#max_queue_batch_size)
- [`types::DEFAULT_QUEUE_MAX_BATCH_TIMEOUT_SECS`](types.md#default_queue_max_batch_timeout_secs)
- [`types::MAX_QUEUE_BATCH_TIMEOUT_SECS`](types.md#max_queue_batch_timeout_secs)
- [`types::DEFAULT_QUEUE_MAX_RETRIES`](types.md#default_queue_max_retries)
- [`types::MAX_QUEUE_RETRIES`](types.md#max_queue_retries)
- [`types::MAX_QUEUE_RETRY_DELAY_SECS`](types.md#max_queue_retry_delay_secs)
- [`types::is_valid_queue_name`](types.md#is_valid_queue_name)
- [`types::is_reserved_queue_name`](types.md#is_reserved_queue_name)
- [`types::QueuesManifest`](types.md#queuesmanifest)
- [`types::QueueProducer`](types.md#queueproducer)
- [`types::QueueConsumer`](types.md#queueconsumer)
- [`types::WorkerdRuntimeManifest`](types.md#workerdruntimemanifest)
- [`types::WorkerdLimits`](types.md#workerdlimits)
- [`types::EffectiveWorkerdLimits`](types.md#effectiveworkerdlimits)
- [`types::ModuleSpec`](types.md#modulespec)
- [`types::PluginManifest`](types.md#pluginmanifest)
- [`types::OidcManifest`](types.md#oidcmanifest)
- [`types::OidcClientToml`](types.md#oidcclienttoml)
- [`types::JailNetworkNeed`](types.md#jailnetworkneed)

## Modules

- [`address`](address.md): Address-space policy independent of hostname matching.
- [`egress`](egress.md): Shared host/egress allowlist policy (workerd JS bridge + native mediator).
- [`error`](error.md): Manifest parse and semantic-validation errors.
- [`fmt`](fmt.md): Canonical `plugin.toml` formatting for SDK `fmt` / `fmt --check`.
- [`id`](id.md): Display-alias grammar shared by manifest parse, host registry, and SDKs.
- [`logo`](logo.md): Plugin logo validation: remote `http(s)` URL or relative image path.
- [`types`](types.md): `plugin.toml` types: manifest root and nested capability / workerd tables.
- [`workerd_contract`](workerd_contract.md): Author compatibility date, flag allowlist, and module-extension rules.

## Functions

### parse

```rust
pub fn parse(text: &str) -> Result<PluginManifest>
```

Parses and validates a `plugin.toml` document.

Equivalent to [`PluginManifest::parse`]: deserializes TOML then runs
[`PluginManifest::validate`](types.md#pluginmanifestvalidate).

#### Arguments

* `text` - Full contents of a `plugin.toml` file (UTF-8).

#### Returns

A validated [`PluginManifest`](types.md#pluginmanifest).

#### Errors

Returns [`Error::TomlDe`](error.md#error) on malformed TOML, or [`Error::Message`](error.md#error) when
semantic validation fails (id grammar, runtime requirements, network
domains, logo, and so on).

#### Examples

```rust
use bookclerk_plugin_manifest::parse;

let m = parse(r#"
api_version = 3
id = "sqlite"
runtime = "native"
command = "./bookclerk-plugin-database-sqlite"
entrypoints = ["databaseAdapter"]

[capabilities.network]
mode = "deny"
"#).unwrap();
assert_eq!(m.primary_family().as_str(), "database");
```

## Constants

### PLUGIN_TOML_SCHEMA_JSON

```rust
pub const PLUGIN_TOML_SCHEMA_JSON: &str = "{\n  \"$schema\": \"https://json-schema.org/draft/2020-12/schema\",\n  \"$id\": \"https://bookclerk.dev/schemas/plugin-toml.json\",\n  \"title\": \"BookclerkPluginToml\",\n  \"description\": \"Install descriptor for a Bookclerk plugin (plugin.toml), api_version = 3. Keys use TOML/snake_case. There is no plugin `kind`; handler family is derived from entrypoints and triggers. The `id` is a display/CLI alias, not a globally unique security identity. Durable identity is the host-computed provenance-qualified PluginKey. capabilities.network.domains is workerd-only (hostname allowlist for isolate fetch()). Native guests omit domains and declare capabilities.network.tcp for mediated connect(); ambient AF_INET is denied.\",\n  \"type\": \"object\",\n  \"additionalProperties\": false,\n  \"required\": [\"api_version\", \"id\", \"capabilities\"],\n  \"properties\": {\n    \"api_version\": { \"type\": \"integer\", \"enum\": [3] },\n    \"id\": {\n      \"type\": \"string\",\n      \"minLength\": 2,\n      \"maxLength\": 32,\n      \"pattern\": \"^[a-z0-9]+(?:_[a-z0-9]+)*$\",\n      \"description\": \"Display / CLI alias. Not globally unique and not a privilege. Lowercase letters, digits, and single underscores only (2\u{2013}32 chars); no leading/trailing or doubled `_`. When two installs share an alias, address them with a provenance-qualified PluginKey.\"\n    },\n    \"name\": { \"type\": \"string\" },\n    \"version\": { \"type\": \"string\" },\n    \"logo\": {\n      \"type\": \"string\",\n      \"description\": \"Optional Settings/UI logo: https:// or http:// URL (browser loads directly), or a relative image path under the plugin install root (host serves via GET /api/plugins/{id}/logo when the alias is unambiguous). Embedded paths must not contain \'..\' and must end with .png/.jpg/.jpeg/.gif/.webp/.svg/.ico.\"\n    },\n    \"runtime\": {\n      \"type\": \"string\",\n      \"enum\": [\"native\", \"workerd\"],\n      \"default\": \"native\"\n    },\n    \"command\": { \"type\": \"string\", \"minLength\": 1 },\n    \"args\": {\n      \"type\": \"array\",\n      \"items\": { \"type\": \"string\" }\n    },\n    \"entrypoints\": {\n      \"type\": \"array\",\n      \"description\": \"Named capabilities this plugin exports from PluginWorker.open.\",\n      \"items\": {\n        \"type\": \"string\",\n        \"enum\": [\"storefront\", \"storage\", \"databaseAdapter\", \"remoteLibrary\", \"cli\", \"oidc\"]\n      }\n    },\n    \"workerd\": { \"$ref\": \"#/$defs/WorkerdRuntime\" },\n    \"modules\": {\n      \"type\": \"array\",\n      \"items\": { \"$ref\": \"#/$defs/ModuleSpec\" }\n    },\n    \"triggers\": { \"$ref\": \"#/$defs/Triggers\" },\n    \"events\": { \"$ref\": \"#/$defs/Events\" },\n    \"databases\": {\n      \"type\": \"array\",\n      \"description\": \"Named plugin-owned database bindings (Workers-style).\",\n      \"maxItems\": 8,\n      \"items\": { \"$ref\": \"#/$defs/DatabaseBinding\" }\n    },\n    \"vars\": {\n      \"type\": \"object\",\n      \"description\": \"Operator config binding (`CONFIG`). Declares that the guest receives CONFIG; values come from [[settings]] defaults and the plugin instance document. An empty table is the usual form.\"\n    },\n    \"settings\": {\n      \"type\": \"array\",\n      \"description\": \"Operator settings schema ([[settings]]): keys an instance document may set, their types, enum values, and defaults. The host validates documents against it and renders it in Settings. Non-secret entries require [vars]; type = \\\"secret\\\" entries require [secrets].\",\n      \"maxItems\": 32,\n      \"items\": { \"$ref\": \"#/$defs/SettingSpec\" }\n    },\n    \"secrets\": { \"$ref\": \"#/$defs/NamedBinding\" },\n    \"kv_namespaces\": {\n      \"type\": \"array\",\n      \"description\": \"Durable KV bindings (default name KV) served from the library database: Workers KVNamespace-shaped get/put/delete/list with expiration and metadata, scoped to the plugin and the row\'s namespace. Distinct from [[databases]].\",\n      \"items\": { \"$ref\": \"#/$defs/KvNamespace\" }\n    },\n    \"queues\": { \"$ref\": \"#/$defs/Queues\" },\n    \"work_fs\": { \"$ref\": \"#/$defs/NamedBinding\" },\n    \"oauth\": { \"$ref\": \"#/$defs/NamedBinding\" },\n    \"capabilities\": { \"$ref\": \"#/$defs/Capabilities\" },\n    \"cli\": { \"$ref\": \"#/$defs/CliSchema\" },\n    \"oidc\": { \"$ref\": \"#/$defs/OidcManifest\" }\n  },\n  \"allOf\": [\n    {\n      \"if\": {\n        \"properties\": { \"runtime\": { \"const\": \"workerd\" } },\n        \"required\": [\"runtime\"]\n      },\n      \"then\": { \"required\": [\"workerd\"] }\n    },\n    {\n      \"if\": {\n        \"anyOf\": [\n          { \"properties\": { \"runtime\": { \"const\": \"native\" } } },\n          { \"not\": { \"required\": [\"runtime\"] } }\n        ]\n      },\n      \"then\": {\n        \"required\": [\"command\"],\n        \"properties\": {\n          \"capabilities\": {\n            \"properties\": {\n              \"network\": {\n                \"properties\": {\n                  \"domains\": false\n                }\n              }\n            }\n          }\n        }\n      }\n    },\n    {\n      \"if\": {\n        \"properties\": {\n          \"runtime\": { \"const\": \"workerd\" },\n          \"capabilities\": {\n            \"properties\": {\n              \"network\": {\n                \"properties\": { \"mode\": { \"const\": \"outbound\" } },\n                \"required\": [\"mode\"]\n              }\n            },\n            \"required\": [\"network\"]\n          }\n        },\n        \"required\": [\"runtime\", \"capabilities\"]\n      },\n      \"then\": {\n        \"properties\": {\n          \"capabilities\": {\n            \"properties\": {\n              \"network\": {\n                \"required\": [\"domains\"],\n                \"properties\": {\n                  \"domains\": {\n                    \"type\": \"array\",\n                    \"minItems\": 1,\n                    \"items\": { \"type\": \"string\", \"minLength\": 1 }\n                  }\n                }\n              }\n            }\n          }\n        }\n      }\n    }\n  ],\n  \"$defs\": {\n    \"Capabilities\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"network\"],\n      \"properties\": {\n        \"network\": { \"$ref\": \"#/$defs/NetworkCapabilities\" }\n      }\n    },\n    \"NetworkCapabilities\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"mode\"],\n      \"properties\": {\n        \"mode\": { \"type\": \"string\", \"enum\": [\"deny\", \"outbound\"] },\n        \"domains\": {\n          \"type\": \"array\",\n          \"description\": \"Workerd-only: initial-request host allowlist for isolate egress. Forbidden on runtime=native. The operator may add destinations beyond this list; the guest cannot.\",\n          \"items\": { \"type\": \"string\", \"minLength\": 1 }\n        },\n        \"tcp\": {\n          \"type\": \"array\",\n          \"description\": \"Raw TCP connect grants (host + ports). Fetch domain grants do not imply TCP.\",\n          \"items\": { \"$ref\": \"#/$defs/TcpCapability\" }\n        },\n        \"allow_undeclared_public_redirects\": {\n          \"type\": \"boolean\",\n          \"description\": \"When true, fetch redirects may target undeclared public hosts. Address-space policy still applies.\"\n        },\n        \"address_cidrs\": {\n          \"type\": \"array\",\n          \"description\": \"Explicit CIDR grants beyond the public Internet. No blanket private=true.\",\n          \"items\": { \"type\": \"string\", \"minLength\": 1 }\n        }\n      }\n    },\n    \"TcpCapability\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"host\", \"ports\"],\n      \"properties\": {\n        \"host\": { \"type\": \"string\", \"minLength\": 1 },\n        \"ports\": {\n          \"type\": \"array\",\n          \"items\": { \"type\": \"integer\", \"minimum\": 1, \"maximum\": 65535 },\n          \"minItems\": 1\n        }\n      }\n    },\n    \"Triggers\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"properties\": {\n        \"jobs\": {\n          \"type\": \"array\",\n          \"items\": { \"type\": \"string\", \"minLength\": 1 }\n        }\n      }\n    },\n    \"Events\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"properties\": {\n        \"consumers\": {\n          \"type\": \"array\",\n          \"items\": { \"$ref\": \"#/$defs/EventConsumer\" }\n        },\n        \"producers\": {\n          \"type\": \"array\",\n          \"items\": { \"$ref\": \"#/$defs/EventProducer\" }\n        }\n      }\n    },\n    \"EventConsumer\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"type\"],\n      \"properties\": {\n        \"type\": {\n          \"type\": \"string\",\n          \"minLength\": 1,\n          \"pattern\": \"^[a-z][a-z0-9_]*$\"\n        },\n        \"schema_versions\": {\n          \"type\": \"array\",\n          \"items\": { \"type\": \"integer\", \"minimum\": 1 },\n          \"minItems\": 1\n        },\n        \"supports_suspend\": { \"type\": \"boolean\", \"default\": false },\n        \"resource_class\": { \"type\": \"string\", \"enum\": [\"network\"], \"default\": \"network\" },\n        \"filter\": { \"type\": \"object\" },\n        \"max_retries\": { \"type\": \"integer\", \"minimum\": 0 }\n      }\n    },\n    \"EventProducer\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"type\"],\n      \"properties\": {\n        \"type\": {\n          \"type\": \"string\",\n          \"minLength\": 1,\n          \"pattern\": \"^[a-z][a-z0-9_]*$\"\n        },\n        \"binding\": { \"type\": \"string\", \"minLength\": 1 }\n      }\n    },\n    \"Queues\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"description\": \"Workers Queues: producers are `env` send bindings, consumers are `queue(batch)` triggers. Queue names are `[a-z0-9][a-z0-9._-]*` (at most 63 chars); the `core.` and `host.` prefixes are reserved for the host.\",\n      \"properties\": {\n        \"producers\": { \"type\": \"array\", \"items\": { \"$ref\": \"#/$defs/QueueProducer\" } },\n        \"consumers\": { \"type\": \"array\", \"items\": { \"$ref\": \"#/$defs/QueueConsumer\" } }\n      }\n    },\n    \"QueueProducer\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"queue\", \"binding\"],\n      \"properties\": {\n        \"queue\": {\n          \"type\": \"string\",\n          \"minLength\": 1,\n          \"maxLength\": 63,\n          \"pattern\": \"^[a-z0-9][a-z0-9._-]*$\"\n        },\n        \"binding\": { \"type\": \"string\", \"minLength\": 1, \"pattern\": \"^[A-Z][A-Z0-9_]*$\" }\n      }\n    },\n    \"QueueConsumer\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"queue\"],\n      \"properties\": {\n        \"queue\": {\n          \"type\": \"string\",\n          \"minLength\": 1,\n          \"maxLength\": 63,\n          \"pattern\": \"^[a-z0-9][a-z0-9._-]*$\"\n        },\n        \"max_batch_size\": { \"type\": \"integer\", \"minimum\": 1, \"maximum\": 100, \"default\": 10 },\n        \"max_batch_timeout\": { \"type\": \"integer\", \"minimum\": 0, \"maximum\": 60, \"default\": 5 },\n        \"max_retries\": { \"type\": \"integer\", \"minimum\": 0, \"maximum\": 100, \"default\": 3 },\n        \"dead_letter_queue\": {\n          \"type\": \"string\",\n          \"minLength\": 1,\n          \"maxLength\": 63,\n          \"pattern\": \"^[a-z0-9][a-z0-9._-]*$\"\n        },\n        \"retry_delay\": { \"type\": \"integer\", \"minimum\": 0, \"maximum\": 43200, \"default\": 0 },\n        \"max_concurrency\": { \"type\": \"integer\", \"minimum\": 1 }\n      }\n    },\n    \"SettingSpec\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"key\", \"type\"],\n      \"properties\": {\n        \"key\": {\n          \"type\": \"string\",\n          \"pattern\": \"^[a-z][a-z0-9_]{0,63}$\",\n          \"description\": \"Setting key, unique within the plugin; the CONFIG / SECRETS property name.\"\n        },\n        \"label\": { \"type\": \"string\", \"description\": \"Operator-facing label; defaults to the key.\" },\n        \"type\": {\n          \"type\": \"string\",\n          \"enum\": [\"string\", \"bool\", \"integer\", \"enum\", \"secret\"],\n          \"description\": \"Value type. `secret` values are sealed by the host and delivered in SECRETS, never stored in the instance document.\"\n        },\n        \"values\": {\n          \"type\": \"array\",\n          \"maxItems\": 64,\n          \"description\": \"Allowed values for type = \\\"enum\\\"; forbidden otherwise.\",\n          \"items\": {\n            \"type\": \"object\",\n            \"additionalProperties\": false,\n            \"required\": [\"id\"],\n            \"properties\": {\n              \"id\": { \"type\": \"string\", \"minLength\": 1, \"maxLength\": 1024 },\n              \"label\": { \"type\": \"string\" }\n            }\n          }\n        },\n        \"default\": {\n          \"type\": [\"string\", \"boolean\", \"integer\"],\n          \"description\": \"Default applied when the instance document omits the key. Not allowed for secrets.\"\n        },\n        \"required\": { \"type\": \"boolean\", \"default\": false },\n        \"description\": { \"type\": \"string\" }\n      }\n    },\n    \"DatabaseBinding\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"binding\"],\n      \"properties\": {\n        \"binding\": {\n          \"type\": \"string\",\n          \"pattern\": \"^[A-Z][A-Z0-9_]*$\",\n          \"maxLength\": 32\n        }\n      }\n    },\n    \"NamedBinding\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"properties\": {\n        \"binding\": { \"type\": \"string\", \"minLength\": 1 }\n      }\n    },\n    \"KvNamespace\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"properties\": {\n        \"binding\": { \"type\": \"string\", \"minLength\": 1 },\n        \"namespace\": {\n          \"type\": \"string\",\n          \"minLength\": 1,\n          \"maxLength\": 63,\n          \"pattern\": \"^[a-z0-9][a-z0-9._-]*$\"\n        },\n        \"max_value_bytes\": { \"type\": \"integer\", \"minimum\": 1, \"maximum\": 1048576, \"default\": 65536 },\n        \"max_keys\": { \"type\": \"integer\", \"minimum\": 1, \"maximum\": 1000000, \"default\": 10000 }\n      }\n    },\n    \"WorkerdRuntime\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"compatibility_date\", \"main_module\"],\n      \"properties\": {\n        \"compatibility_date\": {\n          \"type\": \"string\",\n          \"minLength\": 10,\n          \"maxLength\": 10,\n          \"pattern\": \"^[0-9]{4}-[0-9]{2}-[0-9]{2}$\",\n          \"description\": \"Calendar YYYY-MM-DD. A date newer than the pinned workerd compatibility date 2026-10-24 warns and falls back to that pin when the isolate loads. The pattern checks shape only; checkers reject impossible days such as 2026-02-31.\"\n        },\n        \"compatibility_flags\": {\n          \"type\": \"array\",\n          \"description\": \"Exact allowlist: python_workers and disable_python_external_sdk. Both are required together when the guest is Python and forbidden otherwise. The host does not append missing flags.\",\n          \"items\": {\n            \"type\": \"string\",\n            \"enum\": [\"python_workers\", \"disable_python_external_sdk\"]\n          }\n        },\n        \"main_module\": { \"type\": \"string\", \"minLength\": 1 },\n        \"modules_dir\": { \"type\": \"string\", \"default\": \"modules\" },\n        \"entrypoint\": { \"type\": \"string\", \"default\": \"default\" },\n        \"limits\": { \"$ref\": \"#/$defs/WorkerdLimits\" }\n      }\n    },\n    \"WorkerdLimits\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"properties\": {\n        \"cpu_ms\": { \"type\": \"integer\", \"minimum\": 0 },\n        \"subrequests\": { \"type\": \"integer\", \"minimum\": 0 }\n      }\n    },\n    \"ModuleSpec\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"name\"],\n      \"properties\": {\n        \"name\": { \"type\": \"string\" },\n        \"path\": {\n          \"type\": \"string\",\n          \"description\": \"Source file relative to the plugin root. When path is omitted or empty, name is the modules-directory source.\"\n        },\n        \"type\": { \"type\": \"string\", \"default\": \"js\" }\n      }\n    },\n    \"CliSchema\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"properties\": {\n        \"commands\": {\n          \"type\": \"array\",\n          \"items\": { \"$ref\": \"#/$defs/CliCommandSpec\" }\n        }\n      }\n    },\n    \"CliCommandSpec\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"name\"],\n      \"properties\": {\n        \"name\": { \"type\": \"string\" },\n        \"about\": { \"type\": \"string\" },\n        \"args\": {\n          \"type\": \"array\",\n          \"items\": { \"$ref\": \"#/$defs/CliArgSpec\" }\n        }\n      }\n    },\n    \"CliArgSpec\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"name\"],\n      \"properties\": {\n        \"name\": { \"type\": \"string\" },\n        \"long\": { \"type\": \"string\" },\n        \"short\": { \"type\": \"string\", \"maxLength\": 1 },\n        \"kind\": {\n          \"type\": \"string\",\n          \"enum\": [\"string\", \"bool\", \"int\", \"path\"],\n          \"default\": \"string\"\n        },\n        \"required\": { \"type\": \"boolean\" },\n        \"default\": { \"type\": \"string\" },\n        \"about\": { \"type\": \"string\" },\n        \"positional\": { \"type\": \"boolean\" }\n      }\n    },\n    \"OidcManifest\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"properties\": {\n        \"clients\": {\n          \"type\": \"array\",\n          \"items\": { \"$ref\": \"#/$defs/OidcClient\" }\n        }\n      }\n    },\n    \"OidcClient\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"client_id\", \"callback_path\", \"origin_config_key\"],\n      \"properties\": {\n        \"client_id\": { \"type\": \"string\", \"minLength\": 1 },\n        \"display_name\": { \"type\": \"string\" },\n        \"callback_path\": { \"type\": \"string\", \"pattern\": \"^/\" },\n        \"public_client\": { \"type\": \"boolean\", \"default\": true },\n        \"default_scopes\": {\n          \"type\": \"array\",\n          \"items\": { \"type\": \"string\" }\n        },\n        \"issue_refresh_token\": { \"type\": \"boolean\", \"default\": true },\n        \"origin_config_key\": { \"type\": \"string\", \"minLength\": 1 }\n      }\n    }\n  }\n}\n";
```

Embedded JSON Schema text for authoring tools (`check` in TS/Python SDKs).

Identical on disk to `schema/plugin-toml.json`. Prefer this constant over
reading the file when packaging tools as a single crate artifact.
