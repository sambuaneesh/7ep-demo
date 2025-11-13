## Workflow 1: User Registration (POST /demo/register)

Purpose and triggers:
- Purpose: Register a new user with password policy and uniqueness checks.
- Trigger: User submits register form on library.html or POSTs to /demo/register.

Communication patterns:
- Synchronous HTTP request/response (Browser → Servlet).
- In-process calls (Servlet → Business utils → Persistence).
- JDBC queries/inserts (Persistence → H2 DB).
- JSP forwarding (Servlet → result.jsp).
- No transactions; auto-commit per statement.

```mermaid
sequenceDiagram
  autonumber
  actor U as User (Browser)
  participant T as Tomcat/Gretty
  participant RS as RegisterServlet
  participant RU as RegistrationUtils
  participant PL as PersistenceLayer (JDBC)
  participant DB as H2 Database
  participant JSP as result.jsp

  U->>T: POST /demo/register (username, password)
  T->>RS: dispatch
  RS->>RU: processRegistration(username, password)

  alt Validate inputs
    RU->>RU: Check non-empty username/password
    RU-->>RS: If empty -> RegistrationResult(EMPTY_*)
    RS->>JSP: forward with result="successfully registered: false"
    JSP-->>U: HTML render
  else Unique username check
    RU->>PL: searchForUserByName(username)
    PL->>DB: SELECT id FROM auth.user WHERE name=?
    DB-->>PL: Row or empty
    PL-->>RU: Optional<User>
    alt Already exists
      RU-->>RS: RegistrationResult(ALREADY_REGISTERED, false)
      RS->>JSP: forward result="successfully registered: false"
      JSP-->>U: HTML
    else Password policy
      RU->>RU: check length 10–100
      RU->>RU: entropy via Nbvcxz
      alt Bad password
        RU-->>RS: RegistrationResult(BAD_PASSWORD, false) + PasswordResult
        RS->>JSP: forward result with message
        JSP-->>U: HTML
      else Persist user + password hash
        RU->>PL: saveNewUser(username)
        PL->>DB: INSERT INTO auth.user(name) VALUES(?)
        DB-->>PL: generated id
        PL-->>RU: User(id, name)
        RU->>PL: updateUserWithPassword(userId, SHA-256(password))
        PL->>DB: UPDATE auth.user SET password_hash=? WHERE id=?
        DB-->>PL: updated rows
        PL-->>RU: ok
        RU-->>RS: RegistrationResult(SUCCESSFULLY_REGISTERED, true)
        RS->>JSP: forward result="successfully registered: true"
        JSP-->>U: HTML
      end
    end
  end
```

---

## Workflow 2: User Login (POST /demo/login)

Purpose and triggers:
- Purpose: Validate credentials and grant/deny access.
- Trigger: User submits login form or POSTs to /demo/login.

Communication patterns:
- Synchronous HTTP request/response (Browser → Servlet).
- In-process method calls.
- JDBC SELECT for credential verification.
- JSP forwarding (result.jsp).

```mermaid
sequenceDiagram
  autonumber
  actor U as User (Browser)
  participant T as Tomcat/Gretty
  participant LS as LoginServlet
  participant LU as LoginUtils
  participant PL as PersistenceLayer
  participant DB as H2 Database
  participant JSP as result.jsp

  U->>T: POST /demo/login (username, password)
  T->>LS: dispatch
  LS->>LU: isUserRegistered(username, password)

  alt Validate inputs
    LU->>LU: check non-empty values
    alt Missing username/password
      LU-->>LS: "no username/password provided"
      LS->>JSP: forward result="access denied"
      JSP-->>U: HTML
    else Hash and check
      LU->>PL: areCredentialsValid(username, SHA-256(password))
      PL->>DB: SELECT id FROM auth.user WHERE name=? AND password_hash=?
      DB-->>PL: Row or empty
      PL-->>LU: Optional<Boolean> (present when valid)
      alt Present==true
        LU-->>LS: valid
        LS->>JSP: forward result="access granted"
        JSP-->>U: HTML
      else Not valid
        LU-->>LS: invalid
        LS->>JSP: forward result="access denied"
        JSP-->>U: HTML
      end
    end
  end
```

