# Dynamic Interaction Flows and Sequence Diagrams

## Workflow 1: User Registration and Authentication

### Description
Complete user registration and login workflow with password validation and credential storage. This workflow demonstrates synchronous REST communication with database transactions and password strength analysis.

### Communication Patterns
- Synchronous REST API calls
- Database transactions with prepared statements
- Password entropy calculation (synchronous)
- Session management via cookies

```mermaid
sequenceDiagram
    participant User as User/Browser
    participant RegisterServlet as RegisterServlet
    participant RegistrationUtils as RegistrationUtils
    participant PersistenceLayer as PersistenceLayer
    participant H2DB as H2 Database
    participant LoginServlet as LoginServlet
    participant LoginUtils as LoginUtils

    Note over User,H2DB: Registration Phase
    User->>RegisterServlet: POST /demo/register (username, password)
    RegisterServlet->>RegistrationUtils: validateRegistration(username, password)
    RegistrationUtils->>RegistrationUtils: checkPasswordStrength(password)
    RegistrationUtils->>RegistrationUtils: calculateEntropy(password)
    RegistrationUtils->>PersistenceLayer: checkUserExists(username)
    PersistenceLayer->>H2DB: SELECT FROM AUTH.USER WHERE name=?
    H2DB-->>PersistenceLayer: User record (if exists)
    alt User does not exist
        RegistrationUtils->>PersistenceLayer: createUser(username, passwordHash)
        PersistenceLayer->>H2DB: INSERT INTO AUTH.USER (name, password_hash)
        H2DB-->>PersistenceLayer: User ID
        RegistrationUtils-->>RegisterServlet: RegistrationResult(SUCCESS)
    else User exists
        RegistrationUtils-->>RegisterServlet: RegistrationResult(USER_EXISTS)
    end
    RegisterServlet-->>User: Registration status with entropy analysis

    Note over User,H2DB: Login Phase
    User->>LoginServlet: POST /demo/login (username, password)
    LoginServlet->>LoginUtils: authenticateUser(username, password)
    LoginUtils->>PersistenceLayer: getUserByUsername(username)
    PersistenceLayer->>H2DB: SELECT * FROM AUTH.USER WHERE name=?
    H2DB-->>PersistenceLayer: User record with password_hash
    LoginUtils->>LoginUtils: verifyPasswordHash(password, storedHash)
    alt Valid credentials
        LoginUtils-->>LoginServlet: AuthenticationResult(SUCCESS)
        LoginServlet->>LoginServlet: createSessionCookie()
    else Invalid credentials
        LoginUtils-->>LoginServlet: AuthenticationResult(INVALID_CREDENTIALS)
    end
    LoginServlet-->>User: Login status with session cookie
```

## Workflow 2: Library Book Lending Process

### Description
Complete book lending workflow including borrower search, book availability check, and loan creation. Demonstrates complex business rules with database transactions.

### Communication Patterns
- REST API calls with JSON responses
- Database transactions with referential integrity
- Client-side autocomplete via AJAX
- Synchronous data validation

