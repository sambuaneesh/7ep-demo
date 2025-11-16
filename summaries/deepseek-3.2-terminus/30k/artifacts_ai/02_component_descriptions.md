| Component Name | Responsibility | Interfaces (key endpoints or methods) | Depends On | Technologies (frameworks, DBs, patterns) |
|----------------|----------------|--------------------------------------|------------|------------------------------------------|
| AutoInsuranceUI | Desktop UI for insurance premium calculations | Swing UI components, TCP socket server (port 8000) | AutoInsuranceProcessor, Swing framework | Java Swing, Custom TCP protocol |
| AutoInsuranceProcessor | Core business logic for insurance premium calculations | Premium calculation methods with age/claims parameters | Domain objects (AutoInsuranceAction) | Rule-based algorithm, Java business logic |
| AutoInsuranceScriptServer | Socket server for UI automation | TCP commands: set, get, click, quit | AutoInsuranceUI | Java sockets, Custom text protocol |
| PersistenceLayer | Main data access layer | IPersistenceLayer interface, SQL operations | H2 Database, Flyway migrations | JDBC, H2, Flyway, Micro-ORM pattern |
| Authentication Services | User registration, login, credential management | LoginServlet, RegisterServlet, LoginUtils, RegistrationUtils | PersistenceLayer, User domain | Java Servlets, SHA-256 hashing, Nbvcxz entropy |
| Library Services | Book/borrower tracking and lending system | LibraryUtils, Library*Servlets, domain objects | PersistenceLayer, Book/Borrower/Loan | Java Servlets, Domain-driven design |
| Mathematics Services | Computational algorithms and operations | MathServlet, FibServlet, AckServlet, Calculator | None (stateless) | Java Servlets, Recursive/iterative algorithms |
| Helper Components | Utility functions and application lifecycle | ServletUtils, StringUtils, CheckUtils, WebAppListener | Servlet API, Database | Utility patterns, Servlet listeners |
| Frontend Components | Web UI presentation and interaction | HTML pages, CSS styles, JavaScript functions | Backend servlets (via AJAX) | HTML5, CSS3, JavaScript, AJAX |
| Testing Infrastructure | Comprehensive test execution and reporting | Selenium WebDriver, BDD features, ZAP proxy | Application under test | Selenium, Cucumber, Behave, OWASP ZAP |
| CI/CD Infrastructure | Continuous integration and deployment | Jenkins pipeline stages, Git hooks | Git repositories, SonarQube | Jenkins, SonarQube, Gradle, H2O |
| Database Schema | Data persistence and migration management | Flyway migrations, H2 console | JDBC connection pool | H2 Database, Flyway, PostgreSQL mode |