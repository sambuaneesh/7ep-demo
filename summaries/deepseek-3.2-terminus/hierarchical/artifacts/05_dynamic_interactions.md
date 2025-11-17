```markdown
# Dynamic Interaction Flows and Sequence Diagrams

## Workflow 1: User Registration and Authentication

### Description
This workflow handles new user registration and subsequent authentication within the library management system. It ensures secure user onboarding with password strength validation and prevents duplicate registrations.

**Triggers**: User submits registration form with username and password
**Communication Patterns**: Synchronous REST calls, database transactions, password entropy validation

```mermaid
sequenceDiagram
    actor User as User
    participant RegisterServlet as RegisterServlet
    participant RegistrationUtils as RegistrationUtils
    participant IPersistenceLayer as IPersistenceLayer
    participant Database as Database
    participant LoginServlet as LoginServlet
    participant LoginUtils as LoginUtils

    Note over User, Database: Registration Phase
    User->>RegisterServlet: POST /register (username, password)
    RegisterServlet->>RegisterServlet: sanitizeInputs()
    RegisterServlet->>RegistrationUtils: registerUser(username, password)
    
    RegistrationUtils->>RegistrationUtils: validatePasswordStrength(password)
    RegistrationUtils->>IPersistenceLayer: searchForUser(username)
    IPersistenceLayer->>Database: SELECT user by username
    Database-->>IPersistenceLayer: User data
    IPersistenceLayer-->>RegistrationUtils: User object
    
    alt User already exists
        RegistrationUtils-->>RegisterServlet: RegistrationResult(ALREADY_REGISTERED)
    else Password too weak
        RegistrationUtils-->>RegisterServlet: RegistrationResult(BAD_PASSWORD)
    else Registration successful
        RegistrationUtils->>IPersistenceLayer: createUser(username, hashedPassword)
        IPersistenceLayer->>Database: INSERT user
        Database-->>IPersistenceLayer: Success
        RegistrationUtils-->>RegisterServlet: RegistrationResult(SUCCESSFULLY_REGISTERED)
    end
    
    RegisterServlet-->>User: Registration status page

    Note over User, Database: Authentication Phase
    User->>LoginServlet: POST /login (username, password)
    LoginServlet->>LoginServlet: sanitizeInputs()
    LoginServlet->>LoginUtils: authenticateUser(username, password)
    
    LoginUtils->>IPersistenceLayer: searchForUser(username)
    IPersistenceLayer->>Database: SELECT user by username
    Database-->>IPersistenceLayer: User data with hashed password
    IPersistenceLayer-->>LoginUtils: User object
    
    LoginUtils->>LoginUtils: verifyPassword(password, hashedPassword)
    alt Credentials valid
        LoginUtils-->>LoginServlet: Authentication successful
    else Invalid credentials
        LoginUtils-->>LoginServlet: Authentication failed
    end
    
    LoginServlet-->>User: Login result page
