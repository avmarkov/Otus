# Домашняя работа № 10. Секционирование таблицы.

### 1. На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение.
> Для выполнения этой домашней работы я развернул виртуальную машину у себя с помощью VMware.
>
> Установим БД demo из postgrespro.ru
>
> ```sql
> sudo su postgres 
> cd $HOME && wget --quiet https://edu.postgrespro.ru/demo_medium.zip && unzip demo_medium.zip && psql < demo_medium.sql
> ```
> Посмотрим список таблиц:
>
> ```sql
> demo=# \dt bookings.*
> ```
> <image src="images/table_list.png" alt="table_list">

> Будем секционировать таблицу bookings:
> 
> <image src="images/bookings.png" alt="bookings">

> Будем секционировать таблицу bookings по полю book_date. Найдем минимальное и максимальное значение поля book_date:
> ```sql
> demo=# select min(book_date), min(book_date) from bookings.bookings;
> ```
>
> Результат:
>
> <image src="images/min_max_.png" alt="min_max">

> Создаю секционированную по полю book_date таблицу bookings_range 
>
> ```sql
> CREATE TABLE IF NOT EXISTS bookings.bookings_range
> (
>     book_ref character(6) COLLATE pg_catalog."default" NOT NULL,
>     book_date timestamp with time zone NOT NULL,
>     total_amount numeric(10,2) NOT NULL
> ) PARTITION BY RANGE(book_date);
> ```

> Создаю отдельные секции на каждый месяц:
> ```sql
> CREATE TABLE bookings_range_201606 PARTITION OF bookings.bookings_range 
>        FOR VALUES FROM ('2016-06-01'::timestamptz) TO ('2016-07-01'::timestamptz);
> CREATE TABLE bookings_range_201607 PARTITION OF bookings.bookings_range 
>        FOR VALUES FROM ('2016-07-01'::timestamptz) TO ('2016-08-01'::timestamptz);
> CREATE TABLE bookings_range_201608 PARTITION OF bookings.bookings_range 
>        FOR VALUES FROM ('2016-08-01'::timestamptz) TO ('2016-09-01'::timestamptz);
> CREATE TABLE bookings_range_201609 PARTITION OF bookings.bookings_range 
>        FOR VALUES FROM ('2016-09-01'::timestamptz) TO ('2016-10-01'::timestamptz);
> CREATE TABLE bookings_range_201610 PARTITION OF bookings.bookings_range 
>        FOR VALUES FROM ('2016-10-01'::timestamptz) TO ('2016-11-01'::timestamptz);
> ```

> Заполнениим bookings_range с автоматической раскладкой по секциям:
> ```sql
> INSERT INTO bookings.bookings_range SELECT * FROM bookings.bookings;
> ```
> 
> Результат:
> ```sql
> INSERT 0 593433
> ```

> Посмотрим как распределились данные по секционированной таблице:
> ```sql
> SELECT tableoid::regclass, count(*) 
> FROM bookings.bookings_range 
> GROUP BY tableoid;
> ```
>
> <image src="images/sel_part.png" alt="sel_part">

> Удалим внешний ключ tickets_book_ref_fkey, ссылающийся на эту таблицу:
> ```sql
> ALTER TABLE bookings.tickets DROP CONSTRAINT tickets_book_ref_fkey;
> ```
> Удалим таблицу bookings.bookings:
> ```sql
> TRUNCATE bookings.bookings;
> DROP TABLE bookings.bookings;
> ```

> Переименуем таблицу bookings.bookings_range на bookings:
> ```sql
> ALTER TABLE bookings.bookings_range RENAME TO bookings;
> ```

> Добавим первичный ключ:
> ```sql
> ALTER TABLE IF EXISTS bookings.bookings ADD CONSTRAINT bookings_pkey PRIMARY KEY (book_ref, book_date);
> ```

> При попытки вернуть внешний ключ, ссылающийся на таблицу bookings.bookings
> ```sql
> ALTER TABLE IF EXISTS bookings.tickets
> ADD CONSTRAINT tickets_book_ref_fkey FOREIGN KEY (book_ref)
> REFERENCES bookings.bookings (book_ref) MATCH SIMPLE
> ON UPDATE NO ACTION
> ON DELETE NO ACTION;
> ```
> Возникает ошибка ERROR:  there is no unique constraint matching given keys for referenced table "bookings". Очевидно, что ошибка из-за того, что ссылаемся на book_ref таблицы bookings.bookings, который неуникальный. При попытке сделать его уникальным также возникает ошибка, т.к. уникальный ключ должен еще включать и поле, по которому происходит секционирование, т.е. поле book_date.
>
> Подскажите, пожалуйста, как эту проблему поправить.

> Решил проблему таким образом:
>
> Сначала добавил поле book_date в таблицу bookings.tickets
> Затем заполнил это поле, на основе таблицы bookings.bookings:
> 
> ```sql
> UPDATE bookings.tickets
> SET book_date =
>       (SELECT book_date
>        FROM bookings.bookings
>        WHERE bookings.bookings.book_ref = bookings.tickets.book_ref);
> ```

> Результат:
> ```sql 
> UPDATE 829071
> ```

> Затем добавляем ограничение внешнего ключа, состоящего из двух полей bookings.bookings
> ```sql
> ALTER TABLE IF EXISTS bookings.tickets
> ADD CONSTRAINT tickets_book_ref_fkey FOREIGN KEY (book_date, book_ref)
> REFERENCES bookings.bookings (book_date, book_ref) MATCH SIMPLE
> ON UPDATE NO ACTION
> ON DELETE NO ACTION
> NOT VALID;
> ```

> Результат:
> ```sql
> ALTER TABLE
> Query returned successfully in 96 msec.
> ```

> Ограничение внешнего ключа добавилось.
