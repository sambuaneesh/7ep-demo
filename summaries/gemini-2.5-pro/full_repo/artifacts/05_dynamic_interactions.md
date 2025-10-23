### 1. Web App: New User Registration

**Workflow Purpose and Trigger:**
This workflow allows a new user to create an account in the system. It is triggered when a user fills out the registration form on the `library.html` page and submits it, sending an HTTP POST request to the `/register` endpoint. The system validates the input, checks for existing usernames, assesses password strength, and persists the new user to the database if all checks pass.

**Communication Patterns:**
- **Client-Server:** Synchronous HTTP POST request from the browser to the web server.
- **Internal:** Synchronous Java method calls between the Servlet, business logic (`RegistrationUtils`), and persistence layers.
- **Data Persistence:** Synchronous JDBC calls within a database transaction to the H2 database.

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant WebServer as Tomcat
    participant RegisterServlet
    participant RegistrationUtils
    participant PersistenceLayer as IPersistenceLayer
    participant Database as H2 Database

    User->>Browser: Fills and submits registration form
    Browser->>WebServer: HTTP POST /register (username, password)
    WebServer->>RegisterServlet: doPost(request, response)
    RegisterServlet->>RegistrationUtils: processRegistration(username, password)
    
    RegistrationUtils->>PersistenceLayer: searchForUserByName(username)
    PersistenceLayer->>Database: SELECT id FROM auth.user WHERE name = ?
    Database-->>PersistenceLayer: User data (or empty)
    PersistenceLayer-->>RegistrationUtils: Optional<User>

    alt User already exists
        RegistrationUtils-->>RegisterServlet: RegistrationResult(ALREADY_REGISTERED)
        RegisterServlet->>Browser: Renders result page (Already Registered)
    else User does not exist
        RegistrationUtils->>RegistrationUtils: isPasswordGood(password)
        
        alt Password is weak
            RegistrationUtils-->>RegisterServlet: RegistrationResult(BAD_PASSWORD)
            RegisterServlet->>Browser: Renders result page (Bad Password)
        else Password is strong
            RegistrationUtils->>PersistenceLayer: saveNewUser(username)
            PersistenceLayer->>Database: INSERT INTO auth.user (name) VALUES (?)
            Database-->>PersistenceLayer: new user_id
            PersistenceLayer-->>RegistrationUtils: user_id
            
            RegistrationUtils->>PersistenceLayer: updateUserWithPassword(user_id, password)
            PersistenceLayer->>Database: UPDATE auth.user SET password_hash = ? WHERE id = ?
            Database-->>PersistenceLayer: Success
            PersistenceLayer-->>RegistrationUtils: void
            
            RegistrationUtils-->>RegisterServlet: RegistrationResult(SUCCESSFULLY_REGISTERED)
            RegisterServlet->>Browser: Renders result page (Success)
        end
    end
    Browser->>User: Displays result page
