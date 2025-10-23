```markdown
```mermaid
graph TB
    subgraph "7ep-demo Monolithic Web Application"
        direction TB

        subgraph "Web Layer (Java Servlets)"
            AuthServlets["Authentication Servlets</br>(/login, /register)"]
            LibraryServlets["Library Servlets</br>(/registerbook, /lend, /book)"]
            MathServlets["Mathematics Servlets</br>(/math, /fibonacci)"]
            AdminServlets["DB Admin Servlet</br>(/flyway)"]
        end

        subgraph "Business Logic Layer"
            AuthLogic["Authentication Logic</br>(RegistrationUtils, LoginUtils)"]
            LibraryLogic["Library Logic</br>(LibraryUtils)"]
            MathLogic["Mathematics Logic</br>(Calculator, Fibonacci, Ackermann)"]
        end

        subgraph "Data Access Layer (DAL)"
            PersistenceInterface["IPersistenceLayer (Interface)"]
            PersistenceImpl["PersistenceLayer (JDBC Impl.)"]
        end
        
        WebAppListener["WebAppListener (Startup Hook)"]
        
        PersistenceInterface --> PersistenceImpl

        AuthServlets --> AuthLogic
        LibraryServlets --> LibraryLogic
        MathServlets --> MathLogic
        
        AuthLogic --> PersistenceInterface
        LibraryLogic --> PersistenceInterface
        AdminServlets --> PersistenceInterface
    end

    subgraph "External & Infrastructure"
        Browser["Frontend (Browser)</br>HTML, JS (AJAX)"]
        H2DB["H2 Database (In-Memory)"]
        Flyway["Flyway Migrations"]
    end

    subgraph "Standalone Desktop Application"
        DesktopApp["Auto Insurance UI</br>(Java Swing)"]
        ScriptServer["AutoInsuranceScriptServer</br>(TCP/IP Socket)"]
        TestClient["Test Automation Client"]
        
        DesktopApp -.-> ScriptServer
        TestClient -- "TCP Commands" --> ScriptServer
    end

    Browser -- "HTTP Requests" --> AuthServlets
    Browser -- "HTTP Requests" --> LibraryServlets
    Browser -- "HTTP Requests" --> MathServlets

    PersistenceImpl -- "JDBC" --> H2DB
    
    WebAppListener -- "On Startup, Initializes" --> PersistenceImpl
    WebAppListener -- "On Startup, Triggers" --> Flyway
    Flyway -- "Manages Schema" --> H2DB
```

The system is a layered monolith where the frontend browser interacts via HTTP with a web layer of Java Servlets. These servlets delegate requests to distinct business logic components (Authentication, Library, Math), which in turn interact with the database exclusively through a central `IPersistenceLayer` repository interface, cleanly separating business rules from data access. A completely decoupled Java Swing desktop application also exists, featuring its own TCP/IP server for test automation, highlighting a clear boundary from the main web application.
```