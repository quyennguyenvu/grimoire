# Health API

**Owner:** Platform team

Liveness and readiness probe for the orders platform. Load balancers and Kubernetes call it to decide whether to route traffic to an instance. It is public — no authentication required.

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
| Proto file | `proto/example/health/v1/health.proto`             |
| Package    | `example.health.v1`                                |
| Go package | `github.com/example/orders/gen/health/v1;healthv1` |

```protobuf
syntax = "proto3";

package example.health.v1;

option go_package = "github.com/example/orders/gen/health/v1;healthv1";

service HealthService {
  // Report whether the named service is serving.
  rpc Check(CheckRequest) returns (CheckResponse);
}
```

### Servers

| Environment | Address                          | TLS      |
| ----------- | -------------------------------- | -------- |
| Production  | `orders.example.com:443`         | required |
| Staging     | `staging.orders.example.com:443` | required |

### Authentication

None — this service is public. Probes can call it without credentials, before a token is ever issued.

### Conventions

- **Naming:** `snake_case` message fields, `PascalCase` services and RPCs.
- **Deadlines:** probes set a 1s deadline; the server returns `DEADLINE_EXCEEDED` if it can't answer in time.

### Enums

| Enum            | Values                              | Description                                     |
| --------------- | ----------------------------------- | ----------------------------------------------- |
| `ServingStatus` | `SERVING`, `NOT_SERVING`, `UNKNOWN` | Whether the named service is accepting traffic. |

### Errors

RPCs fail with a canonical gRPC status code:

| Code | Name                | Meaning                      |
| ---- | ------------------- | ---------------------------- |
| 4    | `DEADLINE_EXCEEDED` | Probe exceeded its deadline. |
| 13   | `INTERNAL`          | Unexpected server error.     |

## RPCs

### Check

**Status:** 🟢 Stable

Reports whether the named service is currently serving.

```protobuf
rpc Check(CheckRequest) returns (CheckResponse);
```

**Type:** Unary

#### Request — `CheckRequest`

```protobuf
message CheckRequest {
  string service = 1;  // empty checks the whole server
}
```

| Field     | Type   | Required | Description                                  | Example  |
| --------- | ------ | -------- | -------------------------------------------- | -------- |
| `service` | string | no       | Service name; empty checks the whole server. | `orders` |

#### Response — `CheckResponse`

```protobuf
message CheckResponse {
  ServingStatus status = 1;  // see Enums
}
```

| Field    | Type                   | Description                     | Example   |
| -------- | ---------------------- | ------------------------------- | --------- |
| `status` | `ServingStatus` (enum) | Whether the service is serving. | `SERVING` |

#### Example

```bash
grpcurl -d '{"service": ""}' \
  orders.example.com:443 \
  example.health.v1.HealthService/Check
```

**Notes:** unauthenticated; safe to call on a tight interval.
