```mermaid
graph TB
    subgraph "External Clients & Testers"
        WebUser[<i class='fa fa-user'></i> Web User]
        TestAutomation[<i class='fa fa-cogs'></i> Test Automation<br>(Python, Java, C#, JS)]
        DesktopUser[<i class='fa fa-user'></i> Desktop User]
    end

    subgraph "Web Application Monolith (WAR)"
        style "Web Application Monolith (WAR)" fill:#f9f9f9,stroke:#333,stroke-width:2px
        
        subgraph "Presentation Layer"
            AuthServlets["Authentication Servlets<br>(/register, /login)"]
            LibraryServlets["Library Servlets<br>(/lend, /registerbook, ...)"]
            MathServlets["Math Servlets<br>(/fibonacci, /ackermann)"]
            AdminServlet["Admin Servlet<br>(/flyway)"]
        end

        subgraph "Business Logic Layer"
            AuthLogic["Authentication Logic<br>(RegistrationUtils)"]
            LibraryLogic["Library Logic<br>(LibraryUtils)"]
            MathLogic["Math Logic<br>(Fibonacci, Ackermann)"]
        end

        subgraph "Data & Infrastructure"
            DAL["Data Access Layer<br>(IPersistenceLayer)"]
            DB[(<i class='fa fa-database'></i> H2 Database)]
            Flyway["Flyway Migrations"]
            WebAppListener["WebAppListener"]
        end
    end

    subgraph "Desktop Application (Standalone)"
        style "Desktop Application (Standalone)" fill:#f2f2f2,stroke:#333,stroke-width:2px
        DesktopUI[<i class='fa fa-desktop'></i> Auto Insurance UI<br>(Java Swing)]
        DesktopLogic["Auto Insurance Logic<br>(AutoInsuranceProcessor)"]
        TCPServer["TCP Socket Server<br>(Port 8000)"]
    end

    %% Web App Interactions
    WebUser -- "HTTP Requests" --> AuthServlets
    WebUser -- "HTTP Requests" --> LibraryServlets
    TestAutomation -- "HTTP API Calls" --> AdminServlet
    TestAutomation -- "HTTP API Calls" --> AuthServlets
    TestAutomation -- "HTTP API Calls" --> LibraryServlets
    TestAutomation -- "HTTP API Calls" --> MathServlets

    AuthServlets -- "Java Calls" --> AuthLogic
    LibraryServlets -- "Java Calls" --> LibraryLogic
    MathServlets -- "Java Calls" --> MathLogic

    AuthLogic -- "Uses Interface" --> DAL
    LibraryLogic -- "Uses Interface" --> DAL
    AdminServlet -- "Invokes" --> DAL

    DAL -- "SQL / JDBC" --> DB
    WebAppListener -- "On Startup" --> DAL
    DAL -- "Triggers" --> Flyway
    Flyway -- "Manages Schema" --> DB

    %% Desktop App Interactions
    DesktopUser -- "GUI Events" --> DesktopUI
    DesktopUI -- "Java Calls" --> DesktopLogic
    DesktopLogic -- "Updates" --> DesktopUI
    TestAutomation -- "Custom TCP Protocol" --> TCPServer
    TCPServer -- "Drives" --> DesktopLogic
```

The system consists of two primary, decoupled components: a 3-tier monolithic web application and a standalone Java Swing desktop application. The monolith exposes HTTP endpoints via Servlets that delegate to distinct business logic packages (Authentication, Library, Math), with stateful domains communicating to a shared H2 database through a single Data Access Layer. The desktop application is entirely self-contained for user interaction but exposes a custom TCP socket server exclusively for external test automation.