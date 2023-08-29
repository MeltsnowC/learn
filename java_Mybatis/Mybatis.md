[TOC]



# 一、Mybatis简介

## 1、MyBatis历史
---
MyBatis最初是Apache的一个开源项目iBatis, 2010年6月这个项目由Apache Software Foundation迁
移到了Google Code。随着开发团队转投Google Code旗下， iBatis3.x正式更名为MyBatis。代码于
2013年11月迁移到Github。

iBatis一词来源于“internet”和“abatis”的组合，是一个基于Java的持久层框架。 iBatis提供的持久层框架
包括SQL Maps和Data Access Objects（DAO）。

定义：MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

note: **Mybatis其实就是对JDBC的封装**

---
## 2、MyBatis特性
1) MyBatis是可以定制化SQL、存储过程以及高级映射的优秀持久层框架。
2) MyBatis避免了几乎所有的JDBC代码和手动设置参数以及获取结果集
3) MyBatis可以使用简单的XML或注解用于配置和原始映射，将接口和Java的POJO（Plain Old Java Objects，普通的Java对象）映射成数据库中的记录
4) MyBatis是一个半自动的ORM（Object Relation Mapping）框架.(Hibernate是全自动)

---

## 3、和其它持久层框架相比的对比
- JDBC
    - SQL夹杂在Java代码中耦合度高，导致硬编码内伤，（SQL语句的修改就会导致重新编码）
    - 维护不易且实际开发需求中SQL有变化，频繁更改的情况多见
    - 代码冗长，不宜维护，开发效率低
- Hibernate和JPA
    - 操作简便，开发效率高
    - 程序中的长难复杂 SQL 需要绕过框架
    - 内部自动生产的 SQL，不容易做特殊优化
    - 基于全映射的全自动框架，大量字段的 POJO 进行部分映射时比较困难。
    - 反射操作太多，导致数据库性能下降
- MyBatis 
    - 轻量级，性能出色
    - SQL 和 Java 编码分开，功能边界清晰。Java代码专注业务、SQL语句专注数据
    - 开发效率稍逊于HIbernate，但是完全能够接受

---

# 二、搭建MyBatis
---
## 1、开发环境
---
IDE：idea 2019.2

构建工具：maven 3.5.4

MySQL版本：MySQL 5.7

MyBatis版本：MyBatis 3.5.7

---

## 2、创建maven工程

- 打包方式设置为`jar`
- 引入相关依赖

```java
<dependencies>
    <!-- Mybatis核心 -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.7</version>
    </dependency>
    <!-- junit测试 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <!-- MySQL驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.30</version>
    </dependency>
    <!--log4j日志-->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
</dependencies>
```
---

## 3、创建MyBatis的核心配置文件
> 习惯上命名为mybatis-config.xml，这个文件名仅仅只是建议，并非强制要求。将来整合Spring之后，这个配置文件可以省略，所以大家操作时可以直接复制、粘贴。
> 
> 核心配置文件主要用于配置连接数据库的环境以及MyBatis的全局配置信息
>
> 核心配置文件存放的位置是src/main/resources目录下

`mybatis-config.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!--引入properties文件，此时就可以${属性名}的方式访问属性值-->
    <properties resource="jdbc.properties"></properties>

    <settings>
        <!--将表中字段的下划线自动转换为驼峰-->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!--开启延迟加载-->
        <setting name="lazyLoadingEnabled" value="true"/>
    </settings>

    <typeAliases>
        <!--
        typeAlias：设置某个具体的类型的别名
        属性：
            type：需要设置别名的类型的全类名
            alias：设置此类型的别名，若不设置此属性，该类型拥有默认的别名，即类名且不区分大小写
            若设置此属性，此时该类型的别名只能使用alias所设置的值(默认时也不区分大小写)
        -->
<!--        <typeAlias type="com.cheng.mybatis.pojo.User"></typeAlias>-->
        <!--<typeAlias type="com.cheng.mybatis.pojo.User" alias="abc">
        </typeAlias>-->
        <!--以包为单位，设置该包下所有的类型都拥有默认的别名，即类名且不区分大小写-->
        <package name="com.cheng.mybatis.pojo"/>
    </typeAliases>




    <!--设置连接数据库的环境-->
    <!-- environments：设置多个连接数据库的环境
            属性：
                default：设置默认使用的环境的id
-->
    <environments default="development">
        <!-- environment：设置具体的连接数据库的环境信息
            属性：
                id：设置环境的唯一标识，可通过environments标签中的default设置某一个环境的id，
            表示默认使用的环境
        -->
        <environment id="development">
            <!-- transactionManager：设置事务管理方式
                属性：
                    type：设置事务管理方式，type="JDBC|MANAGED"
                    type="JDBC"：设置当前环境的事务管理都必须手动处理
                    type="MANAGED"：设置事务被管理，例如spring中的AOP
            -->
            <transactionManager type="JDBC"/>
            <!--
                dataSource：设置数据源
                属性：
                type：设置数据源的类型，type="POOLED|UNPOOLED|JNDI"
                type="POOLED"：使用数据库连接池，即会将创建的连接进行缓存，下次使用可以从
                缓存中直接获取，不需要重新创建
                type="UNPOOLED"：不使用数据库连接池，即每次使用连接都需要重新创建
                type="JNDI"：调用上下文中的数据源
                -->
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url"
                          value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <!--引入映射文件-->
    <mappers>
<!--        <mapper resource="com/cheng/mybatis/mappers/UserMapper.xml"/>-->
        <!--
            以包为单位，将包下所有的映射文件引入核心配置文件
            注意：此方式必须保证mapper接口和mapper映射文件必须在相同的包下(包名相同)
            -->
        <package name="com.cheng.mybatis.mapper"/>
    </mappers>
</configuration>
```

## 4、创建mapper接口
> MyBatis中的mapper接口相当于以前的dao。但是区别在于，mapper仅仅是接口，我们不需要
提供实现类。相当于jdbc中的XXXDAO接口，mapper文件不需要编写XXXDAOimpl实现类，而是直接通过XXmapper.xml映射文件完成操作。


`EmpMapper.java`

```java
public interface EmpMapper {

    /**
     * 查询所有的员工信息
     */
    List<Emp> getAllEmp();

    /**
     * 查询员工以及员工所对应的部门信息
     */
    Emp getEmpAndDep(@Param("eid") Integer eid);

    /**
     * 通过分布查询查询员工以及员工所对应的部门信息
     * 分布查询第一步：查询员工信息
     */
    Emp getEmpAndDeptByStepOne(@Param("eid") Integer eid);

    /**
     * 通过分布查询查询部门对应的员工信息
     * 分布查询第二部:根据did的值，查询对应的员工信息
     */
    List<Emp> getDeptAndEmpByStepTwo(@Param("did") Integer did);
}
```

## 5、创建MyBatis的映射文件
---

相关概念：ORM（Object Relationship Mapping）对象关系映射。
- 对象：Java的实体类对象
- 关系：关系型数据库
- 映射：二者之间的对应关系


| Java概念| 数据库概念|
|:---:|:---:|
|类|表|
|属性|字段|
|对象|记录\行|

