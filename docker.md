# docker

## よく使うコマンド
~~~sh
# リポジトリの一覧の表示
$ docker images
# コンテナ一覧の表示
$ docker container ls -a
# コンテナの起動(mysql)
$ docker run --name mariadb -e MYSQL_ROOT_PASSWORD=mariadb -p 3306:3306 -d <image>
# コンテナの起動(web)
$ docker run --name nginx -p 80:80 -d <image>
# 軌道済みのコンテナへ擬似ttyで接続
$ docker exec -it <container> /bin/bash
# コンテナの削除
$ docker rm <container>
# リポジトリの削除
$ docker rmi <image>
~~~

## 参考サイト
[Docker Docs](https://docs.docker.com)  
[Docker チートシート](https://www.qoosky.net/references/206)  
[Docker コンテナ内でサービス起動](http://qiita.com/yunano/items/9637ee21a71eba197345)  

# centos7のイメージ作成とコンテナ起動

公開リポジトリ Docker Hub からイメージを取得。
~~~sh
$ docker search centos
~~~

リポジトリを取得する。  
> [The official build of CentOS.](https://hub.docker.com/r/_/centos/)  
~~~sh
docker pull centos:centos7
~~~

リポジトリが取得できているか確認
~~~sh
$ docker images
~~~

コンテナの作成。-itでターミナルに接続してコマンド入力ができる。
~~~sh
# コンテナを起動。-it:プロセスにttyを割り当て。 -d:コンテナをバックグラウンドで実行
# docker run -it --name <コンテナ名> <レポジトリ名>:<タグ名> /bin/bash
$ docker run -it -d --name centos:centos7 centos:centos7
~~~
> バックグラウンドで起動させたい場合は、-dオプションをつけるが、
Dockerコンテナは起動時にttyが生成されないので、docker runコマンドの後に
オプション -i (--interactive)と-t (--tty) をつけて起動することで、
コンテナに擬似TTY端末を割りあてて対話的にbashが使えるようにする。


コンテナの起動状況を確認する。
~~~sh
# 起動中のコンテナの一覧表示
$ docker ps
$ docker container ls
  CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
  69da5b15e012        centos:centos7      "/bin/bash"         About a minute ago   Up About a minute                       centos7
~~~

コンテナ内でコマンドを送信する。exitで抜ける。
~~~sh
$ docker exec -it centos7 /bin/bash
[root@69da5b15e012 /]# exit
~~~

コンテナの一時停止と再開
~~~sh
# コンテナを一時停止する
$ docker stop centos7
$ docker start centos7
~~~

不要なコンテナとイメージを削除する。
~~~sh
$ docker stop centos7
$ docker rm centos7
$ docker rmi centos:centos7
~~~

# スナップショットの複製

起動中のコンテナをコピーして新しいイメージを作成することができます。
上で作ったコンテナにNginxを導入した直後のイメージを作ってみます。

コンテナを起動して入る
~~~sh
$ docker start centos7
$ docker exec -it centos7 /bin/bash
  [root@69da5b15e012 /]# cat /etc/centos-release
  CentOS Linux release 7.5.1804 (Core) 
~~~

Nginxをインストール
~~~sh
$ rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
$ yum install nginx -y
~~~

リポジトリを複製する前にコンテナから抜けて停止する。
~~~sh
$ docker stop centos7
~~~

スナップショットを複製して保存する。
~~~sh
# docker cimmit <コンテナ名> <リポジトリ名:タグ名>
$ docker commit centos7 centos7/nginx:ver1.0
sha256:795ec9485d253549c842346d9b004971da30b89ef1e73fcdffef26eb0c334150
~~~

作成されたリポジトリを見てみる。
~~~sh
$ docker images

REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
centos7/nginx                  ver1.0              795ec9485d25        5 seconds ago       304MB
centos                         centos7             75835a67d134        3 weeks ago         200MB
~~~

NginxコンテナをWebサーバーとして起動する。
~~~sh
$ docker run --privileged -d -p 8080:80 --name nginx centos7/nginx:ver1.0 /sbin/init
$ docker exec -it nginx /bin/bash
~~~

> CentOS 7ではデーモンがsystemdなので、特権付きかつsystemdでコンテナを起動して、ログインする。通常の起動だと失敗する。

Nginxサービスを起動する。
~~~sh
$ systemctl status nginx.service
  ● nginx.service - nginx - high performance web server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
     Active: inactive (dead)
       Docs: http://nginx.org/en/docs/

$ systemctl enable nginx.service
  Created symlink from /etc/systemd/system/multi-user.target.wants/nginx.service to /usr/lib/systemd/system/nginx.service.

$ systemctl start nginx.service
~~~

起動確認
~~~sh
# ドキュメントルートの確認
$ cat /usr/share/nginx/html/index.html 
$ exit

# ホスト側からつながればOK
$ curl -s http://localhost:8080/ | head

# ログで確認してみる。
$ docker cp nginx:/var/log/nginx/access.log .
$ cat ./access.log
~~~

Nginxの設定が終わったので、リポジトリへ上書きする。
~~~sh
$ docker stop nginx
$ docker commit nginx centos7/nginx:ver1.0
# 再度起動して確認してみる
$ docker run --privileged -d --name nginx -p 8080:80 centos7/nginx:ver1.0 /sbin/init
~~~

> ちなみに上記のようにコンテナへサービスをインストールして、コンテナの中でサービスを起動するというのはdockerの利用想定から外れているので、上記のような面倒な操作が必要。  
通常はnginxのみのコンテナをリポジトリから探して、コンテナを起動すれば良い。例えば[公式のNginxコンテナ](https://hub.docker.com/_/nginx/)は以下の指定だけで使用できる。
>> docker run -d -p 80:80 --name webserver nginx  
debianベースに最低限のパッケージのみでがインストールされたコンテナが起動する。

このように、一つのコンテナに行くつもサービスを入れるのではなく、サービスごとに独立したコンテナを立てて、コンテナ間で通信させたりして利用することで、サービスごとの再利用性を高めることができる。

# ボリューム

コンテナは未使用の時は削除するものなので、mariadbのDBや、apacheのログのように、コンテナ内に保存したデータも一緒に消えてしまう。ボリュームと呼ぶdocker専用のデータ領域にこれらを保存すると、コンテナが消えても残すことができる。

ボリュームを使用する場合は、コンテナの起動時に-vオプションを指定する。なおデータ領域をホストOSのHDDにすることもできるが、権限など面倒な問題が起きることがある。以下ではdockerのボリュームにmariadbのデータを保存するケースで説明する。

mariadbのリポジトリをゲット
~~~sh
$ docker pull mariadb:10.2.11
~~~

ボリューム指定でコンテナを起動する。
~~~sh
$ docker run -d --name mariadb -v mysql-volume:/var/lib/mysql/ -p 3306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=yes mariadb:10.2.11
~~~

上記では、mysql-volume という名前のボリュームが作られ、/var/lib/mysqlの内容が格納される。そのため、コンテナーを消して再起動してもデータが残る。
~~~
$ docker volume ls
  DRIVER              VOLUME NAME
  local               mysql-volume
~~~

データを消したい場合は、コンテナを停止・削除してから、以下のように消す。
~~~
$ docker volume rm mysql-volume
~~~

ホスト上のWEBのコンテンツを/var/www/htmlに直接マウントすると、コーディングしながらコンテナ上で動作確認ができて便利。
~~~
$ docker run -d -v ../html/:/var/www/html/ -v ../logs:/var/www/html/logs --name webserver centos7/nginx
~~~
