# 同事嫌参数校验太丑？我掏出 SpEL Validator + IDEA 插件，直接让他闭嘴

> 描述：还在用 if/else 做复杂参数校验？SpEL Validator 让你在注解里写表达式，配套 IDEA 插件还能自动补全、错误检查，本文带你实战体验。

标题图：

![2026-02-12-16-17-51](https://cdn.image.sticki.cn/blog/2026-02-12-16-17-51.png)

大家好，我是阿杆。

参数校验这活儿，写 Java 后端的同学肯定没少干。`@NotNull`、`@Size`、`@NotBlank` 一把梭，搞定大部分场景没啥问题。

但总有一些场景，这些基本注解是真搞不定的。
比如你想根据某个字段的值来决定另一个字段要不要校验，或者校验一个枚举值是不是合法的，再或者校验的时候需要查一下数据库……

遇到这种情况怎么办？要么在 Service 层写一堆 if/else，要么自定义 `ConstraintValidator`，代码散落在各处，改起来还容易漏。

所以今天来给大家安利一个我一直在维护的开源参数校验组件——**SpEL Validator**，基于 Spring 表达式，把校验规则直接写在注解里，而且最近还推出了配套的 **IDEA 插件**，开发体验直接拉满。

> 开源地址：[https://github.com/stick-i/spel-validator](https://github.com/stick-i/spel-validator)
>
> 在线文档：[https://spel-validator.sticki.cn/](https://spel-validator.sticki.cn/)

## SpEL Validator 是什么？

简单来说，**SpEL Validator 是一个基于 Spring Expression Language 的参数校验框架**，它不是要替代你熟悉的那些 `@NotNull`、`@NotBlank`，而是在 Jakarta Validation 的基础上做增强，把原来搞不定的那些场景给补上。

用过 `@NotNull` 的同学，上手 SpEL Validator 几乎零成本。直接看几个例子感受下：

**条件式校验**：根据 `switchAudio` 的值决定是否校验 `audioContent`，只有当 `condition` 的表达式返回true时才会开启校验

```java
@NotNull
private Boolean switchAudio;

@SpelNotNull(condition = "#this.switchAudio == true", message = "语音内容不能为空")
private Object audioContent;
```

**枚举值校验**：调用静态方法判断枚举值是否存在，这里是个断言条件，当 `assertTrue` 的表达式返回false时校验不通过

```java
@SpelAssert(assertTrue = " T(cn.sticki.enums.ExampleEnum).getByCode(#this.testEnum) != null ",
    message = "枚举值不合法")
private Integer testEnum;
```

**调用 Spring Bean**：直接在表达式中调用已注入的 Bean

```java
@SpelAssert(assertTrue = "@exampleService.getUser(#this.userId) != null", message = "用户不存在")
private Integer userId;
```

怎么样，够直观吧？不用再到处翻 Validator 了，打开类就能看到所有的校验规则。

不过，你可能也注意到了——这些 SpEL 表达式是写在字符串里的，这意味着它们没有语法高亮、没有字段补全、字段名拼错了也没人提醒你，得等到运行时才能发现。

但是，注意我要说但是了。

我开发了一个配套的 IDEA 插件：**SpEL Validator Support** 。装上之后，编写 SpEL 表达式也会有语法高亮、智能补全、Ctrl+Click 跳转、重命名同步、错误实时提醒，全都有。

> 安装方式：打开 IDEA → Settings → Plugins → 搜索 "SpEL Validator Support" → Install
>
> 插件地址：[JetBrains Marketplace](https://plugins.jetbrains.com/plugin/29693-spel-validator-support)

下面我用 [spel-validator-example](https://github.com/stick-i/spel-validator-example) 项目来演示框架 + 插件的完整开发体验。

## 实战演示

### 准备工作

引入依赖，Spring Boot 3.x 用 jakarta 版：

```xml
<dependency>
  <groupId>cn.sticki</groupId>
  <artifactId>spel-validator-jakarta</artifactId>
  <version>Latest Version</version>
</dependency>
```

Spring Boot 2.x 的同学把 `jakarta` 换成 `javax` 就行。

然后去 IDEA 的 Plugins 里搜索安装 **SpEL Validator Support** 插件，重启 IDEA。

### 案例一：条件式校验 + 智能补全

这是示例项目中最基础的一个场景——`SimpleExampleParamVo`：

```java
@Data
@SpelValid
public class SimpleExampleParamVo {

    @NotNull
    private Boolean switchAudio;

    @SpelNotNull(condition = "#this.switchAudio == true", message = "语音内容不能为空")
    private Object audioContent;

    @SpelAssert(assertTrue = " T(cn.sticki.validator.spel.example.enums.ExampleEnum).getByCode(#this.testEnum) != null ",
        message = "枚举值不合法")
    private Integer testEnum;

    @SpelAssert(assertTrue = "@exampleService.getUser(#this.userId) != null", message = "用户不存在")
    private Integer userId;
}
```

这个类里就包含了三种之前不好处理的校验场景：`switchAudio` 为 true 时才校验 `audioContent`；通过静态方法校验枚举值是否合法；调用 Spring Bean 查一下用户是否存在。全都写在注解里，没有一行 if/else。

装了插件之后，你会发现 SpEL 表达式有了语法高亮，字段名、方法调用、类型引用都有颜色区分。输入 `#this.` 的时候，IDEA 会自动弹出当前类的所有字段。

![2026-02-12-11-10-40](https://cdn.image.sticki.cn/blog/2026-02-12-11-10-40.png)

发起请求看一下校验效果：

```json
// switchAudio=true 但 audioContent 为空 → 校验不通过
{"switchAudio": true, "audioContent": null}
// 响应：{"code": 400, "message": "audioContent 语音内容不能为空"}

// switchAudio=false → audioContent 不校验，通过
{"switchAudio": false, "audioContent": null}
// 响应：{"code": 200, "message": "成功"}
```

### 案例二：分组校验 + 引用导航

再看另一个更有意思的例子，根据 `type` 分组校验不同的字段。

在某些内容比较多的表单场景下，可能会根据用户选择的不同类型，然后展示不同的字段，这时候就需要根据类型来分组校验不同的字段。

```java
@Data
@SpelValid(spelGroups = "#this.type")
public class GroupExampleParamVo {

    @NotNull
    @Pattern(regexp = "^text|audio$")
    private String type;

    @SpelNotNull(group = Group.TEXT)
    private Object textContent;

    @SpelNotNull(group = Group.AUDIO)
    private Object audioContent;

    @SpelNotNull // 未指定分组时，默认被校验
    private Integer other;

    static class Group {
        private static final String TEXT = "'text'";
        private static final String AUDIO = "'audio'";
    }
}
```

当 `type = "text"` 时，只有 `textContent` 和 `other` 会被校验；当 `type = "audio"` 时，只有 `audioContent` 和 `other` 会被校验。分组逻辑写在注解里就完事了，齐活。

有了插件，你可以 **Ctrl+Click** 表达式中的字段名，直接跳转到字段定义。也可以在字段上右键 **Find Usages**，查看它在哪些 SpEL 表达式中被引用了。

### 案例三：调用 Spring Bean

有时候校验逻辑不是简单的判空或比大小，可能需要查一下数据库才能确定参数合不合法。这时候就需要在表达式里调用 Spring Bean 了。

默认情况下，SpEL Validator 不支持调用 Spring Bean，需要先在启动类上加上 `@EnableSpelValidatorBeanRegistrar` 注解开启 Bean 支持：

```java
@EnableSpelValidatorBeanRegistrar
@SpringBootApplication
public class RestApplication {
    public static void main(String[] args) {
        SpringApplication.run(RestApplication.class, args);
    }
}
```

然后在你的 Service 里写个查询方法（这里简单模拟一下）：

```java
@Service
public class ExampleService {

    public User getUser(int id) {
        User user = new User();
        user.setId(id);
        user.setName("阿杆");
        return user;
    }
}
```

现在就可以在校验注解中直接引用这个 Bean 了：

```java
@SpelAssert(assertTrue = "@exampleService.getUser(#this.userId) != null", message = "用户不存在")
private Integer userId;
```

`@exampleService` 就是 Spring 容器里那个 Bean 的名字，SpEL 原生语法。

这里只是用简单例子演示一下调用方式。实际开发中你可以调用任何已注入的 Bean 方法来做校验，查缓存查数据库都行。当然了，校验里别搞太重的操作，查个缓存差不多得了，别搞成查了三张表还调了两个远程接口😅。

有了插件的加持，`@exampleService` 同样能被识别和高亮。如果你不小心写错了 Bean 名或字段名，插件也会给你标红提醒。

![2026-02-12-15-13-07](https://cdn.image.sticki.cn/blog/2026-02-12-15-13-07.png)

## 更多能力速览

除了上面演示的场景，SpEL Validator 还提供了丰富的约束注解：

| 注解 | 说明 | 对标 Jakarta |
|:---:|:---:|:---:|
| `@SpelAssert` | 逻辑断言 | — |
| `@SpelNotNull` / `@SpelNull` | 空值校验 | `@NotNull` / `@Null` |
| `@SpelNotEmpty` / `@SpelNotBlank` | 非空校验 | `@NotEmpty` / `@NotBlank` |
| `@SpelSize` | 长度校验 | `@Size` |
| `@SpelMin` / `@SpelMax` | 数值范围 | `@Min` / `@Max` |
| `@SpelDigits` | 数字精度 | `@Digits` |
| `@SpelFuture` / `@SpelPast` 等 | 时间校验 | `@Future` / `@Past` |

所有注解都支持 `condition`（条件开关）和 `group`（分组），也支持**国际化消息**，`message = "{key}"` 即可根据 `Accept-Language` 自动切换语言。

如果有自定义约束注解的需求，只要加上 `@SpelConstraint` 元注解，IDEA 插件同样能识别并提供智能支持。

## 最后

OK，差不多就介绍到这里了。总结起来就是：SpEL Validator 负责把校验能力补齐，IDEA 插件负责让你写表达式的时候不再抓瞎，两个搭一起用体验还是很舒服的。

如果你的项目里经常碰到跨字段校验、条件式校验这类场景，不妨试试看，接入成本很低的。

贴一下相关链接：

- GitHub：[https://github.com/stick-i/spel-validator](https://github.com/stick-i/spel-validator)
- 在线文档：[https://spel-validator.sticki.cn/](https://spel-validator.sticki.cn/)
- IDEA 插件：[JetBrains Marketplace](https://plugins.jetbrains.com/plugin/29693-spel-validator-support)
- 示例项目：[https://github.com/stick-i/spel-validator-example](https://github.com/stick-i/spel-validator-example)

如果觉得还不错，顺手给项目点个 ⭐️ 吧～ 你的 star 就是我继续肝下去的动力！
