# PostgreSQL
***Проект***

***Тема: Создание и тестирование высоконагруженного отказоустойчивого кластера PostgreSQL на базе Patroni***

Schema:

![alt text](https://speedmedia.jfrog.com/08612fe1-9391-4cf3-ac1a-6dd49c36b276/https://media.jfrog.com/wp-content/uploads/2022/07/18231453/High_Available_PostgreSQL_Cluster_using_Patroni_and_HAProxy-1.jpg/w_768)

Architecture:
OS: Ubuntu 22.04

Postgres version: 15

Machine: node1                   IP: 10.128.0.23                 Role: Postgresql, Patroni

Machine: node2                   IP: 10.128.0.13                 Role: Postgresql, Patroni

Machine: node3                   IP: 10.128.0.35                 Role: Postgresql, Patroni

Machine: etcdnode              IP: 10.128.0.21           Role: etcd

Machine: haproxynode       IP: 10.128.0.12     Role: HA Proxy



***Инструкция по шагам***
 
Шаг 1 –  Первоначальная настройка node1, node2, node3:

```
sudo apt update

sudo hostnamectl set-hostname nodeN

sudo apt install net-tools

sudo apt install postgresql-15

sudo systemctl stop postgresql

sudo ln -s /usr/lib/postgresql/15/bin/* /usr/sbin/

//sudo apt install python-is-python3 -- опционально, если следующий шаг выдает ошибку

sudo apt -y install python3 python3-pip 

sudo apt install python3-testresources   

sudo pip3 install --upgrade setuptools 

//sudo apt-get install --reinstall libpq-dev -- опционально, если следующий шаг выдает ошибку
 
sudo pip3 install psycopg2 

sudo pip3 install patroni

sudo pip3 install python-etcd
```
 

Шаг 2 –  Первоначальная настройка etcdnode:

```
sudo apt update

sudo hostnamectl set-hostname etcdnode

sudo apt install net-tools

sudo apt -y install etcd 
```
 

Шаг 3 – Первоначальная настройка haproxynode:

```
sudo apt update

sudo hostnamectl set-hostname haproxy

sudo apt install net-tools

sudo apt -y install haproxy
```
 

Шаг 4 – Настраиваем etcd на etcdnode: 

```
sudo vi /etc/default/etcd   

ETCD_LISTEN_PEER_URLS="http://10.128.0.21:2380" 
ETCD_LISTEN_CLIENT_URLS="http://10.128.0.21:2379, http://localhost:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.128.0.21:2380"
ETCD_INITIAL_CLUSTER="default=http://10.128.0.21:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://10.128.0.21:2379"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"

sudo systemctl restart etcd 

sudo systemctl status etcd

curl http://10.128.0.29:2380/members -- проверка
```

 

Шаг 5 – Настраиваем Patroni на всех нодах:

```
sudo vi /etc/patroni.yml

scope: postgres
namespace: /db/
name: <NodeN>

restapi:
    listen: <NODEIP>:8008
    connect_address: <NODEIP>:8008

etcd:
    host: <ETCDIP>:2379

bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
      use_slots: true
      parameters:

  initdb:
  - encoding: UTF8
  - data-checksums

  pg_hba:
  - host replication replicator 127.0.0.1/32 md5
  - host replication replicator <NODE1IP>/0 md5
  - host replication replicator <NODE2IP>/0 md5
  - host replication replicator <NODE3IP>/0 md5
  - host all all 0.0.0.0/0 md5

  users:
    admin:
      password: admin
      options:
        - createrole
        - createdb

postgresql:
  listen: <NODEIP>:5432
  connect_address: <NODEIP>:5432
  data_dir: /data/patroni
  pgpass: /tmp/pgpass
  authentication:
    replication:
      username: replicator
      password: *
    superuser:
      username: postgres
      password: *
  parameters:
      unix_socket_directories: '/var/run/postgresql'

tags:
    nofailover: false
    noloadbalance: false
    clonefrom: false
    nosync: false


sudo mkdir -p /data/patroni

sudo chown postgres:postgres /data/patroni

sudo chmod 700 /data/patroni 

sudo vi /etc/systemd/system/patroni.service

[Unit]
Description=High availability PostgreSQL Cluster
After=syslog.target network.target

[Service]
Type=simple
User=postgres
Group=postgres
ExecStart=/usr/local/bin/patroni /etc/patroni.yml
KillMode=process
TimeoutSec=30
Restart=no

[Install]
WantedBy=multi-user.target
```
 

Шаг 6 – Запускаем Patroni на всех нодах:

```
sudo systemctl start patroni

sudo systemctl status patroni

dmi@node1:~$ sudo systemctl status patroni

●  patroni.service - High availability PostgreSQL Cluster

     Loaded: loaded (/etc/systemd/system/patroni.service; disabled; vendor preset: enabled)

     Active: active (running) since Sun 2024-02-18 17:40:08 EDT; 3min ago

   Main PID: 3430 (patroni)

      Tasks: 13 (limit: 2319)

     Memory: 95.9M

     CGroup: /system.slice/patroni.service

             ├─3430 /usr/bin/python3 /usr/local/bin/patroni /etc/patroni.yml

             ├─4189 postgres -D /data/patroni --config-file=/data/patroni/postgresql.conf --listen_addresses=10.128.0.21 --port=5432 --cluster_name=postgres --wal_level=replica --h>

             ├─4197 postgres: postgres: checkpointer

             ├─4198 postgres: postgres: background writer

             ├─4199 postgres: postgres: walwriter

             ├─4200 postgres: postgres: autovacuum launcher

             ├─4201 postgres: postgres: stats collector

             ├─4202 postgres: postgres: logical replication launcher

             └─4204 postgres: postgres: postgres postgres 10.128.0.21(50256) idle

Feb 18 17:43:46 node1 patroni[3430]: 2022-06-28 06:43:46,405 INFO: Lock owner: node1; I am node1
Feb 18 17:43:46 node1 patroni[3430]: 2022-06-28 06:43:46,410 INFO: no action. i am the leader with the lock
Feb 18 17:43:56 node1 patroni[3430]: 2022-06-28 06:43:56,450 INFO: Lock owner: node1; I am node1
Feb 18 17:43:56 node1 patroni[3430]: 2022-06-28 06:43:56,455 INFO: no action. i am the leader with the lock
Feb 18 17:44:06 node1 patroni[3430]: 2022-06-28 06:44:06,409 INFO: Lock owner: node1; I am node1
Feb 18 17:44:06 node1 patroni[3430]: 2022-06-28 06:44:06,414 INFO: no action. i am the leader with the lock
Feb 18 17:44:16 node1 patroni[3430]: 2022-06-28 06:44:16,404 INFO: Lock owner: node1; I am node1
Feb 18 17:44:16 node1 patroni[3430]: 2022-06-28 06:44:16,407 INFO: no action. i am the leader with the lock
Feb 18 17:44:26 node1 patroni[3430]: 2022-06-28 06:44:26,404 INFO: Lock owner: node1; I am node1
Feb 18 17:44:26 node1 patroni[3430]: 2022-06-28 06:44:26,408 INFO: no action. i am the leader with the lock
```

Шаг 7 – Настройка HAProxy: 

```
sudo vi /etc/haproxy/haproxy.cfg

Replace its context with this:

global

        maxconn 100
        log     127.0.0.1 local2

defaults
        log global
        mode tcp
        retries 2
        timeout client 30m
        timeout connect 4s
        timeout server 30m
        timeout check 5s

listen stats
    mode http
    bind *:7000
    stats enable
    stats uri /

listen postgres
    bind *:5000
    option httpchk
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server node1 <node1_ip>:5432 maxconn 100 check port 8008
    server node2 <node2_ip>:5432 maxconn 100 check port 8008
    server node3 <node3_ip>:5432 maxconn 100 check port 8008

sudo systemctl restart haproxy

sudo systemctl status haproxy

● haproxy.service - HAProxy Load Balancer 
      Loaded: loaded (/lib/systemd/system/haproxy.service; enabled; vendor preset: enabled) 
      Active: active (running) since Sun 2024-02-18 17:54:12 EDT; 10s ago 
        Docs: man:haproxy(1) 
              file:/usr/share/doc/haproxy/configuration.txt.gz 
     Process: 1736 ExecStartPre=/usr/sbin/haproxy -f $CONFIG -c -q $EXTRAOPTS (code=exited,status=0/SUCCESS)   Main PID: 1751 (haproxy) 
     Tasks: 3 (limit: 2319) 
   Memory: 2.1M 
   CGroup: /system.slice/haproxy.service
            ├─1751 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock             
            └─1753 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock 
Feb 18 17:54:22 haproxynode systemd[1]: Starting HAProxy Load Balancer... 
Feb 18 17:54:22 haproxynode haproxy[1751]: [NOTICE] 254/065422 (1751) : New worker #1 (1753) forked 
Feb 18 17:54:22 haproxynode systemd[1]: Started HAProxy Load Balancer. 
Feb 18 17:54:23 haproxynode haproxy[1753]: [WARNING] 254/065423 (1753) : Server postgres/node2 is DOWN, reason: Layer7 wrong status, code: 503, info: 
"HTTP status check returned code <3C>503<3E>">
 Jun 28 06:54:24 haproxynode haproxy[1753]: [WARNING] 254/065424 (1753) : Server postgres/node3 is DOWN, reason: Layer7 wrong status, code: 503, info: 
"HTTP status check returned code <3C>503<3E>">
 ```

Шаг 8 – Тестируем кластер:


Симулируем отказ одной ноды:

![alt text](https://speedmedia.jfrog.com/08612fe1-9391-4cf3-ac1a-6dd49c36b276/https://media.jfrog.com/wp-content/uploads/2022/07/25041531/Step8.png/w_1024)
```
sudo systemctl stop patroni
```
Мастером становится другая нода:

![alt text](https://speedmedia.jfrog.com/08612fe1-9391-4cf3-ac1a-6dd49c36b276/https://media.jfrog.com/wp-content/uploads/2022/07/09193620/Screen-Shot-2022-08-09-at-20.35.51.png/w_1024)
 

Шаг 9 – Заходим на ноду через Haproxy:

```
psql -h <haproxynode_ip> -p 5000 -U postgres

psql -h 10.128.0.12 -p 5000 -U postgres

fkdark@node1:~$ patronictl -c /etc/patroni.yml list
+ Cluster: postgres (6871178537652191317) ---+----+-----------+
| Member | Host          | Role    | State   | TL | Lag in MB |
+--------+---------------+---------+---------+----+-----------+
| node1  | 10.128.0.23 | Replica | running |  2 |         0 |
| node2  | 10.128.0.13 | Leader  | running |  2 |           |
| node3  | 10.128.0.35 | Replica | running |  2 |         0 |
+--------+---------------+---------+---------+----+-----------+
```


