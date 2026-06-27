# Order platform — system context

How the order platform sits among its users and external dependencies. Scope: the
platform as a single black box; internals are out of scope (see a container
diagram for those).

```mermaid
C4Context
    title System Context for Order Platform

    Person(customer, "Customer", "Places and tracks orders")
    Person(ops, "Ops agent", "Handles refunds and disputes")

    System(platform, "Order Platform", "Accepts orders, takes payment, schedules fulfilment")

    System_Ext(psp, "Payment Provider", "Authorizes and captures card payments")
    System_Ext(email, "Email Service", "Sends transactional email")
    System_Ext(wms, "Warehouse System", "Owns inventory and shipping")

    Rel(customer, platform, "Places orders, checks status", "HTTPS")
    Rel(ops, platform, "Issues refunds", "HTTPS")
    Rel(platform, psp, "Authorizes / captures payment", "REST")
    Rel(platform, email, "Sends receipts", "SMTP/API")
    Rel(platform, wms, "Submits fulfilment requests", "REST")
```
