---
title: 
description: 
permalink: 
tags:
  - postgresql
draft: false
date:
---
## 1. Создаем словарь ispell

На примере словаря Лебедева.
1. Качаем словарь [отсюда](http://digitorum.ru/blog/2012/07/16/Russkie-slovari-dlya-ispell.phtml). Словарь сохранен в кодировке win-1251
2. переименовываем файлы 
	1. russian.dic -> russian_lebedev.dict
	2. russian.aff -> russian_lebedev.affix
3. копируем словари в каталог postgresl
	1. windows - "C:\\Program Files\\PostgreSQL\\13\\share\\tsearch_data"
	2. .nix - "/usr/share/postgresql/9.6/tsearch_data/"
4. перекодируем словари в кодировку utf-8
5. создаем в postgresql словарь
```sql
CREATE TEXT SEARCH DICTIONARY lebedev (
   TEMPLATE = ispell,
   dictfile = 'russian_lebedev',
   afffile = 'russian_lebedev',
   stopwords='russian'
);
```

6. Создаем словарь snowball
```sql
CREATE TEXT SEARCH DICTIONARY russian_stem (
    TEMPLATE = snowball,
    Language = russian,
    StopWords = russian
);
```
## 2. Создаем конфигурацию

```sql
CREATE TEXT SEARCH CONFIGURATION conf_name (
    PARSER = default
)
```

## 3. Настраиваем конфигурацию

```sql
ALTER TEXT SEARCH CONFIGURATION config_name
    ALTER MAPPING FOR word, numword, hword, hword_part, numhword, int, float
    WITH lebedev, russian_stem;
```

типы разбираемых токенов и состав словарей могут варьироваться, это частная конфигурация для русскоязычных текстов общего назначения

## 4. Подготавливаем таблицу для поиска

Создаем столбец типа tsvector в котором будет при добавлении новой записи будет автоматически заполняться подготовленное для поиска значения из одного или нескольких столбцов

```sql
ALTER TABLE table_name
    ADD COLUMN search_column tsvector
               GENERATED ALWAYS AS (to_tsvector('config_name', coalesce(column1, '') || ' ' || coalesce(column2, ''))) STORED;

```

и создаем индекс для этого поля

```sql

CREATE INDEX textsearch_idx ON table_name USING GIN (search_column);

```

## Как искать

```sql
select * from table_name
where
    search_column @@ websearch_to_tsquery('config_name', request_string);

```

через sqlalchemy
[https://amitosh.medium.com/full-text-search-fts-with-postgresql-and-sqlalchemy-edc436330a0c](https://amitosh.medium.com/full-text-search-fts-with-postgresql-and-sqlalchemy-edc436330a0c)