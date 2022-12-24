# Домашняя работа № 9. Репликация.

### 1. На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение.
> Я создал виртуальную машину с Ubuntu 22.04.1 LTS в Яндексе. Назвал ее ubuntu2204-vm. Скрин виртуальной машине Яндекса на картинке ниже:
> <image src="images/vm.png" alt="vm">

> Вместо трех ВМ я создал три кластера PostgreSQL:
> 
> <image src="images/3_pg_clusters.png" alt="3_pg_clusters">

> Задаем на всех трех кластерах wal_level=logical:
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
>
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
>
> Создаем БД db_repl и подключаемся к ней:
> ```sql
> postgres=# create database db_repl;
> postgres=# \c db_repl;
> ```
>
> Создаем таблицу test2 и вставляем в нее значения:
> ```sql
> db_repl=# create table test2(id integer, description text);
> db_repl=# INSERT INTO test2(id, description) VALUES (4, 'Четыре'), (5, 'Пять'), (6, 'Шесть');
> ```
> Результат:
> 
> <image src="images/db_repl2.png" alt="db_repl2">
>
> Создаем пустую таблицу test:
> ```sql
> db_repl=# create table test(id integer, description text);
> ```

### 3. На 1 ВМ создаем публикацию таблицы test 
> Подключаемся к первому кластеру (порт 5432) и подключаемся к БД db_repl
> ```sql
> aleksandr@ubuntu2204-vm:~$ sudo -u postgres psql -p 5432
> postgres=# \c db_repl;
> ```
>
> Создаем публикацию test_pub таблицы test
> ```sql
> db_repl=# CREATE PUBLICATION test_pub FOR TABLE test;
> ```
>
> Просмотр созданной публикации
> ```sql
> db_repl=# \dRp+
> ```
> Результат:
>
> <image src="images/test_pub.png" alt="test_pub">
>
> Задаем пароль pas123
> ```sql
> db_repl=# \password
> ```


### 4. На 2 ВМ создаем публикацию таблицы test2 
> Подключаемся ко второму кластеру (порт 5433) и подключаемся к БД db_repl
> ```sql
> aleksandr@ubuntu2204-vm:~$ sudo -u postgres psql -p 5433
> postgres=# \c db_repl;
> ```
>
> Создаем публикацию test2_pub таблицы test2
> ```sql
> db_repl=# CREATE PUBLICATION test2_pub FOR TABLE test2;
> ```
>
> Просмотр созданной публикации
> ```sql
> db_repl=# \dRp+
> ```
> Результат:
>
> <image src="images/test2_pub.png" alt="test2_pub">
>
> Задаем пароль pas123
> ```sql
> db_repl=# \password
> ```

### 5. На ВМ 1 подписываемся на публикацию таблицы test2 с ВМ №2.
> Подключаемся к первому кластеру (порт 5432) и подключаемся к БД db_repl
> ```sql
> aleksandr@ubuntu2204-vm:~$ sudo -u postgres psql -p 5432
> postgres=# \c db_repl;
> ```


> Создаем в первом кластере подписку на таблицу test2 воторого кластера: 
> ```sql
> db_repl=# CREATE SUBSCRIPTION test2_sub 
> CONNECTION 'host=localhost port=5433 user=postgres password=pas123 dbname=db_repl' 
> PUBLICATION test2_pub WITH (copy_data = true);
> ```
>
> Смотрим состояние подписки:
> ```sql
> db_repl=# \dRs
> ```
> Результат:
>
> <image src="images/sub2.png" alt="sub2">

> Посмотрим появились ли данные в таблице test2 первого кластера
> ```sql
> db_repl=# select * from test2;
> ```
>
> <image src="images/sel_test2.png" alt="sel_test2">
>
> Данные появились. Логическая репликация сработала

### 6. На ВМ 2 подписываемся на публикацию таблицы test с ВМ №1. 
> Подключаемся ко второму кластеру (порт 5433) и подключаемся к БД db_repl
> ```sql
> aleksandr@ubuntu2204-vm:~$ sudo -u postgres psql -p 5433
> postgres=# \c db_repl;
> ```


> Создаем во втором кластере подписку на таблицу test первого кластера:
> ```sql
> CREATE SUBSCRIPTION test_sub 
> CONNECTION 'host=localhost port=5432 user=postgres password=pas123 dbname=db_repl' 
> PUBLICATION test_pub WITH (copy_data = true);
> ```
>
> Смотрим состояние подписки:
> ```sql
> db_repl=# \dRs
> ```
> Результат:
>
> <image src="images/sub1.png" alt="sub1">


> Посмотрим появились ли данные в таблице test второго кластера
> ```sql
> db_repl=# select * from test;
> ```
>
> <image src="images/sel_test.png" alt="sel_test">
>
> Данные появились. Логическая репликация сработала

### 7. 3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ). 
> Подключаемся к первому кластеру (порт 5432) и подключаемся к БД db_repl
> ```sql
> aleksandr@ubuntu2204-vm:~$ sudo -u postgres psql -p 5432
> postgres=# \c db_repl;
> ```
>
> Создаем публикацию test2_pub таблицы test2 в первом кластере
> ```sql
> db_repl=# CREATE PUBLICATION test2_pub FOR TABLE test2;
> ```


> Подключаемся ко второму кластеру (порт 5433) и подключаемся к БД db_repl
> ```sql
> aleksandr@ubuntu2204-vm:~$ sudo -u postgres psql -p 5433
> postgres=# \c db_repl;
> ```
>
> Создаем публикацию test_pub таблицы test во втором кластере
> ```sql
> db_repl=# CREATE PUBLICATION test_pub FOR TABLE test;
> ```


> Подключаемся к третьему кластеру (порт 5434)
> ```sql
> aleksandr@ubuntu2204-vm:~$ sudo -u postgres psql -p 5434
> ```
>
> Создаем БД db_repl и подключаемся к ней:
> ```sql
> postgres=# create database db_repl;
> postgres=# \c db_repl;
> ```
>
> Создаем пустые таблицы test и test2:
> ```sql
> db_repl=# create table test(id integer, description text);
> db_repl=# create table test2(id integer, description text);
> ```


> Создаем в третьем кластере подписку на таблицу test2 первого кластера: 
> ```sql
> CREATE SUBSCRIPTION test2_sub 
> CONNECTION 'host=localhost port=5432 user=postgres password=pas123 dbname=db_repl' 
> PUBLICATION test2_pub WITH (copy_data = true);
> ```
>
> Посмотрим появились ли данные в таблице test2 третьего кластера
> ```sql
> db_repl=# select * from test2;
> ```
>
> <image src="images/sel3_test2.png" alt="sel3_test2">
>
> Данные появились. Логическая репликация сработала


> Создаем в третьем кластере подписку на таблицу test второго кластера: 
> ```sql
> CREATE SUBSCRIPTION test_sub 
> CONNECTION 'host=localhost port=5433 user=postgres password=pas123 dbname=db_repl' 
> PUBLICATION test_pub WITH (copy_data = true);
> ```
>
> Посмотрим появились ли данные в таблице test третьего кластера
> ```sql
> db_repl=# select * from test;
> ```
>
> <image src="images/sel3_test.png" alt="sel3_test">
>
> Данные появились. Логическая репликация сработала

### 6. Небольшое описание, того, что получилось.
