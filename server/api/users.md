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
      "controls": {
        "link_accounts": false,
        "download": true,
        "edit_metadata": true,
        "upload": true,
        "request_books": true,
        "see_prices": true
      },
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
              "controls",
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
                    "picture_path": {
                      "type": [
                        "string",
                        "null"
                      ],
                      "description": "Same-origin 128 px copy of `picture_url` for the web app to show,\nwhen this node can sign it.",
                      "example": "/api/images/aHR0cHM6Ly9hdmF0YXJzLmdpdGh1YnVzZXJjb250ZW50LmNvbS91Lzg2NzUzMDk.N14rqh66Bok9-eCqJKBNoQ"
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
                        "description": "Title of the work being listened to, when the library knows it.",
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
              "controls": {
                "type": "object",
                "description": "What the user may do here (the member controls that apply to them;\nsee `GET /api/users/{id}/controls` for the default and overrides).",
                "properties": {
                  "link_accounts": {
                    "type": "boolean",
                    "description": "May connect store accounts ([`MemberControl::LinkAccounts`]).",
                    "example": false
                  },
                  "download": {
                    "type": "boolean",
                    "description": "May take copies of the library's files ([`MemberControl::Download`]).",
                    "example": true
                  },
                  "edit_metadata": {
                    "type": "boolean",
                    "description": "May change metadata ([`MemberControl::EditMetadata`]).",
                    "example": true
                  },
                  "upload": {
                    "type": "boolean",
                    "description": "May upload books ([`MemberControl::Upload`]).",
                    "example": true
                  },
                  "request_books": {
                    "type": "boolean",
                    "description": "May ask branches for books ([`MemberControl::RequestBooks`]).",
                    "example": true
                  },
                  "see_prices": {
                    "type": "boolean",
                    "description": "May see prices and store links ([`MemberControl::SeePrices`]).",
                    "example": true
                  }
                },
                "additionalProperties": false
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
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The caller may not do this.

```json
{
  "error": "forbidden",
  "message": "You do not have access to this resource.",
  "status": 403
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

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
- `400` — The request is invalid. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ApiError`](schemas.md#apierror) (`application/json`); string (`text/plain`)
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
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
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

Status `400`: The request is invalid. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "bad_request",
  "message": "The request could not be understood.",
  "status": 400
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "forbidden",
  "message": "You do not have access to this resource.",
  "status": 403
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `409`: The login is taken.

```json
{
  "error": "conflict",
  "message": "Conflict",
  "status": 409
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: role: invalid type: integer `1`, expected a string at line 1 column 9
```

Schema:

A `text/plain` body: a string.

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

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
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "forbidden",
  "message": "You do not have access to this resource.",
  "status": 403
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `404`: No such user.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `409`: The user is the last Owner or Administrator.

```json
{
  "error": "conflict",
  "message": "Conflict",
  "status": 409
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `PATCH /api/users/{id}`

Patch role/status/display/login/email for a first-party user
(provisioner).

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
- `400` — A value is invalid. An invalid email names itself; any other invalid value gets the standard error body. As `text/plain`, when the body is not syntactically valid JSON. Body: [`UserChangeInvalid`](schemas.md#userchangeinvalid) (`application/json`); string (`text/plain`)
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
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
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

Status `400`: A value is invalid. An invalid email names itself; any other invalid value gets the standard error body. As `text/plain`, when the body is not syntactically valid JSON.

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
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "forbidden",
  "message": "You do not have access to this resource.",
  "status": 403
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `404`: No such user.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: role: invalid type: integer `1`, expected a string at line 1 column 9
```

Schema:

A `text/plain` body: a string.

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/users/default-scope`

Reads the library's default scope, which patrons without their own
follow.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The default scope. Body: [`DefaultScopeJson`](schemas.md#defaultscopejson) (`application/json`)
  - `scope` *[`PatronScope`](schemas.md#patronscope)*, **required** — The scope patrons without their own follow.
- `401` — The caller is not signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Owner or Administrator. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The scope could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users/default-scope' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/users/default-scope' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/users/default-scope",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/users/default-scope", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/users/default-scope")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The default scope.

```json
{
  "scope": {
    "rules": [
      {
        "field": "genre",
        "value": "Kids",
        "effect": "include"
      }
    ]
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "The library's default scope as the scope routes report it.",
  "required": [
    "scope"
  ],
  "properties": {
    "scope": {
      "type": "object",
      "description": "The scope patrons without their own follow.",
      "properties": {
        "rules": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "One rule of a scope: a field, one value of it, and its effect.",
            "required": [
              "field",
              "value",
              "effect"
            ],
            "properties": {
              "field": {
                "type": "string",
                "description": "The field the rule matches.",
                "enum": [
                  "genre",
                  "series",
                  "author",
                  "narrator"
                ]
              },
              "value": {
                "type": "string",
                "description": "The value, compared without case and surrounding space (1 to 256\nbytes).",
                "example": "Kids"
              },
              "effect": {
                "type": "string",
                "description": "Include or exclude.",
                "enum": [
                  "include",
                  "exclude"
                ]
              }
            },
            "additionalProperties": false
          },
          "description": "Visibility rules, at most 16 and 1,024 bytes as JSON. Empty means\nevery book is visible.",
          "maxItems": 16
        }
      },
      "additionalProperties": false
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "unauthorized"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `403`: The caller is not an Owner or Administrator.

```json
{
  "error": "forbidden"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `500`: The scope could not be read.

```json
{
  "error": "the patron's scope or controls could not be read or written"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `PUT /api/users/default-scope`

Replaces the library's default scope. Every patron without their own
scope follows the change at once.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PatronScope`](schemas.md#patronscope)

- `rules` *array of [`ScopeRule`](schemas.md#scoperule)* — Visibility rules, at most 16 and 1,024 bytes as JSON. Empty means every book is visible. At most 16 items.

### Responses

- `200` — The default scope after the change. Body: [`DefaultScopeJson`](schemas.md#defaultscopejson) (`application/json`)
  - `scope` *[`PatronScope`](schemas.md#patronscope)*, **required** — The scope patrons without their own follow.
- `400` — The scope is invalid. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Owner or Administrator. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The default kept changing underneath the write; retry. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The scope could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users/default-scope' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/users/default-scope' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/users/default-scope",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/users/default-scope", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/users/default-scope")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The default scope after the change.

```json
{
  "scope": {
    "rules": [
      {
        "field": "genre",
        "value": "Kids",
        "effect": "include"
      }
    ]
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "The library's default scope as the scope routes report it.",
  "required": [
    "scope"
  ],
  "properties": {
    "scope": {
      "type": "object",
      "description": "The scope patrons without their own follow.",
      "properties": {
        "rules": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "One rule of a scope: a field, one value of it, and its effect.",
            "required": [
              "field",
              "value",
              "effect"
            ],
            "properties": {
              "field": {
                "type": "string",
                "description": "The field the rule matches.",
                "enum": [
                  "genre",
                  "series",
                  "author",
                  "narrator"
                ]
              },
              "value": {
                "type": "string",
                "description": "The value, compared without case and surrounding space (1 to 256\nbytes).",
                "example": "Kids"
              },
              "effect": {
                "type": "string",
                "description": "Include or exclude.",
                "enum": [
                  "include",
                  "exclude"
                ]
              }
            },
            "additionalProperties": false
          },
          "description": "Visibility rules, at most 16 and 1,024 bytes as JSON. Empty means\nevery book is visible.",
          "maxItems": 16
        }
      },
      "additionalProperties": false
    }
  }
}
```

Status `400`: The scope is invalid. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "invalid patron scope: a scope holds at most 64 rules"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "unauthorized"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `403`: The caller is not an Owner or Administrator. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "forbidden"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `409`: The default kept changing underneath the write; retry.

```json
{
  "error": "core.patron_scope kept changing; retry"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: rules: invalid type: integer `1`, expected a sequence at line 1 column 10
```

Schema:

A `text/plain` body: a string.

Status `500`: The scope could not be written.

```json
{
  "error": "the patron's scope or controls could not be read or written"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/users/{id}/scope`

Reads which part of the library a patron sees.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *integer (int64)*, **required** — User id.

### Responses

- `200` — The patron's scope. Body: [`UserScopeJson`](schemas.md#userscopejson) (`application/json`)
  - `user_id` *integer (int64)*, **required** — The user (`users.id`).
  - `scope` *[`PatronScope`](schemas.md#patronscope)*, **required** — The scope that applies to them: their own, or the library's default.
  - `own` *boolean*, **required** — True when an Administrator gave them this scope; false when they follow the library's default.
  - `applies` *boolean*, **required** — Whether the scope applies to the user's role: false for an Owner or Administrator, who always see the whole library.
- `401` — The caller is not signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller may not manage this user. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `404` — No such user. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The scope could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users/<id>/scope' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/users/<id>/scope' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/users/<id>/scope",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/users/<id>/scope", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/users/<id>/scope")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The patron's scope.

```json
{
  "user_id": 7,
  "scope": {
    "rules": [
      {
        "field": "genre",
        "value": "Kids",
        "effect": "include"
      }
    ]
  },
  "own": false,
  "applies": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "A patron's scope as the scope routes report it.",
  "required": [
    "user_id",
    "scope",
    "own",
    "applies"
  ],
  "properties": {
    "user_id": {
      "type": "integer",
      "format": "int64",
      "description": "The user (`users.id`).",
      "example": 7
    },
    "scope": {
      "type": "object",
      "description": "The scope that applies to them: their own, or the library's default.",
      "properties": {
        "rules": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "One rule of a scope: a field, one value of it, and its effect.",
            "required": [
              "field",
              "value",
              "effect"
            ],
            "properties": {
              "field": {
                "type": "string",
                "description": "The field the rule matches.",
                "enum": [
                  "genre",
                  "series",
                  "author",
                  "narrator"
                ]
              },
              "value": {
                "type": "string",
                "description": "The value, compared without case and surrounding space (1 to 256\nbytes).",
                "example": "Kids"
              },
              "effect": {
                "type": "string",
                "description": "Include or exclude.",
                "enum": [
                  "include",
                  "exclude"
                ]
              }
            },
            "additionalProperties": false
          },
          "description": "Visibility rules, at most 16 and 1,024 bytes as JSON. Empty means\nevery book is visible.",
          "maxItems": 16
        }
      },
      "additionalProperties": false
    },
    "own": {
      "type": "boolean",
      "description": "True when an Administrator gave them this scope; false when they\nfollow the library's default.",
      "example": false
    },
    "applies": {
      "type": "boolean",
      "description": "Whether the scope applies to the user's role: false for an Owner or\nAdministrator, who always see the whole library.",
      "example": true
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "unauthorized"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `403`: The caller may not manage this user.

```json
{
  "error": "you may not manage this user"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `404`: No such user.

```json
{
  "error": "user 7 not found"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `500`: The scope could not be read.

```json
{
  "error": "the patron's scope or controls could not be read or written"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `PUT /api/users/{id}/scope`

Gives a patron their own scope, or returns them to the library's
default with `"scope": null`.

Nobody sets their own scope. Owners and Administrators always see the
whole library, so a scope stored for one only applies if their role
changes to Member.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *integer (int64)*, **required** — User id.

### Request body

`application/json`: [`PutUserScope`](schemas.md#putuserscope)

- `scope` *[`PatronScope`](schemas.md#patronscope) or null* — The patron's own scope, or `null` to return them to the library's default.

### Responses

- `200` — The patron's scope after the change. Body: [`UserScopeJson`](schemas.md#userscopejson) (`application/json`)
  - `user_id` *integer (int64)*, **required** — The user (`users.id`).
  - `scope` *[`PatronScope`](schemas.md#patronscope)*, **required** — The scope that applies to them: their own, or the library's default.
  - `own` *boolean*, **required** — True when an Administrator gave them this scope; false when they follow the library's default.
  - `applies` *boolean*, **required** — Whether the scope applies to the user's role: false for an Owner or Administrator, who always see the whole library.
- `400` — The scope is invalid. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller may not manage this user, or tried to set their own scope. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `404` — No such user. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The scope could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users/<id>/scope' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/users/<id>/scope' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/users/<id>/scope",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/users/<id>/scope", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/users/<id>/scope")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The patron's scope after the change.

```json
{
  "user_id": 7,
  "scope": {
    "rules": [
      {
        "field": "genre",
        "value": "Kids",
        "effect": "include"
      }
    ]
  },
  "own": false,
  "applies": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "A patron's scope as the scope routes report it.",
  "required": [
    "user_id",
    "scope",
    "own",
    "applies"
  ],
  "properties": {
    "user_id": {
      "type": "integer",
      "format": "int64",
      "description": "The user (`users.id`).",
      "example": 7
    },
    "scope": {
      "type": "object",
      "description": "The scope that applies to them: their own, or the library's default.",
      "properties": {
        "rules": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "One rule of a scope: a field, one value of it, and its effect.",
            "required": [
              "field",
              "value",
              "effect"
            ],
            "properties": {
              "field": {
                "type": "string",
                "description": "The field the rule matches.",
                "enum": [
                  "genre",
                  "series",
                  "author",
                  "narrator"
                ]
              },
              "value": {
                "type": "string",
                "description": "The value, compared without case and surrounding space (1 to 256\nbytes).",
                "example": "Kids"
              },
              "effect": {
                "type": "string",
                "description": "Include or exclude.",
                "enum": [
                  "include",
                  "exclude"
                ]
              }
            },
            "additionalProperties": false
          },
          "description": "Visibility rules, at most 16 and 1,024 bytes as JSON. Empty means\nevery book is visible.",
          "maxItems": 16
        }
      },
      "additionalProperties": false
    },
    "own": {
      "type": "boolean",
      "description": "True when an Administrator gave them this scope; false when they\nfollow the library's default.",
      "example": false
    },
    "applies": {
      "type": "boolean",
      "description": "Whether the scope applies to the user's role: false for an Owner or\nAdministrator, who always see the whole library.",
      "example": true
    }
  }
}
```

Status `400`: The scope is invalid. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "invalid patron scope: a scope rule needs a value"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "unauthorized"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `403`: The caller may not manage this user, or tried to set their own scope. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "you may not set your own scope"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `404`: No such user.

```json
{
  "error": "user 7 not found"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: invalid type: null, expected struct PutUserScope at line 1 column 4
```

Schema:

A `text/plain` body: a string.

Status `500`: The scope could not be written.

```json
{
  "error": "the patron's scope or controls could not be read or written"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/users/default-controls`

Reads the library's default member controls, which every Member
follows unless a control is set for them alone.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The default controls. Body: [`DefaultControlsJson`](schemas.md#defaultcontrolsjson) (`application/json`)
  - `controls` *[`MemberControls`](schemas.md#membercontrols)*, **required** — The controls every Member follows unless set for them alone. Administrators do not follow it.
- `401` — The caller is not signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Owner or Administrator. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The controls could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users/default-controls' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/users/default-controls' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/users/default-controls",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/users/default-controls", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/users/default-controls")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The default controls.

```json
{
  "controls": {
    "link_accounts": false,
    "download": true,
    "edit_metadata": true,
    "upload": true,
    "request_books": true,
    "see_prices": true
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "The library's default member controls as the control routes report\nthem.",
  "required": [
    "controls"
  ],
  "properties": {
    "controls": {
      "type": "object",
      "description": "The controls every Member follows unless set for them alone.\nAdministrators do not follow it.",
      "properties": {
        "link_accounts": {
          "type": "boolean",
          "description": "May connect store accounts ([`MemberControl::LinkAccounts`]).",
          "example": false
        },
        "download": {
          "type": "boolean",
          "description": "May take copies of the library's files ([`MemberControl::Download`]).",
          "example": true
        },
        "edit_metadata": {
          "type": "boolean",
          "description": "May change metadata ([`MemberControl::EditMetadata`]).",
          "example": true
        },
        "upload": {
          "type": "boolean",
          "description": "May upload books ([`MemberControl::Upload`]).",
          "example": true
        },
        "request_books": {
          "type": "boolean",
          "description": "May ask branches for books ([`MemberControl::RequestBooks`]).",
          "example": true
        },
        "see_prices": {
          "type": "boolean",
          "description": "May see prices and store links ([`MemberControl::SeePrices`]).",
          "example": true
        }
      },
      "additionalProperties": false
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "unauthorized"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `403`: The caller is not an Owner or Administrator.

```json
{
  "error": "forbidden"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `500`: The controls could not be read.

```json
{
  "error": "the patron's scope or controls could not be read or written"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `PUT /api/users/default-controls`

Replaces the library's default member controls. Every Member follows
the change at once, except in the controls set for them alone;
Administrators do not follow the default.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`MemberControls`](schemas.md#membercontrols)

- `link_accounts` *boolean* — May connect store accounts ([`MemberControl::LinkAccounts`]).
- `download` *boolean* — May take copies of the library's files ([`MemberControl::Download`]).
- `edit_metadata` *boolean* — May change metadata ([`MemberControl::EditMetadata`]).
- `upload` *boolean* — May upload books ([`MemberControl::Upload`]).
- `request_books` *boolean* — May ask branches for books ([`MemberControl::RequestBooks`]).
- `see_prices` *boolean* — May see prices and store links ([`MemberControl::SeePrices`]).

### Responses

- `200` — The default controls after the change. Body: [`DefaultControlsJson`](schemas.md#defaultcontrolsjson) (`application/json`)
  - `controls` *[`MemberControls`](schemas.md#membercontrols)*, **required** — The controls every Member follows unless set for them alone. Administrators do not follow it.
- `400` — The body names a control that does not exist. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Owner or Administrator. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The default kept changing underneath the write; retry. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The controls could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users/default-controls' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/users/default-controls' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/users/default-controls",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/users/default-controls", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/users/default-controls")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The default controls after the change.

```json
{
  "controls": {
    "link_accounts": false,
    "download": true,
    "edit_metadata": true,
    "upload": true,
    "request_books": true,
    "see_prices": true
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "The library's default member controls as the control routes report\nthem.",
  "required": [
    "controls"
  ],
  "properties": {
    "controls": {
      "type": "object",
      "description": "The controls every Member follows unless set for them alone.\nAdministrators do not follow it.",
      "properties": {
        "link_accounts": {
          "type": "boolean",
          "description": "May connect store accounts ([`MemberControl::LinkAccounts`]).",
          "example": false
        },
        "download": {
          "type": "boolean",
          "description": "May take copies of the library's files ([`MemberControl::Download`]).",
          "example": true
        },
        "edit_metadata": {
          "type": "boolean",
          "description": "May change metadata ([`MemberControl::EditMetadata`]).",
          "example": true
        },
        "upload": {
          "type": "boolean",
          "description": "May upload books ([`MemberControl::Upload`]).",
          "example": true
        },
        "request_books": {
          "type": "boolean",
          "description": "May ask branches for books ([`MemberControl::RequestBooks`]).",
          "example": true
        },
        "see_prices": {
          "type": "boolean",
          "description": "May see prices and store links ([`MemberControl::SeePrices`]).",
          "example": true
        }
      },
      "additionalProperties": false
    }
  }
}
```

Status `400`: The body names a control that does not exist. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "unknown field `fly`, expected one of `link_accounts`, `download`, `edit_metadata`, `upload`, `request_books`, `see_prices`"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "unauthorized"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `403`: The caller is not an Owner or Administrator. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "forbidden"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `409`: The default kept changing underneath the write; retry.

```json
{
  "error": "core.member_controls kept changing; retry"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: link_accounts: invalid type: integer `1`, expected a boolean at line 1 column 18
```

Schema:

A `text/plain` body: a string.

Status `500`: The controls could not be written.

```json
{
  "error": "the patron's scope or controls could not be read or written"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/users/{id}/controls`

Reads what a patron may do: the library's default, their own
overrides, and what applies.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *integer (int64)*, **required** — User id.

### Responses

- `200` — The patron's controls. Body: [`UserControlsJson`](schemas.md#usercontrolsjson) (`application/json`)
  - `user_id` *integer (int64)*, **required** — The user (`users.id`).
  - `defaults` *[`MemberControls`](schemas.md#membercontrols)*, **required** — The library's default, which Members follow.
  - `overrides` *[`ControlOverrides`](schemas.md#controloverrides)*, **required** — The controls set for this patron alone; the others follow the default for a Member and are allowed for an Administrator.
  - `effective` *[`MemberControls`](schemas.md#membercontrols)*, **required** — What applies to the user now: nothing for a disabled user, everything for an Owner, the overrides alone for an Administrator, and the default with the overrides applied for a Member.
  - `applies` *boolean*, **required** — Whether controls can limit the user's role: false for an Owner, who may always do everything.
  - `follows_default` *boolean*, **required** — Whether the user follows the library's default: true for a Member, false for an Administrator, whom only controls set for them alone limit (and only an Owner or the operator sets those).
- `401` — The caller is not signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller may not manage this user. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `404` — No such user. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The controls could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users/<id>/controls' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/users/<id>/controls' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/users/<id>/controls",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/users/<id>/controls", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/users/<id>/controls")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The patron's controls.

```json
{
  "user_id": 7,
  "defaults": {
    "link_accounts": false,
    "download": true,
    "edit_metadata": true,
    "upload": true,
    "request_books": true,
    "see_prices": true
  },
  "overrides": {
    "link_accounts": false,
    "download": null,
    "edit_metadata": null,
    "upload": null,
    "request_books": null,
    "see_prices": null
  },
  "effective": {
    "link_accounts": false,
    "download": true,
    "edit_metadata": true,
    "upload": true,
    "request_books": true,
    "see_prices": true
  },
  "applies": true,
  "follows_default": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "A patron's member controls as the control routes report them.",
  "required": [
    "user_id",
    "defaults",
    "overrides",
    "effective",
    "applies",
    "follows_default"
  ],
  "properties": {
    "user_id": {
      "type": "integer",
      "format": "int64",
      "description": "The user (`users.id`).",
      "example": 7
    },
    "defaults": {
      "type": "object",
      "description": "The library's default, which Members follow.",
      "properties": {
        "link_accounts": {
          "type": "boolean",
          "description": "May connect store accounts ([`MemberControl::LinkAccounts`]).",
          "example": false
        },
        "download": {
          "type": "boolean",
          "description": "May take copies of the library's files ([`MemberControl::Download`]).",
          "example": true
        },
        "edit_metadata": {
          "type": "boolean",
          "description": "May change metadata ([`MemberControl::EditMetadata`]).",
          "example": true
        },
        "upload": {
          "type": "boolean",
          "description": "May upload books ([`MemberControl::Upload`]).",
          "example": true
        },
        "request_books": {
          "type": "boolean",
          "description": "May ask branches for books ([`MemberControl::RequestBooks`]).",
          "example": true
        },
        "see_prices": {
          "type": "boolean",
          "description": "May see prices and store links ([`MemberControl::SeePrices`]).",
          "example": true
        }
      },
      "additionalProperties": false
    },
    "overrides": {
      "type": "object",
      "description": "The controls set for this patron alone; the others follow the\ndefault for a Member and are allowed for an Administrator.",
      "properties": {
        "link_accounts": {
          "type": [
            "boolean",
            "null"
          ],
          "description": "Overrides [`MemberControls::link_accounts`].",
          "example": false
        },
        "download": {
          "type": [
            "boolean",
            "null"
          ],
          "description": "Overrides [`MemberControls::download`].",
          "example": null
        },
        "edit_metadata": {
          "type": [
            "boolean",
            "null"
          ],
          "description": "Overrides [`MemberControls::edit_metadata`].",
          "example": null
        },
        "upload": {
          "type": [
            "boolean",
            "null"
          ],
          "description": "Overrides [`MemberControls::upload`].",
          "example": null
        },
        "request_books": {
          "type": [
            "boolean",
            "null"
          ],
          "description": "Overrides [`MemberControls::request_books`].",
          "example": null
        },
        "see_prices": {
          "type": [
            "boolean",
            "null"
          ],
          "description": "Overrides [`MemberControls::see_prices`].",
          "example": null
        }
      },
      "additionalProperties": false
    },
    "effective": {
      "type": "object",
      "description": "What applies to the user now: nothing for a disabled user,\neverything for an Owner, the overrides alone for an Administrator,\nand the default with the overrides applied for a Member.",
      "properties": {
        "link_accounts": {
          "type": "boolean",
          "description": "May connect store accounts ([`MemberControl::LinkAccounts`]).",
          "example": false
        },
        "download": {
          "type": "boolean",
          "description": "May take copies of the library's files ([`MemberControl::Download`]).",
          "example": true
        },
        "edit_metadata": {
          "type": "boolean",
          "description": "May change metadata ([`MemberControl::EditMetadata`]).",
          "example": true
        },
        "upload": {
          "type": "boolean",
          "description": "May upload books ([`MemberControl::Upload`]).",
          "example": true
        },
        "request_books": {
          "type": "boolean",
          "description": "May ask branches for books ([`MemberControl::RequestBooks`]).",
          "example": true
        },
        "see_prices": {
          "type": "boolean",
          "description": "May see prices and store links ([`MemberControl::SeePrices`]).",
          "example": true
        }
      },
      "additionalProperties": false
    },
    "applies": {
      "type": "boolean",
      "description": "Whether controls can limit the user's role: false for an Owner, who\nmay always do everything.",
      "example": true
    },
    "follows_default": {
      "type": "boolean",
      "description": "Whether the user follows the library's default: true for a Member,\nfalse for an Administrator, whom only controls set for them alone\nlimit (and only an Owner or the operator sets those).",
      "example": true
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "unauthorized"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `403`: The caller may not manage this user.

```json
{
  "error": "you may not manage this user"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `404`: No such user.

```json
{
  "error": "user 7 not found"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `500`: The controls could not be read.

```json
{
  "error": "the patron's scope or controls could not be read or written"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `PUT /api/users/{id}/controls`

Sets the controls that differ for one patron; every other control
follows the library's default for a Member and is allowed for an
Administrator.

Nobody sets their own. An Owner may always do everything, so
overrides stored for one only apply if their role changes. An
Administrator's overrides limit them while they stay an Administrator;
only an Owner or the operator sets those.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *integer (int64)*, **required** — User id.

### Request body

`application/json`: [`PutUserControls`](schemas.md#putusercontrols)

- `overrides` *[`ControlOverrides`](schemas.md#controloverrides)*, **required** — The controls set for this patron alone; an absent or `null` control follows the library's default for a Member and is allowed for an Administrator, so `{}` clears every override.

### Responses

- `200` — The patron's controls after the change. Body: [`UserControlsJson`](schemas.md#usercontrolsjson) (`application/json`)
  - `user_id` *integer (int64)*, **required** — The user (`users.id`).
  - `defaults` *[`MemberControls`](schemas.md#membercontrols)*, **required** — The library's default, which Members follow.
  - `overrides` *[`ControlOverrides`](schemas.md#controloverrides)*, **required** — The controls set for this patron alone; the others follow the default for a Member and are allowed for an Administrator.
  - `effective` *[`MemberControls`](schemas.md#membercontrols)*, **required** — What applies to the user now: nothing for a disabled user, everything for an Owner, the overrides alone for an Administrator, and the default with the overrides applied for a Member.
  - `applies` *boolean*, **required** — Whether controls can limit the user's role: false for an Owner, who may always do everything.
  - `follows_default` *boolean*, **required** — Whether the user follows the library's default: true for a Member, false for an Administrator, whom only controls set for them alone limit (and only an Owner or the operator sets those).
- `400` — The body names a control that does not exist. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller may not manage this user, or tried to set their own controls. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `404` — No such user. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The user's role changed since the caller was authorized for them; nothing was written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The controls could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users/<id>/controls' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"overrides": {}}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/users/<id>/controls' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"overrides": {}}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/users/<id>/controls",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"overrides": {}},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/users/<id>/controls", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"overrides": {}}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/users/<id>/controls")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"overrides": {}}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The patron's controls after the change.

```json
{
  "user_id": 7,
  "defaults": {
    "link_accounts": false,
    "download": true,
    "edit_metadata": true,
    "upload": true,
    "request_books": true,
    "see_prices": true
  },
  "overrides": {
    "link_accounts": false,
    "download": null,
    "edit_metadata": null,
    "upload": null,
    "request_books": null,
    "see_prices": null
  },
  "effective": {
    "link_accounts": false,
    "download": true,
    "edit_metadata": true,
    "upload": true,
    "request_books": true,
    "see_prices": true
  },
  "applies": true,
  "follows_default": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "A patron's member controls as the control routes report them.",
  "required": [
    "user_id",
    "defaults",
    "overrides",
    "effective",
    "applies",
    "follows_default"
  ],
  "properties": {
    "user_id": {
      "type": "integer",
      "format": "int64",
      "description": "The user (`users.id`).",
      "example": 7
    },
    "defaults": {
      "type": "object",
      "description": "The library's default, which Members follow.",
      "properties": {
        "link_accounts": {
          "type": "boolean",
          "description": "May connect store accounts ([`MemberControl::LinkAccounts`]).",
          "example": false
        },
        "download": {
          "type": "boolean",
          "description": "May take copies of the library's files ([`MemberControl::Download`]).",
          "example": true
        },
        "edit_metadata": {
          "type": "boolean",
          "description": "May change metadata ([`MemberControl::EditMetadata`]).",
          "example": true
        },
        "upload": {
          "type": "boolean",
          "description": "May upload books ([`MemberControl::Upload`]).",
          "example": true
        },
        "request_books": {
          "type": "boolean",
          "description": "May ask branches for books ([`MemberControl::RequestBooks`]).",
          "example": true
        },
        "see_prices": {
          "type": "boolean",
          "description": "May see prices and store links ([`MemberControl::SeePrices`]).",
          "example": true
        }
      },
      "additionalProperties": false
    },
    "overrides": {
      "type": "object",
      "description": "The controls set for this patron alone; the others follow the\ndefault for a Member and are allowed for an Administrator.",
      "properties": {
        "link_accounts": {
          "type": [
            "boolean",
            "null"
          ],
          "description": "Overrides [`MemberControls::link_accounts`].",
          "example": false
        },
        "download": {
          "type": [
            "boolean",
            "null"
          ],
          "description": "Overrides [`MemberControls::download`].",
          "example": null
        },
        "edit_metadata": {
          "type": [
            "boolean",
            "null"
          ],
          "description": "Overrides [`MemberControls::edit_metadata`].",
          "example": null
        },
        "upload": {
          "type": [
            "boolean",
            "null"
          ],
          "description": "Overrides [`MemberControls::upload`].",
          "example": null
        },
        "request_books": {
          "type": [
            "boolean",
            "null"
          ],
          "description": "Overrides [`MemberControls::request_books`].",
          "example": null
        },
        "see_prices": {
          "type": [
            "boolean",
            "null"
          ],
          "description": "Overrides [`MemberControls::see_prices`].",
          "example": null
        }
      },
      "additionalProperties": false
    },
    "effective": {
      "type": "object",
      "description": "What applies to the user now: nothing for a disabled user,\neverything for an Owner, the overrides alone for an Administrator,\nand the default with the overrides applied for a Member.",
      "properties": {
        "link_accounts": {
          "type": "boolean",
          "description": "May connect store accounts ([`MemberControl::LinkAccounts`]).",
          "example": false
        },
        "download": {
          "type": "boolean",
          "description": "May take copies of the library's files ([`MemberControl::Download`]).",
          "example": true
        },
        "edit_metadata": {
          "type": "boolean",
          "description": "May change metadata ([`MemberControl::EditMetadata`]).",
          "example": true
        },
        "upload": {
          "type": "boolean",
          "description": "May upload books ([`MemberControl::Upload`]).",
          "example": true
        },
        "request_books": {
          "type": "boolean",
          "description": "May ask branches for books ([`MemberControl::RequestBooks`]).",
          "example": true
        },
        "see_prices": {
          "type": "boolean",
          "description": "May see prices and store links ([`MemberControl::SeePrices`]).",
          "example": true
        }
      },
      "additionalProperties": false
    },
    "applies": {
      "type": "boolean",
      "description": "Whether controls can limit the user's role: false for an Owner, who\nmay always do everything.",
      "example": true
    },
    "follows_default": {
      "type": "boolean",
      "description": "Whether the user follows the library's default: true for a Member,\nfalse for an Administrator, whom only controls set for them alone\nlimit (and only an Owner or the operator sets those).",
      "example": true
    }
  }
}
```

Status `400`: The body names a control that does not exist. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "unknown field `fly`, expected one of `link_accounts`, `download`, `edit_metadata`, `upload`, `request_books`, `see_prices`"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "unauthorized"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `403`: The caller may not manage this user, or tried to set their own controls. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "you may not set your own controls"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `404`: No such user.

```json
{
  "error": "user 7 not found"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `409`: The user's role changed since the caller was authorized for them; nothing was written.

```json
{
  "error": "user 7 became administrator while their controls were being set; read them again"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `overrides` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The controls could not be written.

```json
{
  "error": "the patron's scope or controls could not be read or written"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

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
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "forbidden",
  "message": "You do not have access to this resource.",
  "status": 403
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `404`: No such user.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

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
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "forbidden",
  "message": "You do not have access to this resource.",
  "status": 403
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `404`: No such user.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/avatars/gravatar/{hash}`

Serves a user's Gravatar as a 128 px JPEG through the cover service.

The `gravatar_hash` of a user (from `/api/auth/me` or `/api/users`) names
the picture. Bookclerk downloads it through the `covers` plugin, so the
browser never contacts Gravatar.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `hash` (path) *string*, **required** — A user's `gravatar_hash`: SHA-256 of the email, 64 lowercase hex digits.

### Responses

- `200` — The picture as a JPEG. Body: binary (`image/jpeg`)
- `304` — `If-None-Match` names the current `ETag`; no body. No body.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — The hash is not 64 lowercase hex digits. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `502` — The email has no Gravatar, or the image is not a supported image. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `503` — This node cannot fetch or process images right now, or too many Gravatars are downloading at once. The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/avatars/gravatar/<hash>' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/avatars/gravatar/<hash>' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/avatars/gravatar/<hash>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/avatars/gravatar/<hash>", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/avatars/gravatar/<hash>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The picture as a JPEG. The body is binary (`image/jpeg`).

Schema:

A `image/jpeg` body: binary bytes.

Status `304`: `If-None-Match` names the current `ETag`; no body. No body.

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`.

```json
{
  "error": "clerk_sign_in_unavailable",
  "message": "This library does not accept clerks on this address.",
  "status": 403
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `404`: The hash is not 64 lowercase hex digits.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `502`: The email has no Gravatar, or the image is not a supported image.

```json
{
  "error": "bad_gateway",
  "message": "An upstream service failed.",
  "status": 502
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `503`: This node cannot fetch or process images right now, or too many Gravatars are downloading at once. The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "service_unavailable",
  "message": "The Bookclerk daemon is not ready to handle this request.",
  "status": 503
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/users/{id}/avatar`

Serves a stored avatar JPEG for any authenticated household member.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *integer (int64)*, **required** — User id.

### Responses

- `200` — The avatar. Body: [`BinaryBody`](schemas.md#binarybody) (`image/jpeg`); [`BinaryBody`](schemas.md#binarybody) (`image/png`); [`BinaryBody`](schemas.md#binarybody) (`image/webp`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — No such user, or the user has no avatar. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The user could not be read. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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

A `image/jpeg` body: binary bytes.

Status `401`: The caller is not signed in.

```json
{
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`.

```json
{
  "error": "clerk_sign_in_unavailable",
  "message": "This library does not accept clerks on this address.",
  "status": 403
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `404`: No such user, or the user has no avatar.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `500`: The user could not be read.

```json
{
  "error": "internal_error",
  "message": "Something went wrong on the Bookclerk daemon.",
  "status": 500
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `503`: The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "session_check_unavailable",
  "message": "Your sign-in could not be checked yet. Try again in a moment.",
  "status": 503
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.
