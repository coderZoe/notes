# Spring AOP的个人理解

**了解AOP之前先了解一下代理模式**

## 代理模式

所谓代理模式，就是在调用方和真实处理对象方之间加一层代理对象。这样调用方不再调用我们的真实处理对象，改而调用我们的代理。这样的好处是：

* 真实处理类就只需要关注我们的业务层实现，不需要关注细枝末节。
* 代理类类似于做了一层封装。封装我们的真实类，在此基础上，加上我们的细枝末节。
* 调用方实际调用起来，真实类和代理类的调用感觉不到任何差别。但我们可以在中间层代理类做更多细节的操作，这样就做到了真实调用方和真实处理类的解耦。

**举个现实中的例子：**

通常我们租房都需要找中介，很少自己直接去找房东的。这样我们实现的调用关系为 ：租客->中介->房东

对于房东来说他是我们的真实处理类，房东更关心的点是签合同和收房租。但租房子除了签合同和收房租外，还需要看房子，谈价钱，交付钥匙等等手续。在这个示例中：签合同和收房租属于我们的核心业务，而谈价格，看房子等就属于我们的周边功能，细枝末节。因此我们的真实类只需要实现签合同和收房租方法，而让中介(也就是我们的代理)实现看房子，谈价钱，交付钥匙等方法。租客在实际调用中只需要调用中介包装过的方法即可。代码如下：

```java
/**
 * 创建房东和中介共同的接口 定义了他们的行为，便于调用方调用
 */

public interface Rent {
    void rent();
}
```

```java
/**
 * 创建真实类 房东
 */

public class Landlord implements Rent {

    public void rent() {
        System.out.println("房东签合同了！！");
        System.out.println("房东收房租了！！");
    }
}
```

```java
/**
 * 创建代理类
 */
public class MyProxy implements Rent {

    //被代理的实例
    private Rent rent;

    public void setRent(Rent rent) {
        this.rent = rent;
    }
    
    public void rent() {
        System.out.println("中介带你看房子了！！");
        System.out.println("中介和你谈价格了！！");
        rent.rent();
        System.out.println("中介把钥匙交给你了！！");
    }

}
```

```java
/**
 *创建租客类 调用中介租房
*/
public class Tenant {
    public static void main(String[] args) {
        MyProxy myProxy = new MyProxy();
        myProxy.setRent(new Landlord());
        myProxy.rent();
    }
}

```

实际执行结果

![image-20200605122729709](C:\Users\90617\AppData\Roaming\Typora\typora-user-images\image-20200605122729709.png)



上面就是Java的代理模式

## AOP

AOP的全称叫做 **Aspect Oriented Program 面向切面编程**

先进行几个定义：

* 核心业务功能：即业务处理时实际关心的操作，比如增删改查
* 周边功能：在写业务代码时附属的功能，比如日志，事务管理等。

在AOP的思想里，周边功能叫做切面，核心业务需要和切面分开独立开发。然后再将切面功能与核心业务编织在一起，这就叫AOP。

AOP能够将那些与业务无关，**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码**，**降低模块间的耦合度**，并**有利于未来的可拓展性和可维护性**。

AOP里面的几个概念：

* 切入点：即调用切面的地方，在某个类的某个方法内调切面。(where)
* 通知，在切入点的什么时间，执行什么操作。比如在删除数据(切入点)里，业务执行之前(时间)加入日志操作(操作)  (when and where)
* 切面：切入点加通知即为切面(在什么地方，什么时间，做什么事)。这样，实际开发中，我们可以独立开发切面模块。
* 织入：加切面加入对象，并创建出代理对象的过程。

**还拿刚才租房举例子**

```java
import org.springframework.stereotype.Component;

/**
 * @author yhs
 * @date 2020/6/5 15:51
 * @description 实际业务类 不关心其他
 */
@Component
public class Landlord {
    public void rent(){
        System.out.println("签合同");
        System.out.println("收房租");
    }
}
```

````java
package com.coderzoe.demo5;

import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

/**
 * @author yhs
 * @date 2020/6/5 15:54
 * @description 切面类 即我们的代理
 */
@Component
@Aspect
public class Section {

    @Before("execution(* com.coderzoe.demo5.Landlord.rent())")
    public void before(){
        System.out.println("带租客看房！！");
        System.out.println("谈价钱！！");
    }
    @After("execution(* com.coderzoe.demo5.Landlord.rent())")
    public void after(){
        System.out.println("交钥匙");
    }
}
````

```java
package com.coderzoe.demo5;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @author yhs
 * @date 2020/6/5 16:04
 * @description
 */
public class Test {

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        Landlord landlord = context.getBean("landlord", Landlord.class);
        landlord.rent();
    }
}
```



运行结果：

![image-20200605174144551](C:\Users\90617\AppData\Roaming\Typora\typora-user-images\image-20200605174144551.png)



这里的Section与我们之前的代理类MyProxy所作的工作一样，不过更通用和简洁。

在切面类中，我们指定了切入点( com.coderzoe.demo5.Landlord.rent() )，同时我们指定了通知@Before和before函数，@After和after函数

通过上面我们可以知道切面=切入点+通知，这样我们就创建了一个切面类，他完全独立于我们的实际业务类，只做切入操作。这样也不难理解，核心功能和周边功能可以独立开发。

**注意**

AOP的使用除了spring-aop包以外，还需要aspectjrt和aspectjweaver依赖

```java
<dependencies>
     <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.2.6.RELEASE</version>
        </dependency>
    <dependency>
        <groupId>aspectj</groupId>
        <artifactId>aspectjrt</artifactId>
        <version>1.5.4</version>
    </dependency>
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.9.5</version>
    </dependency>
</dependencies>
```

