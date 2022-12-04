## 入门

### mybatis是什么

MyBatis 是一款持久层框架，一个半 ORM（对象关系映射）框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生类型、接口和 Java 的 POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

### orm是什么

对象关系映射，是一种为了解决关系型数据库数据与简单Java对象（POJO）的映射关系的技术

过使用描述对象和数据库之间映射的元数据，将程序中的对象自动持久化到关系型数据库中

### 传统JDBC开发存在的问题

 频繁创建数据库连接对象、释放，容易造成系统资源浪费，影响系统性能

sql语句定义、参数设置、结果集处理存在硬编码

结果集处理存在重复代码，处理麻烦。

### mybatis适用场景

- MyBatis专注于SQL本身，是一个足够灵活的DAO层解决方案。
- 对性能的要求很高，或者需求变化较多的项目，如互联网项目，MyBatis将是不错的选择。

## 映射器

### #{}和${}区别

- \#{}是占位符，预编译处理；${}是拼接符，字符串替换，没有预编译处理。
- Mybatis在处理#{}时，#{}传入参数是以字符串传入，会将SQL中的#{}替换为?号，调用PreparedStatement的set方法来赋值。
- Mybatis在处理时 ， 是 原 值 传 入 ， 就 是 把 {}时，是原值传入，就是把时，是原值传入，就是把{}替换成变量的值，相当于JDBC中的Statement编译
- 变量替换后，#{} 对应的变量自动加上单引号 ‘’；变量替换后，${} 对应的变量不会加上单引号 ‘’
- #{} 可以有效的防止SQL注入，提高系统安全性；${} 不能防止SQL 注入

### 模糊查询like如何使用

