# lesson8.1

Создал таблицу

```
CREATE TABLE jsonb_data (
    id SERIAL PRIMARY KEY,
    data JSONB
);
CREATE TABLE
```
Сгенерировал 1000000 записей
```
INSERT INTO jsonb_data (data)
SELECT jsonb_build_object(
    'id', gs,
    'name', 'name_' || gs,
    'value', random() * 100
)
FROM generate_series(1, 1000000) AS gs;
INSERT 0 1000000
postgres=# SELECT COUNT(*) FROM jsonb_data;
  count
---------
 1000000
(1 row)
```
Создал индекс
```
CREATE INDEX idx_jsonb_data ON jsonb_data USING GIN (data);
```
Обновил 1 поле
```
SELECT * FROM pg_indexes WHERE tablename = 'jsonb_data';
postgres=# UPDATE jsonb_data
SET data = jsonb_set(data, '{name}', '"new_name"')
WHERE id = 1;
UPDATE 1
```
Блоатинг TOAST
```
SELECT
    relname,
    n_live_tup AS live_tuples,
    n_dead_tup AS dead_tuples,
    n_dead_tup::float / (n_live_tup + n_dead_tup) * 100 AS dead_tuple_percentage
FROM
    pg_stat_user_tables
WHERE
    relname = 'jsonb_data';
  relname   | live_tuples | dead_tuples | dead_tuple_percentage
------------+-------------+-------------+-----------------------
 jsonb_data |     1000000 |           1 |     9.99999000001e-05
(1 row)
```
Сделал VACUUM
```
VACUUM jsonb_data;
VACUUM
```
```
SELECT
    relname,
    n_live_tup AS live_tuples,
    n_dead_tup AS dead_tuples,
    n_dead_tup::float / (n_live_tup + n_dead_tup) * 100 AS dead_tuple_percentage
FROM
    pg_stat_user_tables
WHERE
    relname = 'jsonb_data';
  relname   | live_tuples | dead_tuples | dead_tuple_percentage
------------+-------------+-------------+-----------------------
 jsonb_data |     1000000 |           0 |                     0
(1 row)
```
