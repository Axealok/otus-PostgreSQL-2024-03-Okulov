# Логический бакап

Разворачиваем ВМ, устанавливаем PG

Создаем таблицу
```
postgres=# create database otus;
CREATE DATABASE
postgres=# \c otus
You are now connected to database "otus" as user "postgres".
otus=# create table students as select generate_series(1, 100) as id, md5(random()::text)::char(10) as fio;
SELECT 100

```

Создаем папку для бакапов и назначаем права

```
okulovan@otus-vm:~$ sudo mkdir /mnt/backup
kulovan@otus-vm:~$ sudo chown postgres:postgres /mnt/backup

```

Сделаем логический бэкап используя утилиту COPY

```
okulovan@otus-vm:~$ sudo -u postgres psql
psql (15.7 (Ubuntu 15.7-1.pgdg22.04+1))
Type "help" for help.

postgres=# \c otus
You are now connected to database "otus" as user "postgres".
otus=# \copy students to '/mnt/backup/students_copy.sql';
COPY 100

```

Создаем вторую таблицу и восстанавливаем данные из бакапа сделанного из 1-й таблицы

```
otus=# create table students2 (i int, fio text);
CREATE TABLE
otus=# \copy students2 from '/mnt/backup/students_copy.sql';
COPY 100
otus=# select * from students2 limit(10);
 i  |    fio     
----+------------
  1 | 47221ac6b0
  2 | e0e1daa11d
  3 | 4fcd64602a
  4 | c8cfe85923
  5 | 060dfa9ac4
  6 | b1a8fb8a63
  7 | a387960274
  8 | 27ed2e2d12
  9 | 03f234e6b9
 10 | d01d2a5bc8
(10 rows)

otus=# \dt
           List of relations
 Schema |   Name    | Type  |  Owner   
--------+-----------+-------+----------
 public | students  | table | postgres
 public | students2 | table | postgres
(2 rows)

```
Создаем бакап базы otus
```
root@otus-vm:/mnt/backup# sudo -u postgres pg_dump -d otus --create -Fc > /mnt/backup/otus_dump.gz 

```

Создаем БД otus2 и восстанавливаем в нее таблицу students2 

```
root@otus-vm:/mnt/backup# sudo -u postgres createdb otus2 && sudo -u postgres pg_restore -d otus2 -t students2 /mnt/backup/otus_dump.gz
root@otus-vm:/mnt/backup# sudo -u postgres psql
psql (15.7 (Ubuntu 15.7-1.pgdg22.04+1))
Введите "help", чтобы получить справку.

postgres=# \l
                                                  Список баз данных
    Имя    | Владелец | Кодировка | LC_COLLATE  |  LC_CTYPE   | локаль ICU | Провайдер локали |     Права доступа     
-----------+----------+-----------+-------------+-------------+------------+------------------+-----------------------
 otus      | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |            | libc             | 
 otus2     | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |            | libc             | 
 postgres  | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |            | libc             | 
 sbtest    | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |            | libc             | =Tc/postgres         +
           |          |           |             |             |            |                  | postgres=CTc/postgres+
           |          |           |             |             |            |                  | sbtest=CTc/postgres
 template0 | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |            | libc             | =c/postgres          +
           |          |           |             |             |            |                  | postgres=CTc/postgres
 template1 | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |            | libc             | =c/postgres          +
           |          |           |             |             |            |                  | postgres=CTc/postgres
(6 строк)

postgres=# \c otus2
Вы подключены к базе данных "otus2" как пользователь "postgres".
otus2=# \dt
            Список отношений
 Схема  |    Имя    |   Тип   | Владелец 
--------+-----------+---------+----------
 public | students2 | таблица | postgres
(1 строка)

otus2=# select count(*) from students2;
 count 
-------
   100
(1 строка)

```
