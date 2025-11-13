## Application Startup: DB Initialization via WebAppListener

Purpose and trigger
- Purpose: Ensure schemas/tables are clean and migrated on app start.
- Trigger: Tomcat initializes the web application context.

Communication patterns
- In-process method calls (ServletContextListener)
- JDBC calls executed by PersistenceLayer
- Flyway migration operations against H2

```mermaid
sequenceDiagram
  autonumber
  participant TC as Tomcat
  participant WAL as WebAppListener
  participant PL as PersistenceLayer (IPersistenceLayer)
  participant FW as Flyway
  participant DB as H2 Database

  TC->>WAL: contextInitialized()
  WAL->>PL: cleanAndMigrateDatabase()
  PL->>FW: clean()
  FW->>DB: DROP schemas/tables
  FW-->>PL: cleaned
  PL->>FW: migrate()
  FW->>DB: CREATE schemas/tables<br/>apply V1, V2 migrations
  FW-->>PL: migrated
  PL-->>WAL: done
  WAL-->>TC: startup complete
```

---

## Authentication: User Registration (POST /register)

Purpose and trigger
- Purpose: Register a new user with password policy and uniqueness checks.
- Trigger: Client submits username/password to /demo/register.

Communication patterns
- HTTP form POST -> Servlet
- In-process calls (RegistrationUtils -> IPersistenceLayer)
- JDBC inserts/updates into AUTH.USER
- JSP forward via ServletUtils to result.jsp

```mermaid
sequenceDiagram
  autonumber
  participant U as Browser/UI
  participant RS as RegisterServlet
  participant RU as RegistrationUtils
  participant PL as PersistenceLayer
  participant DB as H2 (AUTH.USER)
  participant SU as ServletUtils
  participant JSP as result.jsp

  U->>RS: POST /register (username, password)
  alt Missing username
    RS->>SU: forwardToResult("no username provided")
    SU->>JSP: render message
    JSP-->>U: HTML response
  else Missing password
    RS->>SU: forwardToResult("no password provided")
    SU->>JSP: render message
    JSP-->>U: HTML response
  else Inputs present
    RS->>RU: processRegistration(username, password)
    RU->>PL: searchForUserByName(username)
    alt User already exists
      RU-->>RS: RegistrationResult(ALREADY_REGISTERED)
      RS->>SU: forwardToResult("successfully registered: false...status: ALREADY_REGISTERED")
      SU->>JSP: render
      JSP-->>U: HTML
    else User not found
      RU->>RU: isPasswordGood(password)
      alt BAD_PASSWORD (entropy/length)
        RU-->>RS: RegistrationResult(BAD_PASSWORD, message)
        RS->>SU: forwardToResult("...status: BAD_PASSWORD")
        SU->>JSP: render
        JSP-->>U: HTML
      else Password OK
        RU->>PL: saveNewUser(username)
        PL->>DB: INSERT AUTH.USER(name)
        PL-->>RU: userId
        RU->>PL: updateUserWithPassword(userId, hash(SHA-256))
        PL->>DB: UPDATE AUTH.USER SET password_hash=...
        PL-->>RU: updated
        RU-->>RS: RegistrationResult(SUCCESSFULLY_REGISTERED)
        RS->>SU: forwardToResult("successfully registered: true...status: SUCCESSFULLY_REGISTERED")
        SU->>JSP: render
        JSP-->>U: HTML
      end
    end
  end
```

---

## Authentication: Login (POST /login)

Purpose and trigger
- Purpose: Authenticate user credentials.
- Trigger: Client submits username/password to /demo/login.

Communication patterns
- HTTP form POST -> Servlet
- In-process calls (LoginUtils -> IPersistenceLayer.areCredentialsValid)
- JDBC query against AUTH.USER
- JSP forward via ServletUtils to result.jsp

