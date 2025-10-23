| Component Name | Language | Frameworks | Database | Communication | Patterns |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Authentication Service** | Java | Java Servlets, JSP, nbvcxz (password strength) | H2 (via shared Persistence Layer) | HTTP (POST), Direct method calls | Layered Architecture, Repository/DAO, Result Objects |
| **Library Service** | Java | Java Servlets, JSP | H2 (via shared Persistence Layer) | HTTP (GET/POST), Direct method calls | Layered Architecture, Repository/DAO, Result Objects (Enums) |
| **Mathematics Service** | Java | Java Servlets | None (Stateless) | HTTP (POST) | Layered Architecture, Strategy (for algorithm selection) |
| **Desktop Insurance Calculator** | Java | Java Swing | None (Stateless) | Custom TCP Socket Protocol | Thick Client, Rules Engine |
| **Persistence Layer (Shared)** | Java | JDBC, Flyway | H2 (in-memory and file-based) | Direct method calls (via `IPersistenceLayer` interface) | Repository/DAO, Dependency Injection, Database Migration |
| **CI/CD & Quality Assurance Pipeline** | Groovy, Python, Java, C#, JavaScript | Jenkins, Gradle, SonarQube, Selenium, Behave, Pytest, JMeter, Pitest, OWASP ZAP, OWASP DependencyCheck | H2 (for integration tests) | Git Hooks, HTTP (for E2E tests), ZAP Proxy | Pipeline as Code, Quality Gates, Behavior-Driven Development (BDD), Page Object Model (POM) |