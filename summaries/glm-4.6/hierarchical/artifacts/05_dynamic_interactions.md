
## 1. Book Lending Workflow

```mermaid
sequenceDiagram
    participant User as Library User
    participant WebApp as LibraryLendServlet
    participant Utils as LibraryUtils
    participant Persist as PersistenceLayer
    participant DB as H2 Database

    User->>WebApp: POST /lend-book (title, borrower)
    WebApp->>WebApp: validateInputParameters()
    alt Invalid Input
        WebApp->>WebApp: logWarning()
        WebApp->>User: Forward to error page
    else Valid Input
        WebApp->>Utils: lendBook(title, borrower)
        Utils->>Persist: searchBorrowersByName(borrower)
        Persist->>DB: SELECT * FROM borrowers WHERE name = ?
        DB-->>Persist: Borrower data
        Persist-->>Utils: Optional<Borrower>
        
        alt Borrower Not Found
            Utils->>Utils: logBorrowerNotFound()
            Utils-->>WebApp: BORROWER_NOT_FOUND
        else Borrower Found
            Utils->>Persist: searchBooksByTitle(title)
            Persist->>DB: SELECT * FROM books WHERE title = ?
            DB-->>Persist: Book data
            Persist-->>Utils: List<Book>
            
            alt Book Not Found
                Utils->>Utils: logBookNotFound()
                Utils-->>WebApp: BOOK_NOT_FOUND
            else Book Found
                Utils->>Utils: checkBookAvailability(book)
                alt Book Already Borrowed
                    Utils->>Utils: logBookUnavailable()
                    Utils-->>WebApp: BOOK_ALREADY_BORROWED
                else Book Available
                    Utils->>Persist: saveLoan(loan)
                    Persist->>DB: INSERT INTO loans VALUES (?, ?, ?, ?)
                    DB-->>Persist: Success
                    Persist-->>Utils: SUCCESS
                    Utils-->>WebApp: SUCCESS
                end
            end
        end
    end
    
    WebApp->>WebApp: setResultAttribute()
    WebApp->>User: Forward to result page
```

**Description**: The core library circulation workflow enabling registered borrowers to check out available books. Triggered by web form submission or API call for lending operations.

**Communication Patterns**:
- **Synchronous HTTP**: Servlet handles POST requests
- **Database Transactions**: ACID operations for loan creation
- **Layered Architecture**: Clean separation between web, service, and persistence layers
- **Error Handling**: Comprehensive validation at each layer with specific result codes

---

## 2. User Registration with Password Security

```mermaid
sequenceDiagram
    participant User as New Borrower
    participant WebApp as RegisterServlet
    participant Utils as RegistrationUtils
    participant SecLib as Nbvcxz (Security Lib)
    participant Persist as PersistenceLayer
    participant DB as H2 Database

    User->>WebApp: POST /register (username, password)
    WebApp->>WebApp: extractCredentials()
    alt Missing Credentials
        WebApp->>User: Forward with error message
    else Valid Input
        WebApp->>Utils: registerUser(username, password)
        Utils->>SecLib: estimate(password)
        SecLib-->>Utils: PasswordResult(entropy, crackTime)
        
        Weak Password Check
        alt INSUFFICIENT_ENTROPY
            Utils-->>WebApp: BAD_PASSWORD
        else Strong Password
            Utils->>Persist: searchUsersByUsername(username)
            Persist->>DB: SELECT * FROM users WHERE username = ?
            DB-->>Persist: User data
            Persist-->>Utils: Optional<User>
            
            Duplicate Check
            alt User Exists
                Utils-->>WebApp: ALREADY_REGISTERED
            else User Not Exists
                Utils->>Persist: saveUser(user)
                Persist->>DB: INSERT INTO users VALUES (?, ?, ?)
                DB-->>Persist: Success
                Persist-->>Utils: SUCCESS
                Utils-->>WebApp: SUCCESSFULLY_REGISTERED
            end
        end
    end
    
    WebApp->>User: Forward to appropriate result page
```

**Description**: Secure borrower registration workflow with entropy-based password validation. Ensures strong credentials while preventing duplicate accounts.

**Communication Patterns**:
- **Synchronous Validation**: Real-time password strength analysis
- **Security Integration**: Nbvcxz library for entropy calculations
- **Database Constraints**: Unique username enforcement
- **Event Logging**: Comprehensive audit trail for security

