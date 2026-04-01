# API Standards

## RESTful Route Design

Register API routes using `apiResource`:

```php
// routes/api.php
Route::middleware('auth:sanctum')->group(function () {
    Route::apiResource('users', UserController::class);
    Route::apiResource('user-profiles', UserProfileController::class);
});
```

`apiResource` generates these routes automatically:

| Method | URI | Action |
|--------|-----|--------|
| GET | `/api/users` | `index` |
| POST | `/api/users` | `store` |
| GET | `/api/users/{user}` | `show` |
| PUT/PATCH | `/api/users/{user}` | `update` |
| DELETE | `/api/users/{user}` | `destroy` |

## Unified JSON Response Format

### Success Response
```json
{
    "code": 0,
    "message": "success",
    "data": {}
}
```

### Paginated Response
```json
{
    "code": 0,
    "message": "success",
    "data": {
        "items": [],
        "pagination": {
            "total": 100,
            "per_page": 15,
            "current_page": 1,
            "last_page": 7
        }
    }
}
```

### Error Response
```json
{
    "code": 40001,
    "message": "Validation failed",
    "errors": {
        "email": ["The email field is required."]
    }
}
```

## HTTP Status Codes

| Code | When to use |
|------|-------------|
| `200` | Successful GET, PUT/PATCH |
| `201` | Successful POST (resource created) |
| `204` | Successful DELETE (no content) |
| `400` | Bad request |
| `401` | Unauthenticated |
| `403` | Unauthorized (forbidden) |
| `404` | Resource not found |
| `422` | Validation failed |
| `500` | Server error |

## FormRequest Validation

Always use `FormRequest` for input validation. Never use `$request->all()` or `$request->input()` directly in controllers.

```php
// app/Http/Requests/StoreUserRequest.php
class StoreUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true; // Use Policy for authorization
    }

    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'email', 'unique:users,email'],
            'password' => ['required', 'string', 'min:8', 'confirmed'],
        ];
    }
}
```

## API Resource Output

Always use API Resources to format responses. Never return a Model directly.

```php
// app/Http/Resources/UserResource.php
class UserResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at->toISOString(),
        ];
    }
}
```

## Authentication

Use Laravel Sanctum for API token authentication:

```php
// Issue token on login
$token = $user->createToken('api-token')->plainTextToken;

// Protect routes
Route::middleware('auth:sanctum')->group(function () {
    // protected routes
});
```

## Authorization

Use Laravel Policies for authorization checks. Call in controller or via `$this->authorize()`:

```php
public function update(UpdatePostRequest $request, Post $post): PostResource
{
    $this->authorize('update', $post);
    $post = $this->postService->update($post->id, $request->validated());
    return new PostResource($post);
}
```

## Database Transactions

Wrap multi-model operations in a transaction:

```php
public function create(array $data): User
{
    return DB::transaction(function () use ($data) {
        $user = User::create($data);
        $user->profile()->create(['bio' => '']);
        return $user;
    });
}
```

## N+1 Query Prevention

Always eager-load relationships needed in a response:

```php
// Good ✅
User::with(['posts', 'profile'])->paginate(15);

// Bad ❌ — triggers N+1 queries
$users = User::paginate(15);
foreach ($users as $user) {
    $user->posts; // N+1!
}
```

## Pagination

Use Laravel's built-in `paginate()` and return pagination metadata:

```php
// Service
public function paginate(int $perPage = 15): LengthAwarePaginator
{
    return User::query()->with('profile')->latest()->paginate($perPage);
}

// Resource Collection response includes pagination links automatically
return UserResource::collection($users);
```
