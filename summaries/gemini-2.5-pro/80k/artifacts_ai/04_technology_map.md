| Component Name | Language | Frameworks | Database | Communication | Patterns |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Authentication Service** | Java | Java Servlet API, nbvcxz | H2 (via Persistence Service) | HTTP | Layered Architecture |
| **Library Service** | Java, JavaScript | Java Servlet API | H2 (via Persistence Service) | HTTP, AJAX | Layered Architecture, Client-side Data Fetching |
| **Mathematics Service** | Java | Java Servlet API | None | HTTP | Layered Architecture |
| **Persistence Service** | Java | JDBC, Flyway | H2 (in-memory, PostgreSQL mode) | Internal Java Interface, Direct JDBC | Repository/Gateway, Data Mapper, Null Object |
| **Standalone Desktop App** | Java | Java Swing | None | Custom TCP/IP Socket Protocol | Decision Table |
| **Application Lifecycle** | Java, XML | Java Servlet API (`ServletContextListener`), Flyway | H2 (Schema Initialization) | Tomcat Lifecycle Hooks | Database Migration |
| **Frontend** | JavaScript, HTML | Vanilla JS (`XMLHttpRequest`) | None | AJAX (HTTP) | Dynamic UI Rendering, Progressive Enhancement |
| **CI/CD Pipeline** | Groovy (Jenkinsfile) | Jenkins, Gradle, SonarQube, JMeter, OWASP ZAP, Cucumber | H2 (for testing) | Jenkins orchestration | CI/CD, DevOps, Testing Pyramid, BDD |