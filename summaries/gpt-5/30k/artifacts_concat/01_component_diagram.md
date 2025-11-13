```mermaid
graph TB

subgraph Clients
  Browser[Web Browser + JS\n(library.html, endpointcatalog.html)]
  API_Tests[API Tests\n(Python/Java)]
  UI_Tests[Selenium UI Tests]
  DesktopClient[Desktop Automation Client]
end

subgraph AppServer["Tomcat (context /demo)"]
  TomcatNode[Tomcat 9]
  WebAppListener[WebAppListener\n(on startup: clean+migrate DB)]
  subgraph WebApp["Monolithic Web App 'Demo'"]
    subgraph WebLayer["Servlets / Views"]
      AuthServlets[Authentication Servlets\n/register, /login]
      LibraryServlets[Library Servlets\n/registerbook, /registerborrower, /lend,\n/book, /borrower, /listavailable]
      MathServlets[Math Servlets\n/math, /fibonacci, /ackermann]
      DbServlet[DbServlet\n/flyway]
      JSP[result.jsp / restfulresult.jsp]
      H2Console[H2 Console\n/console]
    end
    subgraph Business["Business Logic"]
      AuthUtils[RegistrationUtils, LoginUtils]
      LibraryUtils[LibraryUtils]
      MathAlgos[Algorithms\nFibonacci, Ackermann]
      Helpers[Helpers\nCheckUtils, StringUtils, ServletUtils]
    end
    subgraph Persistence["Persistence"]
      IPersistence[IPersistenceLayer]
      PL[PersistenceLayer]
      Flyway[Flyway Migrations]
    end
  end
end

subgraph Data["Data Stores"]
  H2DB[H2 Database\nSchemas: AUTH, LIBRARY, ADMINISTRATIVE]
  FS[Backup/Restore Scripts]
end

subgraph Desktop["Desktop Application (AutoInsurance)"]
  SwingUI[AutoInsuranceUI (Swing)]
  ScriptServer[AutoInsuranceScriptServer :8000\n(TCP automation)]
  Processor[AutoInsuranceProcessor]
end

subgraph CI["CI/CD & QA Tooling"]
  Jenkins[Jenkins Pipeline]
  Sonar[SonarQube]
  ZAP[OWASP ZAP Proxy]
  JMeter[JMeter]
  H2O[H2O Reports Server]
end

%% Client -> Web App
Browser --> AuthServlets
Browser --> LibraryServlets
Browser --> MathServlets
Browser --> DbServlet
Browser --> H2Console
WebLayer --> JSP
JSP --> Browser

%% Web App internals
AuthServlets --> AuthUtils
LibraryServlets --> LibraryUtils
MathServlets --> MathAlgos
DbServlet --> Flyway

AuthUtils -.uses.-> Helpers
LibraryUtils -.uses.-> Helpers
MathAlgos -.uses.-> Helpers

AuthUtils --> IPersistence
LibraryUtils --> IPersistence
IPersistence --> PL
PL --> H2DB
Flyway --> H2DB
WebAppListener --> Flyway
H2Console --> H2DB
PL -.backup/restore.-> FS

%% Desktop app interactions
DesktopClient --> ScriptServer
ScriptServer --> SwingUI
SwingUI --> Processor

%% CI/CD interactions
Jenkins --> TomcatNode
Jenkins --> Sonar
Jenkins --> JMeter
Jenkins --> ZAP
Jenkins --> H2O

%% Test and proxy traffic
UI_Tests -.controls.-> Browser
API_Tests -.HTTP via proxy.-> ZAP
ZAP --> AuthServlets
ZAP --> LibraryServlets
ZAP --> MathServlets
JMeter --> AuthServlets
JMeter --> LibraryServlets
JMeter --> MathServlets
```

The monolith cleanly separates presentation (Servlets/JSP) from business utilities and a shared persistence layer that mediates all database access to a single H2 instance (schemas AUTH/LIBRARY), with Flyway migrations invoked on startup and via an admin servlet. Authentication and Library are cohesive feature modules that share the PersistenceLayer, while Math endpoints are stateless and compute-only; clients interact synchronously over HTTP, sometimes routed through a ZAP proxy during CI. The AutoInsurance desktop app is an independent process exposing a TCP automation server, and Jenkins orchestrates build/deploy, tests, and quality tools against the Tomcat-hosted web app.