# MyBatis自学笔记




# MyBatis自学笔记

> 大连交通大学 信息学院 刘嘉宁 2021-08-22
>
> 笔记摘自链接：https://www.bilibili.com/video/BV1NE411Q7Nx



## 什么是MyBatis：

- 一个优秀的持久层框架
- 支持定制化SQL，存储过程以及高级映射
- 避免了几乎所有的JDBC代码和手动设置参数以及获取结果集
- MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录



## MyBatis的特点：

- 简单易学。
- 灵活。 
- 解除sql与程序代码的耦合。
- 提供映射标签，支持对象与数据库的orm字段`"对象-关系映射"（Object/Relational Mapping）`关系映射。
- 提供对象关系映射标签，支持对象关系组建维护。
- 提供xml标签，支持编写动态sql。
- **缺点**：sql语句依赖于数据库,不能随意的更换数据库,移植性差



## IDEA中使用maven创建MyBatis项目：

> 父工程（mybatisProject）中创建配置好的依赖会在子模块（mybatis-01）中被继承
>
> ![image-20210822153755815](D:\Document\MyBatis\MyBatis自学笔记.assets\image-20210822153755815.png)

1. ##### 创建maven在pom.xml中添加mysql驱动的依赖，mybatis的依赖。

   ```xml
   <!-- MyBatis的依赖 -->
   <dependency>
     <groupId>org.mybatis</groupId>
     <artifactId>mybatis</artifactId>
     <version>3.4.6</version>
   </dependency>
   ```

2. ##### 在模块src\main\resources目录下创建mybatis-config.xml配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   
   <!--核心配置文件-->
   <configuration>
       <environments default="development">
           <environment id="development">
               <transactionManager type="JDBC"/>
               <dataSource type="POOLED">
                   <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                   <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=utf-8"/>
                   <property name="username" value="root"/>
                   <property name="password" value="129807"/>
               </dataSource>
           </environment>
       </environments>
       <!--注册mapper, 每一个mapper.xml都需要在MyBatis核心配置中注册-->
       <mappers>
           <mapper resource="com/kuang/dao/UserMapper.xml"/>
       </mappers>
   </configuration>
   ```

3. ##### 在utils包下封装MybatisUtils工具类读取这个配置文件

   ```java
   /**
    * sqlSessionFactory --> sqlSession 工具类
    * @author 刘嘉宁
    */
   public class MybatisUtils {
       private static SqlSessionFactory sqlSessionFactory = null;
   
       static{
           try {
               String resource = "mybatis-config.xml";
               InputStream inputStream = Resources.getResourceAsStream(resource);
               sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
           } catch (Exception e) {
               e.printStackTrace();
           }
       }
   
       /**
        * sqlSession中包含了面向数据库执行sql命令所需的所有方法
        */
       public static SqlSession getSqlSession(){
           //openSession(true) 自动提交事务
           return sqlSessionFactory.openSession();
       }
   
   }
   ```

4. ##### 在dao包下根据DAO接口创建Mapper的xml文件内容

   ```java
   /**
    * user表的DAO接口(提供数据库访问对象)
    * @author 刘嘉宁
    */
   public interface UserDao {
       /**
        * 查询user表中所有信息的方法
        * @return 用户的List集合
        */
       List<User> getUserList();
   }
   ```

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <!--namespace: 绑定一个对应的Dao/Mapper接口-->
   <mapper namespace="com.kuang.dao.UserDao">
       <!--id: namespace中对应的方法， resultType: 返回类型-->
       <select id="getUserList" resultType="com.kuang.pojo.User">
           select * from 库.user;
       </select>
   </mapper>
   ```



## 增删改的实现：

1. ##### 编写接口

   添加：*删除修改同理*

   ```java
   /**
    * 添加用户
    * @param user 用户
    * @return 添加成功条数 1:成功 0:失败
    */
   int addUser(User user);
   ```

2. ##### 编写对应的Mapper.xml中的语句

   添加：*删除修改同理*

   ```xml
   <!--id: namespace中对应的方法, parameter: 形参类型-->
   <insert id="addUser" parameterType="com.kuang.pojo.User">
       insert into mybatis.user (id, name, pwd) values(#{id}, #{name}, #{pwd});
   </insert>
   ```