```mermaid
sequenceDiagram
  autonumber
  participant U as Browser/UI
  participant LS as LoginServlet
  participant LU as LoginUtils
  participant PL as PersistenceLayer
  participant DB as H2 (AUTH.USER)
  participant SU as ServletUtils
  participant JSP as result.jsp

  U->>LS: POST /login (username, password)
  alt Missing username
    LS->>SU: forwardToResult("no username provided")
    SU->>JSP: render
    JSP-->>U: HTML
  else Missing password
    LS->>SU: forwardToResult("no password provided")
    SU->>JSP: render
    JSP-->>U: HTML
  else Inputs present
    LS->>LU: isUserRegistered(username, password)
    LU->>PL: areCredentialsValid(username, sha256(password))
    PL->>DB: SELECT password_hash FROM AUTH.USER WHERE name=?
    DB-->>PL: hash or none
    PL-->>LU: true|false
    alt Valid credentials
      LS->>SU: forwardToResult("access granted")
    else Invalid credentials
      LS->>SU: forwardToResult("access denied")
    end
    SU->>JSP: render
    JSP-->>U: HTML
  end
```

---

## Library: Register Book (POST /registerbook)

Purpose and trigger
- Purpose: Add a new book with uniqueness and input validation.
- Trigger: Client submits book title to /demo/registerbook.

Communication patterns
- HTTP POST -> Servlet
- In-process calls (LibraryUtils -> IPersistenceLayer)
- JDBC insert into LIBRARY.BOOK
- JSP forward (result.jsp)

```mermaid
sequenceDiagram
  autonumber
  participant U as Browser/UI
  participant SRV as LibraryRegisterBookServlet
  participant LU as LibraryUtils
  participant PL as PersistenceLayer
  participant DB as H2 (LIBRARY.BOOK)
  participant SU as ServletUtils
  participant JSP as result.jsp

  U->>SRV: POST /registerbook (book)
  alt Empty title
    SRV->>SU: forwardToResult("NO_BOOK_TITLE_PROVIDED")
    SU->>JSP: render
    JSP-->>U: HTML
  else Title provided
    SRV->>LU: registerBook(title)
    LU->>PL: searchBooksByTitle(title)
    alt Already exists
      LU-->>SRV: ALREADY_REGISTERED_BOOK
      SRV->>SU: forwardToResult("ALREADY_REGISTERED_BOOK")
      SU->>JSP: render
      JSP-->>U: HTML
    else Not found
      LU->>PL: saveNewBook(title)
      PL->>DB: INSERT into LIBRARY.BOOK(title)
      PL-->>LU: id
      LU-->>SRV: SUCCESS
      SRV->>SU: forwardToResult("SUCCESS")
      SU->>JSP: render
      JSP-->>U: HTML
    end
  end
```

---

## Library: Register Borrower (POST /registerborrower)

Purpose and trigger
- Purpose: Add a new borrower with uniqueness and input validation.
- Trigger: Client submits name to /demo/registerborrower.

Communication patterns
- HTTP POST -> Servlet
- In-process calls (LibraryUtils -> IPersistenceLayer)
- JDBC insert into LIBRARY.BORROWER
- JSP forward (result.jsp)

```mermaid
sequenceDiagram
  autonumber
  participant U as Browser/UI
  participant SRV as LibraryRegisterBorrowerServlet
  participant LU as LibraryUtils
  participant PL as PersistenceLayer
  participant DB as H2 (LIBRARY.BORROWER)
  participant SU as ServletUtils
  participant JSP as result.jsp

  U->>SRV: POST /registerborrower (borrower)
  alt Empty name
    SRV->>SU: forwardToResult("NO_BORROWER_PROVIDED")
    SU->>JSP: render
    JSP-->>U: HTML
  else Name provided
    SRV->>LU: registerBorrower(name)
    LU->>PL: searchBorrowerDataByName(name)
    alt Already exists
      LU-->>SRV: ALREADY_REGISTERED_BORROWER
      SRV->>SU: forwardToResult("ALREADY_REGISTERED_BORROWER")
      SU->>JSP: render
      JSP-->>U: HTML
    else Not found
      LU->>PL: saveNewBorrower(name)
      PL->>DB: INSERT into LIBRARY.BORROWER(name)
      PL-->>LU: id
      LU-->>SRV: SUCCESS
      SRV->>SU: forwardToResult("SUCCESS")
      SU->>JSP: render
      JSP-->>U: HTML
    end
  end
```

