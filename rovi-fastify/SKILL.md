---
name: rovi-fastify
description: "Fastify backend patterns: entities folder structure, controller with inline DI, service with constructor injection, repository pattern, route registration. Auto-loads when working with Fastify, creating API endpoints, or building backend with Fastify."
---

# ROVI — Fastify

## Project Structure

Use `entities/` instead of `modules/`. Each entity folder is self-contained:

```
src/
  config/
    db/
    server/
  entities/
    [EntityName]/
      controller/
        entityName.controller.ts
      routes/
        entityName.routes.ts
      service/
        entityName.service.ts
      repository/
        entityName.repository.ts
      types/
        entityName.interface.ts
        entityName.type.ts
  auth/
    middleware/
      jwt.middleware.ts
  utils/
    errors/
```

---

## Controller Pattern

**Instantiate dependencies directly in the controller file.** No separate DI container file. The controller is where you see what depends on what.

```typescript
// Pattern: import repo + service, instantiate inline, export controller function
import { FastifyInstance } from "fastify";
import { EntityService } from "../service/entity.service";
import { EntityRepository } from "../repository/entity.repository";

const entityRepository = new EntityRepository();
const entityService = new EntityService(entityRepository);

export async function entityController(fastify: FastifyInstance) {
  fastify.get("/", async (request, reply) => {
    const items = await entityService.findAll();
    return reply.status(200).send({ data: items });
  });

  fastify.post("/", async (request, reply) => {
    const result = await entityService.create(request.body as CreateEntityData);
    return reply.status(201).send({ data: result });
  });
}
```

---

## Service Pattern

Services receive dependencies via **constructor injection**. No classes passed as method parameters.

```typescript
// Pattern: constructor receives interface, methods orchestrate business logic
export class EntityService {
  constructor(
    private readonly entityRepository: IEntityRepository
  ) {}

  async findAll(): Promise<Entity[]> {
    return this.entityRepository.findAll();
  }

  async create(data: CreateEntityData): Promise<Entity> {
    // Business validation lives here
    return this.entityRepository.create(data);
  }
}
```

---

## Repository Pattern

Interface in `types/`, implementation in `repository/`.

```typescript
// Pattern: interface defines contract, class implements it
// types/entity.interface.ts
export interface IEntityRepository {
  findAll(): Promise<Entity[]>;
  findById(id: number): Promise<Entity>;
  create(data: CreateEntityData): Promise<Entity>;
  delete(id: number): Promise<Entity>;
}

// repository/entity.repository.ts
export class EntityRepository implements IEntityRepository {
  // Implementation depends on the DB (Prisma, Drizzle, raw SQL, etc.)
}
```

---

## Route Registration

```typescript
// Pattern: register controller with prefix
import { FastifyInstance } from "fastify";
import { entityController } from "../controller/entity.controller";

export async function entityRoutes(fastify: FastifyInstance) {
  fastify.register(entityController, { prefix: "/entities" });
}
```

---

## Swagger / OpenAPI

**Swagger first.** Every Fastify backend must expose an OpenAPI spec. The frontend depends on it to auto-generate its entire API layer with Orval.

Use `@fastify/swagger` + `@fastify/swagger-ui`. Document every route with schemas. The OpenAPI spec is the contract between backend and frontend.

---

## Rules

- **Swagger first.** Always expose OpenAPI spec. The frontend consumes it with Orval.
- **`entities/` not `modules/`.** Each folder is one entity.
- **Inline DI in controllers.** Instantiate repository → service right there. No separate DI file.
- **Constructor injection in services.** Dependencies via constructor, never as method params.
- **Interface + implementation separated** in `types/` (interface) and `repository/` or `service/` (class).
- **No `fromPersistence` pattern.** Do not add static reconstruction methods to entities unless explicitly requested.
