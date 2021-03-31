INSERT INTO CUBE
======

Synopsis
--------

``` sql
INSERT OVERWRITE CUBE cube_name [WHERE condition]
```

Description
-----------

Similar to INSERT INTO CUBE statement but with this statement the existing data is overwritten. Predicates 
are optional.

Examples
--------

Insert data based on condition into the `orders_cube` cube:

    INSERT OVERWRITE CUBE orders_cube WHERE orderdate > date '1999-01-01';
    INSERT OVERWRITE CUBE orders_cube;

See Also
--------

[INSERT INTO CUBE](./insert-cube.html), [CREATE CUBE](./create-cube.html), [SHOW CUBES](./show-cubes.html), [DROP CUBE](./drop-cube.html)
