# Spring整合Mybais

## Mybatis

**创建Mybatis的步骤**

> * 1. 导入Mybatis依赖
>
>      ```xml
>      <dependency>
>                  <groupId>org.mybatis</groupId>
>                  <artifactId>mybatis</artifactId>
>                  <version>3.5.2</version>
>              </dependency>
>              <dependency>
>                  <groupId>mysql</groupId>
>                  <artifactId>mysql-connector-java</artifactId>
>                  <version>5.1.49</version>
>              </dependency>
>      ```
>
>      
>
> * 2. 编写Mybatis核心配置文件
>
>      ```xml
>      <?xml version="1.0" encoding="UTF-8"?>
>      <!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
>              "http://mybatis.org/dtd/mybatis-3-config.dtd">
>      <configuration>
>          <properties resource="db.properties"/>
>          <settings>
>              <!-- 配置开启驼峰 -->
>              <setting name="mapUnderscoreToCamelCase" value="true"/>
>              <!-- 配置日志 -->
>              <setting name="logImpl" value="STDOUT_LOGGING"/>
>              <!--   开启全局缓存 默认值是true    -->
>              <setting name="cacheEnabled" value="true"/>
>          </settings>
>          <environments default="mysql_developer">
>              <environment id="mysql_developer">
>                  <transactionManager type="jdbc"/>
>                  <dataSource type="pooled">
>                      <property name="driver" value="${driver}"/>
>                      <property name="url" value="${url}"/>
>                      <property name="username" value="${username}"/>
>                      <property name="password" value="${password}"/>
>                  </dataSource>
>              </environment>
>          </environments>
>          <mappers>
>              <mapper resource="com/coderzoe/mapper/UserMapper.xml"/>
>          </mappers>
>      </configuration>
>      ```
>
>      ```properties
>      driver=com.mysql.jdbc.Driver
>      url=jdbc:mysql://localhost:3306/smbms?useSSL=false&useUnicode=true&characterEncoding=UTF-8
>      username=root
>      password=root
>      ```
>
>      
>
> * 3. 编写Mybatis工具类
>
>      ```java
>      package com.coderzoe.utils;
>      
>      import org.apache.ibatis.io.Resources;
>      import org.apache.ibatis.session.SqlSession;
>      import org.apache.ibatis.session.SqlSessionFactory;
>      import org.apache.ibatis.session.SqlSessionFactoryBuilder;
>      
>      import java.io.IOException;
>      import java.io.InputStream;
>      
>      /**
>       * @author yhs
>       * @date 2020/6/7 9:31
>       * @description
>       */
>      public class MybatisUtils {
>          private static ThreadLocal<SqlSession> sessionThreadLocal = new ThreadLocal<>();
>          private static SqlSessionFactory sqlSessionFactory;
>      
>          static {
>              try {
>                  String resource = "mybatis-config.xml";
>                  InputStream inputStream = Resources.getResourceAsStream(resource);
>                  sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
>              } catch (IOException e) {
>                  e.printStackTrace();
>              }
>          }
>      
>          private MybatisUtils() {
>          }
>      
>          public static SqlSession getSqlSession(){
>              SqlSession sqlSession = sessionThreadLocal.get();
>              if(sqlSession==null){
>                  sqlSession = sqlSessionFactory.openSession();
>                  //设置事务自动提交 默认是false
>      //            sqlSession = sqlSessionFactory.openSession(true);
>              }
>              return sqlSession;
>          }
>      
>          public static void closeSqlSession(){
>              SqlSession sqlSession = sessionThreadLocal.get();
>              if(sqlSession!=null){
>                  sqlSession.close();
>                  sessionThreadLocal.remove();
>              }
>          }
>      }
>      ```
>
>      
>
> * 4. 编写实体类与实体类的接口
>
>      ```java
>      package com.coderzoe.entity;
>      
>      /**
>       * @author yhs
>       * @date 2020/6/7 9:28
>       * @description
>       */
>      public class User {
>          private Long id;
>          private String name;
>          private String password;
>      
>          @Override
>          public String toString() {
>              return "User{" +
>                      "id=" + id +
>                      ", name='" + name + '\'' +
>                      ", password='" + password + '\'' +
>                      '}';
>          }
>      
>          public User() {
>          }
>      
>          public Long getId() {
>              return id;
>          }
>      
>          public void setId(Long id) {
>              this.id = id;
>          }
>      
>          public String getName() {
>              return name;
>          }
>      
>          public void setName(String name) {
>              this.name = name;
>          }
>      
>          public String getPassword() {
>              return password;
>          }
>      
>          public void setPassword(String password) {
>              this.password = password;
>          }
>      }
>      ```
>
>      ```java
>      package com.coderzoe.mapper;
>      
>      import com.coderzoe.entity.User;
>      
>      import java.util.List;
>      
>      /**
>       * @author yhs
>       * @date 2020/6/7 9:54
>       * @description
>       */
>      public interface UserMapper {
>          List<User> getUsers();
>      }
>      ```
>
>      
>
> * 5. 编写UserMapper.xml 实现接口
>
>      ```xml
>      <?xml version="1.0" encoding="UTF-8" ?>
>      <!DOCTYPE mapper
>              PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
>              "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
>      <mapper namespace="com.coderzoe.mapper.UserMapper">
>          <select id="getUsers" resultType="com.coderzoe.entity.User">
>              select * from user
>          </select>
>      </mapper>
>      ```
>
>      
>
> *  6. 编写测试类
>
>      ```java
>      import com.coderzoe.entity.User;
>      import com.coderzoe.mapper.UserMapper;
>      import com.coderzoe.utils.MybatisUtils;
>      import org.apache.ibatis.session.SqlSession;
>      import org.junit.Test;
>      
>      import java.util.List;
>      
>      /**
>       * @author yhs
>       * @date 2020/6/7 9:55
>       * @description
>       */
>      public class MybatisTest {
>          @Test
>          public void test1(){
>              SqlSession sqlSession = MybatisUtils.getSqlSession();
>              UserMapper mapper = sqlSession.getMapper(UserMapper.class);
>              List<User> users = mapper.getUsers();
>              System.out.println(users);
>          }
>      }
>      ```



