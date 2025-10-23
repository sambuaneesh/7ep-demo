| Component Name | Language | Frameworks | Database | Communication | Patterns |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Authentication & User Management** | Java | Java Servlet API, JSP, Nbvcxz | H2 | HTTP (POST) | Layered Architecture, Result Object (`RegistrationResult`) |
| **Library Management** | Java | Java Servlet API, JSP | H2 | HTTP (GET, POST) | Layered Architecture, Data Access Object (via `IPersistenceLayer`) |
| **Mathematical Calculations** | Java | Java Servlet API, `java.math.BigInteger` | None | HTTP (POST) | Stateless Service, Strategy (for algorithm selection) |
| **System Administration** | Java | Java Servlet API, Flyway | H2 (Schema Management) | HTTP (GET) | Facade (over Flyway) |
| **Desktop Application (Auto Insurance)** | Java | Java Swing | None | Custom TCP Socket Protocol (Text-based) | Thick Client, Rules Engine |
| **CI/CD & Testing Pipeline** | Groovy, Python, Java, C#, JS | Jenkins, Gradle, Selenium, Cucumber, Behave, NUnit, OWASP ZAP, SonarQube | H2 (File-based) | HTTP (API/UI interaction), TCP (Desktop App) | Behavior-Driven Development (BDD), Page Object Model (POM), DAST |