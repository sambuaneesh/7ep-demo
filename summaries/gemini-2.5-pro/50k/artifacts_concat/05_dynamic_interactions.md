### 1. User Registration Workflow

This workflow describes the process of a new user creating an account. It includes validation for password strength and checks for existing usernames to prevent duplicates.

-   **Trigger**: User submits the registration form via an HTTP POST request to the `/register` endpoint.
-   **Communication Patterns**: Synchronous REST call (HTTP POST), internal Java method calls, JDBC database transaction (SELECT then INSERT).

```mermaid
sequenceDiagram
    actor User
    participant RegisterServlet as Web Layer
    participant RegistrationUtils as Business Logic
    participant PersistenceLayer as Data Access
    participant Database

    User->>+RegisterServlet: POST /register (username, password)
    RegisterServlet->>+RegistrationUtils: registerUser(username, password)
    
    RegistrationUtils->>RegistrationUtils: isPasswordGood(password)?
    note right of RegistrationUtils: Checks length and entropy
    
    RegistrationUtils->>+PersistenceLayer: searchForUserByName(username)
    PersistenceLayer->>+Database: SELECT * FROM users WHERE name = ?
    Database-->>-PersistenceLayer: (User record or empty)
    PersistenceLayer-->>-RegistrationUtils: Optional<User>
    
    alt User Already Exists
        RegistrationUtils-->>RegisterServlet: RegistrationResult(USER_EXISTS)
    else User Does Not Exist
        RegistrationUtils->>RegistrationUtils: hashPassword(password)
        RegistrationUtils->>+PersistenceLayer: saveNewUser(username, hashedPassword)
        PersistenceLayer->>+Database: INSERT INTO users (name, password_hash) VALUES (?, ?)
        Database-->>-PersistenceLayer: (Success)
        PersistenceLayer-->>-RegistrationUtils: (Success)
        RegistrationUtils-->>-RegisterServlet: RegistrationResult(SUCCESS)
    end
    
    RegisterServlet-->>-User: HTTP 200 OK (Renders result page)

```

### 2. User Login Workflow

This workflow details the process of an existing user authenticating with the system. It handles both successful and failed login attempts.

-   **Trigger**: User submits the login form via an HTTP POST request to the `/login` endpoint.
-   **Communication Patterns**: Synchronous REST call (HTTP POST), internal Java method calls, JDBC database query.

```mermaid
sequenceDiagram
    actor User
    participant LoginServlet as Web Layer
    participant LoginUtils as Business Logic
    participant PersistenceLayer as Data Access
    participant Database

    User->>+LoginServlet: POST /login (username, password)
    LoginServlet->>+LoginUtils: isUserRegistered(username, password)
    LoginUtils->>+PersistenceLayer: areCredentialsValid(username, password)
    
    PersistenceLayer->>+Database: SELECT password_hash FROM users WHERE name = ?
    Database-->>-PersistenceLayer: (Hashed password or empty)
    
    alt User Found
        PersistenceLayer->>PersistenceLayer: hash(provided_password)
        PersistenceLayer->>PersistenceLayer: compare(hashed_provided_password, db_hash)
        
        alt Passwords Match
            PersistenceLayer-->>LoginUtils: true
        else Passwords Do Not Match
            PersistenceLayer-->>LoginUtils: false
        end

    else User Not Found
        PersistenceLayer-->>LoginUtils: false
    end
    
    LoginUtils-->>-LoginServlet: boolean (isAuthenticated)
    
    alt Authentication Successful
        LoginServlet-->>-User: HTTP 200 OK (Renders "Access Granted")
    else Authentication Failed
        LoginServlet-->>-User: HTTP 200 OK (Renders "Access Denied")
    end

```

### 3. Lend a Book Workflow

This is a critical business transaction that involves multiple validation steps before a book can be loaned to a borrower. It demonstrates complex data interactions and error handling.

-   **Trigger**: A user submits a form to lend a specific book to a specific borrower via a POST to `/lend`.
-   **Communication Patterns**: Synchronous REST call (HTTP POST), sequential internal method calls, and multiple JDBC queries that form a single business transaction.

