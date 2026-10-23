**bookclerk_plugin_catalog > adapters**

# Module: adapters

## Contents

**Traits**

- [`RegistryAdapter`](#registryadapter) - Registry adapter trait (search / hydrate / versions).

---

## bookclerk_plugin_catalog::adapters::RegistryAdapter

*Trait*

Registry adapter trait (search / hydrate / versions).

**Methods:**

- `source_kind`: Returns the adapter source kind string (`cargo`, `npm`, `pypi`, `static`).
- `search`: Searches this registry for packages matching `query`.
- `fetch_manifest`: Fetches and parses the Bookclerk package manifest for `coordinate`.
- `list_versions`: Lists known versions for `name` in this registry.



