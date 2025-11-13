```mermaid
graph TB

%% Clients
subgraph Clients
  Browser[Browser (library.html, endpointcatalog.html)]
  Tests[Automated Tests (Selenium/Cucumber/pytest)]
  DesktopClient[Desktop Automation Client]
end

subgraph SecurityTooling[Security/Proxy (optional)]
  ZAP[OWASP ZAP Proxy]
end

%% Monolithic Web App
subgraph Monolith[Demo Web Application (Monolith) - Servlet/JSP]
  subgraph WebLayer[Web Layer (Servlets, JSP, Static)]
    AuthServlets[Auth Servlets\n(/register, /login)]
    LibraryServlets[Library Servlets\n(/registerbook, /registerborrower, /lend, /book, /borrower, /listavailable)]
    MathServlets[Math Servlets\n(/math, /fibonacci, /ackermann)]
    DbServlet[DbServlet\n(/flyway clean/migrate/reset)]
    H2Console[H2 Console\n(/console/*)]
    JSPs[JSP Views\n(result.jsp, restfulresult.jsp)]
    Static[Static Assets\n(HTML/JS/CSS)]
  end

  subgraph Business[Business / Algorithms]
    AuthBiz[Auth Utils\n(LoginUtils, RegistrationUtils)]
    LibraryBiz[LibraryUtils]
    MathAlgo[Math Algorithms\n(Fibonacci, Ackermann)]
    Helpers[Helpers\n(ServletUtils, StringUtils, CheckUtils)]
  end

  subgraph Persistence[Persistence Layer]
    IPersistence[IPersistenceLayer (DAO API)]
    PL[PersistenceLayer (JDBC micro-ORM)]
    Flyway[Flyway Migrations]
  end

  WebAppListener[WebAppListener\n(startup cleanAndMigrate)]
end

%% Data Store
subgraph DataStore[H2 Database]
  ADMIN[ADMINISTRATIVE.flyway_schema_history]
  AUTH[AUTH.USER]
  LIB[LIBRARY.BOOK / BORROWER / LOAN]
end

%% Desktop App
subgraph DesktopApp[Auto-Insurance Desktop App (Swing)]
  UI[AutoInsuranceUI (Swing)]
  ScriptServer[AutoInsuranceScriptServer (TCP :8000)]
  Processor[AutoInsuranceProcessor (Business Rules)]
end

%% Client -> Web App flows
Browser --> Static
Browser --> AuthServlets
Browser --> LibraryServlets
Browser --> MathServlets
Browser --> DbServlet
Browser --> H2Console

Tests --> ZAP
ZAP --> AuthServlets
ZAP --> LibraryServlets
ZAP --> MathServlets
ZAP --> DbServlet

%% Web Layer -> Views
AuthServlets --> JSPs
LibraryServlets --> JSPs
MathServlets --> JSPs

%% Web Layer -> Business
AuthServlets --> AuthBiz
LibraryServlets --> LibraryBiz
MathServlets --> MathAlgo
DbServlet --> Flyway

%% Startup hook
WebAppListener --> Flyway

%% Business -> Persistence
AuthBiz --> IPersistence
LibraryBiz --> IPersistence
IPersistence --> PL

%% Persistence -> DB
PL --> AUTH
PL --> LIB
Flyway --> ADMIN
Flyway --> AUTH
Flyway --> LIB

%% H2 Console -> DB
H2Console --> ADMIN
H2Console --> AUTH
H2Console --> LIB

%% Desktop app flows
UI --> ScriptServer
DesktopClient --> ScriptServer
ScriptServer --> Processor
```

The monolithic web app groups domain-specific servlets that synchronously call in-process business utilities and a shared persistence layer; JSPs render responses. The persistence layer centralizes all DAO operations against a single H2 database (schemas AUTH, LIBRARY, ADMINISTRATIVE), with Flyway invoked via a startup listener and an admin servlet. External interactions are limited to synchronous HTTP from browsers/tests (optionally via ZAP) and a separate Swing desktop app communicating over a local TCP server, with no coupling between the desktop and web applications.