```

### 2. Web App: Lend a Book

**Workflow Purpose and Trigger:**
This workflow allows a librarian to lend a registered book to a registered borrower. It is triggered when the librarian fills out the "Borrow a book" form on `library.html` and submits it, sending an HTTP POST to the `/lend` endpoint. The system validates that the book and borrower exist and that the book is currently available before creating a new loan record.

**Communication Patterns:**
- **Client-Server:** Synchronous HTTP POST from the browser.
- **Internal:** Synchronous Java method calls between layers.
- **Data Persistence:** Synchronous JDBC SELECT and INSERT queries to the H2 database.

```mermaid
sequenceDiagram
    actor Librarian
    participant Browser
    participant WebServer as Tomcat
    participant LibraryLendServlet
    participant LibraryUtils
    participant PersistenceLayer as IPersistenceLayer
    participant Database as H2 Database

    Librarian->>Browser: Fills and submits lend book form
    Browser->>WebServer: HTTP POST /lend (book, borrower)
    WebServer->>LibraryLendServlet: doPost(request, response)
    LibraryLendServlet->>LibraryUtils: lendBook(bookTitle, borrowerName, borrowDate)

    LibraryUtils->>PersistenceLayer: searchForBookByTitle(bookTitle)
    PersistenceLayer->>Database: SELECT id FROM library.book WHERE title = ?
    Database-->>PersistenceLayer: Book data
    PersistenceLayer-->>LibraryUtils: Optional<Book>

    LibraryUtils->>PersistenceLayer: searchForBorrowerByName(borrowerName)
    PersistenceLayer->>Database: SELECT id, name FROM library.borrower WHERE name = ?
    Database-->>PersistenceLayer: Borrower data
    PersistenceLayer-->>LibraryUtils: Optional<Borrower>

    alt Book or Borrower not registered
        LibraryUtils-->>LibraryLendServlet: LibraryActionResults(BOOK_NOT_REGISTERED / BORROWER_NOT_REGISTERED)
        LibraryLendServlet->>Browser: Renders result page (Error)
    else Both are registered
        LibraryUtils->>PersistenceLayer: searchForLoanByBook(book)
        PersistenceLayer->>Database: SELECT ... FROM library.loan WHERE book = ?
        Database-->>PersistenceLayer: Loan data (or empty)
        PersistenceLayer-->>LibraryUtils: Optional<Loan>
        
        alt Book is already checked out
            LibraryUtils-->>LibraryLendServlet: LibraryActionResults(BOOK_CHECKED_OUT)
            LibraryLendServlet->>Browser: Renders result page (Book checked out)
        else Book is available
            LibraryUtils->>PersistenceLayer: createLoan(book, borrower, borrowDate)
            PersistenceLayer->>Database: INSERT INTO library.loan (...) VALUES (?, ?, ?)
            Database-->>PersistenceLayer: new loan_id
            PersistenceLayer-->>LibraryUtils: void

            LibraryUtils-->>LibraryLendServlet: LibraryActionResults(SUCCESS)
            LibraryLendServlet->>Browser: Renders result page (Success)
        end
    end
    Browser->>Librarian: Displays result page
```

### 3. Web App: API Call to List Available Books

**Workflow Purpose and Trigger:**
This workflow provides a RESTful API endpoint for clients to retrieve a list of all books that are not currently loaned out. It's triggered by an HTTP GET request to the `/listavailable` endpoint. This is used by the `library.js` script to populate autocomplete/dropdown UI elements dynamically.

**Communication Patterns:**
- **Client-Server:** Asynchronous HTTP GET (AJAX) from browser JavaScript.
- **Internal:** Synchronous Java method calls.
- **Data Persistence:** Synchronous JDBC SELECT query with a JOIN to the H2 database.

```mermaid
sequenceDiagram
    participant BrowserJS as "Browser (library.js)"
    participant WebServer as Tomcat
    participant LibraryBookListAvailableServlet as ListAvailableServlet
    participant LibraryUtils
    participant PersistenceLayer as IPersistenceLayer
    participant Database as H2 Database

    BrowserJS->>WebServer: Async HTTP GET /listavailable
    WebServer->>ListAvailableServlet: doGet(request, response)
    ListAvailableServlet->>LibraryUtils: listAvailableBooks()
    LibraryUtils->>PersistenceLayer: listAvailableBooks()
    PersistenceLayer->>Database: SELECT b.id, b.title FROM library.book b LEFT JOIN library.loan l ON b.id = l.book WHERE l.borrow_date IS NULL
    Database-->>PersistenceLayer: ResultSet of available books
    PersistenceLayer-->>LibraryUtils: Optional<List<Book>>
    LibraryUtils-->>ListAvailableServlet: List<Book>

    alt No books available
        ListAvailableServlet->>BrowserJS: Renders result page ("No books exist...")
    else Books are available
        ListAvailableServlet->>ListAvailableServlet: Formats List<Book> into JSON String
        ListAvailableServlet->>BrowserJS: Renders result page (JSON Array of Books)
    end
    BrowserJS->>BrowserJS: Parses JSON and updates UI (e.g., populates dropdown)
