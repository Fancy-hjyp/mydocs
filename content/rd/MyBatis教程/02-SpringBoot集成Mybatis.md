---
title: MyBatis 集成 SpringBoot
description: MyBatis 集成 SpringBoot
tags: [MyBatis]
date: 2025-08-03
categories: [MyBatis]
weight: 2
type: docs

---
# Spring Boot 集成 MyBatis 完整教程

上一节我们介绍了MyBatis的核心概念，这节课来讲一下如何集成SpringBoot框架。

## 引入 Starter

Spring Boot 为 MyBatis 提供了官方的 Starter，可以大大简化我们的配置工作。只需在 pom.xml 中添加以下依赖：

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.2</version>
</dependency>
```

同时还需要添加数据库驱动依赖，以 MySQL 为例：

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

## 配置数据源

在 application.yml 中添加数据源配置：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
```

Spring Boot 会自动配置 DataSource、SqlSessionFactory 等 Bean。

## 配置 MyBatis

在 application.yml 中添加 MyBatis 相关配置：

```yaml
mybatis:
  # 配置 mapper.xml 文件的位置
  mapper-locations: classpath:mapper/*.xml
  # 配置实体类别名
  type-aliases-package: com.example.entity
  # 开启驼峰命名转换
  configuration:
    map-underscore-to-camel-case: true
```

## 创建实体类

创建 User 实体类：

```java
package com.example.entity;

public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
    
    // 无参构造函数
    public User() {}
    
    // 全参构造函数
    public User(Long id, String name, Integer age, String email) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.email = email;
    }
    
    // Getter 和 Setter 方法
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public Integer getAge() { return age; }
    public void setAge(Integer age) { this.age = age; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", email='" + email + '\'' +
                '}';
    }
}
```

## 创建 Mapper 接口

创建 UserMapper 接口，使用 @Mapper 注解标记：

```java
package com.example.mapper;

import com.example.entity.User;
import org.apache.ibatis.annotations.*;
import java.util.List;

@Mapper
public interface UserMapper {
    
    @Select("SELECT * FROM user")
    List<User> findAll();
    
    @Select("SELECT * FROM user WHERE id = #{id}")
    User findById(Long id);
    
    @Insert("INSERT INTO user(name, age, email) VALUES(#{name}, #{age}, #{email})")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    int insert(User user);
    
    @Update("UPDATE user SET name = #{name}, age = #{age}, email = #{email} WHERE id = #{id}")
    int update(User user);
    
    @Delete("DELETE FROM user WHERE id = #{id}")
    int deleteById(Long id);
}
```

## 使用 XML 方式配置 SQL（可选）

如果 SQL 比较复杂，建议使用 XML 方式配置。首先修改 Mapper 接口：

```java
package com.example.mapper;

import com.example.entity.User;
import java.util.List;

public interface UserMapper {
    List<User> findAll();
    User findById(Long id);
    int insert(User user);
    int update(User user);
    int deleteById(Long id);
    List<User> findByName(String name);
}
```

