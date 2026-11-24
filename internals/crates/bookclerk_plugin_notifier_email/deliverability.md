# bookclerk_plugin_notifier_email::deliverability

The operator's deliverability check, with no I/O of its own.

Three checkers look at the mail from different sides, and this module
reads their answers and merges what they found into advice:

- **Postmark's spam check** scores the message the plugin composes with
  SpamAssassin ([`parse_spamcheck`](#parse_spamcheck)). It sees the content only: the mail
  provider adds DKIM signatures and `Received` lines later, so rules about
  those are left to the other two.
- **DNS records** of the From domain, read over DNS-over-HTTPS
  ([`parse_doh_txt`](#parse_doh_txt)): an SPF record ([`check_spf`](#check_spf)) and a DMARC policy
  ([`check_dmarc`](#check_dmarc)).
- **mail-tester.com**, when the operator names their account: the plugin
  sends a real message through the server to a mail-tester address and
  reads the result ([`parse_mail_tester`](#parse_mail_tester)), which covers what only a
  delivered message shows (DKIM, the server's reverse DNS, blocklists).

Everything a checker returns is untrusted: bodies are capped before they
reach these parsers, and every text that reaches the operator is cut by
the host again.

| Item | Kind | Summary |
| --- | --- | --- |
| [`SpamRule`](#spamrule) | struct | One SpamAssassin rule that fired. |
| [`SpamCheck`](#spamcheck) | struct | Postmark's answer: the score and the rules that fired. |
| [`MailTesterAnswer`](#mailtesteranswer) | enum | What one mail-tester poll found. |
| [`parse_spamcheck`](#parse_spamcheck) | function | Reads Postmark's answer: `{"success": true, "score": "1.5", "rules": [{"score": "0.1", "description": "..."}], "report": "..."}`. Rule names come from the text report's table (`pts rule description`): the row with the rule's points and description, whatever its position. |
| [`postmark_findings`](#postmark_findings) | function | Turns Postmark's answer into its check and advice. |
| [`parse_doh_txt`](#parse_doh_txt) | function | Reads a DNS-over-HTTPS JSON answer (`application/dns-json`) for TXT records: each `Answer` of type 16, its quoted strings joined. |
| [`join_txt_strings`](#join_txt_strings) | function | Joins a TXT record's strings as presentation format writes them (`"v=spf1 " "include:x -all"`), dropping the quotes and unescaping `\"` and `\\`; unquoted text is kept as is. |
| [`check_spf`](#check_spf) | function | Checks the From domain's SPF record (RFC 7208): exactly one TXT record starting `v=spf1`, ending in `-all` or `~all`. |
| [`check_dmarc`](#check_dmarc) | function | Checks the From domain's DMARC policy (RFC 7489): a TXT record at `_dmarc.<domain>` starting `v=DMARC1`. |
| [`dns_findings`](#dns_findings) | function | Turns the DNS lookups into the DNS check and its advice. |
| [`parse_mail_tester`](#parse_mail_tester) | function | Reads a mail-tester JSON answer (`?format=json`). `status: false` means the message has not arrived (`"Mail not found"`), `access: false` that the account may not run tests; otherwise the top-level `mark` (out of 10) and every test object, up to three levels down, whose `statusClass` is `failure` or `warning`, by its `title`. |
| [`mail_tester_findings`](#mail_tester_findings) | function | Turns the mail-tester result into its check and advice. |
| [`merge_advice`](#merge_advice) | function | Merges every check's advice: advice with the same title is one entry citing every source and keeping the most urgent outcome; the list is ordered failed, warning, passed (each in the order raised) and cut to the ABI's cap. |
| [`POSTMARK`](#postmark) | constant | Name of the Postmark check, as checks and advice cite it. |
| [`DNS`](#dns) | constant | Name of the DNS check. |
| [`MAIL_TESTER`](#mail_tester) | constant | Name of the mail-tester check. |
| [`POSTMARK_URL`](#postmark_url) | constant | Postmark's spam check endpoint (free, no account). |
| [`DOH_URL`](#doh_url) | constant | The DNS-over-HTTPS resolver the DNS check asks (JSON API). |
| [`MAIL_TESTER_URL`](#mail_tester_url) | constant | mail-tester.com's site; results are at `/<address local part>`. |
| [`MAIL_TESTER_DOMAIN`](#mail_tester_domain) | constant | The domain mail-tester addresses are at. |
| [`MAX_ANSWER_BYTES`](#max_answer_bytes) | constant | Largest checker answer read, in bytes. |

## Structs

### SpamRule

```rust
pub struct SpamRule {
    pub points: f64,
    pub name: String,
    pub description: String,
}
```

One SpamAssassin rule that fired.

**Fields**

- `points`: Points it added (negative for points taken off).
- `name`: Rule name (`HTML_IMAGE_ONLY_16`); empty when the answer gave none.
- `description`: What it means.

Implements: `Clone`, `Debug`, `PartialEq`

### SpamCheck

```rust
pub struct SpamCheck {
    pub score: f64,
    pub rules: Vec<SpamRule>,
}
```

Postmark's answer: the score and the rules that fired.

**Fields**

- `score`: Total SpamAssassin score.
- `rules`: Rules that fired, in the answer's order.

Implements: `Clone`, `Debug`, `PartialEq`

## Enums

### MailTesterAnswer

```rust
pub enum MailTesterAnswer {
    Waiting,
    Ready { mark: f64, findings: Vec<(bool, String)> },
}
```

What one mail-tester poll found.

**Variants**

- `Waiting`: The message has not arrived yet; ask again.
- `Ready`: The result: the mark out of 10 and the tests that did not pass.
  - `mark`: Mark out of 10.
  - `findings`: `(failed, title)` of each test marked `failure` (true) or
    `warning` (false), in the answer's order.

Implements: `Clone`, `Debug`, `PartialEq`

## Functions

### parse_spamcheck

```rust
pub fn parse_spamcheck(body: &str) -> Result<SpamCheck, String>
```

Reads Postmark's answer: `{"success": true, "score": "1.5", "rules":
[{"score": "0.1", "description": "..."}], "report": "..."}`. Rule names
come from the text report's table (`pts rule description`): the row with
the rule's points and description, whatever its position.

#### Arguments

* `body` - The answer's body.

#### Errors

A short reason when the answer is not a successful check.

### postmark_findings

```rust
pub fn postmark_findings(
    answer: Result<SpamCheck, String>,
) -> (NotifyDeliverabilityCheck, Vec<NotifyDeliverabilityAdvice>)
```

Turns Postmark's answer into its check and advice.

#### Arguments

* `answer` - What [`parse_spamcheck`](#parse_spamcheck) read, or why the check did not
  run.

### parse_doh_txt

```rust
pub fn parse_doh_txt(body: &str) -> Result<Vec<String>, String>
```

Reads a DNS-over-HTTPS JSON answer (`application/dns-json`) for TXT
records: each `Answer` of type 16, its quoted strings joined.

#### Arguments

* `body` - The answer's body.

#### Returns

The records; an empty list for a missing name (`NXDOMAIN`) or no
records.

#### Errors

A short reason when the answer is unreadable or the lookup failed.

### join_txt_strings

```rust
pub fn join_txt_strings(data: &str) -> String
```

Joins a TXT record's strings as presentation format writes them
(`"v=spf1 " "include:x -all"`), dropping the quotes and unescaping `\"`
and `\\`; unquoted text is kept as is.

#### Arguments

* `data` - The record's data.

### check_spf

```rust
pub fn check_spf(domain: &str, records: &[String]) -> NotifyDeliverabilityAdvice
```

Checks the From domain's SPF record (RFC 7208): exactly one TXT record
starting `v=spf1`, ending in `-all` or `~all`.

#### Arguments

* `domain` - The From domain.
* `records` - Its TXT records.

### check_dmarc

```rust
pub fn check_dmarc(domain: &str, records: &[String]) -> NotifyDeliverabilityAdvice
```

Checks the From domain's DMARC policy (RFC 7489): a TXT record at
`_dmarc.<domain>` starting `v=DMARC1`.

#### Arguments

* `domain` - The From domain.
* `records` - The TXT records at `_dmarc.<domain>`.

### dns_findings

```rust
pub fn dns_findings(
    domain: &str,
    spf: Result<Vec<String>, String>,
    dmarc: Result<Vec<String>, String>,
) -> (NotifyDeliverabilityCheck, Vec<NotifyDeliverabilityAdvice>)
```

Turns the DNS lookups into the DNS check and its advice.

#### Arguments

* `domain` - The From domain.
* `spf` - The domain's TXT records, or why the lookup failed.
* `dmarc` - The TXT records at `_dmarc.<domain>`, or why it failed.

### parse_mail_tester

```rust
pub fn parse_mail_tester(body: &str) -> Result<MailTesterAnswer, String>
```

Reads a mail-tester JSON answer (`?format=json`). `status: false` means
the message has not arrived (`"Mail not found"`), `access: false` that
the account may not run tests; otherwise the top-level `mark`
(out of 10) and every test object, up to three levels down, whose
`statusClass` is `failure` or `warning`, by its `title`.

#### Arguments

* `body` - The answer's body.

#### Errors

A short reason when the answer is unreadable.

### mail_tester_findings

```rust
pub fn mail_tester_findings(
    answer: Result<Option<MailTesterAnswer>, String>,
    report_url: &str,
) -> (NotifyDeliverabilityCheck, Vec<NotifyDeliverabilityAdvice>)
```

Turns the mail-tester result into its check and advice.

#### Arguments

* `answer` - The final answer, or why the check did not run (`Ok(None)`
  when no account is set).
* `report_url` - The result page.

### merge_advice

```rust
pub fn merge_advice(
    found: Vec<NotifyDeliverabilityAdvice>,
) -> Vec<NotifyDeliverabilityAdvice>
```

Merges every check's advice: advice with the same title is one entry
citing every source and keeping the most urgent outcome; the list is
ordered failed, warning, passed (each in the order raised) and cut to
the ABI's cap.

#### Arguments

* `found` - Advice from every check, in check order.

## Constants

### POSTMARK

```rust
pub const POSTMARK: &str = "Postmark spam check";
```

Name of the Postmark check, as checks and advice cite it.

### DNS

```rust
pub const DNS: &str = "DNS records";
```

Name of the DNS check.

### MAIL_TESTER

```rust
pub const MAIL_TESTER: &str = "mail-tester.com";
```

Name of the mail-tester check.

### POSTMARK_URL

```rust
pub const POSTMARK_URL: &str = "https://spamcheck.postmarkapp.com/filter";
```

Postmark's spam check endpoint (free, no account).

### DOH_URL

```rust
pub const DOH_URL: &str = "https://cloudflare-dns.com/dns-query";
```

The DNS-over-HTTPS resolver the DNS check asks (JSON API).

### MAIL_TESTER_URL

```rust
pub const MAIL_TESTER_URL: &str = "https://www.mail-tester.com";
```

mail-tester.com's site; results are at `/<address local part>`.

### MAIL_TESTER_DOMAIN

```rust
pub const MAIL_TESTER_DOMAIN: &str = "srv1.mail-tester.com";
```

The domain mail-tester addresses are at.

### MAX_ANSWER_BYTES

```rust
pub const MAX_ANSWER_BYTES: usize = _;
```

Largest checker answer read, in bytes.
