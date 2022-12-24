# Домашняя работа № 9. Репликация.

### 1. На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение.
> Я создал виртуальную машину с Ubuntu 22.04.1 LTS в Яндексе. Назвал ее ubuntu2204-vm. Скрин виртуальной машине Яндекса на картинке ниже:
> <image src="images/vm.png" alt="vm">

> Вместо трех ВМ я создал три кластера PostgreSQL:
> 
> <image src="images/3_pg_clusters.png" alt="3_pg_clusters">

> Задаем на первом кластере wal_level=logical:
> ```sql
> aleksandr@ubuntu2204-vm:~$ sudo -u postgres psql -p 5432
> postgres=# ALTER SYSTEM SET wal_level = logical;
> aleksandr@ubuntu2204-vm:~$ sudo pg_ctlcluster 14 main1 restart;
> ```
> Результат:
>
> <image src="images/wal_level.png" alt="wal_level">

> Создаем БД db_repl и подключаемся к ней:
> ```sql
> postgres=# create database db_repl;
> postgres=# \c db_repl;
> ```

> Создаем таблицу test и вставляем в нее значения:
> ```sql
> db_repl=# create table test(id integer, description text);
> db_repl=# INSERT INTO test(id, description) VALUES (1, 'Один'), (2, 'Два'), (3, 'Три');
> ```
>
> Результат:
> 
> <image src="images/db_repl.png" alt="db_repl">
>
> Создаем пустую таблицу test2:
> ```sql
> db_repl=# create table test2(id integer, description text);
> ```

### 2. На 2 ВМ создаем таблицы test2 для записи, test для запросов на чтение. 
> Подключаемся ко второму кластеру (порт 5433)
> ```sql
> aleksandr@ubuntu2204-vm:~$ sudo -u postgres psql -p 5433
> ```

> Создаем БД db_repl и подключаемся к ней:
> ```sql
> postgres=# create database db_repl;
> postgres=# \c db_repl;
> ```

> Создаем таблицу test2 и вставляем в нее значения:
> ```sql
> db_repl=# create table test2(id integer, description text);
> db_repl=# INSERT INTO test2(id, description) VALUES (4, 'Четыре'), (5, 'Пять'), (6, 'Шесть');
> ```
> Результат:
> 
> <image src="images/db_rep2.png" alt="db_rep2">
>
> Создаем пустую таблицу test:
> ```sql
> db_repl=# create table test(id integer, description text);
> ```

### 3. На 1 ВМ создаем публикацию таблицы test и подписываемся на публикацию таблицы test2 с ВМ №2.



### 4. На 2 ВМ создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test1 с ВМ №1. 

### 5. 3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ). 

### 6. Небольшое описание, того, что получилось.
