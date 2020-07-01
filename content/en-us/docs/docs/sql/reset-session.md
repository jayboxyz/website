RESET SESSION
=============

Synopsis
--------

``` sql
RESET SESSION name
RESET SESSION catalog.name
```

Description
-----------

Reset a session property value to the default value.

Examples
--------

``` sql
RESET SESSION optimize_hash_generation;
RESET SESSION hive.optimized_reader_enabled;
```

See Also
--------

[set-session](./set-session.html), [show-session](./show-session.html)
