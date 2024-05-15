# Нагрузочное тестирование и тюнинг PostgreSQL

Развернул ВМ
2CPU/4Gb/HDD/Ubuntu

## Устанавливаем PG 15

```
sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15
```

## Нагрузочный тест установок по умолчанию

```
okulovan@otus-vm:~$ sudo -u postgres pgbench -c50 -P 10 -T 60 -U postgres postgres
pgbench (15.7 (Ubuntu 15.7-1.pgdg22.04+1))
starting vacuum...end.
progress: 10.0 s, 605.6 tps, lat 78.892 ms stddev 73.990, 0 failed
progress: 20.0 s, 630.6 tps, lat 79.201 ms stddev 75.959, 0 failed
progress: 30.0 s, 637.9 tps, lat 78.229 ms stddev 76.949, 0 failed
progress: 40.0 s, 622.6 tps, lat 79.989 ms stddev 86.054, 0 failed
progress: 50.0 s, 643.9 tps, lat 77.670 ms stddev 80.386, 0 failed
progress: 60.0 s, 657.4 tps, lat 75.486 ms stddev 74.604, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 38028
number of failed transactions: 0 (0.000%)
latency average = 78.440 ms
latency stddev = 78.617 ms
initial connection time = 359.765 ms
tps = 634.812674 (without initial connection time)

```

## Рекомендованные настройки PGTune

```
# DB Version: 15
# OS Type: linux
# DB Type: mixed
# Total Memory (RAM): 4 GB
# CPUs num: 2
# Connections num: 100
# Data Storage: hdd

max_connections = 100
shared_buffers = 1GB
effective_cache_size = 3GB
maintenance_work_mem = 256MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 4
effective_io_concurrency = 2
work_mem = 2621kB
huge_pages = off
min_wal_size = 1GB
max_wal_size = 4GB

```

## Применяем рекомендуемые настройки и тестируем снова

```
okulovan@otus-vm:~$ sudo -u postgres pgbench -c50 -P 10 -T 60 -U postgres postgres
pgbench (15.7 (Ubuntu 15.7-1.pgdg22.04+1))
starting vacuum...end.
progress: 10.0 s, 666.5 tps, lat 71.915 ms stddev 76.148, 0 failed
progress: 20.0 s, 662.5 tps, lat 75.058 ms stddev 77.281, 0 failed
progress: 30.0 s, 578.1 tps, lat 86.366 ms stddev 81.466, 0 failed
progress: 40.0 s, 614.2 tps, lat 81.397 ms stddev 78.492, 0 failed
progress: 50.0 s, 526.8 tps, lat 94.657 ms stddev 97.545, 0 failed
progress: 60.0 s, 511.7 tps, lat 97.470 ms stddev 94.224, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 35648
number of failed transactions: 0 (0.000%)
latency average = 83.728 ms
latency stddev = 84.553 ms
initial connection time = 345.853 ms
tps = 594.676074 (without initial connection time)

```
Рекомендованные настройко оказались не оптимальными (Показатели нагрузочного теста хуже чем с настройками по умолчанию)

## Пробуем согласно документации настроить

```
 /etc/postgresql/15/main/conf.d/recomended.conf   |          1 |    25 | max_connections              | 100                                    | t       | 
 /etc/postgresql/15/main/conf.d/recomended.conf   |          2 |    26 | shared_buffers               | 1GB                                    | f       | 
 /etc/postgresql/15/main/conf.d/recomended.conf   |          3 |    27 | temp_buffers                 | 32MB                                   | t       | 
 /etc/postgresql/15/main/conf.d/recomended.conf   |          4 |    28 | effective_cache_size         | 3GB                                    | t       | 
 /etc/postgresql/15/main/conf.d/recomended.conf   |          5 |    29 | maintenance_work_mem         | 512MB                                  | t       | 
 /etc/postgresql/15/main/conf.d/recomended.conf   |          6 |    30 | checkpoint_completion_target | 0.9                                    | t       | 
 /etc/postgresql/15/main/conf.d/recomended.conf   |          7 |    31 | synchronous_commit           | off                                    | t       | 
 /etc/postgresql/15/main/conf.d/recomended.conf   |          8 |    32 | default_statistics_target    | 500                                    | t       | 
 /etc/postgresql/15/main/conf.d/recomended.conf   |          9 |    33 | random_page_cost             | 4                                      | t       | 
 /etc/postgresql/15/main/conf.d/recomended.conf   |         10 |    34 | effective_io_concurrency     | 2                                      | t       | 
 /etc/postgresql/15/main/conf.d/recomended.conf   |         11 |    35 | work_mem                     | 128MB                                  | t       | 
 /etc/postgresql/15/main/conf.d/recomended.conf   |         12 |    36 | min_wal_size                 | 4GB                                    | t       | 
 /etc/postgresql/15/main/conf.d/recomended.conf   |         13 |    37 | max_wal_size                 | 16GB                                   | t       | 
 /var/lib/postgresql/15/main/postgresql.auto.conf |          3 |    38 | shared_buffers               | 128MB                                  | t       | 

```

