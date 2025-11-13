```mermaid
graph TB

%% External clients
subgraph ext[External Clients]
  user[Web Browser/User]
  api[Automated Tests (Selenium/Behave/PyTest)]
  autoClient[Desktop Automation Client]
end

%% Monolithic web application
subgraph web[Monolithic Web App (Tomcat @ http://localhost:8080/demo)]
  direction TB

  subgraph web.controllers[Servlets (Controllers)]
    authSrv[Authentication Servlets\n- LoginServlet\n- RegisterServlet]
    libSrv[Library Servlets\n- RegisterBook/RegisterBorrower/Lend\n- BookList/BorrowerList/ListAvailable]
    mathSrv[Mathematics Servlets\n- MathServlet/FibServlet/AckServlet]
    dbSrv[DbServlet (/flyway)]
  end

  subgraph web.services[Business/Domain Utils]
    authUtils[Authentication Utils\n- LoginUtils/RegistrationUtils]
    libUtils[LibraryUtils]
    mathAlgos[Math/Expenses (stateless)\n- Calculator/Fibonacci/Ackermann\n- AlcoholCalculator]
  end

  subgraph web.persistence[Persistence]
    ipl[IPersistenceLayer (interface)]
    pl[PersistenceLayer (JDBC + Flyway)]
  end

  subgraph web.views[Views & UI]
    static[Static Pages & JS\n- index.html/library.html\n- catalog.js/library.js]
    jsp[JSP Views\n- result.jsp\n- restfulresult.jsp]
    h2console[H2 Console Servlet (/console)]
  end

  subgraph web.helpers[Cross-cutting Helpers]
    helpers[CheckUtils, StringUtils, ServletUtils]
  end

  listener[WebAppListener]
end

%% Data store
subgraph data[Data Stores]
  h2[(H2 Database\nSchemas: AUTH, LIBRARY, ADMINISTRATIVE)]
end

%% Desktop application
subgraph desktop[Desktop Auto Insurance Demo]
  direction TB
  scriptSrv[AutoInsuranceScriptServer\n(TCP 8000)]
  ui[AutoInsuranceUI (Swing)]
  rules[AutoInsuranceProcessor (Rules Engine)]
end

%% External -> Web app interactions
user -->|Browse| static
user -->|HTTP 8080 (forms)| authSrv
user -->|HTTP 8080 (forms)| libSrv
user -->|HTTP 8080 (forms)| mathSrv
user -->|HTTP 8080| h2console

api -->|HTTP 8080| authSrv
api -->|HTTP 8080| libSrv
api -->|HTTP 8080| mathSrv
api -->|HTTP 8080| dbSrv

static -->|"AJAX (XHR)"| libSrv
static -->|"AJAX (XHR)"| mathSrv

%% Controller -> Service -> Persistence
authSrv --> authUtils
libSrv --> libUtils
mathSrv --> mathAlgos
dbSrv --> pl

authUtils --> ipl
libUtils --> ipl
ipl --> pl
pl -->|JDBC| h2

%% Views
authSrv -->|forward| jsp
libSrv -->|forward| jsp
mathSrv -->|forward| jsp
dbSrv -->|forward| jsp

%% Startup lifecycle
listener -. "on startup: clean+migrate" .-> pl

%% Cross-cutting helpers
helpers -.-> web.controllers
helpers -.-> web.services

%% Desktop app interactions
autoClient -->|TCP 8000| scriptSrv
scriptSrv --> ui
ui --> rules
```

The monolith is layered: HTTP servlets (controllers) delegate to domain-specific Utils, which call a centralized IPersistenceLayer implemented by PersistenceLayer against H2; mathematics/expenses are stateless and bypass persistence. Servlets forward results to JSPs, static pages use AJAX to the same controllers, and Flyway operations are triggered on startup (WebAppListener) and via the DbServlet; the desktop Auto Insurance demo is a separate Swing app driven over a simple TCP automation protocol on port 8000.