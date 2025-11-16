```markdown
| Component Name | Language | Frameworks | Database | Communication | Patterns |
|---------------|----------|------------|----------|---------------|----------|
| Auto Insurance Desktop Application | Java | Swing, Custom Business Logic | None (via Persistence Layer) | Custom TCP Socket Protocol (port 8000) | MVC, Rule-based Calculation, Socket Server |
| Authentication Domain | Java | Java Servlets, JSP | H2 (AUTH schema) | HTTP/REST | Servlet-based MVC, Repository, Password Hashing |
| Library Management Domain | Java | Java Servlets, JSP | H2 (LIBRARY schema) | HTTP/REST | Servlet-based MVC, Repository, Domain-Driven Design |
| Mathematics Domain | Java | Java Servlets | None (stateless) | HTTP/REST | Strategy (multiple algorithm implementations), Functional Programming |
| Persistence Layer | Java | JDBC, Flyway | H2 (multiple schemas) | JDBC | Micro-ORM, Repository, Template Method, Null Object |
| Frontend Components | HTML/CSS/JavaScript | Vanilla JS, Custom CSS | None | HTTP/AJAX | Client-Side MVC, Responsive Design |
| Web Application Core | Java | Java Servlets, Tomcat 9, Log4j2 | H2 (via Persistence Layer) | HTTP/REST | Layered Architecture, Front Controller, Immutability |
| UI Test Server (uitestbox) | Python, Java, JavaScript, C# | Selenium WebDriver, Behave (Python), JUnit, Mocha, NUnit | H2 (test database) | HTTP, WebDriver Protocol | Page Object Model, BDD, Cross-Browser Testing |
| CI/CD Infrastructure (jenkinsbox) | Groovy, Java | Jenkins, Gradle, SonarQube, Gretty | H2 (application database) | HTTP, Git, SSH | Pipeline, Continuous Integration, Quality Gates |
| Testing Infrastructure | Multiple | JUnit, Mockito, Cucumber, JMeter, PITest | H2 (test instances) | Multiple protocols | Test Pyramid, Parameterized Tests, Mutation Testing |
| Security Scanning | Java, Python | OWASP ZAP, DependencyCheck | None | HTTP Proxy | Security Testing, Vulnerability Analysis |
```