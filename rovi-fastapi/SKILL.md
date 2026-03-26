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

Instantiate dependencies inline, same philosophy as Fastify.

```python
from fastapi import APIRouter, Depends
from .service import OwnerService
from .repository import OwnerRepository
from .types.schemas import CreateOwnerSchema, OwnerResponse
from src.config.database import get_db

router = APIRouter(prefix="/owners", tags=["owners"])

def get_owner_service(db=Depends(get_db)) -> OwnerService:
    repository = OwnerRepository(db)
    return OwnerService(repository)

@router.get("/", response_model=list[OwnerResponse])
async def find_all(service: OwnerService = Depends(get_owner_service)):
    return await service.find_all()

@router.post("/", response_model=OwnerResponse, status_code=201)
async def create(data: CreateOwnerSchema, service: OwnerService = Depends(get_owner_service)):
    return await service.create(data)
```

---

## Service

Constructor injection via `__init__`.

```python
from .types.interface import IOwnerRepository
from src.utils.errors import DuplicateError

class OwnerService:
    def __init__(self, owner_repository: IOwnerRepository):
        self._repository = owner_repository

    async def find_all(self):
        return await self._repository.find_all()

    async def create(self, data):
        existing = await self._repository.find_by_email(data.email)
        if existing:
            raise DuplicateError("Owner", "email")
        return await self._repository.create(data)
```

---

## Repository

ABC as interface, concrete class implementing it.

```python
# types/interface.py
from abc import ABC, abstractmethod

class IOwnerRepository(ABC):
    @abstractmethod
    async def find_all(self) -> list:
        ...

    @abstractmethod
    async def find_by_email(self, email: str):
        ...

    @abstractmethod
    async def create(self, data) -> dict:
        ...

# repository.py
from .types.interface import IOwnerRepository

class OwnerRepository(IOwnerRepository):
    def __init__(self, db):
        self._db = db

    async def find_all(self) -> list:
        return await self._db.owners.find_many()

    async def find_by_email(self, email: str):
        return await self._db.owners.find_first(where={"email": email})
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
