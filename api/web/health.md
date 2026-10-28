# Health

Liveness.

Back to the [web API reference](README.md).

## `GET /health`

Serves `GET /health` with a static `ok` payload for liveness probes.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

- `200` — The daemon is serving HTTP. Body: [`HealthResponse`](schemas.md#healthresponse) (`application/json`)
  - `status` *string*, **required** — Always `ok` when the process can serve HTTP.

### Example request

```console
$ curl 'http://127.0.0.1:8787/health'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/health'
```

```python
import requests

response = requests.get("http://127.0.0.1:8787/health")
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/health");
console.log(response.status, await response.text());
```

```rust
let response = reqwest::get("http://127.0.0.1:8787/health").await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The daemon is serving HTTP.

```json
{
  "status": "string"
}
```
