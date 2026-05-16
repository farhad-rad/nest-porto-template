# Requirements: PortoNest Initial

## Introduction

PortoNest is a framework built on top of NestJS implementing the Porto SAP (Software Architectural Pattern) architecture. The project delivers three artifacts:

1. **PortoNest Framework Template** — A NestJS project template with Porto SAP architecture, distributed as an npm CLI package for scaffolding new projects.
2. **PortoNest Kiro Power** — A Kiro power that provides architecture awareness, steering, and hooks for PortoNest projects.
3. **PortoNest VS Code Extension** — A VS Code plugin for linting framework-specific conventions (details TBD).

This requirements document focuses on the framework template and its core Ship layer functionalities.

## Glossary

- **Porto SAP**: Software Architectural Pattern — a two-layer architecture (Containers + Ship) with defined component types and interaction rules.
- **Container**: A modular unit encapsulating a specific domain's business logic (Actions, Tasks, Models, UI, etc.).
- **Section**: A group of related Containers (equivalent to a DDD bounded context). Containers always exist within a Section.
- **Ship Layer**: Infrastructure layer providing shared base classes, interfaces, adapters, and core services used by all Containers.
- **Facade**: A proxy/wrapper pattern that delegates method calls to a configurable driver implementation.
- **Driver**: A concrete implementation of a service abstraction (e.g., RedisCache, MemoryCache are drivers of a CacheService).
- **File Discovery**: Automatic detection and code-generation of import files for Porto components during development via webpack plugin.

---

## Requirement 1: Monorepo Structure & Sample Project

**User Story:** As a developer working on PortoNest, I want a clear monorepo structure separating each deliverable into its own folder, with a sample project that stays in sync as development progresses, so that I can develop, test, and integrate all parts together.

### Acceptance Criteria

#### 1.1 — Folder Structure

1. The project root contains 5 top-level folders: `/power`, `/plugin`, `/package`, `/template`, `/sample`.
2. `/power` contains the Kiro Power source files (POWER.md, steering files, hooks, MCP servers if any).
3. `/plugin` contains the VS Code extension source code.
4. `/package` contains the npm package source (CLI + webpack plugins + dev tooling).
5. `/template` contains the actual PortoNest project template files (the `src/` folder with `ship/` and `containers/`, plus root config files like `package.json`, `tsconfig.json`, `nest-cli.json`, `webpack.config.js`). This is what gets copied/scaffolded when running `npx porto-nest init`.
6. `/sample` contains a sample project that uses local references (e.g., `file:../package` in devDependencies) to `/template`, `/package`, and `/plugin` for integration testing and demonstration.

#### 1.2 — Sample Project

7. The sample project references the package via local path (e.g., `file:../package` in devDependencies).
8. A Kiro hook triggers on checkpoint saves after meaningful progress to template, plugin, or package, and rebuilds/updates the sample project using local references so it stays in sync as development progresses.
9. Sample project content and structure details are TBD.

#### 1.3 — Template Architectural Steering

10. The `/template` folder includes a `.kiro/` directory with comprehensive architectural steering files.
11. These steering files explain Porto SAP architecture, PortoNest-specific rules, component principles, path alias rules, import restrictions, facade system usage, naming conventions, and all musts/must-nots in full detail. Specifically they cover:
    - What Porto SAP is and its principles
    - What PortoNest is and how it extends NestJS
    - All component rules (Actions, Tasks, Controllers, Models, Routes, Requests, Transformers, Views, Exceptions, Sub-Actions)
    - Path alias rules and import restrictions
    - Facade system usage rules
    - Container/Section structure rules
    - File discovery system behavior
    - Naming conventions (lower-kebab-case, component suffixes)
    - What is allowed and what is forbidden
12. The steering files serve as a complete architectural guide for Kiro — any Kiro session in a PortoNest project immediately understands and complies with the architecture.
13. These are **architectural steering only** — project-specific business/product steering is added separately by the developer.

---

## Requirement 2: Template Project Structure

**User Story:** As a developer, I want to scaffold a PortoNest project with the correct Porto SAP folder structure, so that I can immediately start building features following the architecture.

Note: This structure is defined in the `/template` folder of the monorepo and is what gets scaffolded when running `npx porto-nest init`.

### Acceptance Criteria

