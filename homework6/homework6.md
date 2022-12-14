# Домашняя работа № 6. Работа с журналами.

### 1. Настройте выполнение контрольной точки раз в 30 секунд.
> Посмотрим текущий checkpoint_timeout:
> ```sh
> postgres=# show  checkpoint_timeout;
> ```
>
> Результат:
>
> <image src="images/checkpoint_timeout.png" alt="checkpoint_timeout">

> Настроим выполнение контрольной точки раз в 30 секунд:
> ```sh
> postgres=# ALTER SYSTEM SET checkpoint_timeout = '30s';
> ```
>
> Результат:
>
> <image src="images/checkpoint_timeout2.png" alt="checkpoint_timeout2">

### 2. 10 минут c помощью утилиты pgbench подавайте нагрузку.
> Сначала запомним позицию в журнале
> ```sh
> postgres=# SELECT pg_current_wal_insert_lsn();
> ```

> Позиция в журнале:
> ```sh
>  0/50D3EFF0
> ```

> Очистим статистику
> ```sh
> postgres=# SELECT pg_stat_reset_shared('bgwriter');
> ```

> Подаем нагрузку:
> ```sh
> aleksandr@ubuntu2204-vm5:~$ sudo -u postgres pgbench -c8 -P 60 -T 600 -U postgres postgres
> ```

### 3. Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.
> Находим текущую позицию в журнале
> ```sh
> postgres=# SELECT pg_current_wal_insert_lsn();
> ```

> Позиция в журнале:
> ```sh
>  0/6B9C12C8
> ```

> Находим объем журнальных файлов, который был сгенерирован за это время:
> ```sh
> postgres=# SELECT pg_size_pretty('0/6B9C12C8'::pg_lsn - '0/50D3EFF0'::pg_lsn);
> ```
> Объем журнальных файлов:
>
> <image src="images/pg_size_pretty.png" alt="pg_size_pretty">

> Количество записей в журнале:
> ```sh
> postgres=# SELECT '0/6B9C12C8'::pg_lsn - '0/50D3EFF0'::pg_lsn;
> ```
>
> <image src="images/pg_size_pretty_count.png" alt="pg_size_pretty_count">

> Находим количество контрольных точек, с последней очистки:
> ```sh
> postgres=# SELECT checkpoints_timed, checkpoints_req FROM pg_stat_bgwriter;
> ```
> <image src="images/checkpoint_count.png" alt="checkpoint_count">
>
> Количество контрольных точек получилосm 26, но их получилось больше, т.к. не смотря на выполнение их раз/30сек, перед запуском pgbench и после тоже каое-то время прошло.

### 4. Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?
> Смотрю лог-файл:
> ```sh
> aleksandr@ubuntu2204-vm5:~$ sudo grep -i "checkpoint complete" /var/log/postgresql/postgresql-14-main.log
> ```

> Результат:
>
> <image src="images/log.png" alt="log">
> Конрольные точки выполнялись примерно раз/30 сек, но не сточностью до милисекунд. Видимо это связано с процедурой autovacuum.
>
> #### Не уверен, что именно здесь нужно смотреть статистику контрольных точек, если не здесь, подскажите, пожалуйста, где? И на счет вопроса какой объем приходится в среднем на одну контрольную точку. Отсюда могу преположить что в среднем 1800-2100? Так ли это?

### 5. Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.
> Текущий режим режим:
> ```sh
> postgres=# show synchronous_commit;
> ```

> Результат:
> ```sh
>  synchronous_commit
> --------------------
>  on
> ```
>
> Т.е. у нас синхронный режим
>
> Переводим в асинхронный режим:
> ```sh
> postgres=# ALTER SYSTEM SET synchronous_commit = off;
> postgres=# SELECT pg_reload_conf();
> ```

> Запускаем pgbench:
> ```sh
> aleksandr@ubuntu2204-vm5:~$ sudo -u postgres pgbench -c8 -P 60 -T 600 -U postgres postgres
> ```
> Результат:
>
> <image src="images/pgbench2.png" alt="pgbench2">
>
> tps в асинхронном режиме - значительно выше.
>
> Асинхронная запись эффективнее синхронной — фиксация изменений не ждет записи. Однако надежность уменьшается: зафиксированные данные могут пропасть в случае сбоя, если между фиксацией и сбоем прошло менее 3 × wal_writer_delay времени (что при настройке по умолчанию составляет чуть больше полсекунды).


### 6. Создайте новый кластер с включенной контрольной суммой страниц. Создайте таблицу. Вставьте несколько значений. Выключите кластер. Измените пару байт в таблице. Включите кластер и сделайте выборку из таблицы. Что и почему произошло? как проигнорировать ошибку и продолжить работу?
> Включаем checksum:
> ```sh
> aleksandr@ubuntu2204-vm5:~$ sudo su - postgres -c '/usr/lib/postgresql/15/bin/pg_checksums --enable -D "/var/lib/postgresql/15/main"'
> ```

> Создаю БД  таблицу:
> ```sh
> postgres=# create database test_crc;
> postgres=# \c test_crc;
> test_crc=# CREATE TABLE test (
>      id     integer,                                      
> 	   name   text
>  );
> ```

> Вставляем записи в таблицу:
> ```sh
> test_crc=# INSERT INTO test values (1, 'один');
> test_crc=# INSERT INTO test values (2, 'два');
> ```

> Посмотрим где лежит таблица: 
> ```sh
> test_crc=# SELECT pg_relation_filepath('test');
> ```
>
> Здесь:
> ```sh
> base/16393/16394
> ```

> Выключил кластер
> ```sh
> aleksandr@ubuntu2204-vm5:~$ sudo pg_ctlcluster 15 main stop
> ```

> Изменил пару байт таблицы
> ```sh
> aleksandr@ubuntu2204-vm5:~$ sudo dd if=/dev/zero of=/var/lib/postgresql/15/main/base/16393/16394 oflag=dsync conv=notrunc bs=1 count=8
> ```
> Результат:
> 
> <image src="images/crc.png" alt="crc">
>
> Контрольная сумма не совпала, поэтому и ошибка

> Можно проигнорировать эту ошибку так:
> ```sh
> test_crc=# SET ignore_checksum_failure = on;
> ```
> Результат:
> 
> <image src="images/crc_ignore.png" alt="crc_ignore">