```

## Workflow 2: Book Lending (Check-out) Process

### Description
This workflow manages the complete book lending lifecycle, including borrower validation, book availability checking, and loan record creation with date tracking.

**Triggers**: Library staff initiates book check-out for registered borrower
**Communication Patterns**: Synchronous REST calls, database transactions, date arithmetic

```mermaid
sequenceDiagram
    actor Staff as Library Staff
    participant LendServlet as LibraryLendServlet
    participant LibraryUtils as LibraryUtils
    participant IPersistenceLayer as IPersistenceLayer
    participant Database as Database

    Staff->>LendServlet: POST /lend (bookTitle, borrowerName)
    LendServlet->>LendServlet: validateParameters(bookTitle, borrowerName)
    
    LendServlet->>LibraryUtils: lendBook(bookTitle, borrowerName)
    
    LibraryUtils->>IPersistenceLayer: searchBookByTitle(bookTitle)
    IPersistenceLayer->>Database: SELECT book by title
    Database-->>IPersistenceLayer: Book data
    IPersistenceLayer-->>LibraryUtils: Book object
    
    LibraryUtils->>IPersistenceLayer: searchBorrowerByName(borrowerName)
    IPersistenceLayer->>Database: SELECT borrower by name
    Database-->>IPersistenceLayer: Borrower data
    IPersistenceLayer-->>LibraryUtils: Borrower object
    
    alt Book not found
        LibraryUtils-->>LendServlet: LibraryActionResults.BOOK_NOT_REGISTERED
    else Borrower not found
        LibraryUtils-->>LendServlet: LibraryActionResults.BORROWER_NOT_REGISTERED
    else Book already checked out
        LibraryUtils->>IPersistenceLayer: searchLoanByBookId(bookId)
        IPersistenceLayer->>Database: SELECT active loan by book_id
        Database-->>IPersistenceLayer: Loan data
        IPersistenceLayer-->>LibraryUtils: Loan object
        LibraryUtils-->>LendServlet: LibraryActionResults.BOOK_CHECKED_OUT
    else Success path
        LibraryUtils->>DateUtils: getCurrentDate()
        DateUtils-->>LibraryUtils: currentDate
        
        LibraryUtils->>IPersistenceLayer: createLoan(book, borrower, currentDate)
        IPersistenceLayer->>Database: INSERT loan record
        Database-->>IPersistenceLayer: Success
        IPersistenceLayer-->>LibraryUtils: Loan object
        
        LibraryUtils-->>LendServlet: LibraryActionResults.SUCCESS
    end
    
    LendServlet-->>Staff: Lending result page
```

## Workflow 3: Mathematical Computation Service

### Description
This workflow demonstrates educational mathematical computations (Fibonacci, Ackermann) through web interfaces, showcasing different algorithm implementations and computational efficiency.

**Triggers**: User requests mathematical computation via web form
**Communication Patterns**: Synchronous REST calls, algorithm processing, BigInteger operations

```mermaid
sequenceDiagram
    actor User as User
    participant MathServlet as MathServlet
    participant Calculator as Calculator
    participant FibServlet as FibServlet
    participant Fibonacci as Fibonacci
    participant AckServlet as AckServlet
    participant Ackermann as Ackermann

    Note over User, Ackermann: Basic Arithmetic
    User->>MathServlet: POST /math (operation, numbers)
    MathServlet->>MathServlet: parseParameters()
    MathServlet->>Calculator: add(number1, number2)
    Calculator-->>MathServlet: Result
    MathServlet-->>User: Calculation result

    Note over User, Ackermann: Fibonacci Sequence
    User->>FibServlet: POST /fib (n, algorithm)
    FibServlet->>FibServlet: validateInput(n)
    
    alt algorithm = "iterative"
        FibServlet->>Fibonacci: fibAlgo2(n)
    else algorithm = "tailrecursive"
        FibServlet->>TailRecursive: fib(n)
    else default recursive
        FibServlet->>Fibonacci: fib(n)
    end
    
    Fibonacci->>Fibonacci: computeFibonacci(n)
    Fibonacci-->>FibServlet: BigInteger result
    FibServlet-->>User: Fibonacci number

    Note over User, Ackermann: Ackermann Function
    User->>AckServlet: POST /ack (m, n)
    AckServlet->>AckServlet: validateInputs(m, n)
    
    alt algorithm = "iterative"
        AckServlet->>AckermannIterative: ack(m, n)
    else default recursive
        AckServlet->>Ackermann: ack(m, n)
    end
    
    Ackermann->>Ackermann: computeAckermann(m, n)
    Ackermann-->>AckServlet: BigInteger result
    AckServlet-->>User: Ackermann function result
