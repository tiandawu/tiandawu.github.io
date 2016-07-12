---
title: sql笔记
date: 2016-07-12 12:50:42
tags: 学习笔记
categories: 笔记 
banner: http://i.imgur.com/4veQYiv.png
---
sql语句是数据库很重要的一部分，这篇文章主要记录了一些基本的sql语法，包括对数据库的操作、对表的增删改查、以及数据约束和多表查询等。
<!--more-->

## 一、数据库管理
1. 查询所有数据库  
    `show databases;`
2. 创建数据库  
    `create database test;`
3.  查看数据库的默认字符集  
    `show create database test;`
4.  删除数据库 
    `drop database test;`
5.  修改数据库默认字符集
    `alter database test default character set utf8;`

***
## 二、表管理
1. 查看所有表  
    `show tables;`
2. 创建表  
    `create table student(id int,name varchar(20),age int);`
3. 查看表结构  
    `desc student;`
4. 删除表  
    `drop table student;`
5. 修改表  
    - 添加字段  
        `alter table student add column gender varchar(10);`
    - 删除字段  
        `alter table student drop column gender;`
    - 修改字段类型  
        `alter table student modify column age varchar(10);`
    - 修改字段名称  
        `alter table student change column name sname varchar(20);`
    - 修改表名称  
        `alter table student rename to teacher;`
6. 增删改数据  
    - 增加数据  
        - 插入所有字段  
            `insert into student values(1,'devbird',10);`
        - 插入部分字段  
            `insert into student(id,name) values(2,'dawu')`
    - 修改数据  
        `update student set age=20,gender='男' where id=1;`
    - 删除数据  
        - 带条件的删除  
            `delete from student where id=2;`
    - 删除所有数据
        - 方式一：`delete from student;`
        - 方式二：`truncate table student;`
        - 两种方式区别：
            - delete from: ①可以带条件删除；②只能删除表的数据，不能删除表的约束；③使用delete from删除的数据可以回滚（事务）。
            - truncate table:①不能带条件删除；②既可以删除表的数据，也可以删除表的约束；③使用truncate table删除的数据不能回滚（事务）。
7. 查询数据
    - 查询所有列  
        `select * from student;`
    - 查询指定列  
        `select id,name from student;`
    - 查询时添加常量列class  
        `select id,name,'class' from student;`
    - 查询时合并列  
        `select id name,(math+english) as '总成绩' from student;`
    - 查询时去除重复记录  
        `select distinct gender from student;`
    - 条件查询  
        - 逻辑条件： and(与)  or(或)
            - `select * from student where id=1 and name='devbird';`
            - `select * from student where id=1 or name='devbird';`
        - 比较条件：>, <, >=, <=, =, <>(不等于), between A and B(等价于>=A 且<=B)  
            - `select * from student where math>60;`
            - `select * from student where math>=60 and math<=90;`
            - `select * from student where math between 60 and 90;`
            - `select * from student where age<>20;`
        - 判空条件：is null, is not null, ='', <>''  
            - `select * from student where address is null;`
            - `select * from student where address='';`
        - 模糊条件：like
            - %:表示任意个字符；
            - _：表示一个字符；
            - 查询姓李的学生：`select * from student where name like '李%';`
            - 查询姓李且名字只有两个字的学生：`select * from student where name like '李_';`
        
    - 聚合查询  
        -常用的聚合函数：`sum(), max(), min(), avg(), count()`
        - 查询学生math的总成绩：`select sum(math) as '数学总成绩' from student;`
        - 查询math最高分：`select max(math) as '最高分' from student;`
        - 查询math最低分：`select min(math) as '最低分' from student;`
        - 查询math平均分：`select avg(math) as '数学平均分' from student;`
        - 统计有多少学生：`select count(*) from student;`
    - 分页查询  
        - 分页： limit 其实行，查询几行
        - 分页查询当前页数据的sql：`select * from student limit (当前页-1)*每页显示多少条，每页显示多少条;`
    - 查询排序  
        - 语法：order by 字段 asc/desc  (asc:升序；desc:降序)
        - `select * from student order by math asc;` 
        - `select * from student order by math desc;`
    - 分组查询  
        - 按性别分组：`select * from student group by gender;`
        - 统计每组的人数：`select gender,count(*) from student group by gender;`
    - 分组查询后筛选
        -查询总人数大于二的性别：`select gender,count(*) from student group by gender having count(*)>2;`

***
## 三、数据约束

1. 默认值  
    `create table student(id int,name varchar(20),address varchar(30) default '重庆');`
2. 非空  
    `create table student(id int,name varchar(20),gender varchar(10) not null);`
3. 唯一  
    `create table student(id int unique,name varchar(20));`
4. 主键  
    `create table student(id int primary key,name varchar(20));`
5. 自增长  
    `create table student(id int primary key auto_increment,name varchar(20));`
6. 外键  
    - 主表：`create table dept(id int primary key,deptName varchar(20));`
    - 副表：`create table employee(id int primary key,empName varchar(20),deptId int,constraint employee_dept_fk foreign key(deptId) references dept(id));`
    - 当有了外键约束，添加数据的顺序： 先添加主表，再添加副表数据。
    - 当有了外键约束，修改数据的顺序： 先修改副表，再修改主表数据。
    - 当有了外键约束，删除数据的顺序： 先删除副表，再删除主表数据。
7. 级联操作
    - 级联修改： ON UPDATE CASCADE
    - 级联删除： ON DELETE CASCADE
    - 主表：`create table dept(id int primary key,deptName varchar(20));`
    - 副表：`create table employee(id int primary key,empName varchar(20),deptId int,constraint employee_dept_fk foreign key(deptId) references dept(id) on update cascade on delete cascade);`

***

## 四、多表查询
- **内连接查询**
    - 方式一：`select empName,deptName from employee,dept where employee.deptId=dept.id;`
    - 方式二：`select empName,deptName from employee inner join dept on employee.deptId=dept.id;`
    - 使用别名：`SELECT e.empName,d.deptName FROM employee e INNER JOIN dept d ON e.deptId=d.id;`
- **左[外]连接查询**
    - `SELECT d.deptName,e.empName FROM dept d LEFT OUTER JOIN employee e ON d.id=e.deptId;`
    - 用左边表的数据去匹配右边表的数据，如果符合连接条件的结果则显示，如果不符合连接条件则显示null,左外连接：左表的数据一定会完成显示!
- **右[外]连接查询**
    - `SELECT d.deptName,e.empName FROM employee e FROM employee e RIGHT OUTER JOIN dept d ON d.id=e.deptId;`
    - 使用右边表的数据去匹配左边表的数据，如果符合连接条件的结果则显示，如果不符合连接条件则显示null,右外连接：右表的数据一定会完成显示！
- **自连接查询**
    - `SELECT e.empName,b.empName FROM employee e LEFT OUTER JOIN employee b ON e.bossId=b.id;`

