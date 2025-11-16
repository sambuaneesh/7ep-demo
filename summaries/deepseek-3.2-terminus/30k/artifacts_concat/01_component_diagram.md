```mermaid
graph TB
    %% Client Layer
    WebBrowser["Web Browser"]
    DesktopUI["Desktop UI<br/>AutoInsuranceUI"]
    
    %% Presentation Layer
    AuthServlets["Authentication Servlets<br/>LoginServlet, RegisterServlet"]
    LibraryServlets["Library Servlets<br/>Book/Borrower/Loan Management"]
    MathServlets["Mathematics Servlets<br/>MathServlet, FibServlet, AckServlet"]
    DbServlet["Database Servlet<br/>DbServlet"]
    
    %% Business Logic Layer
    AuthUtils["Authentication Utils<br/>LoginUtils, RegistrationUtils"]
    LibraryUtils["Library Utils<br/>LibraryUtils"]
    MathUtils["Mathematics Utils<br/>Calculator, Fibonacci, Ackermann"]
    InsuranceProcessor["Insurance Processor<br/>AutoInsuranceProcessor"]
    
    %% Persistence Layer
    PersistenceLayer["Persistence Layer<br/>IPersistenceLayer"]
    
    %% Infrastructure
    Database["H2 Database"]
    SocketServer["Socket Server<br/>AutoInsuranceScriptServer"]
    CI["CI/CD Infrastructure<br/>Jenkins, SonarQube"]
    TestInfra["Testing Infrastructure<br/>Selenium, JUnit, Mockito"]
    
    %% Interactions
    WebBrowser --> AuthServlets
    WebBrowser --> LibraryServlets
    WebBrowser --> MathServlets
    WebBrowser --> DbServlet
    
    DesktopUI --> InsuranceProcessor
    DesktopUI --> SocketServer
    
    AuthServlets --> AuthUtils
    LibraryServlets --> LibraryUtils
    MathServlets --> MathUtils
    
    AuthUtils --> PersistenceLayer
    LibraryUtils --> PersistenceLayer
    InsuranceProcessor --> PersistenceLayer
    
    PersistenceLayer --> Database
    
    SocketServer -.-> DesktopUI
    CI -.-> TestInfra
    TestInfra -.-> AuthServlets
    TestInfra -.-> LibraryServlets
    TestInfra -.-> MathServlets
    TestInfra -.-> DesktopUI

    %% Styling
    classDef client fill:#e1f5fe
    classDef presentation fill:#f3e5f5
    classDef business fill:#e8f5e8
    classDef persistence fill:#fff3e0
    classDef infrastructure fill:#fce4ec
    
    class WebBrowser,DesktopUI client
    class AuthServlets,LibraryServlets,MathServlets,DbServlet presentation
    class AuthUtils,LibraryUtils,MathUtils,InsuranceProcessor business
    class PersistenceLayer,Database persistence
    class SocketServer,CI,TestInfra infrastructure
```

The component boundaries follow a layered architecture with clear domain separation: presentation (servlets), business logic (utils), and persistence layers. Communication patterns are primarily request-response via HTTP for web components and direct method calls for internal layers, with the persistence layer serving as a centralized data access abstraction. The desktop application operates independently with its own UI and business logic layers, connected via a custom socket protocol for automation.