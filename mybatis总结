

# mybatis总结

## 入门

```
1.导入依赖
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>

2.每个基于mybatis的应用都是以 SqlSessionFactory 的实例为核心的
SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。
而 SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先定制的 Configuration 的实例构建出 SqlSessionFactory 的实例。
 1.首先基于XML配置文件配置mybatis的核心设置用以构建SqlSessionFactory
 该配置文件包含获取数据库连接实例的数据源（DataSource）和决定事务作用域和控制方式的事务管理器（TransactionManager）
 <?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>


3.编写实体类USER
4.编写dao层接口USERDAO
4.编写USERDAO接口的映射文件用以实现USERDAO中的方法
注意USERDAO和USERDAO.xml文件目录位置要保持相对一致
5.在mybatis全局配置文件中注册sql映射器

补充也可以通过注解的方法配置dao层接口中的映射器
```

## 映射配置文件sqlMapper.xml

```
<select id="findById" resultType="com.itheima.domain.User" parameterType="int">
select * from user where id = #{uid}
</select>
resultType ：用于指定结果集的类型。
parameterType:用于指定传入参数的类型
sql 语句中使用#{}字符：它代表占位符，相当于原来 jdbc 部分所学的?，都是用于执行语句时替换实际的数据。具体的数据是由#{}里面的内容决定的。

<insert id="saveUser" parameterType="com.itheima.domain.User">
insert into user(username,birthday,sex,address) 
values(#{username},#{birthday},#{sex},#{address})
</insert>
#{user.username}它会先去找 user 对象，然后在 user 对象中找到 username 属性，并调用getUsername()方法把值取出来。但是我们在 parameterType 属性上指定了实体类名称，所以可以省略 user.而直接写 username

```

## 增删改操作

```
在进行增删改操作的时候必须进行事务提交数据库才会发生变化
提交代码如下
session.commit();来实现事务提交。加入事务提交后的代码如下：
释放资源
session.close();
in.close();

还可以设置自动提交方式
session = factory.openSession(true);
此时事务就设置为自动提交了
```

## 自增长主键

```java
添加用户后，同时还要返回当前新增用户的id值，因为id是自增长主键，由数据库自动增长实现，我们需要获取自增长主键
<insert id="saveUser" parameterType="USER">
<!-- 配置保存时获取插入的 id --> 
<selectKey keyColumn="id" keyProperty="id" useGeneratedKeys=true resultType="int">
select last_insert_id();
</selectKey>
insert into user(username,birthday,sex,address) 
values(#{username},#{birthday},#{sex},#{address})
</insert>

使用自增长主键并且将其赋值为"id"
keyProperty="id" useGeneratedKeys=true
```

## 模糊查询

```java
第一种配置方法--在参数中加%
<select id="findByName" resultType="com.itheima.domain.User" parameterType="String">
select * from user where username like #{username}
</select>
在配置文件中没有加上%，所以在传入字符串参数的时候在参数值中加%
List<User> users = userDao.findByName("%王%");

第二种配置方法--在配置文件中加%,通过拼接字符串实现${}
<select id="findByName" parameterType="string" resultType="com.itheima.domain.User">
select * from user where username like '%${value}%'
</select>
List<User> users = userDao.findByName("王");

注意两种方法不同点
#{}表示一个占位符
可以实现 preparedStatement 向占位符中设置值，自动进行 java 类型和 jdbc 类型转换，#{}可以有效防止 sql 注入。 #{}可以接收简单类型值或 pojo 属性值。 如果 parameterType 传输单个简单类型值，#{}括号中可以是 value 或其它名称。
${}表示拼接sql字符串
```

## 动态SQL语句

我们根据实体类的不同取值，使用不同的 SQL 语句来进行查询。比如在 id 如果不为空时可以根据 id 查询， 如果 username 不同空时还要加入用户名作为条件。这种情况在我们的多条件组合查询中经常会碰到。 

### if标签

```java
<if>标签
<select id="findByUser" resultType="user" parameterType="user">
select * from user where 1=1
<if test="username!=null and username !=''">	// 条件
and username like #{username}					//满足条件动态添加该语句
</if> 
<if test="address != null">
and address like #{address}
</if>
</select>
test 属性中写的是对象的属性名，如果是包装类的对象要使用 OGNL 表达式的写法。
注意where 1=1 的作用！！ 保证where在都不满足的时候sql语句能正常执行
```

