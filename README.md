<p align="center">
  <a href="README.md"><img src="https://img.shields.io/badge/Español-🇪🇸-red?style=for-the-badge" alt="Español" /></a>
  &nbsp;&nbsp;&nbsp;
  <a href="README.en.md"><img src="https://img.shields.io/badge/English-🇺🇸-blue?style=for-the-badge" alt="English" /></a>
</p>

<p align="center">
  <img src="logo.png" alt="Model Domain Architecture Logo" width="1360">
</p>

# Model Domain Architecture (MDA)

La **Model Domain Architecture (MDA)** es un enfoque arquitectónico pragmático, inspirado en principios de Domain-Driven Design (DDD) y Arquitectura Limpia, pero adaptado específicamente a proyectos Laravel donde la separación natural se da por **modelos** más que por dominios conceptuales complejos.

MDA organiza el proyecto alrededor de los *modelos centrales* del sistema (por ejemplo: `User`, `Order`, `Product`, etc.), manteniendo una estructura clara, escalable y fácil de navegar incluso en aplicaciones con múltiples bases de datos.

> ⚠️ Nota importante
> MDA propone una forma organizada de estructurar el código, pero no impone restricciones técnicas rígidas.
> Define caminos recomendados, no reglas absolutas.
> El criterio del desarrollador sigue siendo parte central del diseño.

---

## 🎯 Objetivo

Proveer una arquitectura:

* **Escalable**: que soporte crecimiento sin producir clases gigantes o repositorios inmanejables.
* **Organizada por modelo**: cada modelo tiene su propio mini-módulo con todos sus comportamientos.
* **Predicible**: cualquier desarrollador sabe exactamente dónde buscar la lógica relacionada a un modelo.
* **Práctica**: evita el sobre-engineering innecesario que a veces surge con DDD completo.

> **Nota Importante**
>
> Model Domain Architecture (MDA) no pretende ser una arquitectura formal ni un marco rígido como Domain-Driven Design, Hexagonal o Clean Architecture.
> 
> Su objetivo es ser **un punto de partida claro, simple y práctico** para organizar proyectos Laravel, sin imponer una estructura pesada.
>
> MDA es una **base flexible**, no una regla estricta: cada equipo puede ampliarlo, modificarlo o combinarlo con otras ideas según sus necesidades.
>
> Por ejemplo, MDA no contempla actualmente:
>
> - **Domain Events formales**, donde las entidades emiten eventos internos del dominio (por ejemplo: `Order::created()` → dispara `OrderPlacedEvent` antes de persistirse).  
>   Podrías agregar esto creando una carpeta `Domain/Events` y un mecanismo para despacharlos, si tu proyecto lo necesita.  
>
> - **Value Objects** para representar conceptos inmutables (como Money, Email, PhoneNumber).  
>   Hoy MDA usa DTOs simples, pero podés extenderlo si la lógica lo justifica.
>
> - **Aggregates y aggregate roots**, útiles para dominios más estrictos o ricos en invariantes.
>
> - **Separación infraestructura/dominio** como en arquitecturas hexagonales.
>
> Ninguna de estas cosas contradice MDA: simplemente **no están incluidas por defecto** para mantener la arquitectura simple.
> Si tu proyecto necesita más formalidad o robustez, podés agregar estas capas sin romper el enfoque general.

---

# 1. Filosofía general

En MDA cada modelo del sistema funciona como un "dominio acotado". No un *bounded context* como en DDD, sino un **área funcional delimitada por el propio modelo y su base de datos**.

Cada modelo contiene:

* **Actions** (comportamientos que modifican el estado)
* **Finders** (consultas simples)
* **Queries** (consultas complejas)
* **DTOs**
* **Repositories**
* **Services** (el orquestador principal)

Esto crea pequeñas islas de responsabilidad clara.

---

# 2. Estructura de carpetas

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
> Nota:
> Models se agrupan por base de datos cuando la aplicación utiliza múltiples conexiones.
> Esta es una excepción válida porque responde a necesidades de infraestructura, no de dominio.

---

# 3. Descripción de cada componente

## 3.1 Models

Eloquent Models divididos por base de datos.

Esto es una excepción dentro de la arquitectura porque el modelo **refleja la estructura física**.

Ventajas:

* Facilita ver qué modelos pertenecen a qué base.
* Evita caos cuando existen múltiples conexiones.

---

## 3.2 Finders

Clases con métodos de lecturas simples:

* `findById($id)`
* `findByUserId($userId)`
* `findByOrderId($orderId)`

Todas las búsquedas simples que no requieren joins elaborados o lógica compleja.

Se agrupan **por modelo**, en un único archivo por modelo:

```
OrderFinder.php
UserFinder.php
```

