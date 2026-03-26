---
name: rovi-architecture
description: "Clean Architecture patterns, layered separation, dependency injection, repository pattern, folder structure. Auto-loads when designing systems, creating modules, structuring projects, implementing repositories, use cases, or controllers."
---

# ROVI — Architecture

## Clean Architecture (Layered)

These are dependency layers, not folder names. All layers coexist inside each entity's folder.

```
domain layer          > Entities, interfaces, business types. Zero external dependencies.
application layer     > Use cases. Orchestrate the domain. Depend only on interfaces.
infrastructure layer  > Concrete implementations: repositories, external APIs, DB.
presentation layer    > Controllers, routes, UI. The outermost layer.
```

## Core Principles

- **Interfaces for everything that might change.** Swapping the database or a provider should only change the infrastructure layer.
- **Repository pattern mandatory** for data access.
- **Dependency injection.** Never instantiate dependencies inside a class. Always received via constructor or parameter.
- **Single responsibility:** if a function does more than 2 things, split it.
- **Purposeful abstractions:** every interface exists to enable swapping implementations or facilitate testing. No "just in case" interfaces.

## Patterns

Repository, Service, UseCase/Interactor, Ports & Adapters, Factory, Strategy, Observer/EventEmitter.

---

## Folder Structure

### Principles

- **Feature-based organization.** Each feature is autonomous. Never mix code from one feature into another.
- **Global vs local.** If something is used by only one feature, it lives inside that feature. If shared, it goes up to the global level.
- **Shared infrastructure separated.** DB config, server setup, cloud clients — separated from business features.
- **One store per entity** (frontend). Never a mega-store.
- **File naming:** `camelCase` for stores and utilities, `PascalCase` for components and views.

### Important

The examples below are **references**, not rigid templates. Folder names, nesting depth, and grouping depend on the framework and the project. Always confirm the structure before creating it. What matters is the layered separation principle, not the specific names.

### Frontend (reference)

```
src/
  assets/
  components/               > Global reusable components
  core/
    layout/
    store/                  > Base store configuration
  [feature-group]/
    [FeatureName]/
      components/           > Feature-specific components
      store/                > Feature-specific stores
      views/                > Feature pages
  utils/
```

### Backend (reference)

Everything related to an entity lives inside that entity's folder. You work on one entity, you stay in one place.

```
src/
  [shared-config]/
    db/
    server/
  modules/
    [EntityName]/
      entities/
      interfaces/
      types/
      use-cases/
      dto/
      repositories/
      services/
      controllers/
      middleware/
  utils/
    errors/
```

Names in `[brackets]` are placeholders — they change per project and framework. The key rule: everything for one entity is colocated, no jumping between folders to find related code.

---

## Layer Examples

Read `${CLAUDE_SKILL_DIR}/references/layer-examples.md` for concrete code examples of each layer: domain entities, repository interfaces, use cases, concrete repositories, factories, controllers, middleware, and auxiliary services.
