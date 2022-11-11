# Домашняя работа № 3. Установка и настройка PostgreSQL.

### создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным способом
> Я создал виртуальную машину с Ubuntu 22.04.1 LTS в Яндексе. Назвал ее ubuntu2204-vm2. Скрин виртуальной машине Яндекса на картинке ниже:
> <image src="images/vm_yandex.png" alt="vm_yandex">

### Поставьте на нее PostgreSQL 14 через sudo apt. Проверьте что кластер запущен через sudo -u postgres pg_lsclusters
> Т.к. версия Ubuntu 22.04, устанавливаю PostgreSQL так (знаю, что установится 14-я версия):
> ```sh
> aleksandr@ubuntu2204-vm2:~$ sudo apt-get -y install postgresql
> ```
> Результат:
> 
> <image src="images/postgres_cluster.png" alt="postgres_cluster">

### Зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым
> ```sh
> postgres=# create table test(c1 text);
> postgres=# insert into test values('1');
> \q
> ```
> Результат:
> 
> <image src="images/res_table_create.png" alt="res_table_create">

### Остановите postgres например через sudo -u postgres pg_ctlcluster 14 main stop
> ```sh
> aleksandr@ubuntu2204-vm2:~$ sudo -u postgres pg_ctlcluster 14 main stop
> ```
> Результат:
> 
> <image src="images/stop_cluster.png" alt="stop_cluster">

### Создайте новый standard persistent диск GKE через Compute Engine -> Disks в том же регионе и зоне что GCE инстанс размером например 10GB. Добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk
> Создал диск, добавил к виртуальной машине
> 
> Результат:
> 
> <image src="images/disk2.png" alt="disk2">

> В виртуалке этот новый диск, обозначен как "vdb":
> 
> ```sh
> aleksandr@ubuntu2204-vm2:~$ sudo lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT,LABEL
> ```
>
> <image src="images/disk2_vm.png" alt="disk2_vm">

### Проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux
> Устанавливаем утилиту parted
> 
> ```sh
> aleksandr@ubuntu2204-vm2:~$ sudo apt install parted
> ```

> Найдем диски с отсутствующими схемами разбиения
> ```sh
> aleksandr@ubuntu2204-vm2:~$ sudo parted -l | grep Error
> ```
> Результат:
> 
> <image src="images/disk_part_error.png" alt="disk_part_error">

> Указываю используемый стандарт разбиения:
> ```sh
> aleksandr@ubuntu2204-vm2:~$ sudo parted /dev/vdb mklabel gpt
> ```

> Создаю раздел:
> ```sh
> aleksandr@ubuntu2204-vm2:~$ sudo parted -a opt /dev/vdb mkpart primary ext4 0% 100%
> ```
>
> Результат:
> 
> <image src="images/vdb1.png" alt="vdb1">

> Создаю файловую систему в новом разделе
> ```sh
> aleksandr@ubuntu2204-vm2:~$ sudo mkfs.ext4 -L datapartition /dev/vdb1
> ```
>
> Задаю Label:
> ```sh
> aleksandr@ubuntu2204-vm2:~$ sudo e2label /dev/vdb1 newdisk
> ```
>
> Результат:
> 
> <image src="images/new_file_system.png" alt="new_file_system">

> Создаем дирректорию, куда будет монтироваться диск:
> ```sh
> aleksandr@ubuntu2204-vm2:~$ sudo mkdir -p /mnt/data
> ```

> Монтируем файловую систему
> ```sh
> aleksandr@ubuntu2204-vm2:~$ sudo mount -o defaults /dev/vdb1 /mnt/data
> ```

> Редактируем файл /etc/fstab, чтобы файла система монтировалась автоматически при каждой загрузке сервера
> ```sh
> aleksandr@ubuntu2204-vm2:~$ sudo nano /etc/fstab
> ```
>
> Добавляем в конец файла новый диск:
>
> <image src="images/add_to_fstab.png" alt="add_to_fstab">
>
> Результат:
>
> <image src="images/mounted_ok.png" alt="mounted_ok">

### Перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)
> Перезагрузил. Диск остался примонтированным
>
> <image src="images/mounted_ok_after_restor.png" alt="mounted_ok_after_restor">

### Сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/
> Сделал:
> ```sh
> aleksandr@ubuntu2204-vm2:~$ sudo chown -R postgres:postgres /mnt/data/
> ```

### Перенесите содержимое /var/lib/postgres/14 в /mnt/data - mv /var/lib/postgresql/14 /mnt/data
> Перенес:
> ```sh
> aleksandr@ubuntu2204-vm2:~$ sudo mv /var/lib/postgresql/14 /mnt/data
> ```
>
> <image src="images/move_dir.png" alt="move_dir">

### Попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start. Напишите получилось или нет и почему
> ```sh
> aleksandr@ubuntu2204-vm2:~$ sudo -u postgres pg_ctlcluster 14 main start
> ```
>
> Не получилось. Я думаю, из-за того, что содержимое /var/lib/postgresql/14/main я перенес в другую дирректорию. И дирректория /var/lib/postgresql/14/main - пустая

### Задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/14/main который надо поменять и поменяйте его. Напишите что и почему поменяли
> Поменял файл postgresql.conf. Заменил параметр "data_directory" на новою дирректорию "/mnt/data/14/main".
>
> <image src="images/postgresql_conf.png" alt="postgresql_conf">

### Попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start. Напишите получилось или нет и почему
> Пытаюсь запустить. 
>
> Результат такой:
>
> <image src="images/res_cluster.png" alt="res_cluster">
> 
> Т.е. кластер не запущен. При этом через psql я зашел. И вижу свои таблицы.
> $ \ color {red} {Почему не понимаю, подскажите пожалуйста.} $


### зайдите через через psql и проверьте содержимое ранее созданной таблицы
> Таблицы имеются:
>
> <image src="images/res_table_end.png" alt="res_table_end">
