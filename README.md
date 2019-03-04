# 起動
```
docker-compose up -d
```
初回起動時にmlflowコンテナは自動的にビルドされる

mlflowコンテナを明示的にビルドしたい場合は、下記コマンドを実行する
```
docker-compose build mlflow
```

# セットアップ
## gitlab
1. http://localhost:80 にアクセスし初期パスワード設定
2. rootアカウントでログイン
3. root/work リポジトリを作成しておく
4. http://localhost/admin/runners からrunner用tokenを入手

## gitlab-runner
1.  gitlab-runnerコンテナにログイン
```
docker-compose exec runner bash
```
2. gitlabにアクセスできるか確認(htmlが返るか確認)
```
curl gitlab
```
3. gitlab-runnerのgitlabへの登録

```
docker exec -it intern_runner_1 gitlab-runner register \
  --non-interactive \
  --name docker-runner \
  --url http://gitlab/ci \
  --registration-token ya21qvbyXiT8sPHDKzxe \
  --executor docker \
  --limit 3 \
  --docker-image python:3.6 \
  --docker-volumes /var/run/docker.sock:/var/run/docker.sock \
  --tag-list docker
```
4. http://localhost/admin/runners にアクセスし\
3 で登録したrunnerのTagsを```docker```とする

5. http://localhost/admin/runners にrunnerが登録されていることを確認


## jupyter (mlflow連携)

1. http://localhost:10000 にアクセス
2. terminalを開く
3. サンプルコード(work/main.ipynb)を実行する
4. http://localhost:5000 にアクセスし履歴が残っている確認する

## hosts指定

1. docker network inspectで compose のネットワークのGatewayを調べる
```
~ $ d network  inspect intern_default
[
    {
        "Name": "intern_default",
        "Id": "8944c6239aff1d55fc25f30c0ee4d4d45801a4cdbae3e45e8d137c2476d959e3",
        "Created": "2019-02-02T06:20:43.2183532Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "192.168.160.0/20",
                    "Gateway": "192.168.160.1"
                }
            ]
        },
```
2. GatewayのIPアドレスを以下のようにホストPCに設定する
```
~ $ cat /etc/hosts
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1	localhost
255.255.255.255	broadcasthost
::1             localhost
192.168.128.1 gitlab
192.168.128.1 mlflow
```

## jupyter (gitlab連携)
1. terminalを開く
2. workディレクトリに移動しgitリポジトリ作成\
ちなみにterminal上でのペーストはShift+insertでできる
```
cd work
git config --global user.name "Administrator"
git config --global user.email "admin@example.com"

git init
git remote add origin http://gitlab/root/work.git
git add .
git commit -m "Initial commit"
git push -u origin master
```
3. リポジトリにアクセスして実行結果を見る

## mlflowAPI
1. mlflowの任意のジョブのページにアクセスする
2. ArtifactのFullPathを確認する
```
Full Path: /opt/mlflow/0/8503bacfd46446e9ba4cfd55c10a528a/artifacts/ml_models
Size: 0B
```
3. mlflowコンテナよりサーバーを起動する
```
docker exec -it intern_mlflow_1 /bin/bash
mlflow pyfunc serve -p 1234 --model-path /opt/mlflow/0/680a60a248774b2ea3fdf9ccc6f709ef/artifacts/ml_models -h 0.0.0.0
```

4. リクエストを確認する
- 4次元入力
```
curl -X POST -H "Content-Type:application/json" --data '[[ 1,  0,  1.4,  0]]' http://mlflow:1234/invocations
```
- 応答(クラスタリング)
```
[0]
```