---

## Workflow 3: Library Registration (Books and Borrowers)

Purpose and triggers:
- Purpose: Add books and borrowers to the catalog.
- Trigger: User submits forms on library.html or POSTs to /demo/registerbook or /demo/registerborrower.

Communication patterns:
- Synchronous HTTP.
- In-process business logic.
- JDBC insert/select checks.
- JSP forwarding.

```mermaid
sequenceDiagram
  autonumber
  actor U as User (Browser)
  participant T as Tomcat
  participant RBS as LibraryRegisterBookServlet
  participant RBR as LibraryRegisterBorrowerServlet
  participant LU as LibraryUtils
  participant PL as PersistenceLayer
  participant DB as H2 DB
  participant JSP as result.jsp

  par Register Book
    U->>T: POST /demo/registerbook (book=title)
    T->>RBS: dispatch
    RBS->>LU: registerBook(title)
    LU->>PL: searchBooksByTitle(title)
    PL->>DB: SELECT id, title FROM library.book WHERE title=?
    DB-->>PL: rows or empty
    alt Exists
      LU-->>RBS: ALREADY_REGISTERED_BOOK
      RBS->>JSP: forward result="ALREADY_REGISTERED_BOOK"
      JSP-->>U: HTML
    else Insert
      LU->>PL: saveNewBook(title)
      PL->>DB: INSERT INTO library.book(title) VALUES(?)
      DB-->>PL: generated id
      LU-->>RBS: SUCCESS
      RBS->>JSP: forward result="SUCCESS"
      JSP-->>U: HTML
    end
  and Register Borrower
    U->>T: POST /demo/registerborrower (borrower=name)
    T->>RBR: dispatch
    RBR->>LU: registerBorrower(name)
    LU->>PL: searchBorrowerDataByName(name)
    PL->>DB: SELECT id, name FROM library.borrower WHERE name=?
    DB-->>PL: rows or empty
    alt Exists
      LU-->>RBR: ALREADY_REGISTERED_BORROWER
      RBR->>JSP: forward result="ALREADY_REGISTERED_BORROWER"
      JSP-->>U: HTML
    else Insert
      LU->>PL: saveNewBorrower(name)
      PL->>DB: INSERT INTO library.borrower(name) VALUES(?)
      DB-->>PL: generated id
      LU-->>RBR: SUCCESS
      RBR->>JSP: forward result="SUCCESS"
      JSP-->>U: HTML
    end
  end
```

---

## Workflow 4: Lending a Book (POST /demo/lend) with UI Prefetch

Purpose and triggers:
- Purpose: Lend a book to a borrower if both exist and book not already checked out.
- Trigger: User selects borrower/book via dynamic UI, then POSTs to /demo/lend.

Communication patterns:
- Asynchronous AJAX (prefetch lists).
- Synchronous HTTP for lending.
- In-process logic with multiple DB reads/writes.
- JSP forwarding.

