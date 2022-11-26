# Домашняя работа № 7. Механизм блокировок.

### 1. Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.
> Для выполнения этой домашней работы я развернул виртуальную машину у себя с помощью VMware.

> Посмотрим текущее значение таймаута
> ```sql
> postgres=# SHOW deadlock_timeout;
> ```
>
> Результат:
>
> <image src="images/deadlock_timeout.png" alt="deadlock_timeout">

> Поменяем deadlock_timeout
> ```sql
> postgres=# ALTER SYSTEM SET deadlock_timeout ='200ms';
> ```

> Включим запись в лог, в случае, если транзакция ждала дольше, чем deadlock_timeout:
> ```sql
> postgres=# ALTER SYSTEM SET log_lock_waits = on;
> postgres=# SELECT pg_reload_conf();
> ```
> Результат:
>
> <image src="images/deadlock_timeout2.png" alt="deadlock_timeout2">

> Воспроизведем ситуацию с записью в журнал. Я создал таблицу lock_table(field1 integer), вставил одно значение = 1. И запустил две транзакции в двух разных сессиях, которые обновляли единственное значение этoго поля.
> Соответсвенно, вторая транзакция ожидала завершения первой:
>
> <image src="images/lock1.png" alt="lock1">

> В логе появились следующие записи о блокировки:
>
> <image src="images/log_lock.png" alt="log_lock">

### 2. Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. Пришлите список блокировок и объясните, что значит каждая.
> Создадим БД locks, таблицу accounts в этой БД и вставим в таблицу несколько значений:
> ```sql
> postgres=# create database locks;
> postgres=# \c locks
> 
> locks=# CREATE TABLE accounts(
>   acc_no integer PRIMARY KEY,
>   amount numeric
> );
> 
> locks=# INSERT INTO accounts VALUES (1,1000.00), (2,2000.00), (3,3000.00);
> ```

> Создадим представления над pg_locks для получение необходимой информации о блокироваках:
> ```sql
> CREATE VIEW locks_v AS
> SELECT pid,
>        locktype,
>        CASE locktype
>          WHEN 'relation' THEN relation::regclass::text
>          WHEN 'transactionid' THEN transactionid::text
>          WHEN 'tuple' THEN relation::regclass::text||':'||tuple::text
>        END AS lockid,
>        mode,
>        granted
> FROM pg_locks
> WHERE locktype in ('relation','transactionid','tuple')
> AND (locktype != 'relation' OR relation = 'accounts'::regclass);
> ```

> Начнем первую транзакцию 
> ```sql
> BEGIN;
> SELECT txid_current(), pg_backend_pid();
> ```
>
> <image src="images/tr1_bk_n.png" alt="tr1_bk_n">

> Обновим строку в этой транзакции.
> ```sql
> UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 1;
> SELECT * FROM locks_v WHERE pid = 474;
> ```
> <image src="images/tr1_lock.png" alt="tr1_lock">
>
> Первая транзакция обновляет и, соответственно, блокирует строку, т.е. она удерживает блокировку строки и собственного номера

### 3. Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?

### 4. Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?
> Могут, например, я это воспроизвел в первом пункте этой домашней работы. Две транзакции выполняют единственную команду UPDATE одной и той же таблицы (без where).
> Но запись там всего одна, и они обновляют одну и туже запись, поэтому вторая транзакция блокируется и ожидает выполнение первой.
> Подкорретируйте, пожалуйста, если я неправильно рассуждаю, уж слишком просто получается)
>
> <image src="images/lock1.png" alt="lock1">