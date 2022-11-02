# Домашняя работа № 1

### Cоздать новый проект в Google Cloud Platform, например postgres2022-, где yyyymm год, месяц вашего рождения (имя проекта должно быть уникально на уровне GCP), Яндекс облако или на любых ВМ, докере.
### Далее создать инстанс виртуальной машины с дефолтными параметрами

> Я создал виртуальную машину с Ubuntu 22.04.1 LTS в Яндексе. Назвал ее ubuntu2204-vm1. Скрин виртуальной машине Яндекса на картинке ниже:

> <image src="images/vm_yandex.png" alt="Виртуальная машина Яндекс">

### Добавить свой ssh ключ в metadata ВМ

> Генерирую ключ у себя на машине с Windows:
> ```sh
> PS C:\Users\Aleksandr_Markov> ssh-keygen -t rsa -b 2048
> ```

> Задаю путь и имя ключа:
> ```sh
> c:\Users\Aleksandr_Markov\.ssh\yc_key
> ```

> Вывожу на экран публичный ключ:
> ```sh
> cat C:\Users\Aleksandr_Markov\.ssh\yc_key.pub
> ```

> Результат:
> <image src="images/gen_rsa.png" alt="RSA">

> Затем скопировал публичный ключ и добавил его в Яндекс

### Зайти удаленным ssh (первая сессия), не забывайте про ssh-add
> ```sh
> PS C:\Users\Aleksandr_Markov> ssh -i C:\Users\Aleksandr_Markov\.ssh\yc_key aleksandr@51.250.27.132
> ```
> 
> Результат:
> 
> <image src="images/connect_to_vm.png" alt="connect_to_vm">

### Поставить PostgreSQL

> Т.к. версия Ubuntu 22.04, устанавливаю PostgreSQL так (знаю, что установится 14-я версия):
> ```sh
> aleksandr@ubuntu2204-vm1:~$ sudo apt-get -y install postgresql
> ```
 
> Результат:
> 
> <image src="images/postgres_install.png" alt="postgres_install">

### Зайти вторым ssh (вторая сессия). Запустить везде psql из под пользователя postgres

> Зашел вторым ssh
> Запустил psql из под пользователя postgres в этих двух клиентах:
> ```sh
> aleksandr@ubuntu2204-vm1:~$ sudo -u postgres psql
> ```
 
> Результат:
> 
> <image src="images/psql.png" alt="psql">

### Выключить auto commit
> ```sh
> postgres=# \set AUTOCOMMIT OFF
> ```


### Cделать в первой сессии новую таблицу и наполнить ее данными.
#### create table persons(id serial, first_name text, second_name text);
#### insert into persons(first_name, second_name) values('ivan', 'ivanov');
#### insert into persons(first_name, second_name) values('petr', 'petrov'); 
#### commit; 

посмотреть текущий уровень изоляции: show transaction isolation level

начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции

в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');

сделать select * from persons во второй сессии

видите ли вы новую запись и если да то почему?

завершить первую транзакцию - commit;

сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?

завершите транзакцию во второй сессии

начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
завершить первую транзакцию - commit;
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
завершить вторую транзакцию
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему? ДЗ сдаем в виде миниотчета в markdown в гите

