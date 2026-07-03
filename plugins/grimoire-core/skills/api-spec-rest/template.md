# <API or API Group Name> <!-- omit from toc -->

**Owner:** <team or person>

<One paragraph: what this API does, who calls it, and the problem it solves.>

## Contents <!-- omit from toc -->

- [Orientation](#orientation)
- [Fundamentals](#fundamentals)
  - [Base URLs](#base-urls)
  - [Authentication](#authentication)
  - [Conventions](#conventions)
  - [Response format](#response-format)
  - [Enums](#enums)
  - [Errors](#errors)
- [Endpoints](#endpoints)
  - [Create order](#create-order)

## Orientation

**Fundamentals** describes everything the endpoints share: base URLs, authentication, conventions, the response envelope, shared enums, and the error model. Each endpoint then has its own section under **Endpoints**, with its own status:

| Status        | Meaning                                      |
| ------------- | -------------------------------------------- |
| 🟡 Draft      | Work in progress; may change without notice. |
| 🟢 Stable     | Safe to build against.                       |
| 🔴 Deprecated | Avoid; scheduled for removal.                |

## Fundamentals

### Base URLs

| Environment | Base URL                           |
| ----------- | ---------------------------------- |
| Production  | https://api.example.com/v1         |
| Staging     | https://staging-api.example.com/v1 |

### Authentication

<Scheme — e.g. Bearer token, API key, OAuth2.> Send it on every request:

```http
Authorization: Bearer <token>
```

<How to obtain the credential, required scopes/roles, and token lifetime. If the API is public, write "None — this API is public" and drop the per-endpoint **Auth** notes.>

### Conventions

- **Format:** JSON request and response bodies; `Content-Type: application/json`.
- **Timestamps:** ISO 8601 in UTC (`2026-06-24T09:30:00Z`).
- **IDs:** 64-bit ids are returned as strings (JavaScript-safe).
- **Pagination:** `?page=<n>&limit=<n>`; responses include `total` and `next`.
- **Idempotency:** <if applicable — e.g. send an `Idempotency-Key` header on writes.>

### Response format

<Optional — omit this section if responses aren't wrapped. If every response shares an envelope, document it once here; per-endpoint Response tables then describe only the part that varies, using dotted paths (e.g. `data.id`).>

```json
{
  "data": {},
  "meta": {}
}
```

| Field  | Type   | Description                              | Example |
| ------ | ------ | ---------------------------------------- | ------- |
| `data` | object | The per-endpoint payload (shape varies). | —       |
| `meta` | object | Request/pagination metadata.             | —       |

### Enums

<Optional — omit if there are no shared enums. List values reused across endpoints once here, then reference the enum name from field tables (e.g. `enum (status)`).>

| Enum       | Values                            | Description                      |
| ---------- | --------------------------------- | -------------------------------- |
| `status`   | `pending`, `paid`, `cancelled`    | Order lifecycle state.           |
| `currency` | ISO 4217 code (e.g. `USD`, `EUR`) | Supported settlement currencies. |

### Errors

Every error returns this envelope with the matching HTTP status:

```json
{
  "error": {
    "code": "invalid_request",
    "message": "Human-readable explanation.",
    "details": []
  }
}
```

| Status | Code              | Meaning                                     |
| ------ | ----------------- | ------------------------------------------- |
| 400    | `invalid_request` | Malformed request or failed validation.     |
| 401    | `unauthorized`    | Missing or invalid credentials.             |
| 403    | `forbidden`       | Authenticated but not permitted.            |
| 404    | `not_found`       | Resource does not exist.                    |
| 409    | `conflict`        | State conflict (e.g. duplicate).            |
| 429    | `rate_limited`    | Too many requests — retry after the window. |
| 500    | `internal_error`  | Unexpected server error.                    |

## Endpoints

### Create order

**Status:** 🟢 Stable

<One line: what this endpoint does.>

```http
POST /orders
```

<Optional: longer description, side effects, or when to use it. Omit if the one-liner says enough.>

**Auth:** <required scope/role, or "inherits the default above"; omit for a public API>

#### Path parameters

| Name | Type   | Required | Description          | Example   |
| ---- | ------ | -------- | -------------------- | --------- |
| `id` | string | yes      | Resource identifier. | `ord_123` |

#### Query parameters

| Name     | Type   | Required | Default | Description                          | Example    |
| -------- | ------ | -------- | ------- | ------------------------------------ | ---------- |
| `expand` | string | no       | —       | Comma-separated relations to inline. | `customer` |

#### Request body

| Field      | Type    | Required | Description                   | Example |
| ---------- | ------- | -------- | ----------------------------- | ------- |
| `amount`   | integer | yes      | Total in minor units (cents). | `1500`  |
| `currency` | string  | yes      | ISO 4217 code.                | `USD`   |

```json
{
  "amount": 1500,
  "currency": "USD"
}
```

#### Response

`201 Created` — dotted paths describe nested fields.

| Field         | Type            | Description                    | Example   |
| ------------- | --------------- | ------------------------------ | --------- |
| `id`          | string          | Generated order ID.            | `ord_123` |
| `status`      | enum (`status`) | Order lifecycle state.         | `pending` |
| `amount`      | integer         | Total in minor units.          | `1500`    |
| `currency`    | string          | ISO 4217 code.                 | `USD`     |
| `customer.id` | string          | Customer the order belongs to. | `cus_42`  |

```json
{
  "id": "ord_123",
  "status": "pending",
  "amount": 1500,
  "currency": "USD",
  "customer": { "id": "cus_42" }
}
```

#### Example

```bash
curl -X POST https://api.example.com/v1/orders \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"amount": 1500, "currency": "USD"}'
```

#### Errors

| Status | Code              | When                                |
| ------ | ----------------- | ----------------------------------- |
| 400    | `invalid_request` | `amount` ≤ 0 or unknown `currency`. |
| 409    | `conflict`        | Duplicate `Idempotency-Key`.        |
