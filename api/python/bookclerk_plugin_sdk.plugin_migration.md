# bookclerk\_plugin\_sdk.plugin\_migration

Resource bounds for plugin-owned ``databaseMigrations`` registration.

Semantic BookclerkSQL proof stays on the host. This module only rejects
pathological list / SQL sizes before JSON encoding.

#### require\_plugin\_migration\_registration

```python
def require_plugin_migration_registration(migrations: Any) -> list[Any]
```

Reject a registration that exceeds ABI resource limits.

**Arguments**:

- `migrations` - Ordered ``databaseMigrations`` list.
  

**Returns**:

  The same list when within limits, or ``[]`` when ``migrations`` is not a list.
  

**Raises**:

- `PluginError` - With ``code="payload_too_large"`` when a limit is exceeded.

