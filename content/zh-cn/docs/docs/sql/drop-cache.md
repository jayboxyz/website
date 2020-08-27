
# DROP CACHE

## 摘要

``` sql
DROP CACHE table WHERE condition;
```

## 说明

`DROP CACHE`仅从协调器删除`table`的缓存元数据。 到期时间自动清除 worker 的缓存 或达到大小限制，但重复进行的拆分将不会重用任何缓存的节点分配。

## 示例

`DROP CACHE` 命令支持通过匹配条件字符串删除特定的缓存记录。例如，如果 `SHOW CACHE` 命令显示有一条名为 `sale_id = 24` 的缓存记录存储于 `sales` 表格下，那么运行下面的命令会将其删除且不影响其他的缓存记录。
```sql
    DROP CACHE sales WHERE sale_id = 24;
```

或者删除 `sales` 表格下存储的所有缓存记录。

```sql 
    DROP CACHE sales;
```

## 限制

目前只有Hive连接器支持此功能。 有关更多详细信息，请参见连接器文档。

## 另请参见

[CACHE TABLE](./cache-table.html)、[SHOW CACHE](./show-cache.html)