> 1、映射文件的命名规则：
>
>表所对应的实体类的类名+Mapper.xml
>
>例如：表t_user，映射的实体类为User，所对应的映射文件为UserMapper.xml
因此一个映射文件对应一个实体类，对应一张表的操作,存放在src/main/java/(com/cheng/mybatis/)mapper目录下
>
>MyBatis映射文件用于编写SQL，访问以及操作表中的数据
>
>MyBatis映射文件存放的位置是src/main/resources/(com/cheng/mybatis/)mappers目录下
>
>2、MyBatis中可以面向接口操作数据，要保证两个一致：
 >>- mapper接口的`全类名`和映射文件的`命名空间`（namespace）保持一致
 >>
 >>- mapper接口中方法的`方法名`和映射文件中编写SQL的标签的`id属性`保持一致

`UserMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.cheng.mybatis.mapper.UserMapper">
    <!--int insertUser();-->
    <insert id="insertUser">
        insert into t_user values(null,'张三','123',23,'女','123@qq.com')
    </insert>
</mapper>
```

## 6、通过junit测试功能
```java
 public void testMyBatis() throws IOException {
        //读取MyBatis的核心配置文件
        InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
        //创建sqlsessionFactoryBuilder对象
        SqlSessionFactoryBuilder sessionFactoryBuilder = new SqlSessionFactoryBuilder();
        //通过核心配置文件所对应的字节输入流创建工厂类SqlSessionFactory，获取sqlSessionFactory对象
        SqlSessionFactory sqlSessionFactory = sessionFactoryBuilder.build(is);
        //利用SqlSessionFactory生产SqlSession对象，
     
        //创建SqlSession对象，此时通过SqlSession对象所操作的sql都必须手动提交或回滚事务(没有设置自动提交)
        //SqlSession sqlSession = sqlSessionFactory.openSession();
        //创建SqlSession对象，此时通过SqlSession对象所操作的sql都会自动提交
        SqlSession sqlSession = sqlSessionFactory.openSession(true);
     
        //获取mapper接口的代理实现类对象（用到底层代理模式，返回一个接口对应的实现类对象）
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
     
        //测试功能：调用UserMapper接口中的方法，就可以根据UserMapper的全类名匹配元素文件，通过调用的方法名匹配
        //映射文件中的SQL标签，并执行标签中的SQL语句
        int result = mapper.insertUser();
        //提交事物
        //sqlSession.commit();

        System.out.println("result"+result);
    }
```

> - SqlSession：代表Java程序和数据库之间的会话。（HttpSession是Java程序和浏览器之间的会话）
>
> - SqlSessionFactory：是“生产”SqlSession的“工厂”。
>
> - 工厂模式：如果创建某一个对象，使用的过程基本固定，那么我们就可以把创建这个对象的
相关代码封装到一个“工厂类”中，以后都使用这个工厂类来“生产”我们需要的对象。
---
## 7、加入log4j日志功能
- 添加依赖
- 加入log4j的配置文件
> log4j的配置文件名为log4j.xml，存放的位置是src/main/resources目录下

`log4j.xml`
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">
    <appender name="STDOUT" class="org.apache.log4j.ConsoleAppender">
        <param name="Encoding" value="UTF-8" />
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%-5p %d{MM-dd HH:mm:ss,SSS}
%m (%F:%L) \n" />
        </layout>
    </appender>
    <logger name="java.sql">
        <level value="debug" />
    </logger>
    <logger name="org.apache.ibatis">
        <level value="info" />
    </logger>
    <root>
        <level value="debug" />
        <appender-ref ref="STDOUT" />
    </root>
</log4j:configuration>
```

> ### 日志的级别
FATAL(致命)>ERROR(错误)>WARN(警告)>INFO(信息)>DEBUG(调试)
从左到右打印的内容越来越详细

# 三、核心配置文件详解
---
核心配置文件中标签的顺序：

properties?,settings?,typeAliases?,typeHandlers?,objectFactory?,objectWrapperFactory?,reflectorF
actory?,plugins?,environments?,databaseIdProvider?,mappers?

`log4j.xml`
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">
    <appender name="STDOUT" class="org.apache.log4j.ConsoleAppender">
        <param name="Encoding" value="UTF-8" />
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%-5p %d{MM-dd HH:mm:ss,SSS}
%m (%F:%L) \n" />
        </layout>
    </appender>
    <logger name="java.sql">
        <level value="debug" />
    </logger>
    <logger name="org.apache.ibatis">
        <level value="info" />
    </logger>
    <root>
        <level value="debug" />
        <appender-ref ref="STDOUT" />
    </root>
</log4j:configuration>
```

> ### 日志的级别
FATAL(致命)>ERROR(错误)>WARN(警告)>INFO(信息)>DEBUG(调试)
从左到右打印的内容越来越详细

# 三、核心配置文件详解
---
核心配置文件中标签的顺序：

properties?,settings?,typeAliases?,typeHandlers?,objectFactory?,objectWrapperFactory?,reflectorF
actory?,plugins?,environments?,databaseIdProvider?,mappers?

# 四、MyBatis的增删改查

1、添加

`package com.cheng.mybatis.mapper.UserMapper;`

```java
public interface UserMapper {
    /**
     * Mybatis实体类两个一致：
     * 1.映射文件的namespace和mapper接口的全类名一致
     * 2.映射文件的SQl语句的id和mapper接口的方法名一致
     *
     * 表--实体类--mapper接口--映射文件--核心配置文件
     */
    /**
     * 添加用户信息
     */
    int insertUser();
}
```

`resources/com/cheng/mybatis/mapper/UserMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.cheng.mybatis.mapper.UserMapper">
    <!--int insertUser();-->
    <insert id="insertUser">
        insert into t_user values(null,'张三','123',23,'女','123@qq.com')
    </insert>
</mapper>
```

2、删除

```java
    void deleteUser();
```

```xml
<!--int deleteUser();-->
    <delete id="deleteUser">
        delete from t_user where id = 3
    </delete>
```

3、修改
```java
    void updateUser();
```

```xml
<!--int updateUser();-->
    <update id="updateUser">
        update t_user set username='ybc',password='123' where id = 6
    </update>
```
4、查询一个实体类对象
```java
    User getUserById();
```
>  查询功能的标签必须设置resultType或resultMap
  >> resultType：设置默认的映射关系（一对一，以及属性名和列名相同）
  >>
  >> resultMap:设置自定义的映射关系(一对多，以及属性名和列名不同的情况)

```xml
<!-- User getUserById(); -->
    <select id="getUserById" resultType="com.cheng.mybatis.pojo.User">
        select * from t_user where id = 1
    </select>
```

5、查询集合

```java
    List<User> getAllUser();
```
![image.png](attachment:image.png)

![hello](E:\笔记\java_Redis\image\mybatis-4-1.png)

 

> 由于在全局配置中设置了别名，因此，可以resulttype可以直接写类名对应默认别名。
```xml
<!--    List<User> getAllUser();-->
    <select id="getAllUser" resultType="user">
        select * from t_user
    </select>
```

> 注意：
1、查询的标签select必须设置属性resultType或resultMap，用于设置实体类和数据库表的映射
关系
resultType：自动映射，用于属性名和表中字段名一致的情况
resultMap：自定义映射，用于一对多或多对一或字段名和属性名不一致的情况
2、当查询的数据为多条时，不能使用实体类作为返回值，只能使用集合，否则会抛出异常
TooManyResultsException；但是若查询的数据只有一条，可以使用实体类或集合作为返回值