```mermaid
sequenceDiagram
    participant Librarian as Librarian/Browser
    participant LibraryJS as library.js
    participant BorrowerServlet as LibraryBorrowerListSearchServlet
    participant BookServlet as LibraryBookListSearchServlet
    participant LendServlet as LibraryLendServlet
    participant LibraryUtils as LibraryUtils
    participant PersistenceLayer as PersistenceLayer
    participant H2DB as H2 Database

    Note over Librarian,H2DB: Borrower Search (Autocomplete)
    Librarian->>LibraryJS: Type borrower name in search field
    LibraryJS->>BorrowerServlet: GET /demo/borrower?name=partial
    BorrowerServlet->>PersistenceLayer: searchBorrowersByName(partialName)
    PersistenceLayer->>H2DB: SELECT FROM LIBRARY.BORROWER WHERE name LIKE ?
    H2DB-->>PersistenceLayer: Borrower results
    PersistenceLayer-->>BorrowerServlet: List<Borrower>
    BorrowerServlet-->>LibraryJS: JSON borrower list
    LibraryJS-->>Librarian: Display autocomplete options

    Note over Librarian,H2DB: Book Search and Availability Check
    Librarian->>LibraryJS: Select book title
    LibraryJS->>BookServlet: GET /demo/book?title=bookTitle
    BookServlet->>PersistenceLayer: searchBooksByTitle(bookTitle)
    PersistenceLayer->>H2DB: SELECT FROM LIBRARY.BOOK WHERE title=?
    H2DB-->>PersistenceLayer: Book record
    BookServlet->>PersistenceLayer: isBookAvailable(bookId)
    PersistenceLayer->>H2DB: SELECT COUNT FROM LIBRARY.LOAN WHERE book=? AND return_date IS NULL
    H2DB-->>PersistenceLayer: Loan count
    BookServlet-->>LibraryJS: Book details with availability status

    Note over Librarian,H2DB: Lending Transaction
    Librarian->>LendServlet: POST /demo/lend (bookId, borrowerId)
    LendServlet->>LibraryUtils: lendBook(bookId, borrowerId)
    LibraryUtils->>PersistenceLayer: beginTransaction()
    LibraryUtils->>PersistenceLayer: verifyBookExists(bookId)
    PersistenceLayer->>H2DB: SELECT FROM LIBRARY.BOOK WHERE id=?
    H2DB-->>PersistenceLayer: Book record
    LibraryUtils->>PersistenceLayer: verifyBorrowerExists(borrowerId)
    PersistenceLayer->>H2DB: SELECT FROM LIBRARY.BORROWER WHERE id=?
    H2DB-->>PersistenceLayer: Borrower record
    LibraryUtils->>PersistenceLayer: checkBookAvailability(bookId)
    PersistenceLayer->>H2DB: SELECT COUNT FROM LIBRARY.LOAN WHERE book=? AND active=true
    H2DB-->>PersistenceLayer: Availability status
    alt Book is available
        LibraryUtils->>PersistenceLayer: createLoan(bookId, borrowerId, currentDate)
        PersistenceLayer->>H2DB: INSERT INTO LIBRARY.LOAN (book, borrower, borrow_date)
        H2DB-->>PersistenceLayer: Loan ID
        LibraryUtils->>PersistenceLayer: commitTransaction()
        LibraryUtils-->>LendServlet: LibraryActionResults(SUCCESS)
    else Book is unavailable
        LibraryUtils->>PersistenceLayer: rollbackTransaction()
        LibraryUtils-->>LendServlet: LibraryActionResults(BOOK_UNAVAILABLE)
    end
    LendServlet-->>Librarian: Lending operation result
```

## Workflow 3: Auto Insurance Premium Calculation

### Description
Desktop application workflow for insurance premium calculation with UI automation support via socket server. Demonstrates Swing UI interactions and custom TCP protocol.

### Communication Patterns
- Desktop UI events (Swing)
- Custom TCP socket protocol
- Business rule processing
- Synchronous calculation responses

```mermaid
sequenceDiagram
    participant User as User
    participant UI as AutoInsuranceUI
    participant Processor as AutoInsuranceProcessor
    participant SocketServer as AutoInsuranceScriptServer
    participant Automation as Automation Client

    Note over User,Processor: Manual UI Interaction
    User->>UI: Enter age and number of claims
    User->>UI: Click "Calculate" button
    UI->>Processor: calculatePremium(age, claims)
    Processor->>Processor: applyBusinessRules(age, claims)
    Note right of Processor: Age 16-25: $50/$100/$400<br/>Age 26-85: $25/$50/$200<br/>5+ claims: Policy canceled
    Processor-->>UI: AutoInsuranceAction(premiumIncrease, warningLetter, policyCanceled)
    UI-->>User: Display premium result and warnings

    Note over Automation,Processor: Automation via Socket Server
    Automation->>SocketServer: TCP connect port 8000
    Automation->>SocketServer: "set age 25"
    SocketServer->>UI: setAgeField(25)
    UI-->>SocketServer: Field updated confirmation
    SocketServer-->>Automation: "OK"
    
    Automation->>SocketServer: "set claims 2"
    SocketServer->>UI: setClaimsField(2)
    UI-->>SocketServer: Field updated confirmation
    SocketServer-->>Automation: "OK"
    
    Automation->>SocketServer: "click calculate"
    SocketServer->>UI: triggerCalculateButton()
    UI->>Processor: calculatePremium(25, 2)
    Processor->>Processor: applyBusinessRules(25, 2)
    Processor-->>UI: AutoInsuranceAction($400 increase, warning letter)
    UI->>SocketServer: calculationComplete(result)
    
    Automation->>SocketServer: "get label"
    SocketServer->>UI: getResultLabelText()
    UI-->>SocketServer: "Premium increase: $400"
    SocketServer-->>Automation: "Premium increase: $400"
    
    Automation->>SocketServer: "quit"
    SocketServer->>SocketServer: closeConnection()
    SocketServer-->>Automation: Connection closed
```