🔀 Variaciones posibles en la organización de Finders

MDA propone, por defecto, agrupar todos los métodos de lectura simples de un modelo en un único archivo, por ejemplo:

* UserFinder.php
* OrderFinder.php

Esto mantiene el árbol de directorios liviano y evita crear decenas de clases pequeñas.

Sin embargo, esta no es la única forma válida de organizar los Finders.
Si tu equipo prefiere una estructura más granular —por ejemplo, una clase Finder por consulta:

```
FindUserById.php
FindUsersByRole.php
FindOrdersByCustomer.php
```

También es totalmente compatible con MDA.

La idea central es que:

Las consultas simples estén separadas del servicio y la lógica de negocio.

La organización interna sea coherente para tu proyecto.

El equipo pueda modificar o escalar la estructura sin fricción.

En otras palabras:

>MDA define dónde viven las cosas, pero no cómo deben verse exactamente.
>Cada proyecto puede ajustar el nivel de granularidad que prefiera.

---

## 3.3 Queries

Consultas SQL/Eloquent complejas:

* joins múltiples
* filtros avanzados
* agregaciones
* subconsultas
* lógica de lectura específica

Se agrupan por **carpetas de modelo**, por ejemplo:

```
Queries/Order/OrderQueries.php
Queries/User/UserQueries.php
```

Los queries están aislados porque **es normal** que crezcan mucho.

---

## 3.4 Repositories

Clases enfocadas en **persistencia básica**:

* `create()`
* `update()`
* `delete()`

Evitan la sobrecarga de tener 50 métodos especializados dentro del repositorio.

Las consultas complejas NO se mezclan aquí; van en Queries.

Las búsquedas simples NO se mezclan aquí; van en Finders.

---

## 3.5 Actions

Comportamientos modificadores del dominio:

* actualizar una orden
* recalcular montos
* crear un cliente

Las Actions **usan repositorios** y nunca queries.

---

## 3.6 DTOs

Objetos para transportar datos, ordenar requests y respuestas.

Separados por modelo.

---

## 3.7 Services

Los **orquestadores** del modelo.

Un service normalmente hace:

* validar flujo
* llamar a finders
* usar queries complejos
* llamar a actions
* transformar resultados

Cada Service tiene una interfaz por claridad y testeabilidad.

Aunque algunas veces la interfaz puede omitirse en proyectos simples, en MDA es recomendable porque el Service se vuelve el punto de entrada principal del módulo.

---

## 3.8 Servicios Compuestos

Además de los servicios que están directamente asociados a un modelo específico, en MDA también se contemplan los **Servicios Compuestos**.

Un Servicio Compuesto corresponde a un caso de uso del sistema que:

- **no representa un único modelo**, sino que combina operaciones sobre múltiples modelos,
- **agrega lógica de negocio transversal**,
- y **orquesta llamadas a varios servicios/consultas/repositorios asociados**.

Ejemplos típicos de Servicios Compuestos pueden ser:

- NotificacionesService (usa UserService, PaymentService, etc.)
- Liquidaciones que combinan múltiples entidades
- Reportes o sincronizaciones complejas del ecosistema

### 📌 ¿Dónde se ubican?

A diferencia de los modelos (como `Order`, `User`, etc.), los Servicios Compuestos no tienen un **modelo físico asociado**.  
Sin embargo, para mantener la consistencia de **Model Domain Architecture**, estos servicios:

- se colocan en la carpeta `app/Services/` junto a los demás módulos
- pueden tener su propio conjunto de DTOs, Actions y Queries
  dentro de las carpetas generales correspondientes, agrupados por el nombre del caso de uso

Por ejemplo:

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

### 🧾 ¿Por qué es consistente con MDA?

Aunque un Servicio Compuesto no tiene un modelo Eloquent asociado, sigue:

- la **misma lógica estructural** que los módulos por modelo,
- la organización por **tipo de responsabilidad** (DTOs, Actions, Queries, Services),
- y respeta que cada pieza forme parte de un **área funcional cohesionada** del negocio.

Los Servicios Compuestos son conceptualmente similares a lo que en otras arquitecturas se conoce como:

- Domain Services  
- Application Services (en Clean Architecture)  
- Casos de uso transversales que requieren coordinación de varios modelos.

Integrar este tipo de servicios de forma explícita en MDA permite mantener la estructura **uniforme, predecible y escalable** sin forzar que todo dependa de un único modelo.

---

# 4. Relación entre componentes

```
Controller → Service → (Finder / Query / Action)

Action → Repository
Finder → Eloquent (lecturas simples)
Query → SQL/Eloquent complejo
```

Las dependencias son unidireccionales, claras y limpias.

---

# 5. Ventajas de MDA

### ✓ Escalabilidad natural