```mermaid
sequenceDiagram
  autonumber
  actor U as User (Browser - library.html)
  participant JS as library.js
  participant T as Tomcat
  participant LAS as LibraryLendServlet
  participant LU as LibraryUtils
  participant PL as PersistenceLayer
  participant DB as H2 DB
  participant JSP as result.jsp

  rect rgb(235, 245, 255)
  note over U,JS: Prefetch for dynamic UI (autocomplete/select)
  U->>JS: Load page
  JS->>T: GET /demo/listavailable
  T-->>JS: JSON-like books (restfulresult.jsp)
  JS->>T: GET /demo/borrower
  T-->>JS: JSON-like borrowers (restfulresult.jsp)
  end

  U->>T: POST /demo/lend (book=title, borrower=name)
  T->>LAS: dispatch
  LAS->>LU: lendBook(title, name)

  alt Validate inputs
    LU->>LU: check non-empty title/name
    alt Missing
      LU-->>LAS: NO_*_PROVIDED
      LAS->>JSP: forward result with error enum
      JSP-->>U: HTML
    else Resolve entities
      LU->>PL: searchBooksByTitle(title)
      PL->>DB: SELECT id, title FROM library.book WHERE title=?
      DB-->>PL: rows
      PL-->>LU: Optional<Book>
      LU->>PL: searchBorrowerDataByName(name)
      PL->>DB: SELECT id, name FROM library.borrower WHERE name=?
      DB-->>PL: rows
      PL-->>LU: Optional<Borrower>
      alt Book or borrower missing
        LU-->>LAS: BOOK_NOT_REGISTERED or BORROWER_NOT_REGISTERED
        LAS->>JSP: forward result enum
        JSP-->>U: HTML
      else Check existing loan
        LU->>PL: searchForLoanByBook(book.id)
        PL->>DB: SELECT loan.id, loan.borrow_date, loan.borrower FROM library.loan WHERE loan.book=?
        DB-->>PL: row(s) or empty
        alt Already loaned
          LU-->>LAS: BOOK_CHECKED_OUT
          LAS->>JSP: forward result enum
          JSP-->>U: HTML
        else Create new loan
          LU->>PL: createLoan(book, borrower, today)
          PL->>DB: INSERT INTO library.loan(book, borrower, borrow_date) VALUES(?,?,CURDATE)
          DB-->>PL: generated id
          LU-->>LAS: SUCCESS
          LAS->>JSP: forward result="SUCCESS"
          JSP-->>U: HTML
        end
      end
    end
  end
```

---

## Workflow 5: Catalog Search (GET /demo/book and /demo/borrower)

Purpose and triggers:
- Purpose: List all or search by id/title (book) or id/name (borrower).
- Trigger: GET requests from endpointcatalog.html or tests.

Communication patterns:
- Synchronous HTTP.
- In-process logic.
- JDBC selects.
- JSP forwarding (restfulresult.jsp) with JSON-like arrays.

```mermaid
sequenceDiagram
  autonumber
  actor C as Client (Browser/Test)
  participant T as Tomcat
  participant BSS as LibraryBookListSearchServlet
  participant BRS as LibraryBorrowerListSearchServlet
  participant LU as LibraryUtils
  participant PL as PersistenceLayer
  participant DB as H2 DB
  participant JSP as restfulresult.jsp

  par /demo/book
    C->>T: GET /demo/book?(id|title|none)
    T->>BSS: dispatch
    alt Both id and title provided
      BSS->>JSP: forward result="Error: please search by either title or id, not both"
      JSP-->>C: text
    else Only id
      BSS->>PL: searchBooksById(id)
      PL->>DB: SELECT id, title FROM library.book WHERE id=?
      DB-->>PL: row or empty
      alt Found
        PL-->>BSS: Book
        BSS->>JSP: forward result='[{"Title":"...","Id":"..."}]'
        JSP-->>C: JSON-like
      else Not found
        BSS->>JSP: forward result="No books found with an id of X"
        JSP-->>C: text
      end
    else Only title
      BSS->>PL: searchBooksByTitle(title)
      PL->>DB: SELECT id, title FROM library.book WHERE title=?
      DB-->>PL: rows
      BSS->>JSP: forward result=(list or not found message)
      JSP-->>C: JSON-like
    else None
      BSS->>PL: listAllBooks()
      PL->>DB: SELECT id, title FROM library.book
      DB-->>PL: rows
      BSS->>JSP: forward result=(list or "No books exist in the database")
      JSP-->>C: JSON-like or text
    end
  and /demo/borrower
    C->>T: GET /demo/borrower?(id|name|none)
    T->>BRS: dispatch
    BRS->>PL: appropriate search/list call
    PL->>DB: SELECT queries on library.borrower
    DB-->>PL: rows
    BRS->>JSP: forward result as JSON-like or text
    JSP-->>C: response
  end
```

---

## Workflow 6: List Available Books Semantics (GET /demo/listavailable) and UI Autocomplete

