# bookclerk_naming

Port of Libation's [NamingTemplate] tag template naming engine.

# Audience

Host acquire / settings code that expands folder and file templates. Guest
plugins do not depend on this crate.

Upstream lives in `FileManager.NamingTemplate` (class `NamingTemplate`); the
book/property tags and formatters come from `LibationFileManager.Templates`.
This crate targets classic / Chardonnay parity for property tags,
conditionals, and the text / number / list / name / series / date formatters.

Current Libation templates use `<tag->…<-tag>` conditionals. Older Libation
forms are still accepted for compatibility: `<if series>…<end if>`,
`%asin%`, and bare `<asin>` tags.

Style: `docs/code-documentation.md`.

[NamingTemplate]: https://github.com/rmcrackan/Libation/tree/master/Source/FileManager/NamingTemplate

# Example
```
use bookclerk_naming::{expand_template, BookContext, Contributor};

let book = BookContext {
    isbn: "B00X".into(),
    title_with_subtitle: Some("A Study in Scarlet".into()),
    authors: vec![Contributor::new("Arthur Conan Doyle", None)],
    ..Default::default()
};
let out = expand_template("<author> - <title> [<id>]", &book, None).unwrap();
assert_eq!(out, "Arthur Conan Doyle - A Study in Scarlet [B00X]");
```

## Modules

### [`bookclerk_naming`](bookclerk_naming.md)

*1 enum, 1 struct, 1 type alias, 6 functions*

### [`context`](context.md)

*1 enum, 4 structs*

### [`engine`](engine.md)

*1 struct*