```

## Workflow 4: Auto Insurance Risk Assessment

### Description
This workflow processes auto insurance applications by evaluating risk factors based on driver age and claim history, determining premium adjustments and policy actions.

**Triggers**: User submits insurance application with age and claims data
**Communication Patterns**: Synchronous method calls, risk matrix evaluation, policy decision logic

```mermaid
sequenceDiagram
    actor User as User
    participant AutoInsuranceUI as AutoInsuranceUI
    participant AutoInsuranceProcessor as AutoInsuranceProcessor
    participant AutoInsuranceAction as AutoInsuranceAction

    User->>AutoInsuranceUI: Input age and previous claims
    AutoInsuranceUI->>AutoInsuranceUI: validateInputs(age, claims)
    
    AutoInsuranceUI->>AutoInsuranceProcessor: calculateAction(age, claims)
    
    AutoInsuranceProcessor->>AutoInsuranceProcessor: validateAgeRange(age)
    AutoInsuranceProcessor->>AutoInsuranceProcessor: validateClaimsCount(claims)
    
    AutoInsuranceProcessor->>AutoInsuranceProcessor: assessRisk(age, claims)
    
    alt claims >= 5
        AutoInsuranceProcessor->>AutoInsuranceProcessor: cancelPolicy()
        AutoInsuranceProcessor->>AutoInsuranceAction: createCancelledAction()
    else claims >= 3
        AutoInsuranceProcessor->>AutoInsuranceProcessor: applyPremiumSurcharge()
        AutoInsuranceProcessor->>AutoInsuranceProcessor: escalateWarningLetter()
        AutoInsuranceProcessor->>AutoInsuranceAction: createWarningAction()
    else claims >= 1
        AutoInsuranceProcessor->>AutoInsuranceProcessor: applyModerateSurcharge()
        AutoInsuranceProcessor->>AutoInsuranceProcessor: issueWarningLetter()
        AutoInsuranceProcessor->>AutoInsuranceAction: createWarningAction()
    else no claims
        AutoInsuranceProcessor->>AutoInsuranceAction: createStandardAction()
    end
    
    AutoInsuranceAction-->>AutoInsuranceProcessor: InsuranceAction object
    AutoInsuranceProcessor-->>AutoInsuranceUI: AutoInsuranceAction
    
    AutoInsuranceUI->>AutoInsuranceUI: displayResults(premiumChange, warningLevel, cancellation)
    AutoInsuranceUI-->>User: Insurance decision
```

## Workflow 5: Database Initialization and Migration

### Description
This workflow manages database state during application startup and schema evolution, ensuring data integrity and supporting development/testing workflows.

**Triggers**: Tomcat application startup or administrative database operation
**Communication Patterns**: Event-driven lifecycle, synchronous database operations, Flyway migrations

```mermaid
sequenceDiagram
    participant Tomcat as Tomcat Container
    participant WebAppListener as WebAppListener
    participant DbServlet as DbServlet
    participant PersistenceLayer as PersistenceLayer
    participant Flyway as Flyway Migrations
    participant Database as Database

    Note over Tomcat, Database: Application Startup
    Tomcat->>WebAppListener: contextInitialized()
    WebAppListener->>PersistenceLayer: initialize()
    
    PersistenceLayer->>Flyway: clean()
    Flyway->>Database: DROP existing tables
    Database-->>Flyway: Success
    
    PersistenceLayer->>Flyway: migrate()
    Flyway->>Database: APPLY schema migrations
    Database-->>Flyway: Success
    Flyway-->>PersistenceLayer: Migration complete
    
    PersistenceLayer-->>WebAppListener: Initialization complete

    Note over Tomcat, Database: Administrative Operations
    actor Admin as Administrator
    Admin->>DbServlet: GET /db?action=clean
    DbServlet->>DbServlet: validateAdminAccess()
    DbServlet->>PersistenceLayer: cleanDatabase()
    
    PersistenceLayer->>Flyway: clean()
    Flyway->>Database: DROP all tables
    Database-->>Flyway: Success
    Flyway-->>PersistenceLayer: Clean complete
    
    PersistenceLayer-->>DbServlet: Success
    DbServlet-->>Admin: Database cleaned
    
    Admin->>DbServlet: GET /db?action=migrate
    DbServlet->>PersistenceLayer: migrateDatabase()
    PersistenceLayer->>Flyway: migrate()
    Flyway->>Database: APPLY migrations
    Database-->>Flyway: Success
    Flyway-->>PersistenceLayer: Migration complete
    PersistenceLayer-->>DbServlet: Success
    DbServlet-->>Admin: Database migrated
