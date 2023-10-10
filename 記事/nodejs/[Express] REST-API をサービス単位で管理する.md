複数のサービスで共通のバックエンドを使用するという前提で､サービス単位で REST-API を管理したい､というときのメモ｡
ただしバックエンドは複数のサービスとそれらが提供する REST-API を管理できるというだけで､実際に起動するときは単一のサービスのみ提供する｡

つまり次のような感じ｡

## 構成例
* バックエンドは hogehoge, piyopiyo というサービスをサポートする
* ただしバックエンドは実際には単一のサービスのみを提供する
* つまり
 * hogehoge というサービスを提供する場合は piyopiyo に対する機能は提供しない
 * piyopiyo というサービスを提供する場合は hogehoge に対する機能は提供しない

## この記事を実施した環境
* Windows10 Home 64bit
* node v8.2.1
* npm v4.0.5
* express v4.15.0

## 前提
Express のジェネレータで雛形を生成している｡

## バックエンドの実装
雛形生成時は routes/index.js に REST-API が直接作られている｡

* 雛形生成時の REST-API は index.js で管理

```javascript:雛形生成時のindex.js
var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('index', { title: 'Express' });
});

module.exports = router;
```

この状態でサービスごとに REST-API を作っていくと index.js がどんどん肥大化するし､本来提供しなくて良い(提供してはならない) API まで提供してしまうことになる｡
これを避けるためにサービス単位で REST-API を生成するようにする｡
構成としては

 1. 提供するサービスをファイルで管理(service.conf.js)
 1. サービス単位に REST-API を提供するモジュールを作る
 1. REST-API を提供するモジュールを管理する Factory を作る
 1. index.js で service.conf.js と Factory を require する

という感じで､実際のコードは次｡

* サービス単位で REST-API を管理

```javascript:service.conf.js
// *******************************************
// 提供するサービスはここで管理
// *******************************************

// サービスに応じて hogehoge を変更する
module.exports = {
  'service': 'hogehoge'
};
```

```javascript:hogehoge.js
// *******************************************
// hogehoge サービス
// *******************************************

/**
 * REST-API を作る
 *
 * @param {oject} router 'express.Router()' で生成されるオブジェクト｡コール元で生成されたもの｡
 */
function createApi(router) {

  /* GET home page. */
  router.get('/', function(req, res, next) {
    res.render('index', { title: 'Express-Hogehoge' });
  });
}

/**
 * コンストラクタ
 */
const Hogehoge = function Hogehoge() {}

// prototype 継承に突っ込む
Hogehoge.prototype.createApi = createApi;

module.exports = new Hogehoge();
```

```javascript:piyopiyo.js
// *******************************************
// piyopiyo サービス
// *******************************************

/**
 * REST-API を作る
 *
 * @param {oject} router 'express.Router()' で生成されるオブジェクト｡コール元で生成されたもの｡
 */
function createApi(router) {

  /* GET home page. */
  router.get('/', function(req, res, next) {
    res.render('index', { title: 'Express-Piyopiyo' });
  });
}

/**
 * コンストラクタ
 */
const Piyopiyo = function Piyopiyo() {}

// prototype 継承に突っ込む
Piyopiyo.prototype.createApi = createApi;

module.exports = new Piyopiyo();
```

```javascript:api-factory.js
// *******************************************
// サービスごとのモジュールを管理する Factory
// *******************************************

const hogehoge = require('./hogehoge');
const piyopiyo = require('./piyopiyo');

// サービス管理テーブル
const serviceTable = {
  'hogehoge': hogehoge,
  'piyopiyo': piyopiyo
};

/**
 * REST-API を作る
 *
 * @param {oject} router 'express.Router()' で生成されるオブジェクト｡コール元で生成されたもの｡
 * @param {string} service サービス名
 */
function createApi(router, service) {
  const target = serviceTable[service];
  target.createApi(router);
}

/**
 * コンストラクタ
 */
const ApiFactory = function ApiFactory() {}

// prototype 継承に突っ込む
ApiFactory.prototype.createApi = createApi;

module.exports = new ApiFactory();
```

```javascript:index.js
const express = require('express');
const router = express.Router();
const serviceConf = require('../service.conf');
const factory = require('../app/api-factory');

// *******************************************
// Factory で REST-API を生成する
// *******************************************
factory.createApi(router, serviceConf.service);

module.exports = router;
```

で､これを起動させると､､､

* srvice.conf.js に hogehoge を記載
![rest-api-hogehoge.png](https://qiita-image-store.s3.amazonaws.com/0/193342/e5ccadb1-6eef-0704-4b7e-48fcaf20933c.png)

* srvice.conf.js に piyopiyo を記載
![rest-api-piyopiyo.png](https://qiita-image-store.s3.amazonaws.com/0/193342/e131fc8d-f4cd-512e-1516-3ef3f1c768ec.png)

と言った感じで､同じURLを叩いても違うREST-APIの結果が返ってきている｡

## 構成
この記事で載せたプロジェクトの構成｡
![structure.png](https://qiita-image-store.s3.amazonaws.com/0/193342/96836ed1-7db0-f860-5d78-458c9ff38a9d.png)


## 課題
どうにも疎結合になっていないような気がする｡
この構成だと api-factory.js は service.conf.js で設定されているサービス名を知っている必要があって､そこが気になるところ｡
なんとか service.conf.js で設定されるサービス名を知らなくともサービス名を解決する手段がないものか｡｡｡

いいアイディアが浮かんだら追記します｡
