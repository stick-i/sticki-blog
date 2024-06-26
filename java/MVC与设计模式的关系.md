# MVC模式和设计模式的关系

MVC是23中设计模式中的哪一种？

面试官：MVC架构基于哪种设计模式？

小米一面：说说MVC和设计模式的关系



# 前言

先来看看面试环节吧。

> 面试官：请说说MVC模式是基于哪种设计模式的？
>
> 求职者：MVC本身不就是一种设计模式吗？
>
> 面试官：我的意思是，MVC是基于23中设计模式中的哪一种？
>
> 求职者：难道MVC不是那23种当中的？
>
> 面试官：…………

好吧，这种情况下，大概率是求职者对设计模式的学习比较少，就连我这种没正经学过设计模式的混子，也知道23种设计模式当中没有MVC。

<img src="http://cdn.image.sticki.cn/202403131633314.png" alt="image-20240313163338202" style="zoom: 50%;" />

回归正题，其实MVC模式是基于观察者模式的，你仔细想想，MVC中的Model其实对应的是观察者模式的被观察对象（也叫主题或目标），而View则对应的则是观察者的角色。

如果你想不清楚的话，就继续往下看吧，我带你分析一波。



# MVC模式

MVC 模式代表 Model-View-Controller（模型-视图-控制器） 模式。

- **模型（Model）**：负责管理应用程序的数据和业务逻辑。
- **视图（View）**：显示模型的数据，通常是用户界面元素。
- **控制器（Controller）**：控制器作用于模型和视图上，它控制数据流向模型对象，并在数据变化时更新视图，使视图与模型分离开。

这种模式的目的是实现动态的程序设计，简化对程序的修改和扩展，并使程序某一部分的重复利用成为可能。通过将信息的内部表示与信息的呈现方式分离，使得组件具有较好的重用性。

例如，同一个程序可以使用不同的表现形式来展示相同的数据，比如一批统计数据可以分别用柱状图、条形图来表示。

![MVC模式示例图](http://cdn.image.sticki.cn/202403131820789.png)

## 应用场景

几乎每一个 Web 项目都会使用到 MVC 模式，所以有许多的框架都对此做了集成和封装的工作，像我们 Java 程序员最常见的就是平时使用的 SpringMVC 框架了。

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.0.1.RELEASE</version>
</dependency>
```

当然，现在的 Web 项目中一般只有 Controller 和 Model ，并不存在所谓的 View，那是因为目前流行的前后端分离开发模式，已经将 View 交给前端程序员去做了。

早些年的 Java 程序员，会写一种叫 **jsp** 的东西，就算你没写过，也多少听说过吧。这玩意就是用来展示页面的，也就是 MVC 中的 View 。

那时候的项目结构，是这样子的：

![image-20240313183907174](http://cdn.image.sticki.cn/202403131839218.png)

比现在的结构多了个web文件夹，里面就包含了View层的代码 `index.jsp`，而 jsp 的代码呢，长这样：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>$Title$</title>
  </head>
  <body>
  <%
    System.out.println("hello jsp");
  %>
  </body>
</html>
```

眼熟不🤣，其实就是 HTML 和 Java 的结合写法，最终返回给浏览器的就是一个 HTML 页面。

![R(1)](http://cdn.image.sticki.cn/202403131857601.jpg)

即使现在后端程序员不再写 View 层了，但其实大家的开发思路还是和原来一致的，只是把 View 交给了前端程序员去实现。

除了SpringMVC，服务端的MVC框架还有：Struts、ASP.Net MVC，前端的MVC框架还有：angularjs、reactjs等。



# 观察者模式

前面已经说了，MVC模式就是基于观察者模式的，所以我也不绕弯子，直接给大家介绍一下这个**观察者模式**。

观察者模式是我们常说的23种设计模式中的一种，它定义了对象之间的依赖关系，使得当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知并自动更新。

观察者模式中包含4个角色：

- **目标（Subject）**：也称为主题，它是指被观察的对象。目标中定义了一个观察者的集合，一个目标可以接受任意数量的观察者，同时也提供了一些方法，例如：添加观察者 add() 、删除观察者 delete() 、通知所有观察者 notify() 等。目标类可以是抽象类、具体类或者接口。
- **具体目标（ConcreteSubject）**：具体目标是目标的子类或实现类，通常包含有经常发生改变的数据，当它的状态发生变化时，会向所有的观察者发出通知。如果无需扩展目标类，那具体目标类可以省略。
- **观察者（Observer）**：观察者将对目标的改变做出反应，观察者一般定义为接口或抽象类，接口中声明更新数据的方法 update()。
- **具体观察者（ConcreteObserver）**：具体观察者实现了抽象观察者的 update() 方法，并且会维护一个指向具体目标的引用。当收到观察者的通知开始执行 update() 方法时，会根据观察者的状态来执行对应的逻辑。

下面是我画的结构图：

![观察者模式结构图.drawio](http://cdn.image.sticki.cn/202403150049810.png)

形象一点的话，就是：

![1710435451621](http://cdn.image.sticki.cn/202403150057967.gif)

好了，差不多就是这样。

纸上谈兵终觉浅，看看应用场景可能会更好理解。

## 应用场景

熟悉消息队列的同学应该对 **发布/订阅模式** 有一点了解，就是生产者发送消息到队列中，然后订阅这个队列的所有消费者都会收到这条消息，简单易懂，其实 发布/订阅模式 也是观察者模式的一种变体。



另外，JDK对观察者模式也有支持：

![image-20240315010650591](http://cdn.image.sticki.cn/202404172339479.png)

开发者可以直接使用Observer接口和Observable类来作为观察者模式的抽象层，再自定义具体的观察者类和具体观察目标类，这样可以更方便的在Java中应用观察者模式。



# MVC与观察者模式的关系

在MVC架构中，模型是观察者模式的主题，视图是观察者。当模型的状态发生改变时，视图会自动更新，反映模型的新状态。

让我们回到前面的这张MVC结构图：

![MVC模式示例图](http://cdn.image.sticki.cn/202403131820789.png)

在这张图中，Model层提供的数据是View层所观察的对象，在View层中包含了两个用于显示数据的图表，如果Model层中的数据发生改变，那这两个图表的展示也会随之改变（有可能得刷新一下🤣），所以说MVC中也应用了观察者模式的思想。

懂了吧🤡。



# 后记

最近刚做完一个私接的项目，从产品设计到上线全程由我负责，每天搞到很晚，还挺累的。今天刚交付完，又要开始找工作喽~ 各位朋友如果有工作能帮忙内推一下的，欢迎私信我，小弟先在此谢过啦~

------

好了，知识你已经学走了，要不也给我留下点什么？

![webwxgetmsgimg (1)](http://cdn.image.sticki.cn/202404172336551.jpg)





