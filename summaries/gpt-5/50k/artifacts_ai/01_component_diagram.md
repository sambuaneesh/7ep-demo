```mermaid
graph TB
  B[Web Browser]

  subgraph SC[Tomcat (Servlet Container) — Monolith WAR]
    Static[Static pages & JS\n(index.html, library.html, catalog.js, library.js)]
    JSP[result.jsp / restfulresult.jsp]

    subgraph Servlets[Servlet Controllers]
      AuthS[Authentication\n/LoginServlet, RegisterServlet]
      LibS[Library\n/RegisterBook, RegisterBorrower, Lend, Book, Borrower, ListAvailable]
      MathS[Mathematics\n/Math, /Fibonacci, /Ackermann]
      DbS[DbServlet\n(/flyway)]
      H2Cons[H2 Console\n(/console/*)]
    end

    Listener[WebAppListener\n(@WebListener)]
  end

  subgraph Business[Business Layer (Utils & Algorithms)]
    AuthU[RegistrationUtils, LoginUtils]
    LibU[LibraryUtils]
    MathU[Calculator, Fibonacci*, Ackermann*]
  end

  subgraph Persist[Persistence Layer]
    IPL[IPersistenceLayer (API)]
    PL[PersistenceLayer (JDBC micro‑ORM)]
    Fly[Flyway]
  end

  subgraph DB[H2 Database (in‑memory)]
    H2[(H2 DB)\nSchemas: AUTH, LIBRARY, ADMINISTRATIVE]
  end

  subgraph Desktop[Desktop App (Swing)]
    UI[AutoInsuranceUI]
    Script[Automation Script Server\n(TCP 8000)]
  end

  %% Client -> Web
  B -->|HTTP GET| Static
  B -->|XHR/HTTP| AuthS
  B -->|XHR/HTTP| LibS
  B -->|XHR/HTTP| MathS
  B -->|HTTP| H2Cons
  JSP -->|HTML/text response| B

  %% Web -> Business
  AuthS -->|delegate| AuthU
  LibS -->|delegate| LibU
  MathS -->|delegate| MathU

  %% Business -> Persistence
  AuthU -->|calls| IPL
  LibU -->|calls| IPL
  IPL -->|implemented by| PL

  %% Persistence -> DB
  PL -->|JDBC| H2
  PL --> Fly
  Fly -->|migrations| H2

  %% Admin/startup paths
  DbS -->|clean/migrate| PL
  Listener -->|on startup: cleanAndMigrateDatabase()| PL

  %% H2 Console direct DB access
  H2Cons -->|JDBC| H2

  %% Desktop app (standalone)
  Script -->|UI automation commands| UI
```

Component boundaries follow the layered monolith: HTTP servlets route to domain-specific Utils (Authentication, Library, Math), which in turn use a shared PersistenceLayer (via IPersistenceLayer) backed by H2 and managed by Flyway. All calls are synchronous and in-process; servlets forward to JSP views for responses, while static pages/JS in the WAR issue XHR requests to the same servlets. The desktop Swing app is a separate, standalone component with a TCP automation server and no runtime coupling to the web application.