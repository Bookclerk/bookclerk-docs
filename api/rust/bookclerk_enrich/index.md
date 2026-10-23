# bookclerk_enrich

Source-agnostic Audible metadata enrichment.

Matches library rows that lack an Audible ASIN using public Audible catalog
search + Audnexus (AudioBookshelf-style confidence scoring). Intended for
Libro.fm, Chirp, GraphicAudio, and any future non-Audible sources.

A confident match stores the Audible ASIN and catalog fields on the row;
acquire then overlays Audible chapter trees (and related tags) onto plain
audio that lacks Audible brand intro/outro.

## Modules

### [`enrich`](enrich.md)

*1 constant, 2 structs, 7 functions*

### [`error`](error.md)

*1 enum, 1 type alias*

### [`match_score`](match_score.md)

*2 structs, 9 functions*

### [`public_meta`](public_meta.md)

*1 enum, 24 functions, 4 structs*

