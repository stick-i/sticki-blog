
# 前言
学习Spring框架的过程中，跟着老师总结了以下几中Bean的加载方式，不过老师说还有其它的加载方式，以下八种并不是全部，但也足以用来做很多事情了。

特此记录，以便于给大家一起学习，也便于自己日后回头来看。


# Bean八种的加载方式

## 1.xml+\<bean/>

被配置的bean需要有无参数的构造函数

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	<!-- xml方式声明自己开发的bean -->
	<bean id="user" class="cn.sticki.blog.pojo.domain.User" />
	<!-- xml方式声明第三方bean -->
	<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"/>
</beans>
```


## 2.xml:context+注解(@Component+4个@Bean)

- 使用组件扫描，指定加载bean的位置，spring会自动扫描这个包下的文件。

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
         http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd
         http://www.springframework.org/schema/context
         http://www.springframework.org/schema/context/spring-context.xsd
      ">
  
      <!-- 组件扫描，指定加载bean的位置 -->
      <context:component-scan base-package="cn.sticki.bean,cn.sticki.config"/>
  </beans>
  ```

- 然后在需要被加载的类名上添加@Component注解。也可以使用@Controller、@Service、@Repository定义bean。

  ```java
  @Service
  publice class UserServiceImpl implements UserService {
  }
  ```

- 使用@Bean定义第三方bean，并将所在类定位为配置类或Bean

  ```java
  @Configuration  // 或使用@Component
  public class DBConfig {
      @Bean
      public DruidDataSource dataSource(){
          DruidDataSource ds = new DruidDataSource();
          return ds;
      } 
  }
  ```


## 3.配置类+扫描+注解(@Component+4个@Bean)

使用 `AnnotationConfigApplicationContext(SpringConfig.class);` 来获取 `ApplicationContext`

  ```java
  @ComponentScan({"cn.sticki.bean","cn.sticki.config"})
  public class SpringConfig {
      @Bean
      public DogFactoryBean dog(){
          return new DogFactoryBean();
      }
  }
  ```

和上面的第二种有点类似，就是包的扫描方式有所改变。


### 3.1FactoryBean接口

初始化实现FactoryBean接口的类，实现对bean加载到容器之前的批处理操作。

```java
public class DogFactoryBean implements FactoryBean<Dog> {
    @Override
    public Dog getObject() throws Exception {
        return new Dog();
    }
    @Override
    public Class<?> getObjectType() {
        return Dog.class;
    }
}
```

在下面的bean中，显示的表示为配置`DogFactoryBean` ，但实际上配置的是 `Dog` 。

```java
@Component
public class SpringConfig {
    @Bean
    public DogFactoryBean dog(){
        return new DogFactoryBean();
    }
}
```



### 3.2@ImportResource注解

用于加载配置类并加载配置文件（系统迁移）

```java
@Configuration
@ComponentScan("cn.sticki.bean")
@ImportResource("applicationContext.xml")
public class SpringConfig {
}
```



### 3.3proxyBeanMethods属性

使用 `proxyBeanMethods = true`  可以保障调用此类中的方法得到的对象是从容器中获取的，而不是重新创建的，但要求必须是通过此类调用方法获得的bean。

```java
@Configuration(proxyBeanMethods = true)
public class SpringConfig {
    @Bean
    public User user() {
        System.out.println("user init...");
        return new User();
    }
}
```

```java
public class AppObject {
    public static void main() {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
        SpringConfig config = ctx.getBean("Config", SpringConfig.class);
        // 两次获取的是同一个对象
        config.user();
        config.user();
    }
}
```



## 4.@Import导入bean的类

使用@Import注解导入要注入的bean对应的字节码

```java
@Import(User.class)
public class SpringConfig {
}
```

而被导入的bean无需使用注解声明为bean

```java
public class User{
}
```

这种形式可以有效的降低源代码与spring技术的耦合度（无侵入），在spring技术底层及诸多框架的整合中大量使用。

使用这种方法可以加在配置类，且也可以加在配置类当中的bean。



## 5.AnnotationConfigApplicationContext调用register方法

在容器初始化完毕后使用容器对象手动注入bean

```java
public class App {
    public static void main() {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
        ctx.register(User.class);
        // 打印容器中当前的所有bean
        String[] names = ctx.getBeanDefinitionNames();
        for (String name: names) {
            System.out.println(name);
        }
    }
}
```

必须在容器初始化之后才能使用这种方法。如果重复加载同一个bean，后面加载的会覆盖前面加载的。



## 6.@Import导入ImportSelector接口

导入实现了ImportSelector接口的类，实现对导入源的编程式处理

```java
public class MyImportSelector implements ImportSelector {
    public String selectImports(AnnotationMetadata metadata) {
        // 使用metadata可以获取到导入该类的元类的大量属性，通过对这些属性进行判断，可以达到动态注入bean的效果
        boolean flag = metadata.hasAnnotation("org.springframework.context.annotation.Import");
        if(flag) {
            return new String[]{"cn.sticki.pojo.User"};
        }
        return new String[]{"cn.sticki.pojo.Dog"};
    }
}
```

调用处：

```java
@Import(MyImportSelector.class)
public class SpringConfig {
}
```



## 7.@Import导入ImportBeanDefinitionRegistrar接口

导入实现了ImportBeanDefinitionRegistrar接口的类，通过BeanDefinition的注册器注册实名bean，实现对容器中bean的裁定，例如对现有bean的覆盖，进而达成不修改源代码的情况下更换实现的效果。

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    public String registrarBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        // 使用元数据去做判定，然后再决定要注入哪些bean
        BeanDefinition beanDefinition = BeanDefinitionBuilder.rootBeanDefinition(User.class).getBeanDefinition();
        registry.registerBeanDefinition("user",beanDefinition);
    }
}
```

调用处和上面第六种方式差不多。



## 8.@Import导入BeanDefinitionRegistryPostProcessor接口

导入实现了BeanDefinitionRegistryPostProcessor接口的类，通过BeanDefinition的注册器注册实名bean，实现对容器中bean的最终裁定。其在@Import中的加载顺序为最后一个加载，可以用来做bean覆盖的最终裁定。

```java
public class MyPostProcessor implements BeanDefinitionRegistryPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        // 注意这里注册的是BookServiceImpl4
        BeanDefinition beanDefinition = BeanDefinitionBuilder.rootBeanDefinition(BookServiceImpl4.class).getBeanDefinition();
        registry.registerBeanDefinition("bookService",beanDefinition);
    }
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
    }
}
```

调用处：

```java
// 按照先后顺序加载，但 MyPostProcessor.class 最后才加载
@Import(BookServiceImpl1.class,MyPostProcessor.class, BookServiceImpl.class, MyImportSelector.class)
public class SpringConfig {
}
```
# 后记
本篇文章学习自B站黑马程序员，课程链接：https://www.bilibili.com/video/BV15b4y1a7yG，本本的内容主要来自 `原理篇139 - 原理篇148`，讲述的是bean加载方式。有兴趣的同学也可以去看视频学习。