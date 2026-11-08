# bookclerk_plugin_sdk.limits

Typed `limit_exceeded` errors: the SQL-v1 cap a request or result broke.

The host and every SDK format a limit error through
[`limit_exceeded_message`](#limit_exceeded_message) and parse it back through
[`parse_limit_exceeded`](#parse_limit_exceeded), so the record survives a plain-string hop
(for example through an engine error) and the `details` JSON on the wire.

| Name | Kind | Summary |
| --- | --- | --- |
| [`LimitExceeded`](#limitexceeded) | class | A negotiated cap that a request or its result exceeded. |
| [`limit_exceeded_details`](#limit_exceeded_details) | function | Build the `details` object (`{"limit", "cap", "used"}`). |
| [`limit_exceeded_message`](#limit_exceeded_message) | function | Format the canonical message (`limit exceeded: <name> is <cap>; used <n>`). |
| [`limit_from_details`](#limit_from_details) | function | Read a `LimitExceeded` back from a `details` object. |
| [`parse_limit_exceeded`](#parse_limit_exceeded) | function | Parse a message produced by `limit_exceeded_message`. |

## Classes

### LimitExceeded

```python
@dataclass(frozen=True)
class LimitExceeded
```

A negotiated cap that a request or its result exceeded.

**Attributes**

- `limit`: Capability field name as advertised (`maxResultBytes`, `maxRequestBytes`, ...).
- `cap`: The negotiated cap.
- `used`: The observed value that exceeded `cap`.

## Functions

### limit_exceeded_details

```python
def limit_exceeded_details(limit: LimitExceeded) -> dict[str, Any]
```

Build the `details` object (`{"limit", "cap", "used"}`).

**Args**

- `limit`: Exceeded cap.

**Returns**

JSON-compatible detail object.

### limit_exceeded_message

```python
def limit_exceeded_message(limit: LimitExceeded) -> str
```

Format the canonical message (`limit exceeded: <name> is <cap>; used <n>`).

**Args**

- `limit`: Exceeded cap.

**Returns**

Message text the host and every SDK parse back.

### limit_from_details

```python
def limit_from_details(details: dict[str, Any] | None) -> LimitExceeded | None
```

Read a [`LimitExceeded`](#limitexceeded) back from a `details` object.

**Args**

- `details`: Parsed `details` JSON object, or `None`.

**Returns**

The record, or `None` when a field is missing or mistyped: the name
must match the message grammar (ASCII letters, digits, `_`) and the
counts must be non-negative integers, as [`parse_limit_exceeded`](#parse_limit_exceeded)
would produce.

### parse_limit_exceeded

```python
def parse_limit_exceeded(message: str) -> LimitExceeded | None
```

Parse a message produced by [`limit_exceeded_message`](#limit_exceeded_message).

**Args**

- `message`: Error text, possibly wrapping the canonical message.

**Returns**

The exceeded cap, or `None` when the text is not a limit error.
