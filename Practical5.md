# Practical 5 — Integration Testing with TestContainers

## Complete reference report (practical5-example)

### Overview
This project demonstrates integration testing with TestContainers in Go.  
It uses PostgreSQL for real DB tests and Redis for caching in multi-container tests. The suite covers full CRUD, advanced queries (pattern/count/date), transaction testing, and multi-container cache tests.

[Link to Repo](https://github.com/Dupchuwangmo7/testcontainers-demo)

### Prerequisites
- Go 1.21+
- Docker Desktop running
- ~500 MB free disk for Docker images

### Quick start
```bash
# Clone / navigate to project
cd practicals/practical5-example

# Download dependencies
go mod download

# Run all tests
go test ./... -v

# Run tests with coverage
go test -cover ./repository

# Run specific test
go test ./repository -run TestGetByID -v
```

### Project structure
```
practical5-example/
├── models/
│   └── user.go                          # User data model
├── repository/
│   ├── user_repository.go               # Basic CRUD operations + advanced queries + transactions
│   ├── user_repository_test.go          # Integration tests (Exercises 1–4)
│   ├── cached_user_repository.go        # Redis caching layer
│   └── cached_user_repository_test.go   # Multi-container tests (Exercise 5)
├── migrations/
│   └── init.sql                         # Database schema + seed data
├── go.mod                               # Dependencies
└── README.md                            # This file / project notes
```

### Exercises covered
- Exercise 1–2 (Basic CRUD)
  - TestGetByID — retrieve user by ID
  - TestGetByEmail — retrieve user by email
  - TestCreate — create user (incl. duplicate-email check)
  - TestUpdate — update existing user
  - TestDelete — delete user
  - TestList — list all users

- Exercise 3 (Advanced queries)
  - FindByNamePattern — ILIKE pattern matching
  - CountUsers — total users count
  - GetRecentUsers — users created within last N days

- Exercise 4 (Transactions)
  - BatchCreate / TransferUserData — atomic multi-step operations
  - TestTransactionRollback — ensure rollback on failure
  - TestConcurrentWrites — concurrent behaviour checks

- Exercise 5 (Multi-container)
  - PostgreSQL + Redis setup in tests
  - CachedUserRepository — cache hit/miss, invalidation, TTL

### Running tests (commands)
```bash
# All tests
go test ./... -v

# With coverage
go test -coverprofile=coverage.out ./repository
go tool cover -html=coverage.out -o coverage.html

# Race detection
go test -race ./repository

# Skip slow tests
go test ./repository -short
```

### Understanding the tests
- TestMain in each test file:
  - Starts Docker container(s) via TestContainers
  - Waits for readiness (port/log wait strategies)
  - Runs `migrations/init.sql` to create schema and seed data
  - Runs tests, then terminates containers

- Test isolation strategies:
  - Per-test cleanup (defer / t.Cleanup)
  - Transaction rollback for temporary state
  - Optionally start fresh container per test for stronger isolation

### Container lifecycle (high level)
Test start → Pull/start containers → DB init (migrations) → Tests run → Containers terminated

### Troubleshooting (common issues & fixes)
- Cannot connect to Docker daemon:
  - Ensure Docker Desktop is running. Check with `docker ps`.
- Container startup timeout:
  - Increase wait timeout, e.g. `wait.ForListeningPort("5432/tcp").WithStartupTimeout(60*time.Second)`
  - Allocate more Docker resources in Docker Desktop settings
- Tests slow:
  - First run pulls images; subsequent runs use cached images
  - Pre-pull images in CI to save time
- Port conflicts:
  - TestContainers maps random host ports automatically; use mapped port returned by container

### CI recommendations
- Ensure Docker is available in the runner (or use DinD/service containers)
- Pre-pull images in CI to reduce startup time
- Consider reusing containers in CI only if acceptable for isolation trade-offs

### Key learnings
- Tests run against a real DB (Postgres), catching DB-specific issues missed by mocks
- TestContainers provides reproducible container lifecycle and isolation
- Combination of cleanup, transactions, and fresh containers offers flexible isolation strategies
- Multi-container tests validate cross-service interactions (Postgres + Redis)

### Artifacts to submit
1. ![alt text](Assets/P5(1).png)

![alt text](Assets/P5(2).png)

![alt text](Assets/P5(3).png)