3. ##### 测试使用

   添加：*删除修改同理*

   ```java
   @Test
   public void addUserTest() {
       SqlSession sqlSession = null;
       try {
           sqlSession = MyBatisUtils.getSqlSession();
           UserDao mapper = sqlSession.getMapper(UserDao.class);
           int res = mapper.addUser(new User(5, "mike", "mike123"));
           if (res == 1){
               System.out.println("添加成功");
           }else {
               System.out.println("添加失败");
           }
       } catch (Exception e) {
           e.printStackTrace();
       } finally {
   	    //增删改时注意提交事务!!
           sqlSession.commit();
           sqlSession.close();
       }
   }
   ```

   

## 模糊查询实现方式：

1. 在java执行代码的时候，传递通配符% %

   ```java
   HashMap<String, Object> map = new HashMap<>();
   map.put("userName", "%王%");
   ```

2. 在sql拼接中使用通配符

   ```sql
   select * from mybatis.user where name like "%"#{userName}"%";
   ```



## mybatis-config.xml核心配置文件：

1. ##### 属性： 

   - 可以实现外部配置文件的引用。

   - resource引用的.properties配置文件**优先级高**，属性配置标签中起到补充配置的作用**优先级低**。

   - 属性配置**相当于全局变量**，可以在下文使用`${ 属性的name }`引用到其value

     ```xml
     <!--属性: 实现引用外置配置文件（优先）-->
     <properties resource="db.properties">
         <!--在其中增加属性配置-->
         <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
     </properties>
     ```

2. ##### 别名：

   - 为java类型设置一个短的别名

   - 为包起别名，默认名是包中的类名首字母小写

     ```xml
     <!--别名-->
     <typeAliases>
         <!--为实体类起别名-->
         <typeAlias type="com.kuang.pojo.User" alias="User" />
         <!--为包起别名，默认名是包中的类名首字母小写-->
         <package name="com.kuang.pojo" />
     </typeAliases>
     ```

   - 在实体类上添加注解可以代替第一种方式：

     ```java
     @Alias("user")
     public class User{
     	...
     }
     ```

   - 一些java创建的类型都有默认别名:
     - int --> _int
     - Integer --> int
     - Map --> map

3. ##### 设置：

   - 它们会改变 MyBatis 的运行时行为, 常用设置：

     | 设置名                   | 描述                                                         | 有效值                                                       | 默认值 |
     | :----------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----- |
     | cacheEnabled             | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。     | true \| false                                                | true   |
     | lazyLoadingEnabled       | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。 | true \| false                                                | false  |
     | useGeneratedKeys         | 允许 JDBC 支持自动生成主键，需要数据库驱动支持。如果设置为 true，将强制使用自动生成主键。尽管一些数据库驱动不支持此特性，但仍可正常工作（如 Derby）。 | true \| false                                                | False  |
     | mapUnderscoreToCamelCase | 是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。 | true \| false                                                | False  |
     | logImpl                  | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。        | SLF4J \| LOG4J \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置 |

4. ##### 环境：

   - 指定事务管理器

   - 指定数据源

     ```xml
     <!--环境配置-->
     <environments default="development">
         <environment id="development">
             <!--事务管理器：JDBC/MANAGED-->
             <transactionManager type="JDBC"/>
             <!--数据源：UNPOOLED没有池/POOLED有池/UNDI-->
             <dataSource type="POOLED">
                 <property name="driver" value="${driver}"/>
                 <property name="url" value="${url}"/>
                 <property name="username" value="${username}"/>
                 <property name="password" value="${password}"/>
             </dataSource>
         </environment>
     </environments>
     ```

