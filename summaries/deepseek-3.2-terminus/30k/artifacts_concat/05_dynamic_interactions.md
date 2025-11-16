```markdown
# Dynamic Interaction Flows and Sequence Diagrams

## 1. User Registration Workflow

### Description
Complete user registration process including password validation, entropy analysis, and database persistence. Triggered when new user submits registration form.

**Communication Patterns**: REST API calls, database transactions, synchronous processing

```mermaid
sequenceDiagram
    participant U as User/Browser
    participant RS as RegisterServlet
    participant RU as RegistrationUtils
    participant PL as PersistenceLayer
    participant DB as H2 Database

    U->>RS: POST /demo/register (username, password)
    RS->>RU: registerUser(username, password)
    RU->>RU: validatePasswordComplexity()
    RU->>RU: calculatePasswordEntropy()
    
    alt Password too weak
        RU-->>RS: RegistrationResult(FAILED, "Weak password")
        RS-->>U: Error response (HTTP 400)
    else Password valid
        RU->>PL: getUserByName(username)
        PL->>DB: SELECT * FROM auth.USER WHERE name=?
        
        alt User already exists
            DB-->>PL: User record
            PL-->>RU: Existing user
            RU-->>RS: RegistrationResult(FAILED, "User exists")
            RS-->>U: Error response (HTTP 409)
        else New user
            RU->>RU: hashPassword(password)
            RU->>PL: createUser(username, password_hash)
            PL->>DB: INSERT INTO auth.USER (name, password_hash)
            DB-->>PL: Success with user ID
            PL-->>RU: User object
            RU-->>RS: RegistrationResult(SUCCESS)
            RS-->>U: Success response (HTTP 200)
        end
    end
```

## 2. Book Lending Workflow

### Description
Complete book lending process including availability checks, borrower validation, and loan tracking. Triggered when librarian lends book to borrower.

**Communication Patterns**: REST API calls, database transactions, business rule validation

```mermaid
sequenceDiagram
    participant U as User/Browser
    participant LS as LibraryLendServlet
    participant LU as LibraryUtils
    participant PL as PersistenceLayer
    participant DB as H2 Database

    U->>LS: POST /demo/lend (bookId, borrowerId)
    LS->>LU: lendBook(bookId, borrowerId)
    
    LU->>PL: getBookById(bookId)
    PL->>DB: SELECT * FROM library.BOOK WHERE id=?
    DB-->>PL: Book record
    PL-->>LU: Book object
    
    LU->>PL: getBorrowerById(borrowerId)
    PL->>DB: SELECT * FROM library.BORROWER WHERE id=?
    DB-->>PL: Borrower record
    PL-->>LU: Borrower object
    
    LU->>PL: isBookAvailable(bookId)
    PL->>DB: SELECT COUNT(*) FROM library.LOAN WHERE book=? AND return_date IS NULL
    DB-->>PL: Loan count
    PL-->>LU: Availability status
    
    alt Book not available
        LU-->>LS: LibraryActionResults(ERROR, "Book unavailable")
        LS-->>U: Error response (HTTP 400)
    else Book available
        LU->>PL: createLoan(bookId, borrowerId, currentDate)
        PL->>DB: INSERT INTO library.LOAN (book, borrower, borrow_date)
        DB-->>PL: Success with loan ID
        PL-->>LU: Loan object
        LU-->>LS: LibraryActionResults(SUCCESS)
        LS-->>U: Success response (HTTP 200)
    end
```

## 3. Mathematical Computation Workflow

### Description
Complex mathematical computation (Fibonacci/Ackermann) with algorithm selection and performance optimization. Triggered when user requests mathematical calculation.

**Communication Patterns**: REST API calls, stateless computation, algorithm selection

```mermaid
sequenceDiagram
    participant U as User/Browser
    participant MS as MathServlet/FibServlet/AckServlet
    participant CALC as Calculator/Fibonacci/Ackermann
    participant PL as PersistenceLayer (optional)

    U->>MS: POST /demo/fibonacci (n, algorithm)
    MS->>CALC: calculate(n, algorithm)
    
    alt Recursive Fibonacci
        CALC->>CALC: fibRecursive(n)
        CALC->>CALC: Multiple recursive calls
    else Iterative Fibonacci
        CALC->>CALC: fibIterative(n)
        CALC->>CALC: Loop with O(n) complexity
    else Optimized Fibonacci
        CALC->>CALC: fibOptimized(n)
        CALC->>CALC: Matrix exponentiation O(log n)
    end
    
    CALC-->>MS: Calculation result
    MS-->>U: JSON response with result
```

## 4. Auto Insurance Premium Calculation Workflow

### Description
Desktop-based insurance premium calculation with socket automation interface. Triggered via Swing UI or automated socket commands.

**Communication Patterns**: Socket communication, desktop UI events, business rule processing

```mermaid
sequenceDiagram
    participant C as Client/Automation
    participant SS as AutoInsuranceScriptServer
    participant UI as AutoInsuranceUI
    participant AP as AutoInsuranceProcessor
    participant DM as Domain Models

    C->>SS: TCP socket connect (port 8000)
    SS->>SS: Start command listener
    
    loop Automation commands
        C->>SS: "set age 25"
        SS->>UI: setAgeField(25)
        UI->>DM: update age value
        
        C->>SS: "set claims 2" 
        SS->>UI: setClaimsField(2)
        UI->>DM: update claims value
        
        C->>SS: "click calculate"
        SS->>UI: clickCalculateButton()
        UI->>AP: calculatePremium(age, claims)
        
        AP->>AP: applyInsuranceRules()
        Note over AP: Rules:<br/>0 claims: +$25-50<br/>1 claim: +$50-100<br/>2-4 claims: +$200-400<br/>5+ claims: Cancel policy
        
        AP-->>UI: AutoInsuranceAction(result)
        UI->>UI: updateResultDisplay()
        
        C->>SS: "get label"
        SS->>UI: getResultLabel()
        UI-->>SS: premium result text
        SS-->>C: socket response
    end
    
    C->>SS: "quit"
    SS->>SS: close connection
