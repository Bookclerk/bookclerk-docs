# db-value

## Type Aliases

### DbValue

```ts
type DbValue = 
  | {
  kind: "null";
  value: DbType;
}
  | {
  kind: "boolean";
  value: boolean;
}
  | {
  kind: "int64";
  value: bigint;
}
  | {
  kind: "float64";
  value: number;
}
  | {
  kind: "text";
  value: string;
}
  | {
  kind: "bytes";
  value: Uint8Array;
};
```

Closed Cap'n `DbValue` union.

Members are typed null, bool, int64 (`bigint`), finite float64, UTF-8 text
(U+0000 forbidden), and bytes (`Uint8Array`, 0x00 allowed). Unknown `kind`
values fail closed.

## Functions

### decodeDbValue()

```ts
function decodeDbValue(bytes): DbValue;
```

Decodes a standalone unpacked Cap'n `DbValue` message.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `bytes` | `Uint8Array` | Unpacked Cap'n stream. |

#### Returns

[`DbValue`](#dbvalue)

Domain value.

#### Throws

When the buffer is not a valid `DbValue`.

***

### encodeDbValue()

```ts
function encodeDbValue(value): Uint8Array;
```

Encodes a standalone unpacked Cap'n `DbValue` message.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `value` | [`DbValue`](#dbvalue) | Domain value (`bigint` / `Uint8Array`, not JSON number / `b64:`). |

#### Returns

`Uint8Array`

Unpacked Cap'n stream bytes (same encoding as the Rust SDK).

#### Throws

When `int64` is out of range, `float64` is not finite, or TEXT
contains U+0000.

***

### parseDbValue()

```ts
function parseDbValue(raw): DbValue;
```

Parses a JSON `DbValue`. Unknown union members throw.

`int64` accepts a `bigint` or a finite integer `number` (converted to
`bigint`). `bytes` accepts a `Uint8Array` or a `b64:` JSON string.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `raw` | `unknown` | Parsed JSON object. |

#### Returns

[`DbValue`](#dbvalue)

Typed value.

#### Throws

When `raw` is not a closed `DbValue`.

***

### readDbValue()

```ts
function readDbValue(root): DbValue;
```

Reads a `DbValue` from an unpacked Cap'n struct (2 data words, 1 pointer).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `root` | \{ `getBool`(`i`): `boolean`; `getData`(`i`): `Uint8Array`; `getFloat64`(`i`): `number`; `getInt64`(`i`): `bigint`; `getText`(`i`): `string`; `getUint16`(`i`): `number`; \} | Cap'n struct reader. |
| `root.getBool` | (`i`) => `boolean` | - |
| `root.getData` | (`i`) => `Uint8Array` | - |
| `root.getFloat64` | (`i`) => `number` | - |
| `root.getInt64` | (`i`) => `bigint` | - |
| `root.getText` | (`i`) => `string` | - |
| `root.getUint16` | (`i`) => `number` | - |

#### Returns

[`DbValue`](#dbvalue)

Domain value.

#### Throws

When the union member is unknown or a float64 is not finite.

***

### writeDbValue()

```ts
function writeDbValue(root, value): void;
```

Writes a `DbValue` into an unpacked Cap'n struct (2 data words, 1 pointer).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `root` | \{ `setBool`(`i`, `v`): `void`; `setData`(`i`, `v`): `void`; `setFloat64`(`i`, `v`): `void`; `setInt64`(`i`, `v`): `void`; `setText`(`i`, `v`): `void`; `setUint16`(`i`, `v`): `void`; \} | Cap'n struct builder. |
| `root.setBool` | (`i`, `v`) => `void` | - |
| `root.setData` | (`i`, `v`) => `void` | - |
| `root.setFloat64` | (`i`, `v`) => `void` | - |
| `root.setInt64` | (`i`, `v`) => `void` | - |
| `root.setText` | (`i`, `v`) => `void` | - |
| `root.setUint16` | (`i`, `v`) => `void` | - |
| `value` | [`DbValue`](#dbvalue) | Domain value. |

#### Returns

`void`

Nothing; the struct is mutated in place.

#### Throws

When `int64` is out of range, `float64` is not finite, or TEXT
contains U+0000.

## References

### DbType

Re-exports [DbType](abi.md#dbtype)
