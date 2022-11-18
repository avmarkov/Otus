# Домашняя работа № 5. Настройка autovacuum с учетом оптимальной производительности.

### Cоздать GCE инстанс типа e2-medium и диском 10GB
> Я создал виртуальную машину с Ubuntu 22.04.1 LTS в Яндексе. Назвал ее ubuntu2204-vm3. Скрин виртуальной машине Яндекса на картинке ниже:
> <image src="images/vm_yandex.png" alt="vm_yandex">

### Установить на него PostgreSQL 14 с дефолтными настройками
> Т.к. версия Ubuntu 22.04, устанавливаю PostgreSQL так (знаю, что установится 14-я версия):
> ```sh
> aleksandr@ubuntu2204-vm5:~$ sudo apt-get -y install postgresql
> ```
> Кластер создался:
> 
> <image src="images/cluster_status.png" alt="cluster_status">

### Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
> ```sh
> max_connections = 40
> shared_buffers = 1GB
> effective_cache_size = 3GB
> maintenance_work_mem = 512MB
> checkpoint_completion_target = 0.9
> wal_buffers = 16MB
> default_statistics_target = 500
> random_page_cost = 4
> effective_io_concurrency = 2
> work_mem = 6553kB
> min_wal_size = 4GB
> max_wal_size = 16GB
> ```

> Настроил:
> 
> <image src="images/settings.png" alt="settings">

### Выполнить pgbench -i postgres
> ```sh
> aleksandr@ubuntu2204-vm5:~$ sudo -u postgres pgbench -i postgres
> ```
>
> Результат:
>
> <image src="images/pgbench.png" alt="pgbench">

### Запустить pgbench -c8 -P 60 -T 600 -U postgres postgres
> ```sh
> aleksandr@ubuntu2204-vm5:~$ sudo -u postgres pgbench -c8 -P 60 -T 600 -U postgres postgres
> ```

### Дать отработать до конца
> Результат:
> 
> <image src="images/pgbench_res.png" alt="pgbench_res">

> При этом, настройки autovacuum были такие:
>
> <image src="images/autovacuum1.png" alt="autovacuum1">

### Дальше настроить autovacuum максимально эффективно
> Настроил autovacuum так:
> ```sh
> postgres=# ALTER SYSTEM SET log_autovacuum_min_duration=0;
> postgres=# ALTER SYSTEM SET autovacuum_max_workers = 10;
> postgres=# ALTER SYSTEM SET autovacuum_naptime = 15;
> postgres=# ALTER SYSTEM SET autovacuum_vacuum_threshold = 25;
> postgres=# ALTER SYSTEM SET autovacuum_vacuum_scale_factor = 0.05;
> postgres=# ALTER SYSTEM SET autovacuum_vacuum_cost_delay = 10;
> postgres=# ALTER SYSTEM SET autovacuum_vacuum_cost_limit = 1000;
> ```
>
> <image src="images/autovacuum2.png" alt="autovacuum2">
>
> Результат:
>
> <image src="images/pgbench_res2.png" alt="pgbench_res2">

### Построить график по получившимся значениям так чтобы получить максимально ровное значение tps
> График до настройки autovacuum:
>
> <image src="images/before.png" alt="before">

> График после настройки autovacuum:
>
> <image src="images/after.png" alt="after">