# Runtime Interaction Flows and Sequence Diagrams

## 1) App startup: DB clean-and-migrate (event-driven)

Purpose and trigger
- Ensures a known schema on deployment/start.
- Trigger: Servlet container initializes the web app (ServletContextInitialized).

Communication patterns
- Event-driven listener -> in-process method calls -> JDBC to H2; Flyway migration.
- No HTTP involved.

```mermaid
sequenceDiagram
  participant Tomcat as Tomcat (Servlet Container)
  participant Listener as WebAppListener
  participant PL as PersistenceLayer
  participant Flyway as Flyway
  participant H2 as H2 Database

  Tomcat->>Listener: contextInitialized()
  Listener->>PL: cleanAndMigrateDatabase()
  PL->>Flyway: clean()
  Flyway->>H2: DROP SCHEMAS (ADMINISTRATIVE, AUTH, LIBRARY)
  PL->>Flyway: migrate()
  Flyway->>H2: CREATE SCHEMAS/TABLES via V1,V2
  Flyway-->>PL: migration success
  PL-->>Listener: "cleaned and migrated"
  Listener-->>Tomcat: init complete
```

Error handling
- Flyway/SQL exceptions bubble as runtime errors; startup fails fast (logs).


## 2) Admin DB reset via /flyway endpoint

Purpose and trigger
- Test and demo convenience to reset DB on demand.
- Trigger: GET /demo/flyway?action=clean|migrate|<empty>.

Communication patterns
- HTTP GET -> servlet -> in-process call -> JDBC via Flyway; response via JSP.

```mermaid
sequenceDiagram
  participant User as Tester/Browser
  participant ZAP as (optional) Proxy (ZAP)
  participant DbS as DbServlet (/flyway)
  participant PL as PersistenceLayer
  participant Flyway as Flyway
  participant H2 as H2 DB
  participant JSP as result.jsp

  User->>ZAP: GET /demo/flyway?action=clean
  ZAP->>DbS: forward GET
  DbS->>PL: cleanDatabase()
  PL->>Flyway: clean()
  Flyway->>H2: DROP schemas
  Flyway-->>PL: ok
  DbS->>JSP: forward with result="cleaned"
  JSP-->>User: 200 OK "cleaned"
```

Error handling
- On unknown action, servlet defaults to cleanAndMigrate; errors rendered as plain text via JSP.


## 3) User registration

Purpose and trigger
- Create a new user with password quality enforcement.
- Trigger: POST /demo/register.

Communication patterns
- HTTP POST -> servlet -> business utils -> persistence (SELECT, INSERT, UPDATE) -> JSP.

```mermaid
sequenceDiagram
  participant Browser as Browser (library.html)
  participant RegS as RegisterServlet (/register)
  participant RegU as RegistrationUtils
  participant PL as PersistenceLayer
  participant H2 as H2 DB
  participant JSP as result.jsp

  Browser->>RegS: POST /register (username, password)
  RegS->>RegU: processRegistration(username, password)
  alt missing username/password
    RegU-->>RegS: RegistrationResult(status=EMPTY_*, success=false)
  else inputs present
    RegU->>PL: searchForUserByName(username)
    PL->>H2: SELECT * FROM auth.user WHERE name=?
    H2-->>PL: Optional.empty / User
    alt user exists
      RegU-->>RegS: RegistrationResult(ALREADY_REGISTERED, false)
    else new user
      RegU->>PL: saveNewUser(username)
      PL->>H2: INSERT INTO auth.user(name) VALUES (?)
      H2-->>PL: new user id
      RegU->>PL: updateUserWithPassword(id, password) (SHA-256 hash)
      PL->>H2: UPDATE auth.user SET password_hash=? WHERE id=?
      H2-->>PL: ok
      RegU-->>RegS: RegistrationResult(SUCCESSFULLY_REGISTERED, true)
    end
  end
  RegS->>JSP: forward result string
  JSP-->>Browser: 200 "successfully registered: true|false"
```

Error handling
- Password policy failure yields BAD_PASSWORD with entropy feedback.
- SQL exceptions wrapped in SqlRuntimeException; servlet returns failure text.


## 4) User login

Purpose and trigger
- Authenticate an existing user.
- Trigger: POST /demo/login.

Communication patterns
- HTTP POST -> servlet -> utils -> persistence SELECT -> JSP.

