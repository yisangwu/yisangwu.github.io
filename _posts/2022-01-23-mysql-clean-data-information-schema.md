---
layout: post
title: mysql查看表数据大小，数据清理
tags: linux CentOS mysql
categories: mysql
---


### 对于sql清空表有三种清空方式：

 - 1.delete 是逐行删除速度极慢，不适合大量数据删除
 - 2.truncate  删除所有数据，保留表结构，不能撤消还原
 - 3.drop  删除表，数据和表结构一起删除，快速

 所以，在合理预估数据量级时，尽可能的进行分表（根据Ｓｈａｒｄｉｎｇ　Ｋｅｙ水平分表），便于后续清除表碎片，释放表存储空间。
 


### 1. 查看表占的空间大小：

```mysql
MySQL [(none)]> USE information_schema;

MySQL [information_schema]> SELECT CONCAT(table_schema,'.',table_name) AS 'Table Name', table_rows AS 'Number of Rows', CONCAT(ROUND(data_length/(1024*1024),4),'Mb') AS 'Data Size', CONCAT(ROUND(index_length/(1024*1024),4),'Mb') AS 'Index Size', CONCAT(ROUND((data_length+index_length)/(1024*1024),4),'Mb') AS 'Total'FROM information_schema.TABLES WHERE table_schema LIKE 'table_name';

+-------------------------------------------------+----------------+-----------+------------+---------+
| Table Name                                      | Number of Rows | Data Size | Index Size | Total   |
+-------------------------------------------------+----------------+-----------+------------+---------+
| 表名                                            |    记录行数     |  数据大小  |   索引大小 | 表总大小 |
+-------------------------------------------------+----------------+-----------+------------+---------+
```

### 2. 执行表数据清理：
一般使用脚本，批量多次执行DELETE，sql删除语句类似如下，条件可用自增ID 或者时间。
```mysql
MySQL [database]>DELETE FROM table_name WHERE update_time < "2022-01-23 00:00:00" LIMIT 1000;
```
> 注意，一定要加上条件 和 LIMIT，尤其是超大表。  
> 没有条件限制，在机器性能和硬盘急剧下降时，kill 会一直保持在 query end 耗时很久 


### 3. 执行碎片整理，释放空间：
- 1）当DELETE后面跟条件的时候，则就会出现删除数据后，数据表占用的空间大小不会变。
- 2）不跟条件直接delete的时候，清除了数据，同时数据表的空间也会变为0。


#### 例如：  
· MyISAM引擎表
```mysql
mysql [database]> optimize table 表名;
```
· InnoDB引擎表：
```mysql
mysql [database]> alter table 表名 engine=InnoDB;
```

### 4. 再次查看表占用空间：
```mysql
mysql [database]> show global variables;
```
可以看到 information_schema_stats_expiry 值为 86400。  
此时，可以通过设置set global information_schema_stats_expiry=0来解决，执行会话级的强制刷新，重新再查看，即可看到最新的结果。

