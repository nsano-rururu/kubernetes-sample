# Version

|Kubernetes|MariaDB|Maxscale|
|:--------:|:-----:|:------:|
|1.16.2    |10.4.8 |2.4.2   |

# デプロイ

    kubectl apply -f mariadb.yaml

    kubectl get pod

    NAME                     READY   STATUS    RESTARTS   AGE
    mariadb-0   1/1     Running   0          116s
    mariadb-1   1/1     Running   0          92s
    mariadb-2   1/1     Running   0          56s

    kubectl apply -f maxscale.yaml

# 動作確認

    kubectl exec -it maxscale-0  --container=maxscale -- maxctrl list servers
    ┌─────────┬─────────────────────────────────────────────────┬──────┬─────────────┬─────────────────────────┬──────┐
    │ Server  │ Address                                         │ Port │ Connections │ State                   │ GTID │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────────────┼──────┤
    │ dbserv1 │ mariadb-0.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Master, Synced, Running │      │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────────────┼──────┤
    │ dbserv2 │ mariadb-1.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Slave, Synced, Running  │      │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────────────┼──────┤
    │ dbserv3 │ mariadb-2.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Slave, Synced, Running  │      │
    └─────────┴─────────────────────────────────────────────────┴──────┴─────────────┴─────────────────────────┴──────┘

    kubectl delete pod mariadb-0
    kubectl exec -it maxscale-0  --container=maxscale -- maxctrl list servers
    ┌─────────┬─────────────────────────────────────────────────┬──────┬─────────────┬─────────────────────────┬──────┐
    │ Server  │ Address                                         │ Port │ Connections │ State                   │ GTID │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────────────┼──────┤
    │ dbserv1 │ mariadb-0.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Slave, Synced, Running  │      │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────────────┼──────┤
    │ dbserv2 │ mariadb-1.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Master, Synced, Running │      │
    ├─────────┼─────────────────────────────────────────────────┼──────┼─────────────┼─────────────────────────┼──────┤
    │ dbserv3 │ mariadb-2.mariadb-svc.default.svc.cluster.local │ 3306 │ 0           │ Slave, Synced, Running  │      │
    └─────────┴─────────────────────────────────────────────────┴──────┴─────────────┴─────────────────────────┴──────┘

