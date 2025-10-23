### 1. New User Registration Workflow

This workflow describes the process of a new user registering with the system. It includes input validation, checking for duplicate usernames, password strength verification, and persisting the new user to the database.

**Communication Patterns:**
*   **Client-Server:** Synchronous HTTP POST request from the browser.
*   **Server-Internal:** Direct Java method calls between the Servlet, business logic (`RegistrationUtils`), and data access layers (`PersistenceLayer`).
*   **Server-Database:** JDBC calls executing SQL SELECT and INSERT statements within a transaction.

```mermaid
sequenceDiagram
    actor User
    participant Browser as Frontend
    participant RegisterServlet
    participant RegistrationUtils
    participant PersistenceLayer
    database Database

    User->>+Browser: Fills registration form (username, password) and submits
    Browser->>+RegisterServlet: POST /demo/register (username, password)
    RegisterServlet->>+RegistrationUtils: processRegistration(username, password)

    RegistrationUtils->>+PersistenceLayer: searchForUserByName(username)
    PersistenceLayer->>+Database: SELECT id FROM auth.USER WHERE name = ?
    Database-->>-PersistenceLayer: User record (or null)
    PersistenceLayer-->>-RegistrationUtils: User (or null)

    alt User already exists
        RegistrationUtils-->>-RegisterServlet: RegistrationResult(FAILURE, "already registered")
        RegisterServlet-->>-Browser: HTTP Response (renders "already registered")
        Browser-->>-User: Displays error message
    else User does not exist
        RegistrationUtils->>RegistrationUtils: isPasswordGood(password)
        note right of RegistrationUtils: Validates length and entropy using Nbvcxz library

        alt Password is weak
            RegistrationUtils-->>-RegisterServlet: RegistrationResult(FAILURE, "password too weak")
            RegisterServlet-->>-Browser: HTTP Response (renders "password too weak")
            Browser-->>-User: Displays error message
        else Password is strong
            RegistrationUtils->>RegistrationUtils: hashPassword(password)
            note right of RegistrationUtils: Uses SHA-256
            RegistrationUtils->>+PersistenceLayer: saveNewUser(username, hashedPassword)
            PersistenceLayer->>+Database: INSERT INTO auth.USER (name, password_hash) VALUES (?, ?)
            Database-->>-PersistenceLayer: New user ID
            PersistenceLayer-->>-RegistrationUtils: User object with new ID
            RegistrationUtils-->>-RegisterServlet: RegistrationResult(SUCCESS, "successfully registered")
            RegisterServlet-->>-Browser: HTTP Response (renders success page)
            Browser-->>-User: Displays success message
        end
    end
```

### 2. Book Lending Workflow (Happy Path)

This workflow captures the core business process of a librarian lending a registered book to a registered borrower. It involves verifying the existence of the book and borrower and ensuring the book is currently available before creating a loan record.

**Communication Patterns:**
*   **Client-Server:** Synchronous HTTP POST request.
*   **Server-Internal:** Direct Java method calls between layers.
*   **Server-Database:** A sequence of JDBC calls executing multiple SQL SELECT statements to validate state, followed by an INSERT statement to record the loan. This sequence should occur within a single logical transaction.

