```mermaid
graph TB
    %% Frontend Layer
    subgraph Frontend
        HTML[HTML Pages]
        CSS[CSS Stylesheets]
        JS[JavaScript Client]
    end

    %% Web Application Layer
    subgraph WebApp["Web Application (Tomcat)"]
        subgraph Servlets
            AuthS[Authentication Servlets<br/>Login/Register]
            LibS[Library Servlets<br/>Book/Borrower/Loan]
            MathS[Mathematics Servlets<br/>Math/Fib/Ack]
            DbS[DbServlet<br/>Database Mgmt]
        end
        
        subgraph BusinessLogic
            AuthU[Authentication Utils<br/>Password Validation]
            LibU[Library Utils<br/>Business Rules]
            Calc[Calculator<br/>Arithmetic]
            Fib[Fibonacci<br/>Recursive/Iterative]
            Ack[Ackermann<br/>Recursive/Iterative]
        end
        
        subgraph Utilities
            ServletU[ServletUtils<br/>Request/Response]
            StringU[StringUtils<br/>JSON Escaping]
            CheckU[CheckUtils<br/>Validation]
        end
    end

    %% Desktop Application Layer
    subgraph DesktopApp["Desktop Application"]
        AutoUI[AutoInsuranceUI<br/>Swing Interface]
        AutoProc[AutoInsuranceProcessor<br/>Premium Calculation]
        AutoServer[AutoInsuranceScriptServer<br/>Port 8000]
    end

    %% Persistence Layer
    subgraph Persistence
        PersistLayer[PersistenceLayer<br/>Data Access]
        IPersistLayer[IPersistenceLayer<br/>Interface]
        SqlData[SqlData<br/>SQL Operations]
        ParamObj[ParameterObject<br/>Type-safe Params]
    end

    %% Database Layer
    subgraph Database
        H2[(H2 Database<br/>In-memory/File)]
        Flyway[Flyway Migrations<br/>Schema Management]
    end

    %% Testing Infrastructure
    subgraph Testing
        UITest[UI Test Server<br/>Selenium/Behave]
        ZAP[OWASP ZAP<br/>Security Scanning]
        Jenkins[Jenkins CI/CD<br/>Pipeline Orchestration]
    end

    %% Interaction Flows
    HTML --> AuthS
    HTML --> LibS
    HTML --> MathS
    JS --> LibS
    JS --> AuthS
    
    AuthS --> AuthU
    LibS --> LibU
    MathS --> Calc
    MathS --> Fib
    MathS --> Ack
    
    AuthU --> PersistLayer
    LibU --> PersistLayer
    Calc --> PersistLayer
    
    AutoUI --> AutoProc
    AutoServer --> AutoUI
    AutoProc --> PersistLayer
    
    PersistLayer --> IPersistLayer
    IPersistLayer --> SqlData
    SqlData --> ParamObj
    PersistLayer --> H2
    Flyway --> H2
    
    UITest --> HTML
    UITest --> AutoUI
    ZAP --> HTML
    Jenkins --> UITest
    Jenkins --> ZAP
```

The component boundaries follow a layered architecture with clear separation between presentation (Frontend/Desktop), business logic (WebApp services), and data access (Persistence layer). Communication patterns are primarily request-response via HTTP for web interactions and custom TCP socket protocol for desktop automation. The persistence layer provides a unified data access abstraction that all business domains depend on, while mathematics services remain stateless computational components.