通过上述几步我们实现了使用Mybatis来做增删改查。

总结一下为：

* 1.获得数据源，配置JDBC。

* 2.配置Mybatis，如别名，开启驼峰缓存等等设置。

* 3.编写Mybatis工具，主要包括

  > * 将Mybatis配置文件导入，得到SqlSessionFactory类。
  > * 通过SqlSessionFactory类得到SqlSession类。SqlSession类是我们操作Mybatis的基础。

* 4.通过XML映射，实现Dao层接口的核心业务函数



## Spring整合Mybatis

Spring整合Mybatis就是将Mybatis核心配置文件交由Spring配置文件来配置，同时将我们的SqlSessionFactory与SqlSession，交由Spring来管理(IOC)



>* 1. 导入依赖
>
>     ```xml
>     <dependencies>
>         <dependency>
>                 <groupId>org.mybatis</groupId>
>                 <artifactId>mybatis</artifactId>
>                 <version>3.5.2</version>
>             </dependency>
>             <dependency>
>                 <groupId>mysql</groupId>
>                 <artifactId>mysql-connector-java</artifactId>
>                 <version>5.1.49</version>
>             </dependency>
>             <dependency>
>                 <groupId>org.springframework</groupId>
>                 <artifactId>spring-webmvc</artifactId>
>                 <version>5.2.6.RELEASE</version>
>             </dependency>
>             <!-- Spring操作JDBC的话，需要一个SpringJDBC包 -->
>             <dependency>
>                 <groupId>org.springframework</groupId>
>                 <artifactId>spring-jdbc</artifactId>
>                 <version>5.2.6.RELEASE</version>
>             </dependency>
>             <dependency>
>                 <groupId>junit</groupId>
>                 <artifactId>junit</artifactId>
>                 <version>4.11</version>
>             </dependency>
>             <dependency>
>                 <groupId>aspectj</groupId>
>                 <artifactId>aspectjrt</artifactId>
>                 <version>1.5.4</version>
>             </dependency>
>             <dependency>
>                 <groupId>org.mybatis</groupId>
>                 <artifactId>mybatis-spring</artifactId>
>                 <version>2.0.4</version>
>             </dependency>
>             <dependency>
>                 <groupId>org.aspectj</groupId>
>                 <artifactId>aspectjweaver</artifactId>
>                 <version>1.9.5</version>
>             </dependency>
>         </dependencies>
>     ```
>
>     
>
>* 2. 编写Spring配置文件，整合Mybatis
>
>  ```xml
>  <?xml version="1.0" encoding="GBK"?>
>  <beans xmlns="http://www.springframework.org/schema/beans"
>         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>         xmlns:context="http://www.springframework.org/schema/context"
>         xsi:schemaLocation="http://www.springframework.org/schema/beans
>          https://www.springframework.org/schema/beans/spring-beans.xsd
>          http://www.springframework.org/schema/context
>          https://www.springframework.org/schema/context/spring-context.xsd">
>  
>      <!-- 通过Spring配置数据库的数据源 Spring-jdbc -->
>  
>      <!-- 这里所作的工作是替换mybatis-config.xml里的配置，通过Spring的dataSource来连接数据源 -->
>      <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
>          <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
>          <property name="url" value="jdbc:mysql://localhost:3306/smbms?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
>          <property name="username" value="root"/>
>          <property name="password" value="root"/>
>      </bean>
>  
>      <!-- 这里是获得SqlSessionFactory 通过将上一步的datasource引进来，创建SqlSessionFactory -->
>      <!-- 他对应的代码为Mybatis工具类里的
>              String resource = "mybatis-config.xml";
>              InputStream inputStream = Resources.getResourceAsStream(resource);
>              sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
>       -->
>      <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
>          <property name="dataSource" ref="dataSource"/>
>          <property name="configLocation" value="mybatis-config.xml"/>
>      </bean>
>  
>      <!-- 通过上一步得到的SqlSessionFactory 构造我们的sqlSession类 -->
>      <!-- 他对应我们代码的
>              sqlSession = sqlSessionFactory.openSession();
>      -->
>      <bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
>          <constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory"/>
>      </bean>
>  
>      <context:component-scan base-package="com.coderzoe"/>
>  </beans>
>  ```
>
>  上面的配置文件主要做了三件事：
>
>  > 1. 通过SpringJDBC 创建dataSource(数据源)，替换掉了Mybatis核心配置文件的**environment**标签里的内容
>  > 2. 将SqlSessionFactoryBean(即我们Mybatis的SqlSessionFactory)类注入Spring容器。其中SqlSessionFactoryBean类需要指定一个数据源，也即我们上一步连接JDBC的数据源。另外，SqlSessionFactoryBean类还可以导入之前Mybatis的核心配置文件(主要是一些settings或typeAliases 当然还可以放Mappers) 但其实这些都可以在SqlSessionFactoryBean中进行配置，也即可以完全抛弃之前的mybatis-config.xml配置文件。但为了避免耦合性，我还是保留了Mybatis的配置文件，但让他只做一些Mybatis的个性化配置。
>  > 3. 将SqlSessionTemplate(即我们Mybatis的SqlSession)类注入Spring中，SqlSessionTemplate类的注入必须指出一个包含SqlSessionFactory的初始化，且只能构造器注入，也不难理解，因为我们之前在Mybatis中得到SqlSession也是通过sqlSessionFactory.openSession()
>
>* 3. 编写Mybatis核心配置文件
>
>     ```xml
>     <?xml version="1.0" encoding="UTF-8"?>
>     <!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
>             "http://mybatis.org/dtd/mybatis-3-config.dtd">
>     <configuration>
>         <settings>
>             <!-- 配置开启驼峰 -->
>             <setting name="mapUnderscoreToCamelCase" value="true"/>
>             <!-- 配置日志 -->
>             <setting name="logImpl" value="STDOUT_LOGGING"/>
>             <!--   开启全局缓存 默认值是true    -->
>             <setting name="cacheEnabled" value="true"/>
>         </settings>
>         <mappers>
>             <mapper resource="com/coderzoe/dao/UserDao.xml"/>
>         </mappers>
>     </configuration>
>     ```
>
>     上面我们说过了，这个xml是可以完全抛弃，完全在Spring里配置的，但为了避免耦合，我们还是保留了他的一些配置。
>
>  4. 编写实体类，实体类接口，实体类接口实现类与实体类接口对应Mybatis的XML文件
>
>     ```java
>     package com.coderzoe.entity;
>     
>     /**
>      * @author yhs
>      * @date 2020/6/7 10:04
>      * @description
>      */
>     public class User {
>         private Long id;
>         private String name;
>         private String password;
>     
>         @Override
>         public String toString() {
>             return "User{" +
>                     "id=" + id +
>                     ", name='" + name + '\'' +
>                     ", password='" + password + '\'' +
>                     '}';
>         }
>     
>         public User() {
>         }
>     
>         public Long getId() {
>             return id;
>         }
>     
>         public void setId(Long id) {
>             this.id = id;
>         }
>     
>         public String getName() {
>             return name;
>         }
>     
>         public void setName(String name) {
>             this.name = name;
>         }
>     
>         public String getPassword() {
>             return password;
>         }
>     
>         public void setPassword(String password) {
>             this.password = password;
>         }
>     }
>     ```
>
>     ```java
>     package com.coderzoe.dao;
>     
>     import com.coderzoe.entity.User;
>     
>     import java.util.List;
>     
>     /**
>      * @author yhs
>      * @date 2020/6/7 10:28
>      * @description
>      */
>     public interface UserDao {
>         List<User> getUsers();
>     }
>     ```
>
>     ```xml
>     <?xml version="1.0" encoding="UTF-8" ?>
>     <!DOCTYPE mapper
>             PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
>             "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
>     <mapper namespace="com.coderzoe.dao.UserDao">
>         <select id="getUsers" resultType="com.coderzoe.entity.User">
>             select * from user
>         </select>
>     </mapper>
>     ```
>
>     ```java
>     package com.coderzoe.dao;
>     
>     import com.coderzoe.entity.User;
>     import org.mybatis.spring.SqlSessionTemplate;
>     import org.springframework.beans.factory.annotation.Autowired;
>     import org.springframework.stereotype.Component;
>     
>     import java.util.List;
>     
>     /**
>      * @author yhs
>      * @date 2020/6/7 10:28
>      * @description
>      */
>     @Component
>     public class UserMapperImpl implements UserDao{
>     
>     
>         /**
>          * 由于我们之前已经在Spring的xml中注册了sqlSessionTemplate 所有这里直接拿过来用即可。
>          */
>         @Autowired
>         private SqlSessionTemplate sqlSessionTemplate;
>     
>         @Override
>         public List<User> getUsers() {
>             UserDao mapper = sqlSessionTemplate.getMapper(UserDao.class);
>             return mapper.getUsers();
>         }
>     }
>     ```
>
>     可以看到，我们比上面直接用Mybatis多了一层UserMapperImpl接口实现类，其实这里的工作是封装了我们之前在Test里做的工作(获得SqlSession，然后执行mapper方法)。这里因为我们已经在Spring配置文件中注入了SqlSessionTemplate 所以这里就直接拿来用即可(又是IOC，自动装配)
>
>  5. 编写测试类
>
>     ```java
>     import com.coderzoe.dao.UserDao;
>     import org.junit.Test;
>     import org.springframework.context.ApplicationContext;
>     import org.springframework.context.support.ClassPathXmlApplicationContext;
>     
>     /**
>      * @author yhs
>      * @date 2020/6/7 10:46
>      * @description
>      */
>     public class MyTest {
>         private ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
>         @Test
>         public void test1(){
>             UserDao userMapperImpl = context.getBean("userMapperImpl", UserDao.class);
>             System.out.println(userMapperImpl.getUsers());
>         }
>     }
>     
>     ```

