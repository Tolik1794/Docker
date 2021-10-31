#DockerCompose php-8.0, nginx, postgresql-12, mariadb-10.3

Для использования нам понадобятся:

- [Docker](https://docs.docker.com/install/)
- [Docker Compose](https://docs.docker.com/compose/install/)

Рядом с `docker-compose.yml` нужно создать `.env` файл с переменными 
(пример можно посмотреть [ExampleEnv](.env.example)):

>- `MARIA_FOLDER` - путь к месту, где будет храниться файлы таблиц `MariaDB`
>- `MARIA_DUMP_FOLDER` - путь к папке, где будут находиться файлы бекапов для БД,
путь в контейнере `/home/dump`
>- `PGSQL_FOLDER`, `PGSQL_DUMP_FOLDER` - аналогичные настройки для `PostgreSql`
>- `PROJECTS_FOLDER` - путь к папке с проектами

После установки просто запустите следующую команду в терминале в корневом каталоге, 
где находится файл `docker-compose.yml`.

>docker-compose up -d --build

Если будет конфликт порта 80, нужно завершить процессы связанные с ним, скорее всего это
установленный на локале `nginx`, его можно завершить командой:

```shell
sudo killall nginx
```

##PHP

При входе в контейнер php-fpm дефолтная папка - `/var/www`:
```shell
docker exec -it php-fpm bash
```

##NGINX

Для добавления нового проекта, нужно добавить `nginx` config в папку `nginx/sites`
в папке Docker

##DATABASES

Для того чтобы развернуть бекап базы, нужно войти внутрь контейнера нужной бд
(PostgreSql > `pgsql`, MariaDb > `mariadb`)

Для `MariaDb`:

```shell
docker exec -it mariadb bash
mysql -u root -p
create database DBNAME;
gunzip -c /home/dump/DBFILE.sql.gz | mysql -u root -p DBNAME
```

Для `PostgreSql`:

```shell
docker exec -it pgsql bash
gunzip -c /home/dump/DBFILE.tar.gz | psql -U root DBNAME
```

Для подключения изнутри php-fpm контейнера нужно использовать в качестве хоста
название контейнера с нужной бд, например:

Пример подключение `PostgreSql` в .env файле symfony 5.2

```dotenv
DATABASE_URL=postgresql://root:password@pgsql:5432/tetradka_local?serverVersion=12&charset=utf8
```

Пример подключение `MariaDb` в yml файле symfony 1.4

```yaml
doctrine:
    class: sfDoctrineDatabase
    param:
      dsn: mysql:host=mariadb;dbname=tetradka_local;port=3306
      username: root
      password: password
      encoding: utf8mb4
      attributes:
        use_native_enum: true
        use_dql_callbacks: true
```

Для подключения к базам с локали:
```
host: localhost
#pgsql
port: 15432
#mariadb
port: 15433
```
