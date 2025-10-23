```markdown
```mermaid
graph TB
    subgraph "Clients"
        User
        WebUI["Web UI (HTML/JS/JSP)"]
    end

    subgraph "Monolithic Web Application"
        direction LR
        Servlets["Web Layer (Servlets)"]

        subgraph "Business Logic Layer"
            direction TB
            AuthLogic["Authentication Domain Logic<br/>(RegistrationUtils, LoginUtils)"]
            LibraryLogic["Library Domain Logic<br/>(LibraryUtils)"]
            StatelessLogic["Stateless Logic<br/>(Calculator, AlcoholCalculator)"]
        end

        Persistence["Shared Persistence Layer<br/>(DAO / IPersistenceLayer)"]

        Servlets --> AuthLogic
        Servlets --> LibraryLogic
        Servlets --> StatelessLogic
        AuthLogic --> Persistence
        LibraryLogic --> Persistence
    end

    subgraph "Database"
        direction TB
        H2["H2 Database<br/>(PostgreSQL Mode)"]
        subgraph "Schemas"
            AuthSchema["AUTH Schema (User Table)"]
            LibrarySchema["LIBRARY Schema (Book, Borrower, Loan Tables)"]
        end
        H2 --- AuthSchema
        H2 --- LibrarySchema
    end

    subgraph "Standalone Desktop Application"
        DesktopUI["AutoInsurance UI (Swing)"]
        DesktopLogic["AutoInsurance Processor (Stateless Logic)"]
        DesktopUI --> DesktopLogic
    end

    User --> WebUI
    User --> DesktopUI
    WebUI -- HTTP API Calls --> Servlets
    Persistence -- JDBC --> H2

    style Persistence fill:#f9f,stroke:#333,stroke-width:2px
```

The diagram illustrates a classic N-Tier monolithic architecture where distinct business domains (Authentication, Library) are logically separated in code but are tightly coupled through a single, shared persistence layer. This centralized Data Access Object (DAO) acts as a bottleneck and the primary point of contention,funneling all database interactions through a single component to logically separated schemas within one database instance.
```