5. ##### 映射：

   映射：告诉 MyBatis 去哪里找映射文件

   - 方式一：使用路径绑定xml文件

   ```xml
   <!--注册mapper, 每一个mapper.xml都需要在MyBatis核心配置中注册-->
   <mappers>
       <mapper resource="com/kuang/dao/UserMapper.xml"/>
   </mappers>
   ```

   - 方式二：使用class文件绑定
     - 注意1：接口和它的Mapper配置文件必须同名
     - 注意2：接口和它的Mapper配置文件必须在同一包下

   ```xml
   <!--注册mapper, 每一个mapper.xml都需要在MyBatis核心配置中注册-->
   <mappers>
       <mapper class="com.kuang.dao.UserMapper"/>
   </mappers>
   ```

   - 方式三：package标签
     - 定位到包下，保证mapper文件在dao包下
   
   ```xml
   <mappers>
       <package name="com.bjpn.crm.settings.dao"/>
   </mappers>
   ```



## 生命周期和作用域：

1. ##### MyBatis执行原理：

   **程序通过mybatis-config.xml文件通过SqlSessionFactoryBuilder建造SqlSessionFactory得到SqlSession对象，通过反射动态代理出接口的mapper对象执行对应方法。**

2. ##### SqlSessionFactoryBuilder：

   - 由用户创建，在执行完build（）方法后自动销毁

3. ##### SqlSessionFactory

   - 相当于数据库连接池

   - 由SqlSessionFactoryBuilder创建，程序运行期间一直存在，程序结束时销毁
   - 使用单例模式、静态单例模式，最佳作用域为全局作用域

4. ##### SqlSession

   - 相当于连接到连接池的一个请求，每一个都是单独的线程不是线程安全的，不能共享
   - 在使用时被创建，反射出mapper对象，提交事务后被手动关闭销毁

   - 最佳作用域为请求时或使用时的方法内



## 结果集映射：

- MyBatis会将mapper.xml中查询到的内容依照resultType返回，在返回时会自动创建所需实例(**MyBatis在幕后会自动创建一个resultMap，再基于属性名来映射列到JavaBean的属性上**)，会自动找与表中字段名**相同**的属性名赋值
- 如何解决表字段名与类属性名不一致：使用结果集映射

解决方式：

 1. 为字段起别名，别名与类变量名一致

    ```xml
    <select id="getUserById" resultType="User" parameterType="int">
        select id, name, pwd as pswd from mybatis.user where id = #{id};
    </select>
    ```

2. **结果集映射**：将mapper.xml中select标签的resultType替换为resultMap

   ```xml
   <resultMap id="UserMap" type="User">
       <!--将数据库中的主键字段映射为实体类中的ID-->
       <id column="id" property="ID" />
       <!--将数据库中的pwd字段映射为实体类中的pswd-->
       <result column="pwd" property="pswd" />
   </resultMap>
   <!--id: namespace中对应的方法, resultType: 返回类型, parameter: 形参类型-->
   <select id="getUserById" resultMap="UserMap" parameterType="int">
       select * from mybatis.user where id = #{id};
   </select>
   ```



## 日志工厂：

- 日志：排错，排查异常用。

- 日志工厂：logImpl

| 设置名  | 描述                                                  | 有效值                                                       | 默认值 |
| :------ | :---------------------------------------------------- | :----------------------------------------------------------- | :----- |
| logImpl | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。 | SLF4J \| LOG4J \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置 |

1. LOG4J : 通过配置文件灵活控制日志工厂输出内容
2. STDOUT_LOGGING : 标准日志工厂输出内容
3. NO_LOGGING : 无日志

log4j：

1. **导包**：导入maven中的log4j

   ```xml
   <!-- https://mvnrepository.com/artifact/log4j/log4j -->
   <dependency>
       <groupId>log4j</groupId>
       <artifactId>log4j</artifactId>
       <version>1.2.17</version>
   </dependency>
   ```

2. **设置目录实现**：设置日志的具体实现为log4j，在mybatis-config.xml中添加：

   ```xml
   <!--设置-->
   <settings>
       <!--设置日志工厂-->
       <setting name="logImpl" value="LOG4J"/>
   </settings>
   ```

