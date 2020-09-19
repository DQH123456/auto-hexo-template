---
title: MySQL小结
date: 2020-05-16 21:47:39
---

>  数据库是在大二下学期学的，到现在也忘记了不少，当时给我印象最深的还是外键的概念，记得当时还花了好长时间去理解，不过现在都忘光了，其实越本质的东西越重要，很多人忽视了概念，光去实践了，其实做什么事情都要实践和思考相结合，才能做好，游戏如此，学习亦是如此，那么 趁着有时间好好总结复习一遍。

<!--more-->
## 前期准备

下载安装, 推荐文章👉[MySQL 8.0.19安装教程(windows 64位)](https://blog.csdn.net/qq_37350706/article/details/81707862)

进入数据库
`mysql -u root -p密码 `
更改密码
`alter user 'root'@'localhost' identified by '新密码';`


常用数据类型

类型 | 说明
--|--
int | 整型，例如 int(11) 
double | 浮点型 
varchar | 变长字符串（char是定长），例如varchar(20) 


## 数据库操作
 
创建数据库
`create database <数据库名>;`
使用数据库
`use <数据库名>;`
查看数据库的中的表(上一步之后)
`show tables;`
查看数据库
`show databases;`
删除
`drop database <数据库名>;`
 

##  表

创建表
`create table test( id int(11) primary key,name varchar(20) not null);`
查看表结构
`desc <表名>`
查看建表语句
`show create table <表名>`
删除表
`drop table <表名>`

## 增删改查
增
`insert into test (id,name) values(1,"中文测试");`
删
`delete from test where id=2;`
改
`update test set name="嘻嘻嘻" where id=1;`
查
`select * from test;`

 