### where标签

```
为了简化where 1=1的拼装，可以采用<where>标签来简化开发
<select id="findByUser" resultType="user" parameterType="user"> 
<include refid="defaultSql"></include> 
<where> 
<if test="username!=null and username != '' ">
and username like #{username}
</if> 
<if test="address != null">
and address like #{address}
</if>
</where>
</select>
```

### foreach标签

```
传入多个 id 查询用户信息，用下边两个 sql 实现：
SELECT * FROM USERS WHERE username LIKE '%张%' AND (id =10 OR id =89 OR id=16)
SELECT * FROM USERS WHERE username LIKE '%张%' AND id IN (10,89,16)
我们在进行范围查询的时候，需要将一个集合的值动态添加id值进来
即需要实现IN (10,89,16)或者 (id =10 OR id =89 OR id=16)这种场景

<!-- 查询所有用户在 id 的集合之中 -->
<select id="findInIds" resultType="user" parameterType="queryvo">
<!-- select * from user where id in (1,2,3,4,5); --> <include refid="defaultSql">
</include> <where>
<if test="ids != null and ids.size() > 0">
<foreach collection="ids" open="id in ( " close=")" item="uid"
separator=",">
#{uid}
</foreach>
</if>
</where>
</select> 相当于sql语句select 字段 from user where id in (?)
<foreach>标签用于遍历集合，它的属性：
collection:代表要遍历的集合元素，注意编写时不要写#{}
open:代表语句的开始部分
close:代表结束部分
item:代表遍历集合的每个元素，生成的变量名
sperator:代表分隔符
```

## choose-switch case

```
MyBatis 提供了 choose 元素，按顺序判断 when 中的条件是否成立，
如果有一个成立，则 choose 结束。
当 choose 中所有 when的条件都不满足时，则执行 otherwise 中的 sql。
类似于 Java 的 switch 语句，choose 为 switch，when 为 case，otherwise 则为 default。
<select id="getStudentListChoose" parameterType="Student" resultMap="BaseResultMap">
    SELECT * from STUDENT WHERE 1=1
    <where>
        <choose>
            <when test="Name!=null and student!='' ">
                AND name LIKE CONCAT(CONCAT('%', #{student}),'%')
            </when>
            <when test="hobby!= null and hobby!= '' ">
                AND hobby = #{hobby}
            </when>
            <otherwise>
                AND AGE = 15
            </otherwise>
        </choose>
    </where>
</select>
```

## set

```
set用于update更新字段中
哪个字段中有值才去更新，如果某项为 null 则不进行更新，而是保持数据库原值。
if -set 做搭配
<!--动态Sql: set 标签-->
    <update id="updateSet" parameterType="com.lks.domain.User">
        update users
        <set>
            <if test="name != null and name != ''">
                name = #{name},
            </if>
            <if test="county != null and county != ''">
                county = #{county},
            </if>
        </set>
        where id = #{id}
    </update>
```

## trim

```
trim 是一个格式化标签，可以完成< set > 或者是 < where > 标记的功能。主要有4个参数： prefix：前缀
prefixOverrides：去掉第一个and或者是or
suffix：后缀
suffixOverrides：去掉最后一个逗号，也可以是其他的标记
select * from user 
　　<trim prefix="WHERE" prefixoverride="AND |OR">
　　　　<if test="name != null and name.length()>0"> AND name=#{name}</if>
　　　　<if test="gender != null and gender.length()>0"> AND gender=#{gender}</if>
　　</trim>
完成where的功能，如果name，gender都不为null
select * from user where（and-删除）  name = #{name} and gender = #{gender}
prefixoverride用于删除第一个and，

update user
　　<trim prefix="set" suffixoverride="," suffix=" where id = #{id} ">
　　　　<if test="name != null and name.length()>0"> name=#{name} , </if>
　　　　<if test="gender != null and gender.length()>0"> gender=#{gender} ,  </if>
　　</trim>
suffixoverride=","用与删除最后一个多余的,
```

## resultMap

