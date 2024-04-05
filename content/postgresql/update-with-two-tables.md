---
title: Update with two tables
---

Sometimes it is useful to update one table based on associated data found in a second table.
It is easy to accidentally affect all rows because the update syntax does not have the
`JOIN ... USING (some_id)` phrase that you would usually use to restrict rows by id.

```sql
-- Often not what you intend!
-- Affects all rows because the where clause evaluates to true.
update foo f
set is_active = false
from bar b
where b.val = 'target';

-- Affects rows when both where conditions evaluate to true,
-- which is only the foo associated with 'target' in this db.
update foo f
set is_active = false
from bar b
where b.foo_id = f.foo_id
and b.val = 'target';
```

Full example:

```sql
create table if not exists foo (
    foo_id serial primary key,
    username varchar(50) unique,
    is_active boolean
);

create table if not exists bar (
    bar_id serial primary key,
    foo_id integer references foo,
    val varchar(50)
);

insert into foo (username, is_active)
values
    ('first', true),
    ('second', true),
    ('third', true);

insert into bar (foo_id, val)
values
    (1, 'target'),
    (2, 'not the target'),
    (3, 'not the target');

select *
from foo
join bar using (foo_id);

-- Affects no rows because the where clause evaluates to false.
update foo f
set is_active = false
from bar b
where b.val = 'not in the db';

-- Often not what you intend!
-- Affects all rows because the where clause evaluates to true.
update foo f
set is_active = false
from bar b
where b.val = 'target';

-- Affects rows when both where conditions evaluate to true,
-- which is only the foo associated with 'target' in this db.
update foo f
set is_active = false
from bar b
where b.foo_id = f.foo_id
and b.val = 'target';
```
