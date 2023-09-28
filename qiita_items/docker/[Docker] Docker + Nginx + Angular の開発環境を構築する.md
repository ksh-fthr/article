# ~~作業環境~~
* ~~[Rasppberry Pi3](https://www.raspberrypi.org/)~~
    * ~~[Ubuntu MATE](https://ubuntu-mate.org/raspberry-pi/) 16.04.5 LTS ( GNU/Linux 4.4.38-v7+ armv7l )~~
    * ~~[Docker CE](https://docs.docker.com/install/linux/docker-ce/ubuntu/) ( 18.06.0~ce~3-0~ubuntu )~~
    * ~~[Docker Compose](https://docs.docker.com/compose/) v1.23.0dev~~
* ~~[Angular CLI](https://cli.angular.io/) v7.2.2~~
* ~~[Angular](https://angular.io/) v7.2.0~~
* ~~[Node.js](https://nodejs.org/ja/) v10.12.0~~
* ~~[npm](https://www.npmjs.com/) v6.4.1~~

## ~~補足~~
~~Raspberry Pi3 に加えて次の環境でも動作確認済み~~

* ~~Windows 10 Pro 64bit~~
    * ~~[Docker for Windows](https://docs.docker.com/docker-for-windows/)~~
* ~~Mac OSX Mojave( 10.14.x )~~
    * ~~[Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/)~~


# 作業環境
- 2020/12/27 作業環境を以下に更新しました
- 上記に伴い、これまで「作業環境」と「補足」で挙げていた内容は当該環境での動作未確認につき、取り消し線で修飾しました

| 環境                                                         | バージョン | 備考           |
| ------------------------------------------------------------ | ---------- | -------------- |
| macOS Catalina                                               | v10.15.7   |                |
| [Docker Desktop for Mac](https://hub.docker.com/editions/community/docker-ce-desktop-mac/) | v3.0.3     |                |
| Docker                                                       | v20.10.0, build 7287ab3   | `$ docker --version`  |
| Docker Compose                                               | v1.27.4, build 40524192    | `$ docker-compose --version` |
| [Angular CLI](https://cli.angular.io/)                       | v11.0.5    | `$ ng --version` |
| [Angular](https://angular.io/)                               | v11.0.5    | 同上 |
| [Node.js](https://nodejs.org/ja/)                            | v12.18.3   | `$ node --version` |
| [npm](https://www.npmjs.com/)                                | v6.14.6    | `$ npm --version` |

<details>
<div>
<summary>ng version の結果</summary>

```bash
$ ng version

     _                      _                 ____ _     ___
    / \   _ __   __ _ _   _| | __ _ _ __     / ___| |   |_ _|
   / △ \ | '_ \ / _` | | | | |/ _` | '__|   | |   | |    | |
  / ___ \| | | | (_| | |_| | | (_| | |      | |___| |___ | |
 /_/   \_\_| |_|\__, |\__,_|_|\__,_|_|       \____|_____|___|
                |___/
    

Angular CLI: 11.0.5
Node: 12.18.3
OS: darwin x64

Angular: 11.0.5
... animations, cli, common, compiler, compiler-cli, core, forms
... platform-browser, platform-browser-dynamic, router
Ivy Workspace: Yes

Package                         Version
---------------------------------------------------------
@angular-devkit/architect       0.1100.5
@angular-devkit/build-angular   0.1100.5
@angular-devkit/core            11.0.5
@angular-devkit/schematics      11.0.5
@schematics/angular             11.0.5
@schematics/update              0.1100.5
rxjs                            6.6.0
typescript                      4.0.2
```

</div>
</details>

# 0. 事前準備
まず次の構成でディレクトリを作成する。
これが本記事で行う作業の基本構成（　Docker 環境のベース　）となる。

```sh:ディレクトリ構成
angular-in-docker          # ルートディレクトリ
└ angular-app/             # Angular アプリや Dockerfile 等を管理するディレクトリ
```

# 1. Angular プロジェクトの作成
次に `angular-app` 配下で Angular プロジェクトを作成する。

```sh:Angularプロジェクトを作成〜配置する
$ cd angular-app/
$ ng new app
```

# 2. Docker 環境を構築する
今度は作成した Angular プロジェクトを Docker 環境で動かすための環境構築を行う。

## 2.1. Dockerfile の作成
Angular アプリをビルドし、ビルドした資産を Nginx 上にデプロイするための設定を行う。
この Dockerfile の基本的なコンセプトは以下のとおり。

1. Angular プロジェクトは Docker コンテナ生成時にビルドする
2. ビルドした資産を Nginx の資産にコピーする(デプロイする)
3. 本プロジェクト用の nginx.conf を用意し、そのファイルで Nginx の挙動を制御する
4. アプリは Nginx 上で稼働する

詳細は下記のコメントを参照。

```sh:angular-app/Dockerfile
# -----------------------------------------------------
# Angular アプリをビルドするための環境を構築する
# -----------------------------------------------------
# Angualr のビルドする環境として node をインストール
# 12.18.3 は開発で使用しているバージョン
FROM node:12.18.3 as build-stage

WORKDIR /app
COPY ./app/package*.json /app/

# npm install 中に下記 Warning がでるが回避策が現状見当たらないので放置するしかない
# 内容はプラットフォームが Mac OSX ではないからスキップするというもので、
# 対象となっている fsevents は node_modules 中のライブラリが依存関係で使用しているライブラリ

# Warning の内容
#   npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.6 (node_modules/fsevents):
#   npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.6: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"x64"})

RUN npm install
COPY ./app/ /app/
ARG configuration=production

# Angular アプリをビルドする
RUN npm run build -- --output-path=./dist/out --configuration $configuration

# -----------------------------------------------------
# Nginx の Docker 環境を構築する
# -----------------------------------------------------
FROM nginx:1.15

# ビルドした成果物を Docker 上の Nginx のドキュメントとして扱うためにコピー(デプロイ)
COPY --from=build-stage /app/dist/out/ /usr/share/nginx/html

# Nginx の設定ファイルを Docker 上の Nginx にコピー
COPY ./docker/nginx/nginx.conf /etc/nginx/nginx.conf
```

## 2.2. .dockerignore の作成
Docker イメージをビルドする際にコピー対象から除外するリソースを指定する。
今回のケースでは Angular プロジェクト上の `node_modules` を除外している。これは当該フォルダ配下にある依存ライブラリが大量にあるため。

```sh:angular-app/.dockerignore
./app/node_modules
```

## 2.2. nginx.conf の作成
Docker 上で起動する Nginx の設定ファイルとして `nginx.cong` を `angular/app/docker/nginx/` 配下に作成する。
このファイルに記載した内容で Nginx の挙動を左右できるので、今後もこのファイルは更新されていくことになる。

ここではあくまで初期設定としての内容を示す。

```sh:angular-app/docker/nginx/nginx.conf
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    server {
        listen 8080;
        server_name  localhost;

        root   /usr/share/nginx/html;
        index  index.html index.htm;
        include /etc/nginx/mime.types;

        gzip on;
        gzip_min_length 1000;
        gzip_proxied expired no-cache no-store private auth;
        gzip_types text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript;

        location / {
            try_files $uri $uri/ /index.html;
        }
    }
}
```

## 2.3. docker-compose.yaml の作成
Docker コンテナを Docker Compose 経由で管理するための設定ファイルを作成する。

```yaml:docker-compose.yaml
version: '3'
services:
  app:
    build: angular-app/
    container_name: "angular-app"
    ports:
      - "80:8080"
    environment:
      TZ: "Asia/Tokyo"
```

# 3. 最終的なプロジェクト構成
Angular プロジェクト、及び Docker の各設定ファイルを含めた構成は以下のとおり。
フロントエンド開発はこの構成で行っていく。

なお Git で管理する都合上、 [1. Angular プロジェクトの作成](#1-angular-プロジェクトの作成) のときに生成された `.editorconfig` や `.gitignore`、`README.md` 等はルートディレクトリの直下に移動させてある。

```sh:プロジェクト構成
angular-in-docker        # ルートディレクトリ
├ .editorconfig          # エディタの設定ファイル(ng new で勝手に生成される)
├ .gitignore             # Git 管理から除外する資産を管理する
├ LICENSE                # ライセンスファイル
├ README.md              # アプリケーションの README
├ angular-app/           # Angular アプリや Dockerfile 等を管理するディレクトリ
│ ├ .dockerignore        # Docker イメージ作成時にコピー対象から除外する資産を管理する
│ ├ Dockerfile           # Docker コンテナを管理する
│ ├ app/                 # Angular アプリ
│ │ ├ angular.json       # Angular プロジェクトの設定ファイル
│ │ ├ e2e/               # e2e テストの資産
│ │ ├ karma.conf.js      # JavaScript のテストランナー Karma の設定ファイル
│ │ ├ node_modules/      # npm install で導入されるライブラリ群
│ │ ├ package.json       # プロジェクトの依存関係を管理する. プロジェクト配下での npm install は初期状態ではこれを元にライブラリがインストールされる
│ │ ├ src/               # Angular プロジェクトの本体となるソース群
│ │ ├ tsconfig.app.json  # TypeScriptおよびAngularテンプレートコンパイラオプションを含む、アプリケーション固有の TypeScript の設定
│ │ ├ tsconfig.json      # TypeScript -> JavaScript へのトランスパイルの設定
│ │ ├ tsconfig.spec.json # アプリケーションテスト用の TypeScript の設定
│ │ └ tslint.json        # 静的解析のチェッカーファイル
│ └ docker/              # Docker 用 Nginx の設定ファイル
│   └ nginx/
│      └ nginx.conf
└ docker-compose.yaml    # 複数の Docker コンテナを管理する
```

Angular アプリの各設定ファイルについては公式の [Angular 日本語ドキュメント-ワークスペースとプロジェクトのファイル構造](https://angular.jp/guide/file-structure) を参照。

# 4. Docker コンテナの起動と確認
では作成した環境を実際に動かしてみる。

## 4.1. Docker Compose 経由で Docker コンテナを起動する
次のコマンドを実行し、Docker コンテナを起動する。

```
$ docker-compose up -d --build
```

`-d` はバックグラウンドで起動するオプション。
もし次の動作確認で画面が表示されない場合は `-d` を外してコマンドを実行すると、起動時の情報がコンソール上に出力されるので解決の糸口となる。

## 4.2. 動作確認
ブラウザより `http://<ホストのIPアドレス>` にアクセスする。以下の画面が表示されれば導入とコンテナの起動は成功している。
![スクリーンショット 2020-12-27 13.19.08.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/67992d83-327b-7fb6-b849-b31426b68e56.png)
※ 開発状況によって表示される内容は異なる

## 4.3. Docker Compose によるサービスの停止とコンテナの停止
Docker Compose によるサービスの停止とコンテナの停止については次のコマンドを実行する。
( 目的に応じて次のどちらかを実行する )

* サービスの停止

サービスを停止する。Docker コンテナは削除されない。

```sh:サービスの停止
$ docker-compose stop
```

* コンテナの停止

サービスの停止とサービスを提供するコンテナの削除、それからネットワークも削除する。 

```sh:コンテナの停止
$ docker-compose down
```

# 今回の記事で作成した環境

[こちら](https://github.com/ksh-fthr/angular-in-docker) にアップしているので､よろしければご参考まで｡


# 参考
* [Angular in Docker with Nginx](https://medium.com/@tiangolo/angular-in-docker-with-nginx-supporting-environments-built-with-multi-stage-docker-builds-bb9f1724e984)
* [Your Angular apps as Docker containers](https://medium.com/@DenysVuika/your-angular-apps-as-docker-containers-471f570a7f2)
* [Angular 日本語ドキュメント-ワークスペースとプロジェクトのファイル構造](https://angular.jp/guide/file-structure)
