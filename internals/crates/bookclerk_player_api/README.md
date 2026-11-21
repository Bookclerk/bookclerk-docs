# bookclerk_player_api

The protocol half of the host's `/player` API: what audiobook player
apps written for an Audiobookshelf server send and expect, with no I/O
(`docs/adr/player-api.md`).

The host (`bookclerkd`) mounts the routes and answers them from its
playback core and library store; this crate holds everything about the
wire that does not need the daemon, so the fuzz targets and the unit
tests drive the same code the server does:

- [`request`]: sans-IO parsers for every body, header and query value
  an app sends (sign-in, refresh, play, sync, offline sessions,
  progress, bookmarks, list queries, filters), each bounded and typed.
- [`wire`]: the response shapes the apps read, with OpenAPI schemas.
- [`socket`]: the Engine.IO and Socket.IO framing of the
  websocket-only endpoint at `/player/socket.io`.
- [`oauth`]: player passwords and the browser sign-in's parameters.
- [`ids`]: the ids the API hands out for libraries, items and files.
- [`time`]: seconds on the wire, milliseconds in the playback core.
- [`tracks`]: how long each audio file of an item plays.

Nothing here decides who may see or play what: the host applies the
patron's scope and the playback grant before it builds an answer.

## Modules

- [`ids`](ids.md): The ids the `/player` API hands out and reads back.
- [`oauth`](oauth.md): Sign-in for apps: player passwords and the browser sign-in.
- [`request`](request.md): Sans-IO parsers for what player apps send the `/player` API.
- [`socket`](socket.md): Engine.IO (v4) and Socket.IO (v5) framing of `/player/socket.io`.
- [`time`](time.md): Lengths of time on the wire and in the playback core.
- [`tracks`](tracks.md): How long each audio file of an item plays, for the track list apps seek across.
- [`wire`](wire.md): The response shapes player apps read.

## Constants

### COMPATIBLE_SERVER_VERSION

```rust
pub const COMPATIBLE_SERVER_VERSION: &str = "2.38.0";
```

The Audiobookshelf server release whose API this one answers like. Apps
turn features on by version (the refresh-token flow needs 2.26.0 or
later), so `/status` and the sign-in answers state it.
