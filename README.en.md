<p align="center">
  <a href="README.md"><img src="https://img.shields.io/badge/Español-🇪🇸-red?style=for-the-badge" alt="Español" /></a>
  &nbsp;&nbsp;&nbsp;
  <a href="README.en.md"><img src="https://img.shields.io/badge/English-🇺🇸-blue?style=for-the-badge" alt="English" /></a>
</p>

<p align="center">
  <img src="logo.jpg" alt="Model Domain Architecture Logo" width="1024">
</p>

> **Important Note**
>
> Model Domain Architecture (MDA) is a **pragmatic and flexible** architectural approach, inspired by Domain-Driven Design and Clean Architecture principles, but **adapted to real-world Laravel project needs**.
>
> Its goal is to be a **clear and practical starting point**, not a rigid or exhaustive framework like DDD, Hexagonal, or Clean Architecture.  
> MDA defines a recommended organization of layers and responsibilities, **but does not technically prevent direct access to internal components** (Finder, Query, Action, etc.).  
>
> The recommended path is always to go through Services, and only in specific cases use direct access, with judgment and responsibility. This foundation can be expanded or adjusted according to the project's needs.

- [Model Domain Architecture (MDA)](#model-domain-architecture-mda)
- [🎯 Goal](#-goal)
- [1. General Philosophy](#1-general-philosophy)
- [2. Folder Structure](#2-folder-structure)
- [3. Description of Each Component](#3-description-of-each-component)
  - [3.1 Models](#31-models)
  - [3.2 Finders](#32-finders)
  - [3.3 Queries](#33-queries)
  - [3.4 Repositories](#34-repositories)
  - [3.5 Actions](#35-actions)
  - [3.6 DTOs](#36-dtos)
  - [3.7 Services](#37-services)
  - [3.8 Use Cases](#38-use-cases)
  - [3.9 Extensions and Additional Components](#39-extensions-and-additional-components)
- [4. Relationship Between Components](#4-relationship-between-components)
- [5. MDA Advantages](#5-mda-advantages)
- [6. Summarized Flow Example](#6-summarized-flow-example)
- [7. When to Use MDA](#7-when-to-use-mda)
- [8. How to Use MDA (Recommendations and Best Practices)](#8-how-to-use-mda-recommendations-and-best-practices)
  - [8.1 The Recommended Path](#81-the-recommended-path)
  - [8.2 Non-Strict Basic Rules](#82-non-strict-basic-rules)
  - [8.3 Business Logic Placement](#83-business-logic-placement)
  - [8.4 Pragmatism Over Purism](#84-pragmatism-over-purism)
  - [8.5 MDA is a Starting Point](#85-mda-is-a-starting-point)
- [9. Conclusion](#9-conclusion)

# Model Domain Architecture (MDA)

**Model Domain Architecture (MDA)** is a pragmatic architectural approach, inspired by Domain-Driven Design (DDD) and Clean Architecture principles, but specifically adapted to Laravel projects where the natural separation occurs by **models** rather than by complex conceptual domains.

MDA organizes the project around the system's *core models* (for example: `User`, `Order`, `Product`, etc.), maintaining a clear, scalable, and easy-to-navigate structure even in applications with multiple databases.

---

## 🎯 Goal

To provide an architecture that is:

* **Scalable**: supports growth without producing giant classes or unmanageable repositories.
* **Organized by model**: each model has its own mini-module with all its behaviors.
* **Predictable**: any developer knows exactly where to look for logic related to a model.
* **Practical**: avoids the unnecessary over-engineering that sometimes arises with full DDD.

---

# 1. General Philosophy

In MDA, each system model functions as a "bounded domain." Not a *bounded context* as in DDD, but a **functional area delimited by the model itself and its database**.

Each model contains:

* **Actions** (behaviors that modify state)
* **Use Cases**
* **Finders** (simple queries)
* **Queries** (complex queries)
* **DTOs**
* **Repositories**
* **Services** (the main orchestrator)

This creates small islands of clear responsibility.

---

# 2. Folder Structure

```
app/
  Actions/
    Order/
    User/
    Product/
  DTO/
    Order/
    User/
    Product/
  Finders/
    OrderFinder.php
    UserFinder.php
    ProductFinder.php
  Queries/
    Order/
    User/
    Product/
  Repositories/
    OrderRepository.php
    UserRepository.php
    ProductRepository.php
  Services/
    Interfaces/
      OrderServiceInterface.php
      ClientServiceInterface.php
    OrderService.php
    ClientService.php
  UseCases/  
  Models/
    Main/
      Order.php
      User.php
      Product.php
    Logistic/
      Delivery.php
```

> Note:
> Models are grouped by database when the application uses multiple connections.
> This is a valid exception because it responds to infrastructure needs, not domain needs.

---

# 3. Description of Each Component

## 3.1 Models

Eloquent Models divided by database.

This is an exception within the architecture because the model **reflects the physical structure**.

Advantages:

* Makes it easy to see which models belong to which database.
* Avoids chaos when multiple connections exist.

---

## 3.2 Finders

> An object whose purpose is to find and return domain entities or collections, encapsulating how data is accessed.

* `findById($id)`
* `findByUserId($userId)`
* `findByLastOrderWithPendingItemsByUserId($userId)`

All can be complex and still be Finders.

They are grouped **by model**, in a single file per model:

```
OrderFinder.php
UserFinder.php
```

🔀 Possible Variations in Finder Organization

By default, MDA proposes grouping all simple read methods for a model in a single file, for example:

* UserFinder.php
* OrderFinder.php

This keeps the directory tree lightweight and avoids creating dozens of small classes.

However, this is not the only valid way to organize Finders.
If your team prefers a more granular structure—for example, one Finder class per query:

```
FindUserById.php
FindUsersByRole.php
FindOrdersByCustomer.php
```

This is also fully compatible with MDA.

The central idea is that:

Simple queries should be separated from the service and business logic.

The internal organization should be consistent for your project.

The team can modify or scale the structure without friction.

In other words:

> MDA defines where things live, but not exactly how they should look.
> Each project can adjust the level of granularity it prefers.

General rule:

> * If it returns entities: Finder
> * If it returns data to display/use: Query

---

## 3.3 Queries

> An object oriented to specific reads for consumption, typically optimized, aggregated, or projected.

Complex SQL/Eloquent queries:

* multiple joins
* advanced filters
* aggregations
* subqueries
* specific read logic

They are grouped by **model folders**, for example:

```
Queries/Order/OrderQueries.php
Queries/User/UserQueries.php
```

Function examples:

* getCreditsSummaryByClientId($clientId)
* getTotalClientOrdersForDashboard($clientId)

Queries are isolated because **it is normal** for them to grow significantly.

---

## 3.4 Repositories

Classes focused on **basic persistence**:

* `create()`
* `update()`
* `delete()`

They avoid the overhead of having 50 specialized methods within the repository.

Data queries are NOT mixed here; they go in Queries.

Model retrieval is NOT mixed here; they go in Finders.

---

## 3.5 Actions

Domain modifier behaviors:

* UpdateOrderAction
* CreateClientAction
* DeleteUserAction

> Actions represent concrete operations that modify the state of one or more models

Actions use repositories and never queries.

Actions are conceptually similar to what is called in other places:

- Command (Command Pattern) 
- Application-level Command (CQRS light)

---

## 3.6 DTOs

DTOs (Data Transfer Objects) are used to transport data between layers, organize requests and responses, and define clear contracts between components.

In MDA, the use of DTOs is **recommended but not mandatory**.

Like other components, they are separated by model.

**Usage Recommendations**

- DTOs are recommended in Actions that persist or modify state, especially when receiving multiple data points.
  ```
  UpdateUserAction::execute(UpdateUserDTO $dto);
  ```  
- DTOs are useful as contracts between:
  - Controller → Use Case
  - Use Case → Action
  - Service → Action  
- In projects that seek to isolate the database or ORM, Repositories can return DTOs instead of models, although this is not a requirement for MDA.  
- DTOs can also be used as output objects (Response DTOs) to avoid directly exposing models or internal structures.

**Considerations**

- It is not necessary to use DTOs in all cases.
- For simple flows, direct use of parameters or models may be sufficient.
- The goal of DTOs in MDA is to **improve clarity and maintainability**, not to add unnecessary complexity.

> If you feel you're creating DTOs "just because," you probably don't need them there.

---

## 3.7 Services

Services encapsulate model-specific logic.

A Service can:

- Use Finders for simple reads
- Use Queries for complex reads
- Execute Repositories for persistence
- Apply business rules specific to the model

Services **should not decide high-level flows** or contain composite use case logic. That is the responsibility of **Actions** or **Use Cases**.

---

## 3.8 Use Cases

A **Use Case** corresponds to a system use case that:

- **does not represent a single model**, but combines operations over multiple models,
- **adds cross-cutting business logic**,
- and **orchestrates calls to several associated services/queries/repositories**.

Typical examples of Use Cases can be:

- CheckoutProcess
- CloseAccountProcess
- ImportExternalData
- Settlements that combine multiple entities
- Complex ecosystem reports or synchronizations

Use Cases are conceptually similar to what is known in other architectures as:

- Domain Services  
- Application Services (in Clean Architecture)  
- Cross-cutting use cases that require coordination of multiple models.

Explicitly integrating this type of class into MDA allows maintaining a **uniform, predictable, and scalable** structure without forcing everything to depend on a single model.

---

## 3.9 Extensions and Additional Components

MDA defines the fundamental layers, but **does not claim to be exhaustive**.

If your project needs components not mentioned here

(Value Objects, Validators, Rules, Policies, etc.),

you can extend the architecture with judgment:

- Create folders at the root only when the concept does not clearly fit into UseCases, Actions, Services, Models, or Repositories
- Group by logical criteria (functional or technical)
- Maintain consistency with the existing structure

Some concepts can live directly at the root, while others can be grouped under a common semantic container.

**Valid examples:**

- `app/ValueObjects/` for cross-cutting immutable objects
- `app/Validators/` for complex domain validations
- `app/DomainLogic/Discounts/` for specialized business logic

**Guiding principle:**

> MDA gives you the base structure to organize your code.  
> Each project has specific needs.  
> Extend the architecture naturally when needed,  
> but always asking yourself:  
> "Does this have its own entity or should it live within a Service, Action, or Use Case?"

---

# 4. Relationship Between Components

Recommended flow:

```
Controller → Use Case → Services 
Services → Finder / Query
Services → Action → Repository
```

A Use Case is not always necessary, so you can call the Service directly from the controller:

```
Controller → Services
```

Where:

  * **Controller**: adapts input/output and delegates.
  * **Use Case**: use case that orchestrates the complete flow and makes high-level decisions.
  * **Services**: model-specific or reusable logic.
  * **Finder / Query**: read data access.
  * **Action**: specific write or state modification operation.
  * **Repository**: persistence (create/update/delete).

Specific relationships:

  * Use Case → Services
  * Services → Finder / Query
  * Services → Action
  * Action → Repository
  * Finder → Eloquent (simple reads)
  * Query → Complex SQL/Eloquent

Dependencies must be **unidirectional, clear, and clean**.

---

# 5. MDA Advantages

### ✓ Natural Scalability

As the project grows, only folders within the corresponding model grow.

### ✓ Immediate Navigation

Everything related to `Order` is in the same "model domain."

### ✓ No Giant Super-Repositories

Specific queries are not mixed with basic CRUD operations.

### ✓ Aligned with the Reality of Many Laravel Projects

Many projects do not have well-defined business domains to apply full DDD.

### ✓ Compatible with Partial DDD

If a large domain appears one day (for example, "Finance"), it can group several models under one context.

---

# 6. Summarized Flow Example

Order Controller:

```
$order = $orderService->findById($id);
```

Service:

```
return $this->finder->findById($id);
```

Complex Query:

```
$orders = $orderService->getOrdenesConDescuento();
```

Service calls:

* Queries/Order/getOrdenesConDescuento

Update:

```
$orderService->actualizarEstado($id, $dto);
```

Service → Action → Repository

---

# 7. When to Use MDA

Ideal for projects:

* Laravel
* with multiple models
* with very diverse queries
* with a need for order and scalability
* where full DDD is too much

Not ideal for:

* extremely small systems

---

# 8. How to Use MDA (Recommendations and Best Practices)

MDA defines a clear structure and well-defined responsibilities, but does not impose rigid technical restrictions.
It is not an architecture that "prevents you" from doing things, but one that suggests preferred paths.

This section describes how MDA is expected to be used in practice, and what decisions are recommended… though not mandatory.

---

## 8.1 The Recommended Path

The general idea is:

Controller → Use Case → Services → (Finder / Query / Repository)

Controllers do not contain business logic.
They delegate execution to a Use Case or a service.

Use Cases:
- coordinate services
- define the flow
- handle high-level decisions

From a Use Case you can:

- read data using Finders or Queries
- execute reusable logic using Services
- persist using Repositories

This flow maintains:

- traceability
- lower coupling
- clear code intent

---

## 8.2 Non-Strict Basic Rules

MDA does not technically block other accesses.

For example:

- You can call an Action directly from a Controller
- You can reuse an Action from different entry points
- You can execute a Query without going through a Service
- You can call a Finder directly from a Service

👉 Nothing prevents it.

The key difference is **intent**.

Recommendation:

- Direct access to Finders / Queries are exceptions
- If direct access starts to repeat:
  - it probably deserves its own Use Case
  - or the use case was not well modeled

MDA prioritizes clarity over technical enforcement.

---

## 8.3 Business Logic Placement

Business logic should live outside of controllers

Preferably within:

* Services
* Use Cases
* (or Domain logic if the project requires it)

MDA does not force the use of:
  * Aggregates
  * Domain Events
  * Value Objects

But it does not prohibit them.

If a project grows and needs more advanced concepts, MDA can:

  * coexist with them
  * serve as an organizational base
  * or be gradually extended

---

## 8.4 Pragmatism Over Purism

MDA is born from practice, not from theory.

If a rule:
  * unnecessarily complicates the code
  * delays a critical delivery
  * adds friction without clear benefit

Consciously breaking it is preferable to following it blindly.

The key is:
  * understanding why the recommendation exists
  * and what the cost of ignoring it is

---

## 8.5 MDA is a Starting Point

MDA does not attempt to:

  * cover all possible scenarios
  * compete with complete architectures like DDD
  * define an absolute truth

Its goal is:

  * offer a clear and usable base
  * reduce chaos in real projects
  * provide structure without over-engineering

If at some point you need more, you probably:

> already know better than MDA what is needed.

---

# 9. Conclusion

**Model Domain Architecture (MDA)** offers a perfect balance between organization, scalability, and simplicity. Its fundamental premise is that models are the central axis of design, and everything related to a model lives alongside it.

It is a practical, realistic, and highly usable architecture for teams working with Laravel, especially with multiple databases and domains that do not fit perfectly into traditional DDD.

---
