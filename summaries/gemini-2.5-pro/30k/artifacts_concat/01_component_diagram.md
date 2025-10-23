```markdown
graph TB
    subgraph "External Actors"
        direction LR
        User["User (Browser)"]
        DesktopUser["User (Desktop)"]
        CICD["CI/CD & Test Automation<br/>(Jenkins, Selenium, Pytest)"]
    end

    subgraph "Monolithic Web Application (WAR)"
        direction TB
        Frontend["Frontend<br/>(HTML, JS, JSP)"]
        WebLayer["Web Layer<br/>(Java Servlets)"]

        subgraph "Business Logic Layer"
            direction LR
            AuthLogic["Authentication Domain<br/>(RegistrationUtils, LoginUtils)"]
            LibraryLogic["Library Domain<br/>(LibraryUtils)"]
            MathLogic["Mathematics Domain<br/>(Calculator, Fibonacci)"]
        end

        Persistence["Shared Persistence Layer<br/>(Centralized DAO using JDBC)"]

        subgraph "H2 Database (managed by Flyway)"
            direction LR
            AuthDB[("AUTH Schema<br/>(users)")]
            LibraryDB[("LIBRARY Schema<br/>(books, loans)")]
        end

        Frontend -- "HTTP/AJAX Requests" --> WebLayer
        WebLayer -- "Java Call" --> AuthLogic
        WebLayer -- "Java Call" --> LibraryLogic
        WebLayer -- "Java Call" --> MathLogic

        AuthLogic -- "Uses" --> Persistence
        LibraryLogic -- "Uses" --> Persistence

        Persistence -- "SQL" --> AuthDB
        Persistence -- "SQL" --> LibraryDB
    end

    subgraph "Standalone Desktop Application (JAR)"
        direction TB
        DesktopUI["AutoInsurance UI<br/>(Java Swing)"]
        InsuranceLogic["AutoInsurance Processor<br/>(Calculation Logic)"]
        TestServer["Test Automation Server<br/>(Listens on TCP 8000)"]
    end

    User -- "Interacts with" --> Frontend
    DesktopUser -- "Interacts with" --> DesktopUI
    DesktopUI -- "Java Call" --> InsuranceLogic
    DesktopUI -- "Starts" --> TestServer

    CICD -- "HTTP API/UI Tests" --> WebLayer
    CICD -- "TCP Scripting" --> TestServer
```

The architecture consists of a layered monolithic web application and a separate desktop application. The monolith's core business domains (Authentication, Library) are tightly coupled through a shared persistence layer, which communicates with a single database that has logically separated schemas. The desktop application is fully self-contained, operating independently and exposing a unique TCP server for test automation.
```