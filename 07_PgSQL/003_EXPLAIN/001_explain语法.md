# EXPLAIN

### 语法

```sql
EXPLAIN [ ( option [, ...] ) ] statement
EXPLAIN [ ANALYZE ] [ VERBOSE ] statement
```

常用参数：

```sql
ANALYZE [ boolean ] -- 默认FALSE。通过实际执行SQL获得SQL命令的实际执行计划
VERBOSE [ boolean ] -- 默认FALSE。显示计划的附加信息
COSTS [ boolean ]   -- 默认TRUE。显示每个计划节点的启动成本和总成本，及估计行数和每行宽度。
FORMAT { TEXT | XML | JSON | YAML } -- 默认"TEXT"。指定输出格式。
```

**注意：**

添加`ANALYZE`选项通过实际执行`SQL`来获得命令实际的执行计划，因此可以看到每一步真正耗费了多长时间，以及它实际返回的行数。

如果你不希望`EXPLAIN`影响真正数据，在执行`INSERT`,` UPDATE`,`DELETE`, `MERGE`, `CREATE TABLE AS` 或`EXECUTE`语句时，可以将`EXPLAIN ANALYZE`放到一个事务中，执行完毕后回滚。

命令如下：

```sql
BEGIN;
EXPLAIN ANALYZE ...;
ROLLBACK;
```



此外，`ANALYZE VERBOSE`选项的顺序不能交换：

```sql
explain analyze verbose select * from custom; -- 正确，analyze在verbose前面
explain verbose analyze select * from custom; -- 错误，verbose不能在analyze前面
```



### 几个`explain`语句的例子：

```sql
## 最常用格式一
EXPLAIN select ... ...
## 最常用格式二。会真正执行SQL语句！
EXPLAIN ANALYZE select ... ...
## 指定输出JOSN格式
EXPLAIN (ANALYZE, FORMAT json) select ... ...
## 不输出启动成本，预估行数；输出格式YAML
EXPLAIN (costs false, FORMAT YAML) select... ...
```

