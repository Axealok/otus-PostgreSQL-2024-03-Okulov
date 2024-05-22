# Репликация
## Подготовка тестового стенда
Для тестов использую Развернуто 4 ВМ с идентичными параметрами
Развернул 1-ю ВМ установил PG 15
Изменил pg_hba.conf и listen_address чтоб кластера видели друг друга
Клонировал 1-ю ВМ в 2, 3 и 4 ВМ
На клонированных ВМ пришлось пересоздать пароль для УЗ  postgres,  иначе не проходила проверка подлинности по паролю

```
psql: error: подключиться к серверу "otus-vm" (192.168.230.94), порту 5432 не удалось: ВАЖНО:  пользователь "postgres" не прошёл проверку подлинности (по паролю)
подключиться к серверу "otus-vm" (192.168.230.94), порту 5432 не удалось: ВАЖНО:  пользователь "postgres" не прошёл проверку подлинности (по паролю)

```

На 1 ВМ создаем таблицы test и наполняем записями и пустую test2.

```
ostgres=# show wal_level; 
 wal_level 
-----------
 replica
(1 row)

postgres=# \l
                                                 List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges   
-----------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
 postgres  | postgres | UTF8     | ru_RU.UTF-8 | ru_RU.UTF-8 |            | libc            | 
 template0 | postgres | UTF8     | ru_RU.UTF-8 | ru_RU.UTF-8 |            | libc            | =c/postgres          +
           |          |          |             |             |            |                 | postgres=CTc/postgres
 template1 | postgres | UTF8     | ru_RU.UTF-8 | ru_RU.UTF-8 |            | libc            | =c/postgres          +
           |          |          |             |             |            |                 | postgres=CTc/postgres
(3 rows)

postgres=# create database otus;
CREATE DATABASE
postgres=# \c otus
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
You are now connected to database "otus" as user "postgres".
otus=# create table test as select generate_series(1, 10) as id, md5(random()::text)::char(10) as fio;
SELECT 10
otus=# select * from test;
 id |    fio     
----+------------
  1 | deb68ee844
  2 | 2c780db929
  3 | 83a5cbb7fc
  4 | f7ca8d61df
  5 | b2eb0d72bb
  6 | 5f769aa9be
  7 | da566219df
  8 | 1fd0952b7d
  9 | 9d8cc05873
 10 | 1f0125b8b4
(10 rows)
otus=# create table test2 (id int, fio text);
CREATE TABLE
otus=# \dt
         List of relations
 Schema | Name  | Type  |  Owner   
--------+-------+-------+----------
 public | test  | table | postgres
 public | test2 | table | postgres
(2 rows)

```

Аналогично на 2 ВМ создаем таблицы test2 для записи, test для запросов на чтение.

```
okulovan@b1-t-oan:~$ sudo -u postgres psql -h otus-vm-2
[sudo] пароль для okulovan: 
could not change directory to "/home/okulovan": Отказано в доступе
Password for user postgres: 
psql (15.7 (Ubuntu 15.7-1.pgdg22.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.

postgres=# ^C
postgres=# create database otus;
CREATE DATABASE
postgres=# \c otus
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
You are now connected to database "otus" as user "postgres".
otus=# create table test2 as select generate_series(1, 10) as id, md5(random()::text)::char(10) as fio;
SELECT 10
otus=# create table test (id int, fio text);
CREATE TABLE
otus=# \dt
         List of relations
 Schema | Name  | Type  |  Owner   
--------+-------+-------+----------
 public | test  | table | postgres
 public | test2 | table | postgres
(2 rows)

```
Создаем публикацию таблицы test и подписываемся на публикацию таблицы test2 с ВМ №2.

