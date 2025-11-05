```mermaid
graph TB
    subgraph "External Actors & QA"
        User(["<font size=5><b>User</b></font><br/>(via Browser)"])
        TestingSuite["<b>Automated Testing Suite</b><br/>(Selenium, HtmlUnit, Cucumber BDD)"]
    end

    subgraph "7ep-demo Application"
        subgraph "Application Core Layers"
            WebLayer["<b>Web Presentation Layer</b><br/>Servlets for Library, Auth, Math"]
            BusinessLayer["<b>Business Logic Layer</b><br/>LibraryUtils, LoginUtils, RegistrationUtils, Math Calculators"]
            PersistenceLayer["<b>Persistence Layer</b><br/>IPersistenceLayer, PersistenceLayer<br/><i>(JDBC, Flyway Migrations)</i>"]
            DomainModel["<b>Domain Model</b><br/>Book, Borrower, Loan, User,<br/>RegistrationResult"]
        end

        subgraph "Supporting Infrastructure"
            AppLifecycle["<b>Application Lifecycle</b><br/>WebAppListener"]
            Helpers["<b>Utilities / Helpers</b><br/>CheckUtils, StringUtils, ServletUtils"]
        end

        subgraph "Self-Contained Educational Modules"
            AutoInsurance["<b>Auto Insurance System</b><br/>UI, Processor, Client-Server Demo"]
            OtherModules["<b>Other Modules</b><br/>Cartesian Product, Expenses"]
        end
    end

    %% --- Core Application Interactions ---
    User --> WebLayer
    WebLayer -- "Delegates to" --> BusinessLayer
    WebLayer -- "Uses for dispatch" --> Helpers

    BusinessLayer -- "Uses (CRUD ops via Interface)" --> PersistenceLayer
    BusinessLayer -- "Operates on" --> DomainModel
    BusinessLayer -- "Uses for validation" --> Helpers

    PersistenceLayer -- "Manages" --> DomainModel
    
    AppLifecycle -- "Initializes / Migrates DB on Startup" --> PersistenceLayer

    %% --- Testing Interactions ---
    TestingSuite -- "UI/API Tests" --> WebLayer
    TestingSuite -- "BDD/Unit Tests & DB Setup" --> BusinessLayer
    TestingSuite -- "DB Reset" --> PersistenceLayer
    TestingSuite -- "Tests Modules" --> AutoInsurance
    TestingSuite -- "Tests Modules" --> OtherModules
    
    %% --- Styling ---
    classDef core fill:#e6f3ff,stroke:#005cb3,stroke-width:2px;
    classDef support fill:#e6ffe6,stroke:#006400,stroke-width:2px;
    classDef qa fill:#fff0e6,stroke:#d95f00,stroke-width:2px;
    classDef external fill:#f2f2f2,stroke:#333,stroke-width:2px;
    classDef educational fill:#f3e6ff,stroke:#6a00a7,stroke-width:2px;

    class WebLayer,BusinessLayer,PersistenceLayer,DomainModel core;
    class AppLifecycle,Helpers support;
    class TestingSuite qa;
    class User external;
    class AutoInsurance,OtherModules educational;
```

The system employs a classic layered architecture where a Web Presentation Layer (Servlets) handles HTTP requests and delegates operations to a cohesive Business Logic Layer. This layer, in turn, interacts with an abstracted Persistence Layer for all data storage and retrieval, ensuring a clear separation of concerns. All components rely on a central Domain Model for data representation and are rigorously validated by a comprehensive, multi-level automated testing suite.