---

## 3. Multi-Algorithm Mathematics Computation

```mermaid
sequenceDiagram
    participant User as Student
    participant WebApp as FibServlet
    participant Processor as FibonacciIterative
    participant TailUtil as TailRecursive
    participant WebApp2 as AckServlet
    participant AckProc as AckermannIterative

    User->>WebApp: POST /fib (n, algo)
    WebApp->>WebApp: parseParameters()
    
    alt Algorithm Selection
    opt Fast Doubling (fibAlgo1)
        WebApp->>Processor: fibAlgo1(n)
        Processor->>Processor: matrixExponentiation()
        Processor-->>WebApp: BigInteger result
    else Iterative (fibAlgo2)
        WebApp->>Processor: fibAlgo2(n)
        Processor->>Processor: loopIteration()
        Processor-->>WebApp: BigInteger result
    end
    
    WebApp->>User: Forward with result

    Note over User, AckProc: Later: Ackermann Calculation
    
    User->>WebApp2: POST /ack (m, n, type)
    alt Tail Recursive (stack-safe)
        WebApp2->>AckProc: calculateIterative(m, n)
        AckProc->>TailUtil: tailie(init, iter, pred, final)
        TailUtil->>TailUtil: streamIterations()
        TailUtil-->>AckProc: BigInteger result
    else Classic Recursive
        WebApp2->>AckProc: calculateRecursive(m, n)
        AckProc->>AckProc: recursiveCalls()
        AckProc-->>WebApp2: BigInteger result
    end
    
    WebApp2->>User: Forward with result
```

**Description**: Educational module demonstrating algorithm complexity comparisons through web-accessible mathematical computations with multiple implementation strategies.

**Communication Patterns**:
- **Algorithm Selection**: User chooses between recursive/iterative implementations
- **Stack Safety**: Tail recursion optimization for deep recursions
- **Big Integer Handling**: Arbitrary precision arithmetic
- **Performance Demonstration**: Visualizing complexity differences

---

## 4. Auto Insurance Risk Assessment

```mermaid
sequenceDiagram
    participant User as Insurance Agent
    participant UI as AutoInsuranceUI
    participant Proc as AutoInsuranceProcessor
    participant Client as AutoInsuranceScriptClient
    participant Tester as DesktopTester

    User->>UI: Enter age and claims history
    UI->>Proc: processClaims(age, claims)
    
    alt Invalid Claims Data
        Proc-->>UI: throw InvalidClaimsException
        UI->>User: Display error message
    else Valid Data
        Proc->>Proc: applyBusinessRules()
        
        par Age-Based Assessment
            proc->>proc: evaluateAgeRisk(age)
        and Claims History
            proc->>proc: calculateClaimsImpact(claims)
        end
        
        Proc->>Proc: determineWarningLetter()
        Proc->>Proc: calculatePolicyAction()
        Proc-->>UI: AutoInsuranceAction
        UI->>User: Display premium increase and warnings
    end

    Note over User, Tester: Automated Testing Scenario
    
    Tester->>Client: connect()
    Client->>Client: establishSocket()
    Tester->>Client: setAge(22)
    Tester->>Client: setClaims(1)
    Tester->>Client: calculate()
    Client-->>UI: Send calculation command
    UI->>Proc: processClaims(22, 1)
    Proc-->>UI: Action object
    UI-->>Client: Return result
    Client-->>Tester: getLabel()
    Tester->>Tester: assertEquals("$100 increase, LTR1")
```

**Description**: Insurance premium calculation system demonstrating business rule processing with both GUI and automated testing interfaces.

**Communication Patterns**:
- **Socket Communication**: Remote scripting capability for automation
- **Business Rule Engine**: Complex conditional logic processing
- **Error Boundaries**: Custom exceptions for invalid inputs
- **Testing Abstraction**: DesktopTester provides clean API for automation

---

## 5. Database Initialization and Recovery

