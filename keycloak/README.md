# Version

|Kubernetes|Keycloak|MariaDB|
|:--------:|:-----:|:------:|
|1.16.2    |7.0.1  |10.4.8  |

# デプロイ

    kubectl apply -f keycloakdb.yaml
    kubectl apply -f keycloak.yaml

# 動作確認

    ブラウザを起動して以下のアドレスにアクセス
    http://マスターノードのIPアドレスまたはホスト名:30080/