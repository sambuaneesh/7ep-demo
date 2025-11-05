```mermaid
graph TB
    subgraph "External Actors"
        User[User / API Client]
    end

    subgraph "Presentation Layer (Spring MVC)"
        direction LR
        subgraph "Owner & Pet Management"
            OwnerController[OwnerController]
            PetController[PetController]
            VisitController[VisitController]
        end
        subgraph "Veterinarian Management"
            VetController[VetController]
        end
        subgraph "System & Web"
             SystemControllers[System Controllers <br> Welcome, Crash]
        end
    end

    subgraph "Service & Data Access Layer (Spring Data JPA)"
        direction LR
        subgraph "Owner & Pet Persistence"
            OwnerRepository[OwnerRepository]
            PetTypeRepository[PetTypeRepository]
        end
        subgraph "Veterinarian Persistence"
            VetRepository[VetRepository <br> @Cacheable]
        end
    end

    subgraph "Domain Model"
        direction LR
        OwnerModel[Owner, Pet, Visit]
        VetModel[Vet, Specialty]
        BaseModel[BaseEntity, Person]
    end

    subgraph "Cross-Cutting Concerns (System Package)"
        direction LR
        I18n[Internationalization]
        ErrHandling[Global Error Handling]
        CachingConfig[Cache Configuration]
    end
    
    subgraph "Persistence"
        DB[(Database)]
    end

    %% Main Flow
    User --> OwnerController
    User --> VetController
    User --> SystemControllers

    OwnerController --> OwnerRepository
    PetController --> OwnerRepository & PetTypeRepository
    VisitController --> OwnerRepository
    VetController --> VetRepository

    OwnerRepository --> DB
    PetTypeRepository --> DB
    VetRepository --> DB

    %% Layer Dependencies
    OwnerController & PetController & VisitController -- uses --> OwnerModel
    VetController -- uses --> VetModel
    OwnerRepository & PetTypeRepository -- manages --> OwnerModel
    VetRepository -- manages --> VetModel
    OwnerModel & VetModel -- extends --> BaseModel
    
    %% Cross-Cutting Application
    I18n & ErrHandling -- applies to --> "Presentation Layer (Spring MVC)"
    CachingConfig -.-> VetRepository
```

The system exhibits a classic layered architecture, separating the Presentation (Spring MVC Controllers), Data Access (Spring Data JPA Repositories), and a central Domain Model. Communication is streamlined with controllers directly invoking repositories to manage persistence for distinct domain components like Owner/Pet and Vet. Cross-cutting concerns such as caching, error handling, and internationalization are managed in a dedicated system component, applying their services across the appropriate layers.