# 五、MyBatis获取参数值的两种方式 :❤️❤️❤️❤️❤️
(mybatis-demo2)
---

MyBatis获取参数值的两种方式：`${}`和`#{}`

${}的本质就是字符串拼接，#{}的本质就是占位符赋值

- ${}使用字符串拼接的方式拼接sql，若为字符串类型或日期类型的字段进行赋值时，需要手动加单引号；
- 但是#{}使用占位符赋值的方式拼接sql，此时为字符串类型或日期类型的字段进行赋值时，可以自动添加单引号

---

## Mybatis获取参数值的各种情况：

### 1. mapper接口的参数为获取单个的字面量类型

---

可以使用#{}和${}
> 以`任意名称`来获取参数值，但是注意${}外要添加单引号

`XXMapper.java`

```java
  /*传入单个参数的时候，根据参数返回对象记录*/
    User getUserByUserName(String username);
```


`XXMapper.xml`

```xml
    <!-- User getUserByUserName(String username);-->
    <select id="getUserByUserName"  resultType="user">
        <!--select * from t_user where username= #{username} -->
        select * from t_user where username = '${username}'
    </select> 

```
### 2. mapper接口的参数为多个字面量类型
---
此时mybatis会将这些参数放入一个map集合中，以两种方式进行存储
- 以[arg0 arg1 ...]为键，以参数为值
- 以[param1 param2 ...]为键，以参数为值

因此只需要通过#{}或 ${}
> 以`键`的方式访问值即可，仍需要注意${}外要加单引号。可以混合使用,例如参数为(name,age) 可以用#{arg0} #{param2}来提取参数

`XXMapper.java`
```java
 /*传入多个参数时，验证登录功能*/
    User checkLogin(String username,String password);
```
`XXMapper.xml`
```xml
<!--    User checkLogin(String username,String password);-->
    <select id="checkLogin" resultType="user">
        select * from t_user where username = #{arg0} and password = #{arg1}
    </select>
```

### 3. map集合类型的参数

---

若mapper接口方法的参数有多个时，可以手动将这些参数放在一个map中存储。

只需要通过 ${}和#{}
> 访问map集合的`键`就可以获取相对应的值，注意${}需要手动加单引号

`XXMapper.java`
```java
    /*用map方式传入多个参数*/
    User checkLoginByMap(Map<String,Object> map);
```
`XXMapper.xml`
```xml
<!--    User checkLoginByMap(Map<String,Object> map);-->
    <select id="checkLoginByMap" resultType="user">
        select * from t_user where username = #{username} and password = #{password}
    </select>
```

### 4、实体类类型的参数

---

mapper接口方法的参数是实体类类型的参数

此时可以使用${}和#{}，
> 通过访问实体类对象中的`属性名`获取属性值，注意${}需要手动加单引号

`XXMapper.java`
```java
    /*插入一个实体对象*/
    int insertUser(User user);
```
`XXMapper.xml`
```xml
<!--    int insertUser(User user);-->
    <insert id="insertUser">
        insert into t_user values(null,#{username},#{password},#{age},#{sex},#{email})
    </insert>
```

### 5、使用@Param标识参数
---
可以通过@Param注解标识mapper接口中的方法参数

此时mybatis会将这些参数放入一个map集合中，以两种方式进行存储
- 以@param注解的值为键，以参数为值
- 以[param1 param2 ...]为键，以参数为值

> 因此只需要通过#{}或 ${}以`键`的方式访问值即可，仍需要注意${}外要加单引号。
---

# 六、MyBatis的各种查询功能
---
## 1. 查询一个实体类对象
`XXMapper.java`

```java
    /**
    * 根据用户id查询用户信息
    * @param id
    * @return
    */
    User getUserById(@Param("id") int id);
```
`XXMapper.xml`
```xml
<!--User getUserById(@Param("id") int id);-->
    <select id="getUserById" resultType="User">
        select * from t_user where id = #{id}
    </select>

```
## 2、查询一个list集合
`XXMapper.java`

```java
   /**
     * 查询所有用户的记录
     */
    List<User> getAllUser();
```
`XXMapper.xml`

```xml

<!--List<User> getAllUser();-->
    <select id="getAllUser" resultType="User">
        select * from t_user
    </select>
```
## 3、查询单个数据
---
`XXMapper.java`
```java
    /**
     * 查询记录总数
     */
    Integer getCount();
```
`XXMapper.xml`
```xml
<!--    Integer getCount();-->
    <select id="getCount" resultType="java.lang.Integer">
        select count(*) from t_user
    </select>
```
## 4、查询一条数据为map集合
---
`XXMapper.java`
```java
    /**
     * 根据查询用户信息，用map返回
     */
    Map<String,Object> getUserByIdToMap(@Param("id") Integer id);
```
`XXMapper.xml`
```xml
<!--    Map<String,Object> getUserByIdToMap(@Param("id") Integer id);-->
    <select id="getUserByIdToMap" resultType="map">
        select * from t_user where id = #{id}
    </select>
```
## 5、查询多条数据为map集合
---
- 方法一：

`XXMapper.java`
```java
/**
     * 查询所有用户信息，保存至map中
     */
    List<Map<String,Object>> getAllUserToMap();
```
`XXMapper.xml`
```xml
<!--    List<Map<String,Object>> getAllUserToMap();-->
    <select id="getAllUserToMap" resultType="map">
        select * from t_user
    </select>
```
- 方法二
`XXMapper.java`
```java
   
    @MapKey("id")  //以集合中查询到的某个字段为键，以查询到的map集合为值，
    // 这里是以id为键，id所在的map集合为值
    Map<String,Object> getAllUserToMap();
```
> 将表中的数据以map集合的方式查询，一条数据对应一个map；若有多条数据，就会产生多个map集合，并
且最终要以一个map的方式返回数据，此时需要通过@MapKey注解设置map集合的键，值是每条数据所对应的
map集合
`XXMapper.xml`
```xml
<!--    List<Map<String,Object>> getAllUserToMap();-->
    <select id="getAllUserToMap" resultType="map">
        select * from t_user
    </select>
```

> 1.若查询出的数据只有一条，
     >> - 可以通过实体类对象
     >>
     >> - list集合接收（即方法的返回值是实体类或者集合）
     >>
     >> - 通过map集合接收,结果以字段为键，以字段值为值。
     >>
     >> {password=123, sex=女, id=2, age=23, email=123@qq.com, username=张三}
>
> 2.若查询出的数据有多条
     >> - 则可以通过实体类类型的List集合接收，
     >>
     >> - 可以通过map类型的List集合接收，
     >>
     >> - 可以在mapper接口方法上添加@MapKey注解，此时就可以将每条数据转化成的map集合作为值，以某个字段的值作为键，放在同一个map集合中。map<key,map<>>
     >>
     >> 注意：一定不能通过实体类对象接收，否则会报异常TooManyResultsException
>
> Mybatis设置了默认的类型别名
     >> java.lang.Integer-->int,Integer
     >>
     >> int-->_int,_integer
     >>
     >> Map-->map
     >>
     >> String-->string


# 七、特殊SQL的执行
---
## 模糊查询

`XXMapper.java`

