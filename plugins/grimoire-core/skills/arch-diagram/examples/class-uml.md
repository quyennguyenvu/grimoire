# Ordering domain — class model (PlantUML)

The domain types and their relationships, shown in PlantUML because it expresses
generics, stereotypes, and a port/adapter split more precisely than Mermaid.
This needs a PlantUML renderer (server or Confluence PlantUML macro) — it does
not render natively on GitHub.

```plantuml
@startuml
package "ordering" {
  interface OrderRepository {
    +save(order: Order): void
    +byId(id: string): Order
  }

  class Order {
    -id: string
    -status: Status
    -items: LineItem[]
    +place(): void
    +total(): Money
  }

  class LineItem {
    -sku: string
    -qty: int
    -unitPrice: Money
  }

  enum Status {
    PENDING
    PAID
    CANCELLED
  }

  class PostgresOrderRepository <<adapter>>

  Order "1" *-- "1..*" LineItem : contains
  Order --> Status
  OrderRepository <|.. PostgresOrderRepository
  PostgresOrderRepository ..> Order : persists
}
@enduml
```
