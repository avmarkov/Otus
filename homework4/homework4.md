# Домашняя работа № 4. Работа с базами данных, пользователями и правами

### 1. Cоздайте новый кластер PostgresSQL 14
> Я создал виртуальную машину с Ubuntu 22.04.1 LTS в Яндексе. Назвал ее ubuntu2204-vm3. Скрин виртуальной машине Яндекса на картинке ниже:
> <image src="images/vm_yandex.png" alt="vm_yandex">
>
> Т.к. версия Ubuntu 22.04, устанавливаю PostgreSQL так (знаю, что установится 14-я версия):
> ```sh
> aleksandr@ubuntu2204-vm3:~$ sudo apt-get -y install postgresql
> ```
> Кластер создался:
> 
> <image src="images/pg_cluster.png" alt="pg_cluster">
### 2. Зайдите в созданный кластер под пользователем postgres
> Зашел:
> ```sh
> aleksandr@ubuntu2204-vm3:~$ sudo -u postgres psql
> ```
> <image src="images/postgres.png" alt="postgres">

### 3. Cоздайте новую базу данных testdb
> ```sh
> postgres=# CREATE DATABASE testdb;
> ```

### 4. Зайдите в созданную базу данных под пользователем postgres
> ```sh
> postgres=# \c testdb
> ```

### 5. Создайте новую схему testnm
> ```sh
> testdb=# CREATE SCHEMA testnm;
> ```

### 6. Создайте новую таблицу t1 с одной колонкой c1 типа integer
> ```sh
> testdb=# CREATE TABLE t1(c1 integer);
> ```

### 7. Вставьте строку со значением c1=1
> ```sh
> testdb=# INSERT INTO t1 VALUES(1);
> ```

### 8. Создайте новую роль readonly
> ```sh
> testdb=# CREATE ROLE readonly;
> ```

### 9. Дайте новой роли право на подключение к базе данных testdb
> ```sh
> testdb=# GRANT CONNECT ON DATABASE testdb TO readonly;
> ```

### 10. Дайте новой роли право на использование схемы testnm
> ```sh
> testdb=# GRANT USAGE ON SCHEMA testnm to readonly;
> ```

### 11. Дайте новой роли право на select для всех таблиц схемы testnm
> ```sh
> testdb=# GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;
> ```

### 12. Создайте пользователя testread с паролем test123
> ```sh
> testdb=# CREATE USER testread WITH password 'test123';
> ```

### 13. Дайте роль readonly пользователю testread
> ```sh
> testdb=# GRANT readonly TO testread;
> ```

### 14. Зайдите под пользователем testread в базу данных testdb
> ```sh
> testdb=# \c testdb testread
>```
> Подключиться не получилось. Результат:
>
> <image src="images/user_testread.png" alt="user_testread">

> Подключиться так получилось:
> ```sh
> aleksandr@ubuntu2204-vm3:~$ psql -h localhost -U testread -d testdb -W
>```

### 15. Cделайте select * from t1;
> Сделал
> ```sh
> testdb=> select * from t1;
>```

### 16. Получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)
> Не получилось
>
> <image src="images/sel_res.png" alt="sel_res">

### 17. Напишите что именно произошло в тексте домашнего задания
> Мы создали таблицу t1. Создали схему testnm. Создали роль readonly, дали этой роли права на подключение к базе данных testdb,
> на использование схемы testnm, на select для всех таблиц схемы testnm. Затем создали пользователя testread и дали ему роль readonly.

### 18. У вас есть идеи почему? ведь права то дали?
> Не получилось, т.к. таблица t1 создалась в схеме public. А права роли readonly мы дали только для схемы testnm, поэтому к схеме public нет доступа.
> О чем и говорится в тексте ошибки при выполнении запроса "select * from t1" - ERROR:  permission denied for table t1

### 19. Посмотрите на список таблиц
> Список таблиц такой:
>
> <image src="images/table_list.png" alt="table_list">
>
> У таблицы t1 схема - public. Что и требовалось доказать)

### 20. Подсказка в шпаргалке под пунктом 20

### 21. А почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)
> Потому что search_path - "$user", public. Т.к. $USER нет(а ее реально нет, т.к. нет схемы с именем равным имени пользователя), таблица по умолчанию создалась в схеме public.
>
> <image src="images/search_path.png" alt="search_path">

