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
sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14
> ```sh

> Результат:
>
> <image src="images/docker.png" alt="docker">
### развернуть контейнер с клиентом postgres

### подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк

### подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера

### удалить контейнер с сервером

### создать его заново

### подключится снова из контейнера с клиентом к контейнеру с сервером

### проверить, что данные остались на месте

### оставляйте в ЛК ДЗ комментарии что и как вы делали и как боролись с проблемами