```

## Workflow 6: Comprehensive Library Search Operations

### Description
This workflow handles multiple search patterns for library resources including books, borrowers, and availability status, supporting both administrative and patron use cases.

**Triggers**: User searches for books or borrowers using various criteria
**Communication Patterns**: Synchronous REST calls, database queries, JSON response formatting

```mermaid
sequenceDiagram
    actor User as User/Staff
    participant SearchServlet as LibraryBookListSearchServlet
    participant BorrowerServlet as LibraryBorrowerListSearchServlet
    participant AvailableServlet as LibraryBookListAvailableServlet
    participant LibraryUtils as LibraryUtils
    participant IPersistenceLayer as IPersistenceLayer
    participant Database as Database

    Note over User, Database: Book Search Operations
    User->>SearchServlet: GET /booksearch?id=123
    SearchServlet->>SearchServlet: parseSearchParameters()
    SearchServlet->>LibraryUtils: searchBookById(123)
    LibraryUtils->>IPersistenceLayer: searchBookById(123)
    IPersistenceLayer->>Database: SELECT book by id
    Database-->>IPersistenceLayer: Book data
    IPersistenceLayer-->>LibraryUtils: Book object
    LibraryUtils-->>SearchServlet: Book object
    SearchServlet->>SearchServlet: formatJsonResponse(book)
    SearchServlet-->>User: JSON book data

    User->>SearchServlet: GET /booksearch?title=Programming
    SearchServlet->>LibraryUtils: searchBookByTitle("Programming")
    LibraryUtils->>IPersistenceLayer: searchBookByTitle("Programming")
    IPersistenceLayer->>Database: SELECT books by title pattern
    Database-->>IPersistenceLayer: List of books
    IPersistenceLayer-->>LibraryUtils: List<Book>
    LibraryUtils-->>SearchServlet: List<Book>
    SearchServlet-->>User: JSON book list

    Note over User, Database: Borrower Search Operations
    User->>BorrowerServlet: GET /borrowersearch?name=John
    BorrowerServlet->>LibraryUtils: searchBorrowerByName("John")
    LibraryUtils->>IPersistenceLayer: searchBorrowerByName("John")
    IPersistenceLayer->>Database: SELECT borrower by name
    Database-->>IPersistenceLayer: Borrower data
    IPersistenceLayer-->>LibraryUtils: Borrower object
    LibraryUtils-->>BorrowerServlet: Borrower object
    BorrowerServlet-->>User: JSON borrower data

    Note over User, Database: Availability Check
    User->>AvailableServlet: GET /availablebooks
    AvailableServlet->>LibraryUtils: listAllAvailableBooks()
    LibraryUtils->>IPersistenceLayer: listAllBooks()
    IPersistenceLayer->>Database: SELECT all books
    Database-->>IPersistenceLayer: List<Book>
    IPersistenceLayer-->>LibraryUtils: List<Book>
    
    LibraryUtils->>LibraryUtils: filterAvailableBooks(books)
    LibraryUtils->>IPersistenceLayer: searchActiveLoans()
    IPersistenceLayer->>Database: SELECT active loans
    Database-->>IPersistenceLayer: List<Loan>
    IPersistenceLayer-->>LibraryUtils: List<Loan>
    
    LibraryUtils->>LibraryUtils: computeAvailability(books, loans)
    LibraryUtils-->>AvailableServlet: List<Book> available books
    AvailableServlet-->>User: JSON available books list
