
# 同事嫌参数校验太丑，我直接掏出了更优雅的 SpEL Validator

某次代码评审，同事吐槽代码里的接口参数校验“又长又乱”：要么到处散落自定义校验器，要么硬塞 if/else 在服务层，跨字段、条件式、还要调用枚举/服务判断的场景尤其难看。

我当场就给大家介绍了一套开源的参数校验组件：**SpEL Validator**——基于 Spring 表达式，把“何时校验、校验什么”直接写进注解里。既不替代 Jakarta Validation，又能在复杂规则下写得自然、紧贴领域模型，规则和数据终于放到了一起。

> 开源地址：[https://github.com/stick-i/spel-validator](https://github.com/stick-i/spel-validator)
>
> 在线文档：[https://spel-validator.sticki.cn/](https://spel-validator.sticki.cn/)
>

## 1. 它解决了什么痛点
+ 多字段联合、条件式校验  
例：contentType 决定校验 audioContent 或 videoContent

```java
@NotNull
private int contentType;

@SpelNotNull(condition = "#this.contentType == 1", message = "语音内容不能为空")
private Object audioContent;

@SpelNotNull(condition = "#this.contentType == 2", message = "视频内容不能为空")
private Object videoContent;
```

+ 枚举合法性、复杂断言  
例：调用静态方法校验枚举存在性/数据长度等

```java
@SpelAssert(assertTrue = "T(cn.sticki.enums.UserStatusEnum).getByCode(#this.userStatus) != null",
    message = "用户状态不合法")
private Integer userStatus;
```

+ 需要透传 Spring Bean 的业务校验  
例：依赖数据库/缓存查询（需启用 `@EnableSpelValidatorBeanRegistrar`）

```java
@SpelAssert(assertTrue = "@userService.getById(#this.userId) != null", message = "用户不存在")
private Long userId;
```

**一句话概括：在注解里直接写表达式，优雅表达“何时校验、校验什么”，把过去要写成独立校验器/样板代码的逻辑，嵌入在域模型的注解中，让规则与数据天然贴合。**

## 2. 用法非常轻量
引入依赖（选一）：Spring Boot 2.x 用 javax 版；3.x 用 jakarta 版

+ javax（Spring Boot 2.x）

```xml
<dependency>
  <groupId>cn.sticki</groupId>
  <artifactId>spel-validator-javax</artifactId>
  <version>Latest Version</version>
</dependency>
```

+ jakarta（Spring Boot 3.x）

```xml
<dependency>
  <groupId>cn.sticki</groupId>
  <artifactId>spel-validator-jakarta</artifactId>
  <version>Latest Version</version>
</dependency>
```

两步开启

+ 方法参数上：`@Valid` 或 `@Validated`
+ 被校验类/字段/参数上：`@SpelValid`

示例：

```java
@RestController
@RequestMapping("/example")
public class ExampleController {
    @PostMapping("/simple")
    public Resp<Void> simple(@RequestBody @Valid SimpleParam p) { 
        return Resp.ok(null); 
    }
}

@Data
@SpelValid
public class SimpleParam {
    @NotNull 
    private Boolean switchAudio;

    @SpelNotNull(condition = "#this.switchAudio == true", message = "语音内容不能为空")
    private Object audioContent;
}
```

异常处理仍走 Spring/Jakarta 的主流通道（`BindException` / `MethodArgumentNotValidException`），原有代码风格不必改变。

## 3. 注解能力一览（与标准注解对标）
启动注解

+ `@SpelValid`：激活 SpEL 约束系统，可放在类、字段、方法参数、构造参数。

通用约束注解（均支持 `condition`、`message`、`group`）

+ 断言/空值/长度/数值/时间等
    - `@SpelAssert`（≈ `@AssertTrue`）
    - `@SpelNotNull` / `@SpelNull` / `@SpelNotEmpty` / `@SpelNotBlank`
    - `@SpelSize`（字符串/集合/Map/数组）
    - `@SpelMin` / `@SpelMax`（支持 `Number` 与 `CharSequence`；支持 `inclusive`）
    - `@SpelDigits`（整数/小数位数控制，支持 `Number` 与 `CharSequence`）
    - `@SpelPast` / `@SpelPastOrPresent` / `@SpelFuture` / `@SpelFutureOrPresent`（覆盖 JDK8 time、`Date/Calendar`、各 `Chrono*` 类型）

注解均内置国际化消息键，可按需覆盖。

## 4. 表达式写法与上下文
+ 基于 SpEL，支持算术、关系、逻辑、三元、成员访问、集合、空安全、空合并等操作。
+ 根对象为当前被验证的对象，直接用 `#this.fieldName` 引用其他字段。
+ 可调用静态方法：`T(全类名).method(...)`
+ 可调用 Spring Bean：`@beanName.method(...)`（需启用 `@EnableSpelValidatorBeanRegistrar`）

这使业务校验与领域知识复用更自然：调用 Enum 工具、Number/BigDecimal 处理、或领域服务，皆可在注解里完成。

## 5. 分组与条件：两层可组合
+ 条件开关（`condition`）  
任何约束注解都可带 `condition`，决定该注解是否生效。
+ 分组（`group` + `spelGroups`）  
`@SpelValid(spelGroups = "#this.type")` 启用分组语义；  
约束注解的 `group` 指定表达式数组。当 `spelGroups` 与 `group` 有交集（`equals`）时生效；`group` 为空则默认生效。  
这与 Jakarta 原生 `groups` 并行不冲突，前者服务于 SpEL 体系，后者仍可用于标准注解。

## 6. 国际化与消息插值
+ 多语言资源在 `spel-validator-constrain/src/main/resources/cn/sticki/spel/validator` 下提供默认翻译（zh、en、ja、ko 等）。
+ 你可以通过 `ResourceBundleMessageResolver.addBasenames("YourBundle")` 将自定义资源包加入优先队列，覆盖默认键。
+ 在注解 `message` 中使用 `{your.key}` 指定键；消息里如含 `{}` 或 `\` 需转义为 `\\{ \\}` 与 `\\`。

## 7. 性能与工程可用性
FAQ 的压测数据（基于示例项目、JDK8/SpringBoot 2.7）：

+ 预热后单次校验通常 **0~1ms**；
+ 强压 1500 ~ 9000 请求下，平均 **0.11 ~ 0.13ms/次**；
+ 解析 SpEL 占大头（33%~65%），但整体可接受，且还有优化空间。

工程性细节：

+ 字段与注解层缓存（`ConcurrentHashMap`）减少反射与注解扫描开销。
+ 错误统一落入 Jakarta Validation 体系，继续沿用原有全局异常处理与返回体包装。
+ 既可按 Web 入口触发，也可在服务内部直接 `validateObject`，便于单元测试或离线任务复用。

## 8. 和 Jakarta/Bean Validation 的关系
+ 不是替代，而是扩展：继续使用你熟悉的 `@NotNull/@Size` 等标准注解；当遇到“跨字段/条件式/复杂逻辑/需调用 Bean 或静态方法”时，用对应的 SpEL 注解来补齐空白。
+ 触发机制严格遵循规范：只有 `@Valid/@Validated + @SpelValid` 同时存在，SpEL 校验才会生效。
+ 分组保持一致心智模型：Jakarta 原生 `groups` 仍可用；SpEL 的 `spelGroups/group` 更贴近表达式驱动的动态分组需求。

## 9. 可扩展性：自定义你的约束
+ 自定义步骤与 Jakarta 的方式几乎一致：
    1. 定义注解并包含 `message/condition/group` 三属性；
    2. 实现 `SpelConstraintValidator<MyAnno>`，在 `isValid` 中取字段值并校验；可覆写 `supportType` 收窄类型；
    3. 在注解上用 `@SpelConstraint(validatedBy = XxxValidator.class)` 关联校验器。  
得益于 `SpelValidExecutor` 的统一驱动，你的自定义注解天然具备 `condition`、分组、I18n 能力。

## 10. 适用边界与注意事项
+ 能用标准注解解决的，尽量先用标准注解；SpEL 注解适合“条件式/跨字段/复杂逻辑”的增量增强。
+ 表达式应避免副作用：不要修改对象状态；`Validator` 接口也强调**线程安全**与**不可变**。
+ 启用调用 Spring Bean 时，记得添加 `@EnableSpelValidatorBeanRegistrar`。

## 结语
SpEL Validator 在不破坏现有校验体系的前提下，让“条件式/跨字段/复杂逻辑校验”回归到直观、声明式的写法，规则靠近数据、表达式靠近业务。它与 Jakarta Validation 既兼容又互补：你可以继续使用熟悉的 `@NotNull/@Size`，同时把“过去很难写顺”的场景交给 `@SpelNotNull/@SpelAssert/@SpelSize/...` 这套 SpEL 注解去解决。

若你的项目里存在以下任一特征，那么SpEL Validator便值得一试：

+ 参数规则取决于其他字段或分组；
+ 需要调用静态方法/领域服务完成复杂判断；
+ 希望在不破坏原有异常/返回体治理的前提下增强校验能力；
+ 追求更可读、更贴近业务的“校验即声明”。

项目文档已覆盖入门、注解索引、SpEL 要点、I18n、FAQ 与升级日志，源码结构清晰、测试与报告完善，接入成本低。在线文档：[https://spel-validator.sticki.cn/](https://spel-validator.sticki.cn/)

现在就把那些分散在各处的“如果…则校验…”逻辑收拢回模型，让校验像业务规则一样清晰可见吧。