## Workflow 4: Mathematical Computation Services

### Description
Mathematical function computation workflow showing different algorithm implementations (recursive vs iterative) and their performance characteristics.

### Communication Patterns
- REST API calls
- Stateless computation services
- Algorithm selection based on input size
- Performance-optimized implementations

```mermaid
sequenceDiagram
    participant User as User/Browser
    participant MathServlet as MathServlet
    participant FibServlet as FibServlet
    participant AckServlet as AckServlet
    participant Fibonacci as Fibonacci
    participant FibonacciIterative as FibonacciIterative
    participant Ackermann as Ackermann
    participant AckermannIterative as AckermannIterative

    Note over User,AckermannIterative: Basic Arithmetic
    User->>MathServlet: POST /demo/math (operation, operands)
    MathServlet->>MathServlet: validateParameters(operation, operands)
    MathServlet->>Calculator: executeOperation(operation, operands)
    Calculator-->>MathServlet: computation result
    MathServlet-->>User: JSON response with result

    Note over User,AckermannIterative: Fibonacci Sequence
    User->>FibServlet: POST /demo/fibonacci (n=10)
    FibServlet->>FibServlet: validateInput(n)
    alt n <= 20 (small input)
        FibServlet->>Fibonacci: calculate(n) [Recursive]
    else n > 20 (large input)
        FibServlet->>FibonacciIterative: fibAlgo1(n) [Optimized Iterative]
    end
    Fibonacci/FibonacciIterative-->>FibServlet: fibonacci result
    FibServlet-->>User: JSON with fibonacci number and computation time

    Note over User,AckermannIterative: Ackermann Function
    User->>AckServlet: POST /demo/ackermann (m=3, n=2)
    AckServlet->>AckServlet: validateParameters(m, n)
    alt m <= 3 and n <= 10 (moderate size)
        AckServlet->>Ackermann: calculate(m, n) [Recursive]
    else m > 3 or n > 10 (large parameters)
        AckServlet->>AckermannIterative: calculate(m, n) [Iterative with Stack]
    end
    Ackermann/AckermannIterative-->>AckServlet: ackermann result
    AckServlet-->>User: JSON with result and algorithm used
```

## Workflow 5: Database Migration and Management

### Description
Database schema migration and management workflow using Flyway, including both automated startup migrations and manual administrative operations.

### Communication Patterns
- RESTful administrative endpoints
- Database migration framework (Flyway)
- Connection pooling management
- Transactional schema operations

