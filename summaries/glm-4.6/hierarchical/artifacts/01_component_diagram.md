
```mermaid
graph TB
    User[User] -->|HTTP Request| WebLayer
    WebLayer[Web Layer\n(Servlets)] -->|Delegates to| ServiceLayer[Service Layer\n(Utils, Processors)]
    ServiceLayer -->|Manipulates| DomainLayer[Domain Layer\n(Entity & Value Objects)]
    ServiceLayer -->|Reads/Writes| PersistenceLayer[Persistence Layer\n(IPersistenceLayer, SqlData)]
    PersistenceLayer -->|Queries| DB[(H2 Database)]
    
    Infrastructure[Infrastructure\n(Tomcat, Helpers)] -->|Initializes| PersistenceLayer
    WebLayer -->|Uses| Infrastructure
    ServiceLayer -->|Uses| Infrastructure

    TestSuites[Test Suites\n(Unit, BDD, UI)] -.->|Validates| WebLayer
    TestSuites -.->|Validates| ServiceLayer
    TestSuites -.->|Validates| DomainLayer
```

The architecture follows a classic layered design with clear boundaries separating web, business, domain, and data persistence concerns, promoting modularity and maintainability. Communication flows top-down for business operations, with cross-cutting components like infrastructure and testing providing foundational support and validation across the stack.