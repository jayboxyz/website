
# Hive Security Configuration


## Authorization


You can enable authorization checks for the [hive](./hive.html) by setting the `hive.security` property in the Hive catalog properties file. This property must be one of the following values:

| Property Value           | Description                                                  |
| :----------------------- | :----------------------------------------------------------- |
| `legacy` (default value) | Few authorization checks are enforced, thus allowing most operations. The config properties `hive.allow-drop-table`, `hive.allow-rename-table`, `hive.allow-add-column`, `hive.allow-drop-column` and `hive.allow-rename-column` are used. |
| `read-only`              | Operations that read data or metadata, such as `SELECT`, are permitted, but none of the operations that write data or metadata, such as `CREATE`, `INSERT` or `DELETE`, are allowed. |
| `file`                   | Authorization checks are enforced using a config file specified by the Hive configuration property `security.config-file`. See [File Based Authorization](./hive-security.html#file-based-authorization) for details. |
| `sql-standard`           | Users are permitted to perform the operations as long as they have the required privileges as per the SQL standard. In this mode, openLooKeng enforces the authorization checks for queries based on the privileges defined in Hive metastore. To alter these privileges, use the [GRANT](../sql/grant.html) and [REVOKE](../sql/revoke.html) commands. See [SQL Standard Based Authorization](./hive-security.html#sql-standard-based-authorization) for details. |

### SQL Standard Based Authorization

When `sql-standard` security is enabled, openLooKeng enforces the same SQL standard based authorization as Hive does.

Since openLooKeng\'s `ROLE` syntax support matches the SQL standard, and Hive does not exactly follow the SQL standard, there are the following limitations and differences:

-   `CREATE ROLE role WITH ADMIN` is not supported.
-   The `admin` role must be enabled to execute `CREATE ROLE` or `DROP ROLE`.
-   `GRANT role TO user GRANTED BY someone` is not supported.
-   `REVOKE role FROM user GRANTED BY someone` is not supported.
-   By default, all a user\'s roles except `admin` are enabled in a new user session.
-   One particular role can be selected by executing `SET ROLE role`.
-   `SET ROLE ALL` enables all of a user\'s roles except `admin`.
-   The `admin` role must be enabled explicitly by executing`SET ROLE admin`.

## Authentication


The default security configuration of the `/connector/hive` does not use
authentication when connecting to a Hadoop cluster. All queries are executed as the user who runs the openLooKeng process, regardless of which user submits the query.

The Hive connector provides additional security options to support Hadoop clusters that have been configured to use [Kerberos](#kerberos-support).

When accessing `HDFS (Hadoop Distributed File System)`, openLooKeng can [impersonate](./hive-security.html#end-user-impersonation) the end user who is running the query. This can be used with HDFS permissions and `ACLs (Access Control Lists)` to provide additional security for data.


**Warning**

Access to the openLooKeng coordinator should be secured using Kerberos when using Kerberos authentication to Hadoop services. Failure to secure access to the openLooKeng coordinator could result in unauthorized access to
sensitive data on the Hadoop cluster.

See [Coordinator Kerberos Authentication](../security/server.html) and [CLI Kerberos Authentication](../security/cli.html) for information on setting up Kerberos authentication.

   

## Kerberos Support


In order to use the Hive connector with a Hadoop cluster that uses `kerberos` authentication, you will need to configure the connector to work with two services on the Hadoop cluster:

-   The Hive metastore Thrift service
-   The Hadoop Distributed File System (HDFS)

Access to these services by the Hive connector is configured in the properties file that contains the general Hive connector configuration.

**Note**

If your `krb5.conf` location is different from `/etc/krb5.conf` you must set it explicitly using the `java.security.krb5.conf` JVM property in `jvm.config` file.

Example: `-Djava.security.krb5.conf=/example/path/krb5.conf`.


### Hive Metastore Thrift Service Authentication

In a Kerberized Hadoop cluster, openLooKeng connects to the Hive metastore Thrift service using
`SASL (Simple Authentication and Security Layer)` and authenticates using Kerberos. Kerberos authentication for the metastore is configured in the connector\'s properties file using the following properties:

| Property Name                                 | Description                                                  |
| :-------------------------------------------- | :----------------------------------------------------------- |
| `hive.metastore.authentication.type`          | Hive metastore authentication type.                          |
| `hive.metastore.thrift.impersonation.enabled` | Enable Hive metastore end user impersonation.                |
| `hive.metastore.service.principal`            | The Kerberos principal of the Hive metastore service.        |
| `hive.metastore.client.principal`             | The Kerberos principal that openLooKeng will use when connecting to the Hive metastore service. |
| `hive.metastore.client.keytab`                | Hive metastore client keytab location.                       |
| `hive.metastore.krb5.conf.path`               | The Kerberos configuration file location.                    |

#### `hive.metastore.authentication.type`

One of `NONE` or `KERBEROS`. When using the default value of `NONE`, Kerberos authentication is disabled and no other properties need to be configured.

When set to `KERBEROS` the Hive connector will connect to the Hive metastore Thrift service using SASL and authenticate using Kerberos.

This property is optional; the default is `NONE`.

####  `hive.metastore.thrift.impersonation.enabled`

Enable end-user Hive metastore impersonation.

This property is optional; the default is `false`.

#### `hive.metastore.service.principal`

The Kerberos principal of the Hive metastore service. The openLooKeng coordinator will use this to authenticate the Hive metastore.

The `_HOST` placeholder can be used in this property value. When connecting to the Hive metastore, the Hive connector will substitute in the hostname of the **metastore** server it is connecting to. This is useful if the metastore runs on multiple hosts.

Example: `hive/hive-server-host@EXAMPLE.COM` or `hive/_HOST@EXAMPLE.COM`.

This property is optional; no default value.

#### `hive.metastore.client.principal`

The Kerberos principal that openLooKeng will use when connecting to the Hive metastore.

The `_HOST` placeholder can be used in this property value. When connecting to the Hive metastore, the Hive connector will substitute in the hostname of the **worker** node openLooKeng is running on. This is useful if each worker node has its own Kerberos principal.

Example: `openlookeng/openlookeng-server-node@EXAMPLE.COM` or `openlookeng/_HOST@EXAMPLE.COM`.

This property is optional; no default value.

**Warning**

The principal specified by `hive.metastore.client.principal` must have sufficient privileges to remove files and directories within the `hive/warehouse` directory. If the principal does not, only the metadata
will be removed, and the data will continue to consume disk space.

This occurs because the Hive metastore is responsible for deleting the internal table data. When the metastore is configured to use Kerberos authentication, all of the HDFS operations performed by the metastore are impersonated. Errors deleting data are silently ignored.


#### `hive.metastore.client.keytab`

The path to the keytab file that contains a key for the principal specified by  `hive.metastore.client.principal`. This file must be readable by the operating system user running openLooKeng.

This property is optional; no default value.

#### `hive.metastore.krb5.conf.path`
The path of Kerberos configuration file. This file must be readable by the operating system user running openLooKeng.

This property is optional; no default value.

#### Example configuration with `NONE` authentication

``` properties
hive.metastore.authentication.type=NONE
```

The default authentication type for the Hive metastore is `NONE`. When the authentication type is `NONE`, openLooKeng connects to an unsecured Hive metastore. Kerberos is not used.

#### Example configuration with `KERBEROS` authentication

``` properties
hive.metastore.authentication.type=KERBEROS
hive.metastore.thrift.impersonation.enabled=true
hive.metastore.service.principal=hive/hive-metastore-host.example.com@EXAMPLE.COM
hive.metastore.client.principal=openlk@EXAMPLE.COM
hive.metastore.client.keytab=/etc/openlookeng/hive.keytab
hive.metastore.krb5.conf.path=/etc/openlookeng/krb5.conf
```

When the authentication type for the Hive metastore Thrift service is `KERBEROS`, openLooKeng will connect as the Kerberos principal specified by the property `hive.metastore.client.principal`. openLooKeng will authenticate this principal using the keytab specified by the `hive.metastore.client.keytab` property, and will verify that the identity of the metastore matches `hive.metastore.service.principal`.

Keytab files must be distributed to every node in the cluster that runs openLooKeng.

[Additional Information About Keytab Files](#additional-information-about-keytab-files)

### HDFS Authentication

In a Kerberized Hadoop cluster, openLooKeng authenticates to HDFS using Kerberos. Kerberos authentication for HDFS is configured in the connector\'s properties file using the following properties:

| Property Name                       | Description                                                  |
| :---------------------------------- | :----------------------------------------------------------- |
| `hive.hdfs.authentication.type`     | HDFS authentication type. Possible values are `NONE` or `KERBEROS`. |
| `hive.hdfs.impersonation.enabled`   | Enable HDFS end-user impersonation.                          |
| `hive.hdfs.presto.principal`        | The Kerberos principal that openLooKeng will use when connecting to HDFS. |
| `hive.hdfs.presto.keytab`           | HDFS client keytab location.                                 |
| `hive.hdfs.wire-encryption.enabled` | Enable HDFS wire encryption.                                 |

#### `hive.hdfs.authentication.type`

One of `NONE` or `KERBEROS`. When using the default value of `NONE`, Kerberos authentication is disabled and no other properties need to be configured.

When set to `KERBEROS`, the Hive connector authenticates to HDFS using Kerberos.

This property is optional; the default is `NONE`.

#### `hive.hdfs.impersonation.enabled`

Enable end-user HDFS impersonation.

The section`End User Impersonation` gives an in-depth explanation of HDFS impersonation.

This property is optional; the default is `false`.

#### `hive.hdfs.presto.principal`

The Kerberos principal that openLooKeng will use when connecting to HDFS.

The `_HOST` placeholder can be used in this property value. When connecting to HDFS, the Hive connector will substitute in the hostname of the **worker** node openLooKeng is running on. This is useful if each worker node has its own Kerberos principal.

Example: `openlookeng-hdfs-superuser/openlookeng-server-node@EXAMPLE.COM` 

or `openlookeng-hdfs-superuser/_HOST@EXAMPLE.COM`.

This property is optional; no default value.

#### `hive.hdfs.presto.keytab`

The path to the keytab file that contains a key for the principal specified by `hive.hdfs.presto.principal`. This file must be readable by the operating system user running openLooKeng.

This property is optional; no default value.

#### `hive.hdfs.wire-encryption.enabled`

In a Kerberized Hadoop cluster that uses HDFS wire encryption, this should be set to `true` to enable openLooKeng to access HDFS. Note that using wire encryption may impact query execution performance.

#### Example configuration with `NONE` authentication

``` properties
hive.hdfs.authentication.type=NONE
```

The default authentication type for HDFS is `NONE`. When the authentication type is `NONE`, openLooKeng connects to HDFS using Hadoop\'s simple authentication mechanism. Kerberos is not used.

#### Example configuration with `KERBEROS` authentication

``` properties
hive.hdfs.authentication.type=KERBEROS
hive.hdfs.presto.principal=hdfs@EXAMPLE.COM
hive.hdfs.presto.keytab=/etc/openlookeng/hdfs.keytab
```

When the authentication type is `KERBEROS`, openLooKeng accesses HDFS as the principal specified by the `hive.hdfs.presto.principal` property. openLooKeng will authenticate this principal using the keytab specified by the `hive.hdfs.presto.keytab` keytab.

Keytab files must be distributed to every node in the cluster that runs openLooKeng.

[Additional Information About Keytab Files](./hive-security.html#additional-information-about-keytab-files)

## End User Impersonation


### Impersonation Accessing HDFS

openLooKeng can impersonate the end user who is running a query. In the case of a user running a query from the command line interface, the end user is the username associated with the openLooKeng CLI process or argument to the optional `--user` option. Impersonating the end user can provide additional security when accessing HDFS if HDFS permissions or ACLs are used.

HDFS Permissions and ACLs are explained in the [HDFS Permissions Guide](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsPermissionsGuide.html).

#### `NONE` authentication with HDFS impersonation

``` properties
hive.hdfs.authentication.type=NONE
hive.hdfs.impersonation.enabled=true
```

When using `NONE` authentication with impersonation, openLooKeng impersonates the user who is running the query when accessing HDFS. The user openLooKeng is running as must be allowed to impersonate this user, as discussed in the section `configuring-hadoop-impersonation`. Kerberos is not used.

#### `KERBEROS` Authentication With HDFS Impersonation

``` properties
hive.hdfs.authentication.type=KERBEROS
hive.hdfs.impersonation.enabled=true
hive.hdfs.presto.principal=openlk@EXAMPLE.COM
hive.hdfs.presto.keytab=/etc/openlookeng/hdfs.keytab
```

When using `KERBEROS` authentication with impersonation, openLooKeng impersonates the user who is running the query when accessing HDFS. The principal specified by the `hive.hdfs.presto.principal` property must be
allowed to impersonate this user, as discussed in the section `configuring-hadoop-impersonation`. openLooKeng authenticates `hive.hdfs.presto.principal` using the keytab specified by `hive.hdfs.presto.keytab`.

Keytab files must be distributed to every node in the cluster that runs openLooKeng.

[Additional Information About Keytab Files](#additional-information-about-keytab-files)

### Impersonation Accessing the Hive Metastore

openLookeng supports impersonating the end user when accessing the Hive metastore.
Metastore impersonation can be enabled with

    hive.metastore.thrift.impersonation.enabled=true

When using `KERBEROS` Metastore authentication with impersonation, the principal
specified by the `hive.metastore.client.principal` property must be allowed to
impersonate the current openLooKeng user, as discussed in the section
[Impersonation in Hadoop](#impersonation-in-hadoop).

### Impersonation in Hadoop

In order to use `NONE` **authentication with HDFS impersonation** or `KERBEROS` **Authentication With HDFS Impersonation**, the Hadoop cluster must be configured to allow the user or principal that openLooKeng is running as to impersonate the users who log in to openLooKeng. Impersonation in Hadoop is configured in the file `core-site.xml`. A complete description of the configuration options can be found in the [Hadoop
documentation](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/Superusers.html#Configurations).

## Additional Information About Keytab Files


Keytab files contain encryption keys that are used to authenticate principals to the Kerberos `KDC (Key Distribution Center)`. These encryption keys must be stored securely; you should take the same precautions to protect them that you would to protect ssh private keys.

In particular, access to keytab files should be limited to the accounts that actually need to use them to authenticate. In practice, this is the user that the openLooKeng process runs as. The ownership and permissions on
keytab files should be set to prevent other users from reading or modifying the files.

Keytab files need to be distributed to every node running openLooKeng. Under common deployment situations, the Hive connector configuration will be the same on all nodes. This means that the keytab needs to be in the
same location on every node.

You should ensure that the keytab files have the correct permissions on every node after distributing them.

## File Based Authorization


The config file is specified using JSON and is composed of three sections, each of which is a list of rules that are matched in the orderspecified in the config file. The user is granted the privileges from the first matching rule. All regexes default to `.*` if not specified.

### Schema Rules

These rules govern who is considered an owner of a schema.

-   `user` (optional): regex to match against user name.
-   `schema` (optional): regex to match against schema name.
-   `owner` (required): boolean indicating ownership.

### Table Rules

These rules govern the privileges granted on specific tables.

-   `user` (optional): regex to match against user name.
-   `schema` (optional): regex to match against schema name.
-   `table` (optional): regex to match against table name.
-   `privileges` (required): zero or more of `SELECT`, `INSERT`,
    `DELETE`, `OWNERSHIP`, `GRANT_SELECT`.

### Session Property Rules

These rules govern who may set session properties.

-   `user` (optional): regex to match against user name.
-   `property` (optional): regex to match against session property name.
-   `allowed` (required): boolean indicating whether this session
    property may be set.

See below for an example.

``` json
{
  "schemas": [
    {
      "user": "admin",
      "schema": ".*",
      "owner": true
    },
    {
      "user": "guest",
      "owner": false
    },
    {
      "schema": "default",
      "owner": true
    }
  ],
  "tables": [
    {
      "user": "admin",
      "privileges": ["SELECT", "INSERT", "DELETE", "OWNERSHIP"]
    },
    {
      "user": "banned_user",
      "privileges": []
    },
    {
      "schema": "default",
      "table": ".*",
      "privileges": ["SELECT"]
    }
  ],
  "sessionProperties": [
    {
      "property": "force_local_scheduling",
      "allow": true
    },
    {
      "user": "admin",
      "property": "max_split_size",
      "allow": true
    }
  ]
}
```
