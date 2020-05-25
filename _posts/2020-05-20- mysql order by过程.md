---
layout:     post
title:      MySQL 锁
subtitle:   MySQL 锁
date:       2020-05-13
author:     ZW
header-img: img/mysql_img.jpg
catalog: 	 true
tags:
    - MySQL
---

# 表结构

```
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `city` varchar(16) NOT NULL,
  `name` varchar(16) NOT NULL,
  `age` int(11) NOT NULL,
  `addr` varchar(128) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `city` (`city`)
) ENGINE=InnoDB;
```

```sql
select city,name,age from t where city='杭州' order by name limit 1000 ;
```

# 全字段排序

1. 初始化 sort_buffer，确定放入 name、city、age 这三个字段；
2. 从索引 city 找到第一个满足 city='杭州’条件的主键 id.
3. 到主键 id 索引取出整行，取 name、city、age 三个字段的值，存入 sort_buffer 中；
4. 从索引 city 取下一个记录的主键 id；
5. 重复步骤 3、4 直到 city 的值不满足查询条件为止;
6. 对 sort_buffer 中的数据按照字段 name 做快速排序；
7. 按照排序结果取前 1000 行返回给客户端


步骤 6中 “按 name 排序”这个动作，可能在内存中完成，也可能需要使用外部排序，这取决于排序所需的内存和参数 ***sort_buffer_size***。

外部排序一般使用归并排序算法。可以这么简单理解，MySQL 将需要排序的数据分成 N 份，每一份单独排序后存在这些临时文件中。然后把这 N 个有序文件再合并成一个有序的大文件。 可以通过 ***number_of_tmp_files** 查看使用的临时文件数量。

# rowid排序

如果 MySQL 认为排序的单行长度太大会怎么做呢？ 也就是 [全字段排序]() 中步骤1 `name、city、age` 太长。
 
***max_length_for_sort_data***，是 MySQL 中专门控制用于排序的行数据的长度的一个参数。它的意思是，如果单行的长度超过这个值，MySQL 就认为单行太大，要换一个算法。
 
 
1. 初始化 sort_buffer，确定放入两个字段，即 name 和 id
2. 从索引 city 找到第一个满足 city='杭州’条件的主键 id；
3. 到主键 id 索引取出整行，取 name、id 这两个字段，存入 sort_buffer 中；
4. 从索引 city 取下一个记录的主键 id
5. 重复步骤 3、4 直到不满足 city='杭州’条件为止
6. 对 sort_buffer 中的数据按照字段 name 进行排序
7. 遍历排序结果，取前 1000 行，并按照 id 的值回到原表中取出 city、name 和 age 三个字段返回给客户端


相比 [全字段排序]() ,rowid 在步骤7多一次回表的过程



并不是所有的 order by 语句，都需要排序操作的。从上面分析的执行过程，我们可以看到，MySQL 之所以需要生成临时表，并且在临时表上做排序操作，其原因是原来的数据都是无序的



参考 丁奇 ·《MySQL实战45讲》
    
参考 《高性能MySQL》

