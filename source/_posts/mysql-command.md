---
title: SQL 语句分类
date: 2020-05-19 08:31:31
tags: MySQL
---

> 本文基于 MySQL 5.7 (10.1.9-MariaDB)

<!-- more -->



### 一. 数据定义语言 DDL (Data Definition Language)

1. 对象： 数据库和表

2. 关键词： create alter drop truncate
	- 创建数据库：create database school;
	- 删除数据库：drop database school;
	- 切换数据库：use school;
	- 创建表：create table student
	- 查看数据库里存在的表：show tables;
	- 修改表：alter table student rename (to) teacher;
	- 删除表：drop table student;
	- 查看生成表的sql语句：show create table student;
	- 查看表结构：desc student;
	- 删除当前表再新建一个一模一样的表结构：truncate


### 二. 数据操纵语言 DML (Data Manipulation Language)　　　

1. 对象：纪录(行)

2. 关键词：insert update delete

	- 插入：insert into student(id,name) values(01,'tonbby');

	- 更新：update student set name = 'tonbby',score = '99' where id = 01;

	- 删除：delete from tonbby where id = 01;
	truncate是删除表，再重新创建这个表，属于DDL；delete是一条一条删除表中的数据，属于DML。

 

### 三. 数据查询语言 DQL (Data Query Language)
1. 对象：纪录(行)

2. 关键词：select
	- 查询语句： select ... from student where 条件 group by 分组字段 having 条件 order by 排序字段

 

### 四. 数据控制语言 DCL (Data Control Language)  

1. 对象：数据库和表，用户

2. 关键字：grant revoke
	- 创建用户：CREATE USER 
	- 删除用户：DROP USER 
	- 授权用户访问指定表: grant
	- 回收用户方位全息：revoke
