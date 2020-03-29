# Version

|Kubernetes|MariaDB|Maxscale|
|:--------:|:-----:|:------:|
|1.16.2    |10.4.8 |2.4.2   |

# デプロイ

    kubectl apply -f mariadb.yaml

    kubectl get pod

    NAME        READY   STATUS    RESTARTS   AGE
    mariadb-0   1/1     Running   0          6m22s
    mariadb-1   1/1     Running   0          4m24s
    mariadb-2   1/1     Running   0          2m16s

    kubectl apply -f maxscale.yaml

    kubectl exec -it maxscale-0  --container=maxscale -- maxctrl list servers
    ┌─────────┬─────────────────────────────────────────────────┬──────┬─────────────┬─────────────────┬──────────┐
    │ Server  │ Address                                         │ Port │ Connections │ State           │ GTID     │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────┼──────────┤
    │ server1 │ mariadb-0.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Master, Running │ 0-3000-0 │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────┼──────────┤
    │ server2 │ mariadb-1.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Slave, Running  │ 0-3000-0 │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────┼──────────┤
    │ server3 │ mariadb-2.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Slave, Running  │ 0-3000-0 │
    └─────────┴─────────────────────────────────────────────────┴──────┴─────────────┴─────────────────┴──────────┘

# 動作確認

    kubectl run -it --rm --image=mariadb:10.4 --restart=Never mariadb-client -- mysql -h maxscale-svc -u maxuser -pmaxpwd  -P 4006
    If you don't see a command prompt, try pressing enter.
    MariaDB [(none)]> create database sample;
    Query OK, 1 row affected (0.002 sec)

    MariaDB [(none)]> use sample;
    Database changed
    MariaDB [sample]> create table test(id int,name varchar(20));
    Query OK, 0 rows affected (0.102 sec)

    MariaDB [sample]> insert into test values(1,'test1');
    Query OK, 1 row affected (0.020 sec)

    MariaDB [sample]> insert into test values(2,'test2');
    Query OK, 1 row affected (0.011 sec)

    MariaDB [sample]> exit
    Bye
    pod "mariadb-client" deleted
    PS C:\Users\rurur\OneDrive\Desktop\k8s\mariadb>  kubectl exec -it maxscale-0  --container=maxscale -- maxctrl list servers

    ┌─────────┬─────────────────────────────────────────────────┬──────┬─────────────┬─────────────────┬──────────┐
    │ Server  │ Address                                         │ Port │ Connections │ State           │ GTID     │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────┼──────────┤
    │ server1 │ mariadb-0.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Master, Running │ 0-3000-4 │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────┼──────────┤
    │ server2 │ mariadb-1.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Slave, Running  │ 0-3000-4 │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────┼──────────┤
    │ server3 │ mariadb-2.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Slave, Running  │ 0-3000-4 │
    └─────────┴─────────────────────────────────────────────────┴──────┴─────────────┴─────────────────┴──────────┘

    kubectl delete pod mariadb-0

    kubectl exec -it maxscale-0  --container=maxscale -- maxctrl list servers
    ┌─────────┬─────────────────────────────────────────────────┬──────┬─────────────┬─────────────────┬──────────┐
    │ Server  │ Address                                         │ Port │ Connections │ State           │ GTID     │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────┼──────────┤
    │ server1 │ mariadb-0.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Down            │ 0-3000-4 │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────┼──────────┤
    │ server2 │ mariadb-1.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Master, Running │ 0-3000-4 │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────┼──────────┤
    │ server3 │ mariadb-2.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Slave, Running  │ 0-3000-4 │
    └─────────┴─────────────────────────────────────────────────┴──────┴─────────────┴─────────────────┴──────────┘

    kubectl run -it --rm --image=mariadb:10.4 --restart=Never mariadb-client -- mysql -h maxscale-svc -u maxuser -pmaxpwd  -P 4006
    If you don't see a command prompt, try pressing enter.
    MariaDB [(none)]> use sample;
    Reading table information for completion of table and column names
    You can turn off this feature to get a quicker startup with -A

    Database changed
    MariaDB [sample]> insert into test values(3,'test3');
    Query OK, 1 row affected (0.016 sec)

    MariaDB [sample]> exit

    kubectl exec -it maxscale-0  --container=maxscale -- maxctrl list servers
    ┌─────────┬─────────────────────────────────────────────────┬──────┬─────────────┬─────────────────┬──────────┐
    │ Server  │ Address                                         │ Port │ Connections │ State           │ GTID     │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────┼──────────┤
    │ server1 │ mariadb-0.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Down            │ 0-3000-4 │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────┼──────────┤
    │ server2 │ mariadb-1.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Master, Running │ 0-3001-5 │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────┼──────────┤
    │ server3 │ mariadb-2.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Slave, Running  │ 0-3001-5 │
    └─────────┴─────────────────────────────────────────────────┴──────┴─────────────┴─────────────────┴──────────┘

    kubectl exec -it maxscale-0  --container=maxscale -- maxctrl list servers
    ┌─────────┬─────────────────────────────────────────────────┬──────┬─────────────┬─────────────────┬──────────┐
    │ Server  │ Address                                         │ Port │ Connections │ State           │ GTID     │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────┼──────────┤
    │ server1 │ mariadb-0.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Running         │ 0-3000-0 │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────┼──────────┤
    │ server2 │ mariadb-1.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Master, Running │ 0-3001-5 │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────┼──────────┤
    │ server3 │ mariadb-2.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Slave, Running  │ 0-3001-5 │
    └─────────┴─────────────────────────────────────────────────┴──────┴─────────────┴─────────────────┴──────────┘

    kubectl run -it --rm --image=mariadb:10.4 --restart=Never mariadb-client -- mysql -h mariadb-0.mariadb-svc -u root -proot -P 3306

    MariaDB [(none)]> RESET MASTER;
    Query OK, 0 rows affected (0.060 sec)

    MariaDB [(none)]> STOP SLAVE;
    Query OK, 0 rows affected (0.007 sec)

    MariaDB [(none)]> SET GLOBAL gtid_slave_pos='0-3001-5';
    Query OK, 0 rows affected (0.133 sec)

    MariaDB [(none)]> CHANGE MASTER TO
        ->     MASTER_HOST='mariadb-1.mariadb-svc.default.svc.cluster.local',
        ->     MASTER_PORT=3306,
        ->     MASTER_USER='maxuser',
        ->     MASTER_PASSWORD='maxpwd',
        ->     MASTER_USE_GTID=slave_pos;
    Query OK, 0 rows affected (0.046 sec)

    MariaDB [(none)]> START SLAVE;
    Query OK, 0 rows affected (0.039 sec)

    MariaDB [(none)]> SET GLOBAL gtid_strict_mode=ON;
    Query OK, 0 rows affected (0.002 sec)

    MariaDB [(none)]> exit
    kubectl exec -it maxscale-0  --container=maxscale -- maxctrl list servers
    ┌─────────┬─────────────────────────────────────────────────┬──────┬─────────────┬─────────────────┬──────────┐
    │ Server  │ Address                                         │ Port │ Connections │ State           │ GTID     │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────┼──────────┤
    │ server1 │ mariadb-0.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Slave, Running  │ 0-3001-5 │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────┼──────────┤
    │ server2 │ mariadb-1.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Master, Running │ 0-3001-5 │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────┼──────────┤
    │ server3 │ mariadb-2.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Slave, Running  │ 0-3001-5 │
    └─────────┴─────────────────────────────────────────────────┴──────┴─────────────┴─────────────────┴──────────┘