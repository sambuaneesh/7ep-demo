```mermaid
graph TB
  Browser[Browser / UI\n(index.html, library.html, endpointcatalog.html, catalog.js, library.js)]

  subgraph WebApp[Monolithic Web Application (/demo) on Tomcat]
    subgraph Presentation[Presentation]
      AuthServlets[Auth Servlets\n- LoginServlet (/login)\n- RegisterServlet (/register)]
      LibraryServlets[Library Servlets\n- RegisterBook (/registerbook)\n- RegisterBorrower (/registerborrower)\n- Lend (/lend)\n- Book List/Search (/book)\n- Borrower List (/borrower)\n- ListAvailable (/listavailable)]
      MathServlets[Math Servlets\n- Math (/math)\n- Fibonacci (/fibonacci)\n- Ackermann (/ackermann)]
      DbServlet[DbServlet (/flyway)]
      H2Console[H2 Console Servlet (/console/*)]
      ServletUtils[ServletUtils (forwarding)]
      JSPs[JSP Views\nresult.jsp, restfulresult.jsp]
    end

    subgraph Domain[Business / Domain]
      AuthUtils[Auth Utils\n(LoginUtils, RegistrationUtils)]
      LibraryUtils[Library Utils]
      MathUtils[Math Utils\n(Calculator, Fibonacci*, Ackermann*)]
    end

    subgraph Persistence[Persistence Layer]
      IP[IPersistenceLayer (interface)]
      PL[PersistenceLayer (JDBC + micro ORM)]
      ORM[ORM Helpers\n(SqlData, ParameterObject, Templates)]
      Flyway[Flyway Migrations]
    end

    WebAppListener[WebAppListener (startup clean+migrate)]
  end

  H2[(H2 Database\nSchemas: AUTH, LIBRARY)]
  subgraph Desktop[Desktop Demo App]
    AutoUI[AutoInsurance UI (Swing)]
    ScriptServer[Automation Script Server\nTCP 127.0.0.1:8000]
  end
  DesktopClients[Automation Clients\n(ScriptClient, DesktopTester, etc.)]

  %% External interactions
  Browser -->|HTTP /login,/register| AuthServlets
  Browser -->|HTTP /registerbook,/registerborrower,/lend,/book,/borrower,/listavailable| LibraryServlets
  Browser -->|HTTP /math,/fibonacci,/ackermann| MathServlets
  Browser -->|HTTP /flyway| DbServlet
  Browser -->|HTTP /console| H2Console

  %% Presentation to domain
  AuthServlets --> AuthUtils
  LibraryServlets --> LibraryUtils
  MathServlets --> MathUtils

  %% View rendering
  AuthServlets --> ServletUtils
  LibraryServlets --> ServletUtils
  MathServlets --> ServletUtils
  DbServlet --> ServletUtils
  ServletUtils --> JSPs

  %% Domain to persistence
  AuthUtils --> IP
  LibraryUtils --> IP
  IP --> PL
  PL --> ORM
  PL -->|JDBC| H2

  %% DB admin paths
  WebAppListener --> Flyway
  DbServlet --> Flyway
  Flyway -->|clean/migrate| H2
  H2Console -->|JDBC| H2

  %% Desktop app and its automation
  AutoUI -.starts.-> ScriptServer
  DesktopClients -->|TCP 8000| ScriptServer
```

The monolith follows a layered structure: servlets handle HTTP and forward to JSPs via ServletUtils, business logic resides in Utils classes, and stateful domains (Auth, Library) access the database through IPersistenceLayer implemented by a JDBC-based PersistenceLayer and Flyway-managed H2 schemas. Math endpoints are compute-only and do not use persistence, while startup and admin operations run Flyway (via WebAppListener and /flyway). A separate desktop Swing app exposes a TCP automation server; apart from browser HTTP and this local TCP channel, all other interactions are in-process.