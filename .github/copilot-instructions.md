# Copilot Custom Instructions

## 项目上下文 / Project Context

这是一个基于 Laravel 的 RESTful API 项目。所有代码建议应遵循 Laravel 最佳实践和 PSR-12 编���标准。

## 语言与框架 / Language & Framework

- 使用 PHP 8.2+ 语法特性（枚举、只读属性、命名参数、联合类型、交叉类型）
- 框架为 Laravel 11+
- 始终在文件顶部添加 `declare(strict_types=1);`
- 所有方法必须声明参数类型和返回类型

## 架构模式 / Architecture Patterns

- 采用 Controller → Service → Repository / Model 分层架构
- Controller 只负责接收请求和返回响应，不包含业务逻辑
- Service 层处理所有业务逻辑
- 使用 `FormRequest` 进行所有输入验证
- 使用 `API Resource` 进行所有输出格式化

## API 开发规范 / API Standards

- RESTful 路由设计，使用 `apiResource` 注册路由
- 路由使用 `kebab-case`，如 `/api/user-profiles`
- 统一 JSON 响应格式：`{"code": 0, "message": "success", "data": {}}`
- 分页使用 Laravel 内置 paginate，返回 pagination 元信息
- 使用 HTTP 状态码：200/201/204/400/401/403/404/422/500

## 数据库 / Database

- 使用 Eloquent ORM，避免原始 SQL
- 数据库字段使用 `snake_case`
- 始终使用迁移文件管理数据库变更
- 关联操作使用数据库事务（DB::transaction）
- 模型使用 SoftDeletes
- 注意 N+1 查询问题，使用 `with()` 预加载关联

## 命名规范 / Naming

- Controller: `PascalCase` + `Controller` 后缀
- Model: `PascalCase` 单数
- Request: `Store/Update` + `ModelName` + `Request`
- Resource: `ModelName` + `Resource`
- Service: `ModelName` + `Service`
- 数据库表名: `snake_case` 复数
- 路由参数: `camelCase`

## 安全 / Security

- 使用 Laravel Sanctum 进行 API 认证
- 始终通过 FormRequest 验证和清洗输入
- 使用 Policy 进行授权检查
- 敏感配置使用 `.env` 环境变量
- 不要在响应中暴露敏感字段（password、remember_token 等）

## 测试 / Testing

- 为每个 API 端点编写 Feature 测试
- 使用 Laravel 模型工厂生成测试数据
- 测试方法命名: `test_能描述行为的名称` 或 `/** @test */`
- 使用 `assertJson`、`assertStatus` 断言 API 响应

## 禁止事项 / DON'Ts

- ❌ 不要在 Controller 中写业务逻辑
- ❌ 不要使用 `DB::raw()` 除非绝对必要
- ❌ 不要硬编码配置值
- ❌ 不要使用 `env()` 在 config 文件之外获取环境变量
- ❌ 不要忽略类型声明
- ❌ 不要在循环中执行数据库查询（N+1 问题）
- ❌ 不要返回整个 Model 作为 API 响应，使用 Resource

## 代码示例 / Code Examples

### Good ✅
```php
public function store(StoreUserRequest $request): JsonResponse
{
    $user = $this->userService->create($request->validated());
    return response()->json([
        'code' => 0,
        'message' => 'Created successfully',
        'data' => new UserResource($user),
    ], 201);
}
```

### Bad ❌
```php
public function store(Request $request)
{
    $user = User::create($request->all());
    return response()->json($user);
}
```