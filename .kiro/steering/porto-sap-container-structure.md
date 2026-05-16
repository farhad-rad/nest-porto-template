---
inclusion: auto
---

# Porto SAP Container Structure

## Standard Container Layout

Every Container MUST follow this structure for consistency. Components may vary but the layout is uniform.

```
ContainerName/
├── Actions/                    # Use cases (one run() per file)
├── Tasks/                      # Reusable business logic units
├── Models/                     # Data representations
├── Value-Objects/              # Simple value entities
├── Events-Publisher/           # Broadcast events
├── Events-Subscriber/         # Listen to events
├── Cron-Jobs/                  # Scheduled tasks
├── Exceptions/                 # Container-specific errors
├── Policies/                   # Authorization rules
├── Contracts/                  # Interfaces
├── Configs/                    # Container configuration
├── Mails/
│   ├── Templates/
│   └── Attachments/
├── Data/
│   ├── Migrations/             # DB schema changes
│   ├── Seeders/                # Initial/test data
│   ├── Factories/              # Test data generators
│   ├── Criteria/               # Complex queries
│   ├── Repositories/           # Data access abstraction
│   ├── Validators/             # Data validation
│   ├── Transporters/           # DTOs
│   └── Rules/                  # Business rules
├── Tests/
│   ├── Unit/                   # Unit tests for Tasks
│   ├── Integration/
│   ├── Performance/
│   └── Security/
└── UI/
    ├── API/
    │   ├── Routes/             # API endpoints (one per file)
    │   ├── Controllers/        # API controllers
    │   ├── Requests/           # Validation/auth rules
    │   ├── Transformers/       # JSON response formatting
    │   └── Tests/
    │       └── Functional/
    ├── WEB/
    │   ├── Routes/             # Web endpoints
    │   ├── Controllers/        # Web controllers
    │   ├── Requests/           # Validation/auth rules
    │   ├── Views/              # HTML templates
    │   └── Tests/
    │       └── Acceptance/
    └── CLI/
        ├── Routes/             # CLI commands
        ├── Commands/           # CLI handlers
        └── Tests/
            └── Functional/
```

## Sections Structure

Sections group related Containers (bounded contexts):

```
src/
├── Containers/
│   ├── SectionA/               # e.g., Inventory
│   │   ├── ContainerA1/        # e.g., Product
│   │   ├── ContainerA2/        # e.g., Stock
│   │   └── ContainerA3/        # e.g., Supplier
│   ├── SectionB/               # e.g., Order
│   │   ├── ContainerB1/        # e.g., Cart
│   │   ├── ContainerB2/        # e.g., Order
│   │   └── ContainerB3/        # e.g., Invoice
│   └── SectionC/               # e.g., Payment
│       ├── ContainerC1/        # e.g., PaymentMethod
│       └── ContainerC2/        # e.g., Transaction
└── Ship/
    ├── ContainersBay/          # Base classes for all containers
    ├── BridgeDeck/             # Shared interfaces & schemas
    ├── ShipBallast/            # Integration adapters
    └── EngineRoom/             # Core services (DI, loaders)
```

## Key Structural Rules

1. All Containers in a project MUST have the same folder structure
2. A Container may not use all component types, but available ones must be in the correct location
3. UI types (API, WEB, CLI) are separated within each Container
4. Tests are co-located: unit tests at Container root, functional tests within each UI type
5. Each Route file contains a single Endpoint
6. Each Action/Task file contains a single class with a single `run()` function

## Data Flow Pattern

```
Route → Middleware → Controller → Action → [Task1, Task2, Task3] → Action → Controller → Transformer/View → Response
```

## CQRS (Optional Enhancement)

- Commands (write operations): Handled by Data Services/Tasks
- Queries (read operations): Managed by Repositories
- Separates read/write models for performance and scalability
