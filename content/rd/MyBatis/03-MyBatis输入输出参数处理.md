---
title: MyBatis 输入输出参数处理
description: MyBatis 输入输出参数处理
tags: [MyBatis]
date: 2025-08-03
categories: [MyBatis]
---
# 背景
在上一节我们简单使用 MyBatis 进行了简单查询，但是实际应用中需要考虑以下三点内容：输入参数、输出参数、SQL 语句编写。

- **输入参数映射**：负责将方法参数映射到 SQL 语句中的占位符。
- **输出参数映射**：负责将查询结果映射到方法返回值。
- **SQL 语句编写**：负责定义查询逻辑。

# 输入参数映射
MyBatis 可以通过 `parameterType` 属性指定输入参数的类型，但 `parameterType` 不是必须的，MyBatis 会根据方法参数自动推断出 `parameterType`。
输入参数可以是一个或多个参数，每个参数可以是简单类型、对象类型、数组、`Map` 等。针对不同的情况，处理方式如下：

针对输入参数为简单类型（如 `Integer`、`String`、`Boolean`），可以直接在 Mapper 接口中定义方法参数，然后在 XML 文件中根据参数名使用即可。

**示例**：
Mapper 接口定义：
```java
List<User> selectUserByAge(Integer age);
那么在Mapper.xml文件中，就可以直接使用`#{age}`来引用这个参数。
```xml
<select id="selectUserByAge" resultType="User">
SELECT * FROM user WHERE age = #{age}
</select>
```
说明
* MyBatis 会利用反射机制解析方法参数，并将参数值传递给 SQL 语句中的占位符。
* resultType 是必须的，用于指定查询结果的映射类型。

## 输入参数为对象类型
如果输入参数是对象类型，可以在 SQL 语句中的占位符中使用 `#{对象属性名}` 来引用对象属性的值。
假设定义了一个User对象，包含id、name、age三个属性。如果想要在sql中使用User对象的id属性，就需要在Mapper.xml文件中使用`#{id}`来引用该属性值。
Mapper接口定义如下：
```java
User selectUserByUser(User user);
```
在这个例子中，我们定义了一个方法selectUserByUser，该方法接收一个User对象，返回一个User对象。

接下来想要通过user对象的id属性查询用户信息，那么可以在Mapper.xml文件中定义如下的SQL语句：
```xml
<select id="selectUserByUser" resultType="com.example.mybatis.entity.User">
    select * from tb_user where id = #{id}
</select>
```
当方法只有一个参数时，MyBatis 会将该参数作为根对象，因此可以直接使用 #{id}。
如果有多个参数，则需要使用 #{参数名.属性名} 的方式引用对象属性。
注意到在使用id属性时并没有使用#{user.id}的方式，这是因为当入参只有一个参数时，MyBatis会自动将该参数作为根对象，因此可以直接使用#{id}来引用User对象的id属性。

但是当输入参数是多个时，就需要使用`#{参数名.属性名}`的方式来引用参数的属性。
假设定义一个方法selectUserByIdAndUser，该方法接收id和User对象，返回一个User对象。
```java
User selectUserByIdAndUser(Integer id, User user);
```
那么在Mapper.xml文件中，定义SQL语句时，不可以直接使用#{name}，因为MyBatis无法确定是使用哪个参数的name属性。而是需要使用#{参数名.属性名}的方式。
```xml
<select id="selectUserByIdAndUser" resultType="com.example.mybatis.entity.User">
    select * from tb_user where id = #{id} and name = #{user.name}
</select>
```

## 输入参数有多个
如果输入参数有多个，MyBatis 会自动将参数封装成 `Map` 对象，参数名作为 key，参数值作为 value。此时可以直接使用#{参数名}，MyBatis会自动将参数封装成Map对象，并使用参数名作为key，参数值作为value。
假设Mapper接口定义如下：
```java
User selectUserByIdAndName(Integer id, String name);
```
那么在Mapper.xml文件中，就可以直接使用#{id}和#{name}来引用这两个参数。
```xml
<select id="selectUserByIdAndName" resultType="com.example.mybatis.entity.User">
    select * from tb_user where id = #{id} and name = #{name}
</select>
```

## 输入参数取别名
当然如果想要为参数取一个别名，可以使用@Param注解。下面的例子中，我们为id参数取了一个别名userId，
```java
User selectUserByIdAndUser(@Param("userId") Integer id, User user);
```
那么在SQL语句中就可以直接使用#{userId}来引用id参数。
```xml
<select id="selectUserByIdAndUser" resultType="com.example.mybatis.entity.User">
    select * from tb_user where id = #{userId} and name = #{user.name}
</select>
```

## 输入参数类型与数据库字段类型不一致
假设现在 User 对象的 `age` 属性是 `String` 类型，而数据库中的 `age` 字段是 `INTEGER` 类型。

MyBatis 不会对参数类型进行转换，而是将参数值直接传递给数据库。数据库会尝试将参数值转换为字段的类型（如果可能）。

**示例**：
Mapper接口定义如下：
```java
List<User> selectUserByAgeString(String age);
```
其中，数据库中age字段是Integer类型，而方法参数age是String类型。
在Mapper.xml文件中，可以直接使用#{age}来引用方法参数age。
```xml
<select id="selectUserByAgeString" resultType="com.example.mybatis.entity.User">
    SELECT * FROM user WHERE age = #{age}
    </select>
```
说明：
* 如果传入的参数是 "20"，数据库会将其转换为 INTEGER 并执行查询。
* 如果传入的参数是 "abc"，数据库会尝试将其转换为 INTEGER，但会失败并抛出错误。
* 为了避免潜在的错误，建议在 Java 层进行显式的类型转换。
并不是所有的类型mysql都会自动转换，string与int之间自动转换的前提是string是数字字符串。
mybatis会自动将参数类型转换为数据库字段类型，但是前提是数据库字段类型和参数类型之间存在可转换关系。