```
作用
建立 SQL 查询结果字段与实体属性的映射关系信息
查询的结果集转换为 java 对象，方便进一步操作。
将结果集中的列与 java 对象中的属性对应起来并将值填充进去
<!-- resultMap最终还是要将结果映射到pojo上，type就是指定映射到哪一个pojo -->
    <!-- id：设置ResultMap的id -->
    <resultMap type="order" id="orderResultMap">
        <!-- 定义主键 ,非常重要。如果是多个字段,则定义多个id -->
        <!-- property：主键在pojo中的属性名 -->
        <!-- column：主键在数据库中的列名 -->
        <id property="id" column="id" />

        <!-- 定义普通属性result -->
        <result property="userId" column="user_id" />
        <result property="number" column="number" />
        <result property="createtime" column="createtime" />
        <result property="note" column="note" />
    </resultMap>
 
在做查询的时候就调用resultMap="orderResultMap" 即可
<select id="selectByPrimaryKey" resultMap="orderResultMap" parameterType="Object">
    select id,userId,number,createtime,note from order where id=#{id}
</select>
```

## 联合查询

1.使用级联属性封装查询结果

2.使用association标签表示联合了一个对象

## 抽取重复的语句代码片段

```
<!-- 抽取重复的语句代码片段 --> 
<sql id="defaultSql">
select * from user
</sql>
用include调用sql
<select id="findAll" resultType="user"> 
<include refid="defaultSql"></include>
</select>
```

## 多表查询

```
以最简单的用户和账户模型来分析mybatis多表关系，用户为User表，账户为Account表
查询所有账户信息，关联查询账户所对应的用户信息
SELECT 
account.*,user.username,user.address
FROM account,user WHERE account.uid = user.id
为了能够封装上面 SQL 语句的查询结果，定义 AccountCustomer 类中要包含账户信息同时还要包含用户信息，所以我们要在定义 AccountUser 类时可以继承 User 类。

SQL映射器
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper 
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd"> <mapper namespace="com.itheima.dao.IAccountDao">
<!-- 配置查询所有操作-->
<select id="findAll" resultType="accountuser">
select a.*,u.username,u.address from account a,user u where a.uid =u.id;
</select> 
</mapper>
注意：因为上面查询的结果中包含了账户信息同时还包含了用户信息，所以我们的返回值类型 returnType
的值设置为 AccountUser 类型，这样就可以接收账户信息和用户信息了。
```

### 一对多查询

```
查询所有用户信息以及用户关联的账户信息
SELECT
u.*, acc.id id,
acc.uid,
acc.money
FROM
user u
LEFT JOIN account acc ON u.id = acc.uid
将查询的信息，写成实体类，然后编写实体类对应的持久层DAO接口
实现dao接口中的方法从而实现一对多查询
```

## 延迟加载策略

```
延迟加载：在需要用到数据时才进行加载，不需要用到数据时候就不加载数据，延迟加载也称为懒加载。
1.好处：先从单表查询，需要时再从关联表去做关联查询，大大提高数据库性能，因为查询单表比关联查询多张表速度快。
2.坏处：只有用到的时候才会进行数据库查询。
3.使用场景：查询账户(Account)信息并且关联查询用户(User)信息。如果先查询账户(Account)信息即可满足要求，当我们需要查询用户(User)信息时再查询用户(User)信息。把对用户(User)信息的按需去查询就是延迟加载
```

### assocation实现延迟加载

```
1.在全局配置文件中开启懒加载
<setting name="lazyLoadingEnabled" value="true"/> 
<setting name="aggressiveLazyLoading" value="false"/>

账户的持久层映射文件
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.dao.IAccountDao">
<!-- 建立对应关系 -->
<resultMap type="account" id="accountMap">
<id column="aid" property="id"/>
<result column="uid" property="uid"/>
<result column="money" property="money"/>
<!-- 它是用于指定从表方的引用实体属性的 -->
<association property="user" javaType="user" select="com.itheima.dao.IUserDao.findById" 
column="uid">
</association>
</resultMap>
<select id="findAll" resultMap="accountMap"> select * from account
</select>
</mapper>

//配置查询所有账户
<select id="findAll" resultMap="accountMap"> 
select * from account 
</select>


用户的持久层映射文件
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd"> 
<mapper namespace="com.itheima.dao.IUserDao"> 
<!-- 根据id查询 --> 
<select id="findById" resultType="user" parameterType="int" > 
select * from user where id = #{uid} 
</select> 
</mapper>

如果只获取账户信息，则不会查询用户信息，如果需要用户想信息了，则会查询用户信息做到按需查询

```