1. The template project root contains `src/` directory with two subdirectories: `src/ship/` and `src/containers/`.
2. All folder names use lower-kebab-case convention.
3. Containers always exist within a Section: `src/containers/{section-name}/{container-name}/`.
4. The Ship layer is organized into its four areas under `src/ship/`: base classes (containers-bay), shared interfaces (bridge-deck), integration adapters (integration), and core services (engine-room).
5. Standard root files exist: `package.json`, `tsconfig.json`, `nest-cli.json`, `webpack.config.js`.
6. Path aliases are configured: `@ship` maps to `src/ship/`, `@containers` maps to `src/containers/`, `~~` maps to `src/ship/imports/`.
7. The template includes a `.kiro/` directory with comprehensive architectural steering files that explain Porto SAP and PortoNest rules in full detail, so that Kiro immediately understands and complies with the architecture when working in the project.

---

## Requirement 3: File Discovery System (Engine Room)

**User Story:** As a developer, I want Porto components to be automatically discovered and registered during development, so that I don't need to manually import and wire every component.

### Acceptance Criteria

#### 3.1 — Plugin Execution

1. A webpack plugin (`PortoCodegenPlugin`) runs before each compilation during development (watch mode) and on build.
2. The plugin watches `src/ship/` and `src/containers/` directories for file changes.
3. The plugin is also executed via the `prepare` script in `package.json`, so that generated files exist after `npm install` before the first dev/build run.

#### 3.2 — Discovery Logic

4. The plugin gathers well-defined Porto components (e.g., configs, migrations, events) from across all containers and/or the ship.
5. Each well-defined component type has its own specific discovery/gathering function that defines how files are found and how the generated output is structured.
6. Discovery functions glob for specific file patterns across Ship and/or Containers.
7. Each discovery function can target: Ship only (flag 1), Containers only (flag 2), or both (flag 3, default).
8. New discovery functions can be added by developers by creating a file in the discovery configuration directory and exporting a function.

#### 3.3 — Output

9. Auto-generated TypeScript files are written to `src/ship/imports/` containing aggregated imports of discovered components.
10. The `src/ship/imports/` folder contains a `.gitkeep` file so the folder is tracked in git.
11. All auto-generated files inside `src/ship/imports/` are gitignored — they are regenerated on install/build/watch.
12. Generated files include a banner comment indicating they are auto-generated and must not be edited manually.
13. Files are only rewritten when their content actually changes (content-hash comparison).

#### 3.4 — Import Paths & Aliases

14. The `src/ship/imports/` folder is aliased as `~~` in path configuration. Example: `import from '~~/config'` resolves to `src/ship/imports/config.ts`.
15. Import paths within generated files use the configured aliases (`@ship`, `@containers`).
16. Unique hashed identifiers are generated for each discovered file to avoid naming collisions.

---

## Requirement 4: Facade System (`src/ship/integration/facade-system/`)