```java
    /**
     * 根据用户名模糊查询用户信息
     */
    List<User> getUserByLike(@Param("username") String username);
```
`XXMapper.xml`

```xml
<!--    List<User> getUserByLike(@Param("username") String username);-->
    <select id="getUserByLike" resultType="User">
<!--  这里使用#{}的话，无法正确表示占位符，在引号中，mysql会按照字符串来解析？，所以无法实现占位，
因此可以使用${}来拼接字符串    -->
       <!-- select * from t_user where username like '%${username}%'-->
        <!-- 也可以使用concat进行字符串拼接-->
       <!-- select * from t_user where username like concat('%',#{username},'%')-->
        select * from t_user where username like "%"#{username}"%"
    </select>
```
>模糊查询：
> - 可以使用"%"#{username}"%"进行拼接
> - 也可以使用concat('%',#{username},'%')
> - 或者'%${username}%'
---
## 批量删除
`XXMapper.java`
```java
/**
     * 批量删除
     */
    int deleteMore(@Param("ids")String ids);   //ids是id拼接之后的结果类似于：1，2，3，4
```
`XXMapper.xml`

```xml
<!--    int deleteMore(@Param("ids")String ids);-->
    <delete id="deleteMore">
        delete from t_user where id in (${ids})
    </delete>
    
```

> 不能使用#{}，因为会在in () 中自动加上'',这是不正确是sql语句
>
> 会报错Truncated incorrect DOUBLE value: '1,2'
>
> 应该使用${}

## 动态设置表名，查询指定表中的数据(分表之后)
`XXMapper.java`

```java
    /*
    * 查询指定表中的数据
    * */
    List<User> getUserByTableName(@Param("tableName") String tableName);
```
`XXMapper.xml`
```xml
<!--    List<User> getUserByTableName(@Param("tableName") String tableName);-->
    <select id="getUserByTableName" resultType="User">
        select * from ${tableName}
    </select>
```
> 动态表名查询:水平分表的时候可能会用到
    >>必须使用${}

## 添加功能获取自增的主键
`XXMapper.java`
```java
    /**
     * 添加一个用户信息，并返回自增的id
     */
    void insertUser(User user);
```
`XXMapper.xml`
```xml
    <insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
        insert into t_user values(null,#{username},#{password},#{age},#{sex},#{email})
    </insert>
```
> 获取自增的逐渐需要设置insert标签的属性：
 >> useGeneratedKeys：设置使用自增的主键
 >>
 >> keyProperty：因为增删改有统一的返回值是受影响的行数，因此只能将获取的自增的主键放在传输的参
数user对象的某个属性中

# 八、自定义映射resultMap(mybatis-demo3)
## 1. resultMap处理字段和属性的映射关系

若字段名和实体类中的属性名不一致，则可以通过resultMap设置自定义映射

> ### 解决字段名和属性名不一致的情况：
    >> a>为字段取别名，保持和属性名一致
    >>
    >> b>设置全局配置，将下划线自动映射为驼峰
    >>> `<setting name="mapUnderscoreToCamelCase" value="true"/>`
    >>
    >> c>通过resultMap设置自定义的映射关系

```xml    
    <resultMap id="empResultMap" type="Emp">
      <id property="eid" column="eid"></id>
      <result property="empName" column="emp_name"></result>
      <result property="age" column="age"></result>
      <result property="sex" column="sex"></result>
      <result property="email" column="email"></result>
    </resultMap>
```

> resultMap:设置自定义映射关系
    >> - id:唯一标识，不能重复
    >>
    >> - type:设置映射关系中的实体类类型
    >>
    >>子标签:
        >> - id:设置主键映射关系
        >> - result:设置普通字段映射关系
        属性:
        >>> - property:设置映射关系中的属性名,必须是type属性设置的实体类对应的属性名
        >>> - column:设置映射关系中的字段名，sql查询出的的字段名。

`XXMapper.java`

```java
   /**
     * 查询所有的员工信息
     */
    List<Emp> getAllEmp();
```
`XXMapper.xml`
```xml
<!--    List<Emp> getAllEmp();-->
    <resultMap id="empResultMap" type="Emp">
        <id property="eid" column="eid"></id>
        <result property="empName" column="emp_name"></result>
        <result property="age" column="age"></result>
        <result property="sex" column="sex"></result>
        <result property="email" column="email"></result>
    </resultMap>

    <select id="getAllEmp" resultMap="empResultMap">
        select * from t_emp
    </select>
```

## 2. resultMap处理多对一映射处理
---
> 查询员工信息以及员工所对应的部门信息

> 多对一关系：在多对应的实体类中添加单一属性，在一对应的实体类中添加集合属性。

`XXMapper.java`
```java
    /**
     * 查询员工以及员工所对应的部门信息
     */
    Emp getEmpAndDep(@Param("eid") Integer eid);
```

### 方法一：级联属性赋值

`XXMapper.xml`
```xml
<!--处理多对一映射关系方式一：级联属性赋值-->
    <resultMap id="EmpAndDeptResultMapOne" type="Emp">
        <id property="eid" column="eid"></id>
        <result property="empName" column="emp_name"></result>
        <result property="age" column="age"></result>
        <result property="sex" column="sex"></result>
        <result property="email" column="email"></result>
        <!--级联属性赋值-->
        <result property="dept.did" column="did"></result>
        <result property="dept.deptName" column="dept_name"></result>
    </resultMap>

<!--    Emp getEmpAndDep(@Param("eid") Integer eid);-->
<!--    这里不能用resultType，因为emp中包含一个dept对象，并且查询出的数据需要和对象对应-->
    <select id="getEmpAndDep" resultMap="EmpAndDeptResultMapOne">
        select * from t_emp left join t_dept on t_emp.did = t_dept.did where t_emp.eid=#{eid}
    </select>
```

### 方式二：使用association标签：

`XXMapper.xml`
```xml
<!--处理多对一映射关系方式二：association-->
    <resultMap id="EmpAndDeptResultMapTwo" type="Emp">
        <id property="eid" column="eid"></id>
        <result property="empName" column="emp_name"></result>
        <result property="age" column="age"></result>
        <result property="sex" column="sex"></result>
        <result property="email" column="email"></result>
        <!--  专门处理多对一关系      -->
        <association property="dept" javaType="Dept">
            <id property="did" column="did"></id>
            <result property="deptName" column="dept_name"></result>
        </association>
    </resultMap>

    <select id="getEmpAndDep" resultMap="EmpAndDeptResultMapTwo">
        select * from t_emp left join t_dept on t_emp.did = t_dept.did where t_emp.eid=#{eid}
    </select>
```
> 通过association标签处理：处理多对一的映射关系
> - property:需要处理的多对一映射关系的属性名，
> - javaType:该属性的类型

### 方式三：使用分步查询
分布查询员工对应的部门信息：
- 第一步查询获取员工信息
- 第二步根据员工信息获取员工对应的部门信息

`XXMapper.java`
```java
    /**
     * 通过分布查询查询员工以及员工所对应的部门信息
     * 分布查询第一步：查询员工信息
     */
    Emp getEmpAndDeptByStepOne(@Param("eid") Integer eid);
```

`XXMapper.xml`
```xml
    <resultMap id="EmpAndDeptByStepResultMap" type="Emp">
        <id property="eid" column="eid"></id>
        <result property="empName" column="emp_name"></result>
        <result property="age" column="age"></result>
        <result property="sex" column="sex"></result>
        <result property="email" column="email"></result>
        <!--
            select:设置分布查询的sql的唯一标识(namespace.SQLId或者mapper接口全类名.方法名)
            column:设置分布查询的条件
            fetchType:在开启了全局的延迟加载之后，可以通过此属性手动控制延迟加载的效果，
            fetchType="lazy|eager":lazy表示延迟加载，eager表示立即加载
        -->
        <association property="dept"
                     select="com.cheng.mybatis.mapper.DeptMapper.getEmpAndDeptByStepTwo"
                     column="did" fetchType="eager">
        </association>
    </resultMap>


<!--    Emp getEmpAndDeptByStepOne(@Param("eid") Integer eid);-->
    <select id="getEmpAndDeptByStepOne" resultMap="EmpAndDeptByStepResultMap">
        select * from t_emp where eid = #{eid}
    </select>
```
- select:设置分布查询的sql的唯一标识(namespace.SQLId或者mapper接口全类名.方法名)
    - column:设置分布查询的条件
    - fetchType:在开启了全局的延迟加载之后，可以通过此属性手动控制延迟加载的效果，
    - fetchType="lazy|eager":lazy表示延迟加载，eager表示立即加载

根据association标签的select选项，调用分布查询的第二步，根据select选项的路径（命名空间.方法名）调用方法。

`XXMapper.java`
```java
    /**
     * 通过分布查询查询员工以及员工所对应的部门信息
     * 分布查询第二步：通过did查询员工对应的部门信息
     */
    Dept getEmpAndDeptByStepTwo(@Param("did") Integer did);
```
`XXMapper.xml`
```xml
<!--    Dept getEmpAndDeptByStepTwo(@Param("did") Integer did);-->

    <select id="getEmpAndDeptByStepTwo" resultType="Dept">
        select * from t_dept where did = #{did}
    </select>
```

---
## 3. resultMap处理一对多映射处理

> 获取部门以及部门所有员工的信息(一对多)

---

### 1. collection标签解决一对多的问题

> collection:处理一对多的映射关系
> - oftype:表示该属性所对应的集合中存储数据的类型

`XXMapper.java`
```java
    /**
     * 获取部门以及部门所有员工的信息
     */
    Dept getDeptAndEmp(@Param("did") Integer did);
```

`XXMapper.xml`
```xml
    <resultMap id="deptAndEmpResultMap" type="Dept">
        <id property="did" column="did"></id>
        <result property="deptName" column="dept_name"></result>
        <!--
            collection:处理一对多的映射关系
            oftype:表示该属性所对应的集合中存储数据的类型
        -->
        <collection property="emps" ofType="Emp">
            <id property="eid" column="eid"></id>
            <result property="empName" column="emp_name"></result>
            <result property="age" column="age"></result>
            <result property="sex" column="sex"></result>
            <result property="email" column="email"></result>
        </collection>
    </resultMap>
<!--    Dept getDeptAndEmp(@Param("did") Integer did);-->
    <select id="getDeptAndEmp" resultMap="deptAndEmpResultMap">
        select * from t_dept left join t_emp on t_dept.did = t_emp.did where t_dept.did=#{did}
    </select>
```

### 2. 分布查询

---
- 先查找部门信息
- 再根据部门信息查找员工信息

`XXMapper.java`
```java
    /**
     * 通过分布查询部门信息，以及根据部门信息获取员工信息
     * 分布查询第一步：根据did查询部门信息
     */
    Dept getDeptAndEmpByStepOne(@Param("did") Integer did);
```
`XXMapper.xml`
```xml
    <resultMap id="deptAndEmpByStepResultMap" type="Dept">
        <id property="did" column="did"></id>
        <result property="deptName" column="dept_name"></result>

        <collection property="emps"
                    select="com.cheng.mybatis.mapper.EmpMapper.getDeptAndEmpByStepTwo"
                    column="did"></collection>
    </resultMap>
<!--    Dept getDeptAndEmpByStepOne(@Param("did") Integer did);-->
    <select id="getDeptAndEmpByStepOne" resultMap="deptAndEmpByStepResultMap">
        select * from t_dept where did = #{did}
    </select>
```

分布查询第二步:

`XXMapper.java`
```java
    /**
     * 通过分布查询查询部门对应的员工信息
     * 分布查询第二部:根据did的值，查询对应的员工信息
     */
    List<Emp> getDeptAndEmpByStepTwo(@Param("did") Integer did);
```
`XXMapper.xml`
```xml
<!--    List<Emp> getDeptAndEmpByStepTwo(@Param("did") Integer did);-->
    <select id="getDeptAndEmpByStepTwo" resultType="Emp">
        select * from t_emp where did = #{did}
    </select>
```

> 分布查询优点：
>
> 可以实现延迟加载，但是必须在核心配置文件中设置全局配置信息：
>
> - lazyLoadingEnabled：延迟加载的全局开关。当开启时，所有关联对象都会延迟加载
>
> - aggressiveLazyLoading：当开启时，任何方法的调用都会加载该对象的所有属性。 否则，每个
属性会按需加载
> 
> 此时就可以实现按需加载，获取的数据是什么，就只会执行相应的sql。此时可通过association和
collection中的fetchType属性设置当前的分步查询是否使用延迟加载，fetchType="lazy(延迟加
载)|eager(立即加载)"

