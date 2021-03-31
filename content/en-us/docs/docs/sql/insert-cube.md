INSERT INTO CUBE
======

Synopsis
--------

``` sql
INSERT INTO CUBE cube_name [WHERE condition]
```

Description
-----------

Insert data into a star-tree cube. Predicate information is optional. If predicate provided, only data matching 
the given predicate are processed from the source table and inserted into the cube. Otherwise, entire 
data from the source table is processed and inserted into Cube.

Examples
--------

Insert data based on condition into the `orders_cube` cube:

    INSERT INTO CUBE orders_cube WHERE orderdate > date '1999-01-01';
    INSERT INTO CUBE order_all_cube;

See Also
--------

[INSERT OVERWRITE CUBE](./insert-overwrite-cube.html), [CREATE CUBE](./create-cube.html), [SHOW CUBES](./show-cubes.html), [DROP CUBE](./drop-cube.html)


Limitations
----------
1. Insert statement does not allow different columns to be used in the where clause for successive inserts.

```sql
   CREATE CUBE orders_cube ON orders WITH (AGGREGATIONS = (count(*)), GROUP = (orderdate));
   
   INSERT INTO CUBE orders_cube WHERE orderdate BETWEEN date '1999-01-01' AND date '1999-01-05';
   
   -- This statement would fail because its possible the Cube already contain rows matching the given predicate.
   INSERT INTO CUBE orders_cube WHERE location = 'Canada';
```
Note: this means that columns used in the first insert must be used in every insert predicate following the first to avoid inserting duplicate data.

2. Range Predicate issue. 

```sql
   CREATE CUBE orders_cube ON orders WITH (AGGREGATIONS = (count(*)), GROUP = (orderdate));
   
   INSERT INTO CUBE orders_cube WHERE orderdate BETWEEN date '1999-01-01' AND date '1999-01-05';
   INSERT INTO CUBE orders_cube WHERE orderdate BETWEEN date '1999-01-06' AND date '1999-01-10';
   
   SET SESSION enable_star_tree_index=true;
   
   -- This statement uses orders_cube --
   SELECT count(*) FROM orders WHERE orderdate BETWEEN date '1999-01-03' AND date '1999-01-04';
   -- This statement uses orders_cube --
   SELECT count(*) FROM orders WHERE orderdate BETWEEN date '1999-01-07' AND date '1999-01-09';
   -- This statement does not user orders_cube because the two predicates used in the INSERT statement cannot be merged 
   -- and the TupleDomain evaluation to check cubePredicate.contains(statementPredicate) evaluates to false
   
   SELECT count(*) FROM orders WHERE orderdate BETWEEN date '1999-01-04' AND date '1999-01-07';
```
      