```mermaid
sequenceDiagram
  participant Browser as Browser (library.html)
  participant LogS as LoginServlet (/login)
  participant LogU as LoginUtils
  participant PL as PersistenceLayer
  participant H2 as H2 DB
  participant JSP as result.jsp

  Browser->>LogS: POST /login (username, password)
  LogS->>LogU: isUserRegistered(username, password)
  alt missing username/password
    LogU-->>LogS: "no username/password provided"
  else
    LogU->>PL: areCredentialsValid(username, password)
    PL->>H2: SELECT id FROM auth.user WHERE name=? AND password_hash=SHA256(?)
    H2-->>PL: row exists? (Optional.of(true) | Optional.empty)
    PL-->>LogU: Optional<Boolean>
    alt present true
      LogU-->>LogS: "access granted"
    else not present
      LogU-->>LogS: "access denied"
    end
  end
  LogS->>JSP: forward result
  JSP-->>Browser: 200 "access granted|denied|no ... provided"
```

Error handling
- Missing inputs handled with explicit messages.
- DB errors -> runtime exception -> error page/logs.


## 5) Library UI prefill (AJAX): available books and borrowers

Purpose and trigger
- Improve UX by pre-populating lending controls.
- Trigger: library.js on page load/change.

Communication patterns
- Async XHR (GET) from browser -> servlet -> utils -> persistence SELECT -> JSP (raw text array).

```mermaid
sequenceDiagram
  participant UI as library.html + library.js
  participant AvS as LibraryBookListAvailableServlet (/listavailable)
  participant BorS as LibraryBorrowerListSearchServlet (/borrower)
  participant LibU as LibraryUtils
  participant PL as PersistenceLayer
  participant H2 as H2 DB
  participant JSP as restfulresult.jsp

  UI->>AvS: GET /listavailable
  AvS->>LibU: listAvailableBooks()
  LibU->>PL: listAvailableBooks()
  PL->>H2: SELECT b.id,b.title FROM book b LEFT JOIN loan l ON b.id=l.book WHERE l.borrow_date IS NULL
  H2-->>PL: rows[]
  AvS->>JSP: forward JSON-like array
  JSP-->>UI: 200 [ {Title,Id}, ... ]

  UI->>BorS: GET /borrower (no params)
  BorS->>LibU: listAllBorrowers()
  LibU->>PL: listAllBorrowers()
  PL->>H2: SELECT id,name FROM borrower
  H2-->>PL: rows[]
  BorS->>JSP: forward JSON-like array
  JSP-->>UI: 200 [ {Name,Id}, ... ]
```

Error handling
- Empty DB -> "No books/borrowers exist in the database" strings.


## 6) Librarian flow: register book, register borrower, lend book

Purpose and trigger
- End-to-end circulation: add entities and lend.
- Trigger: POSTs from library.html forms.

Communication patterns
- HTTP POSTs -> servlets -> business utils -> persistence INSERT/SELECT/INSERT -> JSP.

```mermaid
sequenceDiagram
  participant Browser as Browser (library.html)
  participant RegBook as LibraryRegisterBookServlet (/registerbook)
  participant RegBor as LibraryRegisterBorrowerServlet (/registerborrower)
  participant LendS as LibraryLendServlet (/lend)
  participant LibU as LibraryUtils
  participant PL as PersistenceLayer
  participant H2 as H2 DB
  participant JSP as result.jsp

  Browser->>RegBook: POST book="The DevOps Handbook"
  RegBook->>LibU: registerBook(title)
  LibU->>PL: saveNewBook(title)
  PL->>H2: INSERT INTO library.book(title) VALUES (?)
  H2-->>PL: new book id
  RegBook->>JSP: result="SUCCESS"
  JSP-->>Browser: "SUCCESS"

  Browser->>RegBor: POST borrower="alice"
  RegBor->>LibU: registerBorrower(name)
  LibU->>PL: saveNewBorrower(name)
  PL->>H2: INSERT INTO library.borrower(name) VALUES (?)
  H2-->>PL: new borrower id
  RegBor->>JSP: result="SUCCESS"
  JSP-->>Browser: "SUCCESS"

  Browser->>LendS: POST book="The DevOps Handbook", borrower="alice"
  LendS->>LibU: lendBook(title,name,Date.now)
  LibU->>PL: searchBooksByTitle(title)
  PL->>H2: SELECT id,title FROM book WHERE title=?
  H2-->>PL: Book
  LibU->>PL: searchBorrowerDataByName(name)
  PL->>H2: SELECT id,name FROM borrower WHERE name=?
  H2-->>PL: Borrower
  LibU->>PL: searchForLoanByBook(Book)
  PL->>H2: SELECT ... FROM loan WHERE book=?
  H2-->>PL: Optional.empty
  LibU->>PL: createLoan(Book,Borrower,Date)
  PL->>H2: INSERT INTO library.loan(book,borrower,borrow_date) VALUES (?,?,?)
  H2-->>PL: new loan id
  LendS->>JSP: result="SUCCESS"
  JSP-->>Browser: "SUCCESS"
```