```
otus=# alter system set wal_level = logical;
ALTER SYSTEM
otus=# \q
okulovan@b1-t-oan:~$ ssh okulovan@otus-vm
okulovan@otus-vm's password: 
Last login: Wed May 22 09:29:37 2024 from 192.168.230.14
okulovan@otus-vm:~$ sudo pg_ctlcluster 15 main restart
[sudo] пароль для okulovan: 
okulovan@otus-vm:~$ sudo -u postgres psql
psql (15.7 (Ubuntu 15.7-1.pgdg22.04+1))
Type "help" for help.

postgres=# show wal_level;
 wal_level 
-----------
 logical
(1 row)

postgres=# \c otus
You are now connected to database "otus" as user "postgres".
otus=# CREATE PUBLICATION test_pub FOR TABLE test;
CREATE PUBLICATION
otus=# \dRp+
                            Publication test_pub
  Owner   | All tables | Inserts | Updates | Deletes | Truncates | Via root 
----------+------------+---------+---------+---------+-----------+----------
 postgres | f          | t       | t       | t       | t         | f
Tables:
    "public.test"

otus=# create subscription test2_sub connection 'host=otus-vm-2 port=5432 user=postgres password=postgres dbname=otus' publication test2_pub with (copy_data = true);
ЗАМЕЧАНИЕ:  на сервере публикации создан слот репликации "test2_sub"
CREATE SUBSCRIPTION
otus=# \dRp+
                            Publication test_pub
  Owner   | All tables | Inserts | Updates | Deletes | Truncates | Via root 
----------+------------+---------+---------+---------+-----------+----------
 postgres | f          | t       | t       | t       | t         | f
Tables:
    "public.test"

otus=# \dRs
            List of subscriptions
   Name    |  Owner   | Enabled | Publication 
-----------+----------+---------+-------------
 test2_sub | postgres | t       | {test2_pub}
(1 row)

otus=# select * from test;
 id |    fio     
----+------------
  1 | deb68ee844
  2 | 2c780db929
  3 | 83a5cbb7fc
  4 | f7ca8d61df
  5 | b2eb0d72bb
  6 | 5f769aa9be
  7 | da566219df
  8 | 1fd0952b7d
  9 | 9d8cc05873
 10 | 1f0125b8b4
(10 rows)

otus=# select * from test2;
 id |    fio     
----+------------
  1 | 72b7cca521
  2 | 6f6cced064
  3 | 6771130cfc
  4 | b66e47b796
  5 | 21a3cd1f23
  6 | 4fb1185103
  7 | b891f2c453
  8 | 62795f6a5b
  9 | 93ef839629
 10 | cc179fd723
(10 rows)
```

Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test1 с ВМ №1.

```
okulovan@b1-t-oan:~$ ssh okulovan@otus-vm
okulovan@otus-vm's password: 
Last login: Tue May 21 16:36:13 2024 from 192.168.230.14
okulovan@otus-vm:~$ sudo -u postgres psql
[sudo] пароль для okulovan: 
psql (15.7 (Ubuntu 15.7-1.pgdg22.04+1))
Type "help" for help.

postgres=# alter system set wal_level = logical;
ALTER SYSTEM
postgres=# \q
okulovan@otus-vm-2:~$ sudo pg_ctlcluster 15 main restart
[sudo] пароль для okulovan: 
okulovan@otus-vm-2:~$ sudo -u postgres psql
psql (15.7 (Ubuntu 15.7-1.pgdg22.04+1))
Type "help" for help.

postgres=# show wal_level;
 wal_level 
-----------
 logical
(1 row)

otus=# create subscription test_sub connection 'host=otus-vm port=5432 user=postgres password=postgres dbname=otus' publication test_pub with (copy_data = true);
ЗАМЕЧАНИЕ:  на сервере публикации создан слот репликации "test_sub"
CREATE SUBSCRIPTION
otus=# \dRs
            List of subscriptions
   Name   |  Owner   | Enabled | Publication 
----------+----------+---------+-------------
 test_sub | postgres | t       | {test_pub}
(1 row)

otus=# CREATE PUBLICATION test2_pub FOR TABLE test2;
CREATE PUBLICATION
otus=# \dRs
            List of subscriptions
   Name   |  Owner   | Enabled | Publication 
----------+----------+---------+-------------
 test_sub | postgres | t       | {test_pub}
(1 row)

otus=# \dRp+
                           Publication test2_pub
  Owner   | All tables | Inserts | Updates | Deletes | Truncates | Via root 
----------+------------+---------+---------+---------+-----------+----------
 postgres | f          | t       | t       | t       | t         | f
Tables:
    "public.test2"

otus=# select * from test;
 id |    fio     
----+------------
  1 | deb68ee844
  2 | 2c780db929
  3 | 83a5cbb7fc
  4 | f7ca8d61df
  5 | b2eb0d72bb
  6 | 5f769aa9be
  7 | da566219df
  8 | 1fd0952b7d
  9 | 9d8cc05873
 10 | 1f0125b8b4
(10 rows)

otus=# select * from test2;
 id |    fio     
----+------------
  1 | 72b7cca521
  2 | 6f6cced064
  3 | 6771130cfc
  4 | b66e47b796
  5 | 21a3cd1f23
  6 | 4fb1185103
  7 | b891f2c453
  8 | 62795f6a5b
  9 | 93ef839629
 10 | cc179fd723
(10 rows)
```
3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ).