```mermaid
sequenceDiagram
    participant Admin as Administrator
    participant DbServlet as DbServlet
    participant WebAppListener as WebAppListener
    participant PersistenceLayer as PersistenceLayer
    participant Flyway as Flyway
    participant H2DB as H2 Database
    participant ConnectionPool as JdbcConnectionPool

    Note over Admin,H2DB: Application Startup Migration
    WebAppListener->>WebAppListener: contextInitialized()
    WebAppListener->>PersistenceLayer: initializeDataSource()
    PersistenceLayer->>ConnectionPool: createPool(jdbc:h2:mem:training)
    PersistenceLayer->>Flyway: configure(schemas: ADMINISTRATIVE, LIBRARY, AUTH)
    PersistenceLayer->>Flyway: migrate()
    Flyway->>H2DB: SELECT FROM ADMINISTRATIVE.flyway_schema_history
    H2DB-->>Flyway: Migration history
    Flyway->>H2DB: Execute pending migration scripts
    H2DB-->>Flyway: Schema updated
    Flyway-->>PersistenceLayer: Migration successful
    PersistenceLayer-->>WebAppListener: Database ready

    Note over Admin,H2DB: Manual Database Management
    Admin->>DbServlet: GET /demo/flyway?action=clean
    DbServlet->>PersistenceLayer: cleanDatabase()
    PersistenceLayer->>Flyway: clean()
    Flyway->>H2DB: DROP ALL OBJECTS
    H2DB-->>Flyway: Database cleaned
    Flyway-->>PersistenceLayer: Clean completed
    PersistenceLayer-->>DbServlet: Success
    DbServlet-->>Admin: Database cleaned successfully

    Admin->>DbServlet: GET /demo/flyway?action=migrate
    DbServlet->>PersistenceLayer: migrateDatabase()
    PersistenceLayer->>Flyway: migrate()
    Flyway->>H2DB: Execute all migration scripts
    H2DB-->>Flyway: Schemas created: AUTH, LIBRARY, ADMINISTRATIVE
    Flyway-->>PersistenceLayer: Migration completed
    PersistenceLayer-->>DbServlet: Migration successful
    DbServlet-->>Admin: Database migrated to latest version
```

## Workflow 6: CI/CD Testing Pipeline

### Description
End-to-end CI/CD testing workflow showing the complete testing pipeline from code commit to production deployment with multiple testing stages.

### Communication Patterns
- Jenkins pipeline orchestration
- Multi-stage testing coordination
- Cross-service integration testing
- Asynchronous test execution
- Event-driven reporting

```mermaid
sequenceDiagram
    participant Developer as Developer
    participant Git as Git Repository
    participant Jenkins as Jenkins Server
    participant Build as Build System
    participant UnitTests as Unit Test Suite
    participant DBTests as Database Tests
    participant BDDTests as BDD Tests
    participant UITests as UI Test Server
    participant SonarQube as SonarQube
    participant ZAP as OWASP ZAP
    participant Reports as Test Reports

    Developer->>Git: git push (code commit)
    Git->>Jenkins: Post-receive hook trigger
    Jenkins->>Build: Stage 1: Build
    Build->>Build: gradlew build (multi-module)
    Build-->>Jenkins: Build artifacts

    Jenkins->>UnitTests: Stage 2: Unit Tests
    UnitTests->>UnitTests: JUnit + Mockito tests
    UnitTests->>UnitTests: Mathematics domain tests
    UnitTests->>UnitTests: Authentication tests
    UnitTests-->>Jenkins: Unit test results

    Jenkins->>DBTests: Stage 3: Database Tests
    DBTests->>DBTests: Persistence layer integration
    DBTests->>DBTests: Flyway migration tests
    DBTests-->>Jenkins: Database test results

    Jenkins->>BDDTests: Stage 4: BDD Tests
    BDDTests->>BDDTests: Cucumber (Java) features
    BDDTests->>BDDTests: Behave (Python) scenarios
    BDDTests-->>Jenkins: BDD test results

    Jenkins->>SonarQube: Stage 5: Static Analysis
    SonarQube->>SonarQube: Code quality analysis
    SonarQube->>SonarQube: Security vulnerability scan
    SonarQube-->>Jenkins: Quality gate status

    Jenkins->>UITests: Stage 6: UI Testing
    UITests->>UITests: Selenium WebDriver tests
    UITests->>ZAP: Security scanning proxy
    ZAP-->>UITests: Security report
    UITests-->>Jenkins: UI test results

    Jenkins->>Reports: Stage 7: Report Generation
    Reports->>Reports: Aggregate test results
    Reports->>Reports: Generate HTML reports
    Reports->>Reports: Dependency check reports
    Reports-->>Jenkins: Comprehensive test reports

    Jenkins->>Jenkins: Stage 8: Deployment Decision
    alt All tests passed
        Jenkins->>Jenkins: Deploy to production
        Jenkins-->>Developer: Build successful - deployed
    else Tests failed
        Jenkins-->>Developer: Build failed - review reports
    end
```

