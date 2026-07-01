# Exchange Rates API <!-- omit from toc -->

**Owner:** Data team

Public foreign-exchange reference rates. Anyone can read the latest and historical rates — no account or credentials required.

## Contents <!-- omit from toc -->

- [How to read this doc](#how-to-read-this-doc)
- [Basics](#basics)
  - [Base URLs](#base-urls)
  - [Authentication](#authentication)
  - [Conventions](#conventions)
  - [Errors](#errors)
- [Endpoints](#endpoints)
  - [List currencies](#list-currencies)
  - [Get latest rates](#get-latest-rates)

## How to read this doc

**Basics** describes everything the endpoints share: base URLs, authentication, conventions, and the error model. Each endpoint then has its own section under **Endpoints**, with its own status:

| Status        | Meaning                                      |
| ------------- | -------------------------------------------- |
| 🟡 Draft      | Work in progress; may change without notice. |
| 🟢 Stable     | Safe to build against.                       |
| 🔴 Deprecated | Avoid; scheduled for removal.                |

## Basics

### Base URLs

| Environment | Base URL                         |
| ----------- | -------------------------------- |
| Production  | https://api.rates.example.com/v1 |

### Authentication

None — this API is public. No credentials, API key, or `Authorization` header is required. Requests are rate-limited by client IP (see Conventions).

### Conventions

- **Format:** JSON response bodies; `Content-Type: application/json`.
- **Timestamps:** ISO 8601 dates in UTC (`2026-06-24`).
- **Rate limit:** 60 requests/minute per IP; the `Retry-After` header gives the cooldown on a `429`.

### Errors

Every error returns this envelope with the matching HTTP status:

```json
{
  "error": {
    "code": "not_found",
    "message": "Human-readable explanation."
  }
}
```

| Status | Code              | Meaning                                     |
| ------ | ----------------- | ------------------------------------------- |
| 400    | `invalid_request` | Malformed request or failed validation.     |
| 404    | `not_found`       | Resource does not exist.                    |
| 429    | `rate_limited`    | Too many requests — retry after the window. |
| 500    | `internal_error`  | Unexpected server error.                    |

## Endpoints

### List currencies

**Status:** 🟢 Stable

Returns the currencies the service quotes.

```http
GET /currencies
```

#### Response

`200 OK`

| Field         | Type   | Description                   | Example     |
| ------------- | ------ | ----------------------------- | ----------- |
| `data`        | array  | Array of currency objects.    | —           |
| `data[].code` | string | ISO 4217 code.                | `USD`       |
| `data[].name` | string | Human-readable currency name. | `US Dollar` |

```json
{
  "data": [
    { "code": "USD", "name": "US Dollar" },
    { "code": "EUR", "name": "Euro" }
  ]
}
```

#### Example

```bash
curl https://api.rates.example.com/v1/currencies
```

---

### Get latest rates

**Status:** 🟢 Stable

Returns the latest rates for a base currency.

```http
GET /rates/latest
```

#### Query parameters

| Name      | Type   | Required | Default | Description                                  | Example   |
| --------- | ------ | -------- | ------- | -------------------------------------------- | --------- |
| `base`    | string | no       | `USD`   | Base currency (ISO 4217).                    | `EUR`     |
| `symbols` | string | no       | —       | Comma-separated codes to limit the response. | `EUR,JPY` |

#### Response

`200 OK`

| Field   | Type   | Description                                  | Example        |
| ------- | ------ | -------------------------------------------- | -------------- |
| `base`  | string | Base currency.                               | `USD`          |
| `date`  | string | Date the rates were published (UTC).         | `2026-06-24`   |
| `rates` | object | Map of currency code to rate against `base`. | `{"EUR":0.92}` |

```json
{
  "base": "USD",
  "date": "2026-06-24",
  "rates": {
    "EUR": 0.92,
    "JPY": 157.3
  }
}
```

#### Example

```bash
curl "https://api.rates.example.com/v1/rates/latest?base=USD&symbols=EUR,JPY"
```

#### Errors

| Status | Code              | When                                |
| ------ | ----------------- | ----------------------------------- |
| 400    | `invalid_request` | `base` is not a supported currency. |
