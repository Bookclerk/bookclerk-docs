# bookclerk_media

Clear-media packaging: remux, metadata fix-up, and MP3 encode.

# Audience

Host acquire / media-worker code. Store plugins may call helpers that are
re-exported here, but DRM (Adrm / CENC) stays in those plugins. The
ISO-BMFF reading and writing underneath is [`bookclerk_mp4`](../bookclerk_mp4/bookclerk_mp4.md), which both this
crate and those plugins share; the keys and ciphers stay with the plugin.

Style: `docs/code-documentation.md`. Product narrative: `docs/media.md`.

# Where the work runs

The async entry points in this module do not decode anything themselves.
They build a [`MediaJob`](job.md#bookclerk_mediajobmediajob) and hand it to a [`MediaPool`](pool.md#bookclerk_mediapoolmediapool), which runs it in a
short-lived child process confined to the paths that job declared. Hosts
install the pool once at startup with [`init_pool`](pool.md#bookclerk_mediapoolinit_pool); anything that skips
that gets a default pool, which still isolates as long as the worker binary
is installed alongside the host.

The synchronous functions ([`remux_trimmed`](native.md#bookclerk_medianativeremux_trimmed), [`align_chapter_starts`](chapter_align.md#bookclerk_mediachapter_alignalign_chapter_starts),
[`package_m4b_from_pcm`](package_m4b.md#bookclerk_mediapackage_m4bpackage_m4b_from_pcm), and the MP4 parsing helpers) run in-process by
design. They are cheap, take already-parsed input, or are the implementation
the worker itself calls.

## Modules

### [`bookclerk_media`](bookclerk_media.md)

*1 struct, 3 functions*

### [`brand`](brand.md)

*1 struct, 4 functions*

### [`chapter_align`](chapter_align.md)

*1 struct, 2 functions*

### [`error`](error.md)

*1 enum, 1 type alias*

### [`job`](job.md)

*3 enums*

### [`metadata`](metadata.md)

*1 constant, 1 struct, 2 functions*

### [`mp3`](mp3.md)

*1 function*

### [`native`](native.md)

*1 function*

### [`package_m4b`](package_m4b.md)

*1 struct, 2 functions*

### [`pool`](pool.md)

*1 enum, 2 structs, 4 functions, 5 constants*

