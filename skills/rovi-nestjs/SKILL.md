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
        [entityName].interface.ts
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

## Repository

Same pattern: interface in `types/`, implementation in `repository/`.

```typescript
// Pattern: interface defines contract
// types/entity.interface.ts
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

- **Swagger first.** Always expose OpenAPI spec. The frontend consumes it with Orval.
- **NestJS's DI system.** Use `@Inject` for interfaces, no manual instantiation.
- **Constructor injection always.** Same philosophy as Fastify, NestJS just automates it.
- **Interface + class separated** in `types/` and `repository/`.
- **DTOs for validation.** Use `class-validator` + `class-transformer` with DTOs.
- **Guards, not middleware** for auth in NestJS.
- **No `fromPersistence` pattern.** Do not add static reconstruction methods unless explicitly requested.
- **Ask monorepo vs separate repos.** When the user asks for both frontend and backend, always ask first. Never assume.