**User Story:** As a developer, I want to define services with swappable driver implementations (like Laravel's Cache with Redis/Memory drivers, Logger with stdout/file drivers, Filesystem with local/S3 drivers), so that I can change service backends without modifying consumer code and use fake drivers in tests.

### Acceptance Criteria

#### 4.1 — `@Facade` Decorator

1. A `@Facade(InterfaceOrAbstract)` class decorator exists that transforms a class into a facade wrapper.
2. The decorator is applied on top of the main service class — the class that consumers will inject:
   ```ts
   @Facade(ILoggerService)
   class LoggerService {}
   ```
3. The decorator makes the class an `@Injectable()` NestJS service automatically.
4. Only an interface or abstract class is accepted as the `@Facade` argument — it serves as the injection token/name indicator.

#### 4.2 — Facade Wrapper Behavior

5. The facade wrapper class is always registered as a **singleton** in NestJS DI, regardless of driver lifetimes.
6. The injection token is the interface/abstract class passed to `@Facade`, so consumers inject the interface and receive the facade wrapper transparently:
   ```ts
   class SomeTask {
     constructor(private readonly logger: ILoggerService) {}
   }
   ```
   Here `logger` receives the facade wrapper instance, not a driver directly.
7. When methods are called on the facade wrapper, it delegates to the currently active driver instance resolved via NestJS's internal DI system.
8. The facade wrapper uses a **lazy service loader** for the driver — the actual driver is resolved from NestJS DI when first needed.

#### 4.3 — Driver Lifetime & Resolution

9. Driver lifetime (singleton, request-scoped, transient) is managed entirely by NestJS DI — the Facade System does **not** control or manage driver lifetimes.
10. The service abstract class (not the Facade System) is responsible for:
    - Declaring which drivers exist and their class mappings
    - Configuration parsing logic (which driver is active)
    - Any additional services the drivers need
    - All driver resolution logic
11. The Facade System only provides the **mechanism and methods** for services to declare their drivers and facade-related functionalities — it does not find, determine, or configure drivers itself.

#### 4.4 — Hot-Swapping & `.via()` Method

12. The facade wrapper supports **hot-swapping** the active driver at runtime because what is injected is always the wrapper class instance, not the driver directly.
13. Hot-swapping works in two modes:
    - **Global swap**: Change the active driver for all subsequent calls across all consumers.
    - **Local/one-shot swap**: Use a specific driver for just one call via the `.via(driverName)` method, without changing the global default (similar to Laravel's `via` method on services).
14. The `.via(driverName)` method returns a reference to the specified driver for one-shot usage.
15. The hot-swap mechanism is implemented via a service within the facade wrapper class to be applicable.
16. A user may want to use one specific driver only one time (one-shot) or swap globally for all instances.

#### 4.5 — Module Registration

17. The decorated class gains a static `forRoot()` method for NestJS module registration — can be added to the `imports` array of NestJS modules.
18. The decorated class gains a static `faked()` method for test module registration with a fake/mock driver, enabling easy test isolation.

#### 4.6 — Scope & Location

19. Containers can define and use facades — facades are **not** restricted to the Ship layer.
20. The Facade System is a set of utilities (decorators, base classes, helpers) located at `src/ship/integration/facade-system/` — it is **not** a NestJS module itself. The facade wrapper class created via `@Facade` is the NestJS standalone Injectable singleton.
21. The Facade System provides the infrastructure; individual services built with it can live anywhere (Ship or Containers).

---

## Requirement 5: Path Alias System

**User Story:** As a developer, I want a structured path alias system that enforces Porto SAP dependency rules at the import level, so that my code remains portable, decoupled, and ready for microservices extraction.

### Acceptance Criteria

#### 5.1 — Alias Definitions

1. `@ship/path/to/file` resolves to `src/ship/path/to/file.ts` — used for importing from the Ship layer.
2. `@containers/{section_name}/{container_name}/path/to/file` resolves to `src/containers/{section_name}/{container_name}/path/to/file.ts` — used only for cross-section references (dangerous).
3. `~~/file` resolves to `src/ship/imports/file.ts` — the exclusive way to access auto-generated discovery imports.
4. `~this/path` resolves to `src/containers/{current-section}/{current-container}/path.ts` — references within the same container.
5. `~{container_name}/path` resolves to `src/containers/{current-section}/{container_name}/path.ts` — cross-container reference within the same section.
6. `@ship/imports` must NEVER be used — `~~` is the only valid way to access `src/ship/imports/`.
7. `~this` and `~{container_name}` are meaningless/invalid inside the Ship layer.

#### 5.2 — Implementation

8. During development, a webpack plugin resolves all path aliases at compile time.
9. For production builds, aliases are compiled away into regular paths — zero runtime overhead in the built output.
10. The VS Code extension must understand, resolve, and provide IntelliSense for these aliases since they are non-standard and IDEs won't recognize them natively.

#### 5.3 — Import Rules Inside Containers

11. Files within the **same component** (e.g., files within `services/logger-service/`) MUST use **relative paths** to reference each other. This is mandatory, not optional — ensures the component is portable across containers.
12. Files within the same component are FORBIDDEN from using any alias (`~this`, `@ship`, etc.) to reference each other — only relative paths.
13. Any reference to a **different component within the same container** (e.g., from an action to a task) MUST use `~this/`.
14. Relative paths are FORBIDDEN for anything outside the current component's folder boundary.
15. Ship references MUST use `@ship/`.
16. Auto-generated import references MUST use `~~`.
17. Cross-container references within the same section MUST use `~{container_name}/` (not `@containers/{section}/{container}/`). These are flagged as **warning** — allowed by Porto but not recommended.
18. Cross-section references MUST use `@containers/{section_name}/{container_name}/`. These are flagged as **danger/error** and are forbidden by default.

#### 5.4 — Import Rules Inside Ship

19. Files within the **same component** in the Ship (e.g., files within `ship/services/logger-service/`) MUST use **relative paths** to reference each other (same rule as containers).
20. Files within the same ship component are FORBIDDEN from using any alias to reference each other — only relative paths.
21. References to other Ship components (different component) MUST use `@ship/`.
22. Auto-generated import references MUST use `~~`.
23. Any reference to containers (`@containers/...`) from within the Ship is **FORBIDDEN** — the build must fail. The Ship must never rely on any container's existence.
24. `~this` and `~{container_name}` are invalid in Ship context — using them must produce an error.

#### 5.5 — Cross-Section Rules & `// @cross-section` Flag

25. Cross-section references are NOT allowed by default — no exceptions for events or any other component type.
26. The ONLY way to make a cross-section reference valid is to add a `// @cross-section` comment flag at the top of the file.
27. The `// @cross-section` comment works like `// TODO` or `// @eslint-suppress-*` — it is recognized as a TODO item, meaning: "this file needs refactoring when splitting to microservices."
28. Without the `// @cross-section` flag, any cross-section import causes the **build to fail**.
29. Files flagged with `// @cross-section` must be implemented using a broker, HTTP API, or other inter-service communication mechanism.
30. When converting a monolithic PortoNest project to microservices, a copy of the entire Ship + one entire Section is extracted as a standalone microservice. All `// @cross-section` flagged files identify exactly which code needs refactoring into third-party service calls (synchronous or async).

#### 5.6 — Component Folder Import Rule

31. All services/components that have an `index.ts` MUST be imported via their folder name only (which resolves to `index.ts`). Direct file imports into a component's internals from outside that component are FORBIDDEN.
32. This rule applies universally to all services in both Ship and Containers — it ensures components are portable and their internal structure can change without breaking external consumers.

---

## Requirement 6: npm Package (CLI + Dev Tooling)

**User Story:** As a developer, I want a single npm package that both scaffolds new PortoNest projects and provides the build tooling (webpack plugins, import linting) as a devDependency, so that I have one unified tool for project creation and development infrastructure.

### Acceptance Criteria

#### 6.1 — Package Dual Role

1. A single npm package serves both as a CLI tool and as a devDependency in PortoNest projects.
2. When used as CLI: `npx porto-nest init <project-name>` scaffolds a new project.
3. When used as devDependency: provides webpack plugins and build tooling to the project.
4. The scaffolded project always includes this package in its `devDependencies`.

#### 6.2 — CLI Capabilities

5. `init` command scaffolds a complete PortoNest project with all Ship layer infrastructure pre-configured.
6. Additional artisan-like CLI commands when run inside a project folder (TBD — to be discussed in future).

#### 6.3 — Dev Tooling (provided as devDependency)

7. Provides the `PortoCodegenPlugin` (file discovery webpack plugin).
8. Provides the path alias resolution webpack plugin.
9. Provides build-time import rule enforcement (import linting that fails the build on violations).
10. Provides the `prepare` script functionality for generating import files on `npm install`.

---

## Requirement 7: Kiro Power

**User Story:** As a developer using Kiro, I want a power that understands PortoNest architecture, so that Kiro can enforce rules and assist with component creation.

### Acceptance Criteria

1. The power provides steering files that inform Kiro of Porto SAP architecture rules, component principles, and naming conventions.
2. The power creates hooks that trigger when Porto components are created, edited, or requested to be created.
3. Hooks enforce Porto SAP rules (e.g., Actions must have single `run()` function, Tasks must not call other Tasks, Controllers must not contain business logic).
4. The power is installable in any PortoNest project.

---

## Requirement 8: VS Code Extension

**User Story:** As a developer, I want VS Code to lint my PortoNest project for architecture violations and provide full IntelliSense for custom path aliases, so that I get immediate feedback on rule violations and seamless navigation.

> **Note:** Every error reported by the VS Code extension is also enforced by the webpack plugin at build time — the build process must fail on any error-level violation. This applies universally to all error-level diagnostics in this requirement.

### Acceptance Criteria

#### 8.1 — Path Alias IntelliSense

1. The extension resolves all custom path aliases (`@ship`, `@containers`, `~~`, `~this`, `~{container_name}`) and provides **full IntelliSense** — every file-referencing feature that VS Code supports must work with these aliases, including but not limited to: go-to-definition, find all references, count references, peek definition, autocomplete/suggestions, hover information, rename/refactor across files, and any other referencing capability VS Code provides.
2. The extension understands the context (Ship vs Container, which section/container the file is in) to determine which aliases are valid and how they resolve.

#### 8.2 — Import Rule Enforcement

3. **Error**: Relative imports that cross the current component's folder boundary.
4. **Error**: Non-relative imports used within the same component (files in the same component MUST use relative paths).
5. **Error**: `~this` or `~{container_name}` usage inside the Ship layer.
6. **Error**: Any `@containers/...` import from within the Ship layer.
7. **Error**: Cross-section imports (`@containers/{section}/{container}/`) without a `// @cross-section` flag at the top of the file.
8. **Warning**: Cross-container imports within the same section (`~{container_name}/`) — allowed but not recommended.
9. **Error**: Using `@ship/imports` instead of `~~`.
10. **Error**: Direct file imports into a component's internals from outside that component (must import via folder name / index.ts).

#### 8.3 — Cross-Section Flag Recognition

11. Recognizes `// @cross-section` as a TODO-like comment flag (similar to `// TODO` or `// @eslint-suppress-*`).
12. Suppresses cross-section lint errors when the flag is present at the top of the file.
13. Highlights flagged files as TODO items in the editor.

#### 8.4 — Config Component Linting

14. **Error**: Config file name does not match the `registerAs()` name inside it.
15. **Error**: Duplicate config names across the application.
16. **Error**: Direct import of a config component file (`*.config.ts`) from anywhere other than the auto-generated `src/ship/imports/config.ts`.

#### 8.5 — Additional Functionality (TBD)

16. Further linting rules and features to be defined in future discussions.

---

## Requirement 9: Config Component & Configuration Service

**User Story:** As a developer, I want a standardized config component pattern with automatic discovery and a centralized configuration service, so that all application configuration is consistent, validated, and accessible anywhere.

### Acceptance Criteria

#### 9.1 — Config Component Structure

1. Config files live in `src/ship/config/` or `src/containers/{section}/{container}/config/`.
2. Config files are named `{config_name}.config.ts` (e.g., `app.config.ts`, `database.config.ts`).
3. Every config file uses `@nestjs/config`'s `registerAs()` and exports it as default:
   ```ts
   import { registerAs } from '@nestjs/config';
   export default registerAs('{config_name}', () => ({
     // config values, typically from process.env with defaults
   }));
   ```
4. The `{config_name}` in the filename MUST match the `{config_name}` in `registerAs()`.
5. Config names MUST be unique across the entire application (all containers + ship).
6. Config values should have descriptive block comments explaining their purpose (Laravel-style documentation blocks).
7. Config component files (`*.config.ts`) MUST NOT be imported directly by any file except the auto-generated `src/ship/imports/config.ts`. Any direct import of a config file is an error.

#### 9.2 — Config Discovery

7. The `PortoCodegenPlugin` discovers all `*.config.ts` files across ship (`src/ship/config/`) and containers (`src/containers/*/*/config/`).
8. Generates `src/ship/imports/config.ts` with hashed imports and a configs array default export.
9. The generated file format:
   ```ts
   import {hash} from '{importPath}';
   // ... more imports
   const configs = [{hash1}, {hash2}, ...];
   export default configs;
   ```

#### 9.3 — Configuration Service (`src/ship/services/configuration-service/`)

10. A NestJS standalone `@Injectable()` service that wraps `@nestjs/config`'s `ConfigService`.
11. Initializes by loading `.env` (or `.env.{APP_ENVIRONMENT}`) and passing discovered configs from `~~/config` to the `@nestjs/config` service.
12. Has an `index.ts` exporting all public classes, methods, and functionalities.
13. External consumers MUST import from the folder name only (resolves to `index.ts`) — no direct file imports into service internals are allowed.
14. Imported once in `AppModule`, injectable anywhere in the application.

#### 9.4 — Linting & Build Enforcement

15. VS Code plugin lints that config filename and `registerAs()` name match.
16. VS Code plugin lints that config names are unique across the whole application.
17. Duplicate config names produce an **error** in VS Code.
18. Webpack must **fail the build** if duplicate config names are detected.
