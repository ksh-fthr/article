# はじめに

Go 言語を使った DB 処理に ORM として GORM/Gen を入れてみたので、その作業時に備忘録として記事にしました。
これから GORM/Gen を使った開発環境を構築する方の一助になれば幸いです。

なお [GORM](https://gorm.io) は Go 言語 における ORM。 [Gen](https://github.com/go-gorm/gen) は GORM をサポートしたコードジェネレーターです。

# 環境と各ツール

|                                                                | バージョン | 備考                                                 |
| -------------------------------------------------------------- | --------- | ---------------------------------------------------- |
| Windows11 Home                                                 | 22H2      |                                                      |
| WSL                                                            | v2.0.9.0  | `wsl --version` で確認                               |
| [Rancher Desktop](https://rancherdesktop.io/)                  | v1.11.1   | メニュー > Help > About Rancher Desktop で確認 |
| [MySQL](https://www.mysql.com/jp/)                             | v8.3.0    | mysql に接続時の出力メッセージで確認                   |
| [Go](https://go.dev/)                                          | v1.22.1   | `go version` で確認                                  |
| [GORM](https://gorm.io)                                        | v1.25.9   | go.mod で確認                                        |
| [Gen](https://github.com/go-gorm/gen)                          | v0.3.26   | go.mod で確認                                        |

# 導入

## GORM のインストール

[公式サイトのガイド](https://gorm.io/docs/)  に従いインストール。

GORM のインストール。

```bash
% go get -u gorm.io/gorm
```

MySQL 用ドライバのインストール。

```bash
% go get -u gorm.io/driver/mysql
```

## Gen のインストール

[公式サイトのガイド](https://gorm.io/gen/index.html)  に従いインストール。

Gen のインストール。

```bash
% go get -u gorm.io/gen
```

# query と model のコードを自動生成する

## ジェネレータの作成

[Gen の QuickStart](https://gorm.io/gen/index.html#Quick-start) に従いコード生成処理 (以降、ジェネレータ) を作成します。
この処理を実行することによって `model` と `query` のコードが自動生成されます。

なお本記事の処理対象である `Contents` と `Articles` は親子関係にあるので、ジェネレータにはこれらのリレーションを定義する設定を盛り込みます。

```go
package main

import (
    // ご自身の環境にあわせてドライバを変更してください
    "gorm.io/driver/mysql"
    "gorm.io/gen"
    "gorm.io/gen/field"
    "gorm.io/gorm"
)

func main() {
    // コード生成ジェネレータの設定です
    // 各パラメータはこちらをご参照ください
    // https://gorm.io/gen/dao.html#gen-Config
    g := gen.NewGenerator(gen.Config{
        OutPath: "./gen/query",  // 出力先
        // 各モードのリファレンスはこちらです
        // https://gorm.io/gen/dao.html#Generator-Modes
        Mode: gen.WithoutContext | gen.WithDefaultQuery | gen.WithQueryInterface,
    })

    // ご自身の環境に応じて Open の引数は変更してください
    dbconf := "mysql:mysqladmin@tcp(172.30.10.100:3306)/mydb?charset=utf8mb4"
    gormdb, _ := gorm.Open(mysql.Open(dbconf))
    g.UseDB(gormdb)

    // リレーションを張りたいテーブルを指定し親子関係を設定します
    // Contents はArticles テーブルを子テーブルに持ちます
    article := g.GenerateModel("Articles")

    // Artilcles はContents テーブルを親テーブルに持ちます
    content := g.GenerateModel("Contents",
        gen.FieldRelate(field.HasMany, "Articles", article, &field.RelateConfig {
            GORMTag: field.GormTag{
                "foreignKey": []string{"ContentID"},
                "references": []string{"ID"},
            },
        }),
    )

    // コードを生成する対象のモデルを設定して...
    g.ApplyBasic(article, content)

    // コード生成を実行します
    g.Execute()
}
```

コードコメントにも記載しましたが、ジェネレータの各設定については下記をご参照ください。

- [DAO Overview > gen-Config](https://gorm.io/gen/dao.html#gen-Config)


またコード中に

```go
    // ご自身の環境に応じて Open の引数は変更してください
    dbconf := "mysql:mysqladmin@tcp(172.30.10.100:3306)/mydb?charset=utf8mb4"
    gormdb, _ := gorm.Open(mysql.Open(dbconf))
```

とありますとおり、上記はあくまで本記事で扱った環境を対象にした設定と処理です。
お試しになる際はご自身の環境に合わせた修正を行ってください。

## ジェネレータ実行の前準備( docker 起動 )

`model` と `query` のジェネレータを作りましたのでこれを実行します。
が、その前にやることがあります。

[今回題材に扱った環境](https://github.com/ksh-fthr/react-and-echo-work) では docker 上で DB を起動していますので、ジェネレータ実行のためにまず docker を起動します。

```bash
# docker 起動. -d オプションで実行するのでプロセスはバックグラウンド起動します
% docker-compose -f docker-compose-backend.yml up -d
[+] Building 0.0s (0/0)                                                                                                                docker:default
[+] Running 3/3
 ✔ Network react-and-echo-work_backend-work-net  Created                                                                                         0.0s
 ✔ Container restapi                             Started                                                                                         0.0s
 ✔ Container storage                             Started                                                                                         0.0s
```

## ジェネレータ実行

docker コンテナが起動しましたら、ジェネレータのある path まで移動します。
( docker で起動した mysql に対するアクセスは docker コンテナに入らずとも行えるようにしてありますので、以降のコマンドも dokcer コンテナに入らずに実行できます )

```bash
# ジェネレータの配置 path に移動する
% cd ${repository_root}/restapi/api/orm # ${repository_root} はリポジトリルートです
% ls -l 
total 8
-rw-r--r-- 1 user user 1570 Apr 13 20:16 orm_generator.go
```

これでコンテナ上でジェネレータを実行する準備ができました。
次のコマンドで実行してコードを生成します。

```bash
% go run orm_generator.go
go: downloading gorm.io/driver/mysql v1.5.6
go: downloading gorm.io/gen v0.3.25
go: downloading gorm.io/gorm v1.25.9
go: downloading golang.org/x/tools v0.20.0
go: downloading gorm.io/datatypes v1.2.0
go: downloading gorm.io/hints v1.1.2
go: downloading gorm.io/plugin/dbresolver v1.5.1
go: downloading github.com/jinzhu/now v1.1.5
go: downloading github.com/jinzhu/inflection v1.0.0
go: downloading golang.org/x/sync v0.7.0
go: downloading golang.org/x/mod v0.17.0
2024/04/14 11:57:02 got 8 columns from table <Articles>
2024/04/14 11:57:02 got 8 columns from table <Contents>
2024/04/14 11:57:02 Start generating code.
2024/04/14 11:57:02 generate model file(table <Articles> -> {model.Article}): /api/orm/gen/model/articles.gen.go
2024/04/14 11:57:02 generate model file(table <Contents> -> {model.Content}): /api/orm/gen/model/contents.gen.go
2024/04/14 11:57:02 generate query file: /api/orm/gen/query/articles.gen.go
2024/04/14 11:57:02 generate query file: /api/orm/gen/query/contents.gen.go
2024/04/14 11:57:02 generate query file: /api/orm/gen/query/gen.go
2024/04/14 11:57:02 Generate code done.
```

## 生成されたファイル

コマンドは正常に実行されました。ファイルが出力されているかを確認します。

```bash
% pwd
${repository_root}/restapi/api/orm/gen # ${repository_root} はリポジトリルートです

# user は実際にはログインしているユーザIDになります
% ls -l
合計 8
drwxr-xr-x 2 user     user     4096  4月  9 22:18 model/
drwxr-xr-x 2 user     user     4096  4月  9 22:18 query/
```

ディレクトリはできていますね。では、次のコマンドでファイルを確認します。

```bash
% ls -lR
.:
合計 8
drwxr-xr-x 2 user     user     4096  4月  9 23:05 model/
drwxr-xr-x 2 user     user     4096  4月  9 23:05 query/

./model:
合計 8
-rw-r----- 1 user     user     1115  4月  9 23:05 articles.gen.go
-rw-r----- 1 user     user      985  4月  9 23:05 contents.gen.go

./query:
合計 28
-rw-r----- 1 user     user     11518  4月  9 23:05 articles.gen.go
-rw-r----- 1 user     user     11336  4月  9 23:05 contents.gen.go
-rw-r----- 1 user     user      2123  4月  9 23:05 gen.go
```

model, query にそれぞれのファイルが出力されていることが確認できました。

# モデルとテーブルを比較してみる

## DB 上のテーブル

さて、出力されたのは `Contents` テーブル と `Articles` テーブルに対するファイルです。
これらが実際のテーブルと合っているかをテーブル情報から確認します。

次のコマンドでコンテナに入ります。 `69ffd527734c` は [前準備](#前準備) で確認した mysql のコンテナID です。


```bash
% docker exec -it 69ffd527734c bash
bash-4.4# 
```

コンテナにはいったので mysql に接続します。

```bash
bash-4.4# mysql -umysql -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.3.0 MySQL Community Server - GPL

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

データベースを確認し、

```mysql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mydb               |
| performance_schema |
+--------------------+
3 rows in set (0.02 sec)
```

処理対象の DB である `mydb` に切り替えたら、

```bash
mysql> use mydb
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
```

テーブルの確認を行います。

```mysql
mysql> show tables;
+----------------+
| Tables_in_mydb |
+----------------+
| Articles       |
| Contents       |
+----------------+
2 rows in set (0.00 sec)
```

ジェネレータで作成されたファイルと同名のテーブルとして `Contents` テーブルと `Articles` テーブルの存在が確認できました。

## 生成されたモデルと実際のテーブルとの比較

テーブルの確認ができたので、生成されたモデルと実際のテーブルの構成を確認します。
( 本記事作成時の状況です。今後 GitHub 上のテーブル定義が更新されることで本記事と齟齬がでる可能性があります )

**Contents**

まずは親である `Contents` から見ていきます。

**モデル**

```go
// Code generated by gorm.io/gen. DO NOT EDIT.
// Code generated by gorm.io/gen. DO NOT EDIT.
// Code generated by gorm.io/gen. DO NOT EDIT.

package model

import (
	"time"
)

const TableNameContent = "Contents"

// Content mapped from table <Contents>
type Content struct {
    ID        int64     `gorm:"column:id;primaryKey;autoIncrement:true" json:"id"`
    Title     string    `gorm:"column:title;not null" json:"title"`
    Author    string    `gorm:"column:author" json:"author"`
    Summary   string    `gorm:"column:summary" json:"summary"`
    Deleted   bool      `gorm:"column:deleted;not null" json:"deleted"`
    CreatedAt time.Time `gorm:"column:created_at" json:"created_at"`
    UpdatedAt time.Time `gorm:"column:updated_at" json:"updated_at"`
    Articles  []Article `gorm:"foreignKey:ContentID;references:ID" json:"articles"`
}

// TableName Content's table name
func (*Content) TableName() string {
	return TableNameContent
}
```

**テーブル**

```mysql
mysql> desc Contents;
+------------+-----------------+------+-----+---------+----------------+
| Field      | Type            | Null | Key | Default | Extra          |
+------------+-----------------+------+-----+---------+----------------+
| id         | bigint unsigned | NO   | PRI | NULL    | auto_increment |
| title      | varchar(255)    | NO   |     | NULL    |                |
| author     | varchar(255)    | YES  |     | NULL    |                |
| summary    | text            | YES  |     | NULL    |                |
| deleted    | tinyint(1)      | NO   |     | NULL    |                |
| created_at | datetime        | YES  |     | NULL    |                |
| updated_at | datetime        | YES  |     | NULL    |                |
+------------+-----------------+------+-----+---------+----------------+
7 rows in set (0.00 sec)
```

子テーブルである `Articles` との関係性を示すものとして以下に注目です。

- 構造体のメンバとして `Articles` が定義されている 
- `Ariticles` は複数行持ち得るのでリストで表現されている
- 両者のリレーションを示すべく `Contents` テーブルの `ID` が **外部キー** として `Articles` の `ContentID` に設定されている


**Articles**

次に `Articles` を見ます。


**モデル**

```go
// Code generated by gorm.io/gen. DO NOT EDIT.
// Code generated by gorm.io/gen. DO NOT EDIT.
// Code generated by gorm.io/gen. DO NOT EDIT.

package model

import (
	"time"
)

const TableNameArticle = "Articles"

// Article mapped from table <Articles>
type Article struct {
    ID        int64     `gorm:"column:id;primaryKey;autoIncrement:true" json:"id"`
    ContentID int64     `gorm:"column:content_id;not null" json:"content_id"`
    Subtitle  string    `gorm:"column:subtitle;not null" json:"subtitle"`
    Body      string    `gorm:"column:body;not null" json:"body"`
    Remarks   string    `gorm:"column:remarks" json:"remarks"`
    Deleted   bool      `gorm:"column:deleted;not null" json:"deleted"`
    CreatedAt time.Time `gorm:"column:created_at" json:"created_at"`
    UpdatedAt time.Time `gorm:"column:updated_at" json:"updated_at"`
}

// TableName Article's table name
func (*Article) TableName() string {
	return TableNameArticle
}
```

**テーブル**

```mysql
mysql> desc Articles;
+------------+-----------------+------+-----+---------+----------------+
| Field      | Type            | Null | Key | Default | Extra          |
+------------+-----------------+------+-----+---------+----------------+
| id         | bigint unsigned | NO   | PRI | NULL    | auto_increment |
| content_id | bigint unsigned | NO   | MUL | NULL    |                |
| subtitle   | varchar(10245)  | NO   |     | NULL    |                |
| body       | text            | NO   |     | NULL    |                |
| remarks    | text            | YES  |     | NULL    |                |
| deleted    | tinyint(1)      | NO   |     | NULL    |                |
| created_at | datetime        | YES  |     | NULL    |                |
| updated_at | datetime        | YES  |     | NULL    |                |
+------------+-----------------+------+-----+---------+----------------+
8 rows in set (0.00 sec)
```

こちらは `Contents` と違って特にみるべき点はありません。
子テーブルとしての `Articles` が素直にモデルとして表現されています。

## 補足(boolean について)

本記事では RDBMS として mysql を使用していますが、mysql では `boolean` を `tinyint(1)` で表します。([MySQL 8.0 リファレンスマニュアル > 11.1.1 数値データ型の構文](https://dev.mysql.com/doc/refman/8.0/ja/numeric-type-syntax.html)) 
今回の例ではテーブル定義において `deleted` を `tinyint(1)` で定義していますが、生成されたモデルでは当該カラムがしっかりと `bool` 型で定義されています。

# まとめにかえて

本記事では GORM と Gen を使った ORM のモデル定義・クエリの出力まで行いました。
公式のリファレンス、また先達の方々の記事のおかげで躓きながらも自分の環境で実現できましたこと、感謝申し上げます。

次はこの記事で作成したモデル・クエリを利用した CRUD について扱ってみます。

# ソースコードについて

本記事であつかったコードは下記にあります。よろしければご参照ください。

- [ORM と ジェネレータ](https://github.com/ksh-fthr/react-and-echo-work/tree/main/restapi/api/orm)

# 参考

- 公式リファレンス
  - [GORM Guides](https://gorm.io/docs/)
  - [GORM Guides > Associations](https://gorm.io/gen/associations.html)
  - [Gen Guides](https://gorm.io/gen/)
  - [DAO Overview > gen-Config](https://gorm.io/gen/dao.html#gen-Config)
  - [DAO Overview > gen-Config > Generator-Modes](https://gorm.io/gen/dao.html#Generator-Modes)

- 公式以外
  - [GoのORM決定版 Genをはじめよう](https://qiita.com/muff1225/items/f660270694f29597df22)
  - [GoのORM「Gen」を使ってみた](https://zenn.dev/mizuko_dev/articles/dbeb7c93883b57)
  - [GormGenのFieldRelateを使用して　テーブルの関連を表した構造体を生成する](https://zenn.dev/rescuenow/articles/da1cb5f574fb0c)

- MySQL
  - [MySQL 8.0 リファレンスマニュアル > 11.1.1 数値データ型の構文](https://dev.mysql.com/doc/refman/8.0/ja/numeric-type-syntax.html)

- 書籍
  - [Go言語プログラミングエッセンス](https://gihyo.jp/book/2023/978-4-297-13419-8)
