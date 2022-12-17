## はじめに

Raspberry Pi3 に Docker をインストールして遊ぶ記事の 2つ目。前回の記事は [こちら](https://qiita.com/ksh-fthr/items/a26efd6fe06a174525a2)。
今回の記事では Dockerfile を使って Web アプリを Docker 上に構築する手順について触れる｡


## 作業環境

* [Rasppberry Pi3](https://www.raspberrypi.org/)
* [Ubuntu MATE](https://ubuntu-mate.org/raspberry-pi/) 16.04.5 LTS ( GNU/Linux 4.4.38-v7+ armv7l )
* [Docker CE](https://docs.docker.com/install/linux/docker-ce/ubuntu/) ( 18.06.0~ce~3-0~ubuntu )
* [Python v3.6.6](https://www.python.org/)
* [Flask v1.0.2](http://flask.pocoo.org/)


## 事前作業

Docker 上に Web アプリを構築するために Docker イメージの作成を行うのだが､環境が真っ白な状態のほうが作業の状況がわかりやすい｡
ということで､まずは事前作業としてホスト上に Docker イメージが存在しない環境にするために Docker イメージの削除を行う｡

### Docker イメージの削除

1. まずは現在のイメージの一覧を確認する

    ```bash:Dockerイメージのリストを確認する
    $ docker image ls
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    hello-world         latest              434c71a1ad35        15 minutes ago      747MB
    python              3.6                 a91b1909dbdd        3 weeks ago         736MB
    ```

1. 次に ``` IMAGE ID``` を指定してイメージを削除する｡

    ```bash:Dockerイメージを削除する
    $ docker rmi  434c71a1ad35 a91b1909dbdd
    .... # 以下､実行結果が出力
    ```

1. Docker イメージが無くなっていることを確認する｡

    ```bash:Dockerイメージが存在しないことを確認する
    $ docker image ls
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    # 一件も出力されない
    ```

これで作業の準備が整った｡


### 補足: Docker コンテナの停止と削除

もし上記のイメージ削除で次のエラーが出た場合は **当該イメージのコンテナが起動している** ことを示している｡

```bash:Dockerイメージの削除でエラーが発生
Error response from daemon: conflict: unable to delete 434c71a1ad35 (cannot be forced) - image is being used by running container 03e7f7ea4734
```

この場合は下記コマンドを実行して起動しているコンテナの確認を行い､停止と削除を行った後､再度イメージの削除を実施すれば良い｡

```bash:Dockerコンテナの確認
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                  NAMES
03e7f7ea4734        hello-world         "python app.py"     4 minutes ago       Up 4 minutes        0.0.0.0:4000->80/tcp   romantic_nobel
```

```bash:Dockerコンテナの停止と削除
$ docker stop 03e7f7ea4734 && docker rm 03e7f7ea4734
03e7f7ea4734
03e7f7ea4734
```

これでコンテナがいなくなっているはず｡

```bash:Dockerコンテナをもう一度確認する
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
# 一件も出力されない
```

再度､ [Docker イメージの削除](#docker-イメージの削除) からイメージの削除を行って事前準備の完了となる｡


## Web アプリの構築

開発言語は Python 3.6.x。
Web フレームワークは Flask を使用して｢ Hello World ｣を表示するだけの単純な Web アプリを開発する｡
( 基本的に [こちら](https://docs.docker.com/get-started/part2/#build-the-app) の内容をなぞる感じ )


### 作業ディレクトリ

次のコマンドを実行して､後続の作業を行う作業ディレクトリを作成する｡

```bash:作業ディレクトリを作成する
$ mkdir hello-world
```

あとは作成した ```hello-world``` ディレクトリに移動。その配下での作業となる｡

### Dockerfile の作成

さて、 Web アプリを作成するとしても、その Web アプリを載せるためのコンテナが必要となる｡
というわけで､コンテナを作るための手段として次の内容を記載した Dockerfile を作成する｡

```bash:Dockerファイル
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

* Dockerfile で扱えるコマンドについては本家のリファレンスを見るのが間違いがないので、より踏み込んだ内容を知りたい方は下記を参照されたい
* Dockerfile のリファレンス
       * [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

### requirements.txt の作成

ここでは前項で作成した Dockerfile 中に出てきた requirements.txt を作成する｡
このファイルには Web アプリで使用するパッケージのリストとしての役割を持たせたい｡

今回は Flask を使用するので､ファイルの中身は

```:requirements.txt
Flask
```

となる｡


### Web アプリの作成

Web アプリの実体をコーディングする｡
内容はルートパス( '/' )にアクセスされたら､｢ Hello World ｣を表示するだけ｡

```python:app.py
from flask import Flask
import os
import socket

app = Flask(__name__)

@app.route("/")
def hello():
  html = "<h3>Hello World!</h3>"
  return html.format()

if __name__ == "__main__":
  app.run(host='0.0.0.0', port=80)
```


### Docker コンテナのビルドと起動

では実際にコンテナを起動して動きを見てみるが､その前に Docker イメージを作成しておく必要がある｡
流れとしては

1. Docker イメージの作成
2. Docker コンテナの起動

となる｡


### Docker イメージのビルド

1. まず次のコマンドを実行して Docker イメージが一つも存在しないことを確認する｡

    ```bash:Dockerイメージが存在しないことを確認する
    $ docker image ls
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    # 一件も出力されない
    ```

1. 次いで Docker イメージをビルド｡
下記のコマンドでイメージ名｢ hello-world ｣の Docker イメージがビルドされる｡

    ```bash:Dockerイメージのビルド
    $ docker build -t hello-world .
    .... # 以下､実行結果が出力
    ```
     * ｢ -t ｣を指定することで名前がつけられる
     * オプションについては [docker build#options](https://docs.docker.com/engine/reference/commandline/build/#options) を参照


1. ビルド完了後､もう一度 Docker イメージのリストを確認する｡

    ```bash:Dockerイメージのリストを確認する
    $ docker image ls
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    hello-world         latest              434c71a1ad35        3 minutes ago       747MB
    python              3.6                 a91b1909dbdd        3 weeks ago         736MB
    ```

    指定したイメージ名｢ hello-world ｣でイメージが作成されていることが確認できる｡
    また､ここで｢ python ｣は Docker ファイルの｢ FROM ｣で指定したベースイメージである｡


### Docker コンテナの起動

コンテナの起動は次のコマンドで行う｡

```bash:コンテナの起動
$ docker run -d -p 4000:80 hello-world
```

```run``` コマンドのオプションについて補足すると､

* ｢ -d ｣を指定するとコンテナは **バックグラウンドで起動** する
* ｢ -p ｣を指定することで **コンテナのポートを公開** する
* オプションについては [docker run#options](https://docs.docker.com/engine/reference/commandline/run/#options) を参照

意味となる｡
今回は ｢ 4000:80 ｣と指定しているので､｢ ホストの 4000 番ポート ｣と｢ コンテナの 80 番ポート ｣を紐づけいている｡

これで ｢ ホストの4000 番ポート ｣にアクセスすることで､ ｢ Docker コンテナの 80 番ポート ｣にアクセス可能な Webサービスを持つコンテナが起動したので､次の項目で実際にアクセスして確認してみる｡



### Web ページの確認

* ローカルホストからコマンドで確認

    次のコマンドで Web アプリにアクセスして結果が表示されることを確認する｡

    ```bash:curlコマンドでWebアプリへのアクセスを確認する
    $ curl http://localhost:4000
    ```

    ```html:WebページのHTMLがレスポンスで返却されている
    <h3>Hello World!</h3>
    ```

* Web ブラウザから確認

    Docker コンテナが起動しているホストの IP アドレスを指定してブラウザからアクセス｡
![docker-container-verify.png](https://qiita-image-store.s3.amazonaws.com/0/193342/fb68e644-806c-4487-8319-4b7b6b4bd761.png)


    ```curl``` コマンドを実行したときと同じ結果がブラウザで表示されることが確認できた｡


## まとめ

* Dockerfile を作成して Docker イメージの作成を行った
* コンテナ起動時に実行される Web アプリケーションを作成した

とりあえずこれで Dockerfile による開発環境の構築が行えるようになった｡
あとは docker-compose にも手を出して､開発環境の構築手順の単純化や共有化に持っていけるようにしたい｡


## 参考

* [Get Started, Part 1: Orientation and setup](https://docs.docker.com/get-started/)
* [Get Started, Part 2: Containers](https://docs.docker.com/get-started/part2/)
* [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
* [docker build](https://docs.docker.com/engine/reference/commandline/build/)
* [docker run](https://docs.docker.com/engine/reference/commandline/run/)
