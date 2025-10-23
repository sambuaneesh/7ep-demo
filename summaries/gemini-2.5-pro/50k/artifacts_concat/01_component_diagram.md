```markdown
```mermaid
graph TB
    subgraph "Users"
        WebUser["Web User </br> (Browser)"]
        DesktopUser["Desktop User"]
    end

    subgraph "Web Application Monolith"
        direction TB
        subgraph "API Layer (Servlets)"
            AuthServlet["Authentication Servlets </br> (/login, /register)"]
            LibraryServlet["Library Servlets </br> (/registerbook, /lend, ...)"]
            MathServlet["Mathematics Servlets </br> (/math, /fibonacci, ...)"]
            AdminServlet["Admin Servlet </br> (/flyway)"]
        end

        subgraph "Business Logic Layer"
            AuthLogic["AuthenticationUtils"]
            LibraryLogic["LibraryUtils"]
            MathLogic["Mathematics Logic </br> (Fibonacci, Ackermann)"]
        end

        subgraph "Data Access Layer"
            PersistenceLayer["PersistenceLayer (DAO)"]
        end

        AuthServlet -- "calls" --> AuthLogic
        LibraryServlet -- "calls" --> LibraryLogic
        MathServlet -- "calls" --> MathLogic

        AuthLogic -- "uses" --> PersistenceLayer
        LibraryLogic -- "uses" --> PersistenceLayer
        AdminServlet -- "uses" --> PersistenceLayer
    end

    subgraph "Desktop Application"
        direction TB
        SwingUI["Swing UI"]
        InsuranceLogic["Auto Insurance Logic"]
        TCPServer["TCP Test Server </br> (Port 8000)"]

        SwingUI --> InsuranceLogic
    end

    subgraph "CI/CD & Testing Infrastructure"
        direction LR
        E2E_Tests["E2E Tests </br> (Selenium, Behave)"]
        DAST["DAST Proxy </br> (OWASP ZAP)"]
        Perf_Tests["Performance Tests </br> (JMeter)"]
    end
    
    Database[("H2 Database")]

    WebUser -- "HTTP Requests" --> AuthServlet
    WebUser -- "HTTP Requests" --> LibraryServlet
    WebUser -- "HTTP Requests" --> MathServlet

    PersistenceLayer -- "JDBC" --> Database

    DesktopUser -- "Interacts with" --> SwingUI

    E2E_Tests -- "HTTP Requests via" --> DAST
    DAST -- "Proxied HTTP Requests" --> LibraryServlet
    DAST -- "Proxied HTTP Requests" --> AuthServlet
    E2E_Tests -- "TCP Commands" --> TCPServer

    Perf_Tests -- "HTTP Load" --> LibraryServlet
    Perf_Tests -- "HTTP Load" --> MathServlet
```

The diagram illustrates a system composed of a 3-tier monolithic web application and a separate, standalone desktop application. Within the monolith, logically distinct domains like Authentication and Library are tightly coupled by communicating through a single Data Access Object (DAO) to a shared H2 database, representing a classic layered architecture. The CI/CD pipeline components interact with both applications for comprehensive automated testing, using HTTP for the web monolith and a custom TCP protocol for the desktop client.
```