3. **配置文件**：在resources目录下创建log4j.properties配置文件：

   ```properties
   #将等级为DEBUG的日志信息输出到console和file这两个目的地，console和file的定义在下面的代码
   log4j.rootLogger=DEBUG,console,file
   
   #控制台输出的相关设置
   log4j.appender.console = org.apache.log4j.ConsoleAppender
   log4j.appender.console.Target = System.out
   log4j.appender.console.Threshold=DEBUG
   log4j.appender.console.layout = org.apache.log4j.PatternLayout
   log4j.appender.console.layout.ConversionPattern=[%c]-%m%n
   
   #文件输出的相关设置
   log4j.appender.file = org.apache.log4j.RollingFileAppender
   log4j.appender.file.File=./log/kuang.log
   log4j.appender.file.MaxFileSize=10mb
   log4j.appender.file.Threshold=DEBUG
   log4j.appender.file.layout=org.apache.log4j.PatternLayout
   log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n
   
   #日志输出级别
   log4j.logger.org.mybatis=DEBUG
   log4j.logger.java.sql=DEBUG
   log4j.logger.java.sql.Statement=DEBUG
   log4j.logger.java.sql.ResultSet=DEBUG
   log4j.logger.java.sql.PreparedStatement=DEBUG
   
   ```

4. **使用**:

   1. 导包：`import org.apache.log4j.Logger;`

   2. 创建日志类对象，参数为当前类的class：`static Logger logger = Logger.getLogger(UserDaoTest.class);`

   3. 设置日志级别，在适当位置加入：

      ```java
      logger.info("info: 进入了testlog4j");
      logger.debug("debug: 进入了testlog4j");
      logger.error("error: 进入了testlog4j");
      ```

   4. 在log/网站名.log下追加日志



## 分页查询：

1. **使用pageHelper**：https://pagehelper.github.io/

2. 通过map传入limit的值：

```xml
<select id="getUserByLimit" resultType="user" parameterType="map">
    select * from mybatis.user limit #{startIndex},#{pageSize};
</select>
```

```java
//在测试中：查询第二页的两条数据
Map map = new HashMap();
map.put("startIndex", 2 * 1);
map.put("pageSize", 2);
List user = mapper.getUserByLimit(map);
```



## 使用注解开发：

- **面向接口编程**：定义与实现的分离，解耦合

- **不推荐使用注解映射MyBatis**

- 使用注解映射简单的sql语句（当需要用到resultMap等功能就GG）

  ```java
  /**
   * 根据name查询所有用户信息
   * @return 用户信息的List集合
   */
  @Select("select * from mybatis.user where name = #{sname};")
  List<User> getUserListById(@Param("sname")int name);
  ```



## MyBatis底层原理：

1. **Resources获取加载全局配置文件**
2. 实例化SqlSessionFactoryBuilder构造器
3. 解析配置文件流XMLConfigBuilder
4. Configuration所有的配置信息
5. **SqlSessionFactory实例化**
6. transactional事务管理
7. 创建executor执行器
8. **创建SqlSession**
9. **实现CRUD，执行失败返回第六步**
10. 提交事务
11. **关闭SqlSession**



## Lombok：

lombok: 通过简单的注解形式来简化java代码，提高开发人员的开发效率

- Lombok没法实现多种参数构造器的重载。

- 大大降低了源代码的可读性和完整性

使用lombok：

1. 在IDEA中安装lombok插件，
2. 在项目中导入lombok的jar包
3. 在实体类上加注解

> @Getter / @Setter：为相应的属性自动生成 Getter / Setter 方法
> @FieldNameConstants
> @ToString：生成一个toString()方法，默认情况下，会输出类名、所有属性（会按照属性定义顺序），用逗号来分割。
> @EqualsAndHashCode：生成equals和hasCode
> @AllArgsConstructor, @RequiredArgsConstructor and @NoArgsConstructor：无参构造器、部分参数构造器、全参构造器。
> @Log, @Log4j, @Log4j2, @Slf4j, @XSlf4j, @CommonsLog, @JBossLog, @Flogger, @CustomLog
> @Data：为类的所有属性自动生成setter/getter、equals、canEqual、hashCode、toString方法，final属性无setter方法。
> @Builder
> @SuperBuilder
> @Singular
> @Delegate
> @Value
> @Accessors
> @Wither
> @With
> @SneakyThrows
> @val
> @var
> @UtilityClass



