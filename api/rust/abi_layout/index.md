# abi_layout

Emits `schema/plugin.layout.json` from the Cap'n Proto compiler.

The SDK code generator (`scripts/sdk_emitters.py`) needs struct data-word
counts, pointer counts, per-field offsets, and union discriminants to emit
wire codecs. Those numbers are owned by the compiler's layout algorithm,
so this tool runs `capnp compile -o-` on the hand-written schemas and
projects the resulting `CodeGeneratorRequest` into a stable JSON document
that the Python emitter consumes. Nothing here is hand-derived.

Usage: `cargo run -q -p abi-layout -- [--schema-dir DIR] [--out FILE]`.
With no `--out` the JSON is written to stdout so drift checks can compare
without touching the tree.

## Modules