```

## 5. Database Migration Workflow

### Description
Automated database schema migration process using Flyway, triggered on application startup or via management endpoint.

**Communication Patterns**: Event-driven startup, database transactions, version management

```mermaid
sequenceDiagram
    participant TC as Tomcat Container
    participant WL as WebAppListener
    participant FL as Flyway
    participant PL as PersistenceLayer
    participant DB as H2 Database

    TC->>WL: contextInitialized()
    WL->>PL: initializeDatabase()
    PL->>FL: migrate()
    
    FL->>DB: SELECT * FROM ADMINISTRATIVE.flyway_schema_history
    DB-->>FL: Migration history
    
    loop For each pending migration
        FL->>DB: Execute migration SQL (V1, V2, etc.)
        DB-->>FL: Migration success
        FL->>DB: INSERT INTO flyway_schema_history
        DB-->>FL: History updated
    end
    
    FL-->>PL: Migration complete
    PL-->>WL: Database ready
    WL-->>TC: Context initialized successfully
```

## 6. CI/CD Pipeline Execution Workflow

### Description
Complete continuous integration and deployment pipeline with quality gates, testing stages, and security scanning.

**Communication Patterns**: Event-driven pipeline, REST API calls, asynchronous processing

```mermaid
sequenceDiagram
    participant G as Git Repository
    participant J as Jenkins Server
    participant S as SonarQube
    participant T as Test Servers
    participant A as Application Server

    G->>J: Post-receive hook (code push)
    J->>J: Start pipeline - Build stage
    
    par Unit Tests
        J->>T: Execute JUnit tests
        T-->>J: Test results & coverage
    and Static Analysis
        J->>S: SonarQube scan
        S->>S: Code quality analysis
        S-->>J: Quality gate status
    end
    
    J->>J: BDD Tests stage
    J->>T: Execute Cucumber tests
    T->>A: Test against deployed app
    A-->>T: Test responses
    T-->>J: BDD test results
    
    J->>J: Security Analysis stage
    J->>T: OWASP ZAP scan
    T->>A: Security testing
    A-->>T: Security responses
    T-->>J: Security report
    
    J->>J: Performance Tests stage
    J->>T: Execute JMeter tests
    T->>A: Load testing
    A-->>T: Performance metrics
    T-->>J: Performance report
    
    alt All quality gates passed
        J->>A: Deploy to production
        A-->>J: Deployment success
        J->>J: Pipeline success
    else Quality gate failed
        J->>J: Pipeline failed
        J->>J: Notify developers
    end
```

## 7. Cross-Browser UI Testing Workflow

### Description
Multi-language UI testing framework executing Selenium tests across different technology stacks with centralized reporting.

**Communication Patterns**: Selenium WebDriver, REST API testing, asynchronous test execution

```mermaid
sequenceDiagram
    participant TF as Test Framework (Java/Python/JS/C#)
    participant SD as Selenium WebDriver
    participant B as Browser (Chrome)
    participant A as Application Server
    participant H as H2O Report Server

    TF->>SD: Initialize WebDriver
    SD->>B: Launch Chrome browser
    
    TF->>SD: Navigate to application
    SD->>B: Load /demo/login
    B->>A: HTTP GET /demo/login
    A-->>B: Login page HTML
    B-->>SD: Page loaded
    SD-->>TF: Page ready
    
    TF->>SD: Fill registration form
    SD->>B: Enter username/password
    B->>A: POST /demo/register
    A-->>B: Registration response
    B-->>SD: Form submitted
    SD-->>TF: Registration complete
    
    TF->>SD: Execute test scenarios
    SD->>B: Interact with UI elements
    B->>A: Various API calls
    A-->>B: Application responses
    
    TF->>SD: Capture screenshots/evidence
    SD->>B: Take screenshot
    B-->>SD: Screenshot data
    SD-->>TF: Test evidence
    
    TF->>H: Upload test reports
    H-->>TF: Report storage confirmation
    
    TF->>SD: Close browser
    SD->>B: Terminate session
```

## 8. Error Handling and Recovery Patterns

### Description
Comprehensive error handling across service boundaries including database failures, validation errors, and network issues.

**Communication Patterns**: Exception propagation, transaction rollback, graceful degradation

```mermaid
sequenceDiagram
    participant U as User
    participant S as Servlet
    participant UTL as Business Utils
    participant PL as PersistenceLayer
    participant DB as Database

    U->>S: HTTP Request
    S->>UTL: Process business logic
    
    alt Database Connection Failure
        UTL->>PL: Database operation
        PL->>DB: SQL query
        DB-->>PL: Connection timeout
        PL-->>UTL: SqlRuntimeException
        UTL-->>S: ServiceException
        S->>S: Log error with context
        S-->>U: HTTP 503 - Service Unavailable
    else Business Rule Violation
        UTL->>UTL: Validate business rules
        UTL-->>S: ValidationException
        S->>S: Log warning
        S-->>U: HTTP 400 - Bad Request
    else Network Timeout in CI/CD
        Note over S,DB: External service timeout
        S->>S: Implement circuit breaker
        S-->>U: HTTP 504 - Gateway Timeout
    else Successful with Retry
        UTL->>PL: Operation with retry logic
        PL->>DB: SQL with transaction
        DB-->>PL: Success after retry
        PL-->>UTL: Success result
        UTL-->>S: Normal response
        S-->>U: HTTP 200 - Success
    end
```
```