# bookclerk_mp4

ISO-BMFF (MP4) container plumbing: read a progressive file, pick samples by
media time, and write a faststart `ftyp` + `moov` + `mdat` M4B in one pass.

# No cryptography here

[`remux_progressive`](remux.md#bookclerk_mp4remuxremux_progressive) streams payloads through a [`SampleTransform`](remux.md#bookclerk_mp4remuxsampletransform), and the
only implementation this crate ships is [`CopySamples`](remux.md#bookclerk_mp4remuxcopysamples). A store that has to
turn its own ciphertext into plaintext keeps both the key and the cipher on
its side of that trait, inside its own plugin process.

That split is why two very similar remuxers used to exist: the host needed
one for clear media and the Audible plugin needed one that could decrypt
mid-copy, and neither could take on the other's dependencies. The transform
hook removes the copy without moving the boundary — see `docs/media.md` for
why the boundary is where it is.

## Modules

### [`bookclerk_mp4`](bookclerk_mp4.md)

*3 modules*

### [`boxutil`](boxutil.md)

*11 functions, 2 structs, 36 constants*

### [`edit`](edit.md)

*4 functions*

### [`error`](error.md)

*1 enum, 1 type alias*

### [`fixture`](fixture.md)

*1 struct*

### [`parser`](parser.md)

*1 enum, 3 functions, 3 structs*

### [`patch`](patch.md)

*1 constant, 1 struct, 6 functions*

### [`read`](read.md)

*1 struct*

### [`remux`](remux.md)

*1 trait, 2 functions, 4 structs*

### [`samples`](samples.md)

*2 functions, 2 structs*

