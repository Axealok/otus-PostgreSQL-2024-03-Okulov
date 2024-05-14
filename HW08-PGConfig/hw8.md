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

