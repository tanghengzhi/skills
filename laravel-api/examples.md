# Code Examples

## Controller

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Requests\StoreUserRequest;
use App\Http\Requests\UpdateUserRequest;
use App\Http\Resources\UserResource;
use App\Services\UserService;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Resources\Json\AnonymousResourceCollection;

class UserController extends Controller
{
    public function __construct(
        private readonly UserService $userService,
    ) {}

    public function index(): AnonymousResourceCollection
    {
        $users = $this->userService->paginate();
        return UserResource::collection($users);
    }

    public function store(StoreUserRequest $request): JsonResponse
    {
        $user = $this->userService->create($request->validated());
        return response()->json([
            'code' => 0,
            'message' => 'Created successfully',
            'data' => new UserResource($user),
        ], 201);
    }

    public function show(int $id): UserResource
    {
        $user = $this->userService->findOrFail($id);
        return new UserResource($user);
    }

    public function update(UpdateUserRequest $request, int $id): UserResource
    {
        $user = $this->userService->update($id, $request->validated());
        return new UserResource($user);
    }

    public function destroy(int $id): JsonResponse
    {
        $this->userService->delete($id);
        return response()->json([
            'code' => 0,
            'message' => 'Deleted successfully',
        ]);
    }
}
```

## Service

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\Models\User;
use Illuminate\Contracts\Pagination\LengthAwarePaginator;
use Illuminate\Support\Facades\DB;

class UserService
{
    public function paginate(int $perPage = 15): LengthAwarePaginator
    {
        return User::query()->with('profile')->latest()->paginate($perPage);
    }

    public function findOrFail(int $id): User
    {
        return User::findOrFail($id);
    }

    public function create(array $data): User
    {
        return DB::transaction(function () use ($data) {
            $user = User::create($data);
            $user->profile()->create([]);
            return $user;
        });
    }

    public function update(int $id, array $data): User
    {
        $user = $this->findOrFail($id);
        $user->update($data);
        return $user->refresh();
    }

    public function delete(int $id): bool
    {
        return $this->findOrFail($id)->delete();
    }
}
```

## FormRequest

```php
<?php

declare(strict_types=1);

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
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

## API Resource

```php
<?php

declare(strict_types=1);

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'profile' => new UserProfileResource($this->whenLoaded('profile')),
            'created_at' => $this->created_at->toISOString(),
        ];
    }
}
```

## Migration

```php
<?php

declare(strict_types=1);

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
            $table->softDeletes();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('users');
    }
};
```

## Routes

```php
// routes/api.php
Route::middleware('auth:sanctum')->group(function () {
    Route::apiResource('users', UserController::class);
    Route::apiResource('user-profiles', UserProfileController::class);
});

// Public routes
Route::post('auth/login', [AuthController::class, 'login']);
Route::post('auth/register', [AuthController::class, 'register']);
```

## Feature Test

```php
<?php

declare(strict_types=1);

namespace Tests\Feature;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class UserApiTest extends TestCase
{
    use RefreshDatabase;

    public function test_authenticated_user_can_list_users(): void
    {
        $user = User::factory()->create();
        User::factory()->count(5)->create();

        $response = $this->actingAs($user)
            ->getJson('/api/users');

        $response->assertStatus(200)
            ->assertJsonStructure([
                'data' => [
                    '*' => ['id', 'name', 'email', 'created_at'],
                ],
            ]);
    }

    public function test_authenticated_user_can_create_user(): void
    {
        $actor = User::factory()->create();

        $response = $this->actingAs($actor)
            ->postJson('/api/users', [
                'name' => 'Jane Doe',
                'email' => 'jane@example.com',
                'password' => 'password',
                'password_confirmation' => 'password',
            ]);

        $response->assertStatus(201)
            ->assertJsonPath('code', 0)
            ->assertJsonPath('data.email', 'jane@example.com');

        $this->assertDatabaseHas('users', ['email' => 'jane@example.com']);
    }

    public function test_unauthenticated_request_returns_401(): void
    {
        $this->getJson('/api/users')->assertStatus(401);
    }
}
```

## Good vs Bad

### Controller — Business Logic

```php
// Good ✅ — delegate to Service
public function store(StoreUserRequest $request): JsonResponse
{
    $user = $this->userService->create($request->validated());
    return response()->json(['code' => 0, 'message' => 'Created successfully', 'data' => new UserResource($user)], 201);
}

// Bad ❌ — business logic in Controller
public function store(Request $request): JsonResponse
{
    $user = User::create($request->all());
    $user->profile()->create([]);
    Mail::to($user)->send(new WelcomeMail($user));
    return response()->json($user);
}
```

### Response — Always Use Resource

```php
// Good ✅
return new UserResource($user);

// Bad ❌ — exposes internal fields including password hash
return response()->json($user);
```
