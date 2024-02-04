## @Autowired 和 @Resource 的区别

### 默认注入方式不同

@Autowired 默认的注入方式为byType（根据类型进行匹配），也就是说会优先根据接口类型去匹配并注入 Bean （接口的实现类），如果想要指定名称，可以通过@Qualifier配合使用。
```java
@Autowired
private UserSafetyMapper userSafetyMapper;

// 或
@Autowired @Qualifier("userSafetyMapper")
private UserSafetyMapper userSafetyMapper;
```

@Resource 默认注入方式为 byName（根据命名进行匹配）。如果无法通过名称匹配到对应的实现类的话，注入方式会变为byType。

1. 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常；
2. 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常；
3. 如果指定了type，则从上下文中找到类型匹配的唯一bean进行装配，找不到或者找到多个，都会抛出异常；
4. 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配。

```java
// 1.默认注入方式
@Resource
private UserSafetyMapper userSafetyMapper;

// 2.指定注入方式
@Resource(name = "userSafetyMapper", type = UserSafetyMapper.class)
private UserSafetyMapper userSafetyMapper;
```

### 提供者不同

@Autowired 是spring提供的注解，@Resource 是JDK提供的注解

