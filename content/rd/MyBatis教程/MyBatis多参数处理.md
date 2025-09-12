---
title: MyBatis 多参数处理
tags: [MyBatis, 输入参数映射, 输出参数映射, SQL 语句编写]
categories: [MyBatis]
date: 2024-08-03 16:05:00
---
# MyBatis 多参数处理详解

## 引言

在 MyBatis 中，当 Mapper 接口的方法需要接收多个参数时，我们需要特别注意如何在 SQL 映射文件中正确引用这些参数。特别是当参数包括基础类型（如 int, String 等）和对象类型（如 User, Product 等）时，处理方式会有所不同。

## 多参数处理的几种方式

### 1. 使用 @Param 注解（推荐）

当方法有多个参数时，最常用和推荐的方式是使用 `@Param` 注解为每个参数命名。

```java
public interface UserMapper {
    List<User> selectUsersByPage(@Param("name") String name, @Param("page") Page page);
}
```

在 Mapper XML 文件中引用参数：

```xml
<select id="selectUsersByPage" resultType="User">
    SELECT * FROM user 
    WHERE name LIKE CONCAT('%', #{name}, '%')
    LIMIT #{page.offset}, #{page.limit}
</select>
```

### 2. 使用 arg0、arg1 等索引方式

如果不使用 @Param 注解，MyBatis 默认会使用 arg0、arg1 或 param1、param2 等作为参数名。

```java
public interface UserMapper {
    List<User> selectUsersByPage(String name, Page page);
}
```

在 Mapper XML 文件中引用参数：

```xml
<select id="selectUsersByPage" resultType="User">
    SELECT * FROM user 
    WHERE name LIKE CONCAT('%', #{arg0}, '%')
    LIMIT #{arg1.offset}, #{arg1.limit}
</select>
```

或者使用 param1、param2：

```xml
<select id="selectUsersByPage" resultType="User">
    SELECT * FROM user 
    WHERE name LIKE CONCAT('%', #{param1}, '%')
    LIMIT #{param2.offset}, #{param2.limit}
</select>
```

### 3. 将参数封装为 Map

可以将多个参数封装到 Map 中传递：

```java
public interface UserMapper {
    List<User> selectUsersByPage(Map<String, Object> params);
}
```

```xml
<select id="selectUsersByPage" resultType="User">
    SELECT * FROM user 
    WHERE name LIKE CONCAT('%', #{name}, '%')
    LIMIT #{page.offset}, #{page.limit}
</select>
```

调用时：

```java
Map<String, Object> params = new HashMap<>();
params.put("name", "John");
params.put("page", page);
userMapper.selectUsersByPage(params);
```

### 4. 创建专门的参数对象（DTO）

将多个参数封装到一个专门的对象中：

```java
public class UserQuery {
    private String name;
    private Page page;
    
    // getter 和 setter 方法
}
```

```java
public interface UserMapper {
    List<User> selectUsersByPage(UserQuery query);
}
```

```xml
<select id="selectUsersByPage" resultType="User">
    SELECT * FROM user 
    WHERE name LIKE CONCAT('%', #{name}, '%')
    LIMIT #{page.offset}, #{page.limit}
</select>
```

## 基础类型和对象类型混合使用的示例

假设我们有一个方法，需要根据用户名（String 类型）和分页信息（Page 对象）来查询用户：

### Mapper 接口

```java
public interface UserMapper {
    List<User> findUsers(@Param("username") String username, @Param("page") Page page);
}
```

### User 实体类

```java
public class User {
    private Long id;
    private String username;
    private String email;
    
    // getter 和 setter 方法
}
```

### Page 分页类

```java
public class Page {
    private int offset;
    private int limit;
    
    public Page(int offset, int limit) {
        this.offset = offset;
        this.limit = limit;
    }
    
    // getter 和 setter 方法
}
```

### Mapper XML 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.mapper.UserMapper">
    
    <select id="findUsers" resultType="com.example.entity.User">
        SELECT id, username, email
        FROM user
        WHERE username LIKE CONCAT('%', #{username}, '%')
        LIMIT #{page.offset}, #{page.limit}
    </select>
    
</mapper>
```

## 注意事项

1. **推荐使用 @Param 注解**：这种方式最清晰，便于理解和维护。

2. **参数引用方式**：
   - 基础类型参数直接使用 `#{参数名}` 引用
   - 对象类型参数使用 `#{参数名.属性名}` 引用其属性

3. **避免参数混淆**：当有多个同类型参数时，尤其要注意使用 @Param 注解明确区分。

4. **对象嵌套属性**：对于对象中的对象属性，可以使用 `#{paramName.objectProperty.subProperty}` 的方式引用。

## 总结

在 MyBatis 中处理多个参数时，最推荐的方式是使用 @Param 注解为每个参数命名，这样在 XML 映射文件中可以清晰地引用每个参数。对于基础类型参数，直接使用 `#{参数名}` 引用；对于对象类型参数，使用 `#{参数名.属性名}` 引用其属性。这种处理方式既清晰又易于维护。