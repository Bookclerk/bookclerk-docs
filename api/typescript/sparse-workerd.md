# sparse-workerd

## Type Aliases

### GeneratedConfig

```ts
type GeneratedConfig = object;
```

Paths produced by [materializeConfig](#materializeconfig).

#### Properties

##### configPath

```ts
configPath: string;
```

Absolute path to the Cap'n Proto config file (under [stateDir](#statedir)).

##### importPath

```ts
importPath: string;
```

Pass to `workerd serve --import-path` for `/modules/…` embeds.

##### listenAddr

```ts
listenAddr: string;
```

Loopback listen address (`127.0.0.1:<port>`).

##### stateDir

```ts
stateDir: string;
```

Writable session directory holding `.bookclerk/` + Cap'n Proto.

***

### MaterializeOptions

```ts
type MaterializeOptions = object;
```

Options for [materializeConfig](#materializeconfig).

#### Properties

##### bridgeToken

```ts
bridgeToken: string;
```

Per-isolate bearer for `/rpc` and `/health` (`BRIDGE_TOKEN`).

Required — generate once per smoke/isolate and send on every bridge request.

##### configName?

```ts
optional configName?: string;
```

Cap'n Proto output filename under the session dir (default `workerd-config.capnp`).

##### listenPort

```ts
listenPort: number;
```

Loopback listen port for the HTTP bridge socket (`127.0.0.1:<port>`).

##### sdkRoot?

```ts
optional sdkRoot?: string;
```

Absolute path to the `@bookclerk/plugin-sdk` package root when it differs from
the default (used by unit tests that vendor a fixture package tree).

##### stateDir?

```ts
optional stateDir?: string;
```

Existing session directory for generated embeds (default: allocate under
`$TMPDIR`). Cap'n Proto + `.bookclerk/` are written here — never under the
tainted plugin install root.

***

### WorkerdAsset

```ts
type WorkerdAsset = object;
```

One platform asset entry inside [WorkerdPin](#workerdpin).

#### Properties

##### artifact

```ts
artifact: string;
```

Gzip-compressed release artifact filename on the Cloudflare workerd GitHub release.

##### sha256\_hex

```ts
sha256_hex: string;
```

Expected SHA-256 hex digest of the compressed artifact bytes (not the gunzipped binary).

***

### WorkerdPin

```ts
type WorkerdPin = object;
```

Contents of `workerd-pin.json` shipping with this package.

#### Properties

##### assets

```ts
assets: Record<string, WorkerdAsset>;
```

Platform key → artifact map (`linux-x86_64`, `macos-aarch64`, `windows-x86_64`, …).

##### bundled\_compat\_date

```ts
bundled_compat_date: string;
```

Compatibility date bundled with this pin for Cap'n Proto configs.

##### release\_tag

```ts
release_tag: string;
```

GitHub release tag (for example `v1.20250310.0`) that identifies this pin.

##### version\_stamp

```ts
version_stamp: string;
```

Stamp filename written beside the installed binary to record `release_tag`.

## Functions

### allocateWorkerdStateDir()

```ts
function allocateWorkerdStateDir(pluginRoot): string;
```

Allocate a unique writable session directory for workerd generated embeds.

Keys the leaf by a short hash of the plugin root plus a random nonce so
concurrent sessions cannot clobber Cap'n Proto. Prefer `$TMPDIR` / OS temp;
fall back to `.bookclerk-state` beside the plugin only when temp is unset.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `pluginRoot` | `string` | Plugin install root (used only as an opaque id seed). |

#### Returns

`string`

Canonical absolute session directory.

***

### assertPathInside()

```ts
function assertPathInside(root, candidate): string;
```

Resolves `candidate` under `root` via `path.resolve` + `path.relative` barrier.

Rejects NUL bytes and `..` path components (names like `..draft` and
`edition..2` are allowed). Uses the CodeQL-recognized
`path.relative` / `startsWith("..")` containment pattern — no filesystem
probes before the barrier (those are sinks under local threat modeling).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `root` | `string` | Trusted directory (resolved). |
| `candidate` | `string` | Absolute path or path relative to `root`. |

#### Returns

`string`

Absolute resolved path under `root`.

#### Throws

When the path is empty, contains NUL/`..`, or escapes `root`.

***

### binaryMatchesPin()

```ts
function binaryMatchesPin(bin, pin): boolean;
```

True when `bin` is a usable file matching `pin`.

A sibling version stamp is enough when the binary exists. A stamp without a
binary is not current. When no Bookclerk stamp matches, probe `--version`
on the absolute path with fixed argv (`shell: false`, no `PATH` lookup).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `bin` | `string` | Absolute candidate executable. |
| `pin` | [`WorkerdPin`](#workerdpin) | Loaded workerd pin. |

#### Returns

`boolean`

Whether the binary is present and matches the pin.

***

### binaryName()

```ts
function binaryName(): string;
```

Returns the on-disk workerd binary basename for the current platform.

#### Returns

`string`

`workerd.exe` on Windows, otherwise `workerd`.

***

### copyFileUnder()

```ts
function copyFileUnder(
   root, 
   name, 
   src
): string;
```

Copy `src` to `root` / `name` after resolve + barrier on the destination.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `root` | `string` | Trusted destination directory (must already exist). |
| `name` | `string` | Single path component filename. |
| `src` | `string` | Absolute source file (already validated by the caller). |

#### Returns

`string`

Absolute destination path under `root`.

***

### defaultCacheDir()

```ts
function defaultCacheDir(): string;
```

Resolve the workerd cache directory.

Honors `BOOKCLERK_WORKERD_CACHE` as the selected install root (workspace
`target/`, `/opt`, external volumes). Otherwise uses
`~/.cache/bookclerk/workerd`. Derived files stay under that root.

#### Returns

`string`

Absolute cache directory path.

***

### egressDomainsFor()

```ts
function egressDomainsFor(
   needsPython, 
   mode, 
   base
): string[];
```

Builds the egress allowlist, appending Pyodide hosts when needed.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `needsPython` | `boolean` | Whether the plugin embeds Python modules. |
| `mode` | `string` | Manifest network mode (`outbound` enables Pyodide hosts). |
| `base` | `string`[] | Domains declared in `capabilities.network.domains`. |

#### Returns

`string`[]

Deduplicated domain list for the egress policy JSON.

***

### ensureDirUnder()

```ts
function ensureDirUnder(root, rel): string;
```

Create `root` / `rel` as a directory after resolve + relative/`startsWith` barrier.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `root` | `string` | Trusted directory. |
| `rel` | `string` | Relative suffix (may be multi-segment when each segment is safe). |

#### Returns

`string`

Absolute directory path under `root`.

***

### ensureWorkerd()

```ts
function ensureWorkerd(cacheDir?, root?): Promise<string>;
```

Ensures `cacheDir/workerd` matches the pin, downloading if needed.

Honors `BOOKCLERK_WORKERD_BIN` when that absolute binary exists and matches
the pin (stamp or `--version`). The selected cache directory is the trusted
install root; children stay under it.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `cacheDir?` | `string` | Directory that will hold the binary (default [defaultCacheDir](#defaultcachedir)). |
| `root?` | `string` | Package root for loading the pin (default [packageRoot](#packageroot)). |

#### Returns

`Promise`\<`string`\>

Absolute path to a current `workerd` binary.

#### Throws

When the platform has no pin, download fails, or the digest mismatches.

***

### loadPin()

```ts
function loadPin(root?): WorkerdPin;
```

Loads and parses `workerd-pin.json` from the package root.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `root?` | `string` | Package root override (defaults to [packageRoot](#packageroot)). |

#### Returns

[`WorkerdPin`](#workerdpin)

Parsed pin document.

***

### materializeConfig()

```ts
function materializeConfig(
   pluginRoot, 
   manifest, 
   options
): GeneratedConfig;
```

Materializes bridge assets + Cap'n Proto under a host session directory.

Copies bridge scripts into `stateDir/.bookclerk/`, embeds plugin modules via
Cap'n Proto `/modules/…` + `--import-path` (read-only install root), and
writes the Cap'n Proto config beside those embeds — never under the tainted
plugin install path.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `pluginRoot` | `string` | Plugin directory containing `plugin.toml` and modules. |
| `manifest` | `Manifest` | Validated workerd manifest. |
| `options` | [`MaterializeOptions`](#materializeoptions) | Listen port and bridge token. |

#### Returns

[`GeneratedConfig`](#generatedconfig)

Generated config path, listen address, session dir, and import path.

#### Throws

When `[workerd]` is missing, modules are absent, or the
  bridge token is empty.

***

### packageRoot()

```ts
function packageRoot(): string;
```

Resolves the `@bookclerk/plugin-sdk` package root.

Works whether this module is loaded from `src/sparse-workerd` or
`dist/sparse-workerd`.

#### Returns

`string`

Absolute path to `packages/plugin-sdk`.

***

### platformKey()

```ts
function platformKey(platform?, arch?): string | undefined;
```

Maps Node `process.platform` / `process.arch` to a pin asset key.

#### Parameters

| Parameter | Type | Default value | Description |
| ------ | ------ | ------ | ------ |
| `platform?` | `Platform` | `process.platform` | Node platform string (default `process.platform`). |
| `arch?` | `Architecture` | `process.arch` | Node arch string (default `process.arch`). |

#### Returns

`string` \| `undefined`

Pin asset key, or `undefined` when the host is unsupported.

***

### pluginGlobalOutbound()

```ts
function pluginGlobalOutbound(mode): "blocked" | "egress";
```

Maps a manifest network mode to the plugin worker's `globalOutbound`.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `mode` | `string` | Manifest `capabilities.network.mode` (`outbound` or deny-like). |

#### Returns

`"blocked"` \| `"egress"`

Cap'n Proto outbound service name (`egress` or `blocked`).

***

### refuseSymlinkPath()

```ts
function refuseSymlinkPath(trustedRoot, candidate): string;
```

Require `candidate` under `trustedRoot` with no symlink suffix components.

Resolves the trusted root identity once (a symlinked operator/plugin root is
allowed). Only path components *below* that identity are refused when they
are symlinks.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `trustedRoot` | `string` | Original operator/plugin root. |
| `candidate` | `string` | Path previously produced by [assertPathInside](#assertpathinside). |

#### Returns

`string`

The validated absolute path.

#### Throws

When a suffix component is a symlink or escapes `trustedRoot`.

***

### runSmoke()

```ts
function runSmoke(pluginDir): Promise<string>;
```

Smokes a workerd plugin directory without the Rust `bookclerk-workerd` binary.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `pluginDir` | `string` | Plugin root with `runtime = "workerd"`. |

#### Returns

`Promise`\<`string`\>

Human-readable success summary including describe/health JSON.

#### Throws

When the runtime is not workerd, workerd fails to start, or
  bridge calls fail.

#### Example

```ts
console.log(await runSmoke("./my-plugin"));
```

***

### validateFetchUrl()

```ts
function validateFetchUrl(url): string;
```

Validates a URL before `fetch` / download.

Allows `https:` anywhere, or `http:` only to loopback hosts.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `url` | `string` | Absolute URL string. |

#### Returns

`string`

Canonical href safe to request.

#### Throws

When the URL is invalid or uses a disallowed scheme/host.

***

### validateSpawnExecutable()

```ts
function validateSpawnExecutable(bin, trustedRoot?): string;
```

Validates a workerd (or helper) binary path before spawn.

Requires an absolute path with no NUL bytes. When `trustedRoot` is set,
resolves both paths and requires the binary to stay under that root.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `bin` | `string` | Candidate executable path. |
| `trustedRoot?` | `string` | Optional directory the binary must remain under. |

#### Returns

`string`

Absolute validated path.

#### Throws

When the path is relative, contains NUL, or escapes `trustedRoot`.

***

### writeFileUnder()

```ts
function writeFileUnder(
   root, 
   name, 
   contents
): string;
```

Write `contents` to `root` / `name` (single component) after resolve + barrier.

Assumes `root` already exists (session/cache dirs are created first). No
mkdir of the tainted root — only write the barriered child path.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `root` | `string` | Trusted directory (must already exist). |
| `name` | `string` | Single path component filename. |
| `contents` | `string` \| `ArrayBufferView`\<`ArrayBufferLike`\> | Bytes or string to write. |

#### Returns

`string`

Absolute file path under `root`.
