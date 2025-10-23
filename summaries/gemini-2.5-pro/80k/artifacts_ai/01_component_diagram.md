```markdown
```mermaid
graph TB
    subgraph "Client"
        User[Browser/User]
        UI[Frontend UI <br> (HTML, library.js)]
    end

    subgraph "Web Application (Monolith)"
        subgraph "Presentation Layer (Servlets)"
            AuthServlets[Authentication Servlets <br> /register, /login]
            LibServlets[Library Servlets <br> /lend, /book, /listavailable]
            MathServlets[Mathematics Servlets <br> /math, /fibonacci]
            DbAdminServlet[DB Admin Servlet <br> /flyway]
        end

        subgraph "Business Logic Layer"
            AuthService[Authentication Service <br> (RegistrationUtils, LoginUtils)]
            LibService[Library Service <br> (LibraryUtils)]
            MathService[Mathematics Service <br> (Calculator, Fibonacci)]
        end

        subgraph "Data Access Layer"
            PersistenceGW[IPersistenceLayer <br> (Gateway Interface)]
            PersistenceImpl[PersistenceLayer <br> (H2/JDBC Implementation)]
        end

        subgraph "Database"
            DB[(H2 In-Memory DB <br> with Flyway Migrations)]
        end

        subgraph "Application Lifecycle"
            Lifecycle[WebAppListener <br> (Tomcat Context Listener)]
        end
    end

    subgraph "Standalone Desktop Application"
        DesktopUI[AutoInsurance UI <br> (Java Swing)]
        DesktopLogic[AutoInsurance Processor <br> (Business Rules)]
        DesktopTestServer[AutoInsurance Script Server <br> (For Test Automation)]
    end

    %% --- Web Application Interactions ---
    User -->|Interacts with| UI
    UI -->|AJAX / Form POSTs| AuthServlets
    UI -->|AJAX / Form POSTs| LibServlets
    UI -->|AJAX / Form POSTs| MathServlets

    AuthServlets -->|Delegates to| AuthService
    LibServlets -->|Delegates to| LibService
    MathServlets -->|Delegates to| MathService

    AuthService -->|Uses Contract| PersistenceGW
    LibService -->|Uses Contract| PersistenceGW
    DbAdminServlet -->|Invokes Admin Functions| PersistenceGW
    Lifecycle -->|Initializes on Startup| PersistenceGW

    PersistenceImpl -->|Implements| PersistenceGW
    PersistenceImpl -->|Executes SQL via JDBC| DB

    %% --- Standalone Application Interactions ---
    DesktopUI -->|Uses| DesktopLogic
    DesktopTestServer -->|Drives for Testing| DesktopUI

```

The diagram depicts a classic layered monolithic architecture, separating the system into a Frontend UI, Presentation (Servlets), Business Logic, and a unified Persistence layer. Communication follows a strict top-down flow, where Servlets delegate to business services that exclusively use the `IPersistenceLayer` gateway for all database interactions. The completely separate Desktop Application is also illustrated to show its isolation from the primary web application.
```