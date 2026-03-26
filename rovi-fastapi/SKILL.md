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

## Repository

ABC as interface, concrete class implementing it.

```python
# Pattern: ABC defines contract, class implements it
# types/interface.py
from abc import ABC, abstractmethod

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

- **Swagger first.** FastAPI auto-generates it. Ensure all routes have `response_model` and typed schemas.
- **`entities/` folder structure.** Same as Fastify — each entity is self-contained.
- **`Depends()` for DI wiring** in the controller/router. Factory function that builds the chain.
- **Constructor injection in services.** Via `__init__`, same philosophy.
- **ABC for interfaces.** Python equivalent of TypeScript interfaces.
- **Pydantic for schemas/DTOs.** Validation at the API boundary.
- **No `fromPersistence` pattern equivalent.** No static factory methods unless explicitly requested.
