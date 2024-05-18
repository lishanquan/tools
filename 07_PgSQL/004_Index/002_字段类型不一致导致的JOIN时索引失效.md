### 字段类型不一致导致`JOIN`时索引失效

场景：

`coupon_ticket`表有单独`coupon_id`字段的索引，但是与主表`coupon_info`关联查询时没有使用到索引。

最终排查结果是关联条件`on i.id = t.coupon_id`两侧字段的类型不一致导致的，

因为如果类型不一致，数据库优化器会使用函数（`Join Filter: (i.id = (t.coupon_id)::bpchar)`）进行类型转换，最终导致索引失效。

```sql
explain
select * from coupon_info i join coupon_ticket t on i.id = t.coupon_id
where i.is_delete = false and t.is_delete = false 
order by i.id desc limit 10

QUERY PLAN
Limit  (cost=0.29..10702.30 rows=10 width=1430)
  ->  Nested Loop  (cost=0.29..19895122199.46 rows=18590068 width=1430)
        Join Filter: (i.id = (t.coupon_id)::bpchar)
        ->  Index Scan Backward using coupon_info_pkey on coupon_info i  (cost=0.29..3347.58 rows=16732 width=1063)
              Filter: (NOT is_delete)
        ->  Materialize  (cost=0.00..1991231.83 rows=19842220 width=342)
              ->  Seq Scan on coupon_ticket t  (cost=0.00..1000670.73 rows=19842220 width=342)
                    Filter: (NOT is_delete)
```



**优化方案一（手动进行类型转换）：**

```sql
explain
select * from coupon_info i join coupon_ticket t on cast(i.id as varchar) = t.coupon_id
where i.is_delete = false and t.is_delete = false 
order by i.id desc limit 10

QUERY PLAN
Limit  (cost=0.85..1.18 rows=10 width=1430)
  ->  Nested Loop  (cost=0.85..11033006.97 rows=332000443 width=1430)
        ->  Index Scan Backward using coupon_info_pkey on coupon_info i  (cost=0.29..3347.58 rows=16732 width=1063)
              Filter: (NOT is_delete)
        ->  Index Scan using index_coupon_ticket_coupon_id on coupon_ticket t  (cost=0.56..460.78 rows=19842 width=342)
              Index Cond: ((coupon_id)::text = ((i.id)::character varying)::text)
              Filter: (NOT is_delete)
```



**优化方案二：**

修改`coupon_ticket`表`coupon_id`字段类型跟`coupon_info`表`id`字段类型一致：

```sql
ALTER TABLE coupon_ticket ALTER COLUMN coupon_id TYPE CHAR(24);
```

