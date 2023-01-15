# Домашняя работа № 11. Работа с индексами, join'ами, статистикой.

## 1 вариант:
## Создать индексы на БД, которые ускорят доступ к данным.

### 1. Создать индекс к какой-либо из таблиц вашей БД
> Создадим таблицу test с тремя полями id, col2, is_okay. Заполним таблицу значениями. И сразу создадим B-tree индекс в этом же скрипте.
> ```sql
> create table test as 
> select generate_series as id
>        , generate_series::text || (random() * 10)::text as col2 
>        , (array['Yes', 'No', 'Maybe'])[floor(random() * 3 + 1)] as is_okay
> from generate_series(1, 50000);
> create index idx_test_id on test(id);
> ```

### 2. Прислать текстом результат команды explain, в которой используется данный индекс
> ```sql
> explain
> select id from test where id = 1;
> ```
> Результат:
>
> <image src="images/explain1.png" alt="explain1">
>
> В данном запросе используется только индексное сканирование: "Index Only Scan". Т.е. наш запрос select исполует именно индексное сканирование.

### 3. Реализовать индекс для полнотекстового поиска

### 4. Реализовать индекс на часть таблицы или индекс на поле с функцией
> Создадим индекс на часть таблицы
> Сначала удалим таблицу test:
> ```sql
> drop table test;
> ```

> Создадим таблицу test с тремя полями id, col2, is_okay. Заполним таблицу значениями. И сразу создадим B-tree индекс на часть таблицы на поле id .
> ```sql
> create table test as 
> select generate_series as id
>         , generate_series::text || (random() * 10)::text as col2 
>         , (array['Yes', 'No', 'Maybe'])[floor(random() * 3 + 1)] as is_okay
> from generate_series(1, 50000);
> create index idx_test_id_100 on test(id) where id < 100;
> ```

> Посмотрим план запроса:
> ```sql
> explain
> select * from test where id < 100;
> ```
> Результат:
>
> <image src="images/explain3.png" alt="explain3">
>
> В данном запросе используется только индексное сканирование: "Index Scan using...". Т.е. наш запрос select с условием id < 100 использует именно индексное сканирование. Т.к. индекс у нас именно на часть таблицы (id < 100)

> Если посмотреть на план запроса на другую часть таблицы, где индекса нет, то будет уже не индексное сканирование, а "Seq Scan":
> ```sql
> explain
> select * from test where id > 100;
> ```
>
> <image src="images/explain4.png" alt="explain4">

### 5. Создать индекс на несколько полей
> Удалим таблицу test:
> ```sql
> drop table test;
> ```

> Создадим таблицу test с тремя полями id, col2, is_okay. Заполним таблицу значениями. И сразу создадим составной B-tree индекс на два поля id, is_okay в этом же скрипте.
> ```sql
> create table test as
> select generate_series as id
>         , generate_series::text || (random() * 10)::text as col2
>         , (array['Yes', 'No', 'Maybe'])[floor(random() * 3 + 1)] as is_okay
> from generate_series(1, 50000);
> create index idx_test_id_is_okay on test(id, is_okay);
> ```

> Посмотрим план запроса:
> ```sql
> explain
> select * from test where id = 1 and is_okay = 'True';
> ```
> Результат:
>
> <image src="images/explain2.png" alt="explain2">
>
> В данном запросе используется только индексное сканирование: "Index Scan using...". Т.е. наш запрос select с двумя полями id, is_okay в секции where исполует именно индексное сканирование.

## 2 вариант:
## Написание запросов с различными типами соединений

### 1. Реализовать прямое соединение двух или более таблиц

### 2. Реализовать левостороннее (или правостороннее) соединение двух или более таблиц

### 3. Реализовать кросс соединение двух или более таблиц

### 4. Реализовать полное соединение двух или более таблиц

### 5. Реализовать запрос, в котором будут использованы разные типы соединений

### 6. Сделать комментарии на каждый запрос

### 7. К работе приложить структуру таблиц, для которых выполнялись соединения