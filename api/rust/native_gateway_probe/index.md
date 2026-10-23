# native_gateway_probe

Test-only native guest for the `native_gateway` smoke (never staged or shipped).

Exports one `probe` CLI command. `op=connect` dials through the SDK socket
proxy (`bookclerk_plugin_sdk::net::connect`) and echoes `payload`;
`op=ambient` tries a direct `std::net` TCP connect, which the nested
deny-network jail must refuse. Results are one JSON object on stdout.

## Modules

