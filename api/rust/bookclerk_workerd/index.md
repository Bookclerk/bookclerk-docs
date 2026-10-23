# bookclerk_workerd

Library surface for the pinned Cloudflare `workerd` helper and egress policy.

# Audience

The `bookclerk-workerd` launcher binary and host packaging (`cargo ensure-workerd`).
Guest authors do not link this crate; they run inside the isolate it configures.

See `docs/plugins.md` for jail / workerd isolation requirements.

## Modules

### [`bookclerk_workerd`](bookclerk_workerd.md)

*13 modules*

### [`bridge_http`](bridge_http.md)

*1 function, 2 structs*

### [`bridge_stdio`](bridge_stdio.md)

*1 enum, 1 function*

### [`config`](config.md)

*12 functions, 2 structs, 3 enums, 8 constants*

### [`egress`](egress.md)

*1 struct*

### [`ensure`](ensure.md)

*2 functions*

### [`grant`](grant.md)

*1 struct, 5 constants*

### [`granted`](granted.md)

*1 function, 1 struct, 1 trait, 1 type alias, 3 constants*

### [`granted_http`](granted_http.md)

*4 constants, 4 structs, 6 enums, 8 functions*

### [`inherited_link`](inherited_link.md)

*1 enum*

### [`invoke`](invoke.md)

*1 constant, 1 enum, 1 function, 2 structs*

### [`pin`](pin.md)

*1 struct, 3 constants, 4 functions*

### [`socket_proxy`](socket_proxy.md)

*1 struct, 3 constants, 9 functions*

### [`unix_bind`](unix_bind.md)

*2 constants, 6 functions*

