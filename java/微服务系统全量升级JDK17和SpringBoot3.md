# 文章信息

标题：踩了一堆坑，终于把微服务系统全面升级 JDK17 和 SpringBoot3了

简介：先说结论：非常推荐升级JDK17，成本低收益高。至于SpringBoot3.0，迁移成本比较高，坑也会比较多，但如果是新项目的话，还是可以试试的。

以下是正文

---

最近正在给自己的开源项目校园博客升级到 JDK17 以及 SpringBoot3，正好记录下升级和踩坑的过程，给大家提供一些解决方案的参考。

先说结论：非常推荐升级JDK17，成本低收益高。至于SpringBoot3.0，迁移成本比较高，坑也会比较多，但如果是新项目的话，还是可以试试的。

> PS：项目原来的版本是 JDK8 + SpringBoot2.6。

# 为什么要升级？

- JDK17和SpringBoot3也发布了一段时间了，自己对一些新特性也比较感兴趣，尤其是 Native Image 这个玩意。
- 自己手上刚好有校园博客这个项目，可以用来给进行升级，项目不复杂，但也算五脏俱全，全量升级既可以感受一下变化，也不会太费事。
- JDK17 是一个长期支持的版本（LTS），现在很多开源应用或者一些组件都在往这上面靠，并且大有一种最低支持 JDK17 的趋势。
- 自己在公司所接触到的项目也有一部分是使用的JDK17，并且整体有往这方面靠的趋势，新项目都会直接用JDK17。

总的来说就是 兴趣 + 资源 + 趋势。

# 升级有什么好处？

先来看看 JDK8 -> JDK17 的好处。

- ZGC垃圾回收器，性能提升
- 可以使用 var 作为局部变量类型推断标识符
- 一个文件中可以包含多个public类
- switch 使用起来更加简洁，可以不用再break了。
- instanceof 增强
- 增加不可修改的数据类 record（感觉还是 kotlin 的 data class 好用）
- Text Blocks文本块

实用性很强，非常舒服。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706509890355-1b7b1eb0-e20e-4ff6-8d24-af304a8b03f4.png#averageHue=%23e2e2e2&clientId=ue16165a1-5b7f-4&from=paste&height=192&id=u34b3b773&originHeight=240&originWidth=240&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=32378&status=done&style=none&taskId=u01359c0a-1816-4358-be4b-cc01af6e423&title=&width=192)

---

再看看 SpringBoot3.0 的一些新特性。

- 更好的支持 Native Image，使用 GraalVM 构建原生镜像，可以提供显著的内存和启动性能改进
- 升级到 Spring6.0
- 升级到 Spring Security 6.0
- ......

好吧，感觉上是不如 JDK17 要更有性价比，如果对 Native Image 兴趣不大的话，建议不要升级SpringBoot3.x，因为升级SpringBoot的成本可要比升级JDK高多了。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706507363322-ffba38c2-4ed1-45b4-8bbd-d9fa5b226190.png#averageHue=%23ebebeb&clientId=ue16165a1-5b7f-4&from=paste&height=192&id=ud30811a1&originHeight=240&originWidth=240&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=25781&status=done&style=none&taskId=u550aad5e-aa14-4b64-9a92-7ddb571a1d5&title=&width=192)

# 升级过程分享