A medida que crece el proyecto, solo crecen carpetas dentro del modelo correspondiente.

### ✓ Navegación inmediata

Todo lo relacionado a `Order` está en un mismo "dominio de modelo".

### ✓ Sin super-repositorios gigantes

Las consultas específicas no se mezclan con las CRUD básicas.

### ✓ Alineado a la realidad de muchos proyectos Laravel

Muchos proyectos no tienen dominios de negocio bien definidos para aplicar DDD completo.

### ✓ Compatible con DDD parcial

Si algún día aparece un dominio grande (por ejemplo, "Finanzas"), puede agrupar varios modelos bajo un contexto.

---

# 6. Ejemplo resumido de flujo

Controller de Ordenes:

```
$order = $orderService->findById($id);
```

Service:

```
return $this->finder->findById($id);
```

Consulta compleja:

```
$orders = $orderService->getOrdenesConDescuento();
```

Service llama:

* Queries/Order/getOrdenesConDescuento

Actualización:

```
$orderService->actualizarEstado($id, $dto);
```

Service → Action → Repository

---

# 7. Cuándo usar MDA

Ideal para proyectos:

* Laravel
* con múltiples modelos
* con consultas muy diversas
* con necesidad de orden y escalabilidad
* donde DDD completo es demasiado

No tan ideal en:

* sistemas extremadamente pequeños

---

# 8 Como usar MDA (Recomendaciones y buenas prácticas)

MDA define una estructura clara y responsabilidades bien delimitadas, pero no impone restricciones técnicas rígidas.
No es una arquitectura que “te impide” hacer cosas, sino una que te sugiere caminos preferidos.

Esta sección describe cómo se espera usar MDA en la práctica, y qué decisiones son recomendadas… aunque no obligatorias.

---
## 8.1 El camino recomendado

La idea general es:

```
Controller → Service → (Finder / Query / Action / Repository)
```

Los controllers deberían delegar toda la lógica a un Service.

Los Services actúan como punto de entrada a un caso de uso.

Desde un Service se puede:

  * leer datos mediante Finders o Queries
  * ejecutar lógica mediante Actions
  * persistir mediante Repositories

Este flujo mantiene:

  * trazabilidad
  * menor acoplamiento
  * una intención clara del código

---

## 8.2 Reglas basicas no estrictas

MDA no bloquea técnicamente otros accesos.

Por ejemplo:

  * Podés llamar a un `Finder` directamente desde un controller.
  * Podés ejecutar una `Query` sin pasar por un Service.
  * Podés reutilizar una `Action` desde distintos lugares.

👉 **Nada lo impide**.

Pero la recomendación es:

  * hacerlo solo cuando haya una razón concreta
  * entender que es una excepción, no el patrón principal
  * evitar que se vuelva una práctica constante

Si un acceso directo empieza a repetirse, probablemente:

  * merece su propio Service
  * o indica que el caso de uso no estaba bien modelado

---

## 8.3 Ubicacion de la lógica de negocio

La lógica de negocio debería vivir fuera de controllers

Preferentemente dentro de:

* Services
* Actions
* (o Domain logic si el proyecto lo requiere)

MDA no fuerza el uso de:
  * Aggregates
  * Domain Events
  * Value Objects

Pero no los prohíbe.

Si un proyecto crece y necesita conceptos más avanzados, MDA puede:

  * convivir con ellos
  * servir como base organizacional
  * o ser extendido gradualmente

---
## 8.4 Pragmatismo sobre purismo

MDA nace desde la práctica, no desde la teoría.

Si una regla:
  * complica innecesariamente el código
  * retrasa una entrega crítica
  * agrega fricción sin beneficio claro

Romperla conscientemente es preferible a seguirla a ciegas.

La clave es:
  * entender por qué existe la recomendación
  * y qué costo tiene ignorarla

---

## 8.5 MDA es un punto de partida

MDA no intenta:

  * cubrir todos los escenarios posibles
  * competir con arquitecturas completas como DDD
  * definir una verdad absoluta

Su objetivo es:

  * ofrecer una base clara y usable
  * reducir el caos en proyectos reales
  * dar estructura sin sobreingeniería

Si en algún punto necesitás más, probablemente:

> ya sabés mejor que MDA qué es lo que hace falta.

---

# 9. Conclusión

La **Model Domain Architecture (MDA)** ofrece un balance perfecto entre organización, escalabilidad y simplicidad. Su premisa fundamental es que los modelos son el eje central del diseño, y todo lo relacionado a un modelo vive junto a él.

Es una arquitectura práctica, realista y altamente usable en equipos que trabajan con Laravel, especialmente con múltiples bases de datos y dominios que no encajan perfectamente en DDD tradicional.

---


