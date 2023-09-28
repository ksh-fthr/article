## はじめに

Raspberry Pi3 に Docker をインストールして遊ぶ記事の 3つ目。[前回](https://qiita.com/ksh-fthr/items/e914ff213791b7150008)、[前々回](https://qiita.com/ksh-fthr/items/a26efd6fe06a174525a2)の記事についてご興味あればリンク先をご参照いただければ。

今回は Docker Compose を使った環境構築について｡


## 作業環境

* [Rasppberry Pi3](https://www.raspberrypi.org/)
* [Ubuntu MATE](https://ubuntu-mate.org/raspberry-pi/) 16.04.5 LTS ( GNU/Linux 4.4.38-v7+ armv7l )
* [Docker CE](https://docs.docker.com/install/linux/docker-ce/ubuntu/) ( 18.06.0~ce~3-0~ubuntu )
* [Docker Compose v1.23.0dev](https://docs.docker.com/compose/)
* [Python v3.6.6](https://www.python.org/)
* [Flask v1.0.2](http://flask.pocoo.org/)


## 事前作業

[前回と同じく](https://qiita.com/ksh-fthr/items/e914ff213791b7150008#%E4%BA%8B%E5%89%8D%E4%BD%9C%E6%A5%AD) 事前作業として Docker イメージの削除を行っておく｡


## Docker Compose を使って管理する環境

Docker Compose について触れる前に、まず Docker Compose が管理する対象となる環境について載せておく。

### Dockerfile

[前回のこちらの項目](https://qiita.com/ksh-fthr/items/e914ff213791b7150008#dockerfile-%E3%81%AE%E4%BD%9C%E6%88%90) と同じ。

```bash:Dockerfile
# ベースイメージとして python v3.6 を使用する
FROM python:3.6

# 以降の RUN, CMD コマンドで使われる作業ディレクトリを指定する
WORKDIR /app

# カレントディレクトリにある資産をコンテナ上の ｢/app｣ ディレクトリにコピーする
ADD . /app

# ｢ requirements.txt ｣にリストされたパッケージをインストールする
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Docker に対して｢ 80 番ポート ｣で待受けするよう指定する
EXPOSE 80

# Docker イメージ中の環境変数を指定する
ENV NAME World

# コンテナが起動したときに実行される命令を指定する
# ここでは後述の ｢app.py ｣を実行するよう指示している
CMD ["python", "app.py"]
```

### requirements.txt

[前回](https://qiita.com/ksh-fthr/items/e914ff213791b7150008#requirementstxt-%E3%81%AE%E4%BD%9C%E6%88%90) と異なり、今回は Redis も使いたいので追記している。

```:requirements.txt
Flask
Redis
```

### app.py

こちらも [前回](https://qiita.com/ksh-fthr/items/e914ff213791b7150008#web-%E3%82%A2%E3%83%97%E3%83%AA%E3%81%AE%E4%BD%9C%E6%88%90) と違って Redis 関連のコードを追加している。

```python:app.py
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Redis に接続
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```


## Docker Compose

Dockerfile を利用することで環境構築にかかる手間がかなり楽になるが、Docker Compose を使うともっと楽になる。
具体的に言うと、環境構築に必要な Docker コンテナ群の設定を YAML ファイルで管理することで、各コンテナの導入や設定、起動を Docker Compose 経由で自動化することができる。


### インストール

Docker Compose のインストール手順は [こちら](https://docs.docker.com/compose/install/#install-compose)の *Linux* タブを参考に行ったのだが、ここで提示されているコマンド

```bash:DockerComposeをLinuxにインストールするコマンド
$ sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
```

を実行しても ```docker-compose``` はインストールされない。
理由は ```uname -m``` の部分で、これを単体で実行すると結果は ```armv7l``` となるのだが、このアーキテクチャに対応するリリース資産は [Docker Compose の GitHub](https://github.com/docker/compose/releases/) に存在しないためである。

となると、Raspberry Pi3 に Docker Compose をインストールするには別の方法をとる必要があるのだが、これは既に対応されている方がいた。

* [Raspberry Pi用docker-composeの構築](https://qiita.com/tkyonezu/items/ceaaf41924df39254058)

こちらを参考に作業を行い、無事インストールが完了。
バージョンを確認すると

```bash:DockerComposeのバージョン確認
$ docker-compose version
docker-compose version 1.23.0dev, build 901ee4e
docker-py version: 3.5.0
CPython version: 3.6.4
OpenSSL version: OpenSSL 1.0.1t  3 May 2016
```

と出て、これで Docker Compose を利用する準備が整った。


* 注意

　　上記の参考記事を元にインストール作業を行った後に [本家のインストール手順](https://docs.docker.com/compose/install/#install-compose) を実行すると ```docker-compose``` コマンドへのパスが上書きされて使えなくなる。
　　従って、参考記事の手順で Docker Compose のインストールを行った後は、本家のインストール手順を実行しない方が無難。


### Compose file の作成

前述したが、Docker Compose で自動化するためには YAML ファイル ( デフォルトファイル名は ```docker-compose.yml``` ) でコンテナのアレコレを管理する必要がある。
本記事では次の Compose file を作成し、Docker コンテナの管理を行う。

```yaml:docker-compose.yml
version: '3'
services:
  web:
    build: .
    ports:
     - "4000:80"
    volumes:
     - .:/app
  redis:
    image: "redis:alpine"
```

#### 説明

ファイル名と各プロパティ の説明は次の通り。

* ファイル名
    * ```docker-compose.yml```(デフォルトファイル名)
* version プロパティ
    * ```version: '3'``` とすることで ```swarm mode``` に対応という意味になる
    * ```swarm mode``` についてはクラスタとかの概念の話になってくるのでここでは触れない
* service プロパティ
    * この Compose file で起動するコンテナとして web と redis の 2つを扱う
* web サービス
    * build プロパティ
        * この ```docker-compose.yml``` と同階層にある ```Dockerfile``` をもとにビルドして作成する
    * ports プロパティ
        * ホストの 4000 番ポートとコンテナの 80 番ポートを紐づける( **ホストの 4000 番ポートにアクセスされたらコンテナの 80 番ポートにフォワードする** )
    * volumes プロパティ
        * カレントディレクトリをコンテナの ```/app``` ディレクトリとして定義する
* redis サービス
    * image プロパティ
        * Docker Hub registry から redis のコンテナイメージを取得する

#### 備考

* 本記事で扱っている Docker のバージョンは ```18.06.0``` なので、[こちらのリファレンス](https://docs.docker.com/compose/compose-file/#compose-and-docker-compatibility-matrix) から、対応する Compose file のバージョンは ```3.7``` となる
* Compose file で設定できる内容については本家のリファレンスを見るのが間違いがないので、より踏み込んだ内容を知りたい方は下記を参照されたい
* Compose file のリファレンス
       * [Compose file version 3 reference](https://docs.docker.com/compose/compose-file/)


### Docker Compose によるコンテナの起動

次のコマンドを実行することでコンテナが起動する。

```bash:DockerComposeによるコンテナの起動
$ docker-compose up -d
.... # 以下､実行結果が出力
```

ここで ｢ -d ｣ の指定はコンテナを **バックグラウンドで起動** するという意味。

で、上記コマンド実行時、 Docker Compose はローカル環境の状況に応じて各処理を実施してくれる。
つまりベースイメージがなければベースイメージの取得からコンテナの起動まで、または既にイメージのビルドが済んでいればコンテナ起動だけを行ってくれる動きとなる。


## コンテナの起動確認

```docker-compose up``` を実行したあとの Docker イメージとコンテナの状況を確認すると､

* Docker イメージ

    ```bash:Dockerイメージの確認
    $ docker image ls
    REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
    hello-world-compose_web   latest              8ef63657034d        50 seconds ago      747MB
    redis                     alpine              722b119bf4f2        3 weeks ago         15.1MB
    python                    3.6                 a91b1909dbdd        4 weeks ago         736MB
    ```
Python､redis そして今回の作成対象である hello-world-compose のイメージが作成されていること｡
そして

* Docker コンテナ

    ```bash:Dockerコンテナの確認
    $ docker ps -a
    CONTAINER ID        IMAGE                     COMMAND                  CREATED              STATUS              PORTS                  NAMES
    e7c2585c80c5        hello-world-compose_web   "python app.py"          About a minute ago   Up About a minute   0.0.0.0:4000->80/tcp   hello-world-compose_web_1
    00dbb2060311        redis:alpine              "docker-entrypoint.s…"   About a minute ago   Up About a minute   6379/tcp               hello-world-compose_redis_1
    ````
redis と hello-world-compose_web のコンテナが起動していることがわかる｡


## Web アプリへのアクセス

前項までコンテナの起動まで行ったので､最後に Web アプリへのアクセス結果を載せる｡
「 **ホストのIPアドレス:4000** 」に対してブラウザからアクセスした結果が下記｡

![docker-compose01.png](https://qiita-image-store.s3.amazonaws.com/0/193342/84b2111f-fbac-261c-3b2c-aee8ef7656c4.png)

Visits にはアクセス回数が出力されているが､これは前掲の app.py における

```python:app.pyから抜粋
# 省略

def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

# 省略
```

の

```python:appy.pyからアクセスカウンタの部分を抜粋
visits = redis.incr("counter")
```

の部分｡
Redis コンテナが起動しているので ```try``` ブロックが正常に評価されている｡


### Docker Compose によるサービスの停止

起動したサービスは次のコマンドで停止する。
このコマンドだけだと Docker コンテナは停止しない。

```bash:DockerComposeによるコンテナの停止
$ docker-compose stop
```


## まとめ

* Dockerfile から更に簡単に環境構築するために Docker Compose を使うと便利
    * なんせ Dockerfile を使ってのイメージのビルドや [Docker Hub](https://hub.docker.com/) からのイメージのダウンロードをやってくれる
    * 更にコンテナの起動まで行ってくれる
* [Compose file の作成](#compose-file-の作成) で ```swarm mode``` という単語が出てきた
    * これに触れるとクラスタとかの話になって ```Swarm``` とか ```Kubernetes``` が出てくる
    * が、まだそちらは触ってもいないので、今はそこまでするつもりはない
    * 更に言うと ```Swarm``` も ```Kubernetes``` もどちらも Docker がネイティブサポートしているということで、どっちを学習するのが良いのか...
    * 両方知っているのが良いのだろうけれど、それは必要になったらでいいかな


## 参考

* [Overview of Docker Compose](https://docs.docker.com/compose/overview/)
* [Get started with Docker Compose](https://docs.docker.com/compose/gettingstarted/)
* [Compose file version 3 reference](https://docs.docker.com/compose/compose-file/)
* [Raspberry Pi用docker-composeの構築](https://qiita.com/tkyonezu/items/ceaaf41924df39254058)
