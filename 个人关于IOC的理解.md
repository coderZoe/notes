# 个人关于IOC的理解  

## IOC

IOC即控制反转(Inversion of Control)

了解控制反转前我们先了解下正常逻辑的思路：

> 我需要某个类，我创建了这个类。  

但控制反转正好反过来：

> 类的创建不再由你实现，创建对象的控制权，由你转交给了第三双方容器，你如果需要它，那你就去找第三方容器要，容器已经创建好了。

**举个现实中的例子：**

> 你今天很渴，你想要喝一杯橙汁，如果按正常开发思路，那这杯橙汁的控制权在我，我需要橙汁，我就去创造一个橙汁，我买橙子，买白砂糖，买榨汁机，烧热水，准备制造橙汁。但我们都知道，现实生活中不是这样的，我渴了，需要一杯橙汁，我只需要找家商场购买橙汁，橙汁是商店(或制造商)制造的，我只需要取来用。这个例子就实现了控制反转，我需要橙汁，但我不创建，橙汁的创建是第三方商家创建的，我不需要关心，只需要拿来用即可。

## DI

 DI即依赖注入(Dependency Injection)，他是控制反转的一种实现方式。

依赖注入分为两个词：**依赖**和**注入**。

> 所谓依赖就是我的实现需要依赖于你，我渴了需要一杯橙汁，那么此时我们就可以说我依赖橙汁，或者说我要解决我渴这个问题需要依赖橙汁。

> 而注入是说将你所需的这个依赖注入到自己的类中，通过注入的方式来获得依赖。

举个例子  

> 公司想让你给每个类加个日志 然后需要这些日志输出到文件里  
> 这时你可以简单暴力的做，在每个类加一个FileLog的对象，让这个对象去执行日志的输出



```java
public class Service1{
    private ILogger logger = new FileLogger();
    
    public void add(){
        logger.log("执行增加函数!!");
        //statement
        logger.log("增加函数执行完成!!");
    }
    public void delete(){
        logger.log("执行删除函数!!");
        //statement
        logger.log("删除函数执行完成!!");
    }
    public void update(){
        logger.log("执行修改函数!!");
        //statement
        logger.log("修改函数执行完成!!");
    }
    public void query(){
        logger.log("执行查询函数!!");
        //statement
        logger.log("查询函数执行完成!!");
    }
}
```

```java
public class ServiceN{
    private ILogger logger = new FileLogger();
    
    public void add(){
        logger.log("执行增加函数!!");
        //statement
        logger.log("增加函数执行完成!!");
    }
    public void delete(){
        logger.log("执行删除函数!!");
        //statement
        logger.log("删除函数执行完成!!");
    }
    public void update(){
        logger.log("执行修改函数!!");
        //statement
        logger.log("修改函数执行完成!!");
    }
    public void query(){
        logger.log("执行查询函数!!");
        //statement
        logger.log("查询函数执行完成!!");
    }
}
```

这样你在n个类里加了日志功能，输出到文件。

 突然有一天需求变了，不再是输出到文件了，要求输出到控制台，这时你不可能去替换每个类里的log，而且就算这次全替换了，下次再改成打印输出到服务器怎么办？ 
之所以这里每个类都需要改，是因为我们每个类依赖的日志功能都自己去实现了，那我们可不可以不自己实现，实现由统一的地方来控制。这样改的时候，我只需要改统一的地方就好了。

这就是**工厂设计模式**。

```java
public class Service1{
    private ILogger logger = LoggerFactory.createLogger();
    
    public void add(){
        logger.log("执行增加函数!!");
        //statement
        logger.log("增加函数执行完成!!");
    }
    public void delete(){
        logger.log("执行删除函数!!");
        //statement
        logger.log("删除函数执行完成!!");
    }
    public void update(){
        logger.log("执行修改函数!!");
        //statement
        logger.log("修改函数执行完成!!");
    }
    public void query(){
        logger.log("执行查询函数!!");
        //statement
        logger.log("查询函数执行完成!!");
    }
}
```

```java
public class ServiceN{
    private ILogger logger = LoggerFactory.createLogger();
    
    public void add(){
        logger.log("执行增加函数!!");
        //statement
        logger.log("增加函数执行完成!!");
    }
    public void delete(){
        logger.log("执行删除函数!!");
        //statement
        logger.log("删除函数执行完成!!");
    }
    public void update(){
        logger.log("执行修改函数!!");
        //statement
        logger.log("修改函数执行完成!!");
    }
    public void query(){
        logger.log("执行查询函数!!");
        //statement
        logger.log("查询函数执行完成!!");
    }
}
```

```java
public class LoggerFactory {
   public static ILogger createLogger() {
      return new ServerLogger();
   }
}
```

看，这样我们就实现了**控制反转**，将log的创建不交给每个类来具体实现，而交给一个统一的第三方来实现。我们只需要找第三方要就好了，他给我们造。这样我们需求改了的时候，只需要改工厂方法即可。

但这样又会有问题：每一个类都需要日志功能，每一个类都需要第三方来new一个日志类，但其实每个日志类并没什么不同，这样会造成大量重复的日志类占用资源。那我可不可以将日志类设计为单例的呢？所有的创建都是来自同一个日志类。当然可以。 

```java
public class LoggerFactory {
    private static ILogger logger = new ServerLogger();
    public static ILogger createLogger() {
        return logger ;
    }
}
```

这样我们就只创建了一个日志对象，每次返回的都是他。

但很多时候，我们的需求是变动的不可预测的，如果这时候又有其他的依赖需要的不是单例，而是原型，那我们又得改。此时我们就引入了Spring
在Spring中，我们将依赖类进行托管，控制权交给Spring，我们只需要在每个类里注入依赖类即可 这样的好处前面也说了，我们并不关心这个依赖是怎么构建的或有没有，也不需要在我的代码里new一个依赖，我只需要直接拿来用即可 。

```java
public class Service1{
    private ILogger logger;
    
    public void add(){
        logger.log("执行增加函数!!");
        //statement
        logger.log("增加函数执行完成!!");
    }
    public void delete(){
        logger.log("执行删除函数!!");
        //statement
        logger.log("删除函数执行完成!!");
    }
    public void update(){
        logger.log("执行修改函数!!");
        //statement
        logger.log("修改函数执行完成!!");
    }
    public void query(){
        logger.log("执行查询函数!!");
        //statement
        logger.log("查询函数执行完成!!");
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="serverLogger" class="com.springnovel.perfectlogger.ServerLogger"/>
    <bean id="Service1" class="com.coderzoe.service.Service1">
        <property name="logger" ref="serverLogger"/>
    </bean>
</beans>
```

这样我们的类就只需要引入依赖，具体的创建和赋值，Spring帮你做好了(但你得在xml或注解中配置)