# 九、动态SQL
Mybatis框架的动态SQL技术是一种根据特定条件动态拼装SQL语句的功能，它存在的意义是为了解决
拼接SQL语句字符串时的痛点问题。

## 1. if

> if：根据标签中test属性所对应的表达式决定标签中的内容决定是否需要拼接到SQL中。

if标签可通过test属性的表达式进行判断，若表达式的结果为true，则标签中的内容会执行；反之标签中
的内容不会执行

> 应用场景：多条件查询时，拼接SQL语句

`XXMapper.java`

```java
    /**
     * 多条件查询
     */
    List<Emp> getEmpByCondition(Emp emp);
```

if会执行所有满足条件的语句，但是需要添加1=1恒等条件，否则有可能会导致SQL语句错误。例如where and。。。错误。
`XXMapper.xml`

```xml
    <select id="getEmpByConditionOne" resultType="Emp">
         select * from t_emp where 1=1
         <!--加1=1是为了防止第一个条件为空的时候，语法错误直接where and ...-->
        <if test="empName != null and empName != ''">
            emp_name = #{empName}
        </if>
        <if test="age != null and age != ''">
           and age = #{age}
        </if>
        <if test="sex != null and sex != ''">
          and  sex = #{sex}
        </if>
        <if test="email != null and email != ''">
          and  email = #{email}
        </if>
    </select>
```

## 2. where
---
where和if一般结合使用：
- 当where标签中有内容的时候，会自动生成where关键字，并且将内容前多余的and或or去掉
- 当where标签中没有内容的时候，没用任何作用，（不会自动添加where关键字）
- 注意：where标签不能将其中内容后边多余的and或or去掉。