Error handling
- If book or borrower missing: BOOK_NOT_REGISTERED/BORROWER_NOT_REGISTERED.
- If loan already exists: BOOK_CHECKED_OUT.
- Empty inputs: NO_BOOK_TITLE_PROVIDED/NO_BORROWER_PROVIDED.


## 7) Library search books (by id or title) with validation

Purpose and trigger
- Search or list books for UI/admin.
- Trigger: GET /demo/book?id=.. or title=.. (or none to list all).

Communication patterns
- HTTP GET -> servlet -> utils -> persistence SELECT -> JSP.

```mermaid
sequenceDiagram
  participant Client as Client
  participant BookS as LibraryBookListSearchServlet (/book)
  participant LibU as LibraryUtils
  participant PL as PersistenceLayer
  participant H2 as H2 DB
  participant JSP as restfulresult.jsp

  Client->>BookS: GET /book?id=1
  alt both id and title provided
    BookS-->>Client: 400 "Error: please search by either title or id, not both"
  else id provided
    BookS->>LibU: searchForBookById(1)
    LibU->>PL: searchBooksById(1)
    PL->>H2: SELECT id,title FROM book WHERE id=?
    H2-->>PL: row or none
    alt found
      BookS->>JSP: [{"Title":"...","Id":"1"}]
      JSP-->>Client: 200 JSON-like
    else not found
      BookS-->>Client: 200 "No books found with an id of 1"
    end
  else title provided
    BookS->>LibU: searchForBookByTitle("x")
    LibU->>PL: searchBooksByTitle("x")
    PL->>H2: SELECT id,title FROM book WHERE title=?
    H2-->>PL: row or none
    BookS->>JSP: output
    JSP-->>Client: 200
  else no params
    BookS->>LibU: listAllBooks()
    LibU->>PL: listAllBooks()
    PL->>H2: SELECT id,title FROM book
    H2-->>PL: rows[]
    BookS->>JSP: array or "No books exist in the database"
    JSP-->>Client: 200
  end
```

Error handling
- id parse failure -> "Error: could not parse the book id as an integer".


## 8) Math addition with validation and overflow checks

Purpose and trigger
- Teaching/demo endpoint to add two integers with bounds checking.
- Trigger: POST /demo/math.

Communication patterns
- HTTP POST -> servlet -> validation/compute -> JSP.

```mermaid
sequenceDiagram
  participant Client as Client
  participant MathS as MathServlet (/math)
  participant JSP as restfulresult.jsp

  Client->>MathS: POST item_a=5&item_b=11
  alt both integers within bounds
    MathS-->>JSP: forward result="16"
    JSP-->>Client: 200 "16"
  else non-integer input
    MathS-->>JSP: "Error: only accepts integers"
    JSP-->>Client: 200 error text
  else addend too large
    MathS-->>JSP: "Error: addend too large - maximum of 2,147,483,647..."
    JSP-->>Client: 200 error text
  else sum overflow
    MathS-->>JSP: "Error: sum too large - maximum of (absolute) 2,147,483,647"
    JSP-->>Client: 200 error text
  end
```

Error handling
- Purely input/overflow checks; no DB effects.


## 9) Ackermann compute with algorithm selection

Purpose and trigger
- Demonstrate heavy recursion vs tail-recursive iterative algorithm.
- Trigger: POST /demo/ackermann.

Communication patterns
- HTTP POST -> servlet -> algorithm function -> JSP.

