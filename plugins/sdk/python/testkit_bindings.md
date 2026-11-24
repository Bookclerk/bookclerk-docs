# bookclerk_plugin_sdk.testkit_bindings

In-memory `KV` and `WEBHOOKS` bindings for plugin tests.

They hold the limits and error codes the host applies, so code that passes
against them does not first meet a cap on a deployed host: keys are 1 to
`MAX_KV_KEY_BYTES` UTF-8 bytes, values at most the namespace's
`max_value_bytes`, metadata at most `MAX_KV_METADATA_BYTES` of JSON, and
a webhook pull leases 1 to `MAX_WEBHOOK_PULL_BATCH` requests for at most
`MAX_WEBHOOK_VISIBILITY_MS`. The TypeScript testkit's `createTestKv` and
`createTestWebhooks` behave the same way.

**Example**

```python
>>> from bookclerk_plugin_sdk.testkit import TestKv, TestWebhooks
>>> env = {"KV": TestKv(), "WEBHOOKS": TestWebhooks()}
>>> env["WEBHOOKS"].deliver("changes", b"{}")
```

| Name | Kind | Summary |
| --- | --- | --- |
| [`TestKv`](#testkv) | class | A `KvNamespace` held in memory that applies the host's limits. |
| [`TestWebhooks`](#testwebhooks) | class | A `WebhooksBinding` held in memory that applies the host's limits. |
| [`TestKvOperation`](#testkvoperation) | constant | Operations a test can make fail once with `TestKv.fail_next`. |

## Classes

### TestKv

```python
class TestKv
```

A `KvNamespace` held in memory that applies the host's limits.

**Attributes**

- `max_value_bytes`: The namespace's `max_value_bytes`, never above `MAX_KV_VALUE_BYTES`.
- `max_keys`: The namespace's key-count limit, or `None`.

#### TestKv.__init__

```python
def __init__(
    self,
    *,
    max_value_bytes: int=MAX_KV_VALUE_BYTES,
    max_keys: int | None=None,
    now: Callable[[], int] | None=None,
) -> None
```

Start empty.

**Args**

- `max_value_bytes`: The namespace's value cap.
- `max_keys`: The namespace's key-count limit; a new key past it is `forbidden`.
- `now`: Clock in milliseconds since the Unix epoch.

#### TestKv.fail_next

```python
def fail_next(self, op: TestKvOperation, error: PluginError) -> None
```

Make the next call of `op` raise `error` instead of running.

**Args**

- `op`: The operation.
- `error`: What it raises.

#### TestKv.snapshot

```python
def snapshot(self) -> dict[str, str]
```

Return live keys and their values as text, for assertions.

**Returns**

Key to UTF-8 value, in key order.

#### TestKv.get

```python
async def get(self, key: str, type: str='text') -> Any
```

Read one key.

**Args**

- `key`: The key.
- `type`: `text`, `json` or `bytes`.

**Returns**

The value, or `None` when absent or expired.

#### TestKv.get_with_metadata

```python
async def get_with_metadata(self, key: str, type: str='text') -> tuple[Any, Any]
```

Read one key with its metadata.

**Args**

- `key`: The key.
- `type`: `text`, `json` or `bytes`.

**Returns**

`(value, metadata)`; both `None` when absent.

**Raises**

- `ValueError`: For another `type`.

#### TestKv.put

```python
async def put(
    self,
    key: str,
    value: str | bytes,
    *,
    expiration: int | None=None,
    expiration_ttl: int | None=None,
    metadata: Any=None,
) -> None
```

Write one key, replacing any previous value and metadata.

**Args**

- `key`: The key.
- `value`: `str` (stored as UTF-8) or `bytes`.
- `expiration`: Absolute expiry in seconds since the Unix epoch.
- `expiration_ttl`: Relative expiry in seconds from the write.
- `metadata`: JSON-serializable metadata.

**Raises**

- `PluginError`: `payload_too_large`, `invalid_params` or `forbidden` as the host raises them.

#### TestKv.delete

```python
async def delete(self, key: str) -> None
```

Delete one key; a missing key is not an error.

**Args**

- `key`: The key.

#### TestKv.list

```python
async def list(self, *, prefix: str='', cursor: str='', limit: int=0) -> dict[str, Any]
```

List keys in lexical order, one page at a time.

**Args**

- `prefix`: Only keys starting with this prefix.
- `cursor`: Cursor from a previous page.
- `limit`: Most keys to return (`MAX_LIST_PAGE` when 0).

**Returns**

`{"keys": [...], "list_complete": bool, "cursor"?: str}`.

**Raises**

- `PluginError`: `invalid_params` for a limit over `MAX_LIST_PAGE`.

### TestWebhooks

```python
class TestWebhooks
```

A `WebhooksBinding` held in memory that applies the host's limits.

Leased requests stay leased until acknowledged or retried; lease expiry
is not modelled, so release a lease with `ack(name, retries=[...])`.

#### TestWebhooks.__init__

```python
def __init__(self, *, base_url: str | None='https://library.example/hooks/') -> None
```

Start with nothing waiting.

**Args**

- `base_url`: What `url(name)` starts with, or `None` for a library the Internet cannot reach.

#### TestWebhooks.deliver

```python
def deliver(
    self,
    name: str,
    body: str | bytes,
    headers: dict[str, str] | None=None,
) -> None
```

Queue one inbound request, as if the outside service sent it.

**Args**

- `name`: `[[webhooks]]` name.
- `body`: Body text or bytes.
- `headers`: Request headers.

#### TestWebhooks.mark_missed

```python
def mark_missed(self, name: str) -> None
```

Make the next pull of `name` report `missed`.

**Args**

- `name`: `[[webhooks]]` name.

#### TestWebhooks.waiting

```python
def waiting(self, name: str) -> int
```

Return how many requests wait (not leased) for `name`.

**Args**

- `name`: `[[webhooks]]` name.

**Returns**

The count.

#### TestWebhooks.acked

```python
def acked(self, name: str) -> list[str]
```

Return lease ids acknowledged for `name`, in order.

**Args**

- `name`: `[[webhooks]]` name.

**Returns**

The ids.

#### TestWebhooks.retried

```python
def retried(self, name: str) -> list[str]
```

Return lease ids released for retry for `name`, in order.

**Args**

- `name`: `[[webhooks]]` name.

**Returns**

The ids.

#### TestWebhooks.url

```python
async def url(self, name: str) -> str | None
```

Return the URL an outside service should call for `name`.

**Args**

- `name`: `[[webhooks]]` name.

**Returns**

The URL, or `None` for an unreachable library.

#### TestWebhooks.pull

```python
async def pull(
    self,
    name: str,
    *,
    batch_size: int=DEFAULT_WEBHOOK_PULL_BATCH,
    visibility_timeout_ms: int=0,
) -> WebhookPullResult
```

Lease up to `batch_size` waiting requests, oldest first.

**Args**

- `name`: `[[webhooks]]` name.
- `batch_size`: 1 to `MAX_WEBHOOK_PULL_BATCH`.
- `visibility_timeout_ms`: 0 to `MAX_WEBHOOK_VISIBILITY_MS`.

**Returns**

The leased requests and the missed flag.

**Raises**

- `PluginError`: `invalid_params` for an out-of-range option.

#### TestWebhooks.ack

```python
async def ack(
    self,
    name: str,
    *,
    acks: list[str] | None=None,
    retries: list[str] | None=None,
) -> None
```

Finish leased requests: `acks` are deleted, `retries` offered again first.

**Args**

- `name`: `[[webhooks]]` name.
- `acks`: Lease ids handled.
- `retries`: Lease ids to offer again.

## Constants

### TestKvOperation

```python
TestKvOperation = Literal['get', 'put', 'delete', 'list']
```

Operations a test can make fail once with [`TestKv.fail_next`](#testkvfail_next).
