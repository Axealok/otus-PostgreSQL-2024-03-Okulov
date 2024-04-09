# Разворачиваем ВМ на Yandex.Cloud
```
kulovan@b1-t-oan:~$ yc init
Welcome! This command will take you through the configuration process.
Please go to https://oauth.yandex.ru/authorize?response_type=token&client_id=1a6990aa636648e9b2ef855fa7bec2fb in order to obtain OAuth token.

Please enter OAuth token: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
You have one cloud available: 'cloud-axealok' (id = b1gmigikar5k3ead626j). It is going to be used by default.
Please choose folder to use:
 [1] default (id = b1gj7o4kqguint1p6sn8)
 [2] Create a new folder
Please enter your numeric choice: 1
Your current folder has been set to 'default' (id = b1gj7o4kqguint1p6sn8).
Do you want to configure a default Compute zone? [Y/n] y
Which zone do you want to use as a profile default?
 [1] ru-central1-a
 [2] ru-central1-b
 [3] ru-central1-c
 [4] ru-central1-d
 [5] Don't set default zone
Please enter your numeric choice: 1
Your profile default Compute zone has been set to 'ru-central1-a'.
okulovan@b1-t-oan:~$ yc version
Yandex Cloud CLI 0.123.0 linux/amd64
okulovan@b1-t-oan:~$ yc compute zone list
+---------------+--------+
|      ID       | STATUS |
+---------------+--------+
| ru-central1-a | UP     |
| ru-central1-b | UP     |
| ru-central1-c | UP     |
| ru-central1-d | UP     |
+---------------+--------+

okulovan@b1-t-oan:~$ yc config set compute-default-zone ru-central1-a && yc config get compute-default-zone
ru-central1-a
okulovan@b1-t-oan:~$ yc compute disk-type list
+---------------------------+--------------------------------+
|            ID             |          DESCRIPTION           |
+---------------------------+--------------------------------+
| network-hdd               | Network storage with HDD       |
|                           | backend                        |
| network-ssd               | Network storage with SSD       |
|                           | backend                        |
| network-ssd-io-m3         | Fast network storage with      |
|                           | three replicas                 |
| network-ssd-nonreplicated | Non-replicated network storage |
|                           | with SSD backend               |
+---------------------------+--------------------------------+

okulovan@b1-t-oan:~$ yc vpc network create \
    --name otus-net \
    --description "otus-net" \
> exit
ERROR: positional argument can not be used with flag '--name'
okulovan@b1-t-oan:~$ yc vpc network list
+----------------------+---------+
|          ID          |  NAME   |
+----------------------+---------+
| enpbqfc86e42vr4n994n | default |
+----------------------+---------+

okulovan@b1-t-oan:~$ yc vpc network create \
>     --name otus-net \
>     --description "otus-net" \
> exit
ERROR: positional argument can not be used with flag '--name'
okulovan@b1-t-oan:~$ yc vpc network create \
    --name otus-net \
    --description "otus-net" \
> 
id: enppaiq6g7vhdua55ou2
folder_id: b1gj7o4kqguint1p6sn8
created_at: "2024-04-09T09:26:20Z"
name: otus-net
description: otus-net
default_security_group_id: enped60e3ovetib04auj

okulovan@b1-t-oan:~$ yc vpc network list
+----------------------+----------+
|          ID          |   NAME   |
+----------------------+----------+
| enpbqfc86e42vr4n994n | default  |
| enppaiq6g7vhdua55ou2 | otus-net |
+----------------------+----------+

okulovan@b1-t-oan:~$ yc vpc subnet create \
    --name otus-subnet \
    --range 192.168.0.0/24 \
    --network-name otus-net \
    --description "otus-subnet" \
> 
id: e9bl8qd5j3pira2q0tds
folder_id: b1gj7o4kqguint1p6sn8
created_at: "2024-04-09T09:26:56Z"
name: otus-subnet
description: otus-subnet
network_id: enppaiq6g7vhdua55ou2
zone_id: ru-central1-a
v4_cidr_blocks:
  - 192.168.0.0/24

okulovan@b1-t-oan:~$ yc vpc subnet list
+----------------------+-----------------------+----------------------+----------------+---------------+------------------+
|          ID          |         NAME          |      NETWORK ID      | ROUTE TABLE ID |     ZONE      |      RANGE       |
+----------------------+-----------------------+----------------------+----------------+---------------+------------------+
| b0cihm8a1fi3kgjlodlt | default-ru-central1-c | enpbqfc86e42vr4n994n |                | ru-central1-c | [10.130.0.0/24]  |
| e2ldn06atnosk0228e4i | default-ru-central1-b | enpbqfc86e42vr4n994n |                | ru-central1-b | [10.129.0.0/24]  |
| e9bflub7rhqgjju5jdmr | default-ru-central1-a | enpbqfc86e42vr4n994n |                | ru-central1-a | [10.128.0.0/24]  |
| e9bl8qd5j3pira2q0tds | otus-subnet           | enppaiq6g7vhdua55ou2 |                | ru-central1-a | [192.168.0.0/24] |
+----------------------+-----------------------+----------------------+----------------+---------------+------------------+

okulovan@b1-t-oan:~$ ssh-keygen -t rsa -b 2048
Generating public/private rsa key pair.
Enter file in which to save the key (/home/okulovan/.ssh/id_rsa): yc_key
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in yc_key
Your public key has been saved in yc_key.pub
The key fingerprint is:
SHA256:ij3+nuyqiuy6pxqA2GVTfhXdAe7SzdIPhgG3+220yos okulovan@b1-t-oan
The key's randomart image is:
+---[RSA 2048]----+
|      .   +ooo.. |
|     o   . +...  |
|    + . .   +    |
|o. o . .   o B   |
|+ .     S . * * .|
|.    o .   . + =.|
|.   . +       ..+|
|.o.  . o .  o .. |
|X*....o=*  E +.  |
+----[SHA256]-----+
okulovan@b1-t-oan:~$ ssh-add ~/.ssh/yc_key
/home/okulovan/.ssh/yc_key: No such file or directory
okulovan@b1-t-oan:~$ ssh-add /.ssh/yc_key
/.ssh/yc_key: No such file or directory
okulovan@b1-t-oan:~$ ssh-add /home/okulovan/.ssh/yc_key
/home/okulovan/.ssh/yc_key: No such file or directory
okulovan@b1-t-oan:~$ cd .ssh
bash: cd: .ssh: Нет такого файла или каталога
okulovan@b1-t-oan:~$ ls -a
 .               .bash_logout   .config    snap                        yandex-cloud   Видео       Изображения    'Рабочий стол'
 ..              .bashrc        .local     .sudo_as_admin_successful   yc_key         Документы   Музыка          Шаблоны
 .bash_history   .cache         .profile   .wget-hsts                  yc_key.pub     Загрузки    Общедоступные
okulovan@b1-t-oan:~$ ssh-add /home/okulovan/yc_key
Identity added: /home/okulovan/yc_key (okulovan@b1-t-oan)
okulovan@b1-t-oan:~$ ssh-add ~/yc_key
Identity added: /home/okulovan/yc_key (okulovan@b1-t-oan)

```

