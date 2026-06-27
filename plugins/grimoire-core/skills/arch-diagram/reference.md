# Diagram notation reference

Minimal, render-correct skeletons for each diagram type. Copy the one that
matches the question, then replace the placeholders. Mermaid is the default;
the PlantUML variants are for the fallback cases noted in `SKILL.md`.

## Mermaid

### C4 Context — what systems and people exist and how they link

```mermaid
C4Context
    title System Context for <System>

    Person(user, "User", "Who uses the system")
    System(sys, "<System>", "What it does")
    System_Ext(ext, "<External System>", "Third-party dependency")

    Rel(user, sys, "Uses", "HTTPS")
    Rel(sys, ext, "Calls", "REST")
```

### C4 Container — what containers make up one system

```mermaid
C4Container
    title Containers for <System>

    Person(user, "User")
    Container_Boundary(c, "<System>") {
        Container(web, "Web App", "React", "Serves the UI")
        Container(api, "API", "Go", "Business logic")
        ContainerDb(db, "Database", "PostgreSQL", "Stores state")
    }

    Rel(user, web, "Visits", "HTTPS")
    Rel(web, api, "Calls", "JSON/HTTPS")
    Rel(api, db, "Reads/writes", "SQL")
```

### Sequence — what happens, in what order, between parties

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant API
    participant DB

    User->>API: POST /orders
    API->>DB: INSERT order
    DB-->>API: order id
    API-->>User: 201 Created
    note over API,DB: wrap in a transaction
```

### Class — types and their relationships

```mermaid
classDiagram
    class Order {
        +string id
        +Money total
        +place() void
    }
    class LineItem {
        +string sku
        +int qty
    }
    Order "1" *-- "1..*" LineItem : contains
```

### Entity-relationship — what the persisted data looks like

```mermaid
erDiagram
    CUSTOMER ||--o{ ORDER : places
    ORDER ||--|{ LINE_ITEM : contains
    CUSTOMER {
        string id PK
        string email
    }
    ORDER {
        string id PK
        string customer_id FK
        string status
    }
```

### State machine — states a thing moves through

```mermaid
stateDiagram-v2
    [*] --> Pending
    Pending --> Paid : payment captured
    Pending --> Cancelled : timeout
    Paid --> Refunded : refund issued
    Refunded --> [*]
```

### Flowchart — process or decision logic

```mermaid
flowchart TD
    Start([Start]) --> Check{Valid?}
    Check -- yes --> Save[(Persist)]
    Check -- no --> Reject[Return 400]
    Save --> Done([Done])
```

### Deployment — what runs where at runtime

```mermaid
flowchart LR
    subgraph Cloud["AWS region"]
        subgraph Public["Public subnet"]
            LB[Load Balancer]
        end
        subgraph Private["Private subnet"]
            App[App servers]
            DB[(PostgreSQL)]
        end
    end
    User -->|HTTPS| LB
    LB --> App
    App --> DB
```

### Roadmap — delivery timeline / phasing

```mermaid
gantt
    title Delivery plan
    dateFormat YYYY-MM-DD
    section Phase 1
    Design      :done,    des1, 2026-01-01, 14d
    Build       :active,  bld1, after des1, 21d
    section Phase 2
    Rollout     :         rol1, after bld1, 10d
```

## PlantUML (fallback)

Wrap every diagram in `@startuml` / `@enduml`. PlantUML needs a PlantUML
renderer (server or macro) — it does not render natively on GitHub.

### Class — rich UML (generics, stereotypes, visibility, packages)

```plantuml
@startuml
package "ordering" {
  interface Repository<T> {
    +save(T): void
    +byId(id): T
  }
  class Order {
    -id: string
    -total: Money
    +place(): void
  }
  class OrderRepository <<adapter>>
  Repository <|.. OrderRepository
  OrderRepository ..> Order : persists
}
@enduml
```

### C4 component — detailed component view (C4-PlantUML)

```plantuml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Component.puml

Container_Boundary(api, "API") {
  Component(handler, "Order Handler", "HTTP", "Validates and routes")
  Component(svc, "Order Service", "Domain", "Business rules")
  Component(repo, "Order Repository", "Adapter", "Persistence")
}
ContainerDb(db, "Database", "PostgreSQL")

Rel(handler, svc, "Calls")
Rel(svc, repo, "Uses")
Rel(repo, db, "Reads/writes", "SQL")
@enduml
```