`XXMapper.xml`
``` xml
<select id="getEmpByConditionTwo" resultType="Emp">
        select * from t_emp
        <where>
            <if test="empName != null and empName != ''">
                emp_name = #{empName}
            </if>
            <if test="age != null and age != ''">
                and age = #{age}
            </if>
            <if test="sex != null and sex != ''">
                and  sex = #{sex}
            </if>
            <if test="email != null and email != ''">
                and  email = #{email}
            </if>
        </where>

    </select>
```
## 3. trim

`XXMapper.xml`
```xml
<sql id="empColumns">eid,emp_name,age,sex,email</sql>
<!--    List<Emp> getEmpByCondition(Emp emp);-->
    <select id="getEmpByCondition" resultType="Emp">
        select <include refid="empColumns"></include> from t_emp
        <trim prefix="where" prefixOverrides="and|or">
            <if test="empName != null and empName != ''">
                emp_name = #{empName}
            </if>
            <if test="age != null and age != ''">
                and age = #{age}
            </if>
            <if test="sex != null and sex != ''">
                and  sex = #{sex}
            </if>
            <if test="email != null and email != ''">
                and  email = #{email}
            </if>
        </trim>
    </select>
```
> 其中sql标签用于设置sql片段：
> - <sql id="empColumns">eid,emp_name,age,sex,email</sql>
> - 引用sql片段
> <include refid="empColumns"></include>

trim用于去掉或添加标签中的内容

若标签中有内容时：

- prefix:将trim标签中内容前边添加指定内容

- prefixOverrides:将trim标签中内容前边去掉指定内容

- suffix:将trim标签中内容后边添加指定内容

- suffixOverrides:将trim标签中内容后边去掉指定内容

若标签中没有内容时，trim标签没有任何效果

## 4. choose、when、otherwise,相当于if...else if...else结构，只要有一个条件满足就不会执行其他条件了。

when至少要有一个，otherwise最多只能有一个，可以没有

`XXMapper.java`
```java
    /**
     * 测试choose、when、otherwise执行多条件查询
     */
    List<Emp> getEmpByChoose(Emp emp);
```

`XXMapper.xml`
```xml
<!--List<Emp> getEmpByChoose(Emp emp);-->
    <select id="getEmpByChoose" resultType="Emp">
        select * from t_emp
        <where>
            <choose>
                <when test="empName != null and empName != ''">
                    emp_name = #{empName}
                </when>
                <when test="age != null and age != ''">
                    age = #{age}
                </when>
                <when test="sex != null and sex != ''">
                    sex = #{sex}
                </when>
                <when test="email != null and email != ''">
                    email = #{email}
                </when>
                <otherwise>
                 <!--都不满足查询返回did=1的记录-->
                    eid = 1
                </otherwise>
            </choose>
        </where>
    </select>
```

## 5. foreach
> 通过数组实现批量删除

`XXMapper.java`
```java
    /**
     * 通过数组实现批量删除
     */
    int deleteMoreByArray(@Param("eids") Integer[] eids);
```

`XXMapper.xml`
```xml
    <delete id="deleteMoreByArray">
        delete from t_emp where eid in
        <foreach collection="eids" item="eid" separator="," open="(" close=")">
            #{eid}
        </foreach>
    </delete>
```
方法二：
`XXMapper.xml`
```xml
<!--    int deleteMoreByArray(@Param("eids") Integer[] eids);-->
    <delete id="deleteMoreByArray">
        delete from t_emp where
        <foreach collection="eids" item="eid" separator="or">
            eid=#{eid}
        </foreach>
    </delete>
```
> 相关属性：
> - collection:设置需要循环的数组或集合
> - item:表示数组或集合中的某个元素
> - separator:循环体之间的分隔符
> - open:foreach标签循环的所有内容的开始符
> - close:foreach标签循环的所有内容的结束符

# 十、MyBatis的缓存
## 1.MyBatis的一级缓存
---
一级缓存是SqlSession级别的，通过同一个SqlSession查询的数据会被缓存，下次查询相同的数据，就会从缓存中直接获取，不会从数据库重新访问

`XXMapper.java`
```java
    /**
     * 测试mybatis缓存范围
     */
    Emp getEmpById(@Param("eid") Integer eid);
```

`XXMapper.xml`
```xml
   <!-- Emp getEmpById(@Param("eid") Integer eid);-->
    <select id="getEmpById" resultType="Emp">
        select * from t_emp where eid = #{eid}
    </select>
```

使一级缓存失效的四种情况：
1) 不同的SqlSession对应不同的一级缓存

2) 同一个SqlSession但是查询条件不同

3) 同一个SqlSession两次查询期间执行了任何一次增删改操作

4) 同一个SqlSession两次查询期间手动清空了缓存

`XXTest.java`
```java
     @Test
    public void testCache(){
        SqlSession sqlsession = SqlSessionUtils.getSqlsession();
        CacheMapper mapper = sqlsession.getMapper(CacheMapper.class);
        Emp empById = mapper.getEmpById(1);
        //这意味着mybatis一级缓存的级别是sqlsession级别的。
        CacheMapper mapper2 = sqlsession.getMapper(CacheMapper.class);
        Emp empById2 = mapper2.getEmpById(1);
        System.out.println(empById);
        System.out.println(empById2);
        System.out.println("++++++++++++++++++++");

        //同一个sqlsession期间进行了增删改操作就会清空缓存
        mapper.insertEmp(new Emp(null,"baga",23,"男","456@qq.com"));
        System.out.println(empById);
        System.out.println("++++++++++++++++++");

        SqlSession sqlsession1 = SqlSessionUtils.getSqlsession();
        CacheMapper mapper1 = sqlsession1.getMapper(CacheMapper.class);
        Emp empById1 = mapper1.getEmpById(1);
        System.out.println(empById1);
        //手动清空缓存
        sqlsession1.clearCache();
    }
```
## 2.MyBatis的二级缓存
---
二级缓存是SqlSessionFactory级别，通过同一个SqlSessionFactory创建的SqlSession查询的结果会被缓存；此后若再次执行相同的查询语句，结果就会从缓存中获取

> 二级缓存开启的条件：
>
> - 在核心配置文件中，设置全局配置属性cacheEnabled="true"，默认为true，不需要设置
> - 在映射文件中设置标签<cache />
> - 二级缓存必须在SqlSession关闭或提交之后有效
> - 查询的数据所转换的实体类类型必须实现序列化的接口

> 使二级缓存失效的情况：
> - 两次查询之间执行了任意的增删改，会使一级和二级缓存同时失效

`XXMapper.java`
```java
void insertEmp(Emp emp);
```
`XXMapper.xml`
```xml
<!--    void insertEmp(Emp emp);-->
    <insert id="insertEmp">
        insert into t_emp values (null,#{empName},#{age},#{sex},#{email},null)
    </insert>
```