### 22. Вернитесь в базу данных testdb под пользователем postgres
> Вернулся:
> ```sh
> aleksandr@ubuntu2204-vm3:~$ sudo -u postgres psql
> postgres=# \c testdb
>```

### 23. Удалите таблицу t1
> ```sh
> testdb=# DROP TABLE t1;
>```

### 24. Создайте ее заново но уже с явным указанием имени схемы testnm
> ```sh
> testdb=# CREATE TABLE testnm.t1(c1 integer);
>```

### 25. Вставьте строку со значением c1=1
> ```sh
> testdb=# INSERT INTO testnm.t1 values(1);
>```

### 26. Зайдите под пользователем testread в базу данных testdb
> ```sh
> aleksandr@ubuntu2204-vm3:~$ psql -h localhost -U testread -d testdb -W
>```

### 27. Сделайте select * from testnm.t1;
> ```sh
> testdb=> select * from testnm.t1;
>```

### 28. Получилось?
> Не получилось
>
> <image src="images/sel_err.png" alt="sel_err">

### 29. Eсть идеи почему? если нет - смотрите шпаргалку
> Потому что мы удалили и создали таблицу t1 после того, как мы дали доступ на SELECT пользователю testread. 
> Т.е. после удаления таблицы t1, право на SELECT у пользователя testread пропало. Это право было для существующих на момент назначения прав таблиц.

### 30. Как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку
> Посмотрел шпаргалку
>```sh
> aleksandr@ubuntu2204-vm3:~$ sudo -u postgres psql
> postgres=# \c testdb
> testdb=# ALTER default privileges in SCHEMA testnm grant SELECT on TABLEs to readonly;
>```

> ALTER DEFAULT PRIVILEGES позволяет задавать права, применяемые к объектам, которые будут создаваться в будущем. Соответсвенно, после этой команды такое больше не повторится.

### 31. сделайте select * from testnm.t1;
>```sh
> aleksandr@ubuntu2204-vm3:~$ psql -h localhost -U testread -d testdb -W
> testdb=> select * from testnm.t1;
>```

### 32 Получилось?
> Не получилось
>
> <image src="images/sel3.png" alt="sel3">

### 33. Есть идеи почему? если нет - смотрите шпаргалку
> Потому что ALTER default будет действовать для новых таблиц а grant SELECT on all TABLEs in SCHEMA testnm TO readonly отработал только для существующих на тот момент времени. Надо сделать снова или grant SELECT или пересоздать таблицу

> Поэтому задаю GRANT SELECT еще раз:
>```sh
> aleksandr@ubuntu2204-vm3:~$ sudo -u postgres psql
> postgres=# \c testdb
> testdb=# GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;
>```

### 31. Сделайте select * from testnm.t1;
>```sh
> aleksandr@ubuntu2204-vm3:~$ psql -h localhost -U testread -d testdb -W
> testdb=# testdb=> select * from testnm.t1;
>```

### 32. получилось?
> Получилось.
> 
> <image src="images/sel_ok.png" alt="sel_ok">

### 33. Ура!

### 34. Теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
> Команды выполнились

### 35. А как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?
> Здесь тоже посмотрел шпаргалку.
> Потому что search_path указывает в первую очередь на схему public, при отсутсвии $user. А схема public создается в каждой базе данных по умолчанию. И grant на все действия в этой схеме дается роли public. 
> А роль public добавляется всем новым пользователям. Соответсвенно каждый пользователь может по умолчанию создавать объекты в схеме public любой базы данных, 
> ес-но если у него есть право на подключение к этой базе данных.

### 36. Есть идеи как убрать эти права? если нет - смотрите шпаргалку
> Посмотрел шпаргалку

### 37. Если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды
>```sh
> aleksandr@ubuntu2204-vm3:~$ psql -h localhost -U testread -d testdb -W
>```

> Надо лишить схему publiс право на CREATE схемы
>```sh
> testdb=# revoke CREATE on SCHEMA public FROM public;
>```

> Надо лишить схему publiс всех прав на БД testdb
>```sh
> testdb=# revoke all on DATABASE testdb FROM public; 
>```

>```sh
> aleksandr@ubuntu2204-vm3:~$ psql -h localhost -U testread -d testdb -W
>```

### 38. Теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);
> Команды не выполнились

### 39. Расскажите что получилось и почему.
> Команды не выполнились, т.к мы лишили схему publiс всех прав на БД testdb.