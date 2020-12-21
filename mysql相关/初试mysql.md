## 操作数据库

### 1.1 操作数据库（了解）

1、创建数据库

```sql
CREATE DATABASE [IF NOT EXISTS] 库名;
```

2、删除数据库

```sql
DROP DATABASE [IF EXISTS] 库名;
```

3、使用数据库

```sql
USE `库名`;
```

4、查看数据库

```sql
SHOW DATABASE --查看所有的数据库！
```



### 1.2数据库的数据类型

> **数值** 

- tinyint 	      十分小的数据      1个字节

- smallint        较小的数据          2个字节

- mediumint  中等大小的数据   3个字节  

- **int**                标准的整数           4个字节  （**常用的**）

- big                较大的数据           8个字节

- float              浮点数                  4个字节

- double          浮点数                  8个字节   （精度问题）

- **decimal**        字符串形式的浮点数 

  

> **字符串 **  

- char                    字符串固定大小              0-255 字符

- **varchar**              可变字符串                     0-65535 字符   （常用的String）       

- tinytext               微型文本                         2^8-1字符

- text                      文本串                            2^16-1字符，最高可存4GB的文本  

  ​     


> **时间日期** 

```java
 java.util.Data
```
- data              YYYY-MM-DD     日期格式

- time              HH:mm:ss          时间格式

- datetime      YYYY-MM-DD HH:mm:ss  最常用的时间格式 

- timestamp   时间戳， 1970.1.1到现在的毫秒数！（全球统一值）

- year               年份表示

  
> **null**
- 没有值，未知

- **注意，不要使用NULL进行运算，结果为NULL**

  

### 1.3 数据库的字段属性（重点！）

**Unsigned**：  

- 无符号的整数。

- 声明了该列不能为负数！  

**zerofill**：

- 0填充的。
- 不足的位数，用0来填充。

**自增**：

- 通常理解为自增，自动在上一条记录的基础上+1（默认）。
- 通常用来设计唯一的主键~index，必须是整数类型。
- 可以自定义设计主键自增的起始值和步长。

**Null 、 not Null**：

- 假设设置为not Null，如果不给它赋值，就会报错！
- Null，如果不填写值，默认就是Null ！

**默认**：

- 设置默认的值！

**扩展**

```sql
# alibaba数据库规范中，规定每一个表，都必须存在以下五个字段！
`id`   			# 主键
`version`		#乐观锁
`is_delete`		#伪删除
`gmt_create`	#创建时间
`gmt_update`	#修改时间
```

 

### 1.4 数据表的类型

```sql
-- 关于数据库引擎
Innnodb   # 默认使用
Myisam    # 早年使用
```

|              | Myisam | Innodb        |
| ------------ | ------ | ------------- |
| 事务支持     | 不支持 | 支持          |
| 数据行锁定   | 不支持 | 支持          |
| 外键约束     | 不支持 | 支持          |
| 全文索引     | 支持   | 不支持        |
| 表空间的大小 | 较小   | 较大，约为2倍 |

常规使用操作：

- myisam : 节约空间，速度较快。
- Innodb：安全性高，事务的处理，多表多用户操作。

> 在物理空间上存储的位置：
>
> **所有的数据都在data路径下**

### 1.5 修改表

> 修改表名：

alter table 旧表名 **rename** as 新表名

> 增加表的字段

alter table 表名 **add** 字段名 列属性

> 修改表的字段 （重命名、修改约束！）

alter table 表名 **modify** 字段名 新列属性   -- **修改约束**  

alter table 表名 **change** 旧字段名 新字段名 列属性  -- **重命名**

> 删除表的字段

alter table 表名**drop** 字段名















 
