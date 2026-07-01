# Order Service <!-- omit from toc -->

**Owner:** Payments team

Create and manage customer orders over gRPC. Every RPC shares the proto package, servers, authentication, and status-code model below.

## Contents <!-- omit from toc -->

- [How to read this doc](#how-to-read-this-doc)
- [Basics](#basics)
  - [Package](#package)
  - [Servers](#servers)
  - [Authentication](#authentication)
  - [Conventions](#conventions)
  - [Enums](#enums)
  - [Errors](#errors)
- [RPCs](#rpcs)
  - [CreateOrder](#createorder)
  - [GetOrder](#getorder)
  - [ListOrders](#listorders)
  - [CancelOrder](#cancelorder)

## How to read this doc

**Basics** describes everything the RPCs share: the proto package, servers, authentication, conventions, shared enums, and the status-code model. Each RPC then has its own section under **RPCs**, with its own status:

| Status        | Meaning                                      |
| ------------- | -------------------------------------------- |
| 🟡 Draft      | Work in progress; may change without notice. |
| 🟢 Stable     | Safe to build against.                       |
| 🔴 Deprecated | Avoid; scheduled for removal.                |

## Basics

### Package

| Field      | Value                                              |
| ---------- | -------------------------------------------------- |
| Proto file | `proto/example/orders/v1/orders.proto`             |
| Package    | `example.orders.v1`                                |
| Go package | `github.com/example/orders/gen/orders/v1;ordersv1` |

```protobuf
syntax = "proto3";

package example.orders.v1;

option go_package = "github.com/example/orders/gen/orders/v1;ordersv1";

service OrderService {
  // Create a new order.
  rpc CreateOrder(CreateOrderRequest) returns (Order);
  // Fetch a single order by ID.
  rpc GetOrder(GetOrderRequest) returns (Order);
  // Stream orders matching a filter, newest first.
  rpc ListOrders(ListOrdersRequest) returns (stream Order);
  // Cancel a pending order.
  rpc CancelOrder(CancelOrderRequest) returns (Order);
}
```

### Servers

| Environment | Address                          | TLS      |
| ----------- | -------------------------------- | -------- |
| Production  | `orders.example.com:443`         | required |
| Staging     | `staging.orders.example.com:443` | required |

### Authentication

Bearer token sent as call metadata on every RPC:

```text
authorization: Bearer <token>
```

Issue tokens from the OAuth2 token endpoint with the `orders` scope. Tokens expire after 1 hour.

### Conventions

- **Naming:** `snake_case` message fields, `PascalCase` services and RPCs.
- **Timestamps:** `google.protobuf.Timestamp` (UTC).
- **Pagination:** request `page_size` + `page_token`; response returns `next_page_token` (empty when no more pages).
- **Deadlines:** clients set a deadline on every call; the server returns `DEADLINE_EXCEEDED` when exceeded.
- **Idempotency:** send a `request_id` on writes to retry safely.

### Enums

| Enum     | Values                         | Description            |
| -------- | ------------------------------ | ---------------------- |
| `Status` | `PENDING`, `PAID`, `CANCELLED` | Order lifecycle state. |

### Errors

RPCs fail with a canonical gRPC status code; structured details follow `google.rpc.Status`:

```json
{
  "code": 3,
  "message": "amount must be positive",
  "details": [
    {
      "@type": "type.googleapis.com/google.rpc.BadRequest",
      "fieldViolations": [
        { "field": "amount", "description": "must be greater than 0" }
      ]
    }
  ]
}
```

| Code | Name                  | Meaning                                 |
| ---- | --------------------- | --------------------------------------- |
| 3    | `INVALID_ARGUMENT`    | Malformed request or failed validation. |
| 5    | `NOT_FOUND`           | Order does not exist.                   |
| 7    | `PERMISSION_DENIED`   | Token lacks the required scope.         |
| 9    | `FAILED_PRECONDITION` | Order not in a cancellable state.       |
| 16   | `UNAUTHENTICATED`     | Missing or invalid token.               |

## RPCs

### CreateOrder

**Status:** 🟢 Stable

Creates a new order in the `PENDING` state.

```protobuf
rpc CreateOrder(CreateOrderRequest) returns (Order);
```

**Type:** Unary

**Auth:** token with the `orders:write` scope.

#### Request — `CreateOrderRequest`

```protobuf
message CreateOrderRequest {
  int64 amount = 1;       // minor units (cents)
  string currency = 2;    // ISO 4217 code
  string customer_id = 3;
  string request_id = 4;  // idempotency key
}
```

| Field         | Type   | Required | Description                                       | Example  |
| ------------- | ------ | -------- | ------------------------------------------------- | -------- |
| `amount`      | int64  | yes      | Total in minor units (cents).                     | `1500`   |
| `currency`    | string | yes      | ISO 4217 code.                                    | `USD`    |
| `customer_id` | string | yes      | Customer the order belongs to.                    | `cus_42` |
| `request_id`  | string | no       | Idempotency key; repeats return the first result. | `req_88` |

#### Response — `Order`

```protobuf
message Order {
  string id = 1;
  Status status = 2;  // see Enums
  int64 amount = 3;
  string currency = 4;
  google.protobuf.Timestamp created_at = 5;
}
```

| Field        | Type            | Description            | Example                |
| ------------ | --------------- | ---------------------- | ---------------------- |
| `id`         | string          | Generated order ID.    | `ord_123`              |
| `status`     | `Status` (enum) | Order lifecycle state. | `PENDING`              |
| `amount`     | int64           | Total in minor units.  | `1500`                 |
| `currency`   | string          | ISO 4217 code.         | `USD`                  |
| `created_at` | Timestamp       | Creation time (UTC).   | `2026-05-20T09:30:00Z` |

#### Status codes

| Code               | When                                   |
| ------------------ | -------------------------------------- |
| `INVALID_ARGUMENT` | `amount` ≤ 0 or `currency` is unknown. |

#### Example

```bash
grpcurl -H "authorization: Bearer <token>" \
  -d '{"amount": 1500, "currency": "USD", "customer_id": "cus_42"}' \
  orders.example.com:443 \
  example.orders.v1.OrderService/CreateOrder
```

---

### GetOrder

**Status:** 🟢 Stable

Fetches a single order by ID.

```protobuf
rpc GetOrder(GetOrderRequest) returns (Order);
```

**Type:** Unary

**Auth:** token with the `orders:read` scope.

#### Request — `GetOrderRequest`

```protobuf
message GetOrderRequest {
  string id = 1;
}
```

| Field | Type   | Required | Description | Example   |
| ----- | ------ | -------- | ----------- | --------- |
| `id`  | string | yes      | Order ID.   | `ord_123` |

#### Response — `Order`

Same `Order` message as [CreateOrder](#createorder).

#### Status codes

| Code        | When                  |
| ----------- | --------------------- |
| `NOT_FOUND` | No order has that ID. |

#### Example

```bash
grpcurl -H "authorization: Bearer <token>" \
  -d '{"id": "ord_123"}' \
  orders.example.com:443 \
  example.orders.v1.OrderService/GetOrder
```

---

### ListOrders

**Status:** 🟡 Draft

Streams orders matching a filter, newest first.

```protobuf
rpc ListOrders(ListOrdersRequest) returns (stream Order);
```

**Type:** Server streaming

**Auth:** token with the `orders:read` scope.

#### Request — `ListOrdersRequest`

```protobuf
message ListOrdersRequest {
  Status status = 1;    // optional filter
  int32 page_size = 2;
  string page_token = 3;
}
```

| Field        | Type            | Required | Description                                 | Example       |
| ------------ | --------------- | -------- | ------------------------------------------- | ------------- |
| `status`     | `Status` (enum) | no       | Filter by status; unset returns all.        | `PAID`        |
| `page_size`  | int32           | no       | Max orders per response batch (default 50). | `50`          |
| `page_token` | string          | no       | Cursor from a previous call.                | `eyJvIjoyMH0` |

#### Response — stream of `Order`

The server streams one `Order` message per match, then closes the stream.

#### Example

```bash
grpcurl -H "authorization: Bearer <token>" \
  -d '{"status": "PAID", "page_size": 50}' \
  orders.example.com:443 \
  example.orders.v1.OrderService/ListOrders
```

---

### CancelOrder

**Status:** 🟢 Stable

Cancels a `PENDING` order. Idempotent — cancelling a cancelled order returns the same result.

```protobuf
rpc CancelOrder(CancelOrderRequest) returns (Order);
```

**Type:** Unary

**Auth:** token with the `orders:write` scope.

#### Request — `CancelOrderRequest`

```protobuf
message CancelOrderRequest {
  string id = 1;
  string request_id = 2;  // idempotency key
}
```

| Field        | Type   | Required | Description      | Example   |
| ------------ | ------ | -------- | ---------------- | --------- |
| `id`         | string | yes      | Order ID.        | `ord_123` |
| `request_id` | string | no       | Idempotency key. | `req_91`  |

#### Response — `Order`

Returns the order with `status = CANCELLED`.

#### Status codes

| Code                  | When                         |
| --------------------- | ---------------------------- |
| `FAILED_PRECONDITION` | The order is already `PAID`. |

#### Example

```bash
grpcurl -H "authorization: Bearer <token>" \
  -d '{"id": "ord_123"}' \
  orders.example.com:443 \
  example.orders.v1.OrderService/CancelOrder
```
