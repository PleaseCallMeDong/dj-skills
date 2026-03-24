# Java 后端开发方法论 Skill

## 触发条件

- 手动触发: 输入 `/java` 或 `java`
- 自动触发: 当用户讨论Java后端开发、代码设计、架构等话题时

## 概述

这是一个面向中高级Java开发者的方法论指导skill，涵盖编码规范、设计原则和工程实践三大方面。

---

## 一、编码规范

### 1.1 命名规范

**类名**: 使用UpperCamelCase

```
UserService, OrderController, ProductDAO
```

**方法名/变量名**: 使用lowerCamelCase

```
getUserById, calculateTotalPrice, orderList
```

**常量**: 使用UPPER_SNAKE_CASE

```
MAX_RETRY_COUNT, DEFAULT_PAGE_SIZE
```

**包名**: 全部小写

```
com.example.service, com.example.util
```

### 1.2 代码风格

- **缩进**: 4空格 (不用Tab)
- **行长度**: 最多120字符
- **大括号**: K&R风格

```java
public class UserService {
    public void process() {
        if (condition) {
            doSomething();
        }
    }
}
```

- **空格**: 运算符前后、逗号后加空格
- **空行**: 方法之间空一行，逻辑块之间可空行

### 1.3 注释规范

**必要注释**:

- 类和public方法的Javadoc
- 业务逻辑复杂处
- 特殊处理逻辑（hack、workaround）

**避免**:

- 过度注释（代码自解释）
- 过期注释
- 中文注释（团队统一除外）

### 1.4 日志规范

- 使用 SLF4J 记录日志（禁止直接使用 System.out.println）
- 核心操作需记录 INFO 级别日志，潜在问题需记录 WARN 级别，异常记录 ERROR 级别

---

## 二、设计原则与模式

### 2.1 SOLID原则

| 原则         | 描述          | 实践                              |
|------------|-------------|---------------------------------|
| **S** 单一职责 | 一个类只做一件事    | Controller只负责请求转发，Service处理业务逻辑 |
| **O** 开闭原则 | 对扩展开放，对修改关闭 | 使用接口+策略模式                       |
| **L** 里氏替换 | 子类能替换父类     | 继承层次合理，不要破坏is-a关系               |
| **I** 接口隔离 | 小的专门的接口     | 不需要强迫实现不需要的方法                   |
| **D** 依赖倒置 | 依赖抽象，不依赖具体  | 构造函数注入接口                        |

### 2.2 常用设计模式

**创建型**:

- 单例 (Singleton): 工具类、配置类、连接池
- 工厂方法 (Factory): 复杂对象创建
- 建造者 (Builder): 链式构建复杂对象

**结构型**:

- 装饰器 (Decorator): 动态增强功能
- 代理 (Proxy): 权限控制、远程调用
- 适配器 (Adapter): 接口转换

**行为型**:

- 策略 (Strategy): 多算法切换
- 责任链 (Chain of Responsibility): 多步骤处理
- 观察者 (Observer): 事件通知

### 2.3 DDD领域驱动设计

**核心概念**:

- **实体 (Entity)**: 有唯一标识的对象
- **值对象 (Value Object)**: 无标识的属性集合
- **聚合根 (Aggregate Root)**: 聚合的根实体
- **领域服务 (Domain Service)**: 跨实体的业务逻辑
- **仓库 (Repository)**: 持久化抽象

**分层架构**:

```
Controller -> Service -> Repository
     ↓           ↓          ↓
    DTO       Domain      DAO
          (Entity/VO)
```

---

## 三、工程实践

### 3.1 Git工作流

**分支命名**:

```
feature/user-login     # 新功能
fix/order-cancel       # Bug修复
hotfix/payment-error   # 紧急修复
refactor/order-service # 重构
```

**提交信息格式**:

```
<type>(<scope>): <subject>

<body>

<footer>
```

示例:

```
feat(order): 添加订单取消功能

支持订单状态为"待支付"时取消订单
- 新增 cancel 接口
- 退积分、退库存

Closes #123
```