CONCAT(’%’,#{question},’%’) 使用CONCAT()函数，推荐

使用bind标签

```sql
<select id="listUserLikeUsername" resultType="com.jourwon.pojo.User">
　　<bind name="pattern" value="'%' + username + '%'" />
　　select id,sex,age,username,password from person where username LIKE #{pattern}
</select>
```



### mapper中如何传递多个参数

#### 顺序传参法

```sql
public User selectUser(String name, int deptId);

<select id="selectUser" resultMap="UserResultMap">
    select * from user
    where user_name = #{0} and dept_id = #{1}
</select>
```

\#{}里面的数字代表传入参数的顺序。

这种方法不建议使用，sql层表达不直观，且一旦顺序调整容易出错。

#### @param注解传参

```sql
public User selectUser(@Param("userName") String name, int @Param("deptId") deptId);

<select id="selectUser" resultMap="UserResultMap">
    select * from user
    where user_name = #{userName} and dept_id = #{deptId}
</select>
```

\#{}里面的名称对应的是注解@Param括号里面修饰的名称。

这种方法在参数不多的情况还是比较直观的，推荐使用。

#### map传参

```sql
public User selectUser(Map<String, Object> params);

<select id="selectUser" parameterType="java.util.Map" resultMap="UserResultMap">
    select * from user
    where user_name = #{userName} and dept_id = #{deptId}
</select>
```

\#{}里面的名称对应的是Map里面的key名称。

这种方法适合传递多个参数，且参数易变能灵活传递的情况。

#### javabean 传参

```sql
public User selectUser(User user);

<select id="selectUser" parameterType="com.jourwon.pojo.User" resultMap="UserResultMap">
    select * from user
    where user_name = #{userName} and dept_id = #{deptId}
</select>
```

\#{}里面的名称对应的是User类里面的成员属性。

这种方法直观，需要建一个实体类，扩展不容易，需要加属性，但代码可读性强，业务逻辑处理方便，推荐使用。

### 如何获取生成的主键

**对于支持主键自增的数据库（MySQL）**

```sql
<insert id="insertUser" useGeneratedKeys="true" keyProperty="userId" >
    insert into user( 
    user_name, user_password, create_time) 
    values(#{userName}, #{userPassword} , #{createTime, jdbcType= TIMESTAMP})
</insert>
```

```sql
<selectKey keyColumn="id" resultType="long" keyProperty="id" order="BEFORE">
</selectKey> 
```



### 当实体类中的属性名和表中的字段名不一样 ，怎么办

通过在查询的SQL语句中定义字段名的别名，让字段名的别名和实体类的属性名一致。

```sql
<select id="getOrder" parameterType="int" resultType="com.jourwon.pojo.Order">
       select order_id id, order_no orderno ,order_price price form orders where order_id=#{id};
</select>

```

 通过`<resultMap>`来映射字段名和实体类属性名的一一对应的关系。

```sql
<select id="getOrder" parameterType="int" resultMap="orderResultMap">
	select * from orders where order_id=#{id}
</select>

<resultMap type="com.jourwon.pojo.Order" id="orderResultMap">
    <!–用id属性来映射主键字段–>
    <id property="id" column="order_id">

    <!–用result属性来映射非主键字段，property为实体类属性名，column为数据库表中的属性–>
    <result property ="orderno" column ="order_no"/>
    <result property="price" column="order_price" />
</reslutMap>
```



### Mapper 编写有哪几种方式？



### 什么是MyBatis的接口绑定？有哪些实现方式？

通过注解绑定，就是在接口的方法上面加上 @Select、@Update等注解，里面包含Sql语句来绑定；

通过xml里面写SQL来绑定， 在这种情况下，要指定xml映射文件里面的namespace必须为接口的全路径名。当Sql语句比较简单时候，用注解绑定， 当SQL语句比较复杂时候，用xml绑定，一般用xml绑定的比较多。

### 使用MyBatis的mapper接口调用时有哪些要求？

1、Mapper接口方法名和mapper.xml中定义的每个sql的id相同。

2、Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql 的parameterType的类型相同。

3、Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同。

4、Mapper.xml文件中的namespace即是mapper接口的类路径。

### 最佳实践中，通常一个Xml映射文件，都会写一个Dao接口与之对应，请问，这个Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗

Dao接口，就是人们常说的Mapper接口，接口的全限名，就是映射文件中的namespace的值，接口的方法名，就是映射文件中MappedStatement的id值，接口方法内的参数，就是传递给sql的参数。Mapper接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为key值，可唯一定位一个MappedStatement，举例：com.mybatis3.mappers.StudentDao.findStudentById，可以唯一找到namespace为com.mybatis3.mappers.StudentDao下面id = findStudentById的MappedStatement。在Mybatis中，每一个<select>、<insert>、<update>、<delete>标签，都会被解析为一个MappedStatement对象。

Dao接口里的方法，是不能重载的，因为是全限名+方法名的保存和寻找策略。

Dao接口的工作原理是JDK动态代理，Mybatis运行时会使用JDK动态代理为Dao接口生成代理proxy对象，代理对象proxy会拦截接口方法，转而执行MappedStatement所代表的sql，然后将sql执行结果返回。

### Mybatis的Xml映射文件中，不同的Xml映射文件，id是否可以重复？

不同的Xml映射文件，如果配置了namespace，那么id可以重复；如果没有配置namespace，那么id不能重复；毕竟namespace不是必须的，只是最佳实践而已。

原因就是namespace+id是作为Map<String, MappedStatement>的key使用的，如果没有namespace，就剩下id，那么，id重复会导致数据互相覆盖。有了namespace，自然id就可以重复，namespace不同，namespace+id自然也就不同。

### resultMap和resulttype区别

MyBatis中在查询进行select映射的时候，返回类型可以用resultType，也可以用resultMap，resultType是直接表示返回类型的，而resultMap则是对外部ResultMap的引用，但是resultType跟resultMap不能同时存在。
在MyBatis进行查询映射时，其实查询出来的每一个属性都是放在一个对应的Map里面的，其中键是属性名，值则是其对应的值。

**对于resulttype而已**

如果查询结果只是返回一个值，比如返回String或int，那么可以使用resultType指定简单类型作为输出结果。

还有一种情况就是如果数据库表的字段名和实体bean对象的属性名一样时，那么也可以直接使用resultType返回结果。

MyBatis在执行sql语句时，会把查询出来的字段名和resultType定义实体bean对象的属性进行一一对应，然后再把查询到的值放到实体bean对象的属性中，完成赋值操作。但如果不一样，则会查询出空值。



**对于resultmap**

resultType不需要配置，但是resultMap要配置一下，将数据库表的字段名和实体bean对象类的属性名一一对应关系，这样的话就算你的数据库的字段名和你的实体类的属性名不一样也没有关系，都会给你对应的映射出来，所以resultMap要更强大一些。

关联查询

一对一

```sql
<!-- 订单查询关联用户的resultMap
    将整个查询的结果映射到cn.itcast.mybatis.po.Orders中
     -->
    <resultMap type="cn.itcast.mybatis.po.Orders" id="OrdersUserResultMap">
        <!-- 配置映射的订单信息 -->
        <!-- id：指定查询列中的唯 一标识，订单信息的中的唯 一标识，如果有多个列组成唯一标识，配置多个id
            column：订单信息的唯 一标识 列
            property：订单信息的唯 一标识 列所映射到Orders中哪个属性
          -->
        <id column="id" property="id"/>
        <result column="user_id" property="userId"/>
        <result column="number" property="number"/>
        <result column="createtime" property="createtime"/>
        <result column="note" property=note/>
        
        <!-- 配置映射的关联的用户信息 -->
        <!-- association：用于映射关联查询单个对象的信息
        property：要将关联查询的用户信息映射到Orders中哪个属性
         -->
        <association property="user"  javaType="cn.itcast.mybatis.po.User">
            <!-- id：关联查询用户的唯 一标识
            column：指定唯 一标识用户信息的列
            javaType：映射到user的哪个属性
             -->
            <id column="user_id" property="id"/>
            <result column="username" property="username"/>
            <result column="sex" property="sex"/>
            <result column="address" property="address"/>
        
        </association>
    </resultMap>
```

一对多

```sql
-- 订单及订单明细的resultMap
    使用extends继承，不用在中配置订单信息和用户信息的映射
     -->
    <resultMap type="cn.itcast.mybatis.po.Orders" id="OrdersAndOrderDetailResultMap" extends="OrdersUserResultMap">
        <!-- 订单信息 -->
        <!-- 用户信息 -->
        <!-- 使用extends继承，不用在中配置订单信息和用户信息的映射 -->
        
        
        <!-- 订单明细信息
        一个订单关联查询出了多条明细，要使用collection进行映射
        collection：对关联查询到多条记录映射到集合对象中
        property：将关联查询到多条记录映射到cn.itcast.mybatis.po.Orders哪个属性
        ofType：指定映射到list集合属性中pojo的类型
         -->
         <collection property="orderdetails" ofType="cn.itcast.mybatis.po.Orderdetail">
             <!-- id：订单明细唯 一标识
             property:要将订单明细的唯 一标识 映射到cn.itcast.mybatis.po.Orderdetail的哪个属性
               -->
             <id column="orderdetail_id" property="id"/>
             <result column="items_id" property="itemsId"/>
             <result column="items_num" property="itemsNum"/>
             <result column="orders_id" property="ordersId"/>
         </collection>
        
    
    </resultMap>
```



### @requestParam，@PathVariable，@requestbody区别

#### 1.1、@Pathvariable
通过占位符的方式获取入参，前端示例：url:http://localhost:8080/system/student/${stuSno}
也即是从路径里面去获取变量
后端：

```java
    /**
     * @param stuSno 学号
     * @return 学生信息
     * @description 根据主键获取学生信息
     */
    @GetMapping("/selectByPrimaryKey/{stuSno}")
    public Student selectByPrimaryKey(@PathVariable String stuSno) {
        return studentService.selectByPrimaryKey(stuSno);
    }
```
这种情况是方法参数名称和需要绑定的url中变量名称一致时
若是若方法参数名称和需要绑定的url中变量名称不一致时
后端：

```java
/**
 * @param stuSno 学号
 * @return 学生信息
 * @description 根据主键获取学生信息
 */
@GetMapping("/selectByPrimaryKey/{stuSno}")
public Student selectByPrimaryKey(@PathVariable("stuSno") String sno) {
    return studentService.selectByPrimaryKey(stuSno);
}
```

> **注意：前端传参的URL于后端@RequestMapping的URL必须相同且参数位置一一对应，否则前端会找不到后端地址**

#### 1.2、@RequestParam

 1. 作用
      将请求参数绑定在控制层（controller）方法参数【springmvc注解】
 2. 语法

```java
@RequestParam(value="参数名",required="true/false",default="")
```
![在这里插入图片描述](http://oss.geekmice.cn/blog/2b6de6749c434bb2b4b17a4ed95bfb4a.png)

**value**:表示前端传过来的值名称，如果你不设置，那就默认使用服务端使用的参数名称（stuSno）
不设置：
前端：![在这里插入图片描述](http://oss.geekmice.cn/blog/2b6de6749c434bb2b4b17a4ed95bfb4a.png)

```java
http://localhost:8081/student/selectByPrimaryKey1?stuSno=0001
```

后端：
![在这里插入图片描述](http://oss.geekmice.cn/blog/2b6de6749c434bb2b4b17a4ed95bfb4a.png)

```java
public Student selectByPrimaryKey1(@RequestParam String stuSno) {
```

设置
前端：
![在这里插入图片描述](http://oss.geekmice.cn/blog/2b6de6749c434bb2b4b17a4ed95bfb4a.png)

```java
http://localhost:8081/student/selectByPrimaryKey1?sno=0001
```

后端：
![在这里插入图片描述](http://oss.geekmice.cn/blog/2b6de6749c434bb2b4b17a4ed95bfb4a.png)
此时@requestParam中value="sno"  value可以省略 直接输入“sno”，类似于@RequestMapping
```java
    public Student selectByPrimaryKey1(@RequestParam("sno") String stuSno) {
```
这时候前端传sno并非stuSno，需要在@requestParam中value设置sno
**required**:是否包含该参数，默认为true,表示该请求路径中必须包含该参数，如果不包含就报错
![在这里插入图片描述](http://oss.geekmice.cn/blog/2b6de6749c434bb2b4b17a4ed95bfb4a.png)

```java
2021-08-15 02:26:30.495  WARN 4736 --- [nio-8081-exec-2] .w.s.m.s.DefaultHandlerExceptionResolver : Resolved [org.springframework.web.bind.MissingServletRequestParameterException: Required request parameter 'sno' for method parameter type String is not present]

```
defaultValue:默认参数值，如果设置了该值，required=true将失效，自动变为false,如果没有传该参数，使用默认值；比如说此时 后端直接写成 defaultValue="0001"
![在这里插入图片描述](https://img-blog.csdnimg.cn/c5f95497b6fb4f9b98ae48a69d7de7f5.png)![在这里插入图片描述](https://img-blog.csdnimg.cn/158af15253a14444bf71daa77ad27cea.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dyZWVrNzc3Nw==,size_16,color_FFFFFF,t_70)

 4. 示例说明
 后端controller

```java
    /**
     * @param stuSno 学号
     * @return 学生信息
     * @description 根据主键获取学生信息
     */
    @GetMapping("/selectByPrimaryKey")
    public Student selectByPrimaryKey1(@RequestParam(value = "sno",defaultValue = "0001",required = false) String stuSno) {
        return studentService.selectByPrimaryKey(stuSno);
    }
```
前端暂时使用 postman
![在这里插入图片描述](https://img-blog.csdnimg.cn/203de7c9da7641ad93fd45641fa67138.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dyZWVrNzc3Nw==,size_16,color_FFFFFF,t_70)

#### 1.3、@RequestBody
@RequestBody主要用来接收前端传递给后端的json字符串中的数据的(请求体中的数据的)

 1. @RequestBody直接以String接收前端传过来的json数据
 后端代码

```java
    /**
     * @param stuSno 学号
     * @return 学生信息
     * @description 根据主键获取学生信息
     */
    @GetMapping("/selectByPrimaryKey3")
    public Student selectByPrimaryKey2(@RequestBody String jsonString) {
        // 使用fastjson解析json格式字符串为json对象
        JSONObject jsonObject = JSONObject.parseObject(jsonString);
        // 获取学号
        String stuSno = jsonObject.getString("stuSno");
        return studentService.selectByPrimaryKey(stuSno);
    }
```

前端postman
![在这里插入图片描述](https://img-blog.csdnimg.cn/ef49d1b2847746f9a500775b068d2152.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dyZWVrNzc3Nw==,size_16,color_FFFFFF,t_70)
需要通过fastjson转换json字符串为json对象从而获取相应的值，否则报错
![在这里插入图片描述](https://img-blog.csdnimg.cn/456dba63e0174273ab06e890391d1178.png)

 3. @RequestBody以简单对象接收前端传过来的json数据
 实体类


```java
package com.geekmice.springbootrequestparam.pojo;


import com.fasterxml.jackson.annotation.JsonFormat;

import java.io.Serializable;
import java.util.Date;

public class Student implements Serializable {

    private String stuSno;
    private String stuName;
    private String stuBorn;
    private String stuSex;

    public String getStuSno() {
        return stuSno;
    }

    public void setStuSno(String stuSno) {
        this.stuSno = stuSno;
    }

    public String getStuName() {
        return stuName;
    }

    public void setStuName(String stuName) {
        this.stuName = stuName;
    }

    public String getStuBorn() {
        return stuBorn;
    }

    public void setStuBorn(String stuBorn) {
        this.stuBorn = stuBorn;
    }

    public String getStuSex() {
        return stuSex;
    }

    public void setStuSex(String stuSex) {
        this.stuSex = stuSex;
    }

    @Override
    public String toString() {
        return "Student{" +
                "stuSno='" + stuSno + '\'' +
                ", stuName='" + stuName + '\'' +
                ", stuBorn='" + stuBorn + '\'' +
                ", stuSex='" + stuSex + '\'' +
                '}';
    }
}

```
dao层

```java
    /**
     * @param student 学生信息
     * @return 返回学生信息
     * @description 根据学生对象获取学生信息
     */
    List<Student> selectByPrimaryKeySelective(Student student);
```

 xml
```java
<!--获取学生信息-->
    <select id="selectByPrimaryKeySelective" resultType="student" parameterType="student">
        select
        <include refid="Base_Column_List"/>
        from student
        <where>
            <if test="stuSno != '' and stuSno != null">
                stu_sno = #{stuSno}
            </if>
            <if test="stuName != '' and stuName != null">
                and stu_name = #{stuName}
            </if>
            <if test="stuBorn != '' and stuBorn != null">
                and stu_born = #{stuBorn}
            </if>
            <if test="stuSex != '' and stuSex != null">
                and stu_sex = #{stuSex}
            </if>
        </where>
    </select>
```
service

```java
    /**
     * @description 根据学生对象获取学生信息
     * @param student 学生信息
     * @return 返回学生信息
     */
    List<Student> selectByPrimaryKeySelective(Student student);
    @Override
    public List<Student> selectByPrimaryKeySelective(Student student) {
        return studentDao.selectByPrimaryKeySelective(student);
    }
```
 controller


```java
    /**
     * @param student 学生对象
     * @return 获取对应学生信息
     * @description 用户选择获取对应的学生信息
     */
    @PostMapping("/selectByPrimaryKeySelective")
    public List<Student> selectByPrimaryKeySelective(@RequestBody Student student) {
        return studentService.selectByPrimaryKeySelective(student);
    }
```
postman效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/7c130248d90f49df932afb8aabb364a3.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dyZWVrNzc3Nw==,size_16,color_FFFFFF,t_70)


 5. @RequestBody以复杂对象接收前端传过来的json数据
 复杂对象：Tim


```java
package com.geekmice.springbootrequestparam.pojo;

import java.util.List;

/**
 * @author pmb
 * @create 2021-08-15-4:34
 */
public class Tim {
    // 团队id
    private Integer id;

    // 团队名字
    private String timName;

    // 获得荣誉
    private List<String> honors;

    // 团队成员
    private List<Student> studentList;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getTimName() {
        return timName;
    }

    public void setTimName(String timName) {
        this.timName = timName;
    }

    public List<String> getHonors() {
        return honors;
    }

    public void setHonors(List<String> honors) {
        this.honors = honors;
    }

    public List<Student> getStudentList() {
        return studentList;
    }

    public void setStudentList(List<Student> studentList) {
        this.studentList = studentList;
    }

    @Override
    public String toString() {
        StringBuffer stringHonor = new StringBuffer("荣誉开始：。。。");
        for (String str : honors) {
            stringHonor.append(str);
            stringHonor.append("+");
        }
        StringBuffer stringTim = new StringBuffer("团队成员开始：。。。");
        for (Student student : studentList) {
            stringTim.append(student);
            stringTim.append("-");
        }
        stringHonor.append(stringTim);
        return stringHonor.toString();
    }
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/fc3412ab10a448ad9192a8efde54f60f.png)
postman
![在这里插入图片描述](https://img-blog.csdnimg.cn/0bf0aec9f3134991bfd563e4ad8cf961.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dyZWVrNzc3Nw==,size_16,color_FFFFFF,t_70)

 7. @RequestBody与简单的@RequestParam()同时使用

 controller
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/5a8709a557464a2d8a03eca05d27af73.png)

 postman![在这里插入图片描述](https://img-blog.csdnimg.cn/67729168f4f74453892d522641271d1b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dyZWVrNzc3Nw==,size_16,color_FFFFFF,t_70)

 9. @RequestBody与复杂的@RequestParam()同时使用
controller
![在这里插入图片描述](https://img-blog.csdnimg.cn/efe0caac39c44c4a9db18492f314d143.png)

postman 
![在这里插入图片描述](https://img-blog.csdnimg.cn/7e90dfd272f3425ba44889edaf2f4acc.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dyZWVrNzc3Nw==,size_16,color_FFFFFF,t_70)

 11. @RequestBody接收请求体中的json数据；不加注解接收URL中的数据并组装为对象
 ![在这里插入图片描述](http://oss.geekmice.cn/blog/511aa2261a2b46958265cf463e15ab75.png)

![在这里插入图片描述](http://oss.geekmice.cn/blog/511aa2261a2b46958265cf463e15ab75.png)

#### 1.4、不同之处&应用场景
我认为在单个参数提交 API 获取信息的时候，直接放在 URL 地址里，也就是使用 URI 模板的方式是非常方便的，而不使用 @PathVariable 还需要从 request 里提取指定参数，多一步操作，所以如果提取的是多个参数，而且是多个不同类型的参数，我觉得应该使用其他方式，也就是 @RequestParam


![在这里插入图片描述](http://oss.geekmice.cn/blog/511aa2261a2b46958265cf463e15ab75.png)

![在这里插入图片描述](http://oss.geekmice.cn/blog/3e94937240ee4466b6470a4de3e132dc.png)

### Mybatis是如何将sql执行结果封装为目标对象并返回的？都有哪些映射形式？

### Xml映射文件中，除了常见的select|insert|updae|delete标签之外，还有哪些标签？

#### where

主要是用来简化sql语句中where条件判断的，能智能的处理 and or 条件

```sql
where 1=1 
```

select * from t_blog where content = #{content}，而不是select * from t_blog where and content = #{content}，因为MyBatis会智能的把首个and 或 or 给忽略。



#### foreach

可以在SQL语句中进行迭代一个集合,item，index，collection，open，separator，close。

**（1）item**表示集合中每一个**元素**进行迭代时的**别名。**

**（2）index**指定一个名字，用于表示在迭代过程中，每次**迭代到**的**位置。**

**（3）open**表示该语句**以什么开始。**

**（4）separator**表示在每次进行迭代之间以什么符号作为**分隔符。**

**（5）close**表示**以什么结束。**

（1）如果传入的是**单参数**且**参数类型是一个List**的时候，collection属性值为**list**

（2）如果传入的是**单参数**且参数**类型是一个array数组**的时候，collection的属性值为**array**

（3）如果传入的**参数是多个**的时候，我们就需要把它们封装成一个Map了，当然单参数也可以封装成map，实际上如果你在传入参数的时候，在MyBatis里面也是会把它封装成一个Map的，map的key就是参数名，所以这个时候collection属性值就是传入的List或array对象在自己封装的map里面的key。

```sql
<select id="dynamicForeachTest" resultType="com.mybatis.entity.User">
    select * from t_user where id in
    <foreach collection="list" index="index" item="item" open="(" separator="," close=")">
        #{item}
    </foreach>
</select>
```



#### if

简单的条件判断，以往我们使用其他类型框架或者直接**使用JDBC的时候**， 如果我们要达到同样的选择效果的时候，我们就**需要拼SQL语句**，这是极其麻烦的，比起来，上述的动态SQL就要简单多了。

```sql
<select id="selectSelective" resultMap="BaseResultMap" parameterType="com.jack.springbootmybatis.pojo.Student">
    select
    <include refid="Base_Column_List" />
    from student
    <where>
      <if test="stuNo != null and stuNo != ''">
        stu_no=${stuNo}
      </if>
      <if test="stuName != null and stuName != ''">
        stu_name=${stuName}
      </if>
      <if test="stuBornDate != null and stuBornDate != ''">
        stu_born_date=${stuBornDate}
      </if>
      <if test="stuSex != null and stuSex != ''">
        stu_sex=${stuSex}
      </if>
    </where>
  </select>
```



#### choose

类似于java 语言中的 switch，when元素表示当when中的条件满足的时候就输出其中的内容，**跟JAVA中的switch效果差不多**的是按照条件的顺序，**当when中有条件满足的时候，就会跳出choose**，即所有的when和otherwise条件中，只有一个会输出，当所有的我很条件都不满足的时候就输出otherwise中的内容

```sql
 <select id="selectSelectiveSex" resultMap="BaseResultMap" parameterType="com.jack.springbootmybatis.pojo.Student">
    select
    <include refid="Base_Column_List" />
    from student
    <where>
      <choose>
        <when test="stuSex != null ">
            stu_sex = #{stuSex}
        </when> <when test="stuName != null ">
            stu_name = #{stuName}
        </when>

        <otherwise>
          stu_sex = "中"
        </otherwise>
      </choose>
    </where>
  </select>
```



#### trim

(对包含的内容加上 prefix,或者 suffix 等，前缀，后缀),与之对应的属性是prefix和suffix；可以把包含内容的首部某些内容覆盖，即忽略，也可以把尾部的某些内容覆盖，对应的属性是prefixOverrides和suffixOverrides；正因为trim有这样的功能，所以我们也可以非常简单的利用trim来代替where元素的功能

#### set

主要用于更新时，set元素主要是用在更新操作的时候，它的主要**功能和where元素其实是差不多的**，主要是在包含的语句前输出一个set，然后如果包含的语句是以逗号结束的话将会把该逗号忽略，如果set包含的内容为空的话则会出错。有了set元素我们就可以动态的更新那些修改了的字段。

```sql
<update id="dynamicSetTest" parameterType="Blog">
    update t_blog
    <set>
        <if test="title != null">
            title = #{title},
        </if>
        <if test="content != null">
            content = #{content},
        </if>
        <if test="owner != null">
            owner = #{owner}
        </if>
    </set>
    where id = #{id}
</update>
```

### Mybatis映射文件中，如果A标签通过include引用了B标签的内容，请问，B标签能否定义在A标签的后面，还是说必须定义在A标签的前面？

### MyBatis——》转义字符（大于，小于，大于等于，小于等于）

| 符号     | 小于 | 小于等于 | 大于 | 大于等于 | 和    | 单引号 | 双引号 |
| -------- | ---- | -------- | ---- | -------- | ----- | ------ | ------ |
| 原符号   | <    | <=       | >    | >=       | &     | ’      | "      |
| 替换符号 | &lt; | &lt;=    | &gt; | &gt;=    | &amp; | &apos; | &quot; |

![image-20211213161642675](http://oss.geekmice.cn/blog/image-20211213161642675.png)

## 高级查询

### MyBatis实现一对一，一对多有几种方式，怎么操作的？

有联合查询和嵌套查询。联合查询是几个表联合查询，只查询一次，通过在resultMap里面的association，collection节点配置一对一，一对多的类就可以完成

嵌套查询是先查一个表，根据这个表里面的结果的外键id，去再另外一个表里面查询数据，也是通过配置association，collection，但另外一个表的查询通过select节点配置。



## 插件

### 简述Mybatis的插件运行原理，以及如何编写一个插件。

### Mybatis是如何进行分页的？分页插件的原理是什么？

Mybatis 使用 RowBounds 对象进行分页，它是针对 ResultSet 结果集执行的内存分页，而非物理分页。可以在 sql 内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页。

分页插件的基本原理是使用 Mybatis 提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的 sql，然后重写 sql，根据 dialect 方言，添加对应的物理分页语句和物理分页参数。

### mybatis连接池

### mybatis一个接口，一个xml文件，执行SQL语句如何实现

MyBatis 会先解析这些 XML 文件，通过 XML 文件里面的命名空间 （namespace）跟 DAO 建立关系；然后 XML 中的每段 SQL 会有一个id 跟 DAO 中的接口进行关联。
