---
name: laravel-api
description: Build RESTful APIs with Laravel 11+ using Controller-Service-Model architecture, FormRequest validation, API Resources, and PSR-12 standards. Use when developing Laravel APIs, creating controllers, services, models, migrations, or writing PHP backend code.
---

# Laravel API Development

## Quick Start

```bash
# Scaffold a full resource (Model + Migration + Factory + Seeder + Controller + Request + Resource)
php artisan make:model ModelName -mfscR

# Run migrations
php artisan migrate

# Run tests
php artisan test

# Format code
./vendor/bin/pint
```

## Architecture

Follow the **Controller → Service → Model** layered architecture:

- **Controller** — Receives requests, returns responses. No business logic.
- **Service** — All business logic lives here.
- **Model** — Eloquent ORM. Use SoftDeletes. No raw SQL.
- **FormRequest** — All input validation.
- **API Resource** — All output formatting.

## Naming Conventions

| Class | Convention | Example |
|-------|-----------|---------|
| Controller | `PascalCase` + `Controller` | `UserController` |
| Model | `PascalCase` singular | `User`, `OrderItem` |
| Request | `Store/Update` + Model + `Request` | `StoreUserRequest` |
| Resource | Model + `Resource` | `UserResource` |
| Service | Model + `Service` | `UserService` |
| Repository | Model + `Repository` | `UserRepository` |
| Route | `kebab-case` | `/api/user-profiles` |
| DB field | `snake_case` | `created_at` |
| DB table | `snake_case` plural | `order_items` |

## Key Rules

**DO ✅**
- Use `declare(strict_types=1);` at the top of every file
- Declare parameter and return types on all methods
- Use `$request->validated()` — never `$request->all()`
- Use `with()` to eager-load relationships (prevent N+1)
- Use `DB::transaction()` for multi-model operations
- Use Laravel Sanctum for API authentication
- Use Policy for authorization checks
- Write Feature tests for every API endpoint

**DON'T ❌**
- Write business logic in Controllers
- Use `DB::raw()` unless absolutely necessary
- Hardcode config values — use `config()` or `.env`
- Call `env()` outside of config files
- Return a plain Model as API response — use Resource
- Execute queries inside loops (N+1)
- Expose sensitive fields (`password`, `remember_token`) in responses

## Unified JSON Response Format

```json
{ "code": 0, "message": "success", "data": {} }
```

HTTP status codes: `200` / `201` / `204` / `400` / `401` / `403` / `404` / `422` / `500`

## References

See [coding-conventions.md](coding-conventions.md), [api-standards.md](api-standards.md), [examples.md](examples.md)