然后在 resources/mapper 目录下创建 UserMapper.xml 文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.mapper.UserMapper">
    
    <resultMap id="UserResultMap" type="com.example.entity.User">
        <id property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="age" column="age"/>
        <result property="email" column="email"/>
    </resultMap>
    
    <select id="findAll" resultMap="UserResultMap">
        SELECT id, name, age, email FROM user
    </select>
    
    <select id="findById" resultMap="UserResultMap">
        SELECT id, name, age, email FROM user WHERE id = #{id}
    </select>
    
    <select id="findByName" resultMap="UserResultMap">
        SELECT id, name, age, email FROM user WHERE name LIKE CONCAT('%', #{name}, '%')
    </select>
    
    <insert id="insert" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO user(name, age, email) VALUES(#{name}, #{age}, #{email})
    </insert>
    
    <update id="update">
        UPDATE user SET name = #{name}, age = #{age}, email = #{email} WHERE id = #{id}
    </update>
    
    <delete id="deleteById">
        DELETE FROM user WHERE id = #{id}
    </delete>
    
</mapper>
```

## 创建 Service 层

创建 UserService 接口：

```java
package com.example.service;

import com.example.entity.User;
import java.util.List;

public interface UserService {
    List<User> findAll();
    User findById(Long id);
    boolean save(User user);
    boolean update(User user);
    boolean deleteById(Long id);
    List<User> findByName(String name);
}
```

创建 UserServiceImpl 实现类：

```java
package com.example.service.impl;

import com.example.entity.User;
import com.example.mapper.UserMapper;
import com.example.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserServiceImpl implements UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    @Override
    public List<User> findAll() {
        return userMapper.findAll();
    }
    
    @Override
    public User findById(Long id) {
        return userMapper.findById(id);
    }
    
    @Override
    public boolean save(User user) {
        return userMapper.insert(user) > 0;
    }
    
    @Override
    public boolean update(User user) {
        return userMapper.update(user) > 0;
    }
    
    @Override
    public boolean deleteById(Long id) {
        return userMapper.deleteById(id) > 0;
    }
    
    @Override
    public List<User> findByName(String name) {
        return userMapper.findByName(name);
    }
}
```

## 创建 Controller 层

创建 UserController：

```java
package com.example.controller;

import com.example.entity.User;
import com.example.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping
    public List<User> findAll() {
        return userService.findAll();
    }
    
    @GetMapping("/{id}")
    public User findById(@PathVariable Long id) {
        return userService.findById(id);
    }
    
    @PostMapping
    public String save(@RequestBody User user) {
        boolean result = userService.save(user);
        return result ? "保存成功" : "保存失败";
    }
    
    @PutMapping
    public String update(@RequestBody User user) {
        boolean result = userService.update(user);
        return result ? "更新成功" : "更新失败";
    }
    
    @DeleteMapping("/{id}")
    public String delete(@PathVariable Long id) {
        boolean result = userService.deleteById(id);
        return result ? "删除成功" : "删除失败";
    }
    
    @GetMapping("/search")
    public List<User> search(@RequestParam String name) {
        return userService.findByName(name);
    }
}
```

## 创建主启动类

创建 Spring Boot 主启动类：

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## 数据库表结构

确保数据库中有对应的 user 表：

```sql
CREATE TABLE user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    age INT,
    email VARCHAR(100)
);
```

## 测试接口

启动应用后，可以使用以下接口进行测试：

1. 查询所有用户：
   ```
   GET http://localhost:8080/users
   ```

2. 根据ID查询用户：
   ```
   GET http://localhost:8080/users/1
   ```

3. 添加用户：
   ```
   POST http://localhost:8080/users
   Content-Type: application/json
   
   {
     "name": "张三",
     "age": 25,
     "email": "zhangsan@example.com"
   }
   ```

4. 更新用户：
   ```
   PUT http://localhost:8080/users
   Content-Type: application/json
   
   {
     "id": 1,
     "name": "李四",
     "age": 30,
     "email": "lisi@example.com"
   }
   ```

5. 删除用户：
   ```
   DELETE http://localhost:8080/users/1
   ```

6. 按姓名搜索用户：
   ```
   GET http://localhost:8080/users/search?name=张
   ```

## 总结

通过本教程，我们学习了如何在 Spring Boot 中集成 MyBatis，包括：

1. 添加 MyBatis Starter 依赖
2. 配置数据源和 MyBatis
3. 创建实体类
4. 编写 Mapper 接口（注解方式和 XML 方式）
5. 实现 Service 层
6. 创建 Controller 层
7. 测试 RESTful API

MyBatis 与 Spring Boot 的集成非常简单，Spring Boot 的自动配置机制大大减少了我们的配置工作量。在实际开发中，我们可以根据项目需求选择注解方式或 XML 方式来编写 SQL 语句。