---

## Library: Lend Book (POST /lend)

Purpose and trigger
- Purpose: Create a loan if borrower and book exist and the book is available.
- Trigger: Client submits book and borrower to /demo/lend.

Communication patterns
- HTTP POST -> Servlet
- In-process calls (LibraryUtils -> IPersistenceLayer)
- JDBC: SELECTs for existence and loan status; INSERT into LIBRARY.LOAN
- JSP forward (result.jsp)

```mermaid
sequenceDiagram
  autonumber
  participant U as Browser/UI
  participant SRV as LibraryLendServlet
  participant LU as LibraryUtils
  participant PL as PersistenceLayer
  participant DB as H2 (LIBRARY.*)
  participant SU as ServletUtils
  participant JSP as result.jsp

  U->>SRV: POST /lend (book, borrower)
  alt Empty book or borrower
    SRV->>SU: forwardToResult("NO_BOOK_TITLE_PROVIDED | NO_BORROWER_PROVIDED")
    SU->>JSP: render
    JSP-->>U: HTML
  else Provided
    SRV->>LU: lendBook(title, borrower, Date.now)
    LU->>PL: searchBooksByTitle(title)
    PL->>DB: SELECT * FROM BOOK WHERE title=?
    DB-->>PL: Book or empty
    LU->>PL: searchBorrowerDataByName(borrower)
    PL->>DB: SELECT * FROM BORROWER WHERE name=?
    DB-->>PL: Borrower or empty
    alt Book or borrower not registered
      LU-->>SRV: BOOK_NOT_REGISTERED | BORROWER_NOT_REGISTERED
      SRV->>SU: forwardToResult(error)
      SU->>JSP: render
      JSP-->>U: HTML
    else Both found
      LU->>PL: searchForLoanByBook(book)
      PL->>DB: SELECT * FROM LOAN WHERE book_id=? (open loans)
      DB-->>PL: Loan row or none
      alt Already on loan
        LU-->>SRV: BOOK_CHECKED_OUT
        SRV->>SU: forwardToResult("BOOK_CHECKED_OUT")
        SU->>JSP: render
        JSP-->>U: HTML
      else Available
        LU->>PL: createLoan(book, borrower, date)
        PL->>DB: INSERT into LOAN(book, borrower, borrow_date)
        PL-->>LU: new loan id
        LU-->>SRV: SUCCESS
        SRV->>SU: forwardToResult("SUCCESS")
        SU->>JSP: render
        JSP-->>U: HTML
      end
    end
  end
```

---

## Library: Search/List Books (GET /book) and Borrowers (GET /borrower)

Purpose and trigger
- Purpose: Retrieve entities by id/name/title or list all; support JSON output and human-readable errors.
- Trigger: Client requests GET /demo/book or /demo/borrower with query params.

Communication patterns
- HTTP GET -> Servlet
- In-process calls (LibraryUtils -> IPersistenceLayer)
- JDBC queries
- JSP forward to restfulresult.jsp (plain text/JSON)