## Workflow 7: Error Handling and Recovery Patterns

### Description
Comprehensive error handling workflow showing exception propagation, database transaction recovery, and user-friendly error responses across different system layers.

### Communication Patterns
- Exception propagation through layers
- Database transaction rollback
- Graceful degradation
- User-friendly error messaging

```mermaid
sequenceDiagram
    participant User as User
    participant Servlet as Web Servlet
    participant BusinessLogic as Business Logic (Utils)
    participant PersistenceLayer as PersistenceLayer
    participant H2DB as H2 Database

    Note over User,H2DB: Normal Successful Operation
    User->>Servlet: Valid request
    Servlet->>BusinessLogic: processRequest(data)
    BusinessLogic->>PersistenceLayer: beginTransaction()
    BusinessLogic->>PersistenceLayer: executeOperation(data)
    PersistenceLayer->>H2DB: SQL operation
    H2DB-->>PersistenceLayer: Success
    BusinessLogic->>PersistenceLayer: commitTransaction()
    BusinessLogic-->>Servlet: Success result
    Servlet-->>User: Success response

    Note over User,H2DB: Database Constraint Violation
    User->>Servlet: Request with duplicate data
    Servlet->>BusinessLogic: processRequest(duplicateData)
    BusinessLogic->>PersistenceLayer: executeOperation(duplicateData)
    PersistenceLayer->>H2DB: INSERT (violates unique constraint)
    H2DB-->>PersistenceLayer: SqlRuntimeException (constraint violation)
    PersistenceLayer-->>BusinessLogic: SqlRuntimeException
    BusinessLogic->>BusinessLogic: handleConstraintViolation()
    BusinessLogic-->>Servlet: BusinessError(DUPLICATE_ENTITY)
    Servlet-->>User: User-friendly error: "Item already exists"

    Note over User,H2DB: Transaction Rollback Scenario
    User->>Servlet: Complex multi-step operation
    Servlet->>BusinessLogic: processComplexOperation()
    BusinessLogic->>PersistenceLayer: beginTransaction()
    BusinessLogic->>PersistenceLayer: step1()
    PersistenceLayer->>H2DB: SQL step1
    H2DB-->>PersistenceLayer: Success
    BusinessLogic->>PersistenceLayer: step2()
    PersistenceLayer->>H2DB: SQL step2 (fails)
    H2DB-->>PersistenceLayer: SqlRuntimeException
    PersistenceLayer-->>BusinessLogic: SqlRuntimeException
    BusinessLogic->>PersistenceLayer: rollbackTransaction()
    PersistenceLayer->>H2DB: ROLLBACK
    H2DB-->>PersistenceLayer: Rollback complete
    BusinessLogic-->>Servlet: OperationFailed with rollback
    Servlet-->>User: "Operation failed, no changes made"

    Note over User,H2DB: Validation Error (No DB Impact)
    User->>Servlet: Request with invalid data
    Servlet->>BusinessLogic: processRequest(invalidData)
    BusinessLogic->>BusinessLogic: validateInput(invalidData)
    BusinessLogic-->>Servlet: ValidationError(INVALID_INPUT)
    Servlet-->>User: "Please check your input data"

    Note over User,H2DB: System Recovery
    User->>Servlet: Request during system issue
    Servlet->>PersistenceLayer: database operation
    PersistenceLayer->>H2DB: SQL operation (connection failed)
    H2DB-->>PersistenceLayer: Connection exception
    PersistenceLayer-->>Servlet: DatabaseUnavailableException
    Servlet->>Servlet: fallbackToCachedData()
    Servlet-->>User: "System temporarily unavailable, showing cached data"
```