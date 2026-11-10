# bookclerk_workerd::control

Control messages workerd writes on `--control-fd`: the port each socket bound.

Where the launcher cannot hand workerd an already-bound socket (Windows has
no `--socket-fd`), workerd binds the bridge RPC socket on `127.0.0.1:0`
itself and reports the port the operating system picked with one JSON line
per socket:

```text
{"event":"listen","socket":"rpc","port":53117}
```

The control channel is workerd's stdout, which isolate `console.*` output
shares. The generated config turns on structured logging whenever workerd
reports its port ([`crate::config::ListenSpec::ReportedTcp`](config.md#listenspec)), so every
line an isolate can produce is a JSON object with `timestamp`, `level` and
`message` keys. [`listen_port`](#listen_port) accepts only an object with exactly the
three control keys, so isolate output can never be read as a control
message. The `workerd_control` fuzz target checks both properties.

| Item | Kind | Summary |
| --- | --- | --- |
| [`listen_port`](#listen_port) | function | Reads the port `socket` is listening on from one workerd control line. |
| [`bridge_port`](#bridge_port) | function | Learns the bridge RPC port of one workerd child and forwards its stdout to tracing. |
| [`forward_lines`](#forward_lines) | function | Forwards one workerd output stream to tracing, line by line, on a spawned task. |
| [`RPC_SOCKET`](#rpc_socket) | constant | Name of the bridge RPC socket in the generated workerd config. |
| [`MAX_CONTROL_LINE_BYTES`](#max_control_line_bytes) | constant | Longest control line worth parsing, in bytes; a `listen` message is under 100. |
| [`MAX_LOG_LINE_BYTES`](#max_log_line_bytes) | constant | Longest workerd output line kept for logging, in bytes; the rest of a longer line is read and dropped, so isolate output cannot make the launcher buffer or log an unbounded line. |

## Functions

### listen_port

```rust
pub fn listen_port(line: &[u8], socket: &str) -> Option<NonZeroU16>
```

Reads the port `socket` is listening on from one workerd control line.

#### Arguments

* `line` - One line of workerd's stdout, without the line terminator
  (trailing ASCII whitespace, such as a Windows `\r`, is ignored).
* `socket` - Socket name from the generated config, usually [`RPC_SOCKET`](#rpc_socket).

#### Returns

The bound port when `line` is a JSON object holding exactly `event`
(`"listen"`), `socket` (equal to `socket`) and a non-zero `port`, each key
once. `None` for every other line, including structured log lines, lines
over [`MAX_CONTROL_LINE_BYTES`](#max_control_line_bytes) and other sockets' events.

### bridge_port

```rust
pub async fn bridge_port<R>(
    listen: &ListenSpec,
    stdout: Option<R>,
    timeout: Duration,
) -> Result<u16>
where
    R: AsyncRead + Unpin + Send + 'static,
```

Learns the bridge RPC port of one workerd child and forwards its stdout to tracing.

#### Arguments

* `listen` - Listen mode the child's config was generated with.
* `stdout` - The child's stdout; it carries the control channel when
  `listen` is [`ListenSpec::ReportedTcp`](config.md#listenspec).
* `timeout` - How long workerd gets to report its port.

#### Returns

The port from `listen` when it is fixed, otherwise the port workerd
reported for [`RPC_SOCKET`](#rpc_socket). Every other stdout line, before and after the
report, is logged under the `workerd` target.

#### Errors

For [`ListenSpec::ReportedTcp`]: when `stdout` is `None`, when it ends or
fails before the report (workerd exited, for example because its bind
failed), or when nothing arrives within `timeout`.

### forward_lines

```rust
pub fn forward_lines<R>(stream: Option<R>)
where
    R: AsyncRead + Unpin + Send + 'static,
```

Forwards one workerd output stream to tracing, line by line, on a spawned task.

#### Arguments

* `stream` - The child's stdout or stderr; `None` when it was not piped.

## Constants

### RPC_SOCKET

```rust
pub const RPC_SOCKET: &str = "rpc";
```

Name of the bridge RPC socket in the generated workerd config.

### MAX_CONTROL_LINE_BYTES

```rust
pub const MAX_CONTROL_LINE_BYTES: usize = 1024;
```

Longest control line worth parsing, in bytes; a `listen` message is under 100.

### MAX_LOG_LINE_BYTES

```rust
pub const MAX_LOG_LINE_BYTES: usize = _;
```

Longest workerd output line kept for logging, in bytes; the rest of a
longer line is read and dropped, so isolate output cannot make the
launcher buffer or log an unbounded line.
