# mysql的MRR？
**MRR** 全称: **Multi-Range Read Optimization**
既是：MRR通过将**随机磁盘读**转化为**顺序磁盘读**，从而提高了索引查询的性能。

> - **为什麽要把随机读转化为顺序读？**
> - **怎麽转化的？**
> - **为什麽顺序读就能提升读取性能？**

## **磁盘** ：苦逼的底层劳动人民  
sql执行一个范围查询：  
```sql
> explain select * from stu where age between 10 and 20;
+----+-------------+-------+-------+------+---------+------+------+-----------------------+
| id | select_type | table | type  | key  | key_len | ref  | rows | Extra                 |
+----+-------------+-------+-------+----------------+------+------+-----------------------+
|  1 | SIMPLE      |  stu  | range | age  | 5       | NULL |  960 | Using index condition |
+----+-------------+-------+-------+----------------+------+------+-----------------------+ 
```
当这个sql被执行时，MYSQL会按照下图的方式，去读取磁盘的数据（假设数据不在数据缓冲池里！）  
![mysql查询检索数据](https://pic3.zhimg.com/80/v2-fb6e74daed175cd327b8d7cdfa2d6dbe_1440w.jpg)  
图中**红色线**就是整个的查询过程，**蓝色线**则是磁盘的运动路线。 

这张图是按照**Myisam**的索引结构画的，不过对于**Innodb**也同样适用。
对于Myisam，左边就是字段age的二级索引，右边是存储完整行数据的地方。

先到左边的二级索引找，找到第一条符合条件的记录（实际上每个节点是一个页，一个页可以有很多条记录，这里我们假设每个页只有一条），接着到右边去读取这条数据的完整记录。

读取完后，回到左边，继续找下一条符合条件的记录，找到后，再到右边读取，这时发现这条数据在物理存储位置上离得很远，需要让磁盘和磁头一起做机械运动，去读取这条数据。

第三条、第四条都是一样，每次读取数据，磁盘和磁头都要进行移动寻址。

为了执行这条sql语句，磁盘要不停的转动，磁头要不停的移动，都是很费时的操作。

> 10,000 RPM（Revolutions Per Minute，即转每分） 的机械硬盘，每秒大概可以执行 167 次磁盘读取，所以在极端情况下，MySQL 每秒只能给你返回 167 条数据，这还不算上 CPU 排队时间。   

对于 **Innodb**，也是一样的。 Innodb 是**聚簇索引（cluster index）**，所以只需要把右边也换成一颗叶子节点带有完整数据的 B+ tree 就可以了。  

## 顺序读：一场狂风暴雨的革命

到这里你知道了磁盘随机访问是多么奢侈的事了，所以，很明显，要把随机访问转化成顺序访问：
```sql
mysql > set optimizer_switch='mrr=on';
Query OK, 0 rows affected (0.06 sec)

mysql > explain select * from stu where age between 10 and 20;
+----+-------------+-------+-------+------+---------+------+------+----------------+
| id | select_type | table | type  | key  | key_len | ref  | rows | Extra          |
+----+-------------+-------+-------+------+---------+------+------+----------------+
|  1 | SIMPLE      | tbl   | range | age  |    5    | NULL |  960 | ...; Using MRR |
+----+-------------+-------+-------+------+---------+------+------+----------------+
```
开启了 **MRR**，重新执行 sql 语句，发现 Extra 里多了一个「Using MRR」。 

这下 MySQL 的查询过程会变成这样： 

![MRR优化](https://pic3.zhimg.com/80/v2-1470b535530f67fad4f265d480e9569e_1440w.jpg)  
对于 **Myisam**，在去磁盘获取完整数据之前，会先按照 **rowid** 排好序，再去顺序的读取磁盘。

对于 **Innodb**，则会按照**聚簇索引键值**排好序，再顺序的读取聚簇索引。

顺序读带来的几个好处:   


-  **磁盘和磁头不再需要来回做机械运动。**
  

-  **可以充分利用磁盘预读。**

    比如在客户端请求一页的数据时，可以把后面几页的数据也一起返回，放到数据缓冲池中，这样如果下次刚好需要下一页的数据，就不再需要到磁盘读取。这样做的理论依据是计算机科学中著名的局部性原理：

    > 当一个数据被用到时，其附近的数据也通常会马上被使用。
- **在一次查询中，每一页的数据只会从磁盘读取一次**  
    
    MySQL 从磁盘读取页的数据后，会把数据放到数据缓冲池，下次如果还用到这个页，就不需要去磁盘读取，直接从内存读。  

    但是如果不排序，可能你在读取了第 1 页的数据后，会去读取第2、3、4页数据，接着你又要去读取第 1 页的数据，这时你发现第 1 页的数据，已经从缓存中被剔除了，于是又得再去磁盘读取第 1 页的数据。

    而转化为顺序读后，你会连续的使用第 1 页的数据，这时候按照 MySQL 的缓存剔除机制，这一页的缓存是不会失效的，直到你利用完这一页的数据，由于是顺序读，在这次查询的余下过程中，你确信不会再用到这一页的数据，可以和这一页数据说告辞了。    
    
**顺序读就是通过这三个方面，最大的优化了索引的读取。**

**别忘了，索引本身就是为了减少磁盘 IO，加快查询，而 MRR，则是把索引减少磁盘 IO 的作用，进一步放大。**