## 多对一的处理：

- 在子表实体类中组合主表实体类（替代外键）

  ![image-20210826195315186](D:\Document\MyBatis\MyBatis自学笔记.assets\image-20210826195315186.png)

1. 按照**查询**嵌套处理：

   - 写两个查询类似子查询，然后再查询第一个的时候带上子查询的内容

   ```xml
   <resultMap id="studentTeacher" type="Student">
       <result column="id" property="id" />
       <result column="name" property="name" />
       <!--复杂的属性需要单独处理： 对象(关联)：association-->
       <association javaType="Teacher" property="teacher" select="getTeacher" column="tid" />
   </resultMap>
   <select id="getStudentWithTeacherList" resultMap="studentTeacher">
       select * from student;
   </select>
   <select id="getTeacher" resultType="teacher">
       select * from teacher where id = #{tid(随便写自动匹配)};
   </select>
   ```

2. 按照**结果**嵌套处理：

   - 把要查的SQL写好，然后在resultMap中指定其对应对象、对应名字

   ```xml
   <resultMap id="studentTeacher2" type="Student">
       <result column="sid" property="id" />
       <result column="sname" property="name" />
       <association javaType="Teacher" property="teacher">
           <result column="tid" property="id"/>
           <result column="tname" property="name" />
       </association>
   </resultMap>
   <select id="getStudentWithTeacherList2" resultMap="studentTeacher2">
       select s.id sid, s.name sname, t.id tid, t.name tname
       from student s, teacher t
       where s. tid = t.id;
   </select>
   ```



## 一对多的处理：

- 在主表实体类中添加子表实体类的集合（一个主对应多个子）

![image-20210826195437093](D:\Document\MyBatis\MyBatis自学笔记.assets\image-20210826195437093.png)

1. 按照**查询**嵌套处理

   - 两个SQL中都找的是符合条件的内容

   ```xml
   <resultMap id="teacherStudent2" type="Teacher">
       <result column="id" property="id"/>
       <result column="name" property="name"/>
       <!--复杂的属性需要单独处理： 集合：collection-->
       <!--ofType: 集合中的泛型信息-->
       <collection javaType="List" ofType="Student" property="students" select="getStudentList2" column="id"/>
   </resultMap>
   <select id="getTeacherWithStudentList2" resultMap="teacherStudent2">
       select * from teacher where id = #{tid};
   </select>
   <select id="getStudentList2" resultType="Student">
       select * from student where tid = #{tid};
   </select>
   ```

2. 按照**结果**嵌套处理

   - 将每一个查询的字段均重新命名并重新映射

   ```xml
   <resultMap id="teacherStudent" type="Teacher">
       <result column="tid" property="id"/>
       <result column="tname" property="name"/>
       <collection ofType="Student" property="students">
           <result column="sid" property="id" />
           <result column="sname" property="name" />
           <result column="stid" property="tid" />
       </collection>
   </resultMap>
   <select id="getTeacherWithStudentList" resultMap="teacherStudent" >
       select t.id tid, t.name tname, s.id sid, s.name sname, s.tid stid
       from teacher t
       join student s
       where t.id = s.tid and t.id = #{tid};
   </select>
   ```



## 动态SQL：

什么是动态SQL：指根据不同的条件生成不同的SQL语句（拼接SQL语句），在SQL层面去执行一个逻辑代码。

- 通过动态内容查询：如果map里有title那就以title为条件，如果map里有author那就以author为条件查询，如果都没有，那就查询所有

  ```xml
  <select id="queryBookIf" parameterType="map" resultType="Blog">
      select * from blog where 2>1
      <!--if标签，test中为布尔类型表达式（过滤条件）-->
      <if test="title != null">
          and title=#{title}
      </if>
      <if test="author != null">
          and author=#{author}
      </if>
  </select>
  ```

