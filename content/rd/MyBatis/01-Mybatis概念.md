---
title: MyBatis 概念
description: MyBatis 概念详解
tags: [MyBatis]
date: 2025-08-03
categories: [MyBatis]
weight: 1
type: docs
---
# MyBatis 概念详解

## 什么是 MyBatis？

MyBatis 是一个优秀的持久层框架，它支持普通的 SQL 查询、存储过程和高级映射。MyBatis 消除了几乎所有的 JDBC 代码和参数的手工设置以及结果集的检索。MyBatis 使用简单的 XML 或注解用于配置和原始映射，将接口和 Java 的 POJOs(Plain Old Java Objects)映射成数据库中的记录。

简单来说，MyBatis 就是一个用来简化数据库操作的框架，它封装了 JDBC 操作，让开发者能够更专注于业务逻辑而不是底层的数据访问细节。

## MyBatis 的核心特性

### 1. SQL 与代码分离
- SQL 语句可以写在 XML 文件中，实现 SQL 与代码的分离
- 便于统一管理和维护 SQL 语句
- DBA 可以直接查看和优化 SQL 语句

### 2. 简化数据库操作
- 自动完成结果集与 Java 对象的映射
- 简化了参数设置和结果处理过程
- 减少了大量重复的 JDBC 代码

### 3. 灵活的映射机制
- 支持对象与数据库字段的自动映射
- 支持复杂的数据类型映射
- 提供了强大的动态 SQL 功能

### 4. 缓存机制
- 一级缓存（SqlSession 级别）默认开启
- 支持二级缓存（Mapper 级别）配置

## MyBatis 的核心组件

### SqlSessionFactoryBuilder
该类会根据配置信息生成 SqlSessionFactory 对象。它可以从 XML 配置文件或一个预先配置的 Configuration 实例来构建 SqlSessionFactory。

### SqlSessionFactory
这是创建 SqlSession 的工厂类，是线程安全的，一旦被创建，在整个应用执行期间都会存在。通常一个应用只需要一个 SqlSessionFactory。

### SqlSession
这是 MyBatis 的核心对象，提供了在数据库执行 SQL 命令所需的所有方法。它包含了以数据库为背景的所有执行语句、提交或回滚事务、获取映射器实例等方法。

### Mapper 接口
Mapper 是 MyBatis 中用来定义 SQL 操作的接口。MyBatis 会根据这些接口的定义自动生成实现类，并执行对应的 SQL 语句。

## MyBatis 的工作流程

1. **读取配置文件**：MyBatis 首先读取全局配置文件 mybatis-config.xml
2. **加载映射文件**：加载具体的 SQL 映射文件 Mapper.xml
3. **构建会话工厂**：根据配置信息构建 SqlSessionFactory
4. **创建数据会话**：通过 SqlSessionFactory 创建 SqlSession
5. **执行 SQL 操作**：通过 SqlSession 执行 SQL 并处理结果
6. **关闭会话**：操作完成后关闭 SqlSession

## MyBatis 的两种配置方式

### XML 配置方式
通过 XML 文件来配置 SQL 语句和映射关系，是最常用的方式。

### 注解配置方式
通过在接口方法上添加注解来配置 SQL 语句，适用于简单的 SQL 操作。

## MyBatis 与传统 JDBC 和 Hibernate 的比较

### 与 JDBC 相比
- 简化了数据库操作代码
- 减少了资源管理和异常处理的代码
- 提供了 SQL 与代码的分离机制

### 与 Hibernate 相比
- 更加轻量级，学习成本低
- SQL 更加灵活可控
- 对复杂查询支持更好
- 性能调优更加直接

## MyBatis 的适用场景

- 对 SQL 性能有较高要求的项目
- 需要执行复杂 SQL 查询的应用
- 对数据库操作有精细控制需求的系统
- 团队中有经验丰富的 SQL 开发人员

## 总结

MyBatis 是一个半自动化的 ORM 框架，它在保持 SQL 灵活性的同时，简化了数据库访问的编码工作。它不像 Hibernate 那样完全自动化，而是让开发者在需要的时候可以精确控制 SQL，是一个在 Java 企业级开发中广泛使用的持久层解决方案。