```mermaid
graph TB

%% Clients
subgraph Clients
  Browser[User Browser\n(static HTML+JS)]
  API_Tests[API Tests\n(Python requests)]
  UI_Tests[Selenium UI tests]
end

%% Web Application on Tomcat
subgraph WebApp[Java Web App on Tomcat]
  WebAppListener[WebAppListener\n(startup hook)]

  subgraph Endpoints[Servlet Endpoints]
    AuthServlets[Auth Servlets\n(Login, Register)]
    LibraryServlets[Library Servlets\n(Register/Lend/List/Search)]
    MathServlets[Math/Ack/Fib Servlets]
    DbAdmin[DbServlet (/flyway)]
    H2Console[H2 Web Console (/console)]
  end

  subgraph Business[Business Logic]
    AuthService[Authentication Utils\n(LoginUtils, RegistrationUtils)]
    LibraryService[LibraryUtils]
    MathService[Math Utils\n(Calculator, Fibonacci, Ackermann)]
    Helpers[Helpers\n(CheckUtils, StringUtils, ServletUtils)]
  end

  subgraph Persistence[Persistence Layer]
    IPL[IPersistenceLayer (interface)]
    PL[PersistenceLayer + SqlData/ParameterObject]
  end
end

%% Data
subgraph Data[Data Stores / Migration]
  H2DB[(H2 Database\n(in-memory))]
  Flyway[Flyway Migrations]
end

%% Desktop demo application
subgraph DesktopApp[Desktop Demo (Swing)]
  DesktopClient[Script Client / DesktopTester]
  ScriptServer[Script Server\n(TCP 8000)]
  AutoUI[AutoInsuranceUI (Swing)]
  Processor[AutoInsuranceProcessor]
end

%% Client -> Endpoints
Browser -- HTTP GET/POST --> AuthServlets
Browser -- HTTP GET/POST --> LibraryServlets
Browser -- HTTP GET/POST --> MathServlets
Browser -- HTTP GET --> H2Console

API_Tests -- HTTP --> AuthServlets
API_Tests -- HTTP --> LibraryServlets
API_Tests -- HTTP --> MathServlets

UI_Tests -- drives --> Browser

%% Endpoints -> Business
AuthServlets --> AuthService
LibraryServlets --> LibraryService
MathServlets --> MathService
DbAdmin --> PL

%% Helpers (cross-cutting)
Helpers -. used by .-> AuthServlets
Helpers -. used by .-> LibraryServlets
Helpers -. used by .-> MathServlets
Helpers -. used by .-> AuthService
Helpers -. used by .-> LibraryService

%% Business -> Persistence
AuthService --> IPL
LibraryService --> IPL
IPL --> PL

%% Persistence -> Data
PL -- JDBC --> H2DB
PL -- run --> Flyway
WebAppListener -- cleanAndMigrateDatabase --> PL

%% Desktop flow
DesktopClient -- TCP:8000 --> ScriptServer
ScriptServer --> AutoUI
AutoUI --> Processor
```

Rationale:
The system separates concerns into servlet endpoints (controllers), business services (Utils classes), and a persistence layer (IPersistenceLayer/PersistenceLayer with a micro-ORM) against an in-memory H2 database migrated by Flyway at startup and via DbServlet. Clients interact over HTTP from the browser (static HTML/JS) and automated tests; a separate desktop demo exposes a TCP script server to drive a Swing UI that delegates to a processor, illustrating decoupled, testable interactions.