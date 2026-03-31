# Coding Conventions

## Language & Framework Requirements

- **PHP**: 8.2+ — use enums, readonly properties, named arguments, union types, intersection types, Fibers
- **Framework**: Laravel 11+
- **Standards**: PSR-12 strictly enforced
- **Formatting**: Laravel Pint (`./vendor/bin/pint`)
- **Static Analysis**: PHPStan / Larastan (`./vendor/bin/phpstan analyse`)

## File Header

Every PHP file must start with:

```php
<?php

declare(strict_types=1);
```

## Type Declarations

All methods must declare parameter types and return types:

```php
// Good ✅
public function findOrFail(int $id): User
{
    return User::findOrFail($id);
}

// Bad ❌
public function findOrFail($id)
{
    return User::findOrFail($id);
}
```

## Directory Structure

```
app/
├── Http/
│   ├── Controllers/Api/    # API controllers
│   ├── Middleware/
│   ├── Requests/           # FormRequest classes
│   └── Resources/          # API Resource classes
├── Models/                 # Eloquent models
├── Services/               # Business logic
├── Repositories/           # Data access layer
├── Enums/                  # PHP 8.1+ enums
├── Events/
├── Listeners/
├── Jobs/                   # Queue jobs
├── Notifications/
└── Exceptions/             # Custom exceptions
routes/
└── api.php
database/
├── migrations/
├── factories/
└── seeders/
tests/
├── Feature/                # API endpoint tests
└── Unit/                   # Unit tests
```

## Naming Rules

| Component | Convention | Example |
|-----------|-----------|---------|
| Controller | `PascalCase` + `Controller` suffix | `UserController` |
| Model | `PascalCase` singular | `User`, `OrderItem` |
| Migration | Laravel default timestamp format | `2024_01_01_000000_create_users_table.php` |
| FormRequest | `Store/Update` + Model + `Request` | `StoreUserRequest`, `UpdateUserRequest` |
| API Resource | Model + `Resource` | `UserResource` |
| Resource Collection | Model + `Collection` | `UserCollection` |
| Service | Model + `Service` | `UserService` |
| Repository | Model + `Repository` | `UserRepository` |
| Route path | `kebab-case` | `/api/user-profiles` |
| Route parameter | `camelCase` | `userId` |
| DB table | `snake_case` plural | `order_items` |
| DB column | `snake_case` | `created_at`, `user_id` |
| Enum | `PascalCase` | `UserStatus` |

## Model Requirements

- Always use `SoftDeletes` trait
- Define `$fillable` explicitly — never use `$guarded = []`
- Define relationship methods with proper return types
- Use `$casts` for type casting (dates, enums, JSON)

```php
class User extends Model
{
    use HasFactory, SoftDeletes;

    protected $fillable = ['name', 'email', 'password'];

    protected $hidden = ['password', 'remember_token'];

    protected $casts = [
        'email_verified_at' => 'datetime',
        'status' => UserStatus::class,
    ];

    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}
```
