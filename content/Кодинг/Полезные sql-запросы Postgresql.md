---
title: 
description: 
permalink: 
tags:
  - "#postgresql"
  - "#sql"
draft: false
date:
---

## Посмотреть статистику заполнения по таблице
[Подробное описание](https://postgrespro.ru/docs/postgresql/9.5/view-pg-stats)

```sql
select 
	attname, 
	null_frac, 
	avg_width, 
	n_distinct
from pg_stats
where tablename = 'table_name'
```
- *null_frac* - Доля записей, в которых этот столбец содержит NULL
- *avg_width* - Средний размер элементов в столбце, в байтах
- *n_distinct* - Число больше нуля представляет примерное количество различных значений в столбце. Если это число меньше нуля, его модуль представляет количество различных значений, делённое на количество строк. (Отрицательная форма применяется, когда `ANALYZE` полагает, что число различных значений, скорее всего, будет расти по мере роста таблицы; положительная, когда в столбце, вероятно, будет фиксированное количество возможных значений.) Например, -1 указывает на столбец с уникальным содержимым, в котором количество различных значений совпадает с количеством строк.

## Узнать сколько места на диске занимает таблица

https://postgrespro.ru/docs/postgresql/11/functions-admin#FUNCTIONS-ADMIN-DBSIZE

Минимальный вариант для конкретной таблицы
```sql
select pg_size_pretty(pg_total_relation_size('table_name'))
```
Расширенный вариант
```sql
select 
  t.table_catalog, t.table_schema, t.table_name, t.table_type, 
  pg_size_pretty(pg_total_relation_size(t.table_schema||'.'||t.table_name)) as size
from information_schema.tables as t
order by pg_total_relation_size(t.table_schema||'.'||t.table_name) desc
```
можно отфильровать по:
- *t.table_catalog* - базе данных, 
- *t.table_schema* - схеме
- *t.table_type* - типу таблицы

## Сколько места занимает столбец таблицы
```sql
SELECT pg_size_pretty(sum(pg_column_size(column_name))) FROM osint.site;
```

## Вывести статистику по индексам
```sql
SELECT
  relid::regclass AS table_name,
  indexrelid::regclass AS index_name,
  pg_size_pretty(pg_relation_size(indexrelid::regclass)) AS index_size,
  idx_tup_read,
  idx_tup_fetch,
  idx_scan
  FROM pg_stat_user_indexes JOIN pg_index USING (indexrelid)
```
если хотим исключить из статистики индексы для первичных ключей - добавить в запрос
```sql
WHERE -NOT indisunique
```

- index_size - размер индекса на диске
-   idx_tup_read - Количество элементов индекса, возвращённых при сканированиях по этому индексу
- idx_tup_fetch - Количество живых строк таблицы, отобранных при простых сканированиях по этому индексу
- idx_scan - Количество произведённых сканирований по этому индексу

## Посмотреть какие операторы и типы данных поддерживают индексы
```sql
SELECT am.amname AS index_method,
       opf.opfname AS opfamily_name,
       amop.amopopr::regoperator AS opfamily_operator
    FROM pg_am am, pg_opfamily opf, pg_amop amop
    WHERE opf.opfmethod = am.oid AND
          amop.amopfamily = opf.oid
		  --and am.amname = 'gin'
    ORDER BY index_method, opfamily_name, opfamily_operator;
```

## Еще статистика по таблицам и индексам (на уровне страниц)
https://postgrespro.ru/docs/postgrespro/12/pgstattuple
```sql
create extension pgstattuple;
-- для таблиц
SELECT * FROM pgstattuple('schema.table');
-- для индексов gin
SELECT * FROM pgstatginindex('test_gin_index');
-- есть еще функции для приближенной оценки больших таблиц 
-- и для других типов индексов
```
Для таблиц

|                      |          |                                              |
| -------------------- | -------- | -------------------------------------------- |
| ==table_len==        | `bigint` | Физическая длина отношения в байтах          |
| `tuple_count`        | `bigint` | Количество «живых» кортежей                  |
| `tuple_len`          | `bigint` | Общая длина «живых» кортежей в байтах        |
| ==tuple_percent==    | `float8` | Процент «живых» кортежей                     |
| `dead_tuple_count`   | `bigint` | Количество «мёртвых» кортежей                |
| `dead_tuple_len`     | `bigint` | Общая длина «мёртвых» кортежей в байтах      |
| `dead_tuple_percent` | `float8` | Процент «мёртвых» кортежей                   |
| ==free_space==       | `bigint` | Общий объём свободного пространства в байтах |
| `free_percent`       | `float8` | Процент свободного пространства              |
Пример использования
```sql
SELECT pg_size_pretty(table_len), tuple_percent, pg_size_pretty(free_space) FROM pgstattuple('table_name');
```
получаем
- размер таблицы 170 МБ
- процент живых кортежей  - 53% 
- размер свободного места - 74 МБ

выполняем
``` sql
vacuum full table_name
```
повторяем запрос статистики, получаем
- размер таблицы 97 МБ
- процент живых кортежей  - 94%
- размер свободного места - 2 МБ

В результате получили сокращение времени выполнения запроса на 20%

## Сгенерировать случайную числовую последовательность

Кейс - поиск значений поля принадлежащего сгенерированной последовательности
```sql
select count(company_name) from table_name  
where siren  in (  
    SELECT sir from (select generate_series (1,100000), (floor(random()*(999999999-111111111+1))+111111111)::varchar AS sir) as a  
    )
```
- 100000 количество элементов в генерируемой последовательности
- 999999999 - максимальное значение 
- 111111111 - минимальное значение
