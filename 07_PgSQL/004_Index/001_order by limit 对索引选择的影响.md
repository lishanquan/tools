### order by limit 索引选择问题

当`SQL`查询中包括排序，以及其他字段的过滤条件，并使用`LIMIT`快速返回少量数据时，

**如果满足条件的数据分布在排序键的末端**，那么优化器给出的执行计划可能是不好的，

导致通过排序索引扫描更多的数据后才能命中需要的记录。

然而，数据库目前使用的评估走排序键时，`LIMIT`需要扫描多少条记录，使用了数据均匀分布的假设，所以在数据（满足条件的数据与排序键本身的相关性不均匀）分布不均匀时，导致成本估算不准(oracle干脆走全表扫描)。



**建议优化方法**：

增加索引，创建***等值查询条件列(s)***加***排序列(s)***组成的复合索引，降低扫描量。

示例1：

```sql
select * from tbl where c1=200 and c2 between 100 and 300 order by id limit 10;  
  
增加索引  
  
(c1,id)  -- 索引扫描, filter c2  
  
已有  
(c1,c2)  -- 索引扫描, sort id  
(id)     -- 索引扫描, filter c1,c2  
```

示例2：

```sql
select * from tbl where c1=200 and c2 =200 order by id limit 10;  
  
增加索引  
  
(c1,c2,id)  -- 索引扫描  
  
已有  
(c1,c2)  -- 索引扫描, sort id  
(id)     -- 索引扫描, filter c1,c2  
```



**另外一种处理思路（`force index() 方法`）**：

给排序字段加函数，相当于放弃排序字段的索引，强制走等值查询条件的索引。

示例1：

```sql
--原有写法
SELECT * FROM user u where u.id=100 order by u.update_time

--优化方法一：
--原有索引：idx_user_id(id)
--新建索引：idx_user_id_update_time(id,update_time)

--优化方法二（force index）：
SELECT * FROM user u force index(idx_user_id_update_time) where u.id=100 order by u.update_time
```

示例2：

```sql
--原有写法
ORDER BY create_date DESC 
LIMIT 10 OFFSET 0;

--优化写法，使用函数相当于 force index() 方法的变形
ORDER BY to_char(create_date,'yyyy-mm-dd hh24:mi:ss:ms') DESC 
LIMIT 10 OFFSET 0;
```