> 以下的一切内容均基于我已有的项目【校园博客】进行升级和讲解，源码地址：[https://github.com/stick-i/scblogs](https://github.com/stick-i/scblogs)

既然一切都是基于JDK17的，那我们就先升级JDK吧！

## 升级JDK17

### 下载安装

安装JDK17，这里我直接在IDEA里面下载安装了，很方便：

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705221509198-21e82871-14b8-4c8c-b4e0-1e967e1ca526.png#averageHue=%232f3236&clientId=u7391aebc-5a00-4&from=paste&height=857&id=u0486312c&originHeight=1071&originWidth=1281&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=167755&status=done&style=none&taskId=u5b55ea36-e878-441f-9130-d44ef5063cb&title=&width=1024.8)

> 为了便于自己以后使用 Native Image，这里我直接下载了 GraalVM。

在IDEA中更新项目SDK和模块SDK：

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705221633751-0917113e-3213-4dac-b3fe-1482738c71ca.png#averageHue=%232c2e32&clientId=u7391aebc-5a00-4&from=paste&height=857&id=uB2gB&originHeight=1071&originWidth=1281&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=41885&status=done&style=none&taskId=ub5f35632-a9c3-4a0e-8f0d-c33ce6e664f&title=&width=1024.8)

### Maven构建
更新Maven编译配置：
```xml
<properties>
  <java.version>17</java.version>
  <maven.compiler.source>${java.version}</maven.compiler.source>
  <maven.compiler.target>${java.version}</maven.compiler.target>
</properties>
```
Maven重新打包下：

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705225578316-618b693a-6207-4565-9c57-e7e3724191e2.png#averageHue=%23232529&clientId=u7391aebc-5a00-4&from=paste&height=1112&id=ua76bac89&originHeight=1390&originWidth=2560&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=276995&status=done&style=none&taskId=u28b018f6-1c12-4c2c-a494-e42a06b3273&title=&width=2048)

> 这一步主要是为了更新下内部组件的 JDK 版本。

### 启动服务
测试下有没有其他问题，启动所有微服务项目：

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705225663832-85eec33f-8e14-4c4b-8b8c-61088833c998.png#averageHue=%2326292d&clientId=u7391aebc-5a00-4&from=paste&height=1112&id=udca46b06&originHeight=1390&originWidth=2560&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=502741&status=done&style=none&taskId=u991106e7-64c1-4289-b779-26295ce8e9b&title=&width=2048)

竟然一切正常，也可能与我的项目比较简单有一定的关系，所有服务都成功跑起来了。

### 更新Dockerfile

原来使用的基础镜像是 `java:8-alpine`，更新到了亚马逊的openjdk17版本`amazoncorretto:17-alpine`。

```dockerfile
# 设置JAVA版本
FROM amazoncorretto:17-alpine
# 指定存储卷, 任何向/tmp写入的信息都不会记录到容器存储层
VOLUME /tmp
# 拷贝运行JAR包
ARG JAR_FILE
ADD ${JAR_FILE} app.jar
# 设置JVM运行参数，限定内存大小，并设置时区为东八区
ENV JAVA_OPTS="\
-server \
-Xms256m \
-Xmx512m \
-XX:MetaspaceSize=256m \
-XX:MaxMetaspaceSize=512m \
-Duser.timezone=GMT+08 "
#空参数，方便创建容器时传参
ENV PARAMS=""
# 入口点， 执行JAVA运行命令
ENTRYPOINT ["sh","-c","java -jar $JAVA_OPTS /app.jar $PARAMS"]
```

### 源代码

看起来没什么问题了，先提交上JDK升级的代码，有需要的同学可以查看提交记录：
[build(all): 全量升级到jdk17，更新dockerfile和pom文件。 by stick-i · Pull Request #198 · stick-i/scblogs](https://github.com/stick-i/scblogs/pull/198/commits)

## 升级SpringBoot3.2

为什么选择直接升级到 SpringBoot3.2 而不是 3.0呢？

主要是我开始升级的时候，SpringBoot已经更新到3.2了，而此时的3.0的生命周期已经过半了，目前也还没有推出3.0以上的LTS版本，这么看来我以后总还是要升级的，倒不如现在一起弄了。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705501003759-10cb1109-abf9-4531-a60c-9b1b2a8a91fc.png#averageHue=%23fdfcf6&clientId=u6d3672d6-4852-4&from=paste&height=695&id=u7efc7115&originHeight=869&originWidth=1406&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=66793&status=done&style=stroke&taskId=u2c2d8dd1-4877-4188-842a-13a69324781&title=&width=1124.8)

### 升级pom依赖

跟SpringBoot相关的依赖还是比较多的，尤其是依赖了SpringCore的三方依赖，肯定也是要统一进行升级的。

截至到我写这篇文章的时间，SpringBoot的最新GA版本为 3.2.1：

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705501578823-bfcb986e-49e6-4535-b9a1-c9d967142125.png#averageHue=%23e8e8e8&clientId=u6d3672d6-4852-4&from=paste&height=101&id=uf5a7cfa3&originHeight=126&originWidth=474&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=7753&status=done&style=none&taskId=ub051b4c2-d98b-4e1a-b3c7-067a6926dae&title=&width=379.2)

我选择相信Spring，直接升级最新版！

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705501883699-fbbba153-b6a8-4329-b2b1-8d15cd35386f.png#averageHue=%237a8759&clientId=u6d3672d6-4852-4&from=paste&height=156&id=uc5143dd7&originHeight=240&originWidth=240&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=50132&status=done&style=none&taskId=u8cc62565-4df4-4858-8815-533150f98a4&title=&width=156)

对应的SpringCloud版本为2023.0.0

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705502236587-d8baf868-8782-4a95-aa10-7b20dc47729f.png#averageHue=%23fbfafa&clientId=u6d3672d6-4852-4&from=paste&height=665&id=u8fdef366&originHeight=831&originWidth=1321&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=100607&status=done&style=none&taskId=u3afbef53-6d2c-4697-bb72-c9165377db4&title=&width=1056.8)

其他主要依赖对应升级的情况：

| 依赖项 | 升级前版本 | 升级后版本 | 备注 |
| --- | --- | --- | --- |
| SpringBoot | 2.6.11 | 3.2.1 | 目前的最新版，要踩坑就踩最新的坑🤡 |
| SpringCloud | 2021.0.4 | 2023.0.0 | 对应SpringBoot3.2.x |
| SpringCloudAlibaba | 2021.0.4.0 | 2022.0.0 | 这个库还没出2023的版本，但是2022版也是基于SpringBoot3.0的，应该不会差太多 |
| Mybatis-Plus | 3.5.3.1 | 3.5.5 | **注意**：artifactId 从`mybatis-plus-boot-starter`改成了`mybatis-plus-spring-boot3-starter` |
| druid | 1.2.11 | 1.2.20 | **注意**：artifactId 从`druid-spring-boot-starter`改成了`druid-spring-boot-3-starter` |

对了，建议顺便升级下Maven。

### 解决依赖异常

修改完pom文件之后刷新一下本地依赖包，噢呦，一堆报错：

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705592104624-50fc22b2-2301-4329-b5c2-f9c5edff4cb9.png#averageHue=%2325272b&clientId=u82ede8d3-4fab-4&from=paste&height=470&id=u7890b1ea&originHeight=587&originWidth=2471&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=117889&status=done&style=none&taskId=u69d51f44-2deb-4663-9e66-c41e00bc794&title=&width=1976.8)

我看了一下，就两个问题，分别是 mysql-connector-java 和 javax.servlet-api 这两个包的版本没有被指定，所以Maven找不到对应的包。

为什么没有指定呢？之前也没有指定，但是之前没有报错，说明这两个包之前是有被 spring-boot-starter-parent 所管理的，但是现在它不管了。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705592481318-ad3691cd-9f63-4aaf-8fe3-ff0f992137ba.png#averageHue=%23c0c0c0&clientId=u82ede8d3-4fab-4&from=paste&height=176&id=u1e3a69f7&originHeight=220&originWidth=220&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=28521&status=done&style=none&taskId=u98fdd78b-1fe2-43cf-b311-a560cebb923&title=&width=176)

这得去看看SpringBoot3.0的迁移文档：[https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide)

#### MySQL

在网页里搜索关键字 MySQL，这不就来了：

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705592927121-bb2005fc-1fc1-48b5-9d26-cb83c26342b0.png#averageHue=%23fcfaf0&clientId=u82ede8d3-4fab-4&from=paste&height=113&id=ube809e99&originHeight=141&originWidth=1311&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=28277&status=done&style=none&taskId=ufd682339-f2a2-4b3d-828c-8968b6b7ab6&title=&width=1048.8)

就是说 `mysql:mysql-connector-java` 这个包的坐标改成了 `com.mysql:mysql-connector-j`，让我们更新的时候也顺带改一下。这个简单，全局搜索然后改一下就好了。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705593402677-c49d9b7b-ea2b-4705-90e8-95b8f559be96.png#averageHue=%231f2228&clientId=u82ede8d3-4fab-4&from=paste&height=148&id=u3acb7676&originHeight=185&originWidth=711&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=22212&status=done&style=none&taskId=ub9b0bfe3-afbf-4270-99e2-040d41cd401&title=&width=568.8)

一改完，版本继承的小图标就出来了，不错不错。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706102031492-dd7c0bbc-3918-40e7-8ac2-ae414d5c1b82.png#averageHue=%23eaeaea&clientId=uc47be51a-90f4-4&from=paste&height=192&id=uf5c0453e&originHeight=240&originWidth=240&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=20559&status=done&style=none&taskId=u64bb4618-e3a0-4087-a25f-7ca4c606291&title=&width=192)

#### javax -> jakarta

然后再搜一下关键字 javax，这不就又来了：

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1705592896937-e124d4df-8af0-4400-ac93-98e2f360b78f.png#averageHue=%23fdfcfa&clientId=u82ede8d3-4fab-4&from=paste&height=461&id=ueafe195f&originHeight=576&originWidth=1314&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=101196&status=done&style=none&taskId=u7450b2b1-84c8-4064-8dc9-43956bd038b&title=&width=1051.2)

这个就稍微麻烦一点了，不仅Maven坐标从 `jakarta.servlet:jakarta.servlet-api` 改成了 `javax.servlet:javax.servlet-api`，而且包名也从 javax.xxx.xxx 改成了 jakarta.xxx.xxx，所有导入了 javax 的包都得改。

先更新下pom文件：

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706103079579-42b43771-8577-4e7b-a6db-c4279e785d7a.png#averageHue=%23232a32&clientId=uc47be51a-90f4-4&from=paste&height=161&id=u6d354bb6&originHeight=201&originWidth=1546&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=28507&status=done&style=none&taskId=ub93e3876-0cb0-4b5d-a88e-3cbfd36d8d1&title=&width=1236.8)

然后再全局搜索 javax 替换下：

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706103167437-cca06e82-b85b-4b7c-95b5-8dc6e97f2be5.png#averageHue=%23292c31&clientId=uc47be51a-90f4-4&from=paste&height=669&id=ubbc6d4ff&originHeight=836&originWidth=1361&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=141104&status=done&style=none&taskId=uce0a3a31-650a-436f-911e-1929637748b&title=&width=1088.8)

我试过了，升级完后唯一出现问题的地方就只有一处，但也很容易修改：

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706104624050-4a1ec558-c5f7-4378-903f-b988105fa565.png#averageHue=%23202327&clientId=uc47be51a-90f4-4&from=paste&height=717&id=uc63794b7&originHeight=896&originWidth=2385&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=180687&status=done&style=none&taskId=u823570a8-1674-49bf-b5f1-2dce3d4ed44&title=&width=1908)

> ResponseStatusException 中没有 getStatus() 这个方法了，我使用HttpStatus.valueOf(statusException.getStatusCode().value()) 代替了原来的方法。

做完上面这些后，我的项目已经可以成功编译了，但还不能正常的跑起来。

### 配置文件属性迁移

SpringBoot3 更改了一些配置属性，例如：`spring.redis.host`改为了`spring.data.redis.host`。

这一变更几乎对所有项目都会有影响，要查看所有的变更项，可以在官方文档中进行查找：[https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Configuration-Changelog](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Configuration-Changelog)

但这太silly了，很显然官方也这么认为，所以给开发者提供了一个简单的迁移方法，**引入 **`**spring-boot-properties-migrator**`** **，它会帮你自动检测配置文件中需要修改的地方：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-properties-migrator</artifactId>
  <scope>runtime</scope>
</dependency>
```

> 配置文件属性迁移完毕后，记得删除这里添加的 spring-boot-properties-migrator 依赖。

然后运行项目，当然你的项目大概率是运行不起来的，但别着急，看看你的控制台输出，有没有像我这样的输出内容：

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706164943408-52406bd9-8d02-4c92-9788-ef82ecb89643.png#averageHue=%23242629&clientId=u84de0cbd-eb12-4&from=paste&height=1112&id=uc6a0c425&originHeight=1390&originWidth=2560&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=287664&status=done&style=none&taskId=u1f7daafc-2fe2-440d-8606-aca75cc1091&title=&width=2048)

上面的异常信息其实分为了两个部分，前面部分是需要进行修改的配置：

> The use of configuration keys that have been renamed was found in the environment:
> Property source 'bootstrapProperties-default-redis.yaml,DEFAULT_GROUP':
> **	Key: spring.redis.host**
> **		Replacement: spring.data.redis.host**
> **	Key: spring.redis.password**
> **		Replacement: spring.data.redis.password**
> **	Key: spring.redis.port**
> **		Replacement: spring.data.redis.port**
> Each configuration key has been temporarily mapped to its replacement for your convenience. To silence this warning, please update your configuration to use the new keys.

它也给出了重命名之后的key，这里直接对着描述把自己的配置文件改改就好了，比较简单。

---

后面部分是说有一些配置已经被弃用了，但是它也给出了弃用的原因：

> The use of configuration keys that are no longer supported was found in the environment:
> Property source 'bootstrapProperties-default-springmvc.yaml,DEFAULT_GROUP':
> 	**Key: spring.mvc.throw-exception-if-no-handler-found**
> **		Reason: DispatcherServlet property is deprecated for removal and should no longer need to be configured**
> Property source 'bootstrapProperties-default-redis.yaml,DEFAULT_GROUP':
> **	Key: spring.redis.lettuce.pool.max-active**
> **		Reason: none**
> **	Key: spring.redis.lettuce.pool.max-idle**
> **		Reason: none**
> **	Key: spring.redis.lettuce.pool.max-wait**
> **		Reason: none**
> **	Key: spring.redis.lettuce.pool.min-idle**
> **		Reason: none**
> Please refer to the release notes or reference guide for potential alternatives.

好吧，这里其实有点小坑，只有上面第一个Key给了弃用原因，说是DispatcherServlet属性已经被移除了。但是后面几个redis相关的Key都是没有给弃用原因的。

![](https://cdn.nlark.com/yuque/0/2024/gif/29587979/1706168478658-41aac5cc-be69-4594-afa0-3c8acbccb9f5.gif#averageHue=%23d9bc99&clientId=u84de0cbd-eb12-4&from=paste&height=179&id=u212a949c&originHeight=400&originWidth=412&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=ua5f7aee5-a163-41ba-91ba-741fe0e287a&title=&width=184)

既然这样，那我只能自己去官方文档里找了：[https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Configuration-Changelog](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Configuration-Changelog)

全局搜索下 `lettuce.pool`，你别说，还真让我找到了：

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706166069740-8bda7b31-b40d-46c5-925e-f6d5b36a441f.png#averageHue=%23e2c733&clientId=u84de0cbd-eb12-4&from=paste&height=1078&id=uaf23e39f&originHeight=1348&originWidth=2560&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=98623&status=done&style=none&taskId=ua8678f21-b0e0-4677-a3ec-ac48e6934ad&title=&width=2048)

明明也没有弃用，就是把redis前面加个data罢了，看来 `spring-boot-properties-migrator`也偶有瞎说的情况啊。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706165923396-4c675275-6210-4806-87ff-4bfd90ee5421.png#averageHue=%23c6c8c1&clientId=u84de0cbd-eb12-4&from=paste&height=247&id=u842e265d&originHeight=309&originWidth=350&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=75846&status=done&style=none&taskId=u02365b90-0666-4284-b4e6-a95f9986f22&title=&width=280)

> 再次提醒：配置文件属性迁移完毕后，记得删除之前添加的 spring-boot-properties-migrator 依赖。

### ES版本兼容

> **如果你的es客户端版本和es服务端版本一致（均为8.x），可以直接跳过这部分内容。**

项目里使用了 `spring-boot-starter-data-elasticsearch`，升级SpringBoot3.x 之后，这个依赖的版本也·提高了，对应ES的版本是8.x，而我服务器使用的ES版本是7.x，所以有一些不兼容的问题，启动时出现异常：

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706187239286-f117f9e5-5a3c-4b9d-bdef-8f718e491576.png#averageHue=%2323262a&clientId=u84de0cbd-eb12-4&from=paste&height=838&id=ub43635ac&originHeight=1048&originWidth=1993&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=320048&status=done&style=none&taskId=uafca1f16-9d4e-4527-84bc-42c7f608a55&title=&width=1594.4)

> Caused by: java.lang.RuntimeException: node: [http://xxxxxx,](http://es.if.sticki.cn/,) status: 200, [es/indices.exists] Missing [X-Elastic-Product] header. Please check that you are connecting to an Elasticsearch instance, and that any networking filters are preserving that header.

本来想通过降低 elasticsearch-rest-client 的版本来解决这个问题，但是降低之后又不能兼容 SpringBoot3 了，于是只能另辟蹊径了。

这个说起来比较麻烦，我在 stackoverflow 上找到一篇帖子，里面有对这个问题的描述，可以参考下：[https://stackoverflow.com/questions/71142680/co-elastic-clients-transport-transportexception-es-search-missing-x-elastic](https://stackoverflow.com/questions/71142680/co-elastic-clients-transport-transportexception-es-search-missing-x-elastic)

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706263621618-6f488349-cc87-4e24-9459-66f2268fe9b9.png#averageHue=%23fdfdfd&clientId=u84de0cbd-eb12-4&from=paste&height=414&id=u431c6e5d&originHeight=517&originWidth=916&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=84015&status=done&style=none&taskId=ub0db75f7-a783-46db-a6a4-c03801c8e75&title=&width=732.8)

它讲到了两个问题：

1. 客户端向服务端发送了未知的 Content-Type ，因此其请求被拒绝并返回 406（其实是请求头 compatible-with 不受支持）
2. 客户端需要验证 response 中是否具有 X-Elastic-Product=Elasticsearch 标头，但服务端并没有返回这个。

问题其实蛮清晰的，但是给出的解决方案让我不太满意，还需要自己重新去构建一个 RestClient，自己读取配置文件然后set进去，又得设置账号密码、又得解析Host的，这我可受不了。

![](https://cdn.nlark.com/yuque/0/2024/gif/29587979/1706264009959-512f0f84-e85c-4dac-b2b0-d97c996f3c34.gif#averageHue=%23bababa&clientId=u84de0cbd-eb12-4&from=paste&height=187&id=u68e570b4&originHeight=270&originWidth=270&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=udc4479a5-4136-420a-8fc1-389f14d42b0&title=&width=187)

于是经过我的一顿研究之后，我发现了 `RestClientBuilderCustomizer` 这个类：

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706282566249-cc4e5555-d08e-4d02-b8e7-0c5fc2cad581.png#averageHue=%231e2024&clientId=u84de0cbd-eb12-4&from=paste&height=792&id=u309a4503&originHeight=990&originWidth=1043&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=124925&status=done&style=none&taskId=u8f1610c1-1b3e-41cb-90d7-9c6cc772c97&title=&width=834.4)

> 翻译：回调接口，可以由希望通过RestClientBuilder进一步定制RestClient的bean实现，同时保留默认的自动配置。

只要用这个玩意，就可以在原有的 RestClient 基础上，进行一些定制化的操作，比如说解决上面那两个问题。于是乎，我就写了下面这一段代码：

```java
/**
 * Es 兼容性配置，添加响应头，兼容服务端版本
 * <p>
 * 如果客户端与服务端版本一致，可移除此配置。
 *
 * @author 阿杆
 * @version 1.0
 * @date 2024/1/25 22:29
 */
@Component
public class EsCompatibilityConfig implements RestClientBuilderCustomizer {

	@Override
	public void customize(RestClientBuilder builder) {
	}

	@Override
	public void customize(HttpAsyncClientBuilder builder) {
		// 添加响应头，兼容X-Elastic-Product
		HttpResponseInterceptor httpResponseInterceptor =
				(response, context) -> response.addHeader("X-Elastic-Product", "Elasticsearch");
		builder.addInterceptorLast(httpResponseInterceptor);
		// 自定义默认请求头，目的是禁用兼容性请求头 compatible-with
		builder.setDefaultHeaders(List.of(new BasicHeader(HttpHeaders.CONTENT_TYPE, ContentType.APPLICATION_JSON.toString())));
	}

}
```

这段代码很简单，在构建 RestClient 的过程中插入了一段代码，修改了请求头和响应头，用来兼容ES版本。只需要把这个类注入到Spring Bean中，就可以被 ElasticsearchRestClientConfigurations 自动加载。

![](https://cdn.nlark.com/yuque/0/2024/jpeg/29587979/1706413828444-4b81108b-e3c1-47a5-8fe8-c8c3fd19f175.jpeg#averageHue=%23c4c3c3&clientId=uf4b638b1-986c-4&from=paste&height=164&id=u272bba11&originHeight=330&originWidth=360&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u8a22b777-ba2f-42d9-a616-671a2f92969&title=&width=179)

### WARN trationDelegate$BeanPostProcessorChecker: is not eligible for getting processed....

如图所示，我的项目在升级到 SpringBoot3.x 后出现了大量的 WARN：

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706415434334-76f7ca2e-5347-4ed2-9ee8-45e5e213af32.png#averageHue=%23221737&clientId=uf4b638b1-986c-4&from=paste&height=1112&id=ub12a7136&originHeight=1390&originWidth=2560&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=519946&status=done&style=none&taskId=ubb946f72-bb06-46a8-bd4a-73e93396d9c&title=&width=2048)

虽然不影响项目运行，但是看得我很不爽，那只能想想办法看怎么解决掉这个warn了。

<img src="https://cdn.nlark.com/yuque/0/2024/jpeg/29587979/1706421029620-0eb46366-1423-47a5-8701-a9f3da9940d3.jpeg#averageHue=%23b1b1b1&clientId=uf4b638b1-986c-4&from=paste&height=252&id=u0ef5abce&originHeight=1317&originWidth=1000&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=ua180578a-c060-4871-9b52-4fbbc5063f6&title=&width=191" style="zoom:25%;" />

> 随机截取的一段异常信息，其他的也都差不多：
> 2024-01-28T11:58:45.587+08:00  WARN 1228 --- [user-server] [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.cloud.loadbalancer.config.LoadBalancerAutoConfiguration' of type [org.springframework.cloud.loadbalancer.config.LoadBalancerAutoConfiguration] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying). Is this bean getting eagerly injected into a currently created BeanPostProcessor [lbRestClientPostProcessor]? Check the corresponding BeanPostProcessor declaration and its dependencies.

注意看上面的异常信息，有任何跟我项目有关的东西吗？没有吧

那有任何跟依赖冲突有关的东西吗？看上去也没有

那这个异常什么时候才开始有的？Spring整体升级之后

好，那既然这样，我们可以大胆的认为这是一个SpringBoot的bug。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706502117840-9f90a7ea-0b42-4b35-8a33-daf4e67cbd1b.png#averageHue=%23cecece&clientId=ue16165a1-5b7f-4&from=paste&height=170&id=uc733347a&originHeight=291&originWidth=443&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=68898&status=done&style=none&taskId=u8dffe200-e212-4a2d-9bba-74263bf8ea7&title=&width=258.3999938964844)

一顿搜索之后，我在github上找到了这个 issue：[https://github.com/spring-cloud/spring-cloud-commons/issues/1315](https://github.com/spring-cloud/spring-cloud-commons/issues/1315)

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706420793867-485fd392-fbdc-4f7a-859c-735caabedf7f.png#averageHue=%23e9cf9a&clientId=uf4b638b1-986c-4&from=paste&height=1040&id=u70969af8&originHeight=1300&originWidth=2550&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=262649&status=done&style=none&taskId=uda13ba71-c0ab-4799-b48b-2280d591beb&title=&width=2040)

还真是Spring的bug，不过不是SpringBoot，而是SpringCloud的bug。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706421203190-bfd84548-a7f7-4e2f-9b84-a851b43e59e1.png#averageHue=%23fefefe&clientId=uf4b638b1-986c-4&from=paste&height=1040&id=u05c67f90&originHeight=1300&originWidth=2550&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=182640&status=done&style=none&taskId=ua64490dc-0156-4b8e-9e17-98a78181a04&title=&width=2040)

这位大佬也说了，将会在下一个版本（2023.0.1）中修复它，预计2月20日（今天是1月28日），不过他们会先发布新的Commons，用以修复这个bug。

在我看到这个issue的时候，新版的 SpringCloudCommons已经发布了：[https://spring.io/blog/2024/01/23/spring-cloud-commons-4-1-1-has-been-released](https://spring.io/blog/2024/01/23/spring-cloud-commons-4-1-1-has-been-released)

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706421480379-4c3d9aa3-4060-47e7-8c66-3295c11d71f5.png#averageHue=%23faf9f8&clientId=uf4b638b1-986c-4&from=paste&height=560&id=u72b6600f&originHeight=700&originWidth=1314&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=92677&status=done&style=none&taskId=ufa01bec8-cc4c-4175-8415-69a110217f4&title=&width=1051.2)

于是我对项目中的依赖进行替换，由于这个依赖是从其他Spring-Cloud的组件中自动继承过来的，所以我们只需要在依赖管理里面指定下版本就可以了。

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-commons</artifactId>
      <version>4.1.1</version>
      <scope>compile</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

添加完之后，果然没有再报warn了，之后等 SpringCloud2023.0.1 发布了，再做一下替换就好了。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706421824747-ea93583c-33bc-4af6-bb48-14b8fc07bbb1.png#averageHue=%23dddddd&clientId=uf4b638b1-986c-4&from=paste&height=139&id=u720ef8f9&originHeight=174&originWidth=198&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=26259&status=done&style=none&taskId=u0391c186-0565-4201-b30a-a492a664e77&title=&width=158.4)

### 更新自动注入文件

SpringBoot2.7时已经提出使用 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 代替 `spring.factories`：
[https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.7-Release-Notes#changes-to-auto-configuration](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.7-Release-Notes#changes-to-auto-configuration)

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706434585831-deaba714-6ce8-4a0d-8d18-95b4f83f9e5f.png#averageHue=%23fdfcfa&clientId=uf4b638b1-986c-4&from=paste&height=475&id=ua5fe319b&originHeight=594&originWidth=1244&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=116525&status=done&style=none&taskId=ud20969be-308c-40ee-9f6d-02ea0426e8f&title=&width=995.2)

我升级到 SpringBoot3.2 时，还是支持  spring.factories 的，但再过几个版本可能就不支持了，这边建议直接迁移下，这块几乎没什么成本的。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706434704560-84977b2f-94e0-4e58-bc9f-b1e8cb048dd3.png#averageHue=%2325272b&clientId=uf4b638b1-986c-4&from=paste&height=518&id=u4d50e091&originHeight=647&originWidth=1662&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=92569&status=done&style=none&taskId=ud963ce42-920e-4d66-ae30-2936e84e8b9&title=&width=1329.6)

### 源代码

这部分升级的改动有点多，以为已经搞好了，就提PR到main分支了，结果又蹦出来新的问题。

建议需要升级 SpringBoot3.x 的朋友，在看完这篇文章之后，还是再去把官方文档过一遍，看看有没有其他受影响的地方，这样稳妥一点。

代码已提交到GitHub：

- [https://github.com/stick-i/scblogs/pull/200](https://github.com/stick-i/scblogs/pull/200)
- [https://github.com/stick-i/scblogs/pull/201](https://github.com/stick-i/scblogs/pull/201)
- [https://github.com/stick-i/scblogs/pull/202](https://github.com/stick-i/scblogs/pull/202)

源码建议单个 commit 结合 commit message 来查看，这样会更有条理，而不是整个 pr 一起看。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706435590733-54ea09c0-322d-4e25-a493-e1c38aba90cb.png#averageHue=%23fefcfc&clientId=uf4b638b1-986c-4&from=paste&height=801&id=u8369ee32&originHeight=1001&originWidth=1635&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=150888&status=done&style=none&taskId=ub71a191b-8d50-4f88-9855-7a0e86ddbcc&title=&width=1308)

最后也附上一些我参考到的官方链接：

- SpringBoot3.0升级指南：[https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide)
- 3.2发布记录：[https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.2-Release-Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.2-Release-Notes)
- 3.0发布记录：[https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Release-Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Release-Notes)
- 2.7发布记录：[https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.7-Release-Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.7-Release-Notes)
- 3.0配置更新记录：[https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Configuration-Changelog](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Configuration-Changelog)
- 
# 后记

本来以为我这小项目简单升级下一两天就弄好了，结果前前后后搞了两周，尤其升级 SpringBoot 的时候，出了一顿问题，踩了不少坑。

看在作者这么认真的份上，建议关注趁早关注下，等我以后火了，在坐的各位就都是老粉了！

![454x370.png](https://cdn.nlark.com/yuque/0/2024/png/29587979/1706436823990-4378f9cf-5935-4d68-b6e4-a5789f250129.png#averageHue=%23dddddb&clientId=uf4b638b1-986c-4&from=paste&height=193&id=u4c138a8d&originHeight=370&originWidth=454&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=67351&status=done&style=none&taskId=uef5624a8-a0de-4b46-a0cc-a151ea8037f&title=&width=237.20001220703125)

