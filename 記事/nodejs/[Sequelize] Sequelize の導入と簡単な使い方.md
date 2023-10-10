# はじめに
Node.js 上で ORM を提供するライブラリとして Sequelize がある。
本記事では

* PostgreSQL を使用
* DB アクセスはモデルを使用 ( 生のSQLは使用しない )
* 単一テーブルに対する単純な CRUD を処理を実装

を行うことで、 Sequelize の導入と簡単な使い方をまとめる。
テーブルの JOIN やトランザクションについては本記事では扱わない。

本家サイトは [こちら](http://docs.sequelizejs.com/)。


# 更新情報
## 2021/11/06
- 記事内で扱ったコードを Sequlize `v6.9.0` で確認しました

## 2021/01/20
- 記事内で扱ったコードを Express `v4.17.1`, Sequlize `v6.4.0` で確認しました
- 作業環境を記載しました

# 作業環境

| 環境                                       | バージョン          | 備考                    |
| :----------------------------------------- | :------------------ | :---------------------- |
| [Node.js](https://nodejs.org/ja/)          | v12.18.3            | `$ node --version`      |
| [npm](https://www.npmjs.com/)              | v6.14.6             | `$ npm --version`       |
| [express](https://expressjs.com/)          | ~~v4.14.1~~ v4.17.1 | package.json の記載から |
| [Sequelize](https://sequelize.org/master/) | ~~4.33.3~~ ~~v6.4.0~~ v6.9.0   | 同上                    |


# Sequelize とは

[本家サイト](http://docs.sequelizejs.com/) から抜粋。

> Sequelize is a promise-based ORM for Node.js v4 and up. It supports the dialects PostgreSQL, MySQL, SQLite and MSSQL and features solid transaction support, relations, read replication and more.
>
> (訳: Node.js v4 以降で動く Promise ベースの ORM。PostgreSQL, MySQL, SQLite and MSSQL をサポートし、トランザクションやリレーションシップ、複製機能を有する。) 

# 前提

* PostgreSQL がインストールされていること


# 構成

本記事で扱うディレクトリ構成は次のとおり。
```app/db``` や ```app/model``` 配下に Sequelize を利用したコードを作成していく。

```bash:ディレクトリ構成
express-work                                  # プロジェクトルート
    ├── app/
    │   └── db/                               # Sequlize 利用のためのプログラムを配置
    │     └── model/                          # モデル を配置
    ├── bin/
    │    └── www                              # プログラムの起点
    ├── postman/
    │     └── Sequelize-CRUD.postman_collection.json # CRUD 処理の確認用 Postman データ
    ├── routes/
    │     └── index.js                        # フロントエンドからのアクセスの入り口
    └── app.js                                # 各種ライブラリを使用する設定を記載
```

# インストール

[本家の　Getting started](http://docs.sequelizejs.com/manual/installation/getting-started) のままに進める。

```bash:sequelizeのインストール
$ npm install --save sequelize
$ npm install --save pg pg-hstore
```

これで Sequelize 本体と PostgreSQL を扱うためのライブラリがインストールされた。


# DB と TABLE の作成

本記事で扱う DB: ```company``` と TABLE: ```employee``` を次のスクリプトで作成する。

```sql:initial-db.sql
/**
 * USAGE:
 * psql -f initial-db.sql -Upostgres
 */
drop database if exists company;
create database company OWNER=postgres;

\connect company;

drop table if exists employee;
create table if not exists employee (
	id serial,
    name text not null unique,
    tel text,
    primary key(id)
);

insert into employee(name, tel) values('hogehoge', '000-0000-0000');
insert into employee(name, tel) values('piyopiyo', '111-1111-1111');
insert into employee(name, tel) values('fugafuga', '222-2222-2222');
```

使い方はスクリプト中の USAGE にあるとおり、次のコマンドを実行する。

```bash:スクリプトの実行
$ psql -f create-table.sql -Upostgres 
```

# 接続設定

接続設定は ```app/db/``` 配下に ```db-config.js``` として定義する。

```javascript:db-config.js
const Sequelize = require('sequelize');

/**
 * company に対する接続設定を定義
 */
const dbConfig = new Sequelize('company', 'postgres', 'pgadmin', {
  // 接続先ホストを指定
  host: 'localhost',

  // 使用する DB 製品を指定
  dialect: 'postgres',

  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000
  }
});

module.exports = dbConfig;
```

Sequelize のインスタンスを生成する際にコンストラクタ引数として

* DB
* ユーザ名
* パスワード
* オプション

を指定する。詳しくは [こちら](http://docs.sequelizejs.com/class/lib/sequelize.js~Sequelize.html#instance-constructor-constructor) を参照。


# モデルの作成
## employee モデル
TABLE: ```employee``` に対するモデルを作成する。
モデルの配置先は ```app/model/``` とする。

```javascript:employee.js
const Sequelize = require('sequelize');
const dbConfig = require('../db/db-config');

/**
 * employee テーブルの Entity モデル
 */
const employee = dbConfig.define('employee', {
  id: {
    type: Sequelize.INTEGER,
    primaryKey: true,
    autoIncrement: true 
  },
  name: {
    type: Sequelize.STRING
  },
  tel: {
    type: Sequelize.STRING
  },
}, {
  // タイムスタンプの属性 (updatedAt, createdAt) が不要ならば次のプロパティは false
  timestamps: false,

  // テーブル名を変更したくない場合は次のプロパティを true
  // デフォルトでは sequelize はテーブル名を複数形に変更する
  freezeTableName: true
});

module.exports = employee;
```

# CRUD 処理の実装

モデルを作成したので、そのモデルを使用した CRUD 処理を実装していく。
CRUD 処理は ```app/db``` 配下に ```db-client.js``` として作成する。

## 共通処理

まずは CRUD 処理のなかで使用する共通処理について示す。

```javascript:db-client.js
// Sequelize を使用して CRUD を実装するために必要な import 群
const Sequelize = require('sequelize');        // Sequelize 本体
const dbConfig = require('./db-config');       // 接続設定
const employee = require('../model/employee'); // モデル

/**
 * フロントエンドに返却するクエリ実行結果
 */
var result = {
  status: null,
  record: null,
  message: ""
};

/**
 * クエリ実行結果を初期化する
 */
var initializeResult = function initializeResult() {
  result.status = null,
  result.record = null,
  result.message = ""
};

/**
 * クエリ実行結果をセットする
 * @param {*} status 
 * @param {*} record 
 * @param {*} message 
 */
var setResult = function setResult(status, record, message) {
  initializeResult();
  result.status = status;
  if (record) {
    result.record = record;
  } else {
    result.message = message;
  }

  return result;
};

/** 
 * コンストラクタ
 */
var DbClient = function() {
  // db access
  dbConfig
  .authenticate()
  .then(() => {
    console.log('Connection has been established successfully.');
  })
  .catch((err) => {
    console.error('Unable to connect to the database:', err);
  });
}

// 中略

module.exports = new DbClient();
```

細かいところはコードを見ていただくとして、以下、コードの記述順と前後するが大まかな説明を記す。

* コンストラクタ
    コンストラクタでは初期処理として **```dbConfig.authenticate()```** を実行して DB への認証を行ない、**```module.exports```** することで DB へのアクセス認証が通ったインスタンスを公開している。

* initializeResult
    フロントエンドに返却する処理結果を初期化する。

* setResult
    フロントエンドに返却する処理結果をセットする。
    その際、まず ```initializeResult()``` を実行し、過去の処理結果をクリアする。


## CREATE ( INSERT )

```javascript:db-client.js(CREATE処理を抜粋)
/**
 * レコード登録
 * @param {*} param 
 * @param {*} callback 
 */
DbClient.prototype.register = function register(param, callback) {
  employee.create(param)
  .then((record) => {
    callback(setResult(200, record, null));
  })
  .catch((err) => {
    callback(setResult(500, null, err));
  });
};
```

* **```create()```** メソッドを実行することでレコード登録を行う
  * その際フロントエンドから渡された登録データ ( param ) を ```create()``` メソッドの引数にセットする
* 成功すれば ```then()``` に、失敗すれば ```catch()``` に入る
* どちらのブロックに入っても ```setResult()``` で処理結果をセットし、```callback()``` を実行することで処理結果をフロントエンドに返却する


## READ ( SELECT )

```javascript:db-client.js(READ処理を抜粋)
/**
 * レコード全件取得
 * @param {*} callback 
 */
var findAll = function findAll(callback) {
  employee.findAll()
  .then((record) => {
    callback(setResult(200, record, null));
  })
  .catch((err) => {
    callback(setResult(500, null, err));
  });
};

/**
 * id に紐付くレコードを一件取得
 * @param {*} id 
 * @param {*} callback 
 */
var findById = function findById(id, callback) {
  //employee.findById(id)
  // Sequelize は v5 以降、`findById` から `findByPk` に移行した
  // https://github.com/the-road-to-graphql/the-road-to-graphql/issues/27
  employee.findByPk(id)
  .then((record) => {
    if (record) {
      callback(setResult(200, record, null));
    } else {
      callback(setResult(404, null, null));
    }
  })
  .catch((err) => {
    callback(setResult(500, null, err));
  });
};

/**
 * レコード取得
 * @param {*} query 
 * @param {*} callback 
 */
DbClient.prototype.find = function find(query, callback) {
  if (query.id) {
    findById(query.id, callback);
  } else {
    findAll(callback);
  }
};
```

* 全件取得時は **```findAll()```**、 id 指定による条件指定がある場合は ~~**```findById()```**~~ **```findByPk()```**メソッドを実行することでレコード取得を行う( 注[^1] )
  * id 指定時は query に id プロパティがセットされてくる
* 成功すれば ```then()``` に、失敗すれば ```catch()``` に入る
* どちらのブロックに入っても ```setResult()``` で処理結果をセットし、```callback()``` を実行することで処理結果をフロントエンドに返却する

[^1]: コード中のコメントに記載のとおり、Sequelize `v5` 以降、`findById` は `findByPk` に置き換わりました。


## UPDATE ( UPDATE )

```javascript:db-client.js(UPDATE処理を抜粋)
/**
 * レコード更新
 * @param {*} param 
 * @param {*} query 
 * @param {*} callback 
 */
DbClient.prototype.update = function update(param, query, callback) {
  const filter = {
    where: {
        id: query.id
    }
  };

  employee.update(param, filter)
  .then((record) => {
    callback(setResult(200, record, null));
  })
  .catch((err) => {
    callback(setResult(500, null, err));
  });
};
```

* **```update()```** メソッドを実行することでレコード更新を行う
  * その際、更新対象のレコードを ```update()``` の第一引数に、フィルタする条件を第二引数にセットする
  * この時フィルタ条件は ```where``` オブジェクトとして指定する
    * 今回フィルタ条件として指定した id は query にプロパティとしてセットされてくる 
* 成功すれば ```then()``` に、失敗すれば ```catch()``` に入る
* どちらのブロックに入っても ```setResult()``` で処理結果をセットし、```callback()``` を実行することで処理結果をフロントエンドに返却する


## DELETE ( DELETE )

```javascript:db-client.js(DELETE処理を抜粋)
/**
 * レコード削除
 * @param {*} query 
 * @param {*} callback 
 */
DbClient.prototype.remove = function remove(query, callback) {
  const filter = {
    where: {
        id: query.id
    }
  };

  employee.destroy(filter)
  .then((record) => {
    callback(setResult(200, record, null));
  })
  .catch((err) => {
    callback(setResult(500, null, err));
  });
};
```

* **```destroy()```** メソッドを実行することでレコード更新を行う
  * その際、削除対象のレコードをフィルタする条件を ```destroy()``` の第一引数にセットする
  * この時フィルタ条件は ```where``` オブジェクトとして指定する
    * 今回フィルタ条件として指定した id は query にプロパティとしてセットされてくる 
* 成功すれば ```then()``` に、失敗すれば ```catch()``` に入る
* どちらのブロックに入っても ```setResult()``` で処理結果をセットし、```callback()``` を実行することで処理結果をフロントエンドに返却する


# REST-API としてのインターフェース

上記で実装した CRUD 処理は RET-API として公開する。
そのためのインターフェースを ```index.js``` に実装する。

```javascript:index.js
var express = require('express');
var router = express.Router();
var dbClient = require('../app/db/db-client');

/**
 * HTTP の GET メソッドを待ち受けて employee テーブルからレコードを全件取得して返す
 */
router.get('/employee/find', function(req, res, next) {
  const query = req.query;
  dbClient.find(query, function(result) {
    res.json(result);
  });
});

/**
 * HTTP の POST メソッドを待ち受けて employee 情報を登録する
 */
router.post('/employee/register', function(req, res, next) {
  const addData = req.body;
  dbClient.register(addData, function(result) {
    res.json(result);
  });
});

/**
 * HTTP の PUT メソッドを待ち受けて employee 情報を更新する
 */
router.put('/employee/update', function(req, res, next) {
  const query = {
    id: req.body.id
  };
  const addData = req.body;
  dbClient.update(addData, query, function(result) {
    res.json(result);
  });
});

/**
 * HTTP の DELETE メソッドを待ち受けて employee 情報を削除する
 */
router.delete('/employee/remove', function(req, res, next) {
  const query = {
    id: req.body.id
  };
  dbClient.remove(query, function(result) {
    res.json(result);
  });
});

module.exports = router;
```

フロントエンドから実行された HTTP メソッドを受診するだけの単純なコードなので、特に説明する点はなし。

# 動作確認

動作確認は [Postman](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=ja) から行う。
動作確認に使用した Postman のデータは [こちら](https://github.com/ksh-fthr/express-work/blob/feat_sequelize/postman/Sequelize-CRUD.postman_collection.json) 。


# 参考
- [Model | Sequelize](https://sequelize.org/master/class/lib/model.js~Model.html#static-method-findByPk)
- [sequelize “findbyid” is not a function but apparently “findAll” is](https://stackoverflow.com/questions/41577597/sequelize-findbyid-is-not-a-function-but-apparently-findall-is)
- [ Sequelize v5, findById() was replaced by findByPk(). #27 ](https://github.com/the-road-to-graphql/the-road-to-graphql/issues/27)


# ソースコード

今回の記事で作成したコードは [こちら](https://github.com/ksh-fthr/express-work/tree/feat_sequelize) にアップしてあるのでご参考まで。

また本記事で扱ったコード、DB を Docker 環境としてまとめました｡
ご興味あれば [こちら](https://github.com/ksh-fthr/backend-in-docker) もご参照ください。
