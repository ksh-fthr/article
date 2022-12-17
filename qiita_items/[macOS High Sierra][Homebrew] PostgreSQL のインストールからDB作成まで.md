
## 環境

* macOS High Sierra ( 10.13.x )
* Ruby v2.3.4p301
* Homebrew v1.5.4
* PostgreSQL v10.2


## Homebrewのインストール

コマンドラインから次のコマンドを実行してインストール。

```
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```


## PostgreSQLのインストール

Homebrew で PostgreSQL をインストール。

```
$ brew install postgresql
```


## PostgreSQLのサービス自動起動と停止

### 自動起動

次に示すコマンドで PostgreSQL の自動起動を設定する。
これによって PostgreSQL が起動し、macOS の再起動〜ログインした際にも PostgreSQL が自動で起動する設定となる。

```
$ brew services start postgresql
```

### 停止

PostgreSQL の停止、並びに自動起動をやめたい場合は次のコマンドを実行する。

```
$ brew services stop postgresql
```


## ユーザ作成

```
$ psql -U${USER} postgres
```

で PostgreSQL にログイン。 
ここで、環境変数 ```${USER}``` は PostgreSQL をインストールした際の macOS のログインユーザである。```${USER}``` は PostgreSQL に対してスーパーユーザとして作成されていて、このユーザをそのまま使うのは良くないので、

```
postgres=# create user postgres with SUPERUSER;
CREATE ROLE
```

で OS のユーザとは別に操作用のユーザ ```postgres``` を作成する。

この時  ```with SUPERUSER```  を指定して、作成するユーザにスーパーユーザー権限を与えている点に注目。
ここで ```with SUPERUSER``` 付きで作成しないとDB作成権限等の特別な権限が付与されない。


## DB接続

先ほど作成したユーザでログインできるかを確認する。

```
$ psql -Upostgres
```

次の表示が出たらログイン成功。

```
psql (10.2)
Type "help" for help.

postgres=# 
```

さらに

```
postgres=# \l
                                    List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |     Access privileges     
-----------+----------+----------+-------------+-------------+---------------------------
 postgres  | ${USER}  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | ${USER}  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/"${USER}"            +
           |          |          |             |             | "${USER}"=CTc/"${USER}"
 template1 | ${USER}  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/"${USER}"            +
           |          |          |             |             | "${USER}"=CTc/"${USER}"
(3 rows)
```

psql メタコマンドである ```¥l``` を入力してデータベースのリストが取得できることを確認。


## DB作成

コマンドラインから作成する方法と PostgreSQL にログインして作成する方法を示す。

### コマンドラインから作成する方法

コマンドラインからオーナーに ```postgres``` を指定してデータベース ```hogehoge``` を作成。

```
$ createdb hogehoge -O postgres;
```

* -O [オーナー] or (--owner)
 * 作成するデータベースの所有者となるユーザを指定

### PostgreSQL にログインして作成する方法

PostgreSQL にログインしてオーナーに ```postgres``` を指定してデータベース ```hogehoge2``` を作成。

```
$ psql -Upostgres
psql (10.2)
Type "help" for help.

postgres=# create database hogehoge2 owner=postgres;
CREATE DATABASE
```

### データベースの一覧を確認

前掲の psql メタコマンド ```¥l``` でデータベースの一覧を確認すると

```
postgres=# \l
                                    List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |     Access privileges     
-----------+----------+----------+-------------+-------------+---------------------------
 hogehoge  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 hogehoge2 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres  | ${USER}  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | ${USER}  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/"${USER}"            +
           |          |          |             |             | "${USER}"=CTc/"${USER}"
 template1 | ${USER}  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/"${USER}"            +
           |          |          |             |             | "${USER}"=CTc/"${USER}"
(4 rows)
```

先程作成したデータベース ```hogehoge``` と ```hogehoge2``` がオーナー ```postgres``` で作成されていることが確認できる。


## まとめにかえて

PostgreSQL を macOS ( High Sierra ( 10.13.x ) ) にインストールしてユーザ作成 〜 DB 作成まで確認できた。
これで PostgreSQL の実行環境が取り敢えずは整ったので、あとはマニュアルに沿ってテーブル作成するなりしてアプリ作成に利用する。


## 参考

* [PostgreSQL macOS packages](https://www.postgresql.org/download/macosx/)
* [Homebrew](https://brew.sh/index_ja.html)
* [オープンソースデータベース標準教科書 -PostgreSQL- (Ver1.0.1）](http://oss-db.jp/ossdbtext/text.shtml)
