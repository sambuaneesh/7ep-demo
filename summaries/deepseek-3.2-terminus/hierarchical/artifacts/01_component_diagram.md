```markdown
1. **Mermaid Diagram:**

```mermaid
graph TB
    %% Presentation Layer
    subgraph Presentation
        WEB[Web Servlets]
        SWING[Swing UI]
        REST[REST Endpoints]
    end
    
    %% Business Logic Layer
    subgraph BusinessLogic
        LIB[Library Services]
        AUTH[Authentication Services]
        MATH[Mathematics Engine]
        INS[Insurance Processor]
        EXP[Expense Calculator]
    end
    
    %% Domain Model Layer
    subgraph DomainModel
        LIB_DOM[Library Domain]
        AUTH_DOM[Auth Domain]
    end
    
    %% Persistence Layer
    subgraph Persistence
        PERSIST[Persistence Layer]
        DB[(H2 Database)]
    end
    
    %% Infrastructure Layer
    subgraph Infrastructure
        HELP[Helpers/Utilities]
        TOMCAT[Tomcat Manager]
        TEST[Testing Framework]
    end
    
    %% Interactions
    WEB --> LIB
    WEB --> AUTH
    WEB --> MATH
    SWING --> INS
    SWING --> AUTH
    
    LIB --> LIB_DOM
    AUTH --> AUTH_DOM
    MATH --> HELP
    INS --> HELP
    EXP --> HELP
    
    LIB --> PERSIST
    AUTH --> PERSIST
    MATH --> PERSIST
    
    PERSIST --> DB
    
    HELP --> WEB
    HELP --> LIB
    HELP --> AUTH
    HELP --> MATH
    HELP --> INS
    HELP --> EXP
    
    TOMCAT --> PERSIST
    TOMCAT --> DB
    
    TEST --> WEB
    TEST --> LIB
    TEST --> AUTH
    TEST --> MATH
    TEST --> SWING
    
    %% Key Integration Points
    AUTH -.-> LIB
    MATH -.-> LIB
```

2. **Rationale:**

The component boundaries follow a layered architecture with clear separation between presentation, business logic, domain models, and persistence. Communication patterns primarily use direct method calls within layers, with the persistence layer abstracted through the IPersistenceLayer interface. The helpers package provides cross-cutting utilities used by all other components, while the domain objects ensure type-safe data transfer between layers through immutable patterns.
```