## 缓存

使用缓存， 我们可以避免频繁的与数据库进行交互， 尤其是在查询越多、缓存命中率越高的情况下， 使用缓存对性能的提高更明显。

mybatis 也提供了对缓存的支持， 分为一级缓存和二级缓存。 但是在默认的情况下， 只开启一级缓存（一级缓存是对同一个 SqlSession 而言的）。

### 一级缓存

```
一级缓存是对sqlSession级别的缓存，只要SqlSession没有flush或close，它就存在
同一个 SqlSession 对象， 在参数和 SQL 完全一样的情况先， 只执行一次 SQL 语句（如果缓存没有过期）
同一个sqlSession对象
在参数和SQL语句完全相同的情况下
第一次查询发送了 SQL 语句， 后返回了结果；
第二次查询没有发送 SQL 语句， 直接从内存中获取了结果。
而且两次结果输入一致， 同时断言两个对象相同也通过。
不同的sqlSesson对象
分别使用sqlsession1和sqlSession2进行了相同的查询
从日志中可以看到两次查询都分别从数据库中取出了数据。虽然结果相同，但两个是不同的对象。

如果对缓存进行清空即flushCache=“true”,在sqlmapper中进行配置
<select id="selectByPrimaryKey" flushCache="true" parameterType="java.lang.Integer" resultMap="BaseResultMap">
    select
    <include refid="Base_Column_List" />
    from student
    where student_id=#{id, jdbcType=INTEGER}
</select>

结果： 第一次， 第二次都发送了 SQL 语句， 同时，断言两个对象不同

一级缓存总结
1.同一个 SqlSession 中, Mybatis 会把执行的方法和参数通过算法生成缓存的键值，将键值和结果存放在一个 Map 中， 如果后续的键值一样， 则直接从 Map 中获取数据；
2.不同的 SqlSession 之间的缓存是相互隔离的；
3.用一个 SqlSession， 可以通过配置使得在查询前清空缓存；flushCache=“true”
4.任何的 UPDATE, INSERT, DELETE 语句都会清空缓存。
如果sqlSession去执行commit操作（执行插入、更新、删除），会清空SqlSession中的一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免脏读。


```

## 二级缓存

```
二级缓存是mapper映射级别的缓存，多个SqlSession去操作同一个Mapper映射的sql语句，多个不同SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。
注意当我们在使用二级缓存时，所缓存的类一定要实现java.io.Serializable接口，这种就可以使用序列化方式来保存对象。
二级缓存有全局开关和分开关
全局开关-针对所有映射器 默认为true 
第一步  <setting name="cacheEnabled" value="true"/>

<settings>
  <!--全局地开启或关闭配置文件中的所有映射器已经配置的任何缓存。 -->
  <setting name="cacheEnabled" value="true"/>
</settings>


第二步 <cache></cache> 
分开关--针对某一个映射器开启或者关闭二级缓存，默认是关闭
<cache></cache> 在sqlmapper.xml中添加

<mapper namespace="com.itheima.dao.IUserDao"> 
<!-- 开启二级缓存的支持 --> 
<cache></cache> 
</mapper>

第三步 useCache="true"
<select id="findById" resultType="user" parameterType="int" useCache="true"> 
select * from user where id = #{uid}
</select>

```

## mybatis整合第三方缓存

```
由于mybatis做缓存并不专业用的是map，但是它给了我们一个接口Cache，我们通过实现这个接口可以自定义缓存，本例子用的是Ehcache缓存来强化mybatis的二级缓存
具体配置如下
https://blog.csdn.net/jinhaijing/article/details/84255107

```



## ssm框架整合

### 环境准备

1. 创建数据库和表结构
2. 创建maven工程
3. 导入坐标并建立依赖
4. 编写实体类
5. 编写业务层接口
6. 编写持久层接口

### 整合步骤

1. 保证spring框架在web工程中独立运行
2. 保证springmvc在web工程中独立运行
3. 整合spring到springmvc
4. 保证mybatis在web工程中独立运行
5. 整合spring和mybatis
6. 测试ssm整合结果
