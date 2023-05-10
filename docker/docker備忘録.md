- DockerForWindows の環境が頻繁に壊れる場合、Docker Toolbox での使用を検討する(Windows Update や wsl の更新等で壊れる場合あり)
  今だと Podman とかも選択肢に入る

* docker イメージは命令毎にレイヤーが構成されるので、命令はまとめた方が、イメージサイズが減らせる。ただ、そのような小細工するよりはマルチステージビルドを使用してビルドコンテナと実行コンテナを分離した方が良い。

* docker container run に-d オプションを付けるとバックグラウンドで実行できる

- docker コマンドは新旧あり、新の方が明示的になっているので基本的に新を使用する。
  - 旧
  ```
  docker 操作
  docker stop
  ```
  - 新
  ```
  docker 対象 操作 オプション
  docker container stop
  ```
- ポートフォワーディングで host 側を省略

  - 省略なし

  ```shell
  docker container run -d -p 9000:8080 example/echo/latest
  ```

  - 省略あり

  ```shell
  docker container run -d -p 8080 example/echo/latest
  ```

  省略したホスト側の port 確認

  ```
  docker container ls
  CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                     NAMES
  51abc49c6558   example/echo/latest   "go run /echo/main.go"   9 seconds ago   Up 7 seconds   0.0.0.0:54427->8080/tcp   fervent_mayer
  ```

- Docker イメージの latest の使用は非推奨。再現性がなくなる可能性があるため。

- Docker container 起動時に--name を指定することでコンテナ名を指定できる。指定しない場合は、起動時に動的に命名されるため、コンテナ名を指定した操作をする前に ls コマンドで確認する手間を省ける。

- コンテナの削除
  動作中のコンテナを削除しようとすると,次のように削除できない。
  ```shell
  docker container rm echo1
  Error response from daemon: You cannot remove a running container af324fc162cc9a6ea71c9e14868404796daf6d64502790549b7ce2b52b86e2ed. Stop the container before attempting removal or force remove
  ```
  force オプションにより動作中のコンテナも削除出来る
  ```
  docker container rm -f echo1
  echo1
  ```
- コマンドツール実行用のコンテナなど起動後自動停止するようなコンテナの場合、run コマンドに rm オプションを付加すると自動削除できる

- 1 コンテナ 1 プロセスは無理がある場合がある。公式ドキュメントでは 1 コンテナは１つの関心事に集中すべきという金言がある、

- Docker Swarm を使用する場合に、Docker in Docker で Docker の中に複数ホストを立てる方法と現実に複数ホストを立てる方法がある。  
  現実的なソリューションでは複数ホストを立てた方が良いと思いますが、勉強目的などであれば Docker in Docker は便利。

- visualizer でコンテナの配置を可視化できる
  ![localhost_9000_](https://user-images.githubusercontent.com/49807271/231191058-b06d1998-faa0-4fd6-b899-a6ef5d9fb6d1.png)

- entrykit を使用すると Dokcerfile が動的に記述できる

- docker desktop のアンインストール後に docker で使用していたディストリビューションが残っている場合がある (C:\Users\user\AppData\Local\Docker)。  
  これが 10GB 程度は平気で超えているので削除した方が良い。

```
C:\Users\user>wsl --unregister docker-desktop-data
登録解除。
この操作を正しく終了しました。

C:\Users\user>wsl --unregister docker-desktop
登録解除。
この操作を正しく終了しました。
```

- kubernetes にデプロイされているコンテナ等を確認できるダッシュボードの適用
  - https://kubernetes.io/ja/docs/tasks/access-application-cluster/web-ui-dashboard/

上記ページにあるコマンドを実行

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```

- Extension MarketPlace にある DiskUsage をインストールすると docker のディスク使用量が確認できるようになる  
  右側の「Reclaim space」ボタンを押すと、キャッシュやコンテナーの削除が出来る。
  ![DiskUsage](https://user-images.githubusercontent.com/49807271/232188433-2e77945b-9edb-4395-b1cb-ff15c1b218fb.png)

* kubenetes の API の確認方法

```
PS C:\Users\user> kubectl api-versions
admissionregistration.k8s.io/v1
apiextensions.k8s.io/v1
apiregistration.k8s.io/v1
apps/v1
authentication.k8s.io/v1
authorization.k8s.io/v1
autoscaling/v1
autoscaling/v2
autoscaling/v2beta2
batch/v1
certificates.k8s.io/v1
coordination.k8s.io/v1
discovery.k8s.io/v1
events.k8s.io/v1
flowcontrol.apiserver.k8s.io/v1beta1
flowcontrol.apiserver.k8s.io/v1beta2
networking.k8s.io/v1
node.k8s.io/v1
policy/v1
rbac.authorization.k8s.io/v1
scheduling.k8s.io/v1
storage.k8s.io/v1
storage.k8s.io/v1beta1
v1
```