## 附录

官方提供了Spring整合Mybatis的另一种实现，其实就是对第一种的一层封装，将SqlSession的创建交给了一个父类SqlSessionDaoSupport，不再需要你创建SqlSession。

> * 1. 编写接口实现类
>
>      ```java
>      package com.coderzoe.dao;
>      
>      import com.coderzoe.entity.User;
>      import org.mybatis.spring.SqlSessionTemplate;
>      import org.mybatis.spring.support.SqlSessionDaoSupport;
>      
>      import java.util.List;
>      
>      /**
>       * @author yhs
>       * @date 2020/6/7 13:27
>       * @description
>       */
>      public class UserMapperImpl2 extends SqlSessionDaoSupport implements UserDao{
>      
>          @Override
>          public List<User> getUsers() {
>              SqlSessionTemplate sqlSessionTemplate = getSqlSessionTemplate();
>              UserDao mapper = sqlSessionTemplate.getMapper(UserDao.class);
>              return mapper.getUsers();
>          }
>      }
>      ```
>
> * 2. 编写Spring配置类
>
>      ```xml
><?xml version="1.0" encoding="GBK"?>
>      <beans xmlns="http://www.springframework.org/schema/beans"
>       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>             xmlns:context="http://www.springframework.org/schema/context"
>             xsi:schemaLocation="http://www.springframework.org/schema/beans
>              https://www.springframework.org/schema/beans/spring-beans.xsd
>              http://www.springframework.org/schema/context
>              https://www.springframework.org/schema/context/spring-context.xsd">
>      
>          <!-- 通过Spring配置数据库的数据源 Spring-jdbc -->
>      
>          <!-- 这里所作的工作是替换mybatis-config.xml里的配置，通过Spring的dataSource来连接数据源 -->
>          <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
>              <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
>              <property name="url" value="jdbc:mysql://localhost:3306/smbms?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
>              <property name="username" value="root"/>
>              <property name="password" value="root"/>
>          </bean>
>      
>          <!-- 这里是获得SqlSessionFactory 通过将上一步的datasource引进来，创建SqlSessionFactory -->
>          <!-- 他对应的代码为Mybatis工具类里的
>                  String resource = "mybatis-config.xml";
>                  InputStream inputStream = Resources.getResourceAsStream(resource);
>                  sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
>           -->
>          <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
>              <property name="dataSource" ref="dataSource"/>
>              <property name="configLocation" value="mybatis-config.xml"/>
>          </bean>
>      
>          <!-- 将我们的接口实现类注入Spring，然后注入我们的sqlSessionFactory -->
>          <bean id="userMapperImpl2" class="com.coderzoe.dao.UserMapperImpl2">
>              <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
>          </bean>
>      
>          <context:component-scan base-package="com.coderzoe"/>
>      </beans>
>      ```
>   
> * 3.  编写测试类
>   
>      ```java
>   import com.coderzoe.dao.UserDao;
>      import org.junit.Test;
>   import org.springframework.context.ApplicationContext;
>      import org.springframework.context.support.ClassPathXmlApplicationContext;
>      
>      /**
>       * @author yhs
>       * @date 2020/6/7 10:46
>       * @description
>       */
>      public class MyTest {
>          private ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
>          @Test
>          public void test2(){
>              UserDao userMapperImpl2 = context.getBean("userMapperImpl2", UserDao.class);
>              System.out.println(userMapperImpl2.getUsers());
>          }
>      }
>      ```

