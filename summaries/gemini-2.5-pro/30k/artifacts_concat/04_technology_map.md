| Component Name | Language | Frameworks | Database | Communication | Patterns |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Web Application (Monolith)** | Java 8+ | Java Servlets, JSP, Gretty, Gradle | H2 (via Persistence Layer) | HTTP/HTTPS | Monolith, Layered Architecture, N-Tier |
| **Authentication Domain** | Java | Java Servlets, Nbvcxz (password strength) | H2 (shared, `auth` schema) | HTTP, In-process method calls | Bounded Context (Microservice Candidate) |
| **Library Domain** | Java | Java Servlets | H2 (shared, `library` schema) | HTTP, In-process method calls | Bounded Context (Microservice Candidate) |
| **Mathematics & Expense Domain** | Java | Java Servlets | None (Stateless) | HTTP, In-process method calls | Utility Service, Shared Library |
| **Persistence Layer** | Java | JDBC, Flyway, SLF4J, Log4j2 | H2 (in-memory/file-based, PostgreSQL mode) | JDBC | Data Access Object (DAO), Data Mapper, Template Method |
| **Web Frontend** | HTML, CSS, JavaScript | JSP | None | HTTP (AJAX/XHR, Form Posts) | Client-side rendering enhancements |
| **AutoInsurance Desktop App** | Java | Java Swing | None | Custom raw TCP socket server (port 8000) | Model-View (MVC-like), Test Automation Server |
| **CI/CD & Testing Infrastructure** | Groovy, Python, Shell, Java, C# | Jenkins, Gradle, SonarQube, OWASP ZAP/DependencyCheck, Tomcat, Selenium, JUnit, Mockito, Pytest, Behave, JMeter | H2 (for application under test) | HTTP (APIs, Webhooks), Proxied HTTP (via ZAP) | Pipeline-as-Code, Test Pyramid, BDD, Security Proxy, Page Object Model (POM) |