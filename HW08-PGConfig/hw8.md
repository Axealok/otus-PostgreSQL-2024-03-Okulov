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


