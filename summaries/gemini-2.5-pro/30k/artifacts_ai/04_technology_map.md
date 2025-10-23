| Component Name | Language | Frameworks | Database | Communication | Patterns |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Web App: Authentication** | Java | Java Servlet API, Nbvcxz | H2 (via shared PersistenceLayer) | HTTP (POST) | Password Hashing (SHA-256), Entropy-based Validation, DAO |
| **Web App: Library Management** | Java | Java Servlet API, JSP | H2 (via shared PersistenceLayer) | HTTP (GET/POST), AJAX | Business Logic Facade (`LibraryUtils`), DAO, Autocomplete UI |
| **Web App: Mathematics** | Java | Java Servlet API | None (Stateless) | HTTP (POST) | Stateless Computational Service, Strategy (iterative vs. recursive) |
| **Web App: Persistence Layer** | Java | JDBC, FlywayDB | H2 (manages connection pool) | JDBC | Centralized DAO, Singleton, Custom Micro-ORM, Exception Wrapping |
| **Desktop App: Auto Insurance** | Java | Java Swing | None (Stateless) | TCP Socket (Custom test server on port 8000) | MVC-like, DTO, Custom Test Automation Server |
| **CI/CD Pipeline** | Groovy (Jenkinsfile), Python, Shell | Jenkins (Pipeline-as-Code), Gradle, SonarQube, OWASP ZAP, JUnit | N/A | Git Hooks, HTTP/REST (API calls), ZAP Proxy | Pipeline-as-Code, CI/CD, DAST, SAST, Mutation Testing |