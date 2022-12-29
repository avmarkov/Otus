# Домашняя работа № 10. Секционирование таблицы.

### 1. На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение.
> Для выполнения этой домашней работы я развернул виртуальную машину у себя с помощью VMware.
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