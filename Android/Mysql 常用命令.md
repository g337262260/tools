# Mysql 常用命令

## 登录

- 登录Mysql   

   mysql -u -p    

- 显示当前的数据库   

   show databases

- 使用某一数据库   

   use  test1

## 查询

- 查询数据库中有哪些表   

   show tables

- 查看某一个表中的字段 

  show columns from table1

- 查看表中数据的数量

  select count(*) from table1

- 查询表中的数据

  select  id,name.... from table1

- 按照条件查询

  select id,name....from table1 where  age=12

- 多条件查询

  select id,name....from table1 where  age=12 and  name = "xx"

- 查询中重命名列名或表名

  select id,name from table1 as t where t.age = 12

- GROUP BY 语句用于结合合计函数，根据一个或多个列对结果集进行分组

   select identity_type as regway,count(identity_type) as value from t_user group by identity_type

- 用于返回唯一不同的值

   select distinct id,name from table1

## 插入

- 插入数据

  insert into table1 values (value1,value2,......)

- 指定要插入数据的列

  insert into table1 (row1,row2,......) values (value1,value2,......)

## 修改

- 更新某一条数据

  update table1 set  name = 'gg' where age = 17

## 删除  

- delete from table1 where name = 'gg'

##  高级

- like    (not like)

  select * from table1 where name like 'g%'

- in   

  根据条件查表，in  在...之中

  select * from table1 where name in ('liming','jerry') 

- between   

  介于...之间



