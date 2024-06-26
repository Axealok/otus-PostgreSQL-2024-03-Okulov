# Настройка autovacuum
## Создаем ВМ в Yandex Cloud
[Разворачиваем ВМ на Yandex.Cloud](https://github.com/Axealok/otus-PostgreSQL-2024-03-Okulov/blob/66dd3735de3e0a0595f515e4d4792bc610a41178/HW02-docker%20PG%20install/hw2.md)

Создаем базу данных
```
postgres=# CREATE DATABASE otus;
CREATE DATABASE
```
Запускаем pgbench
```
yc-user@otus-vm:~$ sudo -u postgres pgbench -i postgres
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.09 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 1.23 s (drop tables 0.00 s, create tables 0.02 s, client-side generate 0.89 s, vacuum 0.04 s, primary keys 0.29 s).

```
Запускаем pgbench -c8 -P 6 -T 60 -U postgres postgres
```
yc-user@otus-vm:~$ sudo -u postgres pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (15.6 (Ubuntu 15.6-1.pgdg20.04+1))
starting vacuum...end.
progress: 6.0 s, 436.2 tps, lat 18.223 ms stddev 15.099, 0 failed
progress: 12.0 s, 290.2 tps, lat 27.516 ms stddev 28.400, 0 failed
progress: 18.0 s, 320.0 tps, lat 24.992 ms stddev 21.308, 0 failed
progress: 24.0 s, 438.7 tps, lat 18.114 ms stddev 15.452, 0 failed
progress: 30.0 s, 396.2 tps, lat 20.271 ms stddev 15.213, 0 failed
progress: 36.0 s, 444.8 tps, lat 17.928 ms stddev 14.904, 0 failed
progress: 42.0 s, 280.8 tps, lat 28.484 ms stddev 41.966, 0 failed
progress: 48.0 s, 400.3 tps, lat 19.933 ms stddev 15.768, 0 failed
progress: 54.0 s, 478.3 tps, lat 16.703 ms stddev 14.658, 0 failed
progress: 60.0 s, 379.2 tps, lat 21.033 ms stddev 17.090, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 23196
number of failed transactions: 0 (0.000%)
latency average = 20.660 ms
latency stddev = 20.570 ms
initial connection time = 20.663 ms
tps = 386.557511 (without initial connection time)

```
Применяем указанные настройки
```
max_connections = 40
shared_buffers = 1GB
effective_cache_size = 3GB
maintenance_work_mem = 512MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 500
random_page_cost = 4
effective_io_concurrency = 2
work_mem = 65536kB
min_wal_size = 4GB
max_wal_size = 16GB
```
Запускаем тест заново
```
yc-user@otus-vm:~$ sudo pg_ctlcluster 15 main reload
yc-user@otus-vm:~$ sudo -u postgres pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (15.6 (Ubuntu 15.6-1.pgdg20.04+1))
starting vacuum...end.
progress: 6.0 s, 371.5 tps, lat 21.395 ms stddev 16.972, 0 failed
progress: 12.0 s, 384.5 tps, lat 20.754 ms stddev 25.649, 0 failed
progress: 18.0 s, 468.7 tps, lat 17.054 ms stddev 14.543, 0 failed
progress: 24.0 s, 524.8 tps, lat 15.208 ms stddev 12.055, 0 failed
progress: 30.0 s, 361.3 tps, lat 22.112 ms stddev 17.392, 0 failed
progress: 36.0 s, 491.3 tps, lat 16.262 ms stddev 13.144, 0 failed
progress: 42.0 s, 318.3 tps, lat 25.069 ms stddev 26.155, 0 failed
progress: 48.0 s, 469.2 tps, lat 17.035 ms stddev 14.346, 0 failed
progress: 54.0 s, 567.0 tps, lat 14.052 ms stddev 11.817, 0 failed
progress: 60.0 s, 436.5 tps, lat 18.316 ms stddev 14.518, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 26367
number of failed transactions: 0 (0.000%)
latency average = 18.170 ms
latency stddev = 16.932 ms
initial connection time = 21.190 ms
tps = 439.461529 (without initial connection time)
yc-user@otus-vm:~$ 

```
Мы увеличили доступные PG ресурсы ВМ - соответственно увеличилась производительность примерно на 30%

## Работа с  Autovacuum

Создаем и заполняем таблицу student случайными данными
```
c-user@otus-vm:~$ sudo -u postgres psql -p 5432
psql (15.6 (Ubuntu 15.6-1.pgdg20.04+1))
Type "help" for help.

postgres=# CREATE TABLE student(
postgres(# id serial,
postgres(# fio char(100)
postgres(# );
CREATE TABLE

postgres=# INSERT INTO student(fio) SELECT 'fio' FROM generate_series(1,1000000);
INSERT 0 1000000

postgres=# select count(*) from student;
  count  
---------
 1000000
(1 row)

postgres=# SELECT pg_size_pretty(pg_total_relation_size('student'));
 pg_size_pretty 
----------------
 135 MB
(1 row)


```
5 раз обновить все строчки и добавить к каждой строчке любой символ

```
update student set fio='fio1';
update student set fio='fio12';
update student set fio='fio123';
update student set fio='fio1234';
update student set fio='fio12345';

postgres-# trunc(100*n_dead_tup/(n_live_tup+1))::float AS "ratio%", last_autovacuum
postgres-# FROM pg_stat_user_tables WHERE relname = 'student';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
---------+------------+------------+--------+-------------------------------
 student |    1000000 |          0 |      0 | 2024-05-02 08:01:56.212517+00
(1 row)

update student set fio=concat('fio',1);
update student set fio=concat('fio',2);
update student set fio=concat('fio',3);
update student set fio=concat('fio',4);
update student set fio=concat('fio',5);

postgres=# SELECT pg_size_pretty(pg_total_relation_size('student'));
 pg_size_pretty 
----------------
 539 MB
(1 row)

postgres=# SELECT relname, n_live_tup, n_dead_tup,
trunc(100*n_dead_tup/(n_live_tup+1))::float AS "ratio%", last_autovacuum
FROM pg_stat_user_tables WHERE relname = 'student';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
---------+------------+------------+--------+-------------------------------
 student |    1000000 |     999830 |     99 | 2024-05-02 09:31:11.033465+00
(1 row)

```

Отключаем автовакуум для student

```
postgres=# alter table student set (autovacuum_enabled = off);
ALTER TABLE
postgres=# SELECT relname, n_live_tup, n_dead_tup,
trunc(100*n_dead_tup/(n_live_tup+1))::float AS "ratio%", last_autovacuum
FROM pg_stat_user_tables WHERE relname = 'student';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
---------+------------+------------+--------+-------------------------------
 student |    1000000 |          0 |      0 | 2024-05-02 09:31:59.558909+00
(1 row)

postgres=# 

```
10 раз обновить все строчки и добавить к каждой строчке любой символ

```
update student set fio=concat('fio',1);
---------------
update student set fio=concat('fio',10);

postgres=# SELECT pg_size_pretty(pg_total_relation_size('student'));
 pg_size_pretty 
----------------
 1482 MB
(1 row)

postgres=# SELECT relname, n_live_tup, n_dead_tup,
trunc(100*n_dead_tup/(n_live_tup+1))::float AS "ratio%", last_autovacuum
FROM pg_stat_user_tables WHERE relname = 'student';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
---------+------------+------------+--------+-------------------------------
 student |    1000000 |   10000000 |    999 | 2024-05-02 09:31:59.558909+00
(1 row)

```
При обновлении строки старая помечается на удаление и создается новая. Фактической очистки не происходит поэтому размер в 11 раз больше первоначального

```
postgres=# vacuum full;
VACUUM
postgres=# SELECT relname, n_live_tup, n_dead_tup,
trunc(100*n_dead_tup/(n_live_tup+1))::float AS "ratio%", last_autovacuum
FROM pg_stat_user_tables WHERE relname = 'student';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
---------+------------+------------+--------+-------------------------------
 student |    1000000 |   10000000 |    999 | 2024-05-02 09:31:59.558909+00
(1 row)

postgres=# SELECT pg_size_pretty(pg_total_relation_size('student'));
 pg_size_pretty 
----------------
 135 MB
(1 row)

postgres=# alter table student set (autovacuum_enabled = on);
ALTER TABLE

postgres=# SELECT relname, n_live_tup, n_dead_tup,
trunc(100*n_dead_tup/(n_live_tup+1))::float AS "ratio%", last_autovacuum
FROM pg_stat_user_tables WHERE relname = 'student';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
---------+------------+------------+--------+-------------------------------
 student |    1000000 |          0 |      0 | 2024-05-02 10:04:00.142134+00
(1 row)

```

# Задание со *

С выполнение задания возникли сложности
у меня ни в каком виде не захотел принимать оператор цикла FOR

```
FOR i IN 1..10
		LOOP
			update student set fio=concat('fio',i);
		END loop;

declare
begin
	FOR i IN 1..10
		LOOP
			update student set fio=concat('fio',i);
		END loop;
end;


```
Нашел в документации про анонимное выполнение кода
https://postgrespro.ru/docs/postgrespro/9.5/sql-do
В таком виде отработало
```
do $$declare i integer;
begin
	FOR i IN 1..10
		LOOP
			update student set fio=concat('fio',i);
		END loop;
end$$;
```