```mermaid
sequenceDiagram
  autonumber
  participant U as Browser/UI
  participant BS as LibraryBookListSearchServlet
  participant LU as LibraryUtils
  participant PL as PersistenceLayer
  participant DB as H2 (LIBRARY)
  participant SU as ServletUtils
  participant RJ as restfulresult.jsp

  U->>BS: GET /book?title=<t>&id=<id?>
  alt Both title and id provided
    BS->>SU: forwardToResult("Error: please search by either title or id, not both", RESTFUL)
    SU->>RJ: render text
    RJ-->>U: text/plain
  else Neither provided
    BS->>LU: listAllBooks()
    LU->>PL: listAllBooks()
    PL->>DB: SELECT * FROM BOOK
    DB-->>PL: rows or empty
    alt Empty DB
      BS->>SU: forwardToResult("No books exist in the database", RESTFUL)
    else One or more
      BS->>SU: forwardToResult(JSON[{"Title":"..","Id":".."}], RESTFUL)
    end
    SU->>RJ: render
    RJ-->>U: text/plain
  else Only id provided
    BS->>BS: parseInt(id)
    alt Parse error
      BS->>SU: forwardToResult("Error: could not parse the book id as an integer", RESTFUL)
      SU->>RJ: render
      RJ-->>U: text/plain
    else Parsed ok
      BS->>LU: searchForBookById(id)
      LU->>PL: searchBooksById(id)
      PL->>DB: SELECT * FROM BOOK WHERE id=?
      DB-->>PL: row or none
      alt Not found
        BS->>SU: forwardToResult("No books found with an id of <id>", RESTFUL)
      else Found
        BS->>SU: forwardToResult(JSON [{"Title":"..","Id":".."}], RESTFUL)
      end
      SU->>RJ: render
      RJ-->>U: text/plain
    end
  else Only title provided
    BS->>LU: searchForBookByTitle(title)
    LU->>PL: searchBooksByTitle(title)
    PL->>DB: SELECT * FROM BOOK WHERE title=?
    DB-->>PL: row or none
    alt Not found
      BS->>SU: forwardToResult("No books found with a title of <title>", RESTFUL)
    else Found
      BS->>SU: forwardToResult(JSON [{"Title":"..","Id":".."}], RESTFUL)
    end
    SU->>RJ: render
    RJ-->>U: text/plain
  end
```

```mermaid
sequenceDiagram
  autonumber
  participant U as Browser/UI
  participant BR as LibraryBorrowerListSearchServlet
  participant LU as LibraryUtils
  participant PL as PersistenceLayer
  participant DB as H2 (LIBRARY)
  participant SU as ServletUtils
  participant RJ as restfulresult.jsp

  U->>BR: GET /borrower?name=<n>&id=<id?>
  alt Both name and id provided
    BR->>SU: forwardToResult("Error: please search by either name or id, not both", RESTFUL)
    SU->>RJ: render
    RJ-->>U: text/plain
  else Neither provided
    BR->>LU: listAllBorrowers()
    LU->>PL: listAllBorrowers()
    PL->>DB: SELECT * FROM BORROWER
    DB-->>PL: rows or empty
    alt Empty DB
      BR->>SU: forwardToResult("No borrowers exist in the database", RESTFUL)
    else Found rows
      BR->>SU: forwardToResult(JSON [{"Name":"..","Id":".."}], RESTFUL)
    end
    SU->>RJ: render
    RJ-->>U: text/plain
  else Only id provided
    BR->>BR: parseInt(id)
    alt Parse error
      BR->>SU: forwardToResult("Error: could not parse the borrower id as an integer", RESTFUL)
    else Parsed ok
      BR->>LU: searchForBorrowerById(id)
      LU->>PL: searchBorrowersById(id)
      PL->>DB: SELECT * FROM BORROWER WHERE id=?
      DB-->>PL: row or none
      alt Not found
        BR->>SU: forwardToResult("No borrowers found with an id of <id>", RESTFUL)
      else Found
        BR->>SU: forwardToResult(JSON [{"Name":"..","Id":".."}], RESTFUL)
      end
    end
    SU->>RJ: render
    RJ-->>U: text/plain
  else Only name provided
    BR->>LU: searchForBorrowerByName(name)
    LU->>PL: searchBorrowerDataByName(name)
    PL->>DB: SELECT * FROM BORROWER WHERE name=?
    DB-->>PL: row or none
    alt Not found
      BR->>SU: forwardToResult("No borrowers found with a name of <name>", RESTFUL)
    else Found
      BR->>SU: forwardToResult(JSON [{"Name":"..","Id":".."}], RESTFUL)
    end
    SU->>RJ: render
    RJ-->>U: text/plain
  end
```

---

## Library: List Available Books (GET /listavailable)

Purpose and trigger
- Purpose: Return books not currently on loan for lending UI.
- Trigger: Client (library.js) requests /demo/listavailable.

