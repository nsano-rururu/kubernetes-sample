# Version

|Kubernetes|Redis|
|:--------:|:---:|
|1.18.0    |5.0.8|

# デプロイ

    kubectl apply -f redis.yaml

# 動作確認

    kubectl run -it redis-cli --rm --image redis:5.0.8 --restart=Never -- bash
    If you don't see a command prompt, try pressing enter.
    root@redis-cli:/data# redis-cli -c -h redis-svc -p 6379 -a password
    Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
    redis-svc:6379> set a 1
    OK
    redis-svc:6379> get a
    "1"
    redis-svc:6379> exit
    root@redis-cli:/data# exit