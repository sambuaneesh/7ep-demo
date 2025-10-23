```mermaid
graph TB
    subgraph "External Actors & Tools"
        User
        UITests["UI Tests (Selenium, Behave, etc.)"]
        APITests["API Tests (Python)"]
        PerfSecTests["Performance & Security Tests (JMeter, ZAP)"]
        DesktopUITests["Desktop UI Tests"]
    end

    subgraph "Monolithic Web Application (7ep-demo)"
        WebUI["Web UI (HTML, JS, CSS, JSP)"]
        WebAPI["Web API (Servlets)"]
        
        subgraph "Business Logic Services"
            AuthService["Authentication Service"]
            LibraryService["Library Service"]
            MathService["Mathematics Service"]
            OtherServices["Expenses, CartesianProduct, etc."]
        end

        Persistence["Persistence Layer"]
        Database["H2 Database (managed by Flyway)"]
    end

    subgraph "Desktop Application (desktop_app)"
        DesktopApp["Auto Insurance App (Java Swing)"]
        DesktopLogic["AutoInsuranceProcessor"]
        ScriptingServer["Scripting Server (Socket)"]
    end

    %% -- Interactions --

    %% User and Test Interactions with Web App
    User -- "Interacts via Browser" --> WebUI
    UITests -- "Drives Browser" --> WebUI
    WebUI -- "HTTP/AJAX" --> WebAPI
    APITests -- "HTTP Requests" --> WebAPI
    PerfSecTests -- "HTTP Requests" --> WebAPI
    
    %% Web App Internal Flow
    WebAPI --> AuthService
    WebAPI --> LibraryService
    WebAPI --> MathService
    WebAPI --> OtherServices
    AuthService --> Persistence
    LibraryService --> Persistence
    Persistence <--> Database

    %% Desktop App Interactions
    User -- "Uses UI" --> DesktopApp
    DesktopApp --> DesktopLogic
    DesktopApp -- "Contains" --> ScriptingServer
    DesktopUITests -- "Socket Commands" --> ScriptingServer
```

The system comprises a monolithic Java web application and a distinct Java Swing desktop application. The web application follows a layered architecture where a browser-based UI communicates via HTTP with servlets that orchestrate domain-specific business logic (e.g., Library, Authentication), all utilizing a central persistence layer to interact with an H2 database. The separate desktop application is self-contained and exposes a socket-based server, enabling remote control for automated UI testing.