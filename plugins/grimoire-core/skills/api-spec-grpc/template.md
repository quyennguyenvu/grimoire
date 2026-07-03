# <Service or Service Group Name> <!-- omit from toc -->

**Owner:** <team or person>

<One paragraph: what this service does, who calls it, and the problem it solves.>

## Contents <!-- omit from toc -->

- [Orientation](#orientation)
- [Fundamentals](#fundamentals)
  - [Package](#package)
  - [Servers](#servers)
  - [Authentication](#authentication)
  - [Conventions](#conventions)
  - [Enums](#enums)
  - [Errors](#errors)
- [RPCs](#rpcs)
  - [CreateOrder](#createorder)

## Orientation

**Fundamentals** describes everything the RPCs share: the proto package, servers, authentication, conventions, shared enums, and the status-code model. Each RPC then has its own section under **RPCs**, with its own status:

| Status        | Meaning                                      |
| ------------- | -------------------------------------------- |
| 🟡 Draft      | Work in progress; may change without notice. |
| 🟢 Stable     | Safe to build against.                       |
| 🔴 Deprecated | Avoid; scheduled for removal.                |

## Fundamentals

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
  // Stream orders matching a filter.
  rpc ListOrders(ListOrdersRequest) returns (stream Order);
}
```

### Servers

| Environment | Address                          | TLS      |
| ----------- | -------------------------------- | -------- |
| Production  | `orders.example.com:443`         | required |
| Staging     | `staging.orders.example.com:443` | required |

### Authentication

<Scheme - e.g. bearer token, mTLS.> Send credentials as call metadata on every RPC:

```text
authorization: Bearer <token>
```

<How to obtain the credential, required scopes/roles, and token lifetime. If the service is public, write "None - this service is public" and drop the per-RPC **Auth** notes.>

### Conventions

- **Naming:** `snake_case` message fields, `PascalCase` services and RPCs.
- **Timestamps:** `google.protobuf.Timestamp` (UTC).
- **IDs:** 64-bit ids cross the wire as strings.
- **Pagination:** request `page_size` + `page_token`; response returns `next_page_token` (empty when no more pages).
- **Deadlines:** clients set a deadline on every call; servers honor it and return `DEADLINE_EXCEEDED` when exceeded.
- **Idempotency:** <if applicable - e.g. send a `request_id` field on writes.>

### Enums

<Optional - omit if there are no shared enums. List proto enums reused across RPCs once here, then reference the enum name from field tables (e.g. `Status (enum)`).>

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

| Code | Name                  | Meaning                                  |
| ---- | --------------------- | ---------------------------------------- |
| 3    | `INVALID_ARGUMENT`    | Malformed request or failed validation.  |
| 5    | `NOT_FOUND`           | Resource does not exist.                 |
| 6    | `ALREADY_EXISTS`      | Resource conflict (e.g. duplicate).      |
| 7    | `PERMISSION_DENIED`   | Authenticated but not permitted.         |
| 8    | `RESOURCE_EXHAUSTED`  | Quota or rate limit exceeded.            |
| 9    | `FAILED_PRECONDITION` | System not in a state for the operation. |
| 4    | `DEADLINE_EXCEEDED`   | Call exceeded its deadline.              |
| 14   | `UNAVAILABLE`         | Transient outage - retry with backoff.   |
| 16   | `UNAUTHENTICATED`     | Missing or invalid credentials.          |
| 13   | `INTERNAL`            | Unexpected server error.                 |

## RPCs

### CreateOrder

**Status:** 🟢 Stable

<One line: what this RPC does.>

```protobuf
rpc CreateOrder(CreateOrderRequest) returns (Order);
```

**Type:** <Unary · Server streaming · Client streaming · Bidirectional streaming>

**Auth:** <required scope/role, or "inherits the default above"; omit for a public service>

#### Request — `CreateOrderRequest`

```protobuf
message CreateOrderRequest {
  int64 amount = 1;     // minor units (cents)
  string currency = 2;  // ISO 4217 code
}
```

| Field      | Type   | Required | Description                   | Example |
| ---------- | ------ | -------- | ----------------------------- | ------- |
| `amount`   | int64  | yes      | Total in minor units (cents). | `1500`  |
| `currency` | string | yes      | ISO 4217 code.                | `USD`   |

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

| Code               | When                                |
| ------------------ | ----------------------------------- |
| `INVALID_ARGUMENT` | `amount` ≤ 0 or unknown `currency`. |
| `UNAUTHENTICATED`  | Missing or invalid token.           |

#### Example

```bash
grpcurl -H "authorization: Bearer <token>" \
  -d '{"amount": 1500, "currency": "USD"}' \
  orders.example.com:443 \
  example.orders.v1.OrderService/CreateOrder
```

**Notes:** <deadlines, retries, idempotency, or streaming behavior, if any.>
