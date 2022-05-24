# pgcloud-homework02

**Домашнее задание**
Установка и настройка PostgteSQL в контейнере Docker

**Описание/Пошаговая инструкция выполнения домашнего задания:**
ДЗ по прежнему выполняется на собственной ВМ с установленным docker

Cделать каталог **/var/lib/postgres**
```
# mkdir /var/lib/postgres
```
Развернуть контейнер с PostgreSQL 14 смонтировав в него /var/lib/postgres
```
# # docker run -v /var/lib/postgres:/var/lib/postgresql/data --name jf-postgres -p 5432:5432 -e POSTGRES_PASSWORD=postgres -d postgres
fdc1ff18d1586a306e2209c62b89b9d17799be7f1b8c4ed7f9ee07c4071e43ba

# ll /var/lib/postgres/
total 136
drwx------ 19 systemd-coredump root              4096 мая 24 18:10 ./
drwxr-xr-x 47 root             root              4096 мая 24 15:48 ../
drwx------  5 systemd-coredump systemd-coredump  4096 мая 24 18:10 base/
drwx------  2 systemd-coredump systemd-coredump  4096 мая 24 18:11 global/
drwx------  2 systemd-coredump systemd-coredump  4096 мая 24 18:10 pg_commit_ts/
drwx------  2 systemd-coredump systemd-coredump  4096 мая 24 18:10 pg_dynshmem/
-rw-------  1 systemd-coredump systemd-coredump  4821 мая 24 18:10 pg_hba.conf
-rw-------  1 systemd-coredump systemd-coredump  1636 мая 24 18:10 pg_ident.conf
drwx------  4 systemd-coredump systemd-coredump  4096 мая 24 18:15 pg_logical/
drwx------  4 systemd-coredump systemd-coredump  4096 мая 24 18:10 pg_multixact/
drwx------  2 systemd-coredump systemd-coredump  4096 мая 24 18:10 pg_notify/
drwx------  2 systemd-coredump systemd-coredump  4096 мая 24 18:10 pg_replslot/
drwx------  2 systemd-coredump systemd-coredump  4096 мая 24 18:10 pg_serial/
drwx------  2 systemd-coredump systemd-coredump  4096 мая 24 18:10 pg_snapshots/
drwx------  2 systemd-coredump systemd-coredump  4096 мая 24 18:10 pg_stat/
drwx------  2 systemd-coredump systemd-coredump  4096 мая 24 18:15 pg_stat_tmp/
drwx------  2 systemd-coredump systemd-coredump  4096 мая 24 18:10 pg_subtrans/
drwx------  2 systemd-coredump systemd-coredump  4096 мая 24 18:10 pg_tblspc/
drwx------  2 systemd-coredump systemd-coredump  4096 мая 24 18:10 pg_twophase/
-rw-------  1 systemd-coredump systemd-coredump     3 мая 24 18:10 PG_VERSION
drwx------  3 systemd-coredump systemd-coredump  4096 мая 24 18:10 pg_wal/
drwx------  2 systemd-coredump systemd-coredump  4096 мая 24 18:10 pg_xact/
-rw-------  1 systemd-coredump systemd-coredump    88 мая 24 18:10 postgresql.auto.conf
-rw-------  1 systemd-coredump systemd-coredump 28835 мая 24 18:10 postgresql.conf
-rw-------  1 systemd-coredump systemd-coredump    36 мая 24 18:10 postmaster.opts
-rw-------  1 systemd-coredump systemd-coredump    94 мая 24 18:10 postmaster.pid

```
Сразу проверил, что в папке хостовой системы появились данные.

