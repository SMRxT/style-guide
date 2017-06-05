# SQL Style Guide

## Table of Contents

* [Syntax](#syntax)
* [Naming](#naming)
* [General style principles](#general-style-principles)
* [Schema Design](#schema-design)

## Syntax

### SQL keywords

All SQL keywords should be upper-cased.

```sql
CREATE TABLE blah...

SELECT *
FROM foo
WHERE 1 = 1
    AND NOT EXISTS (SELECT * FROM ...)
    AND foo.something IS NOT NULL
```

### Optional syntax

Prefer to include optional syntax in most cases. By being explicit, we ensure that it is plain to see what the author intended, which is better than assuming they wanted the default, when perhaps they just forgot.

* Be explicit with `INNER JOIN` even though `JOIN` means the same thing. This helps distinguish `LEFT JOIN` from `INNER JOIN` as well.
* Be explicit with `as` in column aliases: `COUNT(*) AS count` instead of `COUNT(*) count`
* Use `ASC` in `ORDER BY` clauses even though it is the default.
* Always specify `NULL` or `NOT NULL` for column definitions, even though `NULL` is the default.


### Indentation

Indentation should be used liberally to help visually separate logical groups of a SQL query.

Indent inner join clauses, sub selects, and additional predicates on a where clause.

#### Examples

```sql
SELECT 
    column1,
    column2,
    column3
FROM some_table
    INNER JOIN other_table ON ...
        AND ...
    INNER JOIN (
        SELECT *
        FROM a_table
    ) alias ON ...
WHERE ...
    AND ...
    AND ...
ORDER BY ...
GROUP BY ...
```


## Naming

### Tables and table-like things

Tables are named using the singular, present tense. Names are lower-cased and use snake casing to separate words.

#### Examples


```sql
CREATE TABLE user...
CREATE TABLE inventory_status...
```

### Columns

* All identifiers end with the `_id` suffix. 
* All version numbers end with the `_version` suffix.
* For both identifiers and version numbers, use the entire entity name as the prefix. It is never okay to use _just_ `id` or _just_ `version` as column names. (E.g. a table of `user` has a `user_id` and `user_version`.)
* Column names are lower case and also use snake casing.
* Prefer more verbose column names that distinguish that column from other tables. 

```sql
CREATE TABLE user (
   user_id serial NOT NULL,
   user_version int DEFAULT(1) NOT NULL,
   name text NOT NULL,
   ...
);
```

Using the full entity name in `id` and `version` names has the nice benefit of matching up when joining tables. It also ensures that when you filter by `id` or `version`, you are filtering by the correct entity.

```sql
SELECT *
FROM clinical.patient p
    INNER JOIN clinical.patient_therapy pt ON pt.patient_id = p.patient_id
WHERE p.patient_id = 5
    AND p.patient_version = 1
```

## General style principles

### Joining

When joining tables, the "joining table" comes first in the join condition.

#### Examples

```sql
SELECT *
FROM table1 t1
    INNER JOIN table2 t2 ON t2.foo = t1.foo
...
```

## Schema Design

* history tables
* valid_between
* schema and referencing; no circular dependencies
* 