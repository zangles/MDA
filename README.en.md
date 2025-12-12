<p align="center">
  <a href="README.md"><img src="https://img.shields.io/badge/Español-🇪🇸-red?style=for-the-badge" alt="Español" /></a>
  &nbsp;&nbsp;&nbsp;
  <a href="README.en.md"><img src="https://img.shields.io/badge/English-🇺🇸-blue?style=for-the-badge" alt="English" /></a>
</p>

<p align="center">
  <img src="logo.jpg" alt="Model Domain Architecture Logo" width="1024">
</p>

> **Important note**
>
> Model Domain Architecture (MDA) is a **pragmatic and flexible architectural approach**, inspired by principles from Domain-Driven Design and Clean Architecture, but **adapted to real-world Laravel projects**.
>
> Its goal is to serve as a **clear and practical starting point**, not as a rigid or exhaustive framework like DDD, Hexagonal, or Clean Architecture.  
> MDA defines a recommended organization of layers and responsibilities, **but it does not technically prevent direct access to internal components** (Finder, Query, Action, etc.).
>
> The recommended path is to always go through Services, and only in specific cases use direct access, with judgment and responsibility. This foundation can be expanded or adjusted according to the project’s needs.


## Model Domain Architecture (MDA)

The **Model Domain Architecture (MDA)** is a pragmatic architectural approach, inspired by Domain-Driven Design (DDD) and Clean Architecture principles, but specifically adapted to Laravel projects where the natural separation occurs by **models** rather than by complex conceptual domains.

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

In MDA, each system model functions as a “bounded domain.” Not a *bounded context* as in DDD, but a **functional area delimited by the model itself and its database**.

Each model contains:

* **Actions** (behaviors that modify the state)
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
  Models/
    Main/
      Order.php
      User.php
      Product.php
    Logistic/
      Delivery.php
```

> Note: Models are grouped by database when the application uses multiple connections. This is a valid exception because it responds to infrastructure needs, not domain needs.

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

Classes with simple read methods:

* `findById($id)`
* `findByUserId($userId)`
* `findByOrderId($orderId)`

All simple searches that do not require elaborate joins or complex logic.

They are grouped **by model**, in a single file per model:

```
OrderFinder.php
UserFinder.php
```

🔀 Possible Variations in Finder Organization

By default, MDA suggests grouping all simple read methods for a model inside a single file, for example:


* UserFinder.php
* OrderFinder.php

This keeps the directory structure lightweight and avoids creating dozens of tiny classes.

However, this is not the only valid approach.
If your team prefers a more granular structure —for example, one Finder class per query:

```
FindUserById.php
FindUsersByRole.php
FindOrdersByCustomer.php
```

This is also fully compatible with MDA.

The core idea is that:

Simple queries should be separated from services and business logic.

The internal organization should be consistent for your project.

Your team should be able to adjust or scale the structure without friction.

In other words:

>MDA defines where things live, but not exactly how they must look.
>Each project can choose its preferred level of granularity.

---

## 3.3 Queries

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

Queries are isolated because it is **normal** for them to grow significantly.

---

## 3.4 Repositories

Classes focused on **basic persistence**:

* `create()`
* `update()`
* `delete()`

They avoid the overload of having 50 specialized methods within the repository.

Complex queries are NOT mixed here; they go in Queries.
Simple searches are NOT mixed here; they go in Finders.

---

## 3.5 Actions

Domain modifier behaviors:

* update an order
* recalculate amounts
* create a client

Actions **use repositories** and never queries.

---

## 3.6 DTOs

Objects for transporting data, ordering requests and responses.

Separated by model.

---

## 3.7 Services

The model's **orchestrators**.

A service typically does:

* validate flow
* call finders
* use complex queries
* call actions
* transform results

Each Service has an interface for clarity and testability.

Although the interface can sometimes be omitted in simple projects, in MDA it is recommended because the Service becomes the main entry point of the module.

---

## 3.8 Composite Services

In addition to services that are directly associated with a specific model, **Composite Services** are also contemplated in MDA.

A Composite Service corresponds to a system use case that:

* **does not represent a single model**, but combines operations over multiple models,
* **adds cross-cutting business logic**,
* and **orchestrates calls to several associated services/queries/repositories**.

Typical examples of Composite Services might be:

* NotificationService (uses UserService, PaymentService, etc.)
* Settlements that combine multiple entities
* Complex ecosystem reports or synchronizations

### 📌 Where are they located?

Unlike models (like `Order`, `User`, etc.), Composite Services do not have an **associated physical model**.

However, to maintain the consistency of **Model Domain Architecture**, these services:

* are placed in the `app/Services/` folder along with the other modules
* may have their own set of DTOs, Actions, and Queries within the corresponding general folders, grouped by the use case name

For example:

```
app/
├── Services/
│ ├── Order/
│ ├── User/
│ └── Notification/ ← Servicio Compuesto
│   └── NotificationService.php
├── DTO/
│ ├── Order/
│ ├── User/
│ └── Notification/ ← DTO específico
│   └── NotificationDTO.php
├── Actions/
│ ├── Order/
│ ├── User/
│ └── Notification/ ← Acciones
│   ├── EjecutarNotification.php
│   └── ValidarNotification.php
├── Queries/
│ ├── Order/
│ ├── User/
│ └── Notification/ ← Consultas propias
```

### 🧾 Why is it consistent with MDA?

Although a Composite Service does not have an associated Eloquent model, it follows:

* the **same structural logic** as the model modules,
* the organization by **type of responsibility** (DTOs, Actions, Queries, Services),
* and respects that each piece is part of a **cohesive functional area** of the business.

Composite Services are conceptually similar to what is known in other architectures as:

* Domain Services
* Application Services ( in Clean Architecture)
* Cross-cutting use cases that require coordination of several models.

Explicitly integrating this type of service into MDA allows the structure to be kept **uniform, predictable, and scalable** without forcing everything to depend on a single model.

---

# 4. Relationship between Components

```
Controller → Service → (Finder / Query / Action)