```

### 4. Desktop App: Calculate Auto Insurance Premium

**Workflow Purpose and Trigger:**
This workflow calculates an auto insurance premium adjustment based on the number of previous claims and the driver's age. It is triggered when a user clicks the "Crunch" button in the Java Swing `AutoInsuranceUI`. This is a self-contained, synchronous process within a single application.

**Communication Patterns:**
- **UI Interaction:** Synchronous Java Swing Event (ActionListener).
- **Internal:** Synchronous Java method calls within the same process.
- **Data Persistence:** None. This is a pure computational workflow.

```mermaid
sequenceDiagram
    actor User
    participant AutoInsuranceUI
    participant ActionListener
    participant AutoInsuranceProcessor

    User->>AutoInsuranceUI: Enters Age and selects Claims
    User->>AutoInsuranceUI: Clicks "Crunch" button
    AutoInsuranceUI->>ActionListener: actionPerformed(event)
    
    ActionListener->>AutoInsuranceUI: getIntPreviousClaims()
    AutoInsuranceUI-->>ActionListener: int (claims)
    
    ActionListener->>AutoInsuranceUI: ageField.getText()
    AutoInsuranceUI-->>ActionListener: String (age)

    ActionListener->>AutoInsuranceProcessor: process(claims, age)
    note right of AutoInsuranceProcessor: Business logic (if/else checks) is executed
    AutoInsuranceProcessor-->>ActionListener: AutoInsuranceAction object
    
    ActionListener->>AutoInsuranceUI: setLabel("Premium increase: $...")
    AutoInsuranceUI->>User: Updates result label on screen
```

### 5. Desktop App: Remote UI Automation via Socket Server

**Workflow Purpose and Trigger:**
This workflow allows an external test script to remotely control the `AutoInsuranceUI` for automated testing. It is triggered when a client (`AutoInsuranceScriptClient`) sends a command string over a TCP socket connection. An embedded server (`AutoInsuranceScriptServer`) running in a background thread within the desktop app processes the command and manipulates the UI components directly.

**Communication Patterns:**
- **Client-Server:** Synchronous request/response over a TCP socket.
- **Internal:** Inter-thread communication within the desktop application process (Test Thread -> UI Main Thread). UI updates are handled by calling Swing component methods.
- **Event-Driven:** The `AutoInsuranceScriptServer` runs in a loop, waiting for incoming socket connections and data events.

```mermaid
sequenceDiagram
    participant TestScript as "DesktopTester"
    participant ScriptClient as "AutoInsuranceScriptClient"
    participant ScriptServer as "AutoInsuranceScriptServer (Thread)"
    participant AutoInsuranceUI as "AutoInsuranceUI (Swing EDT)"

    TestScript->>ScriptClient: setAge(22)
    ScriptClient->>ScriptClient: send("set age 22")
    
    note over ScriptClient, ScriptServer: TCP Socket Connection (localhost:8000)
    
    ScriptClient->>ScriptServer: Sends command string "set age 22"
    ScriptServer->>ScriptServer: Parses command
    ScriptServer->>AutoInsuranceUI: setClaimsAge("22")
    
    note right of AutoInsuranceUI: UI component (ageField) text is updated
    
    ScriptServer-->>ScriptClient: Sends response string "OK"
    ScriptClient-->>TestScript: returns "OK"

    TestScript->>ScriptClient: clickCalculate()
    ScriptClient->>ScriptClient: send("click calculate")
    ScriptClient->>ScriptServer: Sends command string "click calculate"
    ScriptServer->>ScriptServer: Parses command
    ScriptServer->>AutoInsuranceUI: claimsCalcButton.doClick()

    note right of AutoInsuranceUI: Simulates a button click, triggering<br/>the insurance calculation workflow.
    
    ScriptServer-->>ScriptClient: Sends response string "OK"
    ScriptClient-->>TestScript: returns "OK"
```