```
okulovan@b1-t-oan:~$ ssh okulovan@otus-vm-3
Last login: Tue May 14 10:56:57 2024 from 192.168.230.14
okulovan@otus-vm-3:~$ sudo -u postgres psql
psql (15.7 (Ubuntu 15.7-1.pgdg22.04+1))
Type "help" for help.

postgres=# alter user postgres password 'postgres';
ALTER ROLE
postgres=# alter system set wal_level = logical;
ALTER SYSTEM
postgres=# \q
okulovan@otus-vm-3:~$ sudo pg_ctlcluster 15 main restart
okulovan@otus-vm-3:~$ sudo -u postgres psql
psql (15.7 (Ubuntu 15.7-1.pgdg22.04+1))
Type "help" for help.

postgres=# create database otus;
CREATE DATABASE
postgres=# create table test (id int, fio text);
CREATE TABLE
postgres=# create table test2 (id int, fio text);
CREATE TABLE
postgres=# create subscription test_sub connection 'host=otus-vm port=5432 user=postgres password=postgres dbname=otus' publication test_pub with (copy_data = true);
ОШИБКА:  не удалось создать слот репликации "test_sub": ОШИБКА:  слот репликации "test_sub" уже существует
postgres=# create subscription test_sub3 connection 'host=otus-vm port=5432 user=postgres password=postgres dbname=otus' publication test_pub with (copy_data = true);
ЗАМЕЧАНИЕ:  на сервере публикации создан слот репликации "test_sub3"
CREATE SUBSCRIPTION
postgres=# create subscription test2_sub3 connection 'host=otus-vm-2 port=5432 user=postgres password=postgres dbname=otus' publication test2_pub with (copy_data = true);
ЗАМЕЧАНИЕ:  на сервере публикации создан слот репликации "test2_sub3"
CREATE SUBSCRIPTION
postgres=# \dRs
             List of subscriptions
    Name    |  Owner   | Enabled | Publication 
------------+----------+---------+-------------
 test2_sub3 | postgres | t       | {test2_pub}
 test_sub3  | postgres | t       | {test_pub}
(2 rows)

postgres=# select * from test;
 id |    fio     
----+------------
  1 | deb68ee844
  2 | 2c780db929
  3 | 83a5cbb7fc
  4 | f7ca8d61df
  5 | b2eb0d72bb
  6 | 5f769aa9be
  7 | da566219df
  8 | 1fd0952b7d
  9 | 9d8cc05873
 10 | 1f0125b8b4
(10 rows)

postgres=# select * from test2;
 id |    fio     
----+------------
  1 | 72b7cca521
  2 | 6f6cced064
  3 | 6771130cfc
  4 | b66e47b796
  5 | 21a3cd1f23
  6 | 4fb1185103
  7 | b891f2c453
  8 | 62795f6a5b
  9 | 93ef839629
 10 | cc179fd723
(10 rows)

``
При создании подписок забыл подключитьсясоздать базу otus. Копии баз создались в  postgres

Пришлось исправиться

```
postgres=# \c otus
You are now connected to database "otus" as user "postgres".
otus=# \dt
Did not find any relations.
otus=# \q
okulovan@otus-vm-3:~$ sudo -u postgres psql
could not change directory to "/home/okulovan": Отказано в доступе
psql (15.7 (Ubuntu 15.7-1.pgdg22.04+1))
Type "help" for help.

postgres=# drop subscription test_sub3;
ЗАМЕЧАНИЕ:  слот репликации "test_sub3" удалён на сервере репликации
DROP SUBSCRIPTION
postgres=# drop subscription test2_sub3;
ЗАМЕЧАНИЕ:  слот репликации "test2_sub3" удалён на сервере репликации
DROP SUBSCRIPTION
postgres=# drop table test;
DROP TABLE
postgres=# drop table test2;
DROP TABLE
postgres=# \c otus
You are now connected to database "otus" as user "postgres".
otus=# create table test (id int, fio text);
CREATE TABLE
otus=# create table test2 (id int, fio text);
CREATE TABLE
otus=# create subscription test_sub3 connection 'host=otus-vm port=5432 user=postgres password=postgres dbname=otus' publication test_pub with (copy_data = true);
ЗАМЕЧАНИЕ:  на сервере публикации создан слот репликации "test_sub3"
CREATE SUBSCRIPTION
otus=# create subscription test2_sub3 connection 'host=otus-vm-2 port=5432 user=postgres password=postgres dbname=otus' publication test2_pub with (copy_data = true);
ЗАМЕЧАНИЕ:  на сервере публикации создан слот репликации "test2_sub3"
CREATE SUBSCRIPTION
otus=# \dRs
             List of subscriptions
    Name    |  Owner   | Enabled | Publication 
------------+----------+---------+-------------
 test2_sub3 | postgres | t       | {test2_pub}
 test_sub3  | postgres | t       | {test_pub}
(2 rows)

```
