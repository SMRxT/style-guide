# SQL Style Guide

## Table of Contents

* [General style principles](#general-style-principles)


## General style principles

### Joining

When joining tables, the "joining table" comes first in the join condition.

#### Examples

```SQL
...
FROM table1 t1
    INNER JOIN table2 t2 ON t2.foo = t1.foo
...
```