```mermaid
sequenceDiagram
    actor Librarian
    participant Browser as Frontend
    participant LibraryLendServlet
    participant LibraryUtils
    participant PersistenceLayer
    database Database

    Librarian->>+Browser: Selects book and borrower, submits "Lend" form
    Browser->>+LibraryLendServlet: POST /demo/lend (book_title, borrower_name)
    LibraryLendServlet->>+LibraryUtils: lendBook(book_title, borrower_name)

    note over LibraryUtils, Database: Verify book, borrower, and book availability
    LibraryUtils->>+PersistenceLayer: searchBooksByTitle(book_title)
    PersistenceLayer->>+Database: SELECT id, title FROM library.BOOK WHERE title = ?
    Database-->>-PersistenceLayer: Book record
    PersistenceLayer-->>-LibraryUtils: Book object

    LibraryUtils->>+PersistenceLayer: searchBorrowerDataByName(borrower_name)
    PersistenceLayer->>+Database: SELECT id, name FROM library.BORROWER WHERE name = ?
    Database-->>-PersistenceLayer: Borrower record
    PersistenceLayer-->>-LibraryUtils: Borrower object

    LibraryUtils->>+PersistenceLayer: searchForLoanByBook(book)
    PersistenceLayer->>+Database: SELECT id FROM library.LOAN WHERE book = ?
    Database-->>-PersistenceLayer: null (no existing loan)
    PersistenceLayer-->>-LibraryUtils: null

    alt Book is available
        note over LibraryUtils, Database: Create the loan record
        LibraryUtils->>+PersistenceLayer: createLoan(book, borrower)
        PersistenceLayer->>+Database: INSERT INTO library.LOAN (book, borrower, borrow_date) VALUES (?, ?, NOW())
        Database-->>-PersistenceLayer: Success
        PersistenceLayer-->>-LibraryUtils: Loan object
        LibraryUtils-->>-LibraryLendServlet: LendResult(SUCCESS)
        LibraryLendServlet-->>-Browser: HTTP Response (renders success page)
        Browser-->>-Librarian: Displays "SUCCESS" message
    else Book is already checked out
        %% This path is detailed in the next diagram %%
    end
```

### 3. Error Handling: Attempting to Lend an Unavailable Book

This workflow illustrates the system's error handling when a librarian attempts to lend a book that is already on loan. It demonstrates the enforcement of a critical business rule.

**Communication Patterns:**
*   **Client-Server:** Synchronous HTTP POST.
*   **Server-Internal:** Direct Java method calls.
*   **Server-Database:** JDBC calls executing SQL SELECT statements that identify a business rule violation. No write operation (INSERT) occurs.

```mermaid
sequenceDiagram
    actor Librarian
    participant Browser as Frontend
    participant LibraryLendServlet
    participant LibraryUtils
    participant PersistenceLayer
    database Database

    Librarian->>+Browser: Tries to lend an already-loaned book
    Browser->>+LibraryLendServlet: POST /demo/lend (book_title, borrower_name)
    LibraryLendServlet->>+LibraryUtils: lendBook(book_title, borrower_name)

    note over LibraryUtils, Database: Verification steps...
    LibraryUtils->>+PersistenceLayer: searchBooksByTitle(book_title)
    PersistenceLayer-->>-LibraryUtils: Book object
    LibraryUtils->>+PersistenceLayer: searchBorrowerDataByName(borrower_name)
    PersistenceLayer-->>-LibraryUtils: Borrower object

    note over LibraryUtils, Database: Check if book is already on loan
    LibraryUtils->>+PersistenceLayer: searchForLoanByBook(book)
    PersistenceLayer->>+Database: SELECT id, book, borrower, borrow_date FROM library.LOAN WHERE book = ?
    Database-->>-PersistenceLayer: Existing Loan record
    PersistenceLayer-->>-LibraryUtils: Loan object (not null)

    alt Book is already checked out
        LibraryUtils-->>-LibraryLendServlet: LendResult(FAILURE, "BOOK_CHECKED_OUT")
        LibraryLendServlet-->>-Browser: HTTP Response (renders error message)
        Browser-->>-Librarian: Displays error "This book is already checked out."
    else Book is available
        %% See Happy Path diagram %%
    end
```

### 4. Asynchronous UI Autocomplete for Available Books

This workflow shows a more modern, dynamic UI interaction. As the user types in a search box, the frontend makes an asynchronous request to the backend to fetch a list of available books, which is then used to populate an autocomplete dropdown menu.

**Communication Patterns:**
*   **Client-Server:** Asynchronous HTTP GET request (AJAX/XHR) initiated by JavaScript.
*   **Data Format:** The server responds with a JSON payload.
*   **Server-Internal:** Direct Java method calls.
*   **Server-Database:** A single, read-only SQL SELECT query.

