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
>
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