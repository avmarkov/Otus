# Домашняя работа № 1

Cоздать новый проект в Google Cloud Platform, например postgres2022-, где yyyymm год, месяц вашего рождения (имя проекта должно быть уникально на уровне GCP), Яндекс облако или на любых ВМ, докере.
Далее создать инстанс виртуальной машины с дефолтными параметрами

> Я создал виртуальную машину с Ubuntu 22.04.1 LTS в Яндексе. Назвал ее ubuntu2204-vm1. Скрин виртуальной машине Яндекса на картинке ниже:

> <image src="images/vm_yandex.png" alt="Виртуальная машина Яндекс">

добавить свой ssh ключ в metadata ВМ

> Генерирую ключ у себя на машине с Windows:
> ```sh
> PS C:\Users\Aleksandr_Markov> ssh-keygen -t rsa -b 2048
> ```
> ```sh

Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\Aleksandr_Markov/.ssh/id_rsa): c:\Users\Aleksandr_Markov\.ssh\yc_key
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in c:\Users\Aleksandr_Markov\.ssh\yc_key.
Your public key has been saved in c:\Users\Aleksandr_Markov\.ssh\yc_key.pub.
The key fingerprint is:
SHA256:NrdTeg97+EJJLMToZXztgFENhjRt5o7YcjrpdrtinAo aleksandr_markov@LYMAR
The key's randomart image is:
+---[RSA 2048]----+
|         =+*++   |
|        . Oo* o  |
|       . + * o   |
|        . . + .  |
|        So.=..   |
|       .oo+++    |
|    E  . *+.o.   |
|     .  X .oo+.  |
|      .=.+oo.+o  |
+----[SHA256]-----+
PS C:\Users\Aleksandr_Markov> cat C:\Users\Aleksandr_Markov\.ssh\yc_key.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDL9V6/DesZNSRLRkv7ymWGi1oLNoZRwgidOwPECuvu1oAMgyOs/SfO1Oie+o/bqEpi7Nc+3CuqnKajpDYSRQmxrNuXGSHUdysTgzH0cBdqf9VP2BSBm8i6/pGmpErmmE+U9hRfPC+hTp0AIXJIP6cHkbM28kEVpu3JHkyFefuczReLCrXvNrhFMclToCXISylmG2SX/sUae7MKeg3JqiqmRYlmfwC7WvKJRxsovVYzu80K3X6QJU/RrTc+jrq6XR7CXFWTyFo4FCFy7aZbqUeMXxiV8DIQd467NqVIUh9RolhFj33WQR+gqb/V33EXAniRtzacdSFdEW91YkMGQGRp aleksandr_markov@LYMAR
PS C:\Users\Aleksandr_Markov>
```





зайти удаленным ssh (первая сессия), не забывайте про ssh-add

поставить PostgreSQL

зайти вторым ssh (вторая сессия)

запустить везде psql из под пользователя postgres

выключить auto commit

сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;

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