Purpose and triggers:
- Purpose: Provide list of “available” books (currently: never-loaned).
- Trigger: library.js AJAX to populate dropdown/autocomplete; manual GET.

Communication patterns:
- Asynchronous AJAX GETs.
- In-process logic.
- JDBC SELECT with LEFT JOIN.
- JSP forwarding (restfulresult.jsp).

```mermaid
sequenceDiagram
  autonumber
  actor U as User (Browser)
  participant JS as library.js
  participant T as Tomcat
  participant LAS as LibraryBookListAvailableServlet
  participant LU as LibraryUtils
  participant PL as PersistenceLayer
  participant DB as H2 DB
  participant JSP as restfulresult.jsp

  U->>JS: Open library.html
  JS->>T: GET /demo/listavailable
  T->>LAS: dispatch
  LAS->>PL: listAvailableBooks()
  PL->>DB: SELECT b.id,b.title FROM library.book b LEFT JOIN library.loan l ON b.id=l.book WHERE l.borrow_date IS NULL
  DB-->>PL: rows (never-loaned only)
  LAS->>JSP: forward result=[{"Title":"t","Id":"1"},...]
  JSP-->>JS: JSON-like array
  JS->>JS: Populate select/autocomplete UI
```

---

## Workflow 7: Mathematics Operations (POST /demo/math, /demo/fibonacci, /demo/ackermann)

Purpose and triggers:
- Purpose: Perform computations with input validation.
- Trigger: Forms on endpointcatalog.html or POSTs.

Communication patterns:
- Synchronous HTTP.
- In-process algorithm execution (CPU-bound).
- No persistence.
- JSP forwarding (restfulresult.jsp).

```mermaid
sequenceDiagram
  autonumber
  actor C as Client (Browser/Test)
  participant T as Tomcat
  participant MS as MathServlet
  participant FS as FibServlet
  participant AS as AckServlet
  participant Algo as Algorithms
  participant JSP as restfulresult.jsp

  par /demo/math
    C->>T: POST /demo/math (item_a, item_b)
    T->>MS: dispatch
    MS->>MS: validate integers and bounds
    alt Invalid
      MS->>JSP: forward "Error: only accepts integers" | "addend too large" | "sum too large"
      JSP-->>C: text
    else Compute
      MS->>MS: sum = a + b
      MS->>JSP: forward sum as text
      JSP-->>C: text
    end
  and /demo/fibonacci
    C->>T: POST /demo/fibonacci (fib_param_n, fib_algorithm_choice)
    T->>FS: dispatch
    FS->>Algo: Fibonacci (recursive | tail_1 | tail_2)
    Algo-->>FS: BigInteger result
    FS->>JSP: forward result
    JSP-->>C: text
  and /demo/ackermann
    C->>T: POST /demo/ackermann (ack_param_m, ack_param_n, ack_algorithm_choice)
    T->>AS: dispatch
    AS->>Algo: Ackermann (recursive | iterative)
    Algo-->>AS: BigInteger result (may be huge)
    AS->>JSP: forward result or error
    JSP-->>C: text
  end
```

---

## Workflow 8: Database Admin via Flyway (GET /demo/flyway)

Purpose and triggers:
- Purpose: Clean and migrate the database schemas for reproducible state in tests/dev.
- Trigger: Manual click from library.html or automated test setup.

Communication patterns:
- Synchronous HTTP.
- In-process calls to IPersistenceLayer → Flyway.
- JDBC DDL execution via Flyway.
- JSP forwarding.

```mermaid
sequenceDiagram
  autonumber
  actor Op as Operator/Test
  participant T as Tomcat
  participant DBS as DbServlet (/flyway)
  participant PL as PersistenceLayer
  participant F as Flyway (embedded)
  participant DB as H2 Database
  participant JSP as result.jsp

  Op->>T: GET /demo/flyway?action=(clean|migrate|default)
  T->>DBS: dispatch
  alt action=clean
    DBS->>PL: cleanDatabase()
    PL->>F: flyway.clean()
    F->>DB: DROP SCHEMA ADMINISTRATIVE|AUTH|LIBRARY CASCADE
    DB-->>F: ok
  else action=migrate
    DBS->>PL: migrateDatabase()
    PL->>F: flyway.migrate()
    F->>DB: CREATE SCHEMAS + CREATE TABLES + INSERT Flyway history
    DB-->>F: ok
  else default
    DBS->>PL: cleanAndMigrateDatabase()
    PL->>F: clean(); migrate()
    F->>DB: execute DDL/DML
  end
  DBS->>JSP: forward result="DB cleaned/migrated/reset"
  JSP-->>Op: HTML
```

