
# [Creation Operators](https://rxjs.dev/guide/operators#creation-operators-1)

## [of](https://rxjs.dev/api/index/function/of)

- 引数に指定したデータからストリームを作り出す
  - 複数指定可能
  - 配列もオブジェクトも指定できるが、生成されるのはあくまで指定した引数の単位ごと
- [サンプルコードはこちら]()

## [from](https://rxjs.dev/api/index/function/from)

- 引数に指定した配列からストリームを作り出す
  - 配列の要素一つに対してストリームを1つ流す
  - 複数指定不可
- [サンプルコードはこちら]()

## of と from の補足

- `scheduler` パラメータ付きで使用する場合、両方とも非推奨
- RxJS v7.x で実装された [`scheduled`](https://rxjs.dev/api/index/function/scheduled) への置き換えを推奨されている
- [こちら](https://rxjs.dev/deprecations/scheduler-argument#scheduler-argument) を参照。

> ```
> This deprecation was introduced in RxJS 6.5 and will become breaking with RxJS 8.
> ```
