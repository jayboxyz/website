+++

weight = 6
title = "系统访问控制"
+++


# 系统访问控制

Presto 将向协调节点进行身份验证的主体的概念与负责运行查询的用户名分开。

例如，在运行 Presto CLI 时，可以使用 `--user` 选项来指定 Presto 用户名。

默认情况下，Presto 协调节点允许任何主体以任何 Presto 用户身份运行查询。在安全的环境中，这很可能不是理想的行为，可能需要自定义。

## 实现

`SystemAccessControlFactory` 负责创建 `SystemAccessControl` 实例。它还定义管理员在 Presto 配置中使用的 `SystemAccessControl` 名称。

`SystemAccessControl` 实现负责：

-   验证给定的主体是否有权以特定的用户身份执行查询。
-   确定给定的用户是否能够更改给定系统属性的值。
-   对所有目录进行访问检查。这些访问检查在任何特定于连接器的检查之前执行，因此可能拒绝 `ConnectorAccessControl` 允许的权限。

必须将 `SystemAccessControl` 和 `SystemAccessControlFactory` 的实现包装为一个插件并将其安装在 Presto 集群上。

## 配置

在协调节点上安装实现 `SystemAccessControl` 和 `SystemAccessControlFactory` 的插件后，使用 `etc/access-control.properties` 文件对其进行配置。除 `access-control.name` 之外，所有属性都特定于 `SystemAccessControl` 实现。

`Access-control.name` 属性由 Presto 用于根据 `SystemAccessControlFactory.getName()` 返回的名称查找注册的 `SystemAccessControlFactory`。其余的属性作为到 `SystemAccessControlFactory.create()` 的映射进行传递。

配置文件示例：

``` {.none}
access-control.name=custom-access-control
custom-property1=custom-value1
custom-property2=custom-value2
```