Развернул контейнер с клиентом postgres
```
# docker run --name jf-postgres-psql -e POSTGRES_PASSWORD=postgres -d postgres
1cbffbe9881079b6bced4e580be823089dfba7c3aff4296835a15e00f8e4e567
```
Подключиться из контейнера с клиентом к контейнеру с сервером и сделать
таблицу с парой строк:
```
# docker exec -ti jf-postgres-psql bash

postgres=# create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov');
CREATE TABLE
INSERT 0 1
INSERT 0 1
postgres=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)
```
Подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/Аналоги - подключусь с WSL хостовой рабочей машины, на которой работает hyper-v, на которой работает ВМ с Ubuntu ^)
```
root@JULLY:/home/ujack# psql -h 10.13.13.136 -U postgres
Password for user postgres:
psql (14.2 (Ubuntu 14.2-1ubuntu1), server 14.3 (Debian 14.3-1.pgdg110+1))
Type "help" for help.

postgres=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)
```

Удалим контейнер с сервером
```
# docker ps -a
CONTAINER ID   IMAGE      COMMAND                  CREATED         STATUS         PORTS                                       NAMES
fdc1ff18d158   postgres   "docker-entrypoint.s…"   7 minutes ago   Up 7 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   jf-postgres
1cbffbe98810   postgres   "docker-entrypoint.s…"   2 hours ago     Up 2 hours     5432/tcp                                    jf-postgres-psql
root@ubuntu-otus-admlinuxpro:/home/ujack/otus-pg-cloudsolutions/pgcloud-homework02# docker stop jf-postgres
jf-postgres
root@ubuntu-otus-admlinuxpro:/home/ujack/otus-pg-cloudsolutions/pgcloud-homework02# docker rm jf-postgres
jf-postgres

# docker ps -a
CONTAINER ID   IMAGE      COMMAND                  CREATED       STATUS       PORTS      NAMES
1cbffbe98810   postgres   "docker-entrypoint.s…"   2 hours ago   Up 2 hours   5432/tcp   jf-postgres-psql

```
Создадим его заново
```
# docker run -v /var/lib/postgres:/var/lib/postgresql/data --name jf-postgres -p 5432:5432 -e POSTGRES_PASSWORD=postgres -d postgres
04abe8a5fbe4a0c4463e17e87c21292bb86e134d37a83c34fc5fc52c01488fc9
```
Подключимся снова из контейнера с клиентом к контейнеру с сервером и
проверим, что данные остались на месте
```
# docker exec -ti jf-postgres-psql bash
root@1cbffbe98810:/# psql -h 172.17.0.2 -U postgres
Password for user postgres:
psql (14.3 (Debian 14.3-1.pgdg110+1))
Type "help" for help.

postgres=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)
```
Done.

### На чем я споткнулся:
1. Банально /var/lib/postgres монтировал в /var/lib/postgres и не видел файлов на хостовой машине.

Как решал:
запустил bash контейнера с сервером, посмотрел где PG хранит данные, как ни странно - папка **/var/lib/postgresql** , а не **/var/lib/postgre**

2. В начале проверил, что на хосте увидел папку **data**, дальше не проверял. Пересоздал сервер, данных нет. Хотя пробрасываю все правильно.

Инспект контейнера показал следующее

```
# docker inspect jf-postgres
...
"Mounts": [
            {
                "Type": "bind",
                "Source": "/var/lib/postgres",
                "Destination": "/var/lib/postgresql",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            },
            {
                "Type": "volume",
                "Name": "90324e1bbc1645c842518181b1cbe705c205a507de3d0c7849e0d2c080292127",
                "Source": "/var/lib/docker/volumes/90324e1bbc1645c842518181b1cbe705c205a507de3d0c7849e0d2c080292127/_data",
                "Destination": "/var/lib/postgresql/data",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
...
```
Т.е. /var/lib/postgresql - прокинута правильно, а вот /var/lib/postgresql/data на самом деле уже монтируется в отдельный volume
/var/lib/docker/volumes/90324e1bbc1645c842518181b1cbe705c205a507de3d0c7849e0d2c080292127/
(видимо это настройка image, на сколько я понимаю.)

3. Из лекции вынес, что имена контейнеров должны резолвится как минимум один из другого и быть доступными в пределах одного bridge. Но имена не доступны, только по ИП, не разбирался еще, оставлю на потом.
