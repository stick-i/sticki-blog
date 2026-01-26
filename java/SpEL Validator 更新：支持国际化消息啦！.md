<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/29587979/1744711014019-8f4d1f65-5307-4298-b051-71c4933b5bbd.png)

🤯我写了一套无敌的参数校验组件④ | 现已支持 i18n

---

[SpEL Validator](https://spel-validator.sticki.cn/)，一个基于 Spring 表达式的参数校验框架，用起来有点像 `jakarta.validation`，但语法更自由，表达力更强，支持各种复杂场景的参数校验。

这次 SpEL Validator 更新了一个实用又温柔的更新：**支持国际化消息（i18n）啦！**（当前版本v0.5.0-beta）

这意味着你可以：

+ 让校验报错信息根据用户语言自动切换；
+ 提供更符合业务场景的提示文案；
+ 允许使用者自定义国际化资源，完全不入侵业务。

这篇文章就带大家快速过一遍 i18n 的用法，也会顺便再介绍一下 SpEL Validator 的基本能力，欢迎新老朋友都来坐坐。

## SpEL Validator 是什么？
一句话介绍：**SpEL Validator 是一个基于 Spring Expression Language 的轻量参数校验框架。**

为什么要造这个轮子？因为用传统的 `jakarta.validation` 写复杂校验太麻烦了，比如你想校验一个字段的值要小于另一个字段，甚至是嵌套对象的某个字段，写起来就特别拧巴。

SpEL Validator 提供了一种更简洁的写法：

```java
@SpelValid
public class TimeRange {
    @SpelAssert(assertTrue = "#this.endTime > #this.startTime")
    private int startTime;
    private int endTime;
}
```

是不是一目了然？只要你能用 SpEL 写出来，就能校验。

## 特性介绍
（已经了解过 SpEL Validator 的同学可以跳过这部分）

+ 枚举值字段校验：

```java
@SpelAssert(assertTrue = " T(cn.sticki.enums.UserStatusEnum).getByCode(#this.userStatus) != null ", message = "用户状态不合法")
private Integer userStatus;
```

+ 多字段联合校验：

```java
@NotNull
private Integer contentType;

@SpelNotNull(condition = "#this.contentType == 1", message = "语音内容不能为空")
private Object audioContent;

@SpelNotNull(condition = "#this.contentType == 2", message = "视频内容不能为空")
private Object videoContent;
```

+ 复杂逻辑校验，调用静态方法：

```java
// 中文算两个字符，英文算一个字符，要求总长度不超过 10
// 调用外部静态方法进行校验
@SpelAssert(assertTrue = "T(cn.sticki.util.StringUtil).getLength(#this.userName) <= 10", message = "用户名长度不能超过10")
private String userName;
```

+ 调用 Spring Bean（需要使用 @EnableSpelValidatorBeanRegistrar 开启Spring Bean支持）：

```java
// 这里只是简单举例，实际开发中不建议这样判断用户是否存在
@SpelAssert(assertTrue = "@userService.getById(#this.userId) != null", message = "用户不存在")
private Long userId;
```



## 国际化消息怎么用？
在之前的版本中，`message` 字段，只能是写死的字符串，写的是什么就展示什么。

但从 **v0.5.0-beta** 起，`message` 支持国际化了！也就是说，你可以写成这样：

```java
@SpelValid
public class TimeRange {
    @SpelAssert(assertTrue = "#this.endTime > #this.startTime", 
                message = "{validation.timerange.invalid}")
    private int startTime;
    private int endTime;
}
```

其中 `validation.timerange.invalid` 是你的 message key。

### 内置资源文件
SpEL Validator 内置了一套国际化资源文件（从hibernate-validator的资源包里头copy过来的），支持多种语言，且默认启用。

### 设置区域信息
SpEL Validator 通过 Spring 提供的 `LocaleContextHolder` 来获取当前的区域设置。 默认情况下，它根据当前 request headers 的 `Accept-Language` 字段来确定区域。

如果你不使用这种方式来确定语言区域，可以通过 `LocaleContextHolder.setLocale()` 方法来手动更新区域（在校验执行之前更新）。

```java
LocaleContextHolder.setLocale(Locale.CHINA);
```

### 自定义消息资源
如果需要自定义消息资源，只需将你的资源包添加到 SpEL Validator 的资源包列表中，假设你的资源包名称为 `ValidationMessages`：

```java
ResourceBundleMessageResolver.addBasenames("ValidationMessages");
```

它会将你的资源包添加到原有资源包列表的最前面，这意味着如果存在相同的key，会优先使用最后添加的资源包文件。你可以利用这一特性，来覆盖框架内部提供的默认message。

### 其他
更详细的国际化消息配置方式可参考在线文档：[国际化指南](https://spel-validator.sticki.cn/guide/i18n.html)

## 结语
虽然这次更新功能不多，但对很多实际使用场景来说都是非常友好的一步。特别是做支持多语言系统的同学，应该能感受到这个小升级的价值。

如果你还没用过 SpEL Validator，欢迎看看[官方文档](https://spel-validator.sticki.cn/)，有清晰的指南和案例。

如果你已经在用了，那快试试看 i18n 的新能力，给用户一条更温柔的提示吧 🐣

---

如果你觉得这个框架还不错，也欢迎给项目点个 ⭐️：[https://github.com/stick-i/spel-validator](https://github.com/stick-i/spel-validator)

一起让参数校验这件小事，变得更轻松一点！

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2024/png/29587979/1727255576938-adc22a0b-99be-48ed-9cd0-b8caebae47b3.png)

