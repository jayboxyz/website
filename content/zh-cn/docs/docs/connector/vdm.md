
# VDM连接器

有一些用例需要在同一个会话或视图中管理和访问多个数据源。此外，用户可能根本不关心数据的分布和来源。虚拟数据市场（VDM）连接器旨在将该特性引入openLooKeng。

VDM连接器支持：

- 创建、更新和删除组合多个目录的视图
- 通过视图访问真实数据
- 通过视图管理用户权限
- 记录每个用户使用VDM视图的情况

## 使用

VDM使用openLooKeng元存储存储其数据库信息。信息可以存储在HDFS或关系数据库中，这取决于openLooKeng元存储的实现。

因此必须先配置元存储。下面是使用RDBMS作为元存储的示例，创建`etc/hetu-metastore.properties`：

    hetu.metastore.type=jdbc
    hetu.meatstore.db.url=jdbc:mysql://....
    hetu.metastore.db.user=root
    hetu.metastore.db.password=123456

对于用户界面，可以从JDBC或命令行界面访问连接器。当前VDM仅支持模式和视图。不支持表。

模式操作与常用的openLooKeng目录相同，包括`create schema`、`drop schema`、`rename schema`和`show schemas`。

视图可以创建在特定的模式下：`create view as ...`，`drop view`。

## 使用示例：

配置数据源`vdm1`，在`etc/catalogs`中创建`vdm1.properties`，内容如下：

    connector.name=vdm

此示例在`vdm1`目录中创建模式`schema1`，并从其他两个不同的数据源创建两个视图。注意，需要提前配置元存储（见[使用](#使用)部分）。

    create schema vdm1.schema1;
    use vdm1.schema1;
    create view view1 as select * from mysql.table.test;
    create view view2 as select * from hive.table.test;
    select * from view1;

VDM数据源也可以通过动态目录API进行管理。有关更多信息，参见[动态目录](../admin/dynamic-catalog.html)。

## 所有支持的CLI查询

| 支持操作| 外部接口（SQL命令）|
|:----------|:----------|
| 添加VDM| `create catalog`（resulful）|
| 删除VDM| `drop catalog`（resulful）|
| 查询所有VDM| `show catalogs`|
| 创建模式| `create schema`|
| 删除模式| `drop schema`|
| 重命名模式| `rename schema`|
| 查询VDM下所有模式| `show schemas`|
| 查询模式下的所有视图| `show tables`|
| 创建/更新视图| `create [or replace] view`|
| 删除视图| `drop view`|
| 按视图查询数据| `select * from view`|
| 查询视图创建信息| `show create view`|
| 查询视图列信息| `desc view`|

