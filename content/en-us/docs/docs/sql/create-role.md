
CREATE ROLE
===========

Synopsis
--------

``` sql
CREATE ROLE role_name
[ WITH ADMIN ( user | USER user | ROLE role | CURRENT_USER | CURRENT_ROLE ) ]
```

Description
-----------

`CREATE ROLE` creates the specified role in the current catalog.

The optional `WITH ADMIN` clause causes the role to be created with the specified user as a role admin. A role admin has permission to drop or grant a role. If the optional `WITH ADMIN` clause is not specified, the role is created with current user as admin.

Examples
--------

Create role `admin` :

    CREATE ROLE admin;

Create role `moderator` with admin `bob`:

    CREATE ROLE moderator WITH ADMIN USER bob;

Limitations
-----------

Some connectors do not support role management. See connector documentation for more details.

See Also
--------

[DROP ROLE](./drop-role.html), [SET ROLE](./set-role.html), [GRANT ROLES](./grant-roles.html), [REVOKE ROLES](./revoke-roles.html)
