```markdown
| Component Name | Language | Frameworks | Database | Communication | Patterns |
|---------------|----------|------------|----------|---------------|----------|
| AutoInsuranceUI | Java | Swing | None | Custom TCP socket protocol (port 8000) | MVC, Server Pattern |
| AutoInsuranceProcessor | Java | None | None | In-process method calls | Business Logic Layer, Rules Engine |
| AutoInsuranceScriptServer | Java | Java Sockets | None | TCP socket commands | Server Pattern, Command Pattern |
| Authentication Service | Java | Java Servlets, JSP | H2 (via PersistenceLayer) | HTTP/REST endpoints | MVC, DAO, Repository Pattern |
| Library Management Service | Java | Java Servlets, JSP | H2 (via PersistenceLayer) | HTTP/REST endpoints | MVC, DAO, Repository Pattern |
| Mathematics Service | Java | Java Servlets, JSP | None | HTTP/REST endpoints | Stateless Service, Strategy Pattern |
| PersistenceLayer | Java | JDBC, Flyway, H2 | H2 Database | JDBC, direct method calls | DAO, Repository, Micro-ORM, Template Method |
| WebAppListener | Java | Java Servlets, Flyway | H2 Database | Servlet context events | Listener Pattern, Initialization Pattern |
| Frontend UI | HTML/CSS/JavaScript | None | None | AJAX/REST calls | Responsive Design, Client-Server |
| CI/CD Infrastructure (Jenkins) | Groovy | Jenkins Pipeline | None | Git hooks, HTTP APIs | Pipeline Pattern, Quality Gates |
| Testing Framework | Java/Python/JavaScript/C# | JUnit, Mockito, Selenium, Cucumber, Behave, Mocha, NUnit | H2 (test database) | HTTP, direct calls | Page Object Model, BDD, Parameterized Tests |
| Quality Analysis (SonarQube) | Java | SonarQube Scanner | None | HTTP APIs | Static Analysis, Quality Gates |
| Security Scanning (OWASP ZAP) | Java | OWASP ZAP | None | HTTP proxy (port 9888) | Security Proxy Pattern |
| Performance Testing | Java | JMeter | None | HTTP load testing | Load Testing Pattern |
| Mutation Testing | Java | PITest | None | Bytecode instrumentation | Mutation Testing Pattern |
```