```
okulovan@otus-vm:~$ sudo -u postgres pgbench -c50 -P 10 -T 60 -U postgres postgres
[sudo] пароль для okulovan: 
pgbench (15.7 (Ubuntu 15.7-1.pgdg22.04+1))
starting vacuum...end.
progress: 10.0 s, 858.4 tps, lat 55.780 ms stddev 56.957, 0 failed
progress: 20.0 s, 885.1 tps, lat 56.326 ms stddev 59.339, 0 failed
progress: 30.0 s, 845.5 tps, lat 58.787 ms stddev 56.996, 0 failed
progress: 40.0 s, 853.1 tps, lat 58.381 ms stddev 59.463, 0 failed
progress: 50.0 s, 874.4 tps, lat 56.809 ms stddev 58.230, 0 failed
progress: 60.0 s, 855.2 tps, lat 58.220 ms stddev 61.199, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 51767
number of failed transactions: 0 (0.000%)
latency average = 57.491 ms
latency stddev = 58.901 ms
initial connection time = 320.212 ms
tps = 864.077967 (without initial connection time)

```

Видим что производительность выросла

## Тестируем через утилиту sysbench

Выданный на вэбинаре мануал не подходит:

- отсутствуют тесты указанные мануале
- не хватает прав пользователю для создания таблиц sbtest вернее на схему public

С sysbench разобрался самостоятельно

sysbench --help

