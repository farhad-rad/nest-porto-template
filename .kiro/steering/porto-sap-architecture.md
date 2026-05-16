---
inclusion: auto
---

# Porto SAP Architecture - Core Principles

This project follows the **Porto SAP (Software Architectural Pattern)** architecture. All code MUST conform to these principles.

## Overview

Porto is a modern software architectural pattern grounded in DDD, Modular, Micro Kernel, MVC, Layered, and ADR architectures. It supports SOLID, OOP, LIFT, DRY, CoC, GRASP, High Cohesion, and Low Coupling principles.

Porto is suited for medium to large backend web applications. It allows starting as a clean monolith and expanding to microservices as needed.

## Two Layers

Porto is composed of exactly two layers:

### 1. Containers Layer (Business Logic)
- Holds ALL application business logic (unique functionalities and operations)
- Where 90% of development time is spent
- Contains modular units called **Containers**, each encapsulating a specific piece of functionality
- Containers are similar to Modules (Modular architecture) or Domains (DDD)

### 2. Ship Layer (Infrastructure)
- Holds shared infrastructure code used across all Containers
- Decouples application code from the framework and 3rd party libraries
- Divided into 4 areas:
  - **Containers Bay (Base Classes)**: Abstract/base classes, utilities that all Container Components inherit from
  - **Bridge Deck (Shared Interfaces)**: Interfaces, event contracts, API specs, types (non-executable definitions)
  - **Ship Ballast (Integration Adapters)**: Adapters, decorators, facades, bridges, proxies, configurations (optional but valuable)
  - **Engine Room (Core Services)**: Automatic DI, class loaders, component registrars (reusable across apps)

## Code Levels

- **High-level code**: Business logic in Containers layer
- **Mid-level code**: General application code in Ship layer
- **Low-level code**: Framework code in vendor/node_modules

## Containers

### Structure
Every Container MUST adhere to the same structure. A Container encapsulates a specific domain/functionality.

### Sections
- A **Section** is a group of related Containers (equivalent to a bounded context in DDD)
- Sections can be deployed separately
- Communication between Sections should use Events/Commands (event-driven)
- Within a Section, Containers can depend on each other directly

### Container Dependencies
- Within a Section: Direct dependencies allowed
- Between Sections: Use event-driven communication (Kafka, RabbitMQ, etc.)
- Actions may call Tasks from other Containers
- Models from different Containers may have relationships

## Request Life Cycle

1. User calls an Endpoint in a Route file
2. Endpoint calls Middleware for Authentication
3. Endpoint calls its corresponding Controller function
4. Request object applies validation and authorization rules
5. Controller calls an Action (passes request data)
6. Action executes business logic by calling multiple Tasks
7. Tasks execute reusable subsets of business logic
8. Action prepares final result for Controller
9. Controller builds response using View or Transformer

## Monolithic to Microservices

- Monolithic = one cargo ship of Containers
- Micro-Services = multiple cargo ships of Containers
- Sections can be extracted and deployed separately
- Communication between services via Events and/or Commands
