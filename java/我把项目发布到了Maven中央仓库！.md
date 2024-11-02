# 我把项目发布到了Maven中央仓库

标题：

- 我把项目发布到 Maven 仓库了，给大伙分享下经验~

- 我总结了一套从 0 到 1 将项目发布到 Maven 中央仓库的教程，速速收藏！

- Maven中央仓库发布全攻略：从0到1，速收藏！

描述：

想让你的Java项目被更多人使用吗？快来学习如何将其发布至Maven中央仓库吧！虽然过程不简单，但我已经帮你完整的总结好啦~ 你直接对着抄就行~

标题图：

![maven](http://cdn.image.sticki.cn/202405071739761.jpg)



# 前言

大家好，我叫阿杆，不叫阿轩

最近写了一个参数校验组件，名字叫 `spel-validator`，是基于 `javax.validation` 的一个扩展，目的是简化参数校验。

我把项目开源到了GitHub https://github.com/stick-i/spel-validator ，但为了便于开发者使用，我决定将其发布到Maven中央仓库。

本文记录了我将项目发布到Maven中央仓库的完整过程，发布时间为 2024年5月5日，希望对你有所帮助。

> 2024 年 3 月 12 日起，官方调整了Maven包的发布方式，以前的老教程可能有些不适用了，如果你有发布 jar 包的需求，参考我这篇文章基本不会有什么问题。
>
> ![官方通知](http://cdn.image.sticki.cn/202405071628427.png)

大致操作步骤如下：
1. 配置sonatype账号
   1. 注册账号
   2. 创建命名空间
   3. 获取 User Token
2. 配置GPG
   1. 安装GPG
   2. 生成密钥
   3. 发送公钥
3. 配置pom文件
4. 发布项目


# 配置sonatype账号

## 注册账号

进入官网：https://central.sonatype.com/

点击右上角 Sign In，然后点击下面的 Sign up，填写账号、密码、邮箱，注册自己的账号。

![注册sonatype账号](http://cdn.image.sticki.cn/202405062215763.gif)

## 创建命名空间

登录自己的账号之后，进入网站主页，点击右上角的 `Publish` 按钮，进入 `Namespace` 模块，然后点击 `Add Namespace` 按钮。

命名空间具有唯一性，必须是属于你且没有被其他人使用的，这里我使用了我的域名 `cn.sticki` 作为命名空间。

> 命名空间对应着 Maven 依赖的 groupId。
>
> 如果你直接通过GitHub注册的sonatype，应该会直接给你授权你账号的github命名空间。
>
> 如果你的命名空间注册失败，那可能是被其他人注册了，但官网上没看到写解决方案，如果你遇到了这种情况，可以网上搜一下解决方案。

![sonatype-新建命名空间](http://cdn.image.sticki.cn/202405062223890.gif)

点击提交后，需要验证命名空间。

![验证域名](http://cdn.image.sticki.cn/202405062232679.png)

个人域名是通过 TXT 记录验证的，需要在域名的 DNS 解析中添加一条 TXT 记录。

![添加域名解析记录](http://cdn.image.sticki.cn/202405062233031.png)

如果你没有域名，也可以使用 GitHub 或者 GitLab、Gitee 的用户名作为命名空间：

- GitHub io.github.username
- GitLab io.gitlab.username
- Gitee io.gitee.username

比如我使用 GitHub 来作为我的命名空间，我的 GitHub 用户名是 `stick-i`，那么命名空间就写 `io.github.stick-i`。

![验证GitHub账号](http://cdn.image.sticki.cn/202405062236080.png)

GitHub的验证方式是通过让你新建仓库来验证的，你只需要根据它指定的名称新建一个仓库就好了。

配置好验证所需要的条件后，点击 `confirm` 按钮，然后等待验证通过，一般几秒钟就可以了。

## 获取 User Token

点击右上角的view account -> 点击Generate User Token -> 点击ok。

![sonatype-生成UserToken](http://cdn.image.sticki.cn/202405062252681.gif)

然后会得到一个这样的数据：

```xml
<server>
	<id>${server}</id>
	<username>xxxxxxx</username>
	<password>xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx</password>
</server>
```

> 这个是你上传jar包时，用来验证你的身份的，不要泄露给别人。

把这段数据复制下来，粘贴到 Maven 的 setting.xml 里面，把 `${server}` 改成你自定义的内容，这个自定义的内容要记下来，后面还要用到的，这里我改成了 `sonatype-sticki`。

![setting.xml](http://cdn.image.sticki.cn/202405071143359.png)

到这里 sonatype 账号就配置完毕了！



# 配置GPG

> 发布到Maven仓库中的所有文件都要使用GPG签名，以保障完整性。因此，我们需要在本地安装并配置GPG。

## 安装GPG

进入官网：https://gnupg.org/download/index.html

下载自己需要的版本，windows的同学跟我的操作一样进行下载就好了：

![下载GPG](http://cdn.image.sticki.cn/202405071154803.gif)

下载完直接安装，无脑下一步就好了，除了最后的文件路径要修改一下，前面的都不用管。

组件选择默认的就可以了，咱也没那么多需求，能用就行😋。

![](http://cdn.image.sticki.cn/202405042209420.png)

安装完之后，windows版会自动帮我们添加系统环境变量，我们随便找个地方打开cmd窗口就可以使用了（uTools的cmd除外）。

如果用不了的话，找到GPG的安装目录，目录的旁边还有一个 `GnuPG` 文件夹，然后进入 GnuPG 文件夹下的bin目录，在这些目录下打开 cmd 窗口。

![我的bin目录](http://cdn.image.sticki.cn/202405071203948.png)

可以使用 `gpg --version` 来验证是否安装成功。

## 生成密钥

命令行输入`gpg --gen-key`

然后根据提示依次输入 Real name 和 Email address，**注意这里的 Real name 必须填你在 sonatype 创建的 namespace。**

> 如果你的namespace是用的自己的域名，那 name 就填域名，如果namespace是github的账号，那 name 就填GitHub账号名称，比如：
>
> - namespace：cn.sticki，Real name：cn.sticki
> - namespace：io.github.sticki，Real name：sticki


然后会弹出窗口让你输入密钥，这个密钥也需要记住，你每次上传的时候都会让你输入密码的。

![](http://cdn.image.sticki.cn/202405042215698.png)


接着我们再回到刚刚的cmd界面，注意最下面的信息，里面有一个公钥：

```cmd
public and secret key created and signed.

pub   ed25519 2024-05-04 [SC] [expires: 2027-05-04]
      D8F859ACBB78E66AB97AB672766C17B3A2AA7414
uid                      cn.sticki <sticki@126.com>
sub   cv25519 2024-05-04 [E] [expires: 2027-05-04]
```

中间 `D8F859ACBB78E66AB97AB672766C17B3A2AA7414` 这段就是你的公钥，私钥它已经作为文件保存到你的电脑里了。

如果你不小心把cmd窗口关掉了，也可以通过 `gpg --list-keys` 命令来重新查看你以前生成过的密钥信息。

## 发送公钥

秘钥对生成好后，需要把公钥上传到公共服务器供sonatype验证：

```sh
gpg --keyserver keyserver.ubuntu.com --send-keys D8F859ACBB78E66AB97AB672766C17B3A2AA7414
```

> 中央服务器当前支持的 GPG 密钥服务器有：
>
> - `keyserver.ubuntu.com`
> - `keys.openpgp.org`
> - `pgp.mit.edu`

发送完需要验证是否成功：

```sh
gpg --keyserver keyserver.ubuntu.com --recv-keys D8F859ACBB78E66AB97AB672766C17B3A2AA7414
```

如果发送失败，可以多试几次，也可以换另外两个网址试试。

参考我的发送和验证过程：

![发送公钥](http://cdn.image.sticki.cn/202405042239075.png)

到这里 GPG 就配置完毕了！



# 配置pom文件

这个部分略微有点麻烦，我直接放上我的完整pom文件给你参考，你再根据我在后面写的说明进行修改和调整就可以了。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>cn.sticki</groupId>
	<artifactId>spel-validator</artifactId>
	<version>0.0.2-beta</version>
	<packaging>jar</packaging>
	<name>Spel Validator</name>
	<url>https://github.com/stick-i/spel-validator</url>

	<description>
		Bean Validator With Spring Expression Language
	</description>

	<developers>
		<developer>
			<id>sticki</id>
			<name>阿杆</name>
			<roles>
				<role>Project Manager</role>
				<role>Developer</role>
			</roles>
			<email>sticki@xxx.com</email>
			<url>https://github.com/stick-i</url>
		</developer>
	</developers>

	<issueManagement>
		<system>GitHub</system>
		<url>https://github.com/stick-i/spel-validator/issues</url>
	</issueManagement>

	<inceptionYear>2024</inceptionYear>

	<licenses>
		<license>
			<name>Apache License 2.0</name>
			<url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
		</license>
	</licenses>

	<scm>
		<connection>scm:git:git://github.com/stick-i/spel-validator.git</connection>
		<developerConnection>scm:git:git@github.com:stick-i/spel-validator.git</developerConnection>
		<url>https://github.com/stick-i/spel-validator</url>
		<tag>HEAD</tag>
	</scm>

	<properties>
		<maven.compiler.source>8</maven.compiler.source>
		<maven.compiler.target>8</maven.compiler.target>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.hibernate.validator</groupId>
			<artifactId>hibernate-validator</artifactId>
			<version>6.2.5.Final</version>
			<optional>true</optional>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.8.1</version>
				<configuration>
					<source>${maven.compiler.source}</source>
					<target>${maven.compiler.target}</target>
					<encoding>${project.build.sourceEncoding}</encoding>
				</configuration>
			</plugin>
			<!-- Source -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-source-plugin</artifactId>
				<version>3.1.0</version>
				<inherited>true</inherited>
				<executions>
					<execution>
						<id>attach-sources</id>
						<goals>
							<goal>jar</goal>
						</goals>
					</execution>
				</executions>
				<configuration>
					<excludeResources>true</excludeResources>
					<useDefaultExcludes>true</useDefaultExcludes>
				</configuration>
			</plugin>
		</plugins>
	</build>

	<profiles>
		<profile>
			<id>release</id>
			<build>
				<plugins>
					<!-- Javadoc -->
					<plugin>
						<groupId>org.apache.maven.plugins</groupId>
						<artifactId>maven-javadoc-plugin</artifactId>
						<version>2.10.4</version>
						<executions>
							<execution>
								<id>attach-javadocs</id>
								<goals>
									<goal>jar</goal>
								</goals>
							</execution>
						</executions>
					</plugin>
					<!-- GPG -->
					<plugin>
						<groupId>org.apache.maven.plugins</groupId>
						<artifactId>maven-gpg-plugin</artifactId>
						<version>1.6</version>
						<configuration>
                            <!-- 改成你自己的路径 -->
							<executable>E:\_Gpg4win\GnuPG\bin\gpg.exe</executable>
						</configuration>
						<executions>
							<execution>
								<phase>verify</phase>
								<goals>
									<goal>sign</goal>
								</goals>
							</execution>
						</executions>
					</plugin>
					<plugin>
						<groupId>org.sonatype.central</groupId>
						<artifactId>central-publishing-maven-plugin</artifactId>
						<version>0.4.0</version>
						<extensions>true</extensions>
						<configuration>
							<!-- 这里的serverId是之前在settings.xml中配置的 -->
							<publishingServerId>sonatype-sticki</publishingServerId>
							<tokenAuth>true</tokenAuth>
						</configuration>
					</plugin>
				</plugins>
			</build>
		</profile>
	</profiles>

</project>
```

以下是对这份文档的一些说明：

1. 项目基本信息：`properties` 以上的部分，虽然都是一些项目信息，但我建议你都保留下来，其中有一些是必须要有的，比如 `name`、`licenses`、`scm` 这几个是必须存在的，其他的我不确定是不是必须的，但还是建议你保留。记得要改成自己的，不要直接复制了就用。

2. 项目依赖：`dependencies`这部分没有要求，根据自己的项目需求来就好了。

3. 打包插件：上传的jar包必须有 源代码、Javadoc、GPG 这三个部分，都是必须的，所以有三个部分的插件。

   由于我自己在本地打包调试的时候不需要这些东西，所以我把一部分插件放在了 `profile` 里，你 deploy 的时候需要勾选上配置，本地 install 则不需要。

   GPG 的插件需要修改下路径，否则会出现找不到 gpg.exe 的异常。

   最后面这个 `central-publishing-maven-plugin` 是用来发布jar包到平台上的，需要修改下 `publishingServerId`，内容对应前面改的 `setting.xml` 文件。

配置完pom文件后，你可以先 `mvn install -P release` 一下，看看有没有报错什么的。

如果你的 Maven 构建的时候出现异常，但异常信息是乱码的：

![构建时异常](http://cdn.image.sticki.cn/202405050002116.png)

这种情况下我们给Maven修改下编码，在Maven的运行时参数中添加 `-Dfile.encoding=GB2312` 即可：

![配置Maven编码](http://cdn.image.sticki.cn/202405050004174.png)

弄好之后再重新构建，就可以看到异常描述了：

![](http://cdn.image.sticki.cn/202405050006044.png)

Javadoc有很多警告，缺少 @return 标签，可以改一下，也可以不改，看你心情。

没别的问题就可以准备发布项目了。

# 发布项目

先 clean 然后再 deploy，会弹出窗口让你输入 GPG 的密码，验证成功后就会开始上传。

![deploy](http://cdn.image.sticki.cn/202405071607874.gif)

如果在 deploy 时出现了这样的异常：

`Deployment 2c57742a-ff70-4f5c-aa75-5f50614a68eb (Deployment) failed while publishing`

可以到 sonatype 的网站上查看失败的原因：

![发布失败](http://cdn.image.sticki.cn/202405071554207.png)

如果一切顺利的话，你可以在 sonatype 的网站上看到你上传成功的但未发布的maven包：

![发布jar包](http://cdn.image.sticki.cn/202405071611673.png)

这个时候我们点击 `Publish` ，Deployment的状态就会变成 `publishing`，意思是正在发布中，大概多十多分钟，它的状态会变为 `published`，这时候就正式发布成功了。

发布完成后，可以搜索到我们发布的 jar 包：

![搜索我们jar包](http://cdn.image.sticki.cn/202405071615901.png)

后续maven中央仓库和阿里镜像仓库等都会自动同步，这样我们就成功发布了我们的项目了。之后如果升级的话，记得要把版本号进行更新，否则是发布不了的。



# 最后

为了防止有些小伙伴遇到其他奇奇怪怪的问题，这里贴一下官网的文档链接，但比较长，需要自己找噢：https://central.sonatype.org/register/central-portal/

终于完成啦~ 本文中发布的项目是我自己写的一个组件，maven坐标是：

```xml
<groupId>cn.sticki</groupId>
<artifactId>spel-validator</artifactId>
<version>0.0.2-beta</version>
```

GitHub地址也一起贴给大家：[https://github.com/stick-i/spel-validator : 一个强大的 Java 参数校验包，基于 SpEL 实现，扩展自 javax.validation 包，几乎支持所有场景下的参数校验。 (github.com)](https://github.com/stick-i/spel-validator)

欢迎大家体验项目，也希望大家可以给我点个 star 支持一下呀~

关注我，分享更多技术干货~ 也可以订阅我的公众号 程序员阿杆，可以第一时间收到文章推送噢~

![](http://cdn.image.sticki.cn/202405071655437.jpg)