```mermaid
sequenceDiagram
    actor User
    participant Browser as Frontend (library.js)
    participant LibraryBookListAvailableServlet
    participant LibraryUtils
    participant PersistenceLayer
    database Database

    User->>+Browser: Types in the 'Lend Book' input field
    Browser->>+LibraryBookListAvailableServlet: Asynchronous GET /demo/listavailable
    note right of Browser: AJAX/XHR call

    LibraryBookListAvailableServlet->>+LibraryUtils: listAvailableBooks()
    LibraryUtils->>+PersistenceLayer: listAvailableBooks()
    PersistenceLayer->>+Database: SELECT b.* FROM library.BOOK b LEFT JOIN library.LOAN l ON b.id = l.book WHERE l.id IS NULL
    Database-->>-PersistenceLayer: List of Book records
    PersistenceLayer-->>-LibraryUtils: List<Book>
    LibraryUtils-->>-LibraryBookListAvailableServlet: List<Book>

    LibraryBookListAvailableServlet->>LibraryBookListAvailableServlet: Format List<Book> as JSON array
    LibraryBookListAvailableServlet-->>-Browser: HTTP 200 OK with JSON payload<br/>[{"id":1, "title":"Book A"}, ...]

    Browser->>Browser: JavaScript callback handles response
    Browser->>User: Displays autocomplete suggestions under the input field
```

### 5. Administrative Workflow: Database Reset via Flyway

This workflow details an administrative or testing-related process where the entire database is wiped clean and re-initialized to a known state using the Flyway migration tool. This is triggered by a simple HTTP GET request.

**Communication Patterns:**
*   **Client-Server:** Synchronous HTTP GET request from a test runner or developer's tool.
*   **Server-Internal:** Direct Java method call to the persistence layer.
*   **Framework Interaction:** The persistence layer delegates the complex task of schema manipulation to the Flyway library.
*   **Server-Database:** Flyway executes Data Definition Language (DDL) and Data Manipulation Language (DML) commands (e.g., `DROP TABLE`, `CREATE TABLE`, `INSERT`) against the database.

```mermaid
sequenceDiagram
    actor Developer/TestRunner
    participant HTTPClient
    participant DbServlet
    participant PersistenceLayer
    participant FlywayLibrary as Flyway Library
    database Database

    Developer/TestRunner->>+HTTPClient: Initiate request to reset DB
    HTTPClient->>+DbServlet: GET /demo/flyway?action=clean

    DbServlet->>+PersistenceLayer: cleanDatabase()
    PersistenceLayer->>+FlywayLibrary: flyway.clean()
    note right of FlywayLibrary: Connects to the database and <br/>executes DROP statements for all <br/>objects in the managed schemas.
    FlywayLibrary->>+Database: DROP TABLE library.LOAN, library.BOOK, ...
    Database-->>-FlywayLibrary: Success
    FlywayLibrary-->>-PersistenceLayer: Clean successful
    PersistenceLayer-->>-DbServlet: Success
    DbServlet-->>-HTTPClient: HTTP 200 OK

    HTTPClient->>+DbServlet: GET /demo/flyway?action=migrate
    DbServlet->>+PersistenceLayer: migrateDatabase()
    PersistenceLayer->>+FlywayLibrary: flyway.migrate()
    note right of FlywayLibrary: Reads migration scripts (V1__, V2__, etc.)<br/> and applies them sequentially.
    FlywayLibrary->>+Database: CREATE TABLE..., ALTER TABLE...
    Database-->>-FlywayLibrary: Success
    FlywayLibrary-->>-PersistenceLayer: Migration successful
    PersistenceLayer-->>-DbServlet: Success
    DbServlet-->>-HTTPClient: HTTP 200 OK
    HTTPClient-->>-Developer/TestRunner: Reports success
```