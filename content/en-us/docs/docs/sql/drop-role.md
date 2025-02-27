
DROP ROLE
=========

Synopsis
--------

``` sql
DROP ROLE role_name
```

Description
-----------

`DROP ROLE` drops the specified role in the current catalog.

For `DROP ROLE` statement to succeed, the user executing it should possess admin privileges for the given role.

Examples
--------

Drop role `admin` :

    DROP ROLE admin;

Limitations
-----------

Some connectors do not support role management. See connector documentation for more details.

See Also
--------

[CREATE ROLE](./create-role.html), [SET ROLE](./set-role.html), [GRANT ROLES](./grant-roles.html), [REVOKE ROLES](./revoke-roles.html)
