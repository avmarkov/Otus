# Домашняя работа № 2

### создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным способом
> Я создал виртуальную машину с Ubuntu 22.04.1 LTS в Яндексе. Назвал ее ubuntu2204-vm1. Скрин виртуальной машине Яндекса на картинке ниже:
> <image src="images/vm_yandex.png" alt="vm_yandex">

### Поставить на нем Docker Engine
> Поставил на нем Docker Engine
> ```sh
> aleksandr@ubuntu2204-vm1:~$ curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER
> ```

### Сделать каталог /var/lib/postgres
> Сделал
>
> <image src="images/dir_postgres.png" alt="/var/lib/postgres">

### Развернуть контейнер с PostgreSQL 14 смонтировав в него /var/lib/postgres
> Создаем docker-сеть: 
> ```sh
> sudo docker network create pg-net
> ```

> Подключаем созданную сеть к контейнеру сервера Postgres:
> ```sh
> sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14
> ```

> Результат:
>
> <image src="images/docker.png" alt="docker">

### Развернуть контейнер с клиентом postgres
> Развернул
> ```sh
> aleksandr@ubuntu2204-vm1:~$ sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-server -U postgres
> ```

### Подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк
> Подключился. Теперь у меня два контейнера. Один- с сервером. Другой - с клиентом.
> <image src="images/client.png" alt="client">

> Сделал таблицу test(id integer, name character varying);
> ```sh
> postgres=# create table test(id integer, name character varying);
> ```
> Результат:
>
><image src="images/table_test.png" alt="table_test">

### Подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера
> Подключился. У меня уже был установлен сервер Postgres на Windows машине. Поэтому я указал путь к psql -  C:\Program Files\PostgreSQL\14\bin>psql
> ```sh
> C:\Program Files\PostgreSQL\14\bin>psql -p 5432 -U postgres -h 158.160.19.119 -d postgres -W
> ```

> Результат:
>
> <image src="images/psql.png" alt="psql">

### Удалить контейнер с сервером
> Удалил
> 
> ```sh
> sudo docker stop 8bf473caf2f8
> sudo docker rm 8bf473caf2f8
> ```

> Результат:
>
> <image src="images/docker_rm.png" alt="docker_rm">


### Создать его заново
> Устанавливаю docker compose
> ```sh
> sudo apt install docker-compose -y
> ```

> При этом файл docker-compose.yml у меня такой:
> ```sh
> version: '3.1'
> services:
>   pg_db:
>     image: postgres:14
>     restart: always
>     environment:
>       - POSTGRES_PASSWORD=postgres
>       - POSTGRES_USER=postgres
>       - POSTGRES_DB=stage
>     volumes:
>       - /var/lib/postgres:/var/lib/postgresql/data
>     ports:
>       - ${POSTGRES_PORT:-5432}:5432
> ```

> Запускаю
> ```sh
> sudo docker-compose up -d
> ```

### Подключиться снова из контейнера с клиентом к контейнеру с сервером. Проверить, что данные остались на месте
> Сначала нужно создать дирректорию:
> ```sh
> sudo su
> cd /var/lib/docker/volumes/aleksandr_pg_project/_data
> ```
> Потом подключаюсь:
> ```sh
> sudo -u postgres psql -h localhost
> ```
 
> Результат:
> 
> <image src="images/res.png" alt="res">
> 
> Данные на месте