```mermaid
sequenceDiagram
    participant Container as Tomcat
    participant Listener as WebAppListener
    participant Persist as PersistenceLayer
    participant Flyway as Flyway Migrator
    participant DB as H2 Database
    participant Admin as DbServlet

    Container->>Listener: contextInitialized()
    Listener->>Persist: cleanAndMigrateDatabase()
    Persist->>Persist: cleanDatabase()
    Persist->>DB: DROP ALL OBJECTS
    DB-->>Persist: Success
    Persist->>Flyway: migrate()
    Flyway->>DB: Execute migration scripts
    DB-->>Flyway: Schema created
    Flyway-->>Persist: Migration complete
    Persist-->>Listener: Ready
    Listener->>Container: Application started

    Note over Container, Admin: Administrative Reset
    
    Admin->>Admin: GET /db?action=reset
    Admin->>Persist: cleanAndMigrateDatabase()
    Persist->>DB: Reset database
    Persist-->>Admin: Success message
    Admin->>Admin: Forward to result page
```

**Description**: Critical infrastructure workflow ensuring proper database state during application startup and administrative resets. Essential for maintaining consistency across deployments.

**Communication Patterns**:
- **Lifecycle Hooks**: ServletContextListener integration
- **Database Migrations**: Flyway version-controlled schema management
- **Administrative Interface**: Web-based database control
- **Idempotent Operations**: Safe repeated execution

---

## 6. Behavior-Driven Test Execution

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant Cuke as Cucumber Runner
    participant StepDef as RegistrationStepDefs
    participant Utils as RegistrationUtils
    participant Persist as PersistenceLayer

    Dev->>Cuke: Execute feature file
    Cuke->>StepDef: Given a database exists
    StepDef->>Persist: cleanDatabase()
    StepDef->>Cuke: Scenario ready

    Cuke->>StepDef: When user registers with strong password
    StepDef->>Utils: registerUser(username, password)
    Utils->>Persist: saveUser()
    Persist-->>Utils: SUCCESS
    Utils-->>StepDef: RegistrationResult
    StepDef->>StepDef: storeResult()

    Cuke->>StepDef: Then registration succeeds
    StepDef->>StepDef: assertSuccess()
    
    alt Test Failure
        StepDef-->>Cuke: Assertion failure
        Cuke->>Dev: Test report with error
    else Test Success
        StepDef-->>Cuke: Scenario passed
        Cuke->>Dev: All scenarios passed
    end
```

**Description**: Automated testing workflow demonstrating Behavior-Driven Development practices with living documentation through executable specifications.

**Communication Patterns**:
- **Gherkin Execution**: Natural language test scenarios
- **Glue Code**: Step definitions bridging language and implementation
- **Test Isolation**: Database cleanup between scenarios
- **Assertion Verification**: Comprehensive validation of outcomes

---

## Error Handling and Recovery Patterns

```mermaid
sequenceDiagram
    participant Client as API Client
    participant Service as LibraryUtils
    participant Persist as PersistenceLayer
    participant Logger as Logging System
    participant ErrorHandler as Error Handler

    Client->>Service: lendBook(title, borrower)
    
    alt Input Validation Error
        Service->>ErrorHandler: CheckUtils.validate()
        ErrorHandler->>Service: throw AssertionException
        Service->>Logger: logValidationFailure()
        Service-->>Client: INVALID_INPUT_RESULT
    else Business Rule Violation
        Service->>Service: checkBusinessRules()
        Service->>Logger: logBusinessViolation()
        Service-->>Client: BOOK_ALREADY_BORROWED
    else Database Error
        Service->>Persist: saveLoan()
        Persist->>Persist: executeSql()
        Persist->>ErrorHandler: catch SQLException
        ErrorHandler->>Persist: throw SqlRuntimeException
        Persist->>Logger: logDatabaseError()
        Persist-->>Service: SqlRuntimeException
        Service->>ErrorHandler: handlePersistenceError()
        Service-->>Client: DATABASE_ERROR_RESULT
    else Success
        Service->>Persist: saveLoan()
        Persist-->>Service: Success
        Service->>Logger: logSuccessfulOperation()
        Service-->>Client: SUCCESS
    end
```

**Description**: Cross-cutting error handling strategy ensuring graceful degradation and proper recovery throughout the system.

**Communication Patterns**:
- **Exception Translation**: Converting low-level exceptions to domain-specific
- **Defensive Programming**: Input validation at boundaries
- **Comprehensive Logging**: Audit trails for all operations
- **Graceful Degradation**: Meaningful error responses to clients