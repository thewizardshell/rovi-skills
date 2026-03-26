---
name: rovi-go
description: "Go backend patterns: standard project layout with cmd/internal, handler/service/repository layers, interface-based repos, Gin framework, sqlc for queries, constructor functions (New*). Auto-loads when working with Go, creating APIs, or building backend with Go and Gin."
---

# ROVI — Go

## Stack

- **Gin** for HTTP routing
- **sqlc** for type-safe SQL queries
- **PostgreSQL** as primary database
- **JWT** for authentication
- **Swagger** (swag) for API documentation

---

## Project Structure

Standard Go layout: `cmd/` for entrypoint, `internal/` for all business code.

```
cmd/
  main.go                        > Entrypoint: wiring, DI, router setup
internal/
  config/
    config.go                    > Env vars, app config
  database/
    db.go                        > DB connection
    migrations/
      001_initial.sql
    queries/
      [entity].sql               > sqlc SQL queries
    models.go                    > sqlc generated models
    querier.go                   > sqlc generated interface
    [entity].sql.go              > sqlc generated Go code
  domain/
    [entity].go                  > Domain structs (pure, no DB deps)
  handler/
    [entity]_handler.go          > HTTP handlers (Gin)
  services/
    [entity]_service.go          > Business logic
  repository/
    [entity]_repository.go       > Interface + implementation
  middleware/
    auth.go
    logger.go
  auth/
    jwt.go                       > Token manager
  utils/
    [helpers].go
  errors/
    errors.go                    > Centralized error vars
Makefile
sqlc.yaml
go.mod
```

---

## Wiring in main.go

All DI happens in `main.go`. Build the chain: repo → service → handler. Register routes.

```go
// Pattern: all instantiation in main, chain repo -> service -> handler
func main() {
    cfg := config.Load()

    db, err := sql.Open("postgres", cfg.DatabaseUrl)
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // DI chain per entity
    entityRepo := repository.NewEntityRepository(db)
    entityService := services.NewEntityService(entityRepo)
    entityHandler := handler.NewEntityHandler(entityService)

    router := gin.Default()
    // ... register routes with entityHandler
    router.Run(":" + cfg.Port)
}
```

---

## Handler

Struct with service dependency, constructor function `New*Handler`.

```go
// Pattern: struct holds service, New* constructor, methods are HTTP handlers
type EntityHandler struct {
    service *services.EntityService
}

func NewEntityHandler(service *services.EntityService) *EntityHandler {
    return &EntityHandler{service: service}
}

func (h *EntityHandler) GetById(c *gin.Context) {
    id, err := strconv.ParseInt(c.Param("id"), 10, 64)
    if err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": "invalid id"})
        return
    }

    entity, err := h.service.GetById(c.Request.Context(), id)
    if err != nil {
        c.JSON(http.StatusNotFound, gin.H{"error": "not found"})
        return
    }

    c.JSON(http.StatusOK, entity)
}
```

Request types defined in the same handler file:

```go
// Pattern: request structs with json tags + binding validation
type CreateEntityRequest struct {
    Name        string `json:"name" binding:"required"`
    Description string `json:"description"`
}
```

---

## Service

Thin layer that orchestrates repository calls. Receives repo via constructor.

```go
// Pattern: struct holds repo interface, New* constructor
type EntityService struct {
    repo repository.EntityRepository
}

func NewEntityService(repo repository.EntityRepository) *EntityService {
    return &EntityService{repo: repo}
}

func (s *EntityService) GetById(ctx context.Context, id int64) (*domain.Entity, error) {
    return s.repo.GetById(ctx, id)
}
```

---

## Repository

**Interface defined in the same file as the implementation.** Exported interface, unexported struct.

```go
// Pattern: exported interface, unexported struct, New* returns interface
type EntityRepository interface {
    GetById(ctx context.Context, id int64) (*domain.Entity, error)
    List(ctx context.Context) ([]domain.Entity, error)
    Create(ctx context.Context, ...) (*domain.Entity, error)
    Delete(ctx context.Context, id int64) error
}

type entityRepository struct {
    queries *database.Queries
}

func NewEntityRepository(db *sql.DB) EntityRepository {
    return &entityRepository{
        queries: database.New(db),
    }
}
```

The repo converts sqlc generated models to domain structs manually.

---

## Domain

Pure structs with JSON tags. No methods, no DB dependencies.

```go
// Pattern: plain struct, json tags, no DB types
type Entity struct {
    Id          int64     `json:"id"`
    Name        string    `json:"name"`
    Description string    `json:"description"`
    CreatedAt   time.Time `json:"createdAt"`
    UpdatedAt   time.Time `json:"updatedAt"`
}
```

---

## Errors

Centralized `var` block with sentinel errors:

```go
var (
    ErrNotFound       = errors.New("resource not found")
    ErrBadRequest     = errors.New("bad request")
    ErrInternalServer = errors.New("internal server error")
)
```

---

## sqlc

SQL queries in `.sql` files, sqlc generates Go code. Config in `sqlc.yaml`.

```sql
-- Pattern: named queries, sqlc generates typed Go functions
-- name: GetEntityByID :one
SELECT * FROM entities WHERE id = $1;

-- name: ListEntities :many
SELECT * FROM entities ORDER BY created_at DESC;
```

---

## Swagger

Use swag comments on handlers for auto-generated docs.

---

## Rules

- **Swagger first.** Always expose OpenAPI spec with swag comments. The frontend consumes it with Orval.
- **`cmd/` + `internal/`** standard layout. No `pkg/` unless publishing a library.
- **Constructor functions `New*`** for all structs with dependencies.
- **Interface in the same file** as the implementation (Go convention).
- **Exported interface, unexported struct.** The constructor returns the interface.
- **sqlc for queries.** No raw SQL strings in Go code, no ORM.
- **Domain structs are pure.** No DB types, no methods. Conversion happens in the repository.
- **Request types in the handler file.** Keep them close to where they're used.
- **All DI wiring in `main.go`.** One place to see the full dependency graph.
- **Swagger comments on every handler.** Spanish descriptions, English field names.
- **Makefile** for common commands (build, run, migrate, sqlc generate, swagger).