Action → Repository
Finder → Eloquent (lecturas simples)
Query → SQL/Eloquent complejo
```

Dependencies are unidirectional, clear, and clean.

---

# 5. MDA Advantages

### ✓ Natural Scalability

As the project grows, only folders within the corresponding model grow.

### ✓ Immediate Navigation

Everything related to `Order` is in the same “model domain.”

### ✓ No Giant Super-Repositories

Specific queries are not mixed with basic CRUD operations.

### ✓ Aligned with the Reality of Many Laravel Projects

Many projects do not have well-defined business domains to apply full DDD.

### ✓ Compatible with Partial DDD

If a large domain appears one day (for example, “Finance”), it can group several models under one context.

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

```
* Queries/Order/getOrdenesConDescuento
```

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

Not ideal in:

* extremely small systems

---
# 8 How to Use MDA (Recommendations and Best Practices)

MDA defines a clear structure and well-defined responsibilities, but does not impose rigid technical restrictions.
It is not an architecture that "prevents you" from doing things, but one that suggests preferred paths.

This section describes how MDA is expected to be used in practice, and what decisions are recommended... though not mandatory.

---
## 8.1 The Recommended Path

The general idea is:

Controller → Service → (Finder / Query / Action / Repository)

Controllers should delegate all logic to a Service.

Services act as the entry point to a use case.

From a Service, you can:

* read data using Finders or Queries
* execute logic using Actions
* persist using Repositories

This flow maintains:

* traceability
* lower coupling
* clear code intent

---

## 8.2 Soft Rules, Not Hard Constraints

MDA does not technically block other accesses.

For example:

  * You can call a `Finder` directly from a controller.
  * You can execute a `Query` without going through a Service.
  * You can reuse an `Action` from different places.

👉 **Nothing prevents it**.

But the recommendation is to:

* do it only when there is a concrete reason
* understand that it is an exception, not the main pattern
* avoid letting it become a constant practice

If a direct access starts to be repeated, it probably:

* deserves its own Service
* or indicates that the use case was not well modeled

---

## 8.3 Business Logic Placement

Business logic should live outside of controllers.

Preferably within:

* Services
* Actions
* (or Domain logic if the project requires it)

MDA does not enforce the use of:
* Aggregates
* Domain Events
* Value Objects

But it does not prohibit them.

If a project grows and requires more advanced concepts, MDA can:

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

The key is to:
* understand why the recommendation exists
* and what the cost of ignoring it is

---

## 8.5 MDA is a Starting Point

MDA does not attempt to:

* cover all possible scenarios
* compete with complete architectures like DDD
* define an absolute truth

Its goal is to:

* offer a clear and usable base
* reduce chaos in real projects
* provide structure without over-engineering

If at some point you need more, you probably:

> already know better than MDA what is needed.

---

# 9. Conclusion

The **Model Domain Architecture (MDA)** offers a perfect balance between organization, scalability, and simplicity. Its fundamental premise is that models are the central axis of the design, and everything related to a model lives alongside it.


It is a practical, realistic, and highly usable architecture for teams working with Laravel, especially with multiple databases and domains that do not fit perfectly into traditional DDD.

