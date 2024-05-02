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
