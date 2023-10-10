# はじめに
本記事では [RxJS](https://rxjs.dev) の [Utility Operators](https://rxjs.dev/guide/operators#utility-operators) について学習することを目的とします。
ただすべてを扱うのは量的な面で難しいので、個人的によく使うメソッドに対する理解を深めたいと思います。

# 本記事で扱う [Utility Operators](https://rxjs.dev/guide/operators#utility-operators)

つぎの Operators を扱います。

- [subscribe](https://rxjs.dev/api/index/function/subscribeOn)
- [unsubscribe]()
- [complete]()
- [tap](https://rxjs.dev/api/index/function/tap)
- [toArray](https://rxjs.dev/api/operators/toArray)

以下、サンプルコードを示しつつ各 Operators の動きを見ていきます。

## [subscribe](https://rxjs.dev/api/index/function/subscribeOn)

- データの購読
- Observable で観察していたデータを購読することで、流れてきたデータに対してアレコレする
- そのデータを加工して別のストリームに流したり、そのデータを別の変数から参照させて同期的に扱うようにしたり、とか色々
- [サンプルコードはこちら](./sample-subscribe.md)

## unsubscribe

- ストリームを **完了** させるための仕組み
- `unsubscribe()` 後にストリームを流そうとしても流れない
- `unsubscribe()` を実行しても購読している側の `subscribe` で `complete` のハンドラが実行されない
- [サンプルコードはこちら](./sample-unsubscribe.md)

## complete

- ストリームを **完了** させるための仕組み
- `complete()` 後にストリームを流そうとしても流れない
- `complete()` を実行すると購読している側の `subscribe` で `complete` のハンドラが実行される
- [サンプルコードはこちら](./sample-complete.md)

## [tap](https://rxjs.dev/api/index/function/tap)

ストリームで流れてきたデータに対して同期的に処理を行う。
[公式の tap ページ](https://rxjs-dev.firebaseapp.com/api/operators/tap) には

> Used to perform side-effects for notifications from the source observable

とあり、副作用について扱うメソッドとのこと。具体的にはログ出したり、事前にデータ検証して例外を発生させたり。
新たに値を生成して返すことはしない。イテレータで言うところの `forEach`。

データに対して加工 & それをベースに処理をつなげたい場合は `map` を使う。

- [サンプルコードはこちら](./sample-tap.md)

## [toArray](https://rxjs.dev/api/operators/toArray)

- `complete` が来るまでの `next` を Array に詰めた値を取得する
- [サンプルコードはこちら](./sample-toArray.md)
  - 一緒に [Filtering Operators](https://rxjs.dev/guide/operators#filtering-operators) の [take](https://rxjs.dev/api/operators/take) も扱っている