```mermaid
sequenceDiagram
  participant Client as Client
  participant AckS as AckServlet (/ackermann)
  participant Ack as Ackermann (recursive)
  participant AckI as AckermannIterative
  participant JSP as restfulresult.jsp

  Client->>AckS: POST ack_param_m=3&ack_param_n=6&ack_algorithm_choice=tail_recursive
  alt choice=tail_recursive
    AckS->>AckI: calculate(3,6)
    AckI-->>AckS: BigInteger result
  else choice=regular_recursive
    AckS->>Ack: calculate(3,6)
    Ack-->>AckS: BigInteger result (risk of stack overflow for larger inputs)
  end
  AckS->>JSP: forward result=<BigInteger>
  JSP-->>Client: 200 text
```

Error handling
- Forward exceptions logged; invalid params lead to generic error text.


## 10) Desktop AutoInsurance calculation via TCP automation

Purpose and trigger
- Demonstrate desktop UI automation and business rules.
- Trigger: Test client connects to port 8000 and issues commands.

Communication patterns
- TCP text protocol (synchronous request/response) -> UI/controller -> processor.

```mermaid
sequenceDiagram
  participant Tester as Desktop Tester (Client)
  participant Server as AutoInsuranceScriptServer (TCP:8000)
  participant UI as AutoInsuranceUI (Swing)
  participant Proc as AutoInsuranceProcessor

  Tester->>Server: connect
  Tester->>Server: set age 22
  Server->>UI: setAge(22)
  Server-->>Tester: OK
  Tester->>Server: set claims 1
  Server->>UI: setClaims(1)
  Server-->>Tester: OK
  Tester->>Server: click calculate
  Server->>UI: clickCalculate()
  UI->>Proc: compute(claims=1, age=22)
  Proc-->>UI: AutoInsuranceAction(+$100, LTR1, not canceled)
  UI-->>Server: updateLabel("Premium increase: $100 ...")
  Server-->>Tester: OK
  Tester->>Server: get label
  Server-->>Tester: "Premium increase: $100 ..."
  Tester->>Server: quit
  Server-->>Tester: OK; close
```

Error handling
- Invalid commands/params -> "FAILURE".
- Business rule errors produce isError=true; label reflects appropriate message.


## 11) CI/CD pipeline: push-to-deploy-test with proxied security tests

Purpose and trigger
- Full DevSecOps feedback loop from commit to deployed-and-tested app.
- Trigger: git push to bare repo on jenkinsbox (post-receive hook).

Communication patterns
- Event (Git hook) -> Jenkins CLI -> pipeline steps (Gradle tasks, REST to ZAP, deploy to Tomcat), tests via HTTP proxied through ZAP.

```mermaid
sequenceDiagram
  participant Dev as Developer
  participant Git as Bare Git Repo (jenkinsbox)
  participant Hook as post-receive Hook
  participant Jnk as Jenkins
  participant Gradle as Gradle Tasks
  participant Sonar as SonarQube
  participant Tomcat as Tomcat (uitestbox)
  participant ZAP as ZAP Proxy
  participant Tests as API/UI Test Runners
  participant App as Demo App (/demo)

  Dev->>Git: git push
  Git->>Hook: trigger
  Hook->>Jnk: jenkins-cli build demo
  Jnk->>Gradle: clean assemble test integrate bdd
  Gradle-->>Jnk: JUnit results
  Jnk->>Gradle: sonarqube
  Gradle->>Sonar: analysis; quality gate
  Sonar-->>Jnk: gate status
  Jnk->>Gradle: deployToTest (copy WAR)
  Gradle->>Tomcat: deploy WAR; restart app
  Jnk->>Gradle: waitForHeartBeat
  Jnk->>ZAP: clear session (HTTP)
  Jnk->>Tests: run API/UI (HTTP_PROXY=http://localhost:9888)
  Tests->>ZAP: proxied HTTP
  ZAP->>App: HTTP requests (/demo endpoints)
  App-->>ZAP: responses
  ZAP-->>Tests: responses
  Jnk->>Gradle: runPerfTests (JMeter)
  Gradle->>App: load
  Jnk->>ZAP: pull HTML report
  ZAP-->>Jnk: report.html
  Jnk-->>Dev: pipeline results + links
```

Error handling
- Failing unit/integration/quality gate halts pipeline.
- Heartbeat/deploy failure stops downstream tests; logs archived.
- ZAP proxy unreachable -> tests fallback or fail depending on stage configuration.