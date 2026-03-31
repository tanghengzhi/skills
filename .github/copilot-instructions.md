# Copilot Custom Instructions — Laravel API Development

This repository is an Agent Skills collection. These instructions apply the **laravel-api** skill for GitHub Copilot.

## Language & Framework

- PHP 8.2+ with Laravel 11+
- `declare(strict_types=1);` at the top of every PHP file
- All methods must declare parameter and return types
- PSR-12 coding standard enforced via Laravel Pint

## Architecture

Follow **Controller → Service → Model** layers:

- **Controller** — Receives requests and returns responses only. No business logic.
- **Service** — All business logic. Injected into Controller via constructor.
- **Model** — Eloquent ORM with `SoftDeletes`. No raw SQL.
- **FormRequest** — All input validation. Use `$request->validated()`.
- **API Resource** — All output formatting. Never return a raw Model.

## Naming Conventions

- Controller: `PascalCase` + `Controller` — `UserController`
- Model: `PascalCase` singular — `User`, `OrderItem`
- Request: `Store/Update` + Model + `Request` — `StoreUserRequest`
- Resource: Model + `Resource` — `UserResource`
- Service: Model + `Service` — `UserService`
- Route path: `kebab-case` — `/api/user-profiles`
- DB table: `snake_case` plural — `order_items`
- DB column: `snake_case` — `created_at`

## API Standards

- Register routes with `Route::apiResource()`
- Route path uses `kebab-case`
- Unified JSON response: `{"code": 0, "message": "success", "data": {}}`
- HTTP status codes: `200` / `201` / `204` / `400` / `401` / `403` / `404` / `422` / `500`
- Use Laravel Sanctum for authentication (`auth:sanctum` middleware)
- Use Policy for authorization (`$this->authorize()`)
- Wrap multi-model operations in `DB::transaction()`
- Eager-load relationships with `with()` to prevent N+1 queries

## DON'Ts

- ❌ No business logic in Controllers
- ❌ No `DB::raw()` unless absolutely necessary
- ❌ No hardcoded config values — use `config()` or `.env`
- ❌ No `env()` outside config files
- ❌ No missing type declarations
- ❌ No queries inside loops (N+1)
- ❌ No raw Model in API responses — use Resource
- ❌ No `$request->all()` — use `$request->validated()`

## Quick Example

```php
// Good ✅
public function store(StoreUserRequest $request): JsonResponse
{
    $user = $this->userService->create($request->validated());
    return response()->json([
        'code' => 0,
        'message' => 'Created successfully',
        'data' => new UserResource($user),
    ], 201);
}

// Bad ❌
public function store(Request $request)
{
    $user = User::create($request->all());
    return response()->json($user);
}
```