```mermaid
sequenceDiagram
    actor User
    participant LibraryLendServlet as Web Layer
    participant LibraryUtils as Business Logic
    participant PersistenceLayer as Data Access
    participant Database

    User->>+LibraryLendServlet: POST /lend (bookTitle, borrowerName)
    LibraryLendServlet->>+LibraryUtils: lendBook(bookTitle, borrowerName)

    note over LibraryUtils: Begin Business Transaction
    
    LibraryUtils->>+PersistenceLayer: searchBooksByTitle(bookTitle)
    PersistenceLayer->>+Database: SELECT * FROM books WHERE title = ?
    Database-->>-PersistenceLayer: (Book record or empty)
    PersistenceLayer-->>-LibraryUtils: Optional<Book>

    alt Book Not Found
        LibraryUtils-->>LibraryLendServlet: LibraryActionResults.BOOK_NOT_REGISTERED
        LibraryLendServlet-->>-User: HTTP 200 OK (Error Message)
    else Book Found
        LibraryUtils->>+PersistenceLayer: searchBorrowerDataByName(borrowerName)
        PersistenceLayer->>+Database: SELECT * FROM borrowers WHERE name = ?
        Database-->>-PersistenceLayer: (Borrower record or empty)
        PersistenceLayer-->>-LibraryUtils: Optional<Borrower>

        alt Borrower Not Found
            LibraryUtils-->>LibraryLendServlet: LibraryActionResults.BORROWER_NOT_REGISTERED
            LibraryLendServlet-->>-User: HTTP 200 OK (Error Message)
        else Borrower Found
            LibraryUtils->>+PersistenceLayer: searchForLoanByBook(book)
            PersistenceLayer->>+Database: SELECT * FROM loans WHERE book_id = ?
            Database-->>-PersistenceLayer: (Loan record or empty)
            PersistenceLayer-->>-LibraryUtils: Optional<Loan>
            
            alt Book Already On Loan
                LibraryUtils-->>LibraryLendServlet: LibraryActionResults.BOOK_CHECKED_OUT
                LibraryLendServlet-->>-User: HTTP 200 OK (Error Message)
            else Book is Available
                LibraryUtils->>+PersistenceLayer: createLoan(book, borrower, currentDate)
                PersistenceLayer->>+Database: INSERT INTO loans (book, borrower, borrow_date) VALUES (?, ?, ?)
                Database-->>-PersistenceLayer: (Success)
                PersistenceLayer-->>-LibraryUtils: (Success)
                LibraryUtils-->>-LibraryLendServlet: LibraryActionResults.SUCCESS
                LibraryLendServlet-->>-User: HTTP 200 OK (Success Message)
            end
        end
    end

```

### 4. List Available Books Workflow

This flow shows a read-only operation where the client requests a list of all books that are not currently checked out.

-   **Trigger**: User's browser makes an asynchronous GET request to `/library-books-available`.
-   **Communication Patterns**: Asynchronous XHR/Fetch call from client, Synchronous REST call (HTTP GET), internal method calls, single complex JDBC query.

```mermaid
sequenceDiagram
    actor User (Browser JS)
    participant LibraryBookListAvailableServlet as Web Layer
    participant LibraryUtils as Business Logic
    participant PersistenceLayer as Data Access
    participant Database

    User->>+LibraryBookListAvailableServlet: GET /library-books-available
    LibraryBookListAvailableServlet->>+LibraryUtils: listAvailableBooks()
    LibraryUtils->>+PersistenceLayer: listAvailableBooks()
    
    note right of PersistenceLayer: Query finds books that do not have a corresponding entry in the loans table.
    PersistenceLayer->>+Database: SELECT b.* FROM books b LEFT JOIN loans l ON b.id = l.book WHERE l.id IS NULL
    Database-->>-PersistenceLayer: ResultSet of available books
    
    PersistenceLayer->>PersistenceLayer: Map ResultSet to List<Book>
    PersistenceLayer-->>-LibraryUtils: List<Book>
    LibraryUtils-->>-LibraryBookListAvailableServlet: List<Book>
    
    LibraryBookListAvailableServlet->>LibraryBookListAvailableServlet: Serialize List<Book> to JSON
    LibraryBookListAvailableServlet-->>-User: HTTP 200 OK (JSON Payload)
```

### 5. Stateless Computation Workflow (Fibonacci)

This diagram illustrates a purely computational, stateless workflow that has no interaction with the persistence layer. It highlights how different domains within the monolith can have vastly different interaction patterns.

-   **Trigger**: User submits a form to calculate a Fibonacci number via POST to `/fibonacci`.
-   **Communication Patterns**: Synchronous REST call (HTTP POST), internal method calls. No database or external service interaction.

```mermaid
sequenceDiagram
    actor User
    participant FibServlet as Web Layer
    participant FibonacciIterative as Business Logic (Algorithm)
    participant Fibonacci as Business Logic (Algorithm)

    User->>+FibServlet: POST /fibonacci (n, algorithm_choice)
    FibServlet->>FibServlet: Parse parameters n and algorithm_choice
    
    alt Algorithm is 'iterative'
        FibServlet->>+FibonacciIterative: calculate(n)
        FibonacciIterative-->>-FibServlet: BigInteger result
    else Algorithm is 'recursive'
        FibServlet->>+Fibonacci: calculate(n)
        note right of Fibonacci: This can be very slow for large n
        Fibonacci-->>-FibServlet: BigInteger result
    end

    FibServlet->>FibServlet: Format result for display
    FibServlet-->>-User: HTTP 200 OK (Renders result page)

```