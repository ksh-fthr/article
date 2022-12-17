[axios](https://github.com/axios/axios) について､いくつか記事を書いておきながら導入についての記事を書いていなかったので､導入について記しつつ [こちらの記事](http://qiita.com/ksh-fthr/items/ba7c80252edad0e7c66c) で示した構成も作ってみた｡

## 更新情報
* 2018/05/15
    * [axios](https://github.com/axios/axios) へのリンクを更新しました( @ledsun 様、ご指摘ありがとうございました )
* 2018/07/02
    * ```axios``` のインスタンス生成時に指定している ```headers``` の内容を修正しました( @dyoshikawa 様、ご指摘ありがとうございました )

## 過去に書いた axios の記事
* [[axios] axios で UnitTest 時に adapter を利用する](http://qiita.com/ksh-fthr/items/9ed895ecb1127edb7c6f) 
* [[axios] 画像データのレスポンスを取得する際にハマった話](http://qiita.com/ksh-fthr/items/ba7c80252edad0e7c66c)

## この記事を実施した環境
* ~~Windows10 Home 64bit~~ macOS High Sierra( 10.13.4 )
* ~~nodist v0.8.8 ( in Windows10 )~~
* n v2.1.7 ( in macOS High Sierra )
* node ~~v8.2.1~~ v9.4.0
* npm ~~v4.0.5~~ v6.1.0
* Express ~~v4.15.0~~ v4.16.0
* axios ~~v0.16.2~~ v0.18.0

## 前提
フロントエンド､バックエンドは [Express](http://expressjs.com/ja/) で実装している｡

## axios とは
[本家](https://github.com/mzabriskie/axios) によると

> Promise based HTTP client for the browser and node.js

とある。
そのままな訳となるが、ブラウザや node.js で動く Promise ベースの HTTP クライアントである｡ REST-API を実行したいときなど､これを使うと実装が簡単にできる｡

## axios の導入
次のコマンドを実行してインストールする｡
一応 ```--save``` で package.json にも反映させておく｡

```
$ npm install axios --save
```

```json:package.json(抜粋)
{
  ...
  "dependencies": {
    "axios": "^0.16.2",
    ...
  },
  ...
}
```

## axios を使ってバックエンド間のやり取りを行う
構成は次の感じ｡

1. バックエンドA はフロントエンドからのリクエストを受け付ける
1. バックエンドA は｢1.｣を受けてバックエンドB へリクエストを投げる
1. バックエンドB は｢2.｣を受けてレスポンスをバックエンドA へ返す
1. バックエンドA は｢3.｣を受けてレスポンスをフロントエンドへ返す

シーケンス図にするとこんなイメージ。
![sequence.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/0af5d792-5436-ecd1-7372-d8aec5f4019e.png)


## バックエンドA の実装
* ここでは上記の｢1.｣｢2.｣｢4.｣を担当する
* バックエンドA はバックエンドB に対してリクエストを投げるので､ここが axios を利用する箇所となる
* コード中の `[]` で囲った番号は上述の番号と合わせてある

`GET` メソッドだけの例となるが以下に実装例を示す

```javascript:バックエンドAのindex.js
var express = require('express');
var router = express.Router();

// axios を require してインスタンスを生成する
const axiosBase = require('axios');
const axios = axiosBase.create({
  baseURL: 'http://localhost:4000', // バックエンドB のURL:port を指定する
  headers: {
    'Content-Type': 'application/json',
    'X-Requested-With': 'XMLHttpRequest'
  },
  responseType: 'json'  
});

// [1] フロントエンドからのリクエストを受け付けて
router.get('/', function(req, res, next) {

  // [2] バックエンドB に対してリクエストを投げる
  axios.get('/title')
  .then(function(response) {

    // [4] フロントエンドに対してレスポンスを返す
    res.render('index', response.data);
  })
  .catch(function(error) {
    console.log('ERROR!! occurred in Backend.')
  });
});

module.exports = router;
```

オプションを指定したい場合、 `GET` メソッドでは ```axios.get(url, option)``` として **第2引数** にセットする｡ 
`PUT` や `POST` メソッドの場合は **第3引数** にセットする｡

[本家のこちらあたり](https://github.com/mzabriskie/axios#instance-methods) が参考になるかと｡

## バックエンドB の実装
* ここでは上記の｢3.｣を担当する
* こちらは単純にリクエストに対してレスポンスを返すだけなので axios は登場しない
* シンプルに Express だけの実装となる
* コード中の `[]` で囲った番号は上述の番号と合わせてある

```バックエンドBのindex.js
var express = require('express');
var router = express.Router();

// [3] フロントエンドA からのリクエストを受けてレスポンスを返す
router.get('/title', function(req, res, next) {
  res.json({ title: 'Express. Responded by BackEnd-B.' });
});

module.exports = router;
```

* なお、バックエンドB ではポート番号を変える必要があるので、忘れずに次の部分を修正しておく

```javascript:bin/www
// 15行目
// デフォルトでは 3000 が指定されているが、そのままではバックエンドA とポート番号が重複するので変更する
// (順序が逆になるが) ここではバックエンドA で指定しているポート番号[4000]を設定する
var port = normalizePort(process.env.PORT || '4000');
```

## 動作確認

1. バックエンドA､バックエンドB をそれぞれ次のコマンドで起動させる

    ```:バックエンドAを起動する
    $ npm start
    ```

    ```:バックエンドBを起動する
    $ npm start
    ```

1. バックエンドA に対してブラウザからアクセスする｡

* バックエンドA の URL(http://localhost:3000) にアクセス
* バックエンドB で返しているレスポンスである ｢ Express. Responded  BackEnd-B. ｣ が表示されていることが確認できる

    ![Responded by BackEnd-B.png](https://qiita-image-store.s3.amazonaws.com/0/193342/386749ea-b17c-a103-8609-64e7a04a6918.png)


## 終わりに
axios を利用すると REST-API を使ってのやり取りが簡単に実装できた｡
で､蛇足ながら｡｡｡
使用する REST-API が少ない場合は上記例のように ```axios.get(url)``` のように個別に書くのもいいが､API が増えてくるとちょっと冗長である｡
というところで､ REST-API を実行する際の入り口として次のようなヤツを用意するのもアリかと思う｡

```javascript:rest-facade.js(REST-APIを実行する際の入り口)
// axios を require してインスタンスを生成する
const axiosBase = require('axios');
const axios = axiosBase.create({
  baseURL: 'http://localhost:4000', // バックエンドB のURL:port を指定する
  headers: {
    'Content-Type': 'application/json',
    'X-Requested-With': 'XMLHttpRequest'
  },
  responseType: 'json'  
});

/**
 * REST-API の GET メソッドを実行する
 *
 * @param {any} url REST-API のURL
 * @param {any} callback 呼び出し元のコールバック処理
 * @param {any} [option=null] axios のオプション
 */
function get(url, callback, option = null) {

  // axios を使って引数で指定された url に対してリクエストを投げる
  axios.get(url, option)
  .then(function(response) {

    // 返ってきたレスポンスはそのまま加工せずに callback で呼び出し元へ渡す
    callback(response);
  })
  .catch(function(error) {
    console.log('ERROR!! occurred in Backend.')
  });
}

/**
 * コンストラクタ
 */
function RestFacade() {}

// prototype チェインに突っ込む
RestFacade.prototype.get = get;

module.exports = new RestFacade();
```

```javascript:rest-facade.jsを使う側
const express = require('express');
const router = express.Router();
const restFacade= require('./rest-facade');

router.get('/', function(req, res, next) {
  restFacade.get('/title', function(response) {
    res.render('index', { title: response.data.title });
  });
});
```

## 補足
### プロキシ関連の設定( Error: getaddrinfo ENOTFOUND の対処 )

プロキシを使用する環境下で、axios で REST-API を実行した際に次のエラーが発生するケースがある。

```:エラー情報
Error: getaddrinfo ENOTFOUND proxy.host.address(1) proxy.host.address(2):NNNN(3)

# (1), (2) の proxy.host.address は自身の環境におけるプロキシのアドレス
# (3) の NNNN はポート番号
```

で、axios 経由で REST-API を実行するときにプロキシを経由したくない場合、次の設定を前述のインスタンス生成時にセットしてやれば良い。

```javascript:バックエンドAのindex.jsからaxiosをrequireしてインスタンスを生成する箇所を抜粋
const axiosBase = require('axios');
const axios = axiosBase.create({
  baseURL: 'http://localhost:4000', // バックエンドB のURL:port を指定する
  headers: {
    'Content-Type': 'application/json',
    'X-Requested-With': 'XMLHttpRequest'
  },

  // -------------------------
  // プロキシを無視する設定
  // -------------------------
  proxy: false,

  responseType: 'json'  
});
```

プロキシ周りの設定については [本家の Request Config](https://github.com/axios/axios#request-config) にプロキシについての記述があるので参考にどうぞ。
