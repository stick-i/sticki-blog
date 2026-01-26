标题：🤯我写了一套无敌的参数校验组件③ | SpEL Validator 之自定义约束注解

描述：这是一套全新的参数校验组件，并非造轮子。SpEL Validator 是一个强大的 Java 参数校验包，基于 SpEL 实现，扩展自 javax.validation 包，用于简化参数校验，几乎支持所有场景下的参数校验。



# 前言

SpEL Validator 是一个强大的 Java 参数校验包，基于 SpEL 实现，扩展自 javax.validation 包，用于简化参数校验，几乎支持所有场景下的参数校验。

GitHub地址：[https://github.com/stick-i/spel-validator](https://github.com/stick-i/spel-validator)



**这是一套全新的参数校验组件，并非造轮子**。看完本文你可能会觉得用不上或不屑于使用，但这玩意确实有应用场景，你不妨稍微留意一下，日后你总会发现有用得上的地方。

### Tips：此乃系列文章，当前为第③篇
本篇只讲如何进行自定义约束注解，对此框架不太了解的朋友可以先看看介绍文章，地址：

系列文章地址：



# 自定义约束注解
如果你使用过 `javax.validation` 的自定义约束注解，那么你会发现 `SpEL Validator` 的自定义约束注解几乎与 `javax.validation` 一致。

下面以 `@SpelNotBlank` 为例，展示如何实现自定义约束注解。



## 创建约束注解类
每个约束注释必须包含以下属性:

+ `String message() default "";` 用于指定约束校验失败时的错误消息。可以定义默认值，后续会支持多语言配置。
+ `String condition() default "";` 用于指定约束开启条件的SpEL表达式，只有满足条件的情况下才会对标记的元素进行校验。
+ `String[] group() default {};` 用于指定分组条件的SpEL表达式。

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@Repeatable(SpelNotBlank.List.class)
public @interface SpelNotBlank {

  String message() default "不能为空字符串";

  String condition() default "";

  String[] group() default {};

  @Target(FIELD)
  @Retention(RUNTIME)
  @Documented
  @interface List {

    SpelNotBlank[] value();

  }

}
```

## 创建约束验证器
创建类 `SpelNotBlankValidator`，实现 `SpelConstraintValidator<T>` 接口，其中泛型 `T` 为要校验的约束注解类，在这里是 `SpelNotBlank`。

实现 `isValid` 方法，校验逻辑在该方法中实现。

`isValid` 方法的参数说明如下：

+ `annotation`：当前约束注解的实例。
+ `obj`：当前校验的根对象。
+ `field`：当前校验的字段。

```java
public class SpelNotBlankValidator implements SpelConstraintValidator<SpelNotBlank> {

  @Override
  public FieldValidResult isValid(SpelNotBlank annotation, Object obj, Field field) throws IllegalAccessException {
    CharSequence fieldValue = (CharSequence) field.get(obj);
    return new FieldValidResult(StringUtils.hasText(fieldValue));
  }

}
```

一般情况下，只需要校验当前字段的值，通过 `field.get(obj)` 即可获取。

有些约束注解可能仅支持特定类型的字段，可以通过重写 `supportType()` 方法来指定支持的类型。默认情况下，支持所有类型。

`NotBlank`一般是只支持字符类型的，所以这里返回 `CharSequence.class`：

```java
public class SpelNotBlankValidator implements SpelConstraintValidator<SpelNotBlank> {

  @Override
  public FieldValidResult isValid(SpelNotBlank annotation, Object obj, Field field) throws IllegalAccessException {
    CharSequence fieldValue = (CharSequence) field.get(obj);
    return new FieldValidResult(StringUtils.hasText(fieldValue));
  }

  private static final Set<Class<?>> SUPPORT_TYPE = Collections.singleton(CharSequence.class);

  @Override
  public Set<Class<?>> supportType() {
    return SUPPORT_TYPE;
  }

}
```

## 关联注解和验证器
在 `SpelNotBlank` 注解上添加 `@SpelConstraint` 注解，指定该注解的验证器为 `SpelNotBlankValidator`。

```java
@Documented
@Retention(RUNTIME)
@Target(FIELD)
@Repeatable(SpelNotBlank.List.class)
@SpelConstraint(validatedBy = SpelNotBlankValidator.class) // 关联验证器
public @interface SpelNotBlank {
  // ...
}
```

## 使用自定义约束注解
完成上面的步骤，就可以在需要校验的字段上使用 `@SpelNotBlank` 注解了，使用方法和内置的约束注解没什么两样：

```java
@Data
@SpelValid // 添加启动注解
public class SimpleExampleParamVo {

  @NotNull
  private Boolean switch;

  @SpelNotBlank(condition = "#this.switch == true")
  private String audioContent;

}
```

这样你就成功创建了一个自定义约束注解啦！

# 总结
创建自定义约束注解的步骤：

1. 创建约束注解类，定义注解属性
2. 创建约束验证器，实现验证逻辑，并设置支持的类型
3. 关联注解类和验证器
4. 使用与测试

还是挺简单的，赶紧去试试吧！

OK兄弟们，如果你能看到这里，说明你对这个组件还算有点兴趣，不妨到GitHub给它点个star，或者给我点个关注，以便于持续关注项目更新动向，谢谢你的支持~~

GitHub地址：[https://github.com/stick-i/spel-validator](https://github.com/stick-i/spel-validator)

