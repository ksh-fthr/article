## この記事を実施した環境
* Windows10 Home 64bit
* node v8.2.1
* npm v4.0.5
* express v4.15.0
* jasmine-node v1.14.5
* Visual Studio Code v1.15.0

## 前提
jasmine-node でユニットテストを書いていること｡
なお jasmine-node は次のコマンドでインストールする｡

```
npm install jasmine-node --save-dev
```

上記実行後の package.json には jasmine-node のエントリが追加されているので確認｡

```javascript:package.jsonから抜粋
  "devDependencies": {
    "jasmine-node": "^1.14.5"
  }
```

## launch.json に次の構成を追加する
jasmine-node をインストールしてユニットテストを記述する環境が整ったので､あとはユニットテストを書くだけ｡
で､ Visual Studio Code でユニットテストのデバッグを行いたい場合､次の構成を launch.json に追加する｡

```javascript:launch.json(ユニットテストをデバッグするための構成)
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug for Unit Test",
      // jasmne-node の cli を指定する
      "program": "${workspaceRoot}/node_modules/jasmine-node/lib/jasmine-node/cli.js",
      "args": [
        // テスト対象のフォルダを指定する
        "test/",
        "--color"
      ],
      "cwd": "${workspaceRoot}"
    }
  ]
}
```
## ユニットテストのデバッグ
ユニットテストの対象は[このエントリ](http://qiita.com/ksh-fthr/items/6351e95922adbff22937) で記載した api-factory.js の createApi｡

* テスト対象

```javascript:api-factory.jsから抜粋
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
```

* テストコード  
とりあえずメソッド呼び出しを検証するだけの単純なもの｡

```javascript:api-factory-spec.js
const express = require('express');
const router = express.Router();
const factry = require('../app/api-factory');
const hogehoge = require('../app/hogehoge');

describe('Test for api-factory', function() {
  beforeEach(function() {
  });
  afterEach(function() {
  });

  it('test for createApi', function() {
    // precondition
    spyOn(hogehoge, "createApi");

    // test
    factry.createApi(router, 'hogehoge');
    expect(hogehoge.createApi).toHaveBeenCalledWith(router);
  });
});
```

* デバッグできるかの確認  
 1. クモマークをクリックしてデバッグペインを開き
 1. リストボックスからユニットテスト用に追加した構成を選択して
 1. リストボックス横の実行をクリックするか､F5実行でデバッグを開始する
  ![jasmine-node-test01.png](https://qiita-image-store.s3.amazonaws.com/0/193342/2ea389ba-1ed1-c8eb-adf4-678b6cb3932e.png)
 1. ブレイクポイントでとまり､ステップ実行が行える
  ![jasmine-node-test02.png](https://qiita-image-store.s3.amazonaws.com/0/193342/5265172f-903f-9552-5434-813a551d82ac.png)

## おまけ
デバッグではなく単純にテストを実行したい場合は package.json に下記を追加して npm test を実行する｡

```javascript:package.json(テスト実行のための設定)
  "scripts": {
    "test": "node ./node_modules/jasmine-node/bin/jasmine-node ./test/"
  },
```

```:テスト実行
npm test
```

```:結果
> my-app@0.0.0 test C:\Users\ksh.fthr\work\Express\my-app
> node ./node_modules/jasmine-node/bin/jasmine-node ./test/

.

Finished in 0.005 seconds
1 test, 1 assertion, 0 failures, 0 skipped
```