1. choose, when, otherwise标签

   - 相当于switch case: ...break; defaule

   ```xml
   <select id="findActiveBlogLike" resultType="Blog">
     SELECT * FROM BLOG WHERE state = ‘ACTIVE’
     <choose>
       <when test="title != null">
         AND title like #{title}
       </when>
       <when test="author != null and author.name != null">
         AND author_name like #{author.name}
       </when>
       <otherwise>
         AND featured = 1
       </otherwise>
     </choose>
   </select>
   ```

2. if, trim, where/set

   - *trim*自动去除那些标签的前缀/后缀

   - *where* 元素只会在子元素有返回任何内容的情况下才插入 “WHERE” 子句。
   - *set*会自动去除多余逗号，

   ```xml
   <select id="findActiveBlogLike" resultType="Blog">
     SELECT * FROM BLOG
     <where>
       <!--这里的第一个and会自动插入-->
       <if test="state != null">
            state = #{state}
       </if>
       <if test="title != null">
           AND title like #{title}
       </if>
       <if test="author != null and author.name != null">
           AND author_name like #{author.name}
       </if>
     </where>
   </select>
   ```

3. foreach

   - 对传入的集合进行遍历（格式化）, 常用在in子句上

   ```xml
   <select id="selectPostIn" resultType="domain.blog.Post" parameterType="map">
     SELECT *
     FROM POST P
     WHERE ID in
     <!--list是map中的一个key对应的是一个集合 最后拼接为：(item1, item2, item3, ... ) -->
     <foreach item="item" index="index" collection="list"
         open="(" separator="," close=")">
           #{item}
     </foreach>
   </select>
   ```

4. sql标签

   - 把公用的sql代码块独立，允许多处include引用

   - 注意1：最好基于单表来定义sql标签
   - 注意2：不要存在where标签

   ```xml
   <!--公用的sql语句块-->
   <sql id="if-title-author">
       <if test="title != null">
           and title=#{title}
       </if>
       <if test="author != null">
           and author=#{author}
       </if>
   </sql>
   
   <!--通过动态内容查询：如果map里有title那就以title为条件，如果map里有author那就以author为条件查询，如果都没有，那就查询所有-->
   <select id="queryBookIf" parameterType="map" resultType="Blog">
       select * from blog
       <where>
           <include refid="if-title-author"></include>
       </where>
   
   </select>
   ```



## 缓存：

什么是缓存：临时放在内存中的表数据信息

为什么使用缓存：减少和数据库的交互次数，减少系统开销，提高系统效率

什么样的数据适合使用缓存：经常查询并不经常改变的数据

- 一级缓存：默认开启，SqlSession级别，本地缓存（最近最少使用）

  - LRU：移除最长时间不用的缓存
  - 一级缓存是以statemntId, param, boundsql, rowsbound为key的hashmap,在下次如果下次查询也匹配到了这个key, 就可以直接取出这个对象而不用去查数据库
  - 当前会话被关闭时，一级缓存就没了，就可以从二级缓存中获取内容
  - select语句使用缓存， insert、update、delete语句刷新缓存
  - `sqlSession.clearCache()`清理缓存

- 二级缓存：手动开启，namespace级别，全局缓存（先进先出）

  - 当前会话被关闭时，一级缓存就没了，就可以从二级缓存中获取内容（同一个mapper内共享二级缓存）

  开启方式： 

  - 在mybatis-config.xml中添加setting

    ```xml
    <!--显式的开启全局缓存-->
    <setting name="cacheEnabled" value="true"/>
    ```

  - 在mapper.xml中添加行

    ```xml
    <!--开启二级缓存-->
    <cache/>
    ```

    或

    ```xml
    <cache eviction="FIFO"
           flushInterval="60000"
           size="512"
           readOnly="true"/>
    ```

  二级缓存问题：

  - `<cache/>`标签默认readOnly为false，需要实例化实体类解决

- **用户在查询时先经过mapper，先看二级缓存有没有，再看一级缓存有没有，如果都没有才会查询数据库**

![image-20210828224727657](D:\Document\MyBatis\MyBatis自学笔记.assets\image-20210828224727657.png)

- 自定义缓存：

  在cache标签中执行type属性

  - ehcache：hibernate用到了，略
  - redis的缓存，略


