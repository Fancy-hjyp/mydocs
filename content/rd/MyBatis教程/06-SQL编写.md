---
title: SQL语句编写
description: SQL语句编写
tags: [MyBatis]
date: 2025-08-10
weight: 6
---
# 背景
在Mybatis中，SQL编写分为两种：静态SQL和动态SQL。为了简化编写的SQL编写，Mybatis提供了许多动态条件。
这些标签包括：if、where、trim、foreach、set、choose、when、otherwise。
有了这些标签，我们就可以编写更简洁和复用性更强的SQL了。

# if 标签
想象这样一个条件查询场景，如果用户可以穿入用户名或者用户ID查询用户，如果没有动态标签语句，我们需要考虑4种情况：
1. 用户名不为空，用户ID为空
2. 用户名不为空，用户ID不为空
3. 用户名为空，用户ID不为空
4. 用户名为空，用户ID为空
那么相对应地就需要编写4个SQL语句，如果使用动态标签语句，只需要编写一个SQL语句，如下：
mapper接口定义如下：
```java
User selectUserByIdOrName(Integer id, String name);
```

```xml
<select id="selectUserByIdOrName" resultType="com.example.mybatis.entity.User">
    select * from tb_user where
    <if test="id != null">
        id = #{id}
    </if>
    <if test="name != null">
        and name = #{name}
    </if>
</select>
```
这样编写SQL语句后，当name为null后，sql中name字段将不会被拼接。

1. 在if标签中，test属性的值为true时，if标签中的内容将被执行。
2. 在if标签中，test属性的值为false时，if标签中的内容将不会被执行。
3. 在if标签中，test属性的值为null时，if标签中的内容将不会被执行。

# where 标签
上面的if标签，可以保证name字段为null时正确运行，但是当id为null时，就会出现select * from user where and name = '章三'，从语法上来说这是不对的，因为where后面多了个and。为了解决这种情况，可以引入where标签。

若没有匹配到条件，where标签将不会被执行。
当匹配到条件时，where标签会自动加上where标签，同时会自动去除where关键词后面的and或者or。

mapper接口还是如下，可以根据id或者name查询用户。
```java
User selectUserByIdOrName(Integer id, String name);
```

```xml
<select id="selectUserByIdOrName" resultType="com.example.mybatis.entity.User">
    select * from tb_user
    <where>
        <if test="id != null">
            id = #{id}
        </if>
        <if test="name != null">
            and name = #{name}
        </if>
    </where>
</select>
```
当输入的参数name不为null时，SQL语句会变成：
```sql
select * from tb_user where name = ''
```

当输入的参数都为null时，SQL语句会变成：
```sql
select * from tb_user
```

# trim 标签
trim标签用于去除SQL语句中多余的内容，共有四个属性：prefixOverrides、suffixOverrides、prefix、suffix
每个属性的作用如下：
prefixOverrides：去除SQL语句开头的指定内容
suffixOverrides：去除SQL语句末尾的指定内容
prefix：在SQL语句开头添加指定内容
suffix：在SQL语句末尾添加指定内容

假设现在要向数据库中插入数据，但是有些内容可能为空，我们希望为空的数据不插入，那么SQL语句就可以这样写
```xml
<insert id="insertUser" parameterType="com.example.mybatis.entity.User">
    insert into tb_user (
    <if test="name != null">
        name,
    </if>
    <if test="age != null">    
        age
    </if>
    )
    values (
    <if test="name != null">
        name,
    </if>
    <if test="age != null">
        age
    </if>
    )
</insert>
```
可以看到，我们用if标签来动态判断要插入的字段，只有字段不为空时，才会在SQL语句中插入该字段。但是这样会有一个问题，就是如果name字段不为空，而age字段为空，那么最终的SQL语句如下：
```sql
insert into user (`name`) values (name,)
```
SQL语句的末尾中就会多出一个逗号，这样SQL语句就会出错。
为了解决这个问题，可以使用trim标签解决这个问题，trim标签会自动在SQL语句的开始和结尾加上左括号和右括号，同时去除SQL结尾的逗号。
```xml
<insert id="insertUser" parameterType="com.example.mybatis.entity.User">
    insert into tb_user
    <trim prefix="(" suffix=")" suffixOverrides=",">
        <if test="name != null">
            name,
        </if>
        <if test="age != null">
            age,
        </if>
    </trim>
    values
    <trim prefix="(" suffix=")" suffixOverrides=",">
        <if test="name != null">
            name,
        </if>
        <if test="age != null">
            age,
        </if>
    </trim>
</insert>
```

# foreach 标签
foreach 标签用于循环操作列表元素，这在批量操作中很常用。

foreach标签属性：
- collection：表示集合
- item：表示集合中的元素
- index：表示索引
- open：表示循环开始时需要拼接的字符串
- close：表示循环结束时需要拼接的字符串
- separator：表示循环中元素之间的分隔符

下面是一个批量查询的示例，mapper接口定义如下：
```java
List<User> selectUserByIds(List<Integer> ids);
```

```xml
<select id="selectUserByIds" resultType="com.example.mybatis.entity.User">
    select * from tb_user
    <where>
        <if test="list != null and list.size > 0">
            id in
            <foreach collection="list" item="id" open="(" separator="," close=")">
                #{id}
            </foreach>
        </if>
    </where>
</select>
```
在上面的SQL语句中，先使用if标签判断数组是否为空，若不为空，再使用foreach标签遍历数组，将数组中的元素作为参数传入SQL语句中。

值得注意的地方是参数名为list，而不是ids。这是mybatis的参数映射机制，对于数组类型，如果不使用@Param注解指明参数名，则默认为list，除非使用@param注解指定参数名，才可以使用指定的名称，如下所示：
```java
List<User> selectUserByIds(@Param("ids") List<Integer> ids);
```

# set 标签
set标签用于更新时使用，类似于where标签，set标签会自动去除末尾的逗号；
下面是使用set标签更新的例子：
Mapper接口定义如下：
```java
Integer updateUserById(User user);
```
xml文件如下：
```xml
<update id="updateUserById">
    update tb_user
    <set>
        <if test="name != null">
            name = #{name},
        </if>
        <if test="age != null">
            age = #{age},
        </if>
    </set>
    <where>
        id = #{id}
    </where>
</update>
```