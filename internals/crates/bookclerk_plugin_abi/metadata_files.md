# bookclerk_plugin_abi::metadata_files

The checks the host runs on what a `metadataFiles` plugin returns and on
what it sends one.

A metadata-file dialect turns the bytes of a sidecar (Audiobookshelf
`metadata.json`, OPF, NFO, CUE, ...) into a [`MetadataFileRecord`](generated.md#metadatafilerecord) and
renders a record back into bytes. The plugin is jailed with no file or
network grants, but its replies are still untrusted: every record is
checked here before the host stores a claim from it. The checks are
sans-IO, so the `metadata_file_record` fuzz target and the host run the
same code.

Two layers:

- [`validate_metadata_file_record`](#validate_metadata_file_record) checks a record on its own: caps,
  text shape, date shape, chapter order, and that every value it carries
  is for a field it says the file states.
- [`check_record_against_dialect`](#check_record_against_dialect) checks it against the dialect the
  manifest declares: a record may not state a field or credit role its
  dialect does not carry.

| Item | Kind | Summary |
| --- | --- | --- |
| [`MetadataFileDialect`](#metadatafiledialect) | struct | One dialect as the plugin's manifest declares it: the fields its format carries and whether it can write files. The host builds this from the `[[metadata_files.dialects]]` row it routed a file to. |
| [`MetadataFileError`](#metadatafileerror) | enum | Why a metadata-file request or reply was refused. |
| [`is_dialect_id`](#is_dialect_id) | function | Whether `id` follows the ABI identifier rule `a-z{0,63}`. |
| [`is_plain_file_name`](#is_plain_file_name) | function | Whether `name` is a plain file name: non-empty, at most `MAX_METADATA_FILE_NAME_BYTES`, not `.` or `..`, and free of `/`, `\` and control characters. |
| [`is_partial_date`](#is_partial_date) | function | Whether `date` is `YYYY`, `YYYY-MM` or `YYYY-MM-DD` with a real month and a day the month can hold (29 February only in a leap year). |
| [`fields_with_values`](#fields_with_values) | function | The fields a record carries a value for, in `MetadataField` order. |
| [`validate_metadata_file_record`](#validate_metadata_file_record) | function | Checks a record on its own, without its dialect. |
| [`check_record_against_dialect`](#check_record_against_dialect) | function | Checks a record against the dialect that produced it: `present` names only fields the dialect carries and `creditRoles` only roles it carries. |
| [`check_parse_params`](#check_parse_params) | function | Checks a `MetadataFiles.parse` request before it is sent: a dialect id, a plain file name, and bytes within `MAX_METADATA_FILE_BYTES`. |
| [`check_render_params`](#check_render_params) | function | Checks a `MetadataFiles.render` request before it is sent: a dialect id, fields named once each, existing bytes within `MAX_METADATA_FILE_BYTES`, and a record that passes `validate_metadata_file_record`. |
| [`check_render_against_dialect`](#check_render_against_dialect) | function | Checks a render request against the dialect it names: the dialect writes, and every field to write is one it carries. |
| [`check_rendered`](#check_rendered) | function | Checks a rendered file: within `MAX_METADATA_FILE_BYTES`. An empty file is allowed: it means the dialect has nothing left to write, and the host decides whether to keep the empty file or remove it. |
| [`decode_metadata_file_record_bytes`](#decode_metadata_file_record_bytes) | function | Decodes one unpacked single-segment Cap'n Proto `MetadataFileRecord`, the form the fuzz target feeds. |
| [`encode_metadata_file_record_bytes`](#encode_metadata_file_record_bytes) | function | Encodes `record` as an unpacked single-segment Cap'n Proto message, the inverse of `decode_metadata_file_record_bytes`. |
| [`MAX_METADATA_FILE_NAME_BYTES`](#max_metadata_file_name_bytes) | constant | Longest file name `MetadataFiles.parse` accepts, in UTF-8 bytes (the usual file-system component limit). |
| [`MAX_METADATA_DIALECT_ID_BYTES`](#max_metadata_dialect_id_bytes) | constant | Longest dialect id, in bytes; dialect ids follow the ABI identifier rule. |

## Structs

### MetadataFileDialect

```rust
pub struct MetadataFileDialect {
    pub id: String,
    pub fields: Vec<MetadataField>,
    pub credit_roles: Vec<CreditRole>,
    pub writes: bool,
}
```

One dialect as the plugin's manifest declares it: the fields its format
carries and whether it can write files. The host builds this from the
`[[metadata_files.dialects]]` row it routed a file to.

**Fields**

- `id`: Dialect id (`[a-z][a-z0-9_]{0,63}`), unique within the plugin.
- `fields`: Fields the format carries; a record may state only these.
- `credit_roles`: Credit roles the format carries when `fields` holds
  [`MetadataField::Credits`](generated.md#metadatafield); empty otherwise.
- `writes`: Whether the dialect renders files (`MetadataFiles.render`), not only
  reads them.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### MetadataFileError

```rust
pub enum MetadataFileError {
    BadDialectId,
    BadFileName,
    BadFileSize(usize),
    DuplicateField(MetadataField),
    DuplicateCreditRole(CreditRole),
    ValueNotPresent(MetadataField),
    CreditRoleNotStated(CreditRole),
    TooMany(&'static str),
    BadText(&'static str),
    BadDate,
    BadChapters,
    FieldNotCarried(MetadataField),
    CreditRoleNotCarried(CreditRole),
    ReadOnlyDialect,
    WrongDialect,
}
```

Why a metadata-file request or reply was refused.

**Variants**

- `BadDialectId`: A dialect id is not `[a-z][a-z0-9_]{0,63}`.
- `BadFileName`: A file name is empty, too long, a dot name, or holds a path
  separator or a control character.
- `BadFileSize`: File bytes are over [`MAX_METADATA_FILE_BYTES`](README.md#max_metadata_file_bytes) (or a rendered file
  is empty).
- `DuplicateField`: A field appears twice in a field list (`present` or a render
  request's `fields`).
- `DuplicateCreditRole`: A credit role appears twice in `creditRoles`.
- `ValueNotPresent`: The record carries a value for a field missing from `present`.
- `CreditRoleNotStated`: A credit's role is not in `creditRoles`.
- `TooMany`: A list is longer than its cap.
- `BadText`: A text value is empty where it may not be, over its cap, or holds
  a NUL or another control character it may not.
- `BadDate`: `publishedAt` is not `YYYY`, `YYYY-MM` or `YYYY-MM-DD`.
- `BadChapters`: Chapters are out of order or nest more than one level at a time.
- `FieldNotCarried`: The record or request names a field the dialect does not carry.
- `CreditRoleNotCarried`: The record names a credit role the dialect does not carry.
- `ReadOnlyDialect`: A render request named a dialect that only reads.
- `WrongDialect`: A request named a dialect other than the one being checked against.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

## Functions

### is_dialect_id

```rust
pub fn is_dialect_id(id: &str) -> bool
```

Whether `id` follows the ABI identifier rule `[a-z][a-z0-9_]{0,63}`.

#### Arguments

* `id` - The dialect id to check.

#### Returns

True when the id is well formed.

### is_plain_file_name

```rust
pub fn is_plain_file_name(name: &str) -> bool
```

Whether `name` is a plain file name: non-empty, at most
[`MAX_METADATA_FILE_NAME_BYTES`](#max_metadata_file_name_bytes), not `.` or `..`, and free of `/`, `\`
and control characters.

#### Arguments

* `name` - The file name to check.

#### Returns

True when the name names one file with no folder.

### is_partial_date

```rust
pub fn is_partial_date(date: &str) -> bool
```

Whether `date` is `YYYY`, `YYYY-MM` or `YYYY-MM-DD` with a real month
and a day the month can hold (29 February only in a leap year).

#### Arguments

* `date` - The date text to check.

#### Returns

True when the date has one of the three shapes and is a real date.

### fields_with_values

```rust
pub fn fields_with_values(record: &MetadataFileRecord) -> Vec<MetadataField>
```

The fields a record carries a value for, in [`MetadataField`](generated.md#metadatafield) order.

A value is anything other than the zero value: a set text, a non-empty
list, a known abridgement or expression kind, or a duration. Credit
roles count as a value for [`MetadataField::Credits`](generated.md#metadatafield).

#### Arguments

* `record` - The record to inspect.

#### Returns

Every field with a value.

### validate_metadata_file_record

```rust
pub fn validate_metadata_file_record(
    record: &MetadataFileRecord,
) -> Result<(), MetadataFileError>
```

Checks a record on its own, without its dialect.

The rules:

- `present` and `creditRoles` name each entry at most once, and every
  field with a value (see [`fields_with_values`](#fields_with_values)) is in `present`;
- every credit's role is in `creditRoles`;
- lists stay within [`MAX_METADATA_FILE_LIST_ENTRIES`](README.md#max_metadata_file_list_entries) and chapters within
  [`MAX_METADATA_FILE_CHAPTERS`](README.md#max_metadata_file_chapters);
- text values are within [`MAX_METADATA_FILE_TEXT_BYTES`](README.md#max_metadata_file_text_bytes) (the
  description within [`MAX_METADATA_FILE_DESCRIPTION_BYTES`](README.md#max_metadata_file_description_bytes)), labels and
  names are non-blank single lines, and only the description may hold
  line breaks or tabs;
- `publishedAt` passes [`is_partial_date`](#is_partial_date);
- chapter starts never decrease, the first chapter has depth 0, and each
  chapter is at most one level deeper than the one before it.

#### Arguments

* `record` - A record a dialect returned (untrusted).

#### Returns

`Ok(())` when the record keeps every rule.

#### Errors

The first rule the record breaks, as a [`MetadataFileError`](#metadatafileerror).

### check_record_against_dialect

```rust
pub fn check_record_against_dialect(
    record: &MetadataFileRecord,
    dialect: &MetadataFileDialect,
) -> Result<(), MetadataFileError>
```

Checks a record against the dialect that produced it: `present` names
only fields the dialect carries and `creditRoles` only roles it carries.

#### Arguments

* `record` - A record that already passed [`validate_metadata_file_record`](#validate_metadata_file_record).
* `dialect` - The dialect the host routed the file to.

#### Returns

`Ok(())` when the record stays inside the dialect.

#### Errors

[`MetadataFileError::FieldNotCarried`](#metadatafileerror) or
[`MetadataFileError::CreditRoleNotCarried`](#metadatafileerror).

### check_parse_params

```rust
pub fn check_parse_params(
    params: &MetadataFileParseParams,
) -> Result<(), MetadataFileError>
```

Checks a `MetadataFiles.parse` request before it is sent: a dialect id,
a plain file name, and bytes within [`MAX_METADATA_FILE_BYTES`](README.md#max_metadata_file_bytes).

#### Arguments

* `params` - The request the host is about to send.

#### Returns

`Ok(())` when the request is well formed.

#### Errors

[`MetadataFileError::BadDialectId`](#metadatafileerror), [`MetadataFileError::BadFileName`](#metadatafileerror)
or [`MetadataFileError::BadFileSize`](#metadatafileerror).

### check_render_params

```rust
pub fn check_render_params(
    params: &MetadataFileRenderParams,
) -> Result<(), MetadataFileError>
```

Checks a `MetadataFiles.render` request before it is sent: a dialect id,
fields named once each, existing bytes within [`MAX_METADATA_FILE_BYTES`](README.md#max_metadata_file_bytes),
and a record that passes [`validate_metadata_file_record`](#validate_metadata_file_record).

#### Arguments

* `params` - The request the host is about to send.

#### Returns

`Ok(())` when the request is well formed.

#### Errors

The first rule the request breaks.

### check_render_against_dialect

```rust
pub fn check_render_against_dialect(
    params: &MetadataFileRenderParams,
    dialect: &MetadataFileDialect,
) -> Result<(), MetadataFileError>
```

Checks a render request against the dialect it names: the dialect
writes, and every field to write is one it carries.

#### Arguments

* `params` - A request that already passed [`check_render_params`](#check_render_params).
* `dialect` - The dialect from the manifest.

#### Returns

`Ok(())` when the dialect can write every requested field.

#### Errors

[`MetadataFileError::WrongDialect`](#metadatafileerror), [`MetadataFileError::ReadOnlyDialect`](#metadatafileerror)
or [`MetadataFileError::FieldNotCarried`](#metadatafileerror).

### check_rendered

```rust
pub fn check_rendered(rendered: &MetadataFileRendered) -> Result<(), MetadataFileError>
```

Checks a rendered file: within [`MAX_METADATA_FILE_BYTES`](README.md#max_metadata_file_bytes). An empty file
is allowed: it means the dialect has nothing left to write, and the host
decides whether to keep the empty file or remove it.

#### Arguments

* `rendered` - A dialect's `render` reply (untrusted).

#### Returns

`Ok(())` when the host may write the bytes.

#### Errors

[`MetadataFileError::BadFileSize`](#metadatafileerror).

### decode_metadata_file_record_bytes

```rust
pub fn decode_metadata_file_record_bytes(bytes: &[u8]) -> Result<MetadataFileRecord>
```

Decodes one unpacked single-segment Cap'n Proto `MetadataFileRecord`, the
form the fuzz target feeds.

#### Arguments

* `bytes` - The message bytes (untrusted).

#### Returns

The decoded record.

#### Errors

Returns the Cap'n Proto error the framing or the codec reports.

### encode_metadata_file_record_bytes

```rust
pub fn encode_metadata_file_record_bytes(record: &MetadataFileRecord) -> Result<Vec<u8>>
```

Encodes `record` as an unpacked single-segment Cap'n Proto message, the
inverse of [`decode_metadata_file_record_bytes`](#decode_metadata_file_record_bytes).

#### Arguments

* `record` - The record to encode.

#### Returns

The message bytes.

#### Errors

Returns the Cap'n Proto error the builder reports.

## Constants

### MAX_METADATA_FILE_NAME_BYTES

```rust
pub const MAX_METADATA_FILE_NAME_BYTES: usize = 255;
```

Longest file name `MetadataFiles.parse` accepts, in UTF-8 bytes (the
usual file-system component limit).

### MAX_METADATA_DIALECT_ID_BYTES

```rust
pub const MAX_METADATA_DIALECT_ID_BYTES: usize = 64;
```

Longest dialect id, in bytes; dialect ids follow the ABI identifier rule.
