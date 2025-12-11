<p align="center">
  <img src="logo.png" alt="Model Domain Architecture Logo" width="1360">
</p>

# Model Domain Architecture (MDA)

La **Model Domain Architecture (MDA)** es un enfoque arquitectónico pragmático, inspirado en principios de Domain-Driven Design (DDD) y Arquitectura Limpia, pero adaptado específicamente a proyectos Laravel donde la separación natural se da por **modelos** más que por dominios conceptuales complejos.

MDA organiza el proyecto alrededor de los *modelos centrales* del sistema (por ejemplo: `Credito`, `Cliente`, `CreditoDetalle`, `Item`, `Acuerdo`, etc.), manteniendo una estructura clara, escalable y fácil de navegar incluso en aplicaciones con múltiples bases de datos.

---

## 🎯 Objetivo

Proveer una arquitectura:

* **Escalable**: que soporte crecimiento sin producir clases gigantes o repositorios inmanejables.
* **Organizada por modelo**: cada modelo tiene su propio mini-módulo con todos sus comportamientos.
* **Predicible**: cualquier desarrollador sabe exactamente dónde buscar la lógica relacionada a un modelo.
* **Práctica**: evita el sobre-engineering innecesario que a veces surge con DDD completo.

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
    Credito/
    Cliente/
    CreditoDetalle/
  DTO/
    Credito/
    Cliente/
    CreditoDetalle/
  Finders/
    CreditoFinder.php
    ClienteFinder.php
    CreditoDetalleFinder.php
  Queries/
    Credito/
    Cliente/
    CreditoDetalle/
  Repositories/
    CreditoRepository.php
    ClienteRepository.php
    CreditoDetalleRepository.php
  Services/
    Interfaces/
      ICreditoService.php
      IClienteService.php
    CreditoService.php
    ClienteService.php
  Models/
    maestro/
      Credito.php
      Cliente.php
      CreditoDetalle.php
    gestion/
      Acuerdo.php
```

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
* `findByCliente($clienteId)`
* `findByCredito($creditoId)`

Todas las búsquedas simples que no requieren joins elaborados o lógica compleja.

Se agrupan **por modelo**, en un único archivo por modelo:

```
CreditoFinder.php
ClienteFinder.php
```

Esto evita crear docenas de pequeñas clases innecesarias.

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
Queries/Credito/GetCreditosParaRefinanciacion.php
Queries/Credito/GetCreditosVencidos.php
```

Los queries están aislados porque **es normal** que crezcan mucho.

---

## 3.4 Repositories

Clases enfocadas en **persistencia básica**:

* `create()`
* `update()`
* `delete()`
* `save()`

Evitan la sobrecarga de tener 50 métodos especializados dentro del repositorio.

---

## 3.5 Actions

Comportamientos modificadores del dominio:

* actualizar un crédito
* recalcular montos
* cerrar un acuerdo

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

Todo lo relacionado a `Credito` está en un mismo "dominio de modelo".

### ✓ Sin super-repositorios gigantes

Las consultas específicas no se mezclan con las CRUD básicas.

### ✓ Alineado a la realidad de muchos proyectos Laravel

Muchos proyectos no tienen dominios de negocio bien definidos para aplicar DDD completo.

### ✓ Compatible con DDD parcial

Si algún día aparece un dominio grande (por ejemplo, "Finanzas"), puede agrupar varios modelos bajo un contexto.

---

# 6. Ejemplo resumido de flujo

Controller de crédito:

```
$credito = $creditoService->findById($id);
```

Service:

```
return $this->finder->findById($id);
```

Consulta compleja:

```
$items = $creditoService->getItemsPagadosConDescuento($id);
```

Service llama:

* Queries/Credito/GetItemsPagadosConDescuento

Actualización:

```
$creditoService->actualizarEstado($id, $dto);
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

# 8. Conclusión

La **Model Domain Architecture (MDA)** ofrece un balance perfecto entre organización, escalabilidad y simplicidad. Su premisa fundamental es que los modelos son el eje central del diseño, y todo lo relacionado a un modelo vive junto a él.

Es una arquitectura práctica, realista y altamente usable en equipos que trabajan con Laravel, especialmente con múltiples bases de datos y dominios que no encajan perfectamente en DDD tradicional.

---


