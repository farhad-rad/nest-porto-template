---
inclusion: auto
---

# Porto SAP Components - Rules & Principles

## Component Types

Components are classes with specific roles and responsibilities. They live inside Containers. Two types:
- **Main Components**: Essential, mandatory (Actions, Tasks, Controllers, Models, Views, Routes, Requests, Transformers, Exceptions, Sub-Actions)
- **Optional Components**: Supplementary, added as needed

## Main Components

### Actions
Actions represent Use Cases of the application.

**Rules:**
- Every Action MUST be responsible for a single use case
- An Action MAY retrieve data from Tasks and pass data to another Task
- An Action MAY call multiple Tasks, including Tasks from other Containers within the same Section
- Actions MAY return data to the Controller
- Actions MUST NOT return a response (Controller's job)
- An Action MUST NOT call another Action — use Sub-Actions instead
- Actions are mainly called from Controllers (also from Event Listeners, Commands, other classes)
- Actions MUST NOT be called from Tasks
- Every Action MUST have only a single function named `run()`
- The `run()` function can accept a Request Object as parameter
- Actions are responsible for handling all expected Exceptions
- Ideally, an Action should be an array of Tasks executed in sequence (pipeline pattern)

### Tasks
Tasks hold shared business logic between multiple Actions across different Containers.

**Rules:**
- Every Task SHOULD have a single responsibility
- A Task MAY receive and return data
- A Task MUST NOT return a response (Controller's job)
- A Task MUST NOT call another Task
- A Task MUST NOT call an Action
- Tasks SHOULD only be called from Actions (can be from Actions of other Containers)
- A Task MUST NOT be called from a Controller
- Every Task SHOULD have only a single function named `run()`
- Use pipeline design pattern for data flow between Tasks

### Sub-Actions
Sub-Actions eliminate code duplication in Actions by sharing a sequence of Tasks.

**Rules:**
- Sub-Actions MUST call Tasks (if no Tasks needed, it should be a Task instead)
- A Sub-Action MAY retrieve/pass data between Tasks
- A Sub-Action MAY call multiple Tasks (even from other Containers)
- Sub-Actions MAY return data to the Action
- Sub-Actions MUST NOT return a response
- Sub-Actions SHOULD NOT call another Sub-Action
- Sub-Actions SHOULD be called from Actions (also from Events, Commands, but NOT from Controllers or Tasks)
- Every Sub-Action MUST have only a single function named `run()`

### Controllers
Controllers validate requests, serve request data, and build responses.

**Rules:**
- Controllers MUST NOT know anything about business logic
- A Controller SHOULD only: (1) Read Request data, (2) Call an Action, (3) Build a Response
- Controllers MUST NOT have business logic
- Controllers MUST NOT call Tasks directly — only Actions
- Controllers CAN only be called by Route Endpoints
- Every Container UI folder (Web, API, CLI) has its own Controllers

### Models
Models represent data in the database (the M in MVC).

**Rules:**
- A Model MUST NOT contain business logic
- A Model should only contain code/data that represents itself (relationships, hidden fields, table name, fillable attributes)
- A single Container MAY contain multiple Models
- A Model MAY define relationships with other Models

### Routes
Routes map incoming HTTP requests to Controller functions.

**Rules:**
- Three types: API Routes, Web Routes, CLI Routes
- API and Web Routes MUST be in separate folders
- Every Container SHOULD have its own Routes
- Every Route file SHOULD contain a single Endpoint
- An Endpoint's only job is to call a Controller function

### Requests
Requests serve user input and apply Validation/Authorization rules.

**Rules:**
- A Request MAY hold Validation/Authorization rules
- Requests SHOULD only be injected in Controllers
- When injected, they automatically validate request data (throw Exception if invalid)
- Requests MAY check if user is authorized

### Transformers (Response Transformers)
Equivalent to Views but for JSON responses. Transform Models into Arrays/JSON.

**Rules:**
- All API responses MUST be formatted via Transformers
- Every Model returned by an API call SHOULD have a corresponding Transformer
- A single Container MAY have multiple Transformers

### Views
Views contain HTML served by the application (the V in MVC).

**Rules:**
- Views SHOULD only be used from Web Controllers
- Views should be separated into multiple files/folders
- A single Container MAY have multiple View files
- Views MUST NOT contain business logic or data manipulation

### Exceptions
Well-defined, expected error handling.

**Rules:**
- Container Exceptions live in Containers; general Exceptions live in Ship
- Tasks, Sub-Tasks, Models, and any class can throw specific Exceptions
- The caller MUST handle all expected Exceptions from the called class
- Actions MUST handle all Exceptions (prevent leaking to upper Components)
- Exception names SHOULD be specific with clear descriptive messages

## Optional Components

Available as needed:
- **Middlewares**: HTTP request/response intermediaries
- **Repositories**: Abstract data persistence logic
- **Service Providers**: Register services, dependency management
- **Migrations**: Database schema versioning
- **Database Seeders**: Initial test data
- **Events Publisher**: Broadcast application events
- **Event Handlers**: Respond to events
- **Data Transfer Objects (DTOs)**: Carry data between systems
- **Jobs**: Background long-running tasks
- **Commands**: Custom CLI commands
- **Tests**: Automated checks (Unit, Integration, Performance, Security)
- **Contracts/Interfaces**: Define expected behaviors
- **Value Objects**: Simple entities without identity
- **Mixins**: Share code between classes (DRY)
- **Factories**: Generate test data
- **Policies**: Authorization policies
- **Criteria**: Complex database queries
- **Configurations**: Application settings
- **Localizations**: UI translations

## Naming Convention

Append the component type to a fully descriptive name:
- Actions: `RegisterUserAction`, `ListProductsAction`, `MakeOrderAction`
- Tasks: `ValidatePaymentTask`, `UpdateInventoryTask`, `DeliverOrderTask`

## AI-Driven Development

Porto's single responsibility principle (one function `run()` per class) makes it optimal for LLM-assisted development. Each file can be developed, tested, and deployed independently.