`XXTest.java`
```java 
@Test
    public void testTwoCache()  {
        InputStream is = null;
        try {
            is = Resources.getResourceAsStream("mybatis-config.xml");
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
            SqlSession sqlSession1 = sqlSessionFactory.openSession(true);
            CacheMapper mapper1 = sqlSession1.getMapper(CacheMapper.class);
            System.out.println(mapper1.getEmpById(1));
            //sqlsession关闭或提交才会开启二级缓存。
            sqlSession1.close();

            SqlSession sqlSession2 = sqlSessionFactory.openSession(true);
            CacheMapper mapper2 = sqlSession2.getMapper(CacheMapper.class);
            System.out.println(mapper2.getEmpById(1));
            sqlSession2.close();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

## 3.二级缓存的相关配置
---
在mapper配置文件中添加的cache标签可以设置一些属性：
- eviction属性：缓存回收策略
  
    LRU（Least Recently Used） – 最近最少使用的：移除最长时间不被使用的对象。
    
    FIFO（First in First out） – 先进先出：按对象进入缓存的顺序来移除它们。
    
    SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象。
    
    WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
    
    默认的是 LRU。
    
- flushInterval属性：刷新间隔，单位毫秒

    默认情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句时刷新
    
- size属性：引用数目，正整数

    代表缓存最多可以存储多少个对象，太大容易导致内存溢出
    
- readOnly属性：只读，true/false

    true：只读缓存；会给所有调用者返回缓存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势。
    
    false：读写缓存；会返回缓存对象的拷贝（通过序列化）。这会慢一些，但是安全，因此默认是false。

## 4.MyBatis缓存查询的顺序
---
- 先查询二级缓存，因为二级缓存中可能会有其他程序已经查出来的数据，可以拿来直接使用。
- 如果二级缓存没有命中，再查询一级缓存
- 如果一级缓存也没有命中，则查询数据库
- SqlSession关闭之后，一级缓存中的数据会写入二级缓存

## 5.整合第三方的缓存EHCache
---
### 添加依赖
`pom.xml`
```xml
<!-- Mybatis EHCache整合包 -->
        <dependency>
            <groupId>org.mybatis.caches</groupId>
            <artifactId>mybatis-ehcache</artifactId>
            <version>1.2.1</version>
        </dependency>
        <!-- slf4j日志门面的一个具体实现 -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>

```
各个jar包的作用:

|jar包名称|作用|
|---|---|
|mybatis-ehcache |Mybatis和EHCache的整合包|
|ehcache |EHCache核心包|
|slf4j-api|SLF4J日志门面包|
|logback-classic|支持SLF4J门面接口的一个具体实现|

### 创建EHCache的配置文件ehcache.xml
`ehache.xml`
```xml
<?xml version="1.0" encoding="utf-8" ?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
    <!-- 磁盘保存路径 -->
    <diskStore path="D:\atguigu\ehcache"/>
    <defaultCache
            maxElementsInMemory="1000"
            maxElementsOnDisk="10000000"
            eternal="false"
            overflowToDisk="true"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU">
    </defaultCache>
</ehcache>
```
### 设置二级缓存的类型

```xml
    <cache type="org.mybatis.caches.ehcache.EhcacheCache"></cache>
```
### 加入logback日志

存在SLF4J时，作为简易日志的log4j将失效，此时我们需要借助SLF4J的具体实现logback来打印日志。

创建logback的配置文件logback.xml
`logback.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="true">
    <!-- 指定日志输出的位置 -->
    <appender name="STDOUT"
              class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
        <!-- 日志输出的格式 -->
        <!-- 按照顺序分别是：时间、日志级别、线程名称、打印日志的类、日志主体内容、换行 -->
            <pattern>[%d{HH:mm:ss.SSS}] [%-5level] [%thread] [%logger] [%msg]%n</pattern>
        </encoder>
    </appender>
    <!-- 设置全局日志级别。日志级别按顺序分别是：DEBUG、INFO、WARN、ERROR -->
    <!-- 指定任何一个日志级别都只打印当前级别和后面级别的日志。 -->
    <root level="DEBUG">
        <!-- 指定打印日志的appender，这里通过“STDOUT”引用了前面配置的appender -->
        <appender-ref ref="STDOUT" />
    </root>
    <!-- 根据特殊需求指定局部日志级别 -->
    <logger name="com.atguigu.crowd.mapper" level="DEBUG"/>
</configuration>
```
EHCache配置文件说明:

|属性名|是否必须|作用|
|:---:|:---:|:---:|
|maxElementsInMemory|是|在内存中缓存的element的最大数目|
|maxElementsOnDisk |是|在磁盘上缓存的element的最大数目，若是0表示无穷大|
|eternal|是|设定缓存的elements是否永远不过期。 如果为true，则缓存的数据始终有效， <br>如果为false那么还要根据timeToIdleSeconds、<br>timeToLiveSeconds判断
|overflowToDisk |是|设定当内存缓存溢出的时候是否将过期的element缓存到磁盘上|
|timeToIdleSeconds|否|当缓存在EhCache中的数据前后两次访问的时间超<br>过timeToIdleSeconds的属性取值时， 这些数据便会删除，默认值是0,也就是可闲置时间无穷大
|timeToLiveSeconds|否|缓存element的有效生命期，默认是0.,也就是element存活时间无穷大|
|diskSpoolBufferSizeMB|否|DiskStore(磁盘缓存)的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区|
|diskPersistent|否|在VM重启的时候是否启用磁盘保存EhCache中的数据，默认是false。|
|diskExpiryThreadIntervalSeconds|否|磁盘缓存的清理线程运行间隔，默认是120秒。每个120s，<br>相应的线程会进行一次EhCache中数据的清理工作
|memoryStoreEvictionPolicy|否|当内存缓存达到最大，有新的element加入的时候， <br>移除缓存中element的策略。 默认是LRU（最近最少使用），可选的有LFU（最不常使用）和FIFO（先进先出）

# 十一、MyBatis的逆向工程

- 正向工程：先创建Java实体类，由框架负责根据实体类生成数据库表。Hibernate是支持正向工程的。

- 逆向工程：先创建数据库表，由框架负责根据数据库表，反向生成如下资源：
    - Java实体类
    
    - Mapper接口
    
    - Mapper映射文件

## 创建逆向工程的步骤

### 添加依赖和插件

`pom.xml`
```xml
<dependencies>
        <!-- Mybatis核心 -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.7</version>
        </dependency>
        <!-- junit测试 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.30</version>
        </dependency>

        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>

        <!-- Mybatis EHCache整合包 -->
        <dependency>
            <groupId>org.mybatis.caches</groupId>
            <artifactId>mybatis-ehcache</artifactId>
            <version>1.2.1</version>
        </dependency>
        <!-- slf4j日志门面的一个具体实现 -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>

    </dependencies>


    <!-- 控制Maven在构建过程中相关配置 -->
    <build>
        <!-- 构建过程中用到的插件 -->
        <plugins>
            <!-- 具体插件，逆向工程的操作是以构建过程中插件形式出现的 -->
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.0</version>
                <!-- 插件的依赖 -->
                <dependencies>
                    <!-- 逆向工程的核心依赖 -->
                    <dependency>
                        <groupId>org.mybatis.generator</groupId>
                        <artifactId>mybatis-generator-core</artifactId>
                        <version>1.3.2</version>
                    </dependency>
                    <!-- 数据库连接池 -->
                    <dependency>
                        <groupId>com.mchange</groupId>
                        <artifactId>c3p0</artifactId>
                        <version>0.9.2</version>
                    </dependency>
                    <!-- MySQL驱动 -->
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>8.0.30</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
```

### 创建MyBatis的核心配置文件

### 创建逆向工程的配置文件

> 文件名必须是：generatorConfig.xml

`generatorConfig.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!--
        targetRuntime: 执行生成的逆向工程的版本
        MyBatis3Simple: 生成基本的CRUD（清新简洁版）
        MyBatis3: 生成带条件的CRUD（奢华尊享版）
    -->
    <context id="DB2Tables" targetRuntime="MyBatis3Simple">
        <!-- 数据库的连接信息 -->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/mybatis"
                        userId="root"
                        password="123456">
        </jdbcConnection>
        <!-- javaBean的生成策略-->
        <javaModelGenerator targetPackage="com.cheng.mybatis.pojo"
                            targetProject=".\src\main\java">
        <!--   如果为true，每个.都表示一层目录         -->
            <property name="enableSubPackages" value="true" />
            <!-- 去掉数据库表字段名前后的空格-->
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!-- SQL映射文件的生成策略 -->
        <sqlMapGenerator targetPackage="com.cheng.mybatis.mapper"
                         targetProject=".\src\main\resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>
        <!-- Mapper接口的生成策略 -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.cheng.mybatis.mapper" targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>
        <!-- 逆向分析的表 -->
        <!-- tableName设置为*号，可以对应所有表，此时不写domainObjectName -->
        <!-- domainObjectName属性指定生成出来的实体类的类名 -->
        <table tableName="t_emp" domainObjectName="Emp"/>
        <table tableName="t_dept" domainObjectName="Dept"/>
    </context>
