# bookclerk_plugin_integration_audiobookshelf::watch

Which Audiobookshelf users the host has already been told about.

The host asks for new users (`poll_events`) on its own schedule, and the
guest may sleep between asks. So the set of users already reported is
kept in the plugin's [`LISTENING_STATE_BINDING`] database, as a hash of
each ABS user id, and a guest started later compares the server's user
list against it. Without the database the set lives in memory and is
seeded again whenever the guest starts, so a user created while the
guest slept is not reported.

[`LISTENING_STATE_BINDING`]: README.md#listening_state_binding

## Functions

### migration

```rust
pub fn migration() -> PluginMigration
```

The `database_migrations` entry that adds the user-watch table.

`abs_watch_users` holds one row per ABS user already reported or seeded,
plus a marker row once the first seed completed; its `person` column is
a hash of the ABS user id, never the id.
