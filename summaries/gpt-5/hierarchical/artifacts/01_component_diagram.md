```mermaid
graph TB

  subgraph Clients_and_Tests
    User[User/Browser]
    Selenium[UI Tests (Selenium/HtmlUnit/Selenified)]
    ApiCalls[ApiCalls (HTTP client)]
    Cucumber[BDD StepDefs]
  end

  subgraph Infrastructure
    Listener[Tomcat WebAppListener]
    AdminWeb[DbServlet (admin)]
  end

  subgraph Web_Tier
    AuthWeb[Auth Servlets\n(Register, Login)]
    LibWeb[Library Servlets\n(Register/Lend/Search/List)]
    MathWeb[Math Servlets\n(Math/Fib/Ack)]
    JSP[Shared JSP Views]
  end

  subgraph Services_and_Domain
    AuthSvc[Auth Utilities\n(RegistrationUtils, LoginUtils)]
    LibSvc[LibraryUtils]
    MathAlgo[Math Algorithms\n(Fibonacci, Ackermann, Calculator)]
    AuthDomain[Auth Domain Objects]
    LibDomain[Library Domain Objects]
    Helpers[Helpers\n(Check/String/Date/Servlet Utils)]
    Nbvcxz[Nbvcxz\nPassword Strength]
  end

  subgraph Persistence
    Port[IPersistenceLayer (Port)]
    Adapter[PersistenceLayer (JDBC/H2) (Adapter)]
    Flyway[Flyway]
    H2[(H2 Database)]
  end

  subgraph Educational_AutoInsurance
    AutoUI[AutoInsuranceUI (Swing)]
    AutoProc[AutoInsuranceProcessor (Rules Engine)]
    ScriptClient[Script Client / DesktopTester]
  end

  %% User and test traffic
  User --> AuthWeb
  User --> LibWeb
  User --> MathWeb

  Selenium --> AuthWeb
  Selenium --> LibWeb
  Selenium --> MathWeb
  Selenium -->|reset DB| AdminWeb

  ApiCalls -->|POST| AuthWeb
  ApiCalls -->|POST| LibWeb

  Cucumber --> LibSvc
  Cucumber --> AuthSvc
  Cucumber -->|clean/migrate, seed| Adapter

  %% Web to services and views
  AuthWeb -->|delegate| AuthSvc
  LibWeb -->|delegate| LibSvc
  MathWeb -->|invoke| MathAlgo

  AuthWeb -->|forward| JSP
  LibWeb -->|forward| JSP
  MathWeb -->|forward| JSP
  AdminWeb -->|forward| JSP

  %% Services to persistence
  AuthSvc -->|storage port| Port
  LibSvc -->|storage port| Port
  AdminWeb -->|clean/migrate| Port

  Port -->|implemented by| Adapter
  Adapter -->|SQL| H2
  Adapter -.->|migrations| Flyway

  Listener -->|cleanAndMigrate at startup| Port

  %% Cross-cutting helpers
  AuthWeb -.->|Servlet forwarding, validation| Helpers
  LibWeb  -.->|Servlet forwarding, validation| Helpers
  MathWeb -.->|Servlet forwarding| Helpers
  AuthSvc -.->|validation/strings| Helpers
  LibSvc  -.->|validation/strings| Helpers
  Adapter -.->|validation/strings| Helpers

  %% Domain usage (implicit within services)
  AuthSvc -.-> AuthDomain
  LibSvc  -.-> LibDomain

  %% External password strength
  AuthSvc -->|evaluate| Nbvcxz

  %% Educational auto-insurance slice
  AutoUI -->|invokes| AutoProc
  ScriptClient -->|TCP script| AutoUI
```

The system follows a hexagonal pattern: thin web controllers delegate business logic to services/utilities, which depend on a persistence port (IPersistenceLayer) implemented by a JDBC/H2 adapter; results are rendered via shared JSPs through ServletUtils. Environment determinism is provided by a Tomcat WebAppListener and an admin DbServlet that drive Flyway clean/migrate, while cross-cutting helpers standardize validation and forwarding. Educational modules (math, auto-insurance) expose pure computation cores behind thin controllers/UIs, and tests drive flows either via HTTP (Selenium/HtmlUnit/ApiCalls) or directly via utilities/persistence in BDD.