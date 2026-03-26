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
func main() {
    cfg := config.Load()

    db, err := sql.Open("postgres", cfg.DatabaseUrl)
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    tokenManager := auth.NewTokenManager(cfg.JWTSecret, cfg.JWTExpiryHrs)

    // DI chain: repo -> service -> handler
    userRepo := repository.NewUserRepository(db)
    userService := services.NewUserService(userRepo)
    userHandler := handler.NewUserHandler(userService, tokenManager)

    taskRepo := repository.NewTaskRepository(db)
    taskService := services.NewTaskService(taskRepo)
    taskHandler := handler.NewTaskHandler(taskService)

    router := gin.Default()
    // ... register routes
    router.Run(":" + cfg.Port)
}
```

---

## Handler

Struct with service dependency, constructor function `New*Handler`.

```go
type TaskHandler struct {
    service *services.TaskService
}

func NewTaskHandler(service *services.TaskService) *TaskHandler {
    return &TaskHandler{service: service}
}

func (h *TaskHandler) GetTask(c *gin.Context) {
    id, err := strconv.ParseInt(c.Param("id"), 10, 64)
    if err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": "invalid task id"})
        return
    }

    task, err := h.service.GetTaskById(c.Request.Context(), id)
    if err != nil {
        c.JSON(http.StatusNotFound, gin.H{"error": "task not found"})
        return
    }

    c.JSON(http.StatusOK, task)
}
```

Request types defined in the same handler file:

```go
type CreateTaskRequest struct {
    Title       string `json:"title" binding:"required"`
    Description string `json:"description"`
    Status      string `json:"status"`
    Priority    string `json:"priority"`
    UserID      int64  `json:"user_id" binding:"required"`
    DueDate     string `json:"due_date"`
}
```

---

## Service

Thin layer that orchestrates repository calls. Receives repo via constructor.

```go
type TaskService struct {
    repo repository.TaskRepository
}

func NewTaskService(repo repository.TaskRepository) *TaskService {
    return &TaskService{repo: repo}
}

func (s *TaskService) GetTaskById(ctx context.Context, id int64) (*domain.Task, error) {
    return s.repo.GetTaskById(ctx, id)
}
```

---

## Repository

**Interface defined in the same file as the implementation.** Exported interface, unexported struct.

```go
// Interface
type TaskRepository interface {
    GetTaskById(ctx context.Context, id int64) (*domain.Task, error)
    ListTaskByUser(ctx context.Context, id int64) ([]domain.Task, error)
    CreateTask(ctx context.Context, ...) (*domain.Task, error)
    UpdateTask(ctx context.Context, ...) (*domain.Task, error)
    DeleteTask(ctx context.Context, id int64) error
}

// Implementation (unexported)
type taskRepository struct {
    queries *database.Queries
}

func NewTaskRepository(db *sql.DB) TaskRepository {
    return &taskRepository{
        queries: database.New(db),
    }
}
```

The repo converts sqlc generated models to domain structs manually:

```go
func (r *taskRepository) GetTaskById(ctx context.Context, id int64) (*domain.Task, error) {
    dbTask, err := r.queries.GetTaskByID(ctx, id)
    if err != nil {
        return nil, err
    }
    return &domain.Task{
        Id:          dbTask.ID,
        Title:       dbTask.Title,
        Description: dbTask.Description.String,
        Status:      dbTask.Status.String,
    }, nil
}
```

---

## Domain

Pure structs with JSON tags. No methods, no DB dependencies.

```go
type Task struct {
    Id          int64     `json:"id"`
    Title       string    `json:"title"`
    Description string    `json:"description"`
    Status      string    `json:"status"`
    Priority    string    `json:"priority"`
    Userid      int64     `json:"userId"`
    Duedate     time.Time `json:"dueDate"`
    CompletedAt time.Time `json:"completedAt"`
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
-- queries/tasks.sql
-- name: GetTaskByID :one
SELECT * FROM tasks WHERE id = $1;

-- name: ListTasksByUser :many
SELECT * FROM tasks WHERE user_id = $1;
```

---

## Swagger

Use swag comments on handlers for auto-generated docs:

```go
// GetTask godoc
// @Summary Obtener tarea por ID
// @Tags tasks
// @Security Bearer
// @Produce json
// @Param id path int true "Task ID"
// @Success 200 {object} domain.Task
// @Router /tasks/{id} [get]
```

---

## Rules

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
