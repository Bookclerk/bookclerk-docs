# bookclerk_plugin_notifier_email::checkers

Reaches the deliverability checkers over HTTPS.

[`HttpCheckers`](#httpcheckers) asks Postmark's spam check, Cloudflare's DNS-over-HTTPS
resolver and mail-tester.com, each through the host's socket proxy, so
only the hosts in the manifest's `tcp` grants the operator approved can
be reached. Answers are read up to
[`crate::deliverability::MAX_ANSWER_BYTES`](deliverability.md#max_answer_bytes) and handed, unparsed, to
[`crate::deliverability`](deliverability.md). Errors never carry the message, a setting or
the checker's own text.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Checkers`](#checkers) | trait | The checkers' answers, as text; tests swap in canned ones. |
| [`HttpCheckers`](#httpcheckers) | struct | The checkers over HTTPS through the host. |

## Traits

### Checkers

```rust
pub trait Checkers {
    fn spamcheck<'life0, 'life1, 'async_trait>(
        &'life0 self,
        raw: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<String, String>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn txt<'life0, 'life1, 'async_trait>(
        &'life0 self,
        name: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<String, String>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn mail_tester<'life0, 'life1, 'async_trait>(
        &'life0 self,
        id: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<String, String>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn pause<'life0, 'async_trait>(
        &'life0 self,
        wait: Duration,
    ) -> Pin<Box<dyn Future<Output = ()> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

The checkers' answers, as text; tests swap in canned ones.

#### Checkers::spamcheck

```rust
fn spamcheck<'life0, 'life1, 'async_trait>(
    &'life0 self,
    raw: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<String, String>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Posts a composed message to Postmark's spam check.

##### Arguments

* `raw` - The whole message, CRLF line ends.

##### Errors

A short reason when the service can't be reached or answers badly.

#### Checkers::txt

```rust
fn txt<'life0, 'life1, 'async_trait>(
    &'life0 self,
    name: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<String, String>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Looks up a name's TXT records over DNS-over-HTTPS.

##### Arguments

* `name` - The DNS name.

##### Errors

A short reason when the resolver can't be reached.

#### Checkers::mail_tester

```rust
fn mail_tester<'life0, 'life1, 'async_trait>(
    &'life0 self,
    id: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<String, String>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Reads one mail-tester result as JSON.

##### Arguments

* `id` - The test address's local part (`<account>-<token>`).

##### Errors

A short reason when mail-tester can't be reached.

#### Checkers::pause

```rust
fn pause<'life0, 'async_trait>(
    &'life0 self,
    wait: Duration,
) -> Pin<Box<dyn Future<Output = ()> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Waits between mail-tester polls.

##### Arguments

* `wait` - How long.

Implemented by: `HttpCheckers`

## Structs

### HttpCheckers

```rust
pub struct HttpCheckers;
```

The checkers over HTTPS through the host.

Implements: `Checkers`, `Clone`, `Copy`, `Debug`, `Default`
