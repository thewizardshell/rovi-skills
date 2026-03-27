---
name: rovi-fastapi
description: "FastAPI backend patterns: entity-based folder structure, service with constructor injection, repository pattern, dependency injection with Depends. Auto-loads when working with FastAPI, creating API endpoints, or building backend with Python and FastAPI."
---

# ROVI — FastAPI

## Philosophy

Same architecture as Fastify/NestJS but in Python. Entity-colocated folders, interface-first (via ABC), constructor injection, repository pattern. FastAPI's `Depends()` handles the DI wiring.

---

## Project Structure

```
src/
  config/
    database.py
    settings.py
  entities/
    [entity_name]/
      controller.py          > Router with endpoints
      service.py
      repository.py
      types/
        interface.py          > ABC (Abstract Base Class)
        entity.py             > Class implementing the ABC with getters/setters
        schemas.py            > Pydantic models
  auth/
    middleware/
    dependencies.py
  utils/
    errors.py
  main.py
```

---

## Controller (Router)

Instantiate dependencies inline via `Depends()`, same philosophy as Fastify.

```python
# Pattern: factory function builds the DI chain, router uses Depends()
from fastapi import APIRouter, Depends

router = APIRouter(prefix="/entities", tags=["entities"])

def get_entity_service(db=Depends(get_db)):
    repository = EntityRepository(db)
    return EntityService(repository)

@router.get("/", response_model=list[EntityResponse])
async def find_all(service: EntityService = Depends(get_entity_service)):
    return await service.find_all()

@router.post("/", response_model=EntityResponse, status_code=201)
async def create(data: CreateEntitySchema, service: EntityService = Depends(get_entity_service)):
    return await service.create(data)
```

---

## Service

Constructor injection via `__init__`.

```python
# Pattern: constructor receives interface, methods have business logic
class EntityService:
    def __init__(self, entity_repository: IEntityRepository):
        self._repository = entity_repository

    async def find_all(self):
        return await self._repository.find_all()

    async def create(self, data):
        # Business validation lives here
        return await self._repository.create(data)
```

---

## Entity Types

`types/` always contains: the **ABC interface**, a **class** implementing it with getters/setters (via `@property`), and Pydantic schemas.

```python
# Pattern: ABC defines the entity contract
# types/interface.py
from abc import ABC, abstractmethod

class IEntity(ABC):
    @property
    @abstractmethod
    def id(self) -> int: ...

    @property
    @abstractmethod
    def name(self) -> str: ...

    @property
    @abstractmethod
    def description(self) -> str: ...

# Pattern: class implements ABC with properties (getters/setters)
# types/entity.py
from .interface import IEntity

class Entity(IEntity):
    def __init__(self, id: int, name: str, description: str, created_at, updated_at):
        self._id = id
        self._name = name
        self._description = description
        self._created_at = created_at
        self._updated_at = updated_at

    @property
    def id(self) -> int:
        return self._id

    @property
    def name(self) -> str:
        return self._name

    @name.setter
    def name(self, value: str):
        self._name = value

    @property
    def description(self) -> str:
        return self._description

    @description.setter
    def description(self, value: str):
        self._description = value
```

The class is **not optional**. Every entity must have its ABC interface and its implementing class with essential getters and setters.

---

## Repository

ABC as interface, concrete class implementing it.

```python
# Pattern: ABC defines repository contract
# types/interface.py (repository interface, can be in same file or separate)
class IEntityRepository(ABC):
    @abstractmethod
    async def find_all(self) -> list: ...

    @abstractmethod
    async def create(self, data) -> dict: ...

# repository.py
class EntityRepository(IEntityRepository):
    def __init__(self, db):
        self._db = db
    # Implementation here
```

---

## Swagger / OpenAPI

**Swagger first.** FastAPI generates OpenAPI automatically from Pydantic models and route decorators. Just make sure `/docs` is accessible and schemas are well-typed. The frontend depends on this spec to auto-generate its API layer with Orval.

Keep Pydantic `response_model` on every route — it feeds the OpenAPI spec directly.

---

## Rules

- **Swagger first — non-negotiable.** FastAPI auto-generates it. Ensure all routes have `response_model` and typed schemas. Without the spec, Orval cannot generate the frontend API layer.
- **Entity class is mandatory.** Every entity in `types/` must have an ABC interface file and a class file implementing it with `@property` getters and setters. The class cannot be omitted.
- **`entities/` folder structure.** Same as Fastify — each entity is self-contained.
- **`Depends()` for DI wiring** in the controller/router. Factory function that builds the chain.
- **Constructor injection in services.** Via `__init__`, same philosophy.
- **ABC for interfaces.** Python equivalent of TypeScript interfaces.
- **Pydantic for schemas/DTOs.** Validation at the API boundary.
- **No `fromPersistence` pattern equivalent.** No static factory methods unless explicitly requested.
- **Ask monorepo vs separate repos.** When the user asks for both frontend and backend, always ask first. Never assume.
