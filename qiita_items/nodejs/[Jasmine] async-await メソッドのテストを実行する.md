await 対象のメソッドに対して、単純にテストケースに async をつけただけだとテストケースが結果に計上されない。
したがって await 対象のメソッドに対してユニットテストを実行する際に少し工夫が必要となる。

## この記事を実施した環境
* Windows10 Home 64bit
* node v8.2.1
* npm v4.0.5
* express v4.15.0
* jasmine-node v1.14.5
* Visual Studio Code v1.15.0

## 前提
jasmine-node でユニットテストを書いていること｡
jasmine-node のインストールについては[こちら](http://qiita.com/ksh-fthr/items/41b57cb8dbeebd7a299f#%E5%89%8D%E6%8F%90)を参照。

## ケース
await 対象のメソッドに対してユニットテストを実行する。

## やりがちなミス
前述のように、テストケースに対して単純に async をつけただけ。

```javascript:これだとexpectの結果が計上されない
it('テスト結果が計上されない', async function() {
  const bar = await hogehoge();
  expect(bar).toEqual('bar');
});
```

## 対応
ヘルパーメソッドを用意してやり、テストケースをラップする。

```javascript:TestHelper.js(ユニットテストのヘルパークラス)
/**
 * ユニットテストのヘルパークラス
 *
 * @constructor
 */
function TestHelper() {}

/**
 * 非同期メソッドのためのヘルパー
 *
 * @param {object} runAsync 補助対象の非同期メソッド
 */
function helperAsync(runAsync) {
  // doneを使って対応する
  return (done) => {
    runAsync().then(done, e => {
      fail(e);
      done();
    })
  };
}

TestHelper.prototype.helperAsync = helperAsync;

module.exports = new TestHelper();
```

``` javascript:ヘルパーでテストケースをラップすることで対応する
const testHelper = require('TestHelper');

it('テスト結果が計上される', testHelper.helperAsync(async function() {
  const bar = await hogehoge();
  expect(bar).toEqual('bar');
}));
```

## 参考
[How to use async/await with Jasmine? #923](https://github.com/jasmine/jasmine/issues/923)
