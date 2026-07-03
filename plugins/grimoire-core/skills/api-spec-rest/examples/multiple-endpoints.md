# Orders API <!-- omit from toc -->

**Owner:** Payments team

Create and manage customer orders. Every endpoint shares the base URL, authentication, and error model below.

## Contents <!-- omit from toc -->

- [Orientation](#orientation)
- [Fundamentals](#fundamentals)
  - [Base URLs](#base-urls)
  - [Authentication](#authentication)
  - [Conventions](#conventions)
  - [Enums](#enums)
  - [Errors](#errors)
- [Endpoints](#endpoints)
  - [Create order](#create-order)
  - [Get order](#get-order)
  - [List orders](#list-orders)
  - [Cancel order](#cancel-order)

## Orientation

**Fundamentals** describes everything the endpoints share: base URLs, authentication, conventions, shared enums, and the error model. Each endpoint then has its own section under **Endpoints**, with its own status:

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

Bearer token (API key). Send it on every request:

```http
Authorization: Bearer <token>
```

Create keys in the dashboard under Settings → API keys. Keys do not expire but can be revoked.

### Conventions

- **Format:** JSON request and response bodies; `Content-Type: application/json`.
- **Timestamps:** ISO 8601 in UTC (`2026-06-24T09:30:00Z`).
- **IDs:** order ids are returned as strings.
- **Pagination:** `?page=<n>&limit=<n>`; responses include `total` and `next`.
- **Idempotency:** send an `Idempotency-Key` header on `POST` to retry safely.

### Enums

| Enum       | Values                            | Description            |
| ---------- | --------------------------------- | ---------------------- |
| `status`   | `pending`, `paid`, `cancelled`    | Order lifecycle state. |
| `currency` | ISO 4217 code (e.g. `USD`, `EUR`) | Settlement currency.   |

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
| 409    | `conflict`        | State conflict (e.g. already cancelled).    |
| 429    | `rate_limited`    | Too many requests — retry after the window. |

## Endpoints

### Create order

**Status:** 🟢 Stable

Creates a new order in the `pending` state.

```http
POST /orders
```

**Auth:** key with the `orders:write` scope.

#### Request body

| Field         | Type    | Required | Description                    | Example  |
| ------------- | ------- | -------- | ------------------------------ | -------- |
| `amount`      | integer | yes      | Total in minor units (cents).  | `1500`   |
| `currency`    | string  | yes      | ISO 4217 code.                 | `USD`    |
| `customer_id` | string  | yes      | Customer the order belongs to. | `cus_42` |

```json
{
  "amount": 1500,
  "currency": "USD",
  "customer_id": "cus_42"
}
```

#### Response

`201 Created`

| Field        | Type            | Description            | Example                |
| ------------ | --------------- | ---------------------- | ---------------------- |
| `id`         | string          | Generated order ID.    | `ord_123`              |
| `status`     | enum (`status`) | Order lifecycle state. | `pending`              |
| `amount`     | integer         | Total in minor units.  | `1500`                 |
| `currency`   | string          | ISO 4217 code.         | `USD`                  |
| `created_at` | string          | Creation time (UTC).   | `2026-06-24T09:30:00Z` |

```json
{
  "id": "ord_123",
  "status": "pending",
  "amount": 1500,
  "currency": "USD",
  "created_at": "2026-06-24T09:30:00Z"
}
```

#### Example

```bash
curl -X POST https://api.example.com/v1/orders \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"amount": 1500, "currency": "USD", "customer_id": "cus_42"}'
```

#### Errors

| Status | Code              | When                                |
| ------ | ----------------- | ----------------------------------- |
| 400    | `invalid_request` | `amount` ≤ 0 or unknown `currency`. |

---

### Get order

**Status:** 🟢 Stable

Fetches a single order by ID.

```http
GET /orders/{id}
```

**Auth:** key with the `orders:read` scope.

#### Path parameters

| Name | Type   | Required | Description | Example   |
| ---- | ------ | -------- | ----------- | --------- |
| `id` | string | yes      | Order ID.   | `ord_123` |

#### Response

`200 OK` — same `Order` body as [Create order](#create-order).

```json
{
  "id": "ord_123",
  "status": "paid",
  "amount": 1500,
  "currency": "USD",
  "created_at": "2026-06-24T09:30:00Z"
}
```

#### Errors

| Status | Code        | When                  |
| ------ | ----------- | --------------------- |
| 404    | `not_found` | No order has that ID. |

---

### List orders

**Status:** 🟢 Stable

Returns a paginated list of orders, newest first.

```http
GET /orders
```

**Auth:** key with the `orders:read` scope.

#### Query parameters

| Name     | Type            | Required | Default | Description                | Example |
| -------- | --------------- | -------- | ------- | -------------------------- | ------- |
| `status` | enum (`status`) | no       | —       | Filter by lifecycle state. | `paid`  |
| `page`   | integer         | no       | 1       | Page number.               | `2`     |
| `limit`  | integer         | no       | 20      | Page size (max 100).       | `50`    |

#### Response

`200 OK`

| Field           | Type            | Description                      | Example   |
| --------------- | --------------- | -------------------------------- | --------- |
| `data[].id`     | string          | Order ID.                        | `ord_123` |
| `data[].status` | enum (`status`) | Lifecycle state.                 | `paid`    |
| `total`         | integer         | Total matching orders.           | `1`       |
| `next`          | string \| null  | URL of the next page, or `null`. | `null`    |

```json
{
  "data": [
    { "id": "ord_123", "status": "paid", "amount": 1500, "currency": "USD" }
  ],
  "total": 1,
  "next": null
}
```

---

### Cancel order

**Status:** 🟡 Draft

Cancels a `pending` order. Idempotent — cancelling a cancelled order returns the same result.

```http
POST /orders/{id}/cancel
```

**Auth:** key with the `orders:write` scope.

#### Path parameters

| Name | Type   | Required | Description | Example   |
| ---- | ------ | -------- | ----------- | --------- |
| `id` | string | yes      | Order ID.   | `ord_123` |

#### Response

`200 OK`

| Field    | Type            | Description                        | Example     |
| -------- | --------------- | ---------------------------------- | ----------- |
| `id`     | string          | Order ID.                          | `ord_123`   |
| `status` | enum (`status`) | Lifecycle state (now `cancelled`). | `cancelled` |

```json
{
  "id": "ord_123",
  "status": "cancelled",
  "amount": 1500,
  "currency": "USD"
}
```

#### Example

```bash
curl -X POST https://api.example.com/v1/orders/ord_123/cancel \
  -H "Authorization: Bearer <token>"
```

#### Errors

| Status | Code       | When                                                 |
| ------ | ---------- | ---------------------------------------------------- |
| 409    | `conflict` | The order is already `paid` and cannot be cancelled. |
