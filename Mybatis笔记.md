# Mybatis

![image-20200622101949965](C:\Users\90617\AppData\Roaming\Typora\typora-user-images\image-20200622101949965.png)

## 前言

**Mybatis：**

> MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录 。（摘自Mybatis官方文档）

**Mybatis与JPA的对比：**

* Mybatis是一个半自动框架，JPA(Hibernate)是一个全自动框架。
* Mybatis相比JPA更加轻量化，学习成本更加的低，入门简单容易。
* Mybatis更适合联表查询，JPA更适合单表查询
* 开发效率：JPA > Mybatis，因为JPA是全自动框架，不需要写任何SQL语句，但Mybatis的所有SQL语句都需要自己写。
* 国内热度：Mybatis>>JPA   国外热度：JPA>>Mybatis

**先修内容：**

* SQL语句
* JDBC



## 为什么学习Mybatis

无论是Mybatis或JPA其实都是**ORM(Object Relational Mapping)**的一种实现框架，都是对**JDBC**的一种封装

**ORM:**

>在关系型数据库和对象之间作一个映射，这样，我们在具体的操作数据库的时候，就不需要再去和复杂的SQL语句打交道，只要像平时操作对象一样操作它就可以

举个例子：

我们当前有一个User表

![image-20200622090529799](C:\Users\90617\AppData\Roaming\Typora\typora-user-images\image-20200622090529799.png)

他会对应一个我们Java中的一个User类

```java
/**
 * @author yhs
 * @date 2020/5/19 22:00
 * @description 数据库实体
 */

public class User {
    private Long id;
    private String name;
    private String password;

    public User() {
    }
    
    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", password='" + password + '\'' +
                '}';
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

现在我们有一个需求，查询用户表中所有的用户

如果用**JDBC**开发：

```java
/**
 * @author yhs
 * @date 2020/6/22 9:10
 * @description
 */