Communication patterns
- HTTP GET -> Servlet
- In-process calls (LibraryUtils -> IPersistenceLayer.listAvailableBooks)
- JDBC query with join/anti-join
- JSP forward to restfulresult.jsp

```mermaid
sequenceDiagram
  autonumber
  participant UI as Browser (library.js)
  participant LS as LibraryBookListAvailableServlet
  participant LU as LibraryUtils
  participant PL as PersistenceLayer
  participant DB as H2 (LIBRARY)
  participant SU as ServletUtils
  participant RJ as restfulresult.jsp

  UI->>LS: GET /listavailable
  LS->>LU: listAvailableBooks()
  LU->>PL: listAvailableBooks()
  PL->>DB: SELECT b.* FROM BOOK b LEFT JOIN LOAN l ON b.id=l.book WHERE l.id IS NULL
  DB-->>PL: rows or empty
  alt No books
    LS->>SU: forwardToResult("No books exist in the database", RESTFUL)
  else Available list
    LS->>SU: forwardToResult(JSON [{"Title":"..","Id":".."}], RESTFUL)
  end
  SU->>RJ: render
  RJ-->>UI: text/plain
```

---

## Mathematics: Addition (POST /math)

Purpose and trigger
- Purpose: Sum two integers with input and overflow validation.
- Trigger: Client submits item_a and item_b to /demo/math.

Communication patterns
- HTTP POST -> Servlet
- In-process compute (Calculator)
- JSP forward to restfulresult.jsp

```mermaid
sequenceDiagram
  autonumber
  participant U as Browser/UI
  participant MS as MathServlet
  participant CALC as Calculator
  participant SU as ServletUtils
  participant RJ as restfulresult.jsp

  U->>MS: POST /math (item_a, item_b)
  MS->>MS: parse integers
  alt Non-integer input
    MS->>SU: forwardToResult("Error: only accepts integers", RESTFUL)
    SU->>RJ: render
    RJ-->>U: text/plain
  else Integers parsed
    MS->>CALC: add(a, b) with int bounds checks
    alt Addend too large
      MS->>SU: forwardToResult("Error: addend too large - maximum of 2,147,483,647...", RESTFUL)
    else Sum overflow
      MS->>SU: forwardToResult("Error: sum too large - maximum of (absolute) 2,147,483,647", RESTFUL)
    else OK
      CALC-->>MS: sum
      MS->>SU: forwardToResult("<sum>", RESTFUL)
    end
    SU->>RJ: render
    RJ-->>U: text/plain
  end
```

---

## Database Admin: Reset/Migrate via /flyway (GET)

Purpose and trigger
- Purpose: Admin/test endpoint to reset or migrate DB.
- Trigger: Client hits /demo/flyway?action=clean|migrate|[empty].

Communication patterns
- HTTP GET -> Servlet
- In-process calls (PersistenceLayer)
- Flyway clean/migrate against H2
- JSP forward to result.jsp

```mermaid
sequenceDiagram
  autonumber
  participant U as Client (tests/UI)
  participant DS as DbServlet
  participant PL as PersistenceLayer
  participant FW as Flyway
  participant DB as H2
  participant SU as ServletUtils
  participant JSP as result.jsp

  U->>DS: GET /flyway?action=<a>
  alt action=clean
    DS->>PL: cleanDatabase()
    PL->>FW: clean()
    FW->>DB: DROP schemas
    FW-->>PL: cleaned
    DS->>SU: forwardToResult("cleaned")
  else action=migrate
    DS->>PL: migrateDatabase()
    PL->>FW: migrate()
    FW->>DB: APPLY migrations
    FW-->>PL: migrated
    DS->>SU: forwardToResult("migrated")
  else default (empty/other)
    DS->>PL: cleanAndMigrateDatabase()
    PL->>FW: clean()
    FW->>DB: DROP schemas
    FW-->>PL: cleaned
    PL->>FW: migrate()
    FW->>DB: APPLY migrations
    FW-->>PL: migrated
    DS->>SU: forwardToResult("cleaned and migrated")
  end
  SU->>JSP: render
  JSP-->>U: HTML
```

