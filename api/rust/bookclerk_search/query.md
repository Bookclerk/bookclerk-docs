**bookclerk_search > query**

# Module: query

## Contents

**Functions**

- [`normalize_lucene_query`](#normalize_lucene_query) - Normalize a classic Libation/Lucene query for Tantivy's parser.

---

## bookclerk_search::query::normalize_lucene_query

*Function*

Normalize a classic Libation/Lucene query for Tantivy's parser.

Handles common shortcuts: `-acquired`, `[tag]`, field synonyms.

```rust
fn normalize_lucene_query(input: &str) -> String
```