---

## Workflow 9: Error Handling and Recovery Patterns

Purpose and triggers:
- Purpose: Illustrate validation errors and DB/runtime failures across endpoints.
- Trigger: Bad inputs or DB exceptions.

Communication patterns:
- Synchronous HTTP.
- In-process validation leading to error strings.
- Unchecked SQL exceptions propagate to container (500) if uncaught.

```mermaid
sequenceDiagram
  autonumber
  actor C as Client
  participant T as Tomcat
  participant RS as RegisterServlet
  participant RU as RegistrationUtils
  participant PL as PersistenceLayer
  participant DB as H2 DB
  participant JSP as result.jsp
  participant Err as Container Error Page

  C->>T: POST /demo/register (username="", password="short")
  T->>RS: dispatch
  RS->>RU: processRegistration
  RU->>RU: validate -> EMPTY_USERNAME or BAD_PASSWORD
  RU-->>RS: RegistrationResult(BAD_PASSWORD,false)
  RS->>JSP: forward "successfully registered: false (BAD_PASSWORD)"
  JSP-->>C: HTML

  C->>T: POST /demo/lend (book="b", borrower="x")
  T->>RS: (representative servlet)
  RS->>PL: createLoan(...)  // after checks
  PL->>DB: INSERT INTO library.loan(...)
  alt DB failure (e.g., constraint violation)
    DB-->>PL: SQLException
    PL-->>RS: throws SqlRuntimeException (unchecked)
    RS-->>T: uncaught runtime exception
    T->>Err: render 500 error page/stack trace (demo)
    Err-->>C: HTTP 500
  end
```

---

## Workflow 10: CI/CD and Security Scanning Pipeline (Git → Jenkins → ZAP → App)

Purpose and triggers:
- Purpose: Build, test, scan, and deploy the application; route API/UI tests through ZAP proxy.
- Trigger: Git push to bare repo triggers Jenkins job via CLI.

Communication patterns:
- Event-driven trigger (Git hook → Jenkins CLI).
- Synchronous stages within Jenkins; external processes (Gradle, Tomcat deploy).
- HTTP traffic proxied through ZAP (9888) during tests.
- Artifacts and reports published to H2O web server.

```mermaid
sequenceDiagram
  autonumber
  actor Dev as Developer
  participant Git as Git Bare Repo (jenkinsbox)
  participant Hook as post-receive Hook
  participant J as Jenkins
  participant G as Gradle
  participant U as uitestbox Tomcat
  participant Z as ZAP Proxy (localhost:9888)
  participant App as Demo Web App (/demo)
  participant DB as H2 (runtime)
  participant R as H2O Reports Server

  Dev->>Git: git push
  Git->>Hook: trigger post-receive
  Hook->>J: Jenkins CLI build "demo"

  J->>G: ./gradlew clean assemble test integrate
  G-->>J: JUnit results
  J->>G: sonarqube, checkQualityGate
  G-->>J: Quality gate status
  J->>G: deployToTest (copy WAR to uitestbox Tomcat)
  G->>U: Deploy WAR
  U->>App: Start webapp, WebAppListener -> cleanAndMigrateDatabase()

  J->>Z: Ensure ZAP running (api.disablekey=true)
  J->>G: runApiTests (HTTP_PROXY=http://127.0.0.1:9888)
  G->>Z: HTTP requests via proxy
  Z->>App: Proxied GET/POST /demo/*
  App->>DB: JDBC queries
  DB-->>App: results
  App-->>Z: responses
  Z-->>G: proxied responses

  J->>G: runBehaveTests (UI BDD, Chrome)
  G->>Z: Browser traffic proxied (if configured)
  Z->>App: proxied UI/API calls
  App-->>Z: responses

  J->>Z: Fetch ZAP report (HTML)
  J->>R: Publish reports to http://jenkinsbox/reports/
  R-->>Dev: Browse reports (ZAP, DependencyCheck, Cucumber, JUnit)
```

