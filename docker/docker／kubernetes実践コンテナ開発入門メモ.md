# Docker/Kubernetes 実践コンテナ開発入門メモ

次の書籍を読んだ時のメモです。

- Dockrt/Kubernetes 実践コンテナ開発入門 著者:山田明憲 出版社:技術評論社

---

### ■docker image pull の例の jenkins イメージのタグ変更

```shell
docker image pull jenkins
Using default tag: latest
Error response from daemon: manifest for jenkins:latest not found: manifest unknown: manifest unknown
```

latest タグが存在しないため、存在するタグに変更する
タグは以下から確認出来る。  
https://hub.docker.com/r/jenkins/jenkins

```
docker image pull jenkins/jenkins:lts-jdk11
lts-jdk11: Pulling from jenkins/jenkins
3e440a704568: Pull complete
4e7ee60831ad: Pull complete
d81b0ac1dd8f: Pull complete
aa4d217f8f45: Pull complete
328c85129d49: Pull complete
df967cd9dca8: Pull complete
159cf70711c2: Pull complete
Digest: sha256:aacbb5797dd210cc048038d9d3e5ab5795ea018fad843ffc1888c547911819ce
Status: Downloaded newer image for jenkins/jenkins:lts-jdk11===========>     ]  69.63MB/76.93MB
docker.io/jenkins/jenkins:lts-jdk11
```

■Compose による複数コンテナの実行における docker-compose.yml の jenkins イメージのタグ修正

適用なタグを指定します。

```shell
version: "3"
services:
  master:
    container_name: master
    image: jenkins:2.60.3
    ports:
      - 8080:8080
    volumes:
      - ./jenkins_home:/var/jenkins_home
```

■apt update でタイムアウト

Dockerfile でイメージの古いタグを指定すると apt update がタイムアウトする

### Dockerfile

```
FROM ubuntu:16.04

RUN apt update
```

ビルドすると次のような出力でビルドが中断される

```
 > [2/6] RUN apt update:
#5 0.529
#5 0.529 WARNING: apt does not have a stable CLI interface. Use with caution in scripts.
#5 0.529
#5 2.145 Get:1 http://security.ubuntu.com/ubuntu xenial-security InRelease [99.8 kB]
#5 2.170 Get:2 http://archive.ubuntu.com/ubuntu xenial InRelease [247 kB]
#5 122.8 Ign:1 http://security.ubuntu.com/ubuntu xenial-security InRelease
#5 123.1 Ign:2 http://archive.ubuntu.com/ubuntu xenial InRelease
#5 123.9 Get:3 http://security.ubuntu.com/ubuntu xenial-security Release [98.8 kB]
#5 124.3 Get:4 http://archive.ubuntu.com/ubuntu xenial-updates InRelease [99.8 kB]
#5 244.4 Err:3 http://security.ubuntu.com/ubuntu xenial-security Release
#5 244.4   Connection timed out [IP: 185.125.190.36 80]
#5 244.9 Ign:4 http://archive.ubuntu.com/ubuntu xenial-updates InRelease
#5 246.2 Get:5 http://archive.ubuntu.com/ubuntu xenial-backports InRelease [97.4 kB]
#5 366.8 Ign:5 http://archive.ubuntu.com/ubuntu xenial-backports InRelease
#5 367.3 Get:6 http://archive.ubuntu.com/ubuntu xenial Release [246 kB]
#5 488.3 Err:6 http://archive.ubuntu.com/ubuntu xenial Release
#5 488.3   Connection timed out [IP: 185.125.190.36 80]
```

イメージのタグを新しいものに更新すれば解決する。

■docker swarm で repository does not exist

```
Error response from daemon: pull access denied for docker18.05.0-ce-dind, repository does not exist or may require 'docker login': denied: requested access to the resource is denied
```

docker login では治らず  
docker hub でパッケージを検索するとリポジトリが存在しなかったので  
docker:23.0.3-dind に置き換えました。

■mysql:5.7 イメージビルド時に 0.840 /bin/sh: apt-get: command not found  
ここにある通り、OS を変更するとビルドが通るようになる。  
https://syoukinkubi.com/?p=941

■Swarm によるデプロイで失敗  
テキストの通りに実施すると以下のようなエラーになる

```
docker container exec -it manager docker stack deploy -c /stack/todo-mysql.yml todo_mysql
open /stack/todo-mysql.yml: no such file or directory
```

解決方法は以下のサイトの通り  
https://dimn-zkym.hatenablog.com/entry/2020/02/29/173955

■kubernetes ダッシュボードのポッドの状態が確認できない  
書籍の通りだと指定の通りの名前空間はないと言われる。

```
kubectl get pod --namespace=kube-system -l k8s-app=kubernetes-dashboard
No resources found in kube-system namespace.
```

名前空間も kubernetes-dashboard にしてやると確認できる。

```
kubectl get pod --namespace=kubernetes-dashboard -l k8s-app=kubernetes-dashboard
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-dashboard-5958dcc96-mf2fn   1/1     Running   0          12m
```

参考:https://stackoverflow.com/questions/57835272/no-resources-found-when-installing-kubernetes-dashboard

■kubernets-dashboard アクセス用のトークンが自動生成されない  
サービスアカウントを作成し、トークン生成コマンドを実行する必要がある

■kubernetes-dashbord が 404 エラー表示される  
下記 Issue にあるように昔からある dashboard のバグのようです。  
ページロード直後に色んな所をクリックすると正常に機能します。  
https://github.com/kubernetes-sigs/kubespray/issues/5347

■kubernetes の ingress の apiVersion の変更

```
apiVersion: extensions/v1beta1
```

が廃止になり、

```
apiVersion: networking.k8s.io/v1
```

になった。
その他記述方法が微妙に変わっている。
https://kubernetes.io/ja/docs/concepts/services-networking/ingress/

■helm init を実行すると Error: unknown command "init" for "helm"になる  
helm のバージョンアップにより、init は不要になりました。
https://github.com/helm/helm/issues/6996

■fluentd イメージの実行に失敗する

```
Error response from daemon: failed to initialize logging driver: dial tcp [::1]:24224: connect: connection refused
```

取り合えず暫定で、log ドライバーの指定アドレス localhost から固定 IP にしたらエラーは発生しないようになりました。

```
 logging:
      driver: "fluentd"
      options:
        fluentd-address: "192.168.0.36:24224"
```

■fluent-pluguin-elasticsearch のインストールで失敗する  
Docker イメージの Ruby バージョンが古いようです。

```
 => ERROR [2/3] RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-rdoc", "--no-ri", "--version", "1.9.2"]                               5.5s
------
 > [2/3] RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-rdoc", "--no-ri", "--version", "1.9.2"]:
#6 5.369 ERROR:  Error installing fluent-plugin-elasticsearch:
#6 5.369        faraday-net_http requires Ruby version >= 2.6.0.
```

■ 訂正メモ  
https://gihyo.jp/assets/files/book/2018/978-4-297-10033-9/download/pdf.md20190828.pdf
