---
name: rovi
description: "Full-stack coding style, Clean Architecture patterns, and development philosophy. Use when writing code, designing systems, reviewing or refactoring code, planning architecture, structuring folders, creating interfaces, implementing repositories, building use cases, setting up state management, writing tests, handling errors, or explaining technical concepts. Enforces interface-first design, layered separation, strict typing, feature-based folder organization, centralized error handling, visual section separators in files, and direct no-analogy explanations. Applies to any language or framework in the stack: TypeScript, JavaScript, Python, Go, React, Next.js, Vue, NestJS, Fastify, Flask, FastAPI, Gin."
---

# ROVI — Style & Architecture Skill

## Overview

This skill defines a complete development philosophy: how to think before coding, how to structure projects, how to write and document code, and how to communicate technical concepts. It is framework-agnostic — the same principles apply regardless of the stack.

**Tech stack:** TypeScript, JavaScript, Python, Go | React, Next.js, Vue | NestJS, Fastify, Flask, FastAPI, Gin | Jotai, Redux, Zustand, Pinia

---

## Communication Rules

1. **No analogies.** Never compare code to real-world objects. Explain code with code.
2. **No unnecessary jargon.** Explain directly, as if telling a coworker what the code does.
3. **Explain the why.** If a library already does something internally, say so. Understanding internals prevents redundant or wrong decisions.
4. **Get to the point.** The first sentence must already answer the question.
5. **Research precisely.** Search for the specific thing being asked. Prioritize official docs and source code. No generic tutorials. If unclear, say so.
6. **Tone:** Direct, technical but accessible, opinionated with reasoning, practical — always land on concrete code.

---

## Thinking Process

Before writing any code, follow this sequence:

1. **Understand the problem.** What is needed? Edge cases? Constraints?
2. **Design the contracts.** Define interfaces first. Establish input/output types. Identify possible errors.
3. **Define the structure.** Which layer does each piece live in? What dependencies are needed? How do parts connect?
4. **Implement.** Start with domain (entities, interfaces), then use cases, then infrastructure and presentation.
5. **Test.** Manual testing first (Postman or browser). Then write automated tests. Verify edge cases from step 1.

---

## Architecture

### Clean Architecture (Layered)

```
domain/          → Entities, interfaces, business types. Zero external dependencies.
application/     → Use cases. Orchestrate the domain. Depend only on interfaces.
infrastructure/  → Concrete implementations: repositories, external APIs, DB.
presentation/    → Controllers, routes, UI. The outermost layer.
```

### Core Principles

- **Interfaces for everything that might change.** Swapping the database or a provider should only change the infrastructure layer.
- **Repository pattern mandatory** for data access.
- **Dependency injection.** Never instantiate dependencies inside a class. Always received via constructor or parameter.
- **Single responsibility:** if a function does more than 2 things, split it.
- **Purposeful abstractions:** every interface exists to enable swapping implementations or facilitate testing. No "just in case" interfaces.

### Patterns

Repository, Service, UseCase/Interactor, Ports & Adapters, Factory, Strategy, Observer/EventEmitter.

### Layer Implementation

Read `references/layer-examples.md` for full code examples of each layer: domain entities with self-validation and `fromPersistence`, repository interfaces, use cases with `execute`, concrete repositories, controllers as pure delegation, middleware as single-purpose functions, auxiliary services (JWT, encryption), and the `common/` folder pattern.

---

## Folder Structure

### Principles

- **Feature/module-based.** Each feature is an autonomous module. Never mix code from one module into another.
- **Global vs local.** If something is used by only one module, it lives inside that module. If shared, it goes up to the global level.
- **Core/Common is infrastructure.** DB config, server setup, cloud clients — separated from business modules.
- **One store per entity** (frontend). Never a mega-store.
- **File naming:** `camelCase` for stores and utilities, `PascalCase` for components and views.

### Frontend

```
src/
  assets/
  components/          → Global reusable components
  core/
    layout/
    store/             → Base store configuration
  features/
    [FeatureName]/
      components/      → Feature-specific components
      store/           → Feature-specific stores
      views/           → Feature pages
  utils/
```

### Backend

```
src/
  common/
    db/
    server/
  modules/
    [ModuleName]/
      domain/
        entities/
        interfaces/
        types/
        factory/
      application/
        use-cases/
        dto/
      infrastructure/
        repositories/
        services/
      presentation/
        controllers/
        middleware/
        docs/            → API documentation (Swagger/OpenAPI) separate from controllers
        exceptions/      → HTTP-specific exceptions for this layer
  utils/
    errors/
```

---

## Style Guide

### Code Rules