---

## Workflow 11: Desktop Auto-Insurance TCP Automation

Purpose and triggers:
- Purpose: Drive Swing UI via a TCP script server to test insurance business logic.
- Trigger: Automation client connects and issues commands.

Communication patterns:
- Synchronous TCP commands over localhost:8000.
- In-process rules evaluation (AutoInsuranceProcessor).
- Text protocol responses.

```mermaid
sequenceDiagram
  autonumber
  actor AT as Automation Test Client
  participant S as AutoInsuranceScriptServer (TCP:8000)
  participant UI as AutoInsuranceUI (Swing)
  participant P as AutoInsuranceProcessor

  AT->>S: connect
  AT->>S: set age 22
  S->>UI: setAge(22)
  S-->>AT: OK

  AT->>S: set claims 2
  S->>UI: setClaims("2")
  S-->>AT: OK

  AT->>S: click calculate
  S->>UI: onClickCalculate()
  UI->>P: evaluate(age=22, claims=2)
  P-->>UI: AutoInsuranceAction(+400, LTR2, canceled=false)
  UI-->>S: update result label
  S-->>AT: OK

  AT->>S: get label
  S-->>AT: "Premium +$400, Warning LTR2"
  AT->>S: quit
  S-->>AT: BYE
```

---

## Workflow 12: H2 Console Access (GET /demo/console)

Purpose and triggers:
- Purpose: Developer opens H2 console to inspect DB (dev-only).
- Trigger: Click link in library.html or manual navigation.

Communication patterns:
- Synchronous HTTP to servlet-mapped H2 console.
- JDBC connections from console to H2 (mem or file).

```mermaid
sequenceDiagram
  autonumber
  actor Dev as Developer
  participant T as Tomcat
  participant H2C as H2 Console Servlet (/demo/console/*)
  participant DB as H2 Database

  Dev->>T: GET /demo/console
  T->>H2C: dispatch
  H2C-->>Dev: HTML console page
  Dev->>H2C: Connect using jdbc:h2:mem:training (MODE=PostgreSQL)
  H2C->>DB: JDBC session
  DB-->>H2C: connection established
  Dev->>H2C: Run SQL queries
  H2C->>DB: Execute SQL
  DB-->>H2C: Results
  H2C-->>Dev: Result tables
```

---

## Notes on Communication and Data Flow Patterns

- Synchronous HTTP:
  - All web app endpoints are classic servlet request/response.
  - Rendering via JSP (result.jsp, restfulresult.jsp) after setting request attributes.

- In-process calls:
  - Servlets delegate to Business Utils (RegistrationUtils, LoginUtils, LibraryUtils), which call PersistenceLayer.

- JDBC access:
  - PersistenceLayer uses prepared statements via H2 JdbcConnectionPool.
  - No explicit transactions; auto-commit true; multi-step operations (e.g., registration then password update) are not atomic.

- Asynchronous interactions:
  - Front-end library.js uses AJAX to prefetch lists.
  - CI/CD trigger from Git hook to Jenkins is event-driven.
  - ZAP proxy intermediates HTTP traffic during tests.

- Error handling:
  - Validation errors returned as human-readable strings or enums.
  - SQLExceptions wrapped into unchecked SqlRuntimeException; uncaught exceptions surface as HTTP 500 responses.
  - Input parsing errors (e.g., non-integer) return explicit “Error: …” messages.

- Data flow highlights:
  - Registration persists user then updates password hash by id.
  - Lending reads book and borrower, checks loan existence, inserts loan with current date.
  - List available uses LEFT JOIN + WHERE l.borrow_date IS NULL (never-loaned semantics).