public class TestUser {
    @Test
    public void getUserList(){
        List<User> userList = new ArrayList<>();
        try {
            String sql = "select * from user";
            //JDBC的配置信息无关紧要，所以用个工具类隐藏了
            Connection connection = BaseDao.getConnection();
            if(connection!=null){
                PreparedStatement preparedStatement = connection.prepareStatement(sql);
                ResultSet resultSet = preparedStatement.executeQuery();
                while (resultSet.next()){
                    User user = new User();
                    user.setId(resultSet.getLong("id"));
                    user.setName(resultSet.getString("name"));
                    user.setPassword(resultSet.getString("password"));
                    userList.add(user);
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }

        System.out.println(userList);
    }
}

```

可以看到对于查询回来的结果，放入了**resultSet**对象中，**我们需要自己从中取数据库的字段，然后将其与我们的User字段类对应上**。

```java
User user = new User();
user.setId(resultSet.getLong("id"));
user.setName(resultSet.getString("name"));
user.setPassword(resultSet.getString("password"));
```

如果使用Mybatis，数据库字段与我们的Entity映射就由框架做了，我们不需要自己做

```java
/**
 * @author yhs
 * @date 2020/5/19 22:06
 * @description User的Dao层
 */
public interface UserMapper {
    /**
     * @data: 2020/06/22 09:29
     * @author: yhs
     * @return: {@link List<User> }
     * @description: 查询全部用户
     */
    List<User> getUserList();
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.coderzoe.dao.UserMapper">
        <select id="getUserList" resultType="com.coderzoe.entity.User">
        select * from smbms.user
    </select>
</mapper>
```

```java
/**
 * @author yhs
 * @date 2020/5/19 22:29
 * @description
 */
public class UserMapperTest {
    @Test
    public void test(){
        SqlSession sqlSession = MybatisUtil.getSqlSession();
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        List<User> userList = userMapper.getUserList();
        System.out.println(userList);
        MybatisUtil.closeSqlSession();
    }
}
```

可以看到，对于查询出来的结果，我们并没有手动将数据库与Java类匹配，这一些都是框架帮我们做的。

总结：

* **学习Mybatis提高了我们的开发效率，将原本繁琐的映射关系由框架来代替我们完成**
* **相比于JPA，Mybatis属于半自动框架，SQL语句自己写，这样查询的灵活度会更高，连表查询会更加容易。**

## Mybatis的学习

### 1. 项目搭建

#### 1.1 导入依赖

使用Mybatis需要导入下面两个依赖：

```xml
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.49</version>
        </dependency>
```

由于Mybatis的Dao层实现是xml，加上Maven本身的特性，我们需要配置Maven的编译

```xml
<build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.xml</include>
                    <include>**/*.properties</include>
                </includes>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                    <include>**/*.properties</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
</build>
```

#### 1.2 配置核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/smbms?useSSL=false&useUnicode=true&characterEncoding=UTF-8"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
      </dataSource>
    </environment>
  </environments>
</configuration>
```

配置文件解释：

```xml
<!-- 一个environments下可以包含多套environment，
	每个environment即为一个数据库连接环境，
	这里通过id指明当前使用哪套环境。 -->  
<environments default="development">
```

```xml
<!-- 事务管理选择为JDBC -->
<transactionManager type="JDBC"/>
```

```xml
<!-- 通过连接池的形式管理数据源 -->
<dataSource type="POOLED">
```

```xml
<!-- 配置JDBC连接 -->
<property name="driver" value="${driver}"/>
<property name="url" value="${url}"/>
<property name="username" value="${username}"/>
<property name="password" value="${password}"/>
```

#### 1.3编写工具类

```java
package com.coderzoe.util;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;
import java.sql.Connection;

/**
 * @author yhs
 * @date 2020/5/18 17:32
 * @description
 */
public class MybatisUtil {
    private static ThreadLocal<SqlSession> sessionThreadLocal = new ThreadLocal<>();
    private static SqlSessionFactory sqlSessionFactory;

    static {
        try {
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private MybatisUtil() {
    }

    public static SqlSession getSqlSession(){
        SqlSession sqlSession = sessionThreadLocal.get();
        if(sqlSession==null){
            sqlSession = sqlSessionFactory.openSession();
            //设置事务自动提交 默认是false
//            sqlSession = sqlSessionFactory.openSession(true);
        }
        return sqlSession;
    }

    public static void closeSqlSession(){
        SqlSession sqlSession = sessionThreadLocal.get();
        if(sqlSession!=null){
            sqlSession.close();
            sessionThreadLocal.remove();
        }
    }
}

```

>**每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为核心的。SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。而 SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先配置的 Configuration 实例来构建出 SqlSessionFactory 实例。**
>
>**既然有了 SqlSessionFactory，顾名思义，我们可以从中获得 SqlSession 的实例。SqlSession 提供了在数据库执行 SQL 命令所需的所有方法。你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句** 
>
>以上均摘自Mybatis官方中文文档

#### 1.4 编写一个Demo

Entity：

```java
package com.coderzoe.entity;

/**
 * @author yhs
 * @date 2020/5/19 22:00
 * @description 数据库实体
 */

public class User {
    private Long id;
    private String name;
    private String password;

    public User() {
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}

```

Dao层接口：

```java
package com.coderzoe.dao;

import com.coderzoe.entity.User;

import java.util.List;

/**
 * @author yhs
 * @date 2020/5/19 22:06
 * @description User的Dao层
 */
public interface UserMapper {
    /**
     * @data: 2020/06/22 09:29
     * @author: yhs
     * @return: {@link List<User> }
     * @description: 查询全部用户
     */
    List<User> getUserList();
}

```

编写Dao层接口实现的xml文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.coderzoe.dao.UserMapper">
        <select id="getUserList" resultType="com.coderzoe.entity.User">
        select * from smbms.user
    </select>
</mapper>
```

将xml文件注册到Mybatis核心配置文件中：

```xml
    <mappers>
        <mapper resource="com/coderzoe/dao/UserMapper.xml"/>
    </mappers>
```

测试：

```java
    @Test
    public void test(){
        SqlSession sqlSession = MybatisUtil.getSqlSession();
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        List<User> userList = userMapper.getUserList();
        System.out.println(userList);
        MybatisUtil.closeSqlSession();
    }
```

**说明：**

> * 1. 通过UserMapper.xml文件实现UserMapper接口
>
>   2. UserMapper.xml文件其实是绑定UserMapper接口的
>
>      ```xml
>      <mapper namespace="com.coderzoe.dao.UserMapper">
>      ```
>
>      通过namespace参数绑定一个Dao层接口
>
>      ```xml
>      <select id="getUserList" resultType="com.coderzoe.entity.User">
>              select * from smbms.user
>      </select>
>      ```
>
>      通过select标签里的id绑定一个接口里的方法，resultType即为SQL查询返回的类型，这里我们查询的是用户表，所以返回结果为用户类
>
>   3. 需要将写好的xml文件注册到Mybatis核心配置文件中

**注意：**

> **可能犯的错**
>
> > 1. xml文件没用注册到Mybatyis核心配置文件里
> > 2. 绑定的接口错误
> > 3. 绑定的方法名错误
> > 4. 返回类型不对
> > 5. Maven项目中没用生成xml文件



### 2. 增删改查	

#### 2.1 增

```java
public interface UserMapper {
    void insertUser(User use);
}
```

```xml
<insert id="insertUser" parameterType="com.coderzoe.entity.User">
        insert into smbms.user(id,name,password) values(#{id},#{name},#{password})
</insert>
```

```java
@Test
public void insert(){
    SqlSession sqlSession = MybatisUtil.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    //增删改需要提交事务
    mapper.insertUser(new User((long)4,"lili","111"));
    sqlSession.commit();
    MybatisUtil.closeSqlSession();
}
```

#### 2.2 删

```java
public interface UserMapper {
    void delete(long id);
}
```

```xml
<delete id="delete" parameterType="long">
    delete from smbms.user where id = #{id}
</delete>
```

```java
@Test
public void delete(){
    SqlSession sqlSession = MybatisUtil.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    mapper.delete(4);
    sqlSession.commit();
    MybatisUtil.closeSqlSession();
}
```

#### 2.3 改

```java
public interface UserMapper {
    void update(User user);
}
```

```xml
<update id="update" parameterType="com.coderzoe.entity.User">
    update smbms.user set id=#{id},name=#{name},password=#{password} where id=#{id};
</update>
```

```java
@Test
public void update(){
    SqlSession sqlSession = MybatisUtil.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    mapper.update(new User(4L,"meimei","beautiful"));
    sqlSession.commit();
    sqlSession.close();
}
```

#### 2.4 查

```java
public interface UserMapper {
    User getUserById(long id);
}
```

```xml
<select id="getUserById" resultType="com.coderzoe.entity.User" parameterType="long">
    select * from smbms.user where id = #{id}
</select>
```

```java
@Test
public void query(){
    SqlSession sqlSession = MybatisUtil.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    User userById = mapper.getUserById((long) 1);
    System.out.println(userById);
    MybatisUtil.closeSqlSession();
}
```

**注：**

不同于JDBC，Mybatis中的事务自动提交默认是**false**，在进行增删改的时候，我们需要手动提交一下事务

```java
sqlSession.commit();
```

#### 2.5  Map的使用

通过上面我们可以看到，Mybatis插入或更新数据时，往往需要一个实体类作为参数，这个实体类的字段名要和Mybatis的占位符相同(下面的#{id},#{name},#{password})

```xml
<insert id="insertUser" parameterType="com.coderzoe.entity.User">
        insert into smbms.user(id,name,password) values(#{id},#{name},#{password})
</insert>
```

但在实际开发中，我们一个实体类可能包含多个字段，但我只需要保存这个实体类中的某几个字段，其他字段为空，我不关心。如果依然用实体类作为参数来传递，在构造时可能会很麻烦，还得将其他字段设空，此时我们就需要引入Map，用Map作为参数传递，**一个Map会对应一个实体类，Map中的每个元素对应实体类中的一个字段，Map的Key对应Mybatis的占位符 Value对应字段的值 这样就可以自定义字段 想任意截取实体类或重命名字段都可以**

```java
public interface UserMapper {
    void insertUser2(Map<String,Object> objectMap);
}
```

```xml
<insert id="insertUser2" parameterType="map">
    insert into smbms.user(id,name,password) values (#{userId},#{userPwd})
</insert>
```

```java
@Test
public void addUser2(){
    SqlSession sqlSession = MybatisUtil.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    HashMap<String, Object> map = new HashMap<>();
    map.put("userId",5);
    map.put("userPwd","mybatisPwd");
    mapper.insertUser2(map);
    sqlSession.commit();
    MybatisUtil.closeSqlSession();
}
```

可以看到我们传递的parameterType不再是User类，而是map，Mybatis中的占位符，也与map的key保持一致。

除了增加和修改，**在实际业务中的多条件查询中，我们也经常会传递Map，每个查询条件对应Map的一个元素**

### 3. 分页

Mybatis分页查询其实就是通过SQL语句的limit实现(**其实JPA底层也是limit实现的**)

```java
public interface UserMapper {
    List<User> findUserByPage(Map<String,Integer> map);
}
```

```xml
<select id="findUserByPage" resultType="com.coderzoe.entity.User" parameterType="map">
    select * from user limit #{startIndex},#{pageSize};
</select>
```

```java
@Test
public void findUserByPage(){
    SqlSession sqlSession = MybatisUtil.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    Map<String,Integer> map = new HashMap<>();
    map.put("startIndex",0);
    map.put("pageSize",3);
    List<User> userByPage = mapper.findUserByPage(map);
    System.out.println(userByPage);
    MybatisUtil.closeSqlSession();
}
```

### 4. 动态SQL

动态SQL其实在我们的现实业务中很常见，比如多条件搜索时，用户不一定把每个条件都填上，那样生成的SQL查询语句就是动态的。

假设现在我们有一张blog表

![image-20200622161521224](C:\Users\90617\AppData\Roaming\Typora\typora-user-images\image-20200622161521224.png)

对应的Java实体类

```java
package com.coderzoe.entity;

import java.util.Date;

/**
 * @author yhs
 * @date 2020/5/28 23:16
 * @description
 */
public class Blog {
    private String id;
    private String title;
    private String author;
    private Date createTime;
    private int views;

    public Blog() {
    }

    @Override
    public String toString() {
        return "Blog{" +
                "id='" + id + '\'' +
                ", title='" + title + '\'' +
                ", author='" + author + '\'' +
                ", createTime=" + createTime +
                ", views=" + views +
                '}';
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public Date getCreateTime() {
        return createTime;
    }

    public void setCreateTime(Date createTime) {
        this.createTime = createTime;
    }

    public int getViews() {
        return views;
    }

    public void setViews(int views) {
        this.views = views;
    }
}

```

用户可以通过标题作者或浏览量来进行筛选，如果采用JDBC操作，我们的代码量就会比较繁琐，SQL语句需要自己拼接：

```java
import com.coderzoe.dao.BaseDao;
import com.coderzoe.entity.Blog;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

/**
 * @author yhs
 * @date 2020/6/22 16:18
 * @description
 */
public class TestBlob {
    public void selectBlob(String title,String author,Integer views ){
        List<Blog> blogList = new ArrayList<>();
        List<Object> paramList  = new ArrayList<>();
        try {
            StringBuilder sql = new StringBuilder();
            sql.append("select * from blog");
            if(title!=null){
                sql.append(" where title like ?");
                paramList.add(title);
                if(author!=null){
                    sql.append(" and author like ?");
                    paramList.add(author);
                }
                if(views!= null){
                    sql.append(" and views = ?");
                    paramList.add(views);
                }
            }else {
                if(author!=null){
                    sql.append(" where author like ?");
                    paramList.add(author);
                    if(views != null){
                        sql.append(" and views = ?");
                        paramList.add(views);
                    }
                }else {
                    if (views!= null){
                        sql.append(" where views = ?");
                        paramList.add(views);
                    }
                }
            }
            Object[] params = paramList.toArray();
            //JDBC的配置信息无关紧要，所以用个工具类隐藏了
            Connection connection = BaseDao.getConnection();
            if(connection!=null){
                PreparedStatement preparedStatement = connection.prepareStatement(sql.toString());
                for(int i = 0; i < params.length; i++){
                    preparedStatement.setObject(i+1,params[i]);
                }
                ResultSet resultSet = preparedStatement.executeQuery();
                while (resultSet.next()){
                    Blog blog = new Blog();
                    blog.setId(resultSet.getString("id"));
                    blog.setTitle(resultSet.getString("title"));
                    blog.setAuthor(resultSet.getString("author"));
                    blog.setCreateTime(resultSet.getDate("create_time"));
                    blog.setViews(resultSet.getInt("views"));
                    blogList.add(blog);
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }

        System.out.println(blogList);
    }
}

```

可以看到，代码量是十分繁琐的，因为我们需要根据用户输入的结果来动态的生成sql语句，比如where的位置 and的位置等等，甚至有没有where 有没有and 都需要我们自己去判断，这还是只有三个参数，如果更多的搜索条件，代码逻辑势必会更加复杂。

如果使用Mybatis：

```java
public interface BlogMapper {
    List<Blog> queryBlogIf(Map<String,Object> map);
}
```

```xml
<select id="queryBlogIf"  parameterType="map" resultType="com.coderzoe.entity.Blog">
    select * from blog
    <where>
        <if test="title!=null">
            title = #{title}
        </if>
        <if test="author!=null">
            and author = #{author}
        </if>
        <if test="view!=null">
            and view > #{view}
        </if>
    </where>
</select>
```

```java
@Test
public void queryBlogIf(){
    SqlSession sqlSession = MybatisUtil.getSqlSession();
    BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
    Map<String,Object> map = new HashMap<>();
    List<Blog> blogs = mapper.queryBlogIf(map);
    System.out.println(blogs);

    map.put("title","Java");
    List<Blog> blogs1 = mapper.queryBlogIf(map);
    System.out.println(blogs1);
    MybatisUtil.closeSqlSession();
}
```

Mybatis提供了一些xml元素供我们做动态的生成sql，如果有更多的筛选字段，在后面重复<if></if>就好了

Mybatis替我们确定了where和and的位置，还有他们之间的空格。

当然，除了动态生成查询语句，增删改也可以

#### 4.1 增

```java
public interface BlogMapper {
    int insertBlog(Blog blog);
}
```

```xml
<sql id="insertKey">
    <trim suffixOverrides=",">
        <if test="id!=null">
            id,
        </if>
        <if test="title!=null">
            title,
        </if>
        <if test="author!=null">
            author,
        </if>
        <if test="create_time!=null">
            create_time,
        </if>
        <if test="views!=null">
            views,
        </if>
    </trim>
</sql>

<sql id="inserValue">
    <trim suffixOverrides=",">
        <if test="id!=null">
            #{id},
        </if>
        <if test="title!=null">
            #{title},
        </if>
        <if test="author!=null">
            #{author},
        </if>
        <if test="createTime!=null">
            #{createTime},
        </if>
        <if test="views!=null">
            #{views},
        </if>
    </trim>
</sql>

<insert id="insertBlog">
    insert into blog (<include refid="insertKey"/>) values(<include refid="inserValue"/>)
</insert>
```

```java
@Test
public void insertBlog(){
    SqlSession sqlSession = MybatisUtil.getSqlSession();
    BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
    Map<String,Object> map = new HashMap<>();
    map.put("id",CreateUUId.getId());
    map.put("author","殷华盛");
    mapper.insertBlog(map);
    sqlSession.commit();

    MybatisUtil.closeSqlSession();
}
```

这里我们使用了include标签，在日常开发中，很多操作可能是重复的代码，这些代码可以提取出来，作为公共代码被其他地方调用，Mybatis的xml文件也是一样，<sql id="inserValue"></sql>内的代码就是公共代码，

<include refid="inserValue"/>做的是调用这部分代码。

#### 4.2 改

```java
public interface BlogMapper {
    int updateBlog(Map<String,Object> map);
}
```

```xml
<update id="updateBlog" parameterType="map">
    update blog
    <set>
        <if test="title!=null">
            title = #{title},
        </if>
        <if test="author!=null">
            author = #{author},
        </if>
    </set>
    where id = #{id}
</update>
```

```java
@Test
public void updateBlog(){
    SqlSession sqlSession = MybatisUtil.getSqlSession();
    BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
    Map<String,Object> map = new HashMap<>();
    map.put("id","3e24216d7b7145bcb5c42f30da5c0d59");
    map.put("title","程序员的自我修养");
    map.put("author","殷华盛");
    mapper.updateBlog(map);

    sqlSession.commit();

    MybatisUtil.closeSqlSession();
}
```

#### 4.3 查

刚才已经讲了通过<where><if></if></where>动态生成SQL，这种需求往往是，用户填入多个字段，如果有用户填的那个字段就进行SQL查找，如果没有就查找全部。

设想这样一种需求，判断用户是否填入字段1，如果是就按字段1进行SQL查找，**否则，判断用户是否输入字段2，如果是就按字段2进行SQL查找**，这样一直下去，**如果都没有匹配的，就做一个默认选项分支**

```java
public interface BlogMapper {
    List<Blog> queryBlogChoose(Map<String,Object> map);
}
```

```xml
<select id="queryBlogChoose"  parameterType="map" resultType="com.coderzoe.entity.Blog">
    select * from blog
    <where>
        <choose>
            <when test="title!=null">
                title = #{title}
            </when>
            <when test="author!=null">
                and author = #{author}
            </when>
            <otherwise>
                and views > 5000
            </otherwise>
        </choose>
    </where>
</select>
```

```java
@Test
public void queryBlogChoose(){
    SqlSession sqlSession = MybatisUtil.getSqlSession();
    BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
    Map<String,Object> map = new HashMap<>();
    List<Blog> blogs = mapper.queryBlogChoose(map);
    System.out.println(blogs);

    map.put("title","Java");
    List<Blog> blogs1 = mapper.queryBlogChoose(map);
    System.out.println(blogs1);

    map.put("author","coderYin111");
    List<Blog> blogs2 = mapper.queryBlogChoose(map);
    System.out.println(blogs2);
}
```

可以看到，上面的需求为：

> 如果用户输入了title，则按title查找，其他不关心，如果没有输入title，就判断是否输入了author，如果是就按author查找，如果都没有就按views > 5000查找

与<if></if>标签的不同，<choose><when></when><otherwise></otherwise></choose>标签类似于switch-case语句，只会选择一个分支走，如果都没有就走default。



我们的SQL查询语句中，除了like  = 往往还会用到**in** 

in关键字后面跟的是一个集合，遍历集合里，符和需求的数据 。**Mybatis通过<foreach></foreach>标签来遍历集合。**

```java
public interface BlogMapper {
    List<Blog> queryBlogForeach(List<String> idList);
}
```

```xml
<select id="queryBlogForeach" parameterType="list" resultType="com.coderzoe.entity.Blog">
    select * from blog where id in 
    <foreach collection="list" item="item" index="index"
             open="(" close=")" separator=",">
        #{item}
    </foreach>
</select>
```

```java
@Test
public void queryBlogForeach(){
    SqlSession sqlSession = MybatisUtil.getSqlSession();
    BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
    List<String> idList = new ArrayList<>();
    idList.add("3e24216d7b7145bcb5c42f30da5c0d59");
    idList.add("bd64862c84ba4c3b8afa800ace342b14");
    idList.add("a82f3634cf5c4aa8b1bee5ab393a1bee");
    List<Blog> blogs = mapper.queryBlogForeach(idList);
    System.out.println(blogs);
}
```

<foreach></foreach>标签中的

<code>collection="list"</code> 指的是需要遍历的集合，这里的list是通过<code> parameterType="list"</code>传进来的。

<code>item="item" index="index"</code>这里是说遍历的每个元素取命叫item，下标叫index 

<code>open="(" close=")" separator=","></code> 指的是循环开始时，前面加 一个左括号<code>(</code>循环结束时后面加一个右括号<code>)</code> ，每个遍历的元素之间，加一个逗号<code>,</code> 构造SQL

```sql
select * from blog where id in (param1,param2,param3...)
```

### 5. 多表联查

### 6. 使用注解开发

### 7. Mybatis核心配置文件

Mybatis配置信息

>configuration（配置）
>properties（属性）
>settings（设置）
>typeAliases（类型别名）
>typeHandlers（类型处理器）
>objectFactory（对象工厂）
>plugins（插件）
>environments（环境配置）
>environment（环境变量）
>transactionManager（事务管理器）
>dataSource（数据源）
>databaseIdProvider（数据库厂商标识）
>mappers（映射器）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource=""/>

    <settings>
        <setting name="" value=""/>
    </settings>

    <typeAliases></typeAliases>

    <typeHandlers></typeHandlers>

    <objectFactory type=""></objectFactory>

    <plugins>
        <plugin interceptor=""></plugin>
    </plugins>

    <environments default="">
        <environment id="">
            <transactionManager type=""></transactionManager>
            <dataSource type=""></dataSource>
        </environment>
    </environments>

    <databaseIdProvider type=""></databaseIdProvider>
    <mappers>
        <mapper resource=""/>
    </mappers>
</configuration>
```

#### 3.1 properties

通过properties属性来引入配置文件，这样就可以只改动相关配置文件，不用改动Mybatis配置。

创建db.properties

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/smbms?useSSL=false&useUnicode=true&characterEncoding=UTF-8
username=root
password=123456
```

更改Mybatis配置信息

```xml
<environments default="mysql_developer">
    <environment id="mysql_developer">
        <transactionManager type="jdbc"/>
        <dataSource type="pooled">
            <property name="driver" value="${driver}"/>
            <property name="url" value="${url}"/>
            <property name="username" value="${username}"/>
            <property name="password" value="${password}"/>
        </dataSource>
    </environment>
</environments>
```

这样我们以后只需要更改db.properties文件中的信息即可。

#### 3.2 typeAliases

在我们写的mapper.xml中可以看到，当我们传参或返回为实体类时，**resultType/parameterType**往往是包名加类名，这样如果包名过深，或者类名过于不好辨识，我们能不能取一个别名呢？用这个别名代替包名+类名

```xml
<select id="getUserById" resultType="com.coderzoe.entity.User" parameterType="long">
    select * from smbms.user where id = #{id}
</select>
```

当然可以！typeAliases标签就是做这个的。

```xml
<typeAliases>
    <typeAlias type="com.coderzoe.entity.User" alias="yhs"/>
</typeAliases>
```

```xml
<select id="getUserList" resultType="yhs">
    select * from smbms.user
</select>
```

除此以外，typeAliases还可以指定包名，Mybatis 会在包名下面搜索需要的 Java Bean，在没有注解的情况下，此时默认的别名就是类名(但首字母会小写)，若有注解，则别名为其注解值。 
```xml
<typeAliases>
    <package name="com.coderzoe.entity"/>
</typeAliases>
```
```xml
<select id="getUserList" resultType="user">
    select * from smbms.user
</select>
```

#### 3.3 settings

Mybatis settings标签内容非常多，主要提供的是一些Mybatis的配置，下面附上官方文档的详细信息

| 设置名                           | 描述                                                         | 有效值                                                       | 默认值                                                |
| -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------------- |
| cacheEnabled                     | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。     | true\| false                                                 | true                                                  |
| lazyLoadingEnabled               | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。 | true \| fase                                                 | false                                                 |
| aggressiveLazyLoading            | 开启时，任一方法的调用都会加载该对象的所有延迟加载属性。 否则，每个延迟加载属性会按需加载（参考 `lazyLoadTriggerMethods`)。 | true \| false                                                | false（3.4.1 及之前的版本中默认为 true）              |
| multipleResultSetsEnabled        | 是否允许单个语句返回多结果集（需要数据库驱动支持）。         | true \| false                                                | true                                                  |
| useColumnLabel                   | 使用列标签代替列名。实际表现依赖于数据库驱动，具体可参考数据库驱动的相关文档，或通过对比测试来观察。 | true \| false                                                | true                                                  |
| useGeneratedKeys                 | 允许 JDBC 支持自动生成主键，需要数据库驱动支持。如果设置为 true，将强制使用自动生成主键。尽管一些数据库驱动不支持此特性，但仍可正常工作（如 Derby）。 | true \| false                                                | False                                                 |
| autoMappingBehavior              | 指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示关闭自动映射；PARTIAL 只会自动映射没有定义嵌套结果映射的字段。 FULL 会自动映射任何复杂的结果集（无论是否嵌套）。 | NONE, PARTIAL, FULL                                          | PARTIAL                                               |
| autoMappingUnknownColumnBehavior | 指定发现自动映射目标未知列（或未知属性类型）的行为。<br />`NONE`: 不做任何反应<br />`WARNING`: 输出警告日志（`'org.apache.ibatis.session.AutoMappingUnknownColumnBehavior'` 的日志等级必须设置为 `WARN`）<br />`FAILING`: 映射失败 (抛出 `SqlSessionException`) | NONE, WARNING, FAILING                                       | NONE                                                  |
| defaultExecutorType              | 配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（PreparedStatement）； BATCH 执行器不仅重用语句还会执行批量更新。 | SIMPLE REUSE BATCH                                           | SIMPLE                                                |
| defaultStatementTimeout          | 设置超时时间，它决定数据库驱动等待数据库响应的秒数。         | 任意正整数                                                   | 未设置 (null)                                         |
| defaultFetchSize                 | 为驱动的结果集获取数量（fetchSize）设置一个建议值。此参数只可以在查询设置中被覆盖。 | 任意正整数                                                   | 未设置 (null)                                         |
| defaultResultSetType             | 指定语句默认的滚动策略。（新增于 3.5.2）                     | FORWARD_ONLY \| SCROLL_SENSITIVE \| SCROLL_INSENSITIVE \| DEFAULT（等同于未设置） | 未设置 (null)                                         |
| safeRowBoundsEnabled             | 是否允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为 false。 | true \| false                                                | False                                                 |
| safeResultHandlerEnabled         | 是否允许在嵌套语句中使用结果处理器（ResultHandler）。如果允许使用则设置为 false。 | true \| false                                                | True                                                  |
| mapUnderscoreToCamelCase         | 是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。 | true \| false                                                | False                                                 |
| localCacheScope                  | MyBatis 利用本地缓存机制（Local Cache）防止循环引用和加速重复的嵌套查询。 默认值为 SESSION，会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地缓存将仅用于执行语句，对相同 SqlSession 的不同查询将不会进行缓存。 | SESSION \| STATEMENT                                         | SESSION                                               |
| jdbcTypeForNull                  | 当没有为参数指定特定的 JDBC 类型时，空值的默认 JDBC 类型。 某些数据库驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。 | JdbcType 常量，常用值：NULL、VARCHAR 或 OTHER。              | OTHER                                                 |
| lazyLoadTriggerMethods           | 指定对象的哪些方法触发一次延迟加载。                         | 用逗号分隔的方法列表。                                       | equals,clone,hashCode,toString                        |
| defaultScriptingLanguage         | 指定动态 SQL 生成使用的默认脚本语言。                        | 一个类型别名或全限定类名。                                   | org.apache.ibatis.scripting.xmltags.XMLLanguageDriver |
| defaultEnumTypeHandler           | 指定 Enum 使用的默认 `TypeHandler` 。（新增于 3.4.5）        | 一个类型别名或全限定类名。                                   | org.apache.ibatis.type.EnumTypeHandler                |
| callSettersOnNulls               | 指定当结果集中值为 null 的时候是否调用映射对象的 setter（map 对象时为 put）方法，这在依赖于 Map.keySet() 或 null 值进行初始化时比较有用。注意基本类型（int、boolean 等）是不能设置成 null 的。 | true \| false                                                | false                                                 |
| returnInstanceForEmptyRow        | 当返回行的所有列都是空时，MyBatis默认返回 `null`。 当开启这个设置时，MyBatis会返回一个空实例。 请注意，它也适用于嵌套的结果集（如集合或关联）。（新增于 3.4.2） | true \| false                                                | false                                                 |
| logPrefix                        | 指定 MyBatis 增加到日志名称的前缀。                          | 任何字符串                                                   | 未设置                                                |
| logImpl                          | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。        | SLF4J \| LOG4J \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置                                                |
| proxyFactory                     | 指定 Mybatis 创建可延迟加载对象所用到的代理工具。            | CGLIB \| JAVASSIST                                           | JAVASSIST （MyBatis 3.3 以上）                        |
| vfsImpl                          | 指定 VFS 的实现                                              | 自定义 VFS 的实现的类全限定名，以逗号分隔。                  | 未设置                                                |
| useActualParamName               | 允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的项目必须采用 Java 8 编译，并且加上 `-parameters` 选项。（新增于 3.4.1） | true \| false                                                | true                                                  |
| configurationFactory             | 指定一个提供 `Configuration` 实例的类。 这个被返回的 Configuration 实例用来加载被反序列化对象的延迟加载属性值。 这个类必须包含一个签名为`static Configuration getConfiguration()` 的方法。（新增于 3.2.3） | 一个类型别名或完全限定类名。                                 | 未设置                                                |
| shrinkWhitespacesInSql           | Removes extra whitespace characters from the SQL. Note that this also affects literal strings in SQL. (Since 3.5.5) | true \| false                                                | false                                                 |

**常用配置：**

```xml
<settings>
    <!-- 配置开启驼峰 -->
    <setting name="mapUnderscoreToCamelCase" value="true"/>
    <!-- 配置日志 需要导入LOG4J的包 -->
    <setting name="logImpl" value="LOG4J"/>
</settings>
```

#### 3.4 mappers

在mappers下将我们写的mapper.xml文件进行注册绑定

```xml
<mappers>
    <mapper resource="com/coderzoe/dao/UserMapper.xml"/>
</mappers>
```

这里其实有多种绑定方法

* 1. 通过resource绑定xml文件

     ```xml
     <mappers>
         <mapper resource="com/coderzoe/dao/UserMapper.xml"/>
     </mappers>
     ```

  2. 通过class绑定Dao层接口

     ```xml
     <mappers>
         <mapper class="com.coderzoe.dao.UserMapper"/>
     </mappers>
     ```

     **注：用这种方法有两点注意内容：**

     * xml配置文件要和接口名称保持一致
     * xml配置文件要和接口在同一个包下

  3. 使用扫描包进行注入绑定

     ```xml
     <mappers>
         <package name="com.coderzoe.dao"/>
     </mappers>
     ```

     这个的好处是不用每次写完一个xml文件都在mappers下进行注册，注册一个包，那这个包下的所有mapper都会被注册。**但也需要注意问题，注意的问题点与2相同**

     **注：用这种方法有两点注意内容：**

     * xml配置文件要和接口名称保持一致
     * xml配置文件要和接口在同一个包下

## Mybatis-Plus





