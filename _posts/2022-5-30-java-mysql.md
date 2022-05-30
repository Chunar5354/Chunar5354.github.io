---
layout: article
title: Java访问MySQL
tags: Java Spring
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

在Java语言中操作MySQL数据库，主要包括JDBC以及MyBatis框架，以及它们在Spring Boot中的应用

<!--more-->

## JDBC

### 通过驱动包的方式使用依赖

#### 下载驱动

在Java中访问mysql需要借助`JDBC`实现，并且需要首先安装驱动包

安装包可以在[这里](https://dev.mysql.com/downloads/connector/j/)下载，注意要在下拉菜单中选择`Platform Independent`

下载后解压文件得到`.jar`文件

#### 加载驱动

- 在vscode中的加载方式

首先需要安装一个Java的插件集合`Project Manager for Java`

安装之后点击左下角`JAVA PROJECT`中的`Referenced Libraries`右侧的加号，将上面的jar文件加载进来

[![Ob9huD.png](https://s1.ax1x.com/2022/05/19/Ob9huD.png)](https://imgtu.com/i/Ob9huD)

- 在IDEA中的加载方式

`File` -> `Project Structure` -> `Modules` -> `点击加号` -> `JARs or Directories` -> 选择路径导入本地jar包

[![XukQKO.png](https://s1.ax1x.com/2022/05/28/XukQKO.png)](https://imgtu.com/i/XukQKO)

### 通过maven管理依赖

再IDEA中，编写pom.xml：

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

### 编写代码

JDBC使用的基本流程：

- 1.实例化驱动

```java
Class.forName("com.mysql.cj.jdbc.Driver");
```

- 2.创建连接

```java
// 需要数据库服务端的信息
// url解释为："jdbc:数据库类型:IP:端口/数据库?是否使用SSL & 是否允许客户端从服务器获取公钥 & 时区 & 编码方式（中文不乱码）"
static final String DB_URL = "jdbc:mysql://hostname:3306/lottery?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC&characterEncoding=UTF-8";
static final String USER = "user";
static final String PASS = "password";

conn = DriverManager.getConnection(DB_URL, USER, PASS);
stmt = conn.createStatement();  // 连接实例化
```

- 3.执行sql语句

```java
sql = "SELECT * FROM table.";
ResultSet rs = stmt.executeQuery(sql);
while (rs.next()) { // 可能有多行结果
    // 通过字段名获取数据
    int id  = rs.getInt("id");
    String name = rs.getString("name");
}
```

- 4.关闭连接

```java
// 从后往前
rs.close();
stmt.close();
conn.close();
```

示例：

```java
import java.sql.*;
 
public class javaSQL {
    // MySQL 8.0 以上版本 - JDBC 驱动名及数据库 URL
    static final String JDBC_DRIVER = "com.mysql.cj.jdbc.Driver";  
    static final String DB_URL = "jdbc:mysql://hostname:3306/lottery?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC";
 
    // 数据库的用户名与密码，需要根据自己的设置
    static final String USER = "user";
    static final String PASS = "password";
 
    public static void main(String[] args) {
        Connection conn = null;
        Statement stmt = null;
        try {
            // 注册 JDBC 驱动
            Class.forName(JDBC_DRIVER);  // forName方法用于获取将指定的类实例化
        
            // 打开链接
            System.out.println("连接数据库...");
            conn = DriverManager.getConnection(DB_URL, USER, PASS);
        
            // 执行查询
            System.out.println(" 实例化Statement对象...");
            stmt = conn.createStatement();
            String sql;
            sql = "SELECT * FROM lottery.Goods";
            ResultSet rs = stmt.executeQuery(sql);
        
            // 展开结果集数据库
            while (rs.next()) {
                // 通过字段检索
                int id  = rs.getInt("id");
                String name = rs.getString("price");
                String url = rs.getString("goodsName");
    
                // 输出数据
                System.out.print("id: " + id);
                System.out.print(", price: " + name);
                System.out.print(", goodsName: " + url);
                System.out.print("\n");
            }
            // 完成后关闭
            rs.close();
            stmt.close();
            conn.close();
        } catch(SQLException se) {
            // 处理 JDBC 错误
            se.printStackTrace();
        } catch(Exception e) {
            // 处理 Class.forName 错误
            e.printStackTrace();
        } finally {
            // 关闭资源
            try {
                if(stmt!=null) stmt.close();
            } catch(SQLException se2) {
            }// 什么都不做
            try {
                if(conn!=null) conn.close();
            } catch(SQLException se) {
                se.printStackTrace();
            }
        }
        System.out.println("Goodbye!");
    }
}
```

## MyBatis

MyBatis是一个Java中更高级的SQL框架，它简化了JDBC的一些操作

详细内容可以查看[官网](https://mybatis.org/mybatis-3/zh/index.html)

在Spring Boot中使用Mybatis有几种不同的方式，下面分别介绍：

### 1.手动创建service

#### 创建测试表

在NySQL中创建测试的表，并添加一条数据：

```sql
CREATE TABLE Goods (
    id INT NOT NULL PRIMARY KEY,
    goodsName VARCHAR(200),
    price INT,
    imgList VARCHAR(200)
) ENGINE InnoDB;

INSERT INTO lottery.Goods(id, goodsName, price, imgList) VALUES(1, "'g1'", 234, "'i1/i2'")
```

#### 配置依赖

编写`pom.xml`

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.2.2</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

mybatis也需要借助jdbc与mysql connector来实现数据库访问

而其中的`lombok`是一个可以通过注解简化Java代码的工具，关于它的使用可以参考[这里](https://kucw.github.io/blog/2020/3/java-lombok/)

#### 配置数据库信息

在`resources/application.properties`中添加：

```
spring.datasource.url=jdbc:mysql://1.15.140.88:3306/lottery?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC&characterEncoding=UTF-8
spring.datasource.username=Chunar
spring.datasource.password=chun.0927
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

server.port=12090
```

#### 编写实体类

实体类对应数据库中的表

```java
package com.example.mybatisdemo.entity;

import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.List;

@Data
@NoArgsConstructor
public class Goods{
    private Long id;
    private String goodsName;
    private Integer price;
    private String imgList;

    public Goods(Long id, String goodsName, Integer price, String imgList) {
        this.id = id;
        this.goodsName = goodsName;
        this.price = price;
        this.imgList = imgList;
    }
}
```

#### 编写service

service是针对实体要实现的方法，包括`接口`和`接口的实现类`

- 1.接口

```java
package com.example.mybatisdemo.service;

import com.example.mybatisdemo.entity.Goods;

import java.util.List;

public interface GoodsServiceInterface {
    public Goods getById(Long id); // 通过id查找单个记录
    public List<Goods> getByName(String goodsName); // 通过name查找多条记录
    public Integer insert(Long id, String goodsName, Integer price, String imgList); // 插入数据
}
```

- 2.接口的实现类

```java
package com.example.mybatisdemo.service;

import com.example.mybatisdemo.entity.Goods;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.stereotype.Service;

import java.sql.ResultSet;
import java.util.List;


@Service
public class GoodsService implements GoodsServiceInterface {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    private RowMapper<Goods> GoodsMappper() {
        RowMapper<Goods> goodsRowMapper = (ResultSet rs, int rownum) -> {
            System.out.println(rs);
            Goods goods = new Goods();
            goods.setId(rs.getLong("id"));
            goods.setGoodsName(rs.getString("goodsName"));
            goods.setPrice(rs.getInt("price"));
            goods.setImgList(rs.getString("imgList"));
            return goods;
        };
        return goodsRowMapper;
    }

    @Override
    public Goods getById(Long id) {
        String sql = "select * from Goods where id = ?";
        Object[] params = new Object[]{id};
        Goods goods = jdbcTemplate.queryForObject(sql, params, GoodsMappper());
        return goods;
    }

    @Override
    public List<Goods> getByName(String goodsName) {
        String sql = "select * from Goods where goodsName = ?";
        Object[] params = new Object[]{goodsName};
        List<Goods> goods = jdbcTemplate.query(sql, params, GoodsMappper());
        return goods;
    }

    @Override
    public Integer insert(Long id, String goodsName, Integer price, String imgList) {
        String sql = "insert into Goods values(?, ?, ?, ?)";
        int res = jdbcTemplate.update(sql, id, goodsName, price, imgList);
        return res;
    }
}
```

#### 编写controller

controller层的目的是对service层进行控制，比如在这里使用`@Autowired`注入service并调用service中定义的方法`getById()`

```java
package com.example.mybatisdemo.controller;

import com.example.mybatisdemo.entity.Goods;
import com.example.mybatisdemo.service.GoodsService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.List;

@Controller
@RequestMapping("/test")
public class TestController {
    @Autowired
    GoodsService goodsService;

    @RequestMapping("")
    @ResponseBody
    public String test() {
        goodsService.insert(1L, "g1", 123, "i1/i2");
        goodsService.insert(2L, "g1", 456, "i3/i4");
        List<Goods> goodsList = goodsService.getByName("g1");
        if (goodsList != null) {
            System.out.println(goodsList);
            return "Get Succeed";
        } else {
            System.out.println("get Failed");
            return "Get Failed";
        }
    }
}
```

#### 运行测试

main函数不用改变，直接运行项目，在浏览器中输入`localhost:12090/test`就可以看到结果

#### 关于Mapper

在service中使用`jdbcTemplate`执行sql语句时用到了`GoodsMappper`，它是一个返回值为`RowMapper<Goods>`类型的函数（RowMapper是一个接口）

传入GoodsMappper的意义是可以将sql语句执行的查询结果`映射`到具体的对象上

GoodsMappper的用法光看代码可能会有一些迷惑，所以详细说明一下：

在官方文档中可以看到[query的用法](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html#queryForObject-java.lang.String-java.lang.Object:A-org.springframework.jdbc.core.RowMapper-)

在上面的例子中使用了lambda表达式，这也是RowMapper用法的一种，具体可以查看其[文档](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/core/RowMapper.html)

其实除了使用lambda表达式，还有另一种实现方式：

- 1.在service中编写一个`GoodsMapper`类

```java
public class GoodsMapper implements RowMapper<Goods> {
    public Goods mapRow(ResultSet rs, int rownum) throws SQLException {
        Goods goods = new Goods();
        goods.setId(rs.getLong("id"));
        goods.setGoodsName(rs.getString("goodsName"));
        goods.setPrice(rs.getInt("price"));
        goods.setImgList(rs.getString("imgList"));
        return goods;
    }
}
```

- 2.修改`GoodsService`

```java
@Service
public class GoodsService implements GoodsServiceInterface {
    @Autowired
    private JdbcTemplate jdbcTemplate;


    @Override
    public Goods getById(Long id) {
        String sql = "select * from Goods where id = ?";
        Object[] params = new Object[]{id};
        // 注意这里传参的方式变成了 new GoodsMapper()
        Goods goods = jdbcTemplate.queryForObject(sql, params, new GoodsMapper());
        return goods;
    }

    @Override
    public List<Goods> getByName(String goodsName) {
        String sql = "select * from Goods where goodsName = ?";
        Object[] params = new Object[]{goodsName};
        // 注意这里传参的方式变成了 new GoodsMapper()
        List<Goods> goods = jdbcTemplate.query(sql, params, new GoodsMapper());
        return goods;
    }

    @Override
    public Integer insert(Long id, String goodsName, Integer price, String imgList) {
        String sql = "insert into Goods values(?, ?, ?, ?)";
        int res = jdbcTemplate.update(sql, id, goodsName, price, imgList);
        return res;
    }
}
```

与使用lambda表达式的方式有相同的功能

### 2.通过注解

配置与实体类不变，在使用注解的方法中，不需要使用service，而是通过mapper代替

#### 编写mapper

```java
package com.example.mybatisxml.mapper;

import com.example.mybatisxml.entity.Goods;
import org.apache.ibatis.annotations.*;

import java.util.List;

@Mapper
public interface GoodsMapper {
    @Select("select * from Goods")
    List<Goods> getAll();

    @Select("select * from Goods where id=#{id}")
    Goods getById(@Param("id") Long id);

    @Insert("insert into Goods(id, goodsName, price, imgList) values(#{id}, #{goodsName}, #{price}, #{imgList})")
    int insert(@Param("id") Long id, @Param("goodsName") String goodsName, @Param("price") Integer price, @Param("imgList") String imgList);

    @Delete("delete from Goods where id < #{num}")
    int delete(@Param("num") Integer num);

    @Update("update Goods set goodsName=#{newName} where id=#{num}")
    int update(@Param("newName") String newName, @Param("num") Integer num);
}
```

使用`@Mapper`注解的接口中可以定义增删查改的方法，分别使用`@Select`等注解，在函数中使用`@Param`注解来处理传参，参数与sql语句中的`#{}`部分对应

#### 编写controller

controller与上面的类似

```java
package com.example.mybatisxml.controller;

import com.alibaba.fastjson.JSONObject;
import com.example.mybatisxml.entity.Goods;
import com.example.mybatisxml.mapper.GoodsMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@EnableTransactionManagement
@RequestMapping("/test")
public class GoodsController {
    // 注入mapper
    @Autowired
    private GoodsMapper goodsMapper;

    @RequestMapping("/getall")
    @ResponseBody
    public JSONObject getAll() {
        List<Goods> goodsList = goodsMapper.getAll();
        JSONObject jsonRes =new JSONObject();
        for (int i = 0; i < goodsList.size(); i++) {
            jsonRes.put(Integer.toString(i), goodsList.get(i));
        }
        return jsonRes;
    }

    @RequestMapping("/getbyid")
    @ResponseBody
    public JSONObject getById() {
        Goods goods = goodsMapper.getById(1L);
        JSONObject jsonRes =new JSONObject();
        jsonRes.put("data", goods);
        return jsonRes;
    }

    @RequestMapping("/insert")
    public int insert() {
        goodsMapper.insert(1L, "g1", 123, "i1/i2");
        int num = goodsMapper.insert(2L, "g1", 456, "i3/i4");
        return num;
    }

    @RequestMapping("/update")
    public int update() {
        int num = goodsMapper.update("g2", 2);
        return num;
    }

    @RequestMapping("/delete")
    public int delete() {
        int num = goodsMapper.delete(3);
        return num;
    }
}
```

通过在浏览器输入相应的地址可以看到结果

注意在select方法的返回中使用了json，它是需要配置依赖的：

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.40</version>
</dependency>
```

### 3.通过xml配置

配置和实体部分仍然相同

#### 编写mapper

在mapper中不再使用注解，而是在xml配置文件中来进行sql语句的设置

```java
package com.example.mybatisxml.mapper;

import com.example.mybatisxml.entity.Goods;
import org.apache.ibatis.annotations.*;

import java.util.List;

@Mapper
public interface GoodsMapper {
    List<Goods> getAll();

    Goods getById(@Param("id") Long id);

    int insert(@Param("id") Long id, @Param("goodsName") String goodsName, @Param("price") Integer price, @Param("imgList") String imgList);

    int delete(@Param("num") Integer num);

    int update(@Param("newName") String newName, @Param("num") Integer num);
}
```

#### 编写xml配置文件

在`resources`下新建`mapper/GoodsMapper.xml`文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.mybatisxml.mapper.GoodsMapper">
    <!-- 自定义要查询的列 -->
    <sql id="Base_Column_List">id, goodsName</sql>
    <!-- 查询多条数据，在后端返回的是list类型，但在这里resultType中仍然要写实体的类型 -->
    <select id="getAll" resultType="com.example.mybatisxml.entity.Goods">
        select
            <include refid="Base_Column_List"/>
        from Goods
    </select>

    <select id="getById" resultType="com.example.mybatisxml.entity.Goods" parameterType="long">
        select * from Goods where id=#{id}
    </select>

    <insert id="insert">
        insert into Goods(id, goodsName, price, imgList) values(#{id}, #{goodsName}, #{price}, #{imgList})
    </insert>

    <update id="update">
        update Goods set goodsName=#{newName} where id=#{num}
    </update>

    <delete id="delete">
        delete from Goods where id&lt;#{num}
    </delete>
</mapper>
```

几个要点：

- 1.在mapper标签中通过`namespace`属性和`GoodsMapper`接口关联起来

- 2.如果不需要查询所有字段可以通过`<sql id="Base_Column_List">id, goodsName</sql>`自定义需要查询的列

- 3.如果查询可能返回多个结果，在`resultType`中也要填写实体类

- 4.xml不支持大于号小于号等符号，需要用转义符替换，比如在上面的`delete`中，就使用了`&lt;`来替换`<`，关于转义字符可以查看[这里](https://cloud.tencent.com/developer/article/1345009)

- 5.需要将xml配置添加到项目的设置中，在`resources/application.properties`中添加：

```
mybatis.mapper-locations=classpath:mapper/*.xml
```

#### controller

controller的编写与使用注解的方式相同

同样可以在浏览器中进行测试
