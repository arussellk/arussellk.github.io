---
title: Upsert with ON CONFLICT ... DO UPDATE
---

```sql
create table if not exists foo (
    foo_id serial primary key,
    col varchar(50)
);

insert into foo (foo_id, col)
values (1, 'insert')
on conflict (foo_id) do update set col = 'upsert';
```
