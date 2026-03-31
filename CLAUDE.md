# CLAUDE.md

## 项目概述 / Project Overview

这是一个技能配置仓库，包含 AI 编程助手（Claude Code / GitHub Copilot）的自定义指令，主要用于 Laravel 接口开发。

## 技术栈 / Tech Stack

- **语言**: PHP 8.2+
- **框架**: Laravel 11+
- **数据库**: MySQL 8.0 / PostgreSQL 15+
- **缓存**: Redis
- **API 规范**: RESTful API
- **认证**: Laravel Sanctum / Passport
- **文档**: Swagger / OpenAPI 3.0

## 编码规范 / Coding Conventions

### PHP / Laravel

- 严格遵循 PSR-12 编码标准
- 使用 PHP 8.2+ 特性：枚举、只读属性、交叉类型、Fiber 等
- 所有方法必须声明参数类型和返回类型
- 使用 strict_types 声明：`declare(strict_types=1);`
- 优先使用 Laravel 内置功能，避免重复造轮子

### 命名规则

- **Controller**: `PascalCase`，以 `Controller` 结尾，如 `UserController`
- **Model**: `PascalCase` 单数形式，如 `User`、`OrderItem`
- **Migration**: Laravel 默认格式 `yyyy_mm_dd_hhmmss_create_xxx_table.php`
- **Request**: `PascalCase`，如 `StoreUserRequest`、`UpdateUserRequest`
- **Resource**: `PascalCase`，如 `UserResource`、`UserCollection`
- **Service**: `PascalCase`，以 `Service` 结尾，如 `UserService`
- **Repository**: `PascalCase`，以 `Repository` 结尾，如 `UserRepository`
- **路由**: 使用 `kebab-case`，如 `/api/user-profiles`
- **数据库字段**: 使用 `snake_case`

### 目录结构

```
app/
├── Http/
│   ├── Controllers/Api/    # API 控制器
│   ├── Middleware/          # 中间件
│   ├── Requests/           # 表单验证请求
│   └── Resources/          # API 资源转换
├── Models/                 # Eloquent 模型
├── Services/               # 业务逻辑层
├── Repositories/           # 数据访问层
├── Enums/                  # 枚举类
├── Events/                 # 事件
├── Listeners/              # 事件监听器
├── Jobs/                   # 队列任务
├── Notifications/          # 通知
└── Exceptions/             # 自定义异常
routes/
├── api.php                 # API 路由
database/
├── migrations/             # 数据库迁移
├── factories/              # 模型工厂
└── seeders/                # 数据填充
tests/
├── Feature/                # 功能测试
└── Unit/                   # 单元测试
```

## API 开发规范 / API Development Standards

### 请求与响应

- 使用 `FormRequest` 进行请求验证
- 使用 `API Resource` 进行响应数据转换
- 统一 JSON 响应格式：

```php
// 成功响应
{
    "code": 0,
    "message": "success",
    "data": {}
}

// 分页响应
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

// 错误响应
{
    "code": 40001,
    "message": "Validation failed",
    "errors": {
        "email": ["The email field is required."]
    }
}
```

### 控制器示例

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

### Service 层示例

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\Models\User;
use Illuminate\Contracts\Pagination\LengthAwarePaginator;
use Illuminate\Database\Eloquent\ModelNotFoundException;

class UserService
{
    public function paginate(int $perPage = 15): LengthAwarePaginator
    {
        return User::query()->latest()->paginate($perPage);
    }

    public function findOrFail(int $id): User
    {
        return User::findOrFail($id);
    }

    public function create(array $data): User
    {
        return User::create($data);
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

## 常用命令 / Common Commands

```bash
# 创建模型 + 迁移 + 工厂 + Seeder + Controller + Request + Resource
php artisan make:model ModelName -mfscR

# 运行迁移
php artisan migrate

# 运行测试
php artisan test

# 路由列表
php artisan route:list --path=api

# 代码格式化 (Laravel Pint)
./vendor/bin/pint

# 静态分析 (PHPStan / Larastan)
./vendor/bin/phpstan analyse
```

## 注意事项 / Important Notes

- ❌ 不要在 Controller 中写业务逻辑，使用 Service 层
- ❌ 不要直接在 Controller 中操作 Model，使用 Service 或 Repository
- ❌ 不要使用 `DB::raw()` 除非绝对必要
- ❌ 不要硬编码配置值，使用 `config()` 或 `.env`
- ✅ 始终使用 Eloquent ORM 和查询构建器
- ✅ 使用 Laravel 的软删除（SoftDeletes）
- ✅ 使用数据库事务处理关联操作
- ✅ 为所有 API 编写 Feature 测试
- ✅ 使用队列处理耗时任务
- ✅ 使用 Redis 缓存高频查询