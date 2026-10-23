# bookclerk_plugin_source_graphicaudio

GraphicAudio content source: auth, library sync, and DRM-free download.

Access paths (see `docs/source-candidates.md`), selected by
`[sources.graphicaudio] access` (default `web`):
1. Browser Player (`/library`) — Magento session + CloudFront cookies
2. Magento ZIP (`My Downloadable Products`) — opt-in (`access = "zip"`)
3. Access App Retrofit API (`/access`) — opt-in device activation (`access = "device"`)

## Modules

### [`bookclerk_plugin_source_graphicaudio`](bookclerk_plugin_source_graphicaudio.md)

*2 modules*

### [`auth`](auth.md)

*1 struct*

### [`catalog`](catalog.md)

*1 struct, 9 functions*

### [`client`](client.md)

*3 structs, 4 constants*

### [`db`](db.md)

*4 functions*

### [`download`](download.md)

*1 struct, 2 constants, 2 functions*

### [`error`](error.md)

*1 enum, 1 type alias*

### [`guest`](guest.md)

*12 functions*

### [`magento`](magento.md)

*1 constant, 3 structs*

### [`options`](options.md)

*3 enums*

### [`source`](source.md)

*1 constant, 1 struct*

### [`sync`](sync.md)

*2 structs, 3 functions*

