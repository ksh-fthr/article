
# 前提

* Windows 10 Pro 64bit メモリ 4G 以上を対象
    * Windows 10 以外だったり、Windows10 でも Home だったり 32bit の場合は対象外


# 環境

* Windows 10 Pro 64bit メモリ 8G
* Docker for Windows v18.09.1
* Docker Compose v1.23.2


# Docker のインストール

## 事前準備( Hyper-V を有効化する )

Microsoft のドキュメントがあるので、それに従い作業すれば良い。

* [Windows 10 上に Hyper-V をインストールする](https://docs.microsoft.com/ja-jp/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v)

やり方は大きく分けて3つ。

1. [PowerShell を使用して Hyper-V を有効にする](https://docs.microsoft.com/ja-jp/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v#enable-hyper-v-using-powershell)
1. [CMD と DISM を使用して Hyper-V を有効にする](https://docs.microsoft.com/ja-jp/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v#enable-hyper-v-with-cmd-and-dism)
1. [[設定] で Hyper-V ロールを有効にする](https://docs.microsoft.com/ja-jp/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v#enable-the-hyper-v-role-through-settings)

上記いずれかの方法で Hyper-V をインストールしてやる。

## Docker for Windows Installer のダウンロード

[公式-Install Docker for Windows](https://docs.docker.com/docker-for-windows/install/) の [Install Docker for Windows desktop app](https://docs.docker.com/docker-for-windows/install/#install-docker-desktop-for-windows-desktop-app) 項中の文中にある **download.docker.com** からダウンロードできる。

## Docker for Windows のインストール

ダウンロードしたインストーラを起動し、ウィザードに従ってインストールを行う。

### バージョン確認

コマンドプロンプトから次を実行してバージョンを確認する。

```sh:Dockerのバージョン確認
$ docker version
```

実行結果は下記のとおり。

```
Client: Docker Engine - Community
 Version:           18.09.1
 API version:       1.39
 Go version:        go1.10.6
 Git commit:        4c52b90
 Built:             Wed Jan  9 19:34:26 2019
 OS/Arch:           windows/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          18.09.1
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.10.6
  Git commit:       4c52b90
  Built:            Wed Jan  9 19:41:49 2019
  OS/Arch:          linux/amd64
  Experimental:     false
```

### Hello World の確認

次に以下を実行して Docker の動作確認とする。

```sh:hello-worldの確認
$ docker run hello-world
```

実行結果は下記のとおり。
エラーが発生することなく実行結果が確認できる。

```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

## Docker Compose

Docker Compose も公式に手順が載っているので、それに従って作業を進める。
公式の手順は [Install Docker Compose](https://docs.docker.com/compose/install/) の [Install Compose](https://docs.docker.com/compose/install/#install-compose) から **Windows タブ**となる。

実際に行う内容はというと、

1. Powershell を **管理者権限** で起動する
1. 次のコマンドを実行して `TLS 1.2` を有効化する。これは GitHub が `TLS 1.2` のみ受け付けているから。
    (参考: [PowerShell で GitHub に wget(Invoke-WebRequest) できない場合の対処方法](https://qiita.com/tikkss@github/items/4db7bec95a7337ca826d))

    ```
    $ [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    ```

1. 次のコマンドを実行して Docker Compose をインストールする

    ```
    $ Invoke-WebRequest "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-Windows-x86_64.exe" -UseBasicParsing -OutFile $Env:ProgramFiles\docker\docker-compose.exe
    ```

    ここで `1.23.2` の部分はインストールしたい Docker Compose のバージョンを指定する。
    今回は現時点の安定版である `1.23.2` を指定した( [docker/compose - release](https://github.com/docker/compose/releases) )。


### バージョン確認

コマンドプロンプトから `docker-compose` に次のコマンドを指定して実行すると､ Docker Compose のバージョン情報を確認できる。

```sh:versionコマンドを実行してバージョンを確認
$ docker-compose version
```

実行結果は下記のとおり。

```
docker-compose version 1.23.2, build 1110ad01
docker-py version: 3.6.0
CPython version: 3.6.6
OpenSSL version: OpenSSL 1.0.2o  27 Mar 2018
```

### バージョン確認(2)

[注記]==>ここから
@rapidliner00 様から頂戴したコメントより､`--version` オプションを指定したケースを追記しました｡
@rapidliner00 様､コメントありがとうございます｡
<==ここまで

上記とは別の方法として､ `docker-compose` に次のオプションを指定してもバージョンを確認することができる｡

```sh:オプションを指定してバージョンを確認
$ docker-compose --version
```

こちらの実行結果は下記となる｡

```
docker-compose version 1.23.2, build 1110ad01
```

ここまでで Docker と Docker Compose がインストールできたので、実際に Docker Compose 経由でコンテナを起動させてみる。


# Docker Compose 経由でコンテナを起動する

## 扱う資産の構成と内容

扱う資産の構成は次のとおり。
( これらの資産は [[Docker] Docker Compose の導入と環境構築](https://qiita.com/ksh-fthr/items/2b899e3e4c918707afb0) で扱ったものと同じ構成と内容 )

```:構成
docker-sample/
├─ app.py
├─ docker-compose.yml
├─ Dockerfile
└─ requirements.txt
```

以下に各ファイルの内容を示す。

* app.py

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

* docker-compose.yml

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

* Dockerfile

```sh:Dockerfile
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

* requirements.txt

```
Flask
Redis
```

## Docker Compose によるコンテナとサービスの起動

以降の内容も [[Docker] Docker Compose の導入と環境構築](https://qiita.com/ksh-fthr/items/2b899e3e4c918707afb0) で扱ったものと内容となるが、本記事でも改めて確認していく。

次のコマンドを実行することでコンテナが起動する。

```sh:DockerComposeによるコンテナの起動
$ docker-compose up -d
Starting docker-sample_web_1   ... done
Starting docker-sample_redis_1 ... done
```

ここで ｢ -d ｣ の指定はコンテナを **バックグラウンドで起動** するという意味。
上記コマンド実行時、 Docker Compose はローカル環境の状況に応じて各処理を実施してくれる。
つまりベースイメージがなければベースイメージの取得からコンテナの起動まで、または既にイメージのビルドが済んでいればコンテナ起動だけを行ってくれる動きとなる。

なお `docker-compose` の各種コマンドについては [Overview of docker-compose CLI](https://docs.docker.com/compose/reference/overview/) に概要があり､また左側のペインからコマンド詳細を確認できる｡
各コマンドの動きを把握したい場合は上記リンクを参照すると良い｡ご参考まで｡

## コンテナの起動確認

`$ docker-compose up` を実行したあとの Docker イメージとコンテナの状況を確認すると､

* Docker イメージ

```sh:Dockerイメージの確認
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
docker-sample_web   latest              c6aceb937568        About a minute ago   932MB
python              3.6                 55fb8aca33df        7 days ago           922MB
redis               alpine              b42dc832c855        3 weeks ago          40.9MB
```

Python､redis そして今回の作成対象である `docker-sample_web` のイメージが作成されていることが確認できる｡
そして

* Docker コンテナ

```sh:Dockerコンテナの確認
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
8441040aecdc        docker-sample_web   "python app.py"          3 minutes ago       Up 3 minutes        0.0.0.0:4000->80/tcp   docker-sample_web_1
2295ecb5a3c1        redis:alpine        "docker-entrypoint.s…"   3 minutes ago       Up 3 minutes        6379/tcp               docker-sample_redis_1
```

redis と docker-sample_web のコンテナが起動していることがわかる｡

## Web アプリへのアクセス

Web アプリの動作確認は `ホストのIPアドレス:4000` に対してブラウザからアクセスして確認する。
![docker_windows03.png](https://qiita-image-store.s3.amazonaws.com/0/193342/624c89f8-206d-2a28-bb84-02b209292894.png)
※ Visits にはアクセス回数

# Docker Compose によるサービスの停止

起動したサービスは次のコマンドで停止する。
このコマンドだけだと Docker コンテナは削除されない。

```sh:DockerComposeによるサービスの停止
$ docker-compose stop
Stopping docker-sample_web_1   ... done
Stopping docker-sample_redis_1 ... done
```

# Docker Compose によるコンテナの停止

Docker Compose から起動したコンテナを停止する場合は次のコマンドを使用する。
このときネットワークも一緒に削除してくれる。

```sh:DockerComposeによるコンテナの停止
$ docker-compose down
Stopping docker-sample_web_1   ... done
Stopping docker-sample_redis_1 ... done
Removing docker-sample_web_1   ... done
Removing docker-sample_redis_1 ... done
Removing network docker-sample_default
```

# ハマりどころ

## Share drive の設定

Docker インストール完了したあとに Share drive の設定を行っていないと、`$ docker-compose up -d` を実行したときに次の表示が出る。
![docker_windows01.png](https://qiita-image-store.s3.amazonaws.com/0/193342/0f3a70c4-2967-2ff7-08c9-1e6a56f4b7d3.png)
これで `Share it` をクリックせずに一定時間経過すると、表示は消えて `$ docker-compose up -d` がエラー終了する。

```:エラー終了時のログ
ERROR: for docker-sample_web_1  Cannot create container for service web: b'Drive has not been shared'

ERROR: for web  Cannot create container for service web: b'Drive has not been shared'
ERROR: Encountered errors while bringing up the project.
```

対処方法は 2点。

1. 画面表示が出た際に `Share it` をクリックする
1. 予め `Share drive` を設定しておく

2 の設定方法は次のとおり。

1. タスクバーの Docker アイコンを右クリック
1. メニューが表示されるので `settings` をクリック
1. 設定画面が表示されるので `Shared Drives` をクリック
1. `C` ドライブのチェックボックにチェックを入れて `Apply` クリック

設定後の画面
![docker_windows02.png](https://qiita-image-store.s3.amazonaws.com/0/193342/19c8ec18-6041-47ca-809d-bb5d6140d172.png)
これで `Share drive` が設定された。
なお前掲の 1 の方法で `Share it` を実行すれば、あとは設定が残るので改めて 2 の方法を行う必要はない。

## Firewall の設定を見直す

セキュリティソフトを入れている場合、次のエラーが発生するケースがある。

```:Firewallによるエラー
docker-sample_redis_1 is up-to-date
Creating docker-sample_web_1 ... error

ERROR: for docker-sample_web_1  Cannot create container for service web: b'Drive sharing seems blocked by a firewall'

ERROR: for web  Cannot create container for service web: b'Drive sharing seems blocked by a firewall'
ERROR: Encountered errors while bringing up the project.
```

この場合は自分の環境に応じて Firewall の設定を見直してやれば良い。

## Webサービスが起動していない

`http://localhost:4000` を表示した際に `このサイトにアクセスできません` といったメッセージが表示された場合、Webサービスが起動していない可能性がある。
そんなときは `$ docker-compose up -d` の `-d` をつけずに、コンテナの起動を確認してみる。

```:バックグラウンドの指定をせずに起動して確認
$ docker-compose up
# ... 省略
web_1    | python: can't open file 'app.py': [Errno 2] No such file or directory
docker-sample_web_1 exited with code 2
# ... 省略
```

上記例のようなメッセージが出ていた場合、`docker-compose.yaml` を次のように修正する。
( 参考: [python: can't open file 'app.py'](https://github.com/docker/compose/issues/1616) 中の [コメント](https://github.com/docker/compose/issues/1616#issuecomment-309419750) )

```yaml:docker-compose.yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "4000:80"
# volumes をコメントアウト or 削除
#    volumes:
#      - .:/app
  redis:
    image: "redis:alpine"
```

この状態でもう一度 `$ docker-compose up` を実行すれば

```:volumesをコメントアウト(もしくは削除)した状態で起動
$ docker-compose up
# ... 省略
web_1    |  * Serving Flask app "app" (lazy loading)
web_1    |  * Environment: production
web_1    |    WARNING: Do not use the development server in a production environment.
web_1    |    Use a production WSGI server instead.
web_1    |  * Debug mode: off
web_1    |  * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
# ... 省略
```

と、正常に起動したことが確認できて( あくまで導入記事なので `WARNING` は無視 )、更に `http://localhost:4000` にブラウザでアクセスすれば [Web アプリへのアクセス](#web-アプリへのアクセス) で添付した画面が表示されることが確認できる。


# おまけ

Docker コンテナの停止と削除、Docker イメージの削除を行いたい場合は次のコマンドを実行する

## Docker コンテナの停止

```sh:Dockerコンテナの停止
$ docker stop 4b3ca0a2590c
```

`4b3ca0a2590c` は `$ docker ps -a` で出力された `CONTAINER ID` で、スペース区切りで複数指定可能。

## Docker コンテナの削除

```sh:Dockerコンテナの削除
$ docker rm 4b3ca0a2590c
```

Docker コンテナの停止と同じく、`4b3ca0a2590c` は `$ docker ps -a` で出力された `CONTAINER ID` 。
こちらもスペース区切りで複数指定可能。

先にコンテナの停止をしておかないと、削除は上手くいかないので注意すること。

## Docker イメージの削除

```sh:Dockerイメージの削除
$ docker rmi 2f1c802f322f
```

`2f1c802f322f` は `$ docker image ls` で出力された `IMAGE ID`。
こちらもスペース区切りで複数指定可能。


# 参考

* [Overview of docker-compose CLI](https://docs.docker.com/compose/reference/overview/)
* [PowerShell で GitHub に wget(Invoke-WebRequest) できない場合の対処方法](https://qiita.com/tikkss@github/items/4db7bec95a7337ca826d)
* [docker-compose コマンドまとめ](https://qiita.com/wasanx25/items/d47caf37b79e855af95f)
