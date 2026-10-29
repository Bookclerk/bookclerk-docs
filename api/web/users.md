# Users

Library users and their claim tickets.

Back to the [web API reference](README.md).

## `GET /api/users`

List first-party users (operator or administrator provisioner).

Includes presence extras: non-expired portal sessions (`online` /
`last_active_at`), durable `last_seen_at` (survives logout), unfinished
listening within the last 30 minutes, and linked storefront accounts.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The library's users. Body: [`UserListResponse`](schemas.md#userlistresponse) (`application/json`)
  - `users` *array of [`UserListEntry`](schemas.md#userlistentry)*, **required** — Every first-party user.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller may not do this. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/users' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/users",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/users", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/users")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The library's users.

```json
{
  "users": [
    {
      "id": 7,
      "role": "owner",
      "status": "active",
      "display_name": "Sam Reader",
      "login_name": "sam",
      "email": "sam@example.com",
      "has_password": true,
      "online": true,
      "last_active_at": "2026-10-28T09:15:10+00:00",
      "last_seen_at": "2026-10-28T09:15:10+00:00",
      "has_avatar": false,
      "avatar_source": "auto",
      "gravatar_hash": "c5e3d8f1a9b24e7b6f0c2a1d4e8b9f3a7c6d5e4f3a2b1c0d9e8f7a6b5c4d3e2f",
      "sso_pictures": [
        {
          "identity_id": 12,
          "provider": "oidc:github",
          "picture_url": "https://avatars.githubusercontent.com/u/8675309"
        }
      ],
      "listening": {
        "title": "The Left Hand of Darkness",
        "provider": "audiobookshelf",
        "last_listened_at": "2026-10-28T09:12:44Z"
      },
      "integrations": [
        {
          "source": "audible",
          "account_id": "sam@example.com"
        }
      ],
      "identities": [
        {
          "provider": "oidc:github",
          "external_user_id": "8675309"
        }
      ]
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "Body of `GET /api/users`.",
  "required": [
    "users"
  ],
  "properties": {
    "users": {
      "type": "array",
      "items": {
        "allOf": [
          {
            "type": "object",
            "description": "The user's profile, role and status.",
            "required": [
              "id",
              "role",
              "status",
              "has_password"
            ],
            "properties": {
              "id": {
                "type": "integer",
                "format": "int64",
                "description": "Surrogate primary key assigned by the database.",
                "example": 7
              },
              "role": {
                "type": "string",
                "description": "First-party role.",
                "enum": [
                  "owner",
                  "administrator",
                  "host",
                  "member"
                ]
              },
              "status": {
                "type": "string",
                "description": "Whether the account may sign in.",
                "enum": [
                  "active",
                  "disabled"
                ]
              },
              "display_name": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Human-readable name shown in the UI, when set.",
                "example": "Sam Reader"
              },
              "login_name": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Local username for password login, when set.",
                "example": "sam"
              },
              "email": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Contact email, when set.",
                "example": "sam@example.com"
              },
              "has_password": {
                "type": "boolean",
                "description": "Whether a local password is stored for the user.",
                "example": true
              }
            }
          },
          {
            "type": "object",
            "required": [
              "online",
              "has_avatar",
              "avatar_source",
              "sso_pictures",
              "integrations",
              "identities"
            ],
            "properties": {
              "online": {
                "type": "boolean",
                "description": "Whether the user has a portal session that has not expired.",
                "example": true
              },
              "last_active_at": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "RFC 3339 time of the most recent activity on an unexpired portal\nsession, when the user is online.",
                "example": "2026-10-28T09:15:10+00:00"
              },
              "last_seen_at": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "RFC 3339 time of the last authenticated portal activity; kept\nafter the session ends.",
                "example": "2026-10-28T09:15:10+00:00"
              },
              "has_avatar": {
                "type": "boolean",
                "description": "Whether an uploaded avatar is stored for the user.",
                "example": false
              },
              "avatar_source": {
                "type": "string",
                "description": "Which avatar the user shows: `auto`, `upload`, `gravatar`, or an\nidentity's picture.",
                "example": "auto"
              },
              "gravatar_hash": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "SHA-256 digest of the user's email for Gravatar, when they have one.",
                "example": "c5e3d8f1a9b24e7b6f0c2a1d4e8b9f3a7c6d5e4f3a2b1c0d9e8f7a6b5c4d3e2f"
              },
              "sso_pictures": {
                "type": "array",
                "items": {
                  "type": "object",
                  "description": "A profile picture an identity provider supplied for a user.",
                  "required": [
                    "identity_id",
                    "provider",
                    "picture_url"
                  ],
                  "properties": {
                    "identity_id": {
                      "type": "integer",
                      "format": "int64",
                      "description": "The linked identity the picture came from.",
                      "example": 12
                    },
                    "provider": {
                      "type": "string",
                      "description": "Identity-broker id (`oidc:google`, `oidc:github`, …).",
                      "example": "oidc:github"
                    },
                    "picture_url": {
                      "type": "string",
                      "description": "HTTPS URL of the picture at the provider or its CDN.",
                      "example": "https://avatars.githubusercontent.com/u/8675309"
                    },
                    "last_used_at": {
                      "type": [
                        "string",
                        "null"
                      ],
                      "description": "RFC 3339 time the identity last signed in, when it has.",
                      "example": "2026-10-27T18:40:02+00:00"
                    }
                  }
                },
                "description": "Pictures the user's identity providers supplied."
              },
              "listening": {
                "oneOf": [
                  {
                    "type": "object",
                    "description": "The most recent unfinished listen within the last 30 minutes, if any.",
                    "required": [
                      "provider",
                      "last_listened_at"
                    ],
                    "properties": {
                      "title": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "Display title when the provider reported one.",
                        "example": "The Left Hand of Darkness"
                      },
                      "provider": {
                        "type": "string",
                        "description": "Integration / storefront provider id.",
                        "example": "audiobookshelf"
                      },
                      "last_listened_at": {
                        "type": "string",
                        "format": "date-time",
                        "description": "RFC 3339 time of the last playback update.",
                        "example": "2026-10-28T09:12:44Z"
                      }
                    }
                  },
                  {
                    "type": "null"
                  }
                ]
              },
              "integrations": {
                "type": "array",
                "items": {
                  "type": "object",
                  "description": "Storefront / integration connection summary for admin user lists.",
                  "required": [
                    "source",
                    "account_id"
                  ],
                  "properties": {
                    "source": {
                      "type": "string",
                      "description": "Content-source or integration plugin id (`audible`, `libro`, …).",
                      "example": "audible"
                    },
                    "account_id": {
                      "type": "string",
                      "description": "Store or operator account id this row belongs to.",
                      "example": "sam@example.com"
                    },
                    "label": {
                      "type": [
                        "string",
                        "null"
                      ],
                      "description": "Optional operator-facing account label.",
                      "example": "Sam's Audible"
                    }
                  }
                },
                "description": "Storefront and integration accounts linked to the user."
              },
              "identities": {
                "type": "array",
                "items": {
                  "type": "object",
                  "description": "An external sign-in identity linked to a user.",
                  "required": [
                    "provider",
                    "external_user_id"
                  ],
                  "properties": {
                    "provider": {
                      "type": "string",
                      "description": "Identity-broker id (`oidc:google`, `oidc:github`, …).",
                      "example": "oidc:github"
                    },
                    "external_user_id": {
                      "type": "string",
                      "description": "The user's id at the provider.",
                      "example": "8675309"
                    },
                    "label": {
                      "type": [
                        "string",
                        "null"
                      ],
                      "description": "Operator-facing label for the identity, when one was set.",
                      "example": "sam-reader"
                    }
                  }
                },
                "description": "External sign-in identities linked to the user."
              }
            }
          }
        ],
        "description": "One user in the list `GET /api/users` returns: the summary plus\npresence, avatar and connection details for administration."
      },
      "description": "Every first-party user."
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `403`: The caller may not do this.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
string
```

Schema:

```json
{
  "type": "string"
}
```

## `POST /api/users`

Owner/admin or operator creates a user and optional invite magic link.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`CreateUserRequest`](schemas.md#createuserrequest)

- `role` *string or null* — Role to assign when permitted (`member` when omitted).
- `display_name` *string or null* — Optional display name for the new user.
- `login_name` *string or null* — Optional local login name for the new user.
- `email` *string or null* — Optional email for the new user.
- `password` *string or null* — Optional password; hashed before persist. Empty or omitted leaves the user passwordless.
- `mint_invite` *boolean* — When true (default), also mint an invite/claim ticket.

### Responses

- `200` — The new user, with an invite link when one was asked for. Body: [`UserCreatedResponse`](schemas.md#usercreatedresponse) (`application/json`)
  - `ok` *boolean*, **required** — Always true; the status code says the user was created.
  - `user` *[`UserSummary`](schemas.md#usersummary)*, **required** — The user as created.
  - `claim_ticket` *string or null* — Opaque single-use claim ticket, when `mint_invite` was true.
  - `invite_url` *string or null* — Invite link that redeems the claim ticket, when one was minted.
- `400` — The request is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — The login is taken. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/users' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/users",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/users", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/users")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The new user, with an invite link when one was asked for.

```json
{
  "ok": true,
  "user": {
    "id": 7,
    "role": "owner",
    "status": "active",
    "display_name": "Sam Reader",
    "login_name": "sam",
    "email": "sam@example.com",
    "has_password": true
  },
  "claim_ticket": "7Qf3kZp0wL2mN9xT1cRb8vHd",
  "invite_url": "https://books.example.com/invite?ticket=7Qf3kZp0wL2mN9xT1cRb8vHd"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Body of `POST /api/users`: the new user and, when one was asked for,\nthe invite that lets them claim it.",
  "required": [
    "ok",
    "user"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always true; the status code says the user was created.",
      "example": true
    },
    "user": {
      "type": "object",
      "description": "The user as created.",
      "required": [
        "id",
        "role",
        "status",
        "has_password"
      ],
      "properties": {
        "id": {
          "type": "integer",
          "format": "int64",
          "description": "Surrogate primary key assigned by the database.",
          "example": 7
        },
        "role": {
          "type": "string",
          "description": "First-party role.",
          "enum": [
            "owner",
            "administrator",
            "host",
            "member"
          ]
        },
        "status": {
          "type": "string",
          "description": "Whether the account may sign in.",
          "enum": [
            "active",
            "disabled"
          ]
        },
        "display_name": {
          "type": [
            "string",
            "null"
          ],
          "description": "Human-readable name shown in the UI, when set.",
          "example": "Sam Reader"
        },
        "login_name": {
          "type": [
            "string",
            "null"
          ],
          "description": "Local username for password login, when set.",
          "example": "sam"
        },
        "email": {
          "type": [
            "string",
            "null"
          ],
          "description": "Contact email, when set.",
          "example": "sam@example.com"
        },
        "has_password": {
          "type": "boolean",
          "description": "Whether a local password is stored for the user.",
          "example": true
        }
      }
    },
    "claim_ticket": {
      "type": [
        "string",
        "null"
      ],
      "description": "Opaque single-use claim ticket, when `mint_invite` was true.",
      "example": "7Qf3kZp0wL2mN9xT1cRb8vHd"
    },
    "invite_url": {
      "type": [
        "string",
        "null"
      ],
      "description": "Invite link that redeems the claim ticket, when one was minted.",
      "example": "https://books.example.com/invite?ticket=7Qf3kZp0wL2mN9xT1cRb8vHd"
    }
  }
}
```

Status `400`: The request is invalid.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `409`: The login is taken.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
string
```

Schema:

```json
{
  "type": "string"
}
```

## `DELETE /api/users/{id}`

Delete a user and their personal data (wishlist / links / sessions); keep acquired books.

Operators/administrators may delete any user. Portal callers may delete
only their own account (self-service).

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *integer (int64)*, **required** — User id.

### Responses

- `200` — The user and their personal data were deleted. Body: [`UserDeletedResponse`](schemas.md#userdeletedresponse) (`application/json`)
  - `ok` *boolean*, **required** — Always true; the status code says the user was deleted.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such user. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — The user is the last Owner or Administrator. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users/<id>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/users/<id>' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/users/<id>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/users/<id>", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/users/<id>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The user and their personal data were deleted.

```json
{
  "ok": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "Body of `DELETE /api/users/{id}`.",
  "required": [
    "ok"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always true; the status code says the user was deleted.",
      "example": true
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `404`: No such user.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `409`: The user is the last Owner or Administrator.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
string
```

Schema:

```json
{
  "type": "string"
}
```

## `PATCH /api/users/{id}`

Patch role/status/display/login for a first-party user (provisioner).

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *integer (int64)*, **required** — User id.

### Request body

`application/json`: [`PatchUserRequest`](schemas.md#patchuserrequest)

- `role` *string or null* — Replacement role (`owner` / `administrator` / `member`) when present.
- `status` *string or null* — Replacement status (`active` / `disabled`) when present.
- `display_name` *string or null* — Replacement display name when present.
- `login_name` *string or null* — Replacement local login name when present.
- `email` *string or null* — Replacement email when present.

### Responses

- `200` — The user after the change. Body: [`UserChangeResponse`](schemas.md#userchangeresponse) (`application/json`)
  - `ok` *boolean*, **required** — Always true; the status code says the change was applied.
  - `user` *[`UserSummary`](schemas.md#usersummary)*, **required** — The user after the change.
- `400` — A value is invalid. An invalid email names itself; any other invalid value gets the standard error body. Body: [`UserChangeInvalid`](schemas.md#userchangeinvalid) (`application/json`)
  - `error` *string*, **required** — Machine slug for the rejected value.
  - `message` *string*, **required** — A sentence to show the person who made the request.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such user. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — The change would leave the library without an owner or administrator (`error` names which), or the login name or email is taken by another user (the standard error body). Body: [`UserChangeConflict`](schemas.md#userchangeconflict) (`application/json`)
  - `error` *string*, **required** — Why the change was refused: `last_owner` or `last_administrator`.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users/<id>' \
    -X PATCH \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/users/<id>' `
    -Method Patch `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.patch(
    "http://127.0.0.1:8787/api/users/<id>",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/users/<id>", {
  method: "PATCH",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .patch("http://127.0.0.1:8787/api/users/<id>")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The user after the change.

```json
{
  "ok": true,
  "user": {
    "id": 7,
    "role": "owner",
    "status": "active",
    "display_name": "Sam Reader",
    "login_name": "sam",
    "email": "sam@example.com",
    "has_password": true
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "Body of a user change that succeeded: the user as stored afterwards.",
  "required": [
    "ok",
    "user"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always true; the status code says the change was applied.",
      "example": true
    },
    "user": {
      "type": "object",
      "description": "The user after the change.",
      "required": [
        "id",
        "role",
        "status",
        "has_password"
      ],
      "properties": {
        "id": {
          "type": "integer",
          "format": "int64",
          "description": "Surrogate primary key assigned by the database.",
          "example": 7
        },
        "role": {
          "type": "string",
          "description": "First-party role.",
          "enum": [
            "owner",
            "administrator",
            "host",
            "member"
          ]
        },
        "status": {
          "type": "string",
          "description": "Whether the account may sign in.",
          "enum": [
            "active",
            "disabled"
          ]
        },
        "display_name": {
          "type": [
            "string",
            "null"
          ],
          "description": "Human-readable name shown in the UI, when set.",
          "example": "Sam Reader"
        },
        "login_name": {
          "type": [
            "string",
            "null"
          ],
          "description": "Local username for password login, when set.",
          "example": "sam"
        },
        "email": {
          "type": [
            "string",
            "null"
          ],
          "description": "Contact email, when set.",
          "example": "sam@example.com"
        },
        "has_password": {
          "type": "boolean",
          "description": "Whether a local password is stored for the user.",
          "example": true
        }
      }
    }
  }
}
```

Status `400`: A value is invalid. An invalid email names itself; any other invalid value gets the standard error body.

```json
{
  "error": "invalid_email",
  "message": "Enter a valid email address."
}
```

Schema:

```json
{
  "type": "object",
  "description": "Body of a `400 Bad Request` from a user change with an invalid email.",
  "required": [
    "error",
    "message"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug for the rejected value.",
      "example": "invalid_email"
    },
    "message": {
      "type": "string",
      "description": "A sentence to show the person who made the request.",
      "example": "Enter a valid email address."
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `404`: No such user.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `409`: The change would leave the library without an owner or administrator (`error` names which), or the login name or email is taken by another user (the standard error body).

```json
{
  "error": "last_owner"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Body of a `409 Conflict` from a user change that would leave the\nlibrary without an owner or administrator.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Why the change was refused: `last_owner` or `last_administrator`.",
      "example": "last_owner"
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
string
```

Schema:

```json
{
  "type": "string"
}
```

## `POST /api/users/{id}/claim-ticket`

Mint a fresh local claim ticket for an active first-party user.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *integer (int64)*, **required** — User id.

### Responses

- `200` — A fresh claim ticket. Body: [`UserClaimTicketResponse`](schemas.md#userclaimticketresponse) (`application/json`)
  - `ok` *boolean*, **required** — Always true; the status code says the ticket was minted.
  - `claim_ticket` *string*, **required** — Opaque single-use claim ticket for the user.
  - `invite_url` *string*, **required** — Invite link that redeems the claim ticket.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such user. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users/<id>/claim-ticket' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/users/<id>/claim-ticket' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/users/<id>/claim-ticket",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/users/<id>/claim-ticket", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/users/<id>/claim-ticket")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: A fresh claim ticket.

```json
{
  "ok": true,
  "claim_ticket": "7Qf3kZp0wL2mN9xT1cRb8vHd",
  "invite_url": "https://books.example.com/invite?ticket=7Qf3kZp0wL2mN9xT1cRb8vHd"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Body of `POST /api/users/{id}/claim-ticket`.",
  "required": [
    "ok",
    "claim_ticket",
    "invite_url"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always true; the status code says the ticket was minted.",
      "example": true
    },
    "claim_ticket": {
      "type": "string",
      "description": "Opaque single-use claim ticket for the user.",
      "example": "7Qf3kZp0wL2mN9xT1cRb8vHd"
    },
    "invite_url": {
      "type": "string",
      "description": "Invite link that redeems the claim ticket.",
      "example": "https://books.example.com/invite?ticket=7Qf3kZp0wL2mN9xT1cRb8vHd"
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `404`: No such user.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
string
```

Schema:

```json
{
  "type": "string"
}
```

## `POST /api/users/{id}/reset-password`

Invalidate a user's password, revoke sessions, and mint a claim ticket for reset.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *integer (int64)*, **required** — User id.

### Responses

- `200` — The password was cleared; a claim ticket for the reset. Body: [`UserPasswordResetResponse`](schemas.md#userpasswordresetresponse) (`application/json`)
  - `ok` *boolean*, **required** — Always true; the status code says the password was cleared.
  - `claim_ticket` *string*, **required** — Opaque single-use claim ticket that lets the user set a new password.
  - `revoked_sessions` *integer (int64)*, **required** — How many of the user's portal sessions were signed out. At least 0.
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such user. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users/<id>/reset-password' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/users/<id>/reset-password' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/users/<id>/reset-password",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/users/<id>/reset-password", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/users/<id>/reset-password")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The password was cleared; a claim ticket for the reset.

```json
{
  "ok": true,
  "claim_ticket": "7Qf3kZp0wL2mN9xT1cRb8vHd",
  "revoked_sessions": 2
}
```

Schema:

```json
{
  "type": "object",
  "description": "Body of `POST /api/users/{id}/reset-password`.",
  "required": [
    "ok",
    "claim_ticket",
    "revoked_sessions"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always true; the status code says the password was cleared.",
      "example": true
    },
    "claim_ticket": {
      "type": "string",
      "description": "Opaque single-use claim ticket that lets the user set a new password.",
      "example": "7Qf3kZp0wL2mN9xT1cRb8vHd"
    },
    "revoked_sessions": {
      "type": "integer",
      "format": "int64",
      "description": "How many of the user's portal sessions were signed out.",
      "example": 2,
      "minimum": 0
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `404`: No such user.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
string
```

Schema:

```json
{
  "type": "string"
}
```

## `GET /api/users/{id}/avatar`

Serves a stored avatar JPEG for any authenticated household member.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *integer (int64)*, **required** — User id.

### Responses

- `200` — The avatar. Body: [`BinaryBody`](schemas.md#binarybody) (`image/jpeg`); [`BinaryBody`](schemas.md#binarybody) (`image/png`); [`BinaryBody`](schemas.md#binarybody) (`image/webp`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — The user has no avatar. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users/<id>/avatar' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/users/<id>/avatar' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/users/<id>/avatar",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/users/<id>/avatar", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/users/<id>/avatar")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The avatar. The body is binary (`image/jpeg`).

Schema:

```json
{
  "type": "string",
  "format": "binary",
  "description": "A request body of raw bytes (a file upload), documented as\n`type: string, format: binary` so generated clients send the bytes\nrather than a JSON array."
}
```

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `404`: The user has no avatar.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
string
```

Schema:

```json
{
  "type": "string"
}
```