其实上面的代码还是很好理解的，就是通过SqlSessionDaoSupport对我们的SqlSession做了一层封装。

没封装前我们需要做的：

> * 创建SqlSessionFactory
> * 通过SqlSessionFactory创建SqlSession
> * 将SqlSession注入我们的UserMapperImpl，做增删改查操作。

封装就是将上面的步骤一二整合了一下，封装的实现通过继承SqlSessionDaoSupport父类。

这个父类做的工作就是帮你创建SqlSession，不再需要你自己创建，但我们都知道SqlSession的创建是需要SqlSessionFactory的，所以在Spring的配置文件中，我们注入了SqlSessionFactory

```xml
 <bean id="userMapperImpl2" class="com.coderzoe.dao.UserMapperImpl2">
        <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
    </bean>
```

那么很多人又要问了，我userMapperImpl2对象没有sqlSessionFactory属性啊，这个property明显是调用的set方法，我没有啊，这怎么调？ 很简单，setSqlSessionFactory()这个方法是你继承的父类SqlSessionDaoSupport里的，我们知道子类继承父类，就是继承了父类的一切东西，所以你的userMapperImpl2自然也是有的。这时你将sqlSessionFactory传给你的继承父类，他用这个sqlSessionFactory给你创造了SqlSession，所以你就可以直接通过getSqlSessionTemplate()得到SqlSession。getSqlSessionTemplate()方法哪来的？当然也是父类里的。

SqlSessionDaoSupport的部分源码：

![image-20200607135318859](C:\Users\90617\AppData\Roaming\Typora\typora-user-images\image-20200607135318859.png)

![image-20200607135427520](C:\Users\90617\AppData\Roaming\Typora\typora-user-images\image-20200607135427520.png)

![image-20200607135655047](C:\Users\90617\AppData\Roaming\Typora\typora-user-images\image-20200607135655047.png)



**其实我个人觉得还是上面的方法更好理解，SqlSessionDaoSupport隐藏了SqlSession的创建，反而不太直观，而且代码量并没有少多少。**