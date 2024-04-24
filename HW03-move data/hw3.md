# Перенос содержимого базы данных PostgreSQL
## Установка тестового полигона

```
C:\Users\okulovan>ssh -i C:/Users/okulovan/OkulovAN okulovan@176.109.103.34
$ sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt$(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15
$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
$ sudo -u postgres psql -p 5432
postgres=# ALTER USER postgres PASSWORD 'postgres';
ALTER ROLE
postgres=# create table test(c1 text);
CREATE TABLE
postgres=# insert into test values('TestValue');
INSERT 0 1
postgres=# select * from test
postgres-# ;
    c1
-----------
 TestValue
(1 row)

$ sudo -u postgres pg_ctlcluster 15 main stop
Warning: stopping the cluster using pg_ctlcluster will mark the systemd unit as failed. Consider using systemctl:
  sudo systemctl stop postgresql@15-main
$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
15  main    5432 down   postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
```

## Подключение дополнительного диска

![Create disk](https://github.com/Axealok/otus-PostgreSQL-2024-03-Okulov/blob/cbfc7fee492c82c03d7735929979f385b8951992/HW03-move%20data/hw3_add_disk.PNG)
![Add disk to VM](https://github.com/Axealok/otus-PostgreSQL-2024-03-Okulov/blob/cbfc7fee492c82c03d7735929979f385b8951992/HW03-move%20data/hw3_add_disk2.PNG)


```
$ sudo fdisk -l
------------
Disk /dev/vdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
------------
$ sudo cfdisk /dev/vdb
```
![Create partition](https://github.com/Axealok/otus-PostgreSQL-2024-03-Okulov/blob/8b172e721be7f22b103cb8639d59caa90b152397/HW03-move%20data/hw3_add_disk3.PNG)
```
$ sudo mkfs.ext4 /dev/vdb
mke2fs 1.46.5 (30-Dec-2021)
Discarding device blocks: done
Creating filesystem with 2621440 4k blocks and 655360 inodes
Filesystem UUID: 3921c77d-ba5d-4815-bc72-639d38dc6c0e
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

$ sudo mount /dev/vdb /mnt/data
mount: /mnt/data: mount point does not exist.
$ sudo mkdir /mnt/data
$ sudo mount /dev/vdb /mnt/data
$ sudo chmod 0777 /mnt/data
$ sudo nano /etc/fstab
```
![Automount](https://github.com/Axealok/otus-PostgreSQL-2024-03-Okulov/blob/e1ab3a14f1be7093be390424b45e071d3b2f02b0/HW03-move%20data/hw3_add_disk4.PNG)

```
$ mv /var/lib/postgresql/15 /mnt/data
$ sudo -u postgres pg_ctlcluster 15 main start
Error: /var/lib/postgresql/15/main is not accessible or does not exist
```
Исправляем путь:

![Automount](https://github.com/Axealok/otus-PostgreSQL-2024-03-Okulov/blob/2e0c0893353bfce91b74dad91a9aaba675f1b389/HW03-move%20data/hw3_pgconf.PNG)
![Automount](https://github.com/Axealok/otus-PostgreSQL-2024-03-Okulov/blob/2e0c0893353bfce91b74dad91a9aaba675f1b389/HW03-move%20data/hw3_pgconf2.PNG)

```
$ sudo -u postgres pg_ctlcluster 15 main start
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@15-main
$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory    Log file
15  main    5432 online postgres /mnt/data/15/main /var/log/postgresql/postgresql-15-main.log
$ sudo -u postgres psql -p 5432
could not change directory to "/home/okulovan": Permission denied
psql (15.6 (Ubuntu 15.6-1.pgdg22.04+1))
Type "help" for help.

postgres=# select * from test;
    c1
-----------
 TestValue
(1 row)

postgres=#
```

## Перенос данных на другую ВМ

На первой ВМ отмонтируем диск

```
$ sudo -u postgres pg_ctlcluster 15 main stop
$ sudo umount /dev/vdb
```

Создаем вторую ВМ