</generatorConfiguration>
```

### 执行MBG插件的generate目标

![image.png](attachment:image.png)
![image.png](..\jupyNotebookFile\photo\mybatis-11-1.png)

### 生成实体类的有参和无参构造函数

### 重写toString方法

## QBC查询
QBC（Query by Criteria）:根据条件查询
> 简易版本生成的只有五个方法：增删改查和根据主键查
>
> Mybatis3版本生成的包含有条件查询。

![image.png](..\jupyNotebookFile\photo\mybatis-11-2.png)

![image-2.png](attachment:image-2.png)


`EmpMapper.java`

该方法表示普通添加方法，当添加的record包含null时，会将数据库中对应的字段设置为Null。

```java
    int insert(Emp record);
```
该方法表示有选择性的添加，只会向数据库中添加不为null的数据。
```java
    int insertSelective(Emp record);
```
```java
    List<Emp> selectByExample(EmpExample example);
```
带有Example后缀的select表示带条件查询，可以以任意字段的所有需求为条件进行查询（and,or...）

查询所有数据：
> `List<Emp> emps = mapper.selectByExample(null);` 当没有输入任何条件参数时就是查询所有数据。

```java
    @Test
    public void testMBG(){
        InputStream is = null;
        try {
            is = Resources.getResourceAsStream("mybatis-config.xml");
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
            SqlSession sqlSession = sqlSessionFactory.openSession(true);
            EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
            //查询所有数据
            List<Emp> emps = mapper.selectByExample(null);
            emps.forEach(emp -> System.out.println(emp));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

按照条件查询：
> 调用example.createCriteria().andXXXAAA()....  其中XX为根据查询的属性条件，AA为条件方法


```java
    //根据条件查询
    EmpExample example = new EmpExample();
    example.createCriteria().andEmpNameEqualTo("张三").andAgeGreaterThanOrEqualTo(20);
    List<Emp> emps1 = mapper.selectByExample(example);
    emps1.forEach(emp -> System.out.println(emp));
```
在此基础上添加or条件，只需要在example后边添加.or()方法即可。
```java
    example.or().andEmailIsNotNull();
```
解析得到的查询语句如下：
```java
    select eid, emp_name, age, sex, email, did from t_emp WHERE ( emp_name = ? and age >= ? ) or( email is not null )
```
当使用MBG修改数据时:

对于null值，也会将sql中对应的字段设置为Null

```java
    mapper.updateByPrimaryKey(new Emp(4,"徐4",20,null,null,2));
```
> 该语句会将eid为4的记录从Emp{eid=4, empName='李4', age=18, sex='男', email='stu@qq.com', did=2}更改为Emp{eid=4, empName='徐4', age=20, sex='null', email='null', did=2}

而使用updateByPrimaryKeySelective时，将不会使用null值更改数据库字段的值

```java
    mapper.updateByPrimaryKeySelective(new Emp(4,"徐4",20,null,null,2));
```
> 该语句会将记录修改为Emp{eid=4, empName='徐4', age=20, sex='男', email='stu@qq.com', did=2}

# 十二、分页插件

## 1、分页插件使用步骤

### 添加依赖
```xml
    <!--   分页插件依赖-->
        <!-- https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper -->
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper</artifactId>
            <version>5.2.0</version>
        </dependency>
```
### 配置分页插件
在MyBatis的核心配置文件中配置插件
`mybatis-config.xml`
```xml
     <plugins>
        <!--设置分页插件-->
        <plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
    </plugins>
```

## 2.分页插件的使用

在查询功能之前使用PageHelper.startPage(int pageNum, int pageSize)开启分页功能,也就是调用mapper.selectXX方法之前。
> - limit: index,pageSize
>
> - index:当前页的起始索引
>
> - pageSize:每页显示的记录数
>
> - index=(pageNum-1)*pageSize

## 3. 获取分页相关数据

可以直接通过`Page<Object> page = PageHelper.startPage(1, 3);`方法返回的page对象进行查看分页信息

> Page{count=true, pageNum=1, pageSize=3, startRow=0, endRow=3, total=12, pages=4, reasonable=false, pageSizeZero=false}
[Emp{eid=1, empName='张三', age=23, sex='男', email='123@123.com', did=1}, Emp{eid=2, empName='李四', age=12, sex='女', email='hello@qq.com', did=2}, Emp{eid=3, empName='王五', age=30, sex='男', email='wagnwu@163.com', did=3}]


还可以在查询功能执行之后，获取PageInfo对象，`PageInfo<Emp> pageInfo = new PageInfo<>(emps, 5);`，利用pageinfo对象查看分页信息，比page对象更全。
> PageInfo{pageNum=1, pageSize=3, size=3, startRow=1, endRow=3, total=12, pages=4, 

> 这里就是上边的Page对象的内容：list=Page{count=true, pageNum=1, pageSize=3, startRow=0, endRow=3, total=12, pages=4, reasonable=false, pageSizeZero=false}[Emp{eid=1, empName='张三', age=23, sex='男', email='123@123.com', did=1}, Emp{eid=2, empName='李四', age=12, sex='女', email='hello@qq.com', did=2}, Emp{eid=3, empName='王五', age=30, sex='男', email='wagnwu@163.com', did=3}], 

> prePage=0, nextPage=2, isFirstPage=true, isLastPage=false, hasPreviousPage=false, hasNextPage=true, navigatePages=5, navigateFirstPage=1, navigateLastPage=4, navigatepageNums=[1, 2, 3, 4]}

> 常用数据：
>
> pageNum：当前页的页码
>
>
> pageSize：每页显示的条数
>
> size：当前页显示的真实条数
>
> total：总记录数
>
> pages：总页数
>
> prePage：上一页的页码
>
> nextPage：下一页的页码
>
> isFirstPage/isLastPage：是否为第一页/最后一页
>
> hasPreviousPage/hasNextPage：是否存在上一页/下一页
>
> navigatePages：导航分页的页码数
>
> navigatepageNums：导航分页的页码，[1,2,3,4,5]




```python

```
