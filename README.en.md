<p align="center">
  <a href="README.md"><img src="https://flagcdn.com/40x30/es.png" alt="Español" width="40" /></a>
  &nbsp;&nbsp;&nbsp;
  <a href="README.en.md"><img src="https://flagcdn.com/40x30/us.png" alt="English" width="40" /></a>
</p>

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

> **Important Note**
>
> Model Domain Architecture (MDA) is not intended to be a formal architecture or a rigid framework like Domain-Driven Design, Hexagonal, or Clean Architecture.
>
> Its goal is to be **a clear, simple, and practical starting point** for organizing Laravel projects, without imposing a heavy structure.
>
> MDA is a **flexible base**, not a strict rule: every team can expand it, modify it, or combine it with other ideas according to their needs.
>
> For example, MDA currently does not include:
>
> * **Formal Domain Events**, where entities emit internal domain events (e.g., `Order::created()` → fires `OrderPlacedEvent` before persistence). You could add this by creating a `Domain/Events` folder and a mechanism to dispatch them, if your project requires it.
> * **Value Objects** to represent immutable concepts (like Money, Email, PhoneNumber). Today MDA uses simple DTOs, but you can extend it if the logic justifies it.
> * **Aggregates and aggregate roots**, useful for stricter domains or those rich in invariants.
> * **Infrastructure/Domain separation** as in hexagonal architectures.
>
> None of these things contradict MDA: they are simply **not included by default** to keep the architecture simple. If your project requires more formality or robustness, you can add these layers without breaking the general approach.

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

This prevents the creation of dozens of unnecessary small classes.

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

# 8. Conclusion

The **Model Domain Architecture (MDA)** offers a perfect balance between organization, scalability, and simplicity. Its fundamental premise is that models are the central axis of the design, and everything related to a model lives alongside it.

It is a practical, realistic, and highly usable architecture for teams working with Laravel, especially with multiple databases and domains that do not fit perfectly into traditional DDD.