# Proyecto de repaso: Laravel de CRUD básico a arquitectura de software

## Objetivo
Repasar Laravel desde cero (instalación, rutas, controladores, vistas, modelos —
conocimiento previo del usuario) hasta dominar patrones de arquitectura usados en
proyectos serios: Service Providers, Services con inyección de dependencias,
Repository Pattern, Policies, Events/Listeners, Jobs/Queues, testing, etc.

## Cómo trabajamos (acordado con el usuario)
- **Modo guiado paso a paso**: Claude explica qué hacer y por qué; el usuario
  escribe el código/comandos; Claude revisa y corrige. NO escribir el código
  del CRUD básico por el usuario — eso es justo lo que quiere repasar con las manos.
- Para temas nuevos/avanzados (a partir de la Fase 4), Claude puede construir el
  ejemplo con más participación y explicar la decisión, pero siempre explicando
  el "por qué" antes del "cómo".
- **Estilo de explicación**: el usuario pidió explícitamente analogías de la
  vida real (construir una casa, un guardia con lista VIP, etc.) en vez de
  explicaciones técnicas densas — dice que tiene TDA y que las explicaciones
  aburridas lo pierden. Por defecto, explicar conceptos nuevos con una
  analogía concreta ANTES o junto con la explicación técnica, no como anexo
  opcional. Mantener el tono conversacional y cercano.
- Stack de vistas: **Blade + Tailwind**.
- Base de datos de desarrollo: SQLite (simplicidad, sin depender de un servidor
  MySQL local) salvo que se decida lo contrario.

## Estado actual
- [x] PHP 8.3.6, Composer 2.10.2 instalados en el entorno (Ubuntu 24.04 WSL2).
- [x] Proyecto Laravel creado en este directorio: **Laravel Framework 13.19.0**.
- [x] Entidad para el CRUD: **Producto** — negocio tipo feria/almacén que vende
      frutas, verduras y productos varios. Campos previstos: `nombre`,
      `descripcion`, `precio` (CLP), `stock`, `categoria`.

## Ruta de aprendizaje (fases)

### Fase 0 — Entorno
- Instalar PHP + extensiones, Composer, instalador de Laravel.
- Crear el proyecto (`laravel new` o `composer create-project`).
- Entender `.env`, `artisan`, estructura de carpetas (`app/`, `routes/`, `resources/`,
  `database/`).

### Fase 1 — Repaso de fundamentos (CRUD básico, lo que el usuario ya conoce)
- Rutas (`web.php`), resource routes.
- Migraciones y modelos Eloquent simples.
- Controladores resource (`--resource`), vistas Blade, layouts con `@extends`/`@yield`/`@section`.
- CRUD completo de una entidad (ej. "Tareas" o "Productos") a mano.

### Fase 2 — Validación
- `$request->validate()` vs Form Requests (`php artisan make:request`).
- Reglas custom de validación.

### Fase 3 — Capa de Servicios (Service Layer)
- Por qué sacar lógica de negocio del controlador.
- Clases `Service` simples, inyección de dependencias vía constructor.
- Diferencia entre "fat controllers" y controladores delgados.

### Fase 4 — Repository Pattern + Contratos (interfaces)
- Interfaces vs implementaciones.
- Por qué (y cuándo) desacoplar Eloquent detrás de un repositorio.
- Binding de interfaz → implementación en el Service Container.

### Fase 5 — Service Providers
- Qué son, ciclo de vida (`register` vs `boot`).
- Crear un Service Provider custom para registrar bindings.
- Singleton vs bind.

### Fase 6 — Eloquent avanzado
- Relaciones (hasMany, belongsTo, belongsToMany, polimórficas).
- Eager loading (evitar N+1), scopes, accessors/mutators (Attribute casting).

### Fase 7 — API REST
- API Resources (`JsonResource`), rutas `api.php`, versionado básico.

### Fase 8 — Autenticación y Autorización
- Breeze/Fortify (auth básica), Policies, Gates.
- Laravel Sanctum para API tokens.

### Fase 9 — Eventos, Listeners y Observers
- Desacoplar efectos secundarios (ej. enviar email al crear un pedido).

### Fase 10 — Jobs y Colas
- Trabajos asíncronos, `ShouldQueue`, drivers de cola.

### Fase 11 — Testing
- Pest o PHPUnit, Feature tests vs Unit tests, factories/seeders.

### Fase 12 — Patrones adicionales y buenas prácticas
- Action classes (single-purpose), DTOs, Value Objects.
- SOLID aplicado a Laravel, cuándo NO sobre-arquitecturar.

### Fase 13 — Extra (según interés)
- Middleware personalizado, caching, Livewire, Docker/deploy.

## Convenciones de este repo
- (se irán documentando decisiones reales conforme avancemos, ej. nombre del
  proyecto, entidad elegida para el CRUD, estructura de carpetas custom, etc.)
