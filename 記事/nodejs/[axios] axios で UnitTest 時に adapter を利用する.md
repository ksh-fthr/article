axios を利用したコードで UnitTest を書く際に、axios を介して通信する箇所をモック化した際のメモを残す。

## ケース
前述した部分と重複するが、axios を利用した非同期通信においてモックを使用して UnitTest を書きたい。
(実際に通信は発生させず、擬似的にテストを行いたい)

## 対応
adapter を利用する。

## adapter 利用時の手順
1. モックのための function を用意する
ここで resolve した結果が テストターゲットの then に返ってくる。

    ```javascript:モックのためのfunction
    const mockAdapter = function mockAdapter() {
      return new Promise(function(resolve) {
        const res = {
          'data': 'HOGE',
          'status': 200
        }
        resolve(res);
      });
    };
    ```

1. axios の第2引数(GET時)に指定するオプションを用意する

    ```javascript:モックを利用するためのオプション
    const adapterOption = {
      // 用意したモックのための function をセット
      'adapter': mockAdapter
    };
    ```

1. 用意したオプションを axios を利用しているコードにセットする

    ```javascript:axios利用しているコードにオプションをセット
    axios.get(url, adapterOption)
    .then(function(res) {
      // モックのための function(mockAdapter) で resolve した結果がここに来る
      console.log(res);
    })
    .catch(function(err) {
      console.log(err);
    });
    ```

* 上記は axios を GET で利用しているケース
* PUT, POST で axios を利用している場合は、第3引数にオプション(ここでは adapterOption )をセットする
* このとき、もし実際に使用している axios を利用している部分のコードがオプションを用意していなかったら、axios 利用部分の実装そのものをモック化してやれば良い

## 参考
* [本家の Request Config 内 adapter の記述](https://github.com/mzabriskie/axios#request-config)

    ```javascript:抜粋
    // `adapter` allows custom handling of requests which makes testing easier.
    // Return a promise and supply a valid response (see lib/adapters/README.md).
    adapter: function (config) {
      /* ... */
    },
    ```
