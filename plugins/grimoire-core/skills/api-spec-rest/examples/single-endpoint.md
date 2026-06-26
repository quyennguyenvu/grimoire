# User API

**Owner:** Identity team

Returns the profile of the authenticated user. Web and mobile clients call it to render the account header and personalize the session.

## How to read this doc

**Basics** describes everything the endpoints share: base URLs, authentication, conventions, and the error model. Each endpoint then has its own section under **Endpoints**, with its own status:

| Status        | Meaning                                      |
| ------------- | -------------------------------------------- |
| 🟡 Draft      | Work in progress; may change without notice. |
| 🟢 Stable     | Safe to build against.                       |
| 🔴 Deprecated | Avoid; scheduled for removal.                |

## Basics

### Base URLs

| Environment | Base URL                           |
| ----------- | ---------------------------------- |
| Production  | https://api.example.com/v1         |
| Staging     | https://staging-api.example.com/v1 |

### Authentication

Bearer token (OAuth2 access token). Send it on every request:

```http
Authorization: Bearer <token>
```

Obtain a token from the OAuth2 token endpoint with the `profile` scope. Tokens expire after 1 hour.

### Conventions

- **Format:** JSON response bodies; `Content-Type: application/json`.
- **Timestamps:** ISO 8601 in UTC (`2026-06-24T09:30:00Z`).

### Errors

Every error returns this envelope with the matching HTTP status:

```json
{
  "error": {
    "code": "unauthorized",
    "message": "Human-readable explanation."
  }
}
```

| Status | Code             | Meaning                         |
| ------ | ---------------- | ------------------------------- |
| 401    | `unauthorized`   | Missing or invalid credentials. |
| 500    | `internal_error` | Unexpected server error.        |

## Endpoints

### Get current user

**Status:** 🟢 Stable

Returns the authenticated user's profile.

```http
GET /me
```

**Auth:** any valid token with the `profile` scope.

#### Query parameters

| Name     | Type   | Required | Default | Description                          | Example        |
| -------- | ------ | -------- | ------- | ------------------------------------ | -------------- |
| `expand` | string | no       | —       | Comma-separated relations to inline. | `organization` |

#### Response

`200 OK`

| Field        | Type   | Description                  | Example                |
| ------------ | ------ | ---------------------------- | ---------------------- |
| `id`         | string | User ID.                     | `usr_8f2a`             |
| `email`      | string | Primary email address.       | `ada@example.com`      |
| `name`       | string | Display name.                | `Ada Lovelace`         |
| `created_at` | string | Account creation time (UTC). | `2025-11-02T14:05:00Z` |

```json
{
  "id": "usr_8f2a",
  "email": "ada@example.com",
  "name": "Ada Lovelace",
  "created_at": "2025-11-02T14:05:00Z"
}
```

#### Example

```bash
curl https://api.example.com/v1/me \
  -H "Authorization: Bearer <token>"
```

#### Errors

| Status | Code           | When                             |
| ------ | -------------- | -------------------------------- |
| 401    | `unauthorized` | The token is missing or expired. |
