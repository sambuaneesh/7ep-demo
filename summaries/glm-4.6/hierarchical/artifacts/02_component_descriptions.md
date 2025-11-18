
| Component Name | Responsibility | Interfaces (key endpoints or methods) | Depends On | Technologies |
|----------------|--------------|----------------------------------------|------------|--------------|
| AutoInsuranceModule | Educational insurance premium calculations | AutoInsuranceProcessor.calculate(), AutoInsuranceUI components | None (self-contained) | Java Swing, Socket Programming, BDD |
| TestingLayer | UI and API validation for library system | SeleniumTests, HtmlUnitTests, ApiCalls.registerUser() | Library Core, Authentication | Selenium WebDriver, HtmlUnit, JUnit |
| LibraryCore | Core library operations (books, borrowers, loans) | LibraryUtils.lendBook(), Servlets (e.g., /lend-book) | Persistence Layer, Domain Objects | Servlets, MVC, Dependency Injection |
| CartesianProductDemo | Mathematical Cartesian product demonstration | CartesianProduct.calculate() | None (stub implementation) | Generics, BDD (Cucumber) |
| Helpers | Cross-cutting utilities (validation, security) | CheckUtils.validate(), ServletUtils.forward() | None | Java Utils, JSON Escaping |
| TomcatIntegration | Application lifecycle and database initialization | WebAppListener.contextInitialized() | Persistence Layer | ServletContextListener, Flyway |
| PersistenceLayer | Database abstraction and schema management | IPersistenceLayer.registerBook(), PersistenceLayer implementations | H2 Database | JDBC, Connection Pooling, Flyway |
| MathematicsModule | Educational mathematical algorithms | Fibonacci.calculate(), Ackermann.compute() | None | BigInteger, Functional Programming |
| Authentication | User registration and login security | RegistrationUtils.register(), LoginServlet | Persistence Layer, Auth Domain Objects | Nbvcxz (password entropy), BDD |
| ExpensesModule | Financial calculations (dinner/alcohol expenses) | AlcoholCalculator.calculate() | None | Value Objects, BDD |
| AuthDomainObjects | User and registration data models | User, PasswordResult, RegistrationResult classes | None | Immutable Objects, Enums |
| LibraryDomainObjects | Core entities (Book, Borrower, Loan) | Book, Borrower, Loan classes | None | Immutable Objects, Factory Methods |
| MathBDDTests | BDD tests for mathematical functions | AckermannStepDefs, FibonacciStepDefs | Mathematics Module | Cucumber, Gherkin |
| SelenifiedTests | Advanced UI automation tests | SelenifiedSample tests | Library Core | Selenified Framework, Flyway |