---

## Desktop Application: Auto Insurance UI Automation (TCP 8000)

Purpose and trigger
- Purpose: Drive Swing UI via socket protocol for automated tests and compute insurance action.
- Trigger: Test client connects and issues set/click/get commands.

Communication patterns
- TCP socket request/response (text protocol)
- Event-driven UI actions (Swing)
- In-process rule evaluation (AutoInsuranceProcessor)

```mermaid
sequenceDiagram
  autonumber
  participant C as AutoInsuranceScriptClient (Tests)
  participant S as AutoInsuranceScriptServer (TCP :8000)
  participant UI as AutoInsuranceUI (Swing)
  participant P as AutoInsuranceProcessor

  C->>S: connect
  loop Scripted steps
    C->>S: set age 24
    S->>UI: setAge(24)
    S-->>C: OK

    C->>S: set claims 1
    S->>UI: setClaims(1)
    S-->>C: OK

    C->>S: click calculate
    S->>UI: clickCalculate()
    UI->>P: evaluate(age=24, claims=1)
    P-->>UI: AutoInsuranceAction{+$100, LTR1, not canceled}
    UI->>UI: updateLabel("Premium +$100, letter LTR1")
    S-->>C: OK

    C->>S: get label
    S->>UI: readLabel()
    UI-->>S: "Premium +$100, letter LTR1"
    S-->>C: "Premium +$100, letter LTR1"
  end
  C->>S: quit
  S-->>C: OK (close)
```

---

## Cross-cutting: Forwarding and Error Logging

Purpose and trigger
- Purpose: Standardize rendering responses and log errors on forward failures.
- Trigger: Any servlet forwards to JSP; dispatcher errors are logged.

Communication patterns
- In-process call to ServletUtils.forwardToResult
- Servlet container RequestDispatcher forwarding to JSP

```mermaid
sequenceDiagram
  autonumber
  participant Srv as Any Servlet
  participant SU as ServletUtils
  participant RD as RequestDispatcher
  participant JSP as result.jsp or restfulresult.jsp
  participant Log as SLF4J/Log4j2

  Srv->>SU: forwardToResult(request, response, path)
  SU->>RD: forward(request, response)
  alt Forward succeeds
    RD->>JSP: render ${result}
    JSP-->>Srv: response committed
  else Forward throws Exception
    RD-->>SU: exception
    SU->>Log: error("Unable to forward to JSP", ex)
    SU-->>Srv: propagate or handle
  end
```

---

## Communication Summary per Workflow

- Registration/Login
  - Transport: HTTP POST (form-encoded)
  - Sync calls: Servlet -> Utils -> IPersistenceLayer -> JDBC/H2
  - Data: AUTH.USER (insert/update/query), SHA-256 hashing (no salt)
  - View: result.jsp

- Library (Register/Lend/Search/List)
  - Transport: HTTP (POST/GET)
  - Sync calls: Servlet -> LibraryUtils -> IPersistenceLayer -> JDBC/H2
  - Data: LIBRARY.BOOK, BORROWER, LOAN
  - Views: result.jsp (mutations), restfulresult.jsp (JSON)

- Math
  - Transport: HTTP POST
  - Pure compute: no DB
  - View: restfulresult.jsp

- DB Admin
  - Transport: HTTP GET
  - Sync calls: Servlet -> PersistenceLayer -> Flyway -> H2
  - View: result.jsp

- Desktop UI Automation
  - Transport: TCP socket (text protocol)
  - Event-driven UI -> Processor compute
  - Response: text strings (“OK”, values)

- Error handling patterns
  - Input validation at Servlet layer → immediate user-facing messages
  - Domain validation in Utils → enumerated outcomes and messages
  - Persistence exceptions wrapped (SqlRuntimeException); no explicit recovery besides test reset via /flyway
  - Forwarding failures logged via SLF4J; fallback is exception propagation (HTTP 500)