### Подготовка
```

okulovan@otus-vm:~$ sysbench --db-driver=pgsql --report-interval=2 --table-size=100000 --tables=24 --threads=64 --time=60 --pgsql-host=localhost --pgsql-port=5432 --pgsql-user=sbtest --pgsql-password=password --pgsql-db=sbtest /usr/share/sysbench/oltp_read_write.lua prepare

```
### Тестирование
```
okulovan@otus-vm:~$ sysbench --db-driver=pgsql --report-interval=2 --table-size=100000 --tables=24 --threads=64 --time=60 --pgsql-host=localhost --pgsql-port=5432 --pgsql-user=sbtest --pgsql-password=password --pgsql-db=sbtest /usr/share/sysbench/oltp_read_write.lua run

okulovan@otus-vm:~$ sysbench --db-driver=pgsql --report-interval=2 --table-size=100000 --tables=24 --threads=64 --time=60 --pgsql-host=localhost --pgsql-port=5432 --pgsql-user=sbtest --pgsql-password=password --pgsql-db=sbtest /usr/share/sysbench/oltp_read_write.lua run
sysbench 1.0.20 (using system LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 64
Report intermediate results every 2 second(s)
Initializing random number generator from current time


Initializing worker threads...

Threads started!

[ 2s ] thds: 64 tps: 234.05 qps: 5064.76 (r/w/o: 3614.05/950.17/500.54) lat (ms,95%): 590.56 err/s: 0.00 reconn/s: 0.00
[ 4s ] thds: 64 tps: 200.11 qps: 3956.73 (r/w/o: 2756.55/800.45/399.72) lat (ms,95%): 411.96 err/s: 0.00 reconn/s: 0.00
[ 6s ] thds: 64 tps: 218.49 qps: 4377.35 (r/w/o: 3060.89/878.97/437.48) lat (ms,95%): 369.77 err/s: 0.00 reconn/s: 0.00
[ 8s ] thds: 64 tps: 240.50 qps: 4794.01 (r/w/o: 3355.51/958.00/480.50) lat (ms,95%): 356.70 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 64 tps: 247.02 qps: 4923.32 (r/w/o: 3445.23/985.06/493.03) lat (ms,95%): 363.18 err/s: 0.00 reconn/s: 0.00
[ 12s ] thds: 64 tps: 259.50 qps: 5185.49 (r/w/o: 3627.49/1039.50/518.50) lat (ms,95%): 320.17 err/s: 0.00 reconn/s: 0.00
[ 14s ] thds: 64 tps: 245.50 qps: 4928.00 (r/w/o: 3461.50/974.50/492.00) lat (ms,95%): 325.98 err/s: 0.00 reconn/s: 0.00
[ 16s ] thds: 64 tps: 238.43 qps: 4781.58 (r/w/o: 3346.51/956.22/478.86) lat (ms,95%): 337.94 err/s: 0.00 reconn/s: 0.00
[ 18s ] thds: 64 tps: 255.53 qps: 5079.16 (r/w/o: 3540.96/1029.13/509.07) lat (ms,95%): 331.91 err/s: 0.00 reconn/s: 0.00
[ 20s ] thds: 64 tps: 256.53 qps: 5144.56 (r/w/o: 3608.90/1020.61/515.06) lat (ms,95%): 325.98 err/s: 0.00 reconn/s: 0.00
[ 22s ] thds: 64 tps: 255.01 qps: 5137.20 (r/w/o: 3601.14/1027.04/509.02) lat (ms,95%): 369.77 err/s: 0.00 reconn/s: 0.00
[ 24s ] thds: 64 tps: 245.47 qps: 4888.99 (r/w/o: 3418.64/977.40/492.95) lat (ms,95%): 344.08 err/s: 0.50 reconn/s: 0.00
[ 26s ] thds: 64 tps: 248.96 qps: 4971.17 (r/w/o: 3485.42/987.83/497.92) lat (ms,95%): 308.84 err/s: 0.00 reconn/s: 0.00
[ 28s ] thds: 64 tps: 264.56 qps: 5285.18 (r/w/o: 3690.82/1065.74/528.62) lat (ms,95%): 303.33 err/s: 0.00 reconn/s: 0.00
[ 30s ] thds: 64 tps: 267.50 qps: 5324.54 (r/w/o: 3729.03/1059.01/536.50) lat (ms,95%): 369.77 err/s: 0.00 reconn/s: 0.00
[ 32s ] thds: 64 tps: 257.52 qps: 5196.30 (r/w/o: 3635.21/1046.56/514.53) lat (ms,95%): 337.94 err/s: 0.50 reconn/s: 0.00
[ 34s ] thds: 64 tps: 252.50 qps: 5046.98 (r/w/o: 3534.49/1007.50/505.00) lat (ms,95%): 325.98 err/s: 0.00 reconn/s: 0.00
[ 36s ] thds: 64 tps: 253.50 qps: 5061.45 (r/w/o: 3540.97/1011.99/508.50) lat (ms,95%): 325.98 err/s: 0.00 reconn/s: 0.00
[ 38s ] thds: 64 tps: 265.97 qps: 5348.92 (r/w/o: 3754.10/1063.89/530.94) lat (ms,95%): 314.45 err/s: 0.00 reconn/s: 0.00
[ 40s ] thds: 64 tps: 264.53 qps: 5281.04 (r/w/o: 3691.88/1059.11/530.05) lat (ms,95%): 314.45 err/s: 0.50 reconn/s: 0.00
[ 42s ] thds: 64 tps: 266.50 qps: 5281.48 (r/w/o: 3689.98/1057.49/534.00) lat (ms,95%): 297.92 err/s: 0.00 reconn/s: 0.00
[ 44s ] thds: 64 tps: 261.45 qps: 5276.91 (r/w/o: 3695.74/1060.78/520.39) lat (ms,95%): 297.92 err/s: 0.00 reconn/s: 0.00
[ 46s ] thds: 64 tps: 280.02 qps: 5622.94 (r/w/o: 3949.31/1110.09/563.54) lat (ms,95%): 297.92 err/s: 0.50 reconn/s: 0.00
[ 48s ] thds: 64 tps: 278.04 qps: 5552.23 (r/w/o: 3876.01/1119.15/557.07) lat (ms,95%): 297.92 err/s: 0.00 reconn/s: 0.00
[ 50s ] thds: 64 tps: 278.98 qps: 5568.19 (r/w/o: 3900.28/1111.94/555.97) lat (ms,95%): 292.60 err/s: 0.00 reconn/s: 0.00
[ 52s ] thds: 64 tps: 263.02 qps: 5279.82 (r/w/o: 3694.72/1057.06/528.03) lat (ms,95%): 287.38 err/s: 0.00 reconn/s: 0.00
[ 54s ] thds: 64 tps: 274.49 qps: 5454.78 (r/w/o: 3814.85/1092.96/546.98) lat (ms,95%): 308.84 err/s: 0.00 reconn/s: 0.00
[ 56s ] thds: 64 tps: 282.51 qps: 5679.69 (r/w/o: 3979.63/1135.04/565.02) lat (ms,95%): 337.94 err/s: 0.00 reconn/s: 0.00
[ 58s ] thds: 64 tps: 285.99 qps: 5691.78 (r/w/o: 3980.35/1139.96/571.48) lat (ms,95%): 297.92 err/s: 0.00 reconn/s: 0.00
[ 60s ] thds: 64 tps: 273.96 qps: 5461.66 (r/w/o: 3819.41/1093.33/548.92) lat (ms,95%): 287.38 err/s: 0.50 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            217028
        write:                           61991
        other:                           31011
        total:                           310030
    transactions:                        15497  (255.02 per sec.)
    queries:                             310030 (5101.97 per sec.)
    ignored errors:                      5      (0.08 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          60.7639s
    total number of events:              15497

Latency (ms):
         min:                                    4.73
         avg:                                  248.15
         max:                                 2733.52
         95th percentile:                      344.08
         sum:                              3845654.68

Threads fairness:
    events (avg/stddev):           242.1406/12.30
    execution time (avg/stddev):   60.0884/0.33

```