**Type类型**: feat, fix, docs, style, refactor, test, chore

### 3.2 API设计

**RESTful规范**:

```
POST   /api/sys          # 只使用post作为唯一的请求
```

**响应格式**:

```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "id": 1,
    "name": "张三"
  },
  "success": true,
  "timestamp": 1767083407394
}
```

**错误处理**:

```json
{
  "code": 500,
  "msg": "参数错误",
  "data": {
    "field": "email",
    "reason": "邮箱格式不正确"
  },
  "success": false,
  "timestamp": 1767083407394
}
```

### 3.3 异常处理

```java
// 业务异常
public class MyException extends RuntimeException {
    private final String code;

    public MyException(String code, String message) {
        super(message);
        this.code = code;
    }
}

// 全局异常处理器
@RestControllerAdvice
public class MyExceptionHandler {

    @ExceptionHandler(MyException.class)
    public ResponseEntity handle(MyException e) {
        return ResponseEntity
                .status(HttpStatus.BAD_REQUEST)
                .body(new ErrorResponse(e.getCode(), e.getMessage()));
    }
}
```

### 3.4 事务管理

```java
// 必需事务
@Transactional(propagation = Propagation.REQUIRED)
public void process() {
    // 业务逻辑
}

// 嵌套事务
@Transactional(propagation = Propagation.NESTED)
public void subProcess() {
    // 保存点回滚
}

// 事务超时
@Transactional(timeout = 30)
public void longOperation() {
    // 30秒超时
}
```

**注意**:

- 避免长事务
- 事务范围要小
- 注意自调用问题（this调用不走代理）

### 3.5 接口调用

```java
// 远程调用要有超时和重试
@Retryable(value = Exception.class, maxAttempts = 3, backoff = @Backoff(delay = 1000))
@CircuitBreaker(name = "userService", fallbackMethod = "fallback")
public User getUser(Long id) {
    return userClient.getUser(id);
}

public User fallback(Long id) {
    // 降级处理
    return defaultUser;
}
```

### 3.6 测试实践

**单元测试**:

```java

@SpringBootTest
class UserServiceTest {

    @Autowired
    private UserService userService;

    @Test
    void testGetUserById() {
        User user = userService.getUserById(1L);
        Assertions.assertNotNull(user);
        Assertions.assertEquals("张三", user.getName());
    }
}
```

**测试分层**:

- **Controller**: Mock Service，验证请求处理
- **Service**: Mock Repository，验证业务逻辑
- **Repository**: 集成测试，验证SQL正确性

---

## 四、性能与安全

### 4.1 性能优化

- **数据库**: 索引、慢查询分析、避免N+1、避免在循环中执行数据库查询
- **缓存**: 多级缓存、缓存穿透/击穿/雪崩
- **异步**: 消息队列、线程池
- **连接池**: 数据库、HTTP、Redis连接池
- **方法复用**: 方法尽可能复用

### 4.2 安全实践

- **参数校验**: 使用@Valid + Bean Validation
- **SQL注入**: 使用PreparedStatement或MyBatis #{}
- **XSS**: 输出转义
- **敏感信息**: 不打印在日志中

```java
public class UserController {

    @PostMapping
    public Response create(@Valid @RequestBody UserCreateDTO dto) {
        // 参数自动校验
        return userService.create(dto);
    }
}

public class UserCreateDTO {
    @NotBlank(message = "用户名不能为空")
    @Size(min = 3, max = 20)
    private String username;

    @NotBlank
    @Email
    private String email;
}
```

---

## 使用方法

当你在开发中遇到以下问题时，可以调用此skill：

1. **代码规范疑问**: 命名、注释、日志等
2. **设计决策**: 是否需要设计模式、如何分层
3. **工程实践**: Git提交规范、API设计、异常处理
4. **性能安全**: 缓存、事务、安全加固

直接输入 `/java` 或描述你的问题即可获得针对性建议。