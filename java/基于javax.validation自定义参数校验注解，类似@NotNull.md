标题：基于javax.validation自定义参数校验注解，类似@NotNull

标题：那些奇奇怪怪的参数校验你是怎么处理的？我选择向 @NotNull 学习😏

描述：作为一名服务端开发程序员，接口的参数校验肯定是要经常写的，我们常用的参数校验方法，是通过 @NotNull、@Size、@NotBlank 等注解，然后配合 @Valid 注解来进行校验的。但这些注解无法完全满足我们的校验需求，这种情况下，我们可以选择自定义一个校验注解，来实现一些定制化的需求。

标题图：

![img](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706626699303-207e877a-0c56-4568-9df4-6cd27df96670.png)

![img](https://cdn.nlark.com/yuque/0/2024/jpeg/29587979/1706413828444-4b81108b-e3c1-47a5-8fe8-c8c3fd19f175.jpeg)

----

# 1. 前言

作为一名服务端开发程序员，接口的参数校验肯定是要经常写的，我们常用的参数校验方法，是通过 @NotNull、@Size、@NotBlank 等注解，然后配合 @Valid 注解来进行校验的，就像这样：

```java
/**
 * 用户注册业务类
 *
 * @author 阿杆
 */
@Data
public class UserRegisterParamVo {

    /**
	 * 用户名
	 */
    @NotNull
    @Pattern(regexp = "^[a-zA-Z0-9_-]{5,16}$", message = "用户名不合符要求")
    String username;

    /**
	 * 密码
	 */
    @NotNull
    @Pattern(regexp = "^[a-zA-Z0-9!@#$%^&*?_-]{5,20}$", message = "密码不合符要求")
    String password;

    /**
	 * 手机号
	 */
    @NotNull
    @Length(min = 11, max = 11, message = "手机号错误")
    String mobile;

    /**
	 * 邮箱
	 */
    @NotNull
    @Email
    String mail;

    /**
	 * 邮箱验证码
	 */
    @NotBlank(message = "邮箱验证码不能为空")
    String mailVerify;

}
```

这种方式很便捷，也很优雅，我们使用的这些注解，基本都来自 javax.validation 这个包下（后面改名为了jakarta）。

包内还提供了很多其他类型的校验注解：

![img](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705051761792-a697d06c-6059-4f10-8c07-92cdb9f5e876.png)

相信大家对这块也不陌生了，我就不过多介绍了，下面主要是想跟大家分享如何根据自己的需求来自定义一个这样的注解。

当然我知道可以通过很多其他的方式来解决参数校验的问题，但我觉得用注解的方式确实很舒服。

![img](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705052427491-e1beeb3d-18bb-414e-9c9b-2ccf46ac4536.png)

# 2. 为什么需要自定义

虽然 javax.validation 提供了一些常用的参数校验注解，但总不能完全满足我们的需求。

比如这样的验证需求：

- 有一个 List<String>，要求这个List不为空，且List当中的所有String都不为空字符串。

这样的需求很正常吧？但基于现有的这些注解，我想不到该如何进行校验。

很显然，validation不能满足我们的所有需求，所以 validation 提供了一种简便的扩展方法，方便开发者扩展自己的约束验证注解。

# 3. 如何自定义

javax.validation 提供了扩展的方法，用户完全可以自定义一个和 @NotNull 类似的注解，然后完全兼容原生的处理方式，很方便。

我们先来观察一下 NotNull 这个注解的源码：

![img](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705053153160-e0e1b501-240f-497f-a892-195458b60c0b.png)

注意到`@Constraint(validatedBy = { })`这个注解了吗？Constant这个单词的意思是“约束”，validatedBy的意思是“通过....验证”，很明显它和参数校验沾点关系。

我们来看下 @Constraint 的描述信息：

![img](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705053172105-48e4802a-ac83-476f-83b1-5dd6d85585b6.png)

一堆英文，没好好学英语的同学肯定看不懂吧？

![img](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705065662943-fea3448e-b9db-4dba-b71e-1554457f5e42.png)

我也看不懂😅。

给你翻译一下，@Constraint 上的描述大概是说，通过这个注解可以把一个你自己定义的注解标记为 Jakarta Bean 的验证约束，但是它要求你自己定义的那个注解必须包含三个属性，分别是：

- String message() default [...]; // 验证失败的错误信息
- Class<?>[] groups() default {}; // 用于分组校验
- Class<? extends Payload>[] payload() default {}; // 留着以后有用



然后 validatedBy 这个字段的意思是说，指定用于验证约束注解验证器，也就是具体的验证逻辑需要写到一个验证器类里，然后用这个参数去指定验证器，验证器必须是`ConstraintValidator`的实现类。



注意这里又出现新目标了，来看看这个 ConstraintValidator 是什么：

![img](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705057573502-5bcfeaf2-dc38-4fe6-8ff5-a9dd35f7a20c.png)

大概意思是说，这个接口是用来定义约束注解的具体约束逻辑的，它的泛型参数为 <A extends Annotation, T>，其中 A 表示这个验证器将要用来验证的注解类，T 表示这个验证器支持验证哪些类型。

然后 initialize 用来初始化验证器，其实是提供给你获取注解参数的。

核心方法是 isValid ，它是用来实现具体验证逻辑的，其中参数 value 就是被验证的那个对象的值，验证通过就返回 true，失败就返回 false，还是很容易理解的。

![img](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705065746697-aca51789-2763-48dc-a4a5-7dc0164b75f2.png)

好，那么好，看到这一步，其实已经基本明确了我们该如何自定义一个约束注解类。

如果你还没看懂，那我再带你完整的梳理一下整个步骤：

1. 定义一个约束注解，比如 @NotNull。
2. 在 @NotNull 上添加 @Constraint 这个注解，参数 validatedBy 暂时先不设置。
3. 按照 @Constraint 的要求，给 @NotNull 添加三个字段：message、group、payload。
4. 定义一个类，实现 ConstraintValidator 这个接口，我们把这个实现类叫 NotNullValidator。
5. 在 NotNullValidator 中实现 isValid 方法，然后在方法里判断下参数 value 是否为null。
6. 回到 @NotNull 注解这里，把 validatedBy 的参数设置为 NotNullValidator.class。
7. 使用 @NotNull 注解进行测试。

这下看懂了吧，如果还没看懂，那只能说明我的表达有问题，不用客气，在评论区吐槽就完事了。

![img](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705065713441-0405739a-0105-4246-86a5-ef897c5e8189.png)

# 4. 如何实现

先定义一个注解 `NotBlankFields`，我们希望它能满足上面提出的需求。

> - 有一个 List<String>，要求这个List不为空，且List当中的所有String都不为空字符串。

我们给 NotBlankFields 加上 @Constraint 这个注解，并且按照 @Constraint 的要求，添加三个必须的属性，弄完之后是这样：

```java
/**
 * @author 阿杆
 * @date 2024/1/16
 */
@Target(FIELD)
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = {})
public @interface NotBlankFields {

    String message() default "所有字段均不能为空";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

}
```

很简单的一个注解。

接下来我们需要实现一个校验器类，用于验证这个约束注解。

类名就叫 `NotBlankFieldsValidator`，需要实现 `ConstraintValidator`这个接口，这样它就能够被识别为一个校验器类了。

```java
/**
 * @author 阿杆
 * @date 2024/1/16
 */
public class NotBlankFieldsValidator implements ConstraintValidator<NotBlankFields, List<String>> {

    @Override
    public void initialize(NotBlankFields constraintAnnotation) {
    }

    @Override
    public boolean isValid(List<String> value, ConstraintValidatorContext context) {
        return false;
    }
}
```

> 这个 initialize 方法并不是一定要重写的，它存在的目的是便于你获取到注解参数，同时让你可以做一些初始化的操作。

注意泛型参数的第一个值为被验证的注解，第二个值为该约束所支持数据类型，这里我们是要校验 List<String>类型，所以第二个泛型参数是这个。

核心的校验逻辑需要在 isValid() 方法中进行实现，只需要判断列表中每个字符串是否为空就好了，很简单：

```java
@Override
public boolean isValid(List<String> value, ConstraintValidatorContext context) {
    if (value == null || value.isEmpty()) {
        return false;
    }
    for (String s : value) {
        if (s == null || s.isBlank()) {
            return false;
        }
    }
    return true;
}
```

最后还有一步，可千万不能忘了需要用 validatedBy 来指定约束注解的验证器。我们回到 NotBlankFields ，把参数补上：

![img](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705368979430-bc902bba-d42e-4d26-af4c-800cfe7e7252.png)

> 这里的代码我提交到了GitHub上，有需要查看源码的朋友可以查看：https://github.com/stick-i/scblogs/pull/204

# 5. 使用效果

写了一个小demo：

```java
@Slf4j
@RestController
@RequestMapping("/register")
@Validated
public class TestController {

    @PostMapping("/test")
    public RestResult<Object> test(@Valid @RequestBody TestParamVo testParamVo) {
        log.info("testParamVo->{}", testParamVo);
        return RestResult.ok();
    }

}

@Data
public class TestParamVo {

    /**
	 * 测试 @NotBlankFields 注解
	 */
    @NotBlankFields
    List<String> testList;

}
```

在使用上和那些NotNull、NotBlank都是一样的。

那我们来调用一下，看看校验是否生效。

## 5.1. 测试

参数值为null的情况下，校验不通过：

![img](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706623980456-09a9edc6-9c96-4b8b-aabe-be1d6ebc5e7a.png)

参数值为空数组的情况下，校验不通过：

![img](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706623998004-48f6babf-1c24-4d06-a5b6-e6017ebb8efe.png)

参数值为三个空字符串组成的非空列表，校验不通过：

![img](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706624087008-c1f4051e-376d-4071-a216-293ebbff96b5.png)

参数值中部分字符串为空字符串的情况，校验不通过：

![img](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706624148503-fe236681-4aa4-45f5-b4df-a3255be40f53.png)

参数正常的情况下，校验通过：

![img](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706626074396-cad3b6ae-2fef-4a38-8c0f-0457f6d15814.png)

所有测试用例都通过了，这样的话，就已经满足我们前面所提出的条件了、

> PS：demo和测试都是基于我已有的项目上进行的，返回值包装、异常处理等都是在项目中已有的了。
>
> 项目名【校园博客】，开源在GitHub上，地址：https://github.com/stick-i/scblogs

# 6. 扩展

## 6.1. 获取注解内的参数

假设我定义的注解是这样的：

```java
@Target(FIELD)
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = {NotBlankFieldsValidator.class})
public @interface NotBlankFields {

    /**
     * 列表中的值是否可以为null
     */
    boolean nullable() default false;

    String message() default "所有字段均不能为空";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

}
```

注意我多加了一个 nullable 字段，提供了一个可选的参数。

那么对应的验证器 NotBlankFieldsValidator 也需要进行修改：

```java
/**
 * @author 阿杆
 * @date 2024/1/16
 */
public class NotBlankFieldsValidator implements ConstraintValidator<NotBlankFields, List<String>> {

    /**
     * 是否可以为null
     */
    private boolean nullable;

    @Override
    public void initialize(NotBlankFields constraintAnnotation) {
        this.nullable = constraintAnnotation.nullable();
    }

    @Override
    public boolean isValid(List<String> value, ConstraintValidatorContext context) {
        if (value == null || value.isEmpty()) {
            return false;
        }
        for (String s : value) {
            if ((!nullable && s == null) || s.isBlank()) {
                return false;
            }
        }
        return true;
    }
}
```

想必看完这个例子，你就已经懂了，那我也就不多说了。



## 6.2. 根据不同情况修改错误信息

这个其实需要用到 isValid(T value, ConstraintValidatorContext context) 方法中的第二个参数，`ConstraintValidatorContext`（译为“约束验证器上下文”）。

来，小亮，给他整个活 ！

> *ዽ ጿጿ ኈ ቼ ዽ ጿጿ ኈ ቼ ዽ ጿጿ ኈ ቼ ዽ ጿጿ ኈ ዽ ጿ*

```java
@Override
public boolean isValid(List<String> value, ConstraintValidatorContext context) {
    if (value == null || value.isEmpty()) {
        return false;
    }
    // 禁用默认的message
    context.disableDefaultConstraintViolation();
    for (String s : value) {
        if (s == null) {
            // 添加错误提示
            context.buildConstraintViolationWithTemplate("字段不能为null").addConstraintViolation();
            return false;
        }
        if (s.isBlank()) {
            // 添加错误提示
            context.buildConstraintViolationWithTemplate("字段不能为空值").addConstraintViolation();
            return false;
        }
    }
    return true;
}
```



# 7. 尾声

好好好，这么简单的东西，硬是让我吹了这么长的篇幅！

希望你学会了之后，明天就去公司写一个，写完之后给同事亮一手！

![img](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706626699303-207e877a-0c56-4568-9df4-6cd27df96670.png)

看在作者这么认真的份上，建议关注趁早关注下，等我以后火了，在坐的各位就都是老粉了！

![img](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706436823990-4378f9cf-5935-4d68-b6e4-a5789f250129.png)