```

## Workflow 7: Error Handling and Recovery Patterns

### Description
This workflow demonstrates the comprehensive error handling and recovery mechanisms throughout the system, including input validation, database error recovery, and exception propagation.

**Triggers**: Various error conditions during system operations
**Communication Patterns**: Exception propagation, validation chains, error response formatting

```mermaid
sequenceDiagram
    actor User as User
    participant Servlet as Any Servlet
    participant CheckUtils as CheckUtils
    participant StringUtils as StringUtils
    participant BusinessLogic as Business Logic
    participant IPersistenceLayer as IPersistenceLayer
    participant Database as Database

    Note over User, Database: Input Validation Chain
    User->>Servlet: POST with invalid input
    Servlet->>Servlet: extractParameters()
    Servlet->>StringUtils: isStringEmpty(input)
    StringUtils-->>Servlet: true (empty)
    Servlet->>CheckUtils: checkStringNotEmpty(input, "Parameter")
    CheckUtils->>CheckUtils: throw AssertionException if empty
    
    Note over User, Database: Business Rule Validation
    User->>Servlet: POST with business rule violation
    Servlet->>BusinessLogic: processRequest(data)
    BusinessLogic->>BusinessLogic: validateBusinessRules(data)
    alt Duplicate registration
        BusinessLogic->>IPersistenceLayer: searchForExisting(data)
        IPersistenceLayer->>Database: SELECT existing record
        Database-->>IPersistenceLayer: Existing record found
        IPersistenceLayer-->>BusinessLogic: Existing object
        BusinessLogic->>BusinessLogic: throw DomainSpecificException
    else Invalid state transition
        BusinessLogic->>BusinessLogic: validateStateTransition()
        BusinessLogic->>BusinessLogic: throw InvalidStateException
    end
    
    Note over User, Database: Database Error Recovery
    BusinessLogic->>IPersistenceLayer: performOperation(data)
    IPersistenceLayer->>Database: EXECUTE operation
    Database->>Database: SQL Exception
    Database-->>IPersistenceLayer: SQLException
    IPersistenceLayer->>IPersistenceLayer: wrapException(SQLException)
    IPersistenceLayer->>IPersistenceLayer: throw SqlRuntimeException
    
    BusinessLogic->>BusinessLogic: catch SqlRuntimeException
    BusinessLogic->>BusinessLogic: logError(exception)
    BusinessLogic->>BusinessLogic: initiateRecovery()
    
    alt Retry possible
        BusinessLogic->>IPersistenceLayer: retryOperation(data)
        IPersistenceLayer->>Database: RETRY operation
        Database-->>IPersistenceLayer: Success
        IPersistenceLayer-->>BusinessLogic: Success
    else Fallback strategy
        BusinessLogic->>BusinessLogic: executeFallbackStrategy()
        BusinessLogic-->>Servlet: Error result with fallback data
    end
    
    Servlet->>Servlet: formatErrorResponse(exception)
    Servlet-->>User: Error page with appropriate message
```

## Communication Patterns Summary

### Synchronous Communication
- **REST API Calls**: All servlet endpoints use synchronous HTTP request-response patterns
- **Database Transactions**: Direct synchronous database operations with prepared statements
- **Method Invocations**: Internal service layer calls between components

### Asynchronous Patterns
- **Event Listeners**: Tomcat lifecycle events for database initialization
- **Background Processing**: Mathematical computations that may run asynchronously for large inputs

### Data Flow Patterns
- **Immutable Data Transfer**: Domain objects flow between layers without modification
- **Parameter Validation Chains**: Multi-layer validation from presentation to persistence
- **Status-Based Communication**: Standardized result objects for consistent error handling

### Error Handling Strategies
- **Defensive Programming**: Comprehensive input validation at all layers
- **Exception Wrapping**: Database exceptions converted to runtime exceptions
- **Recovery Mechanisms**: Retry logic and fallback strategies for database operations
```