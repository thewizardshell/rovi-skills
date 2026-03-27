---
name: rovi-nestjs
description: "NestJS backend patterns: module structure, controller with DI, service with constructor injection, repository pattern, decorators. Auto-loads when working with NestJS, creating modules, or building backend with NestJS."
---

# ROVI — NestJS

## Philosophy

Same architecture as Fastify (see rovi-fastify) but using NestJS's built-in DI system, decorators, and module structure. The principles are identical: interface-first, constructor injection, repository pattern, entity-colocated folders.

---

## Project Structure

```
src/
  config/
    database/
    env/
  modules/
    [EntityName]/
      [entityName].controller.ts
      [entityName].service.ts
      [entityName].module.ts
      repository/
        [entityName].repository.ts
      types/
        [entityName].interface.ts      > Interface defining the entity contract
        [entityName].entity.ts         > Class implementing the interface with getters/setters
        [entityName].dto.ts
  auth/
    guards/
    strategies/
    decorators/
  utils/
    errors/
  app.module.ts
  main.ts
```

---

## Controller

NestJS handles DI via decorators — no manual instantiation needed. But the same idea: you see the dependencies right there.

```typescript
// Pattern: @Controller with injected service, typed DTOs
@Controller("entities")
export class EntityController {
  constructor(private readonly entityService: EntityService) {}

  @Get()
  @UseGuards(JwtAuthGuard)
  async findAll() {
    return this.entityService.findAll();
  }

  @Post()
  async create(@Body() data: CreateEntityDto) {
    return this.entityService.create(data);
  }
}
```

---

## Service

```typescript
// Pattern: @Injectable with @Inject for interface token
@Injectable()
export class EntityService {
  constructor(
    @Inject("IEntityRepository")
    private readonly entityRepository: IEntityRepository
  ) {}

  async findAll(): Promise<Entity[]> {
    return this.entityRepository.findAll();
  }

  async create(data: CreateEntityDto): Promise<Entity> {
    // Business validation lives here
    return this.entityRepository.create(data);
  }
}
```

---

## Module (DI wiring)

```typescript
// Pattern: wire interface token to concrete class
@Module({
  controllers: [EntityController],
  providers: [
    EntityService,
    {
      provide: "IEntityRepository",
      useClass: EntityRepositoryPrisma,
    },
  ],
  exports: [EntityService],
})
export class EntityModule {}
```

---

## Entity Types

`types/` always contains: the **interface** defining the entity contract, a **class** implementing it with getters and setters, and DTOs.

```typescript
// Pattern: interface defines the entity shape
// types/entity.interface.ts
export interface IEntity {
  id: number;
  name: string;
  description: string;
  createdAt: Date;
  updatedAt: Date;
}

// Pattern: class implements interface with constructor, getters, and setters
// types/entity.entity.ts
import type { IEntity } from "./entity.interface";

export class Entity implements IEntity {
  constructor(
    private _id: number,
    private _name: string,
    private _description: string,
    private _createdAt: Date,
    private _updatedAt: Date,
  ) {}

  get id(): number { return this._id; }
  get name(): string { return this._name; }
  get description(): string { return this._description; }
  get createdAt(): Date { return this._createdAt; }
  get updatedAt(): Date { return this._updatedAt; }

  set name(value: string) { this._name = value; }
  set description(value: string) { this._description = value; }
}
```

The class is **not optional**. Every entity must have its interface and its implementing class with essential getters and setters.

---

## Repository

Interface in `types/`, implementation in `repository/`.

```typescript
// Pattern: repository interface defines contract
// types/entity.interface.ts (can be in same file as entity interface or separate)
export interface IEntityRepository {
  findAll(): Promise<Entity[]>;
  findById(id: number): Promise<Entity>;
  create(data: CreateEntityDto): Promise<Entity>;
}

// Pattern: @Injectable class implements interface
// repository/entity.repository.ts
@Injectable()
export class EntityRepositoryPrisma implements IEntityRepository {
  constructor(private prisma: PrismaService) {}
  // Implementation here
}
```

---

## Swagger / OpenAPI

**Swagger first.** Every NestJS backend must expose an OpenAPI spec. The frontend depends on it.

Use `@nestjs/swagger` with `DocumentBuilder`. Use `@ApiTags`, `@ApiOperation`, `@ApiResponse` decorators on controllers. The spec is the contract between backend and frontend.

---

## Rules

- **Swagger first — non-negotiable.** Always expose OpenAPI spec. Without it, Orval cannot generate the frontend API layer. Do not skip or defer this step.
- **Entity class is mandatory.** Every entity in `types/` must have an interface file and a class file implementing it with getters and setters. The class cannot be omitted.
- **NestJS's DI system.** Use `@Inject` for interfaces, no manual instantiation.
- **Constructor injection always.** Same philosophy as Fastify, NestJS just automates it.
- **Interface + class separated** in `types/` and `repository/`.
- **DTOs for validation.** Use `class-validator` + `class-transformer` with DTOs.
- **Guards, not middleware** for auth in NestJS.
- **No `fromPersistence` pattern.** Do not add static reconstruction methods unless explicitly requested.
- **Ask monorepo vs separate repos.** When the user asks for both frontend and backend, always ask first. Never assume.