1. **Short functions.** Max 25-30 lines. If longer, it is doing too much — split it.
2. **One file, one responsibility.** Each file has a single clear purpose.
3. **Self-documenting code.** Names must be descriptive enough to read like prose.
4. **No single-line compression.** Clarity over brevity. Each declaration on its own line.

### Naming

- All code naming is in **English**: functions, variables, classes, interfaces, types, constants.
- All comments and JSDoc/TSDoc documentation are in **Spanish** by default, unless explicitly requested otherwise.

```typescript
interface UserRepository {
  findById(id: string): Promise<User>;
  save(user: User): Promise<void>;
}

class UserRepositoryPostgres implements UserRepository { }
class AuthService { }
class CreateUser { }
const activeUsers = await repository.findActive();
const MAX_SESSION_TIME = 3600;

type OrderStatus = 'pending' | 'processing' | 'completed' | 'cancelled';
```

### Typing

- TypeScript strict mode always. `strict: true`, non-negotiable.
- Never `any`. Use `unknown` and validate.
- Explicit types on function parameters and return types for public contracts.

### Visual Section Separators

Use block comment separators inside files with multiple logical sections:

```
/*================ */
/*   STATE          */
/*================ */

/*================ */
/*   COMPUTED       */
/*================ */

/*================ */
/*   ACTIONS        */
/*================ */
```

---

## Documentation & Comments

- **No emojis.** Ever. Comments are technical documentation.
- **JSDoc/TSDoc on every public function and class.** No exceptions.
- **Explain the "why", not the "what".** Code says what it does; comments explain why that decision was made.
- **Long comments are fine.** If a design decision needs 5 lines, write 5 lines.
- **Use `@example`** when the API is not obvious.
- Comments are written in **Spanish** (unless English is explicitly required).

Read `references/documentation-examples.md` for JSDoc patterns.

---

## Error Handling

Centralized typed errors in a `utils/errors` module. Every error has a code, message, and optional metadata.

```typescript
class AppError extends Error {
  constructor(
    public readonly code: string,
    public readonly message: string,
    public readonly httpStatus: number = 500,
    public readonly metadata?: Record<string, unknown>
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

class NotFoundError extends AppError {
  constructor(resource: string, identifier: string) {
    super('RESOURCE_NOT_FOUND', `${resource} not found: ${identifier}`, 404, { resource, identifier });
  }
}

class ValidationError extends AppError {
  constructor(field: string, detail: string) {
    super('VALIDATION_FAILED', `Validation error on ${field}: ${detail}`, 400);
  }
}

class DuplicateError extends AppError {
  constructor(resource: string, field: string) {
    super('RESOURCE_DUPLICATE', `${resource} already exists with that ${field}`, 409);
  }
}

class UnauthorizedError extends AppError {
  constructor(reason?: string) {
    super('UNAUTHORIZED', reason ?? 'Unauthorized operation', 403);
  }
}
```

---

## State Management (Frontend)

Consistent structure regardless of the state library:

- One store per entity. Never a mega-store.
- Always `isLoading` and `error` as state. Each store manages its own loading and error state.
- Try-catch on every async action. Always with `finally` to reset `isLoading`.
- Visually separated sections: state, computed/derived, actions.
- Optimistic local state updates when appropriate (push to array after create, filter after delete).

Read `references/store-examples.md` for a full store pattern.

---

## Testing

1. Implement the feature completely.
2. Test manually — endpoints with Postman or similar, UI in the browser.
3. Once it works, write automated tests.

- Unit tests for business logic (domain and use cases).
- Integration tests for repositories and endpoints.
- Mock the interfaces — another reason everything has an interface.

Read `references/testing-examples.md` for test patterns.

---

## Hard Rules

### Never

- Write `any` in TypeScript.
- Put emojis in comments.
- Create functions over 30 lines without splitting.
- Hardcode dependencies inside classes or functions.
- Leave code without an interface when there is a possibility of change.
- Put everything in one file.
- Use `console.log` as a debugging strategy in production.
- Ignore errors. Every error is caught, typed, and handled.
- Sacrifice clarity for brevity.
- Explain code with real-world analogies.

### Always

- Define interfaces before implementing.
- Document with detailed JSDoc/TSDoc.
- Type strictly. `strict: true`, no `any`, explicit types.
- Structure in layers. Domain → Application → Infrastructure → Presentation.
- Centralize errors in a dedicated module with clear types.
- Test. Manual first, automated after.
- Think about swappability. Changing an external dependency should touch one file ideally.
- Write naming in English, comments/documentation in Spanish.
- Organize by features with consistent folder structure.
- Use visual section separators inside files.
