# [Filtering Operators](https://rxjs.dev/guide/operators#filtering-operators)
  
## [first](https://rxjs.dev/api/operators/first)

- ストリームが流れてきたとき **最初の値だけ** を次の処理に回す
- [サンプルコードはこちら](./sample-first.md

## [filter](https://rxjs.dev/api/index/function/filter)

- ストリームが流れてきたとき 条件に合致した値を次の処理に回す
- [サンプルコードはこちら](./sample-filter.md)

## [distinctUntilChanged](https://rxjs.dev/api/index/function/distinctUntilChanged)

- ストリームが流れてきたとき `comparator` に指定した条件に合致した値だけを処理する
- `comparator` は省略可能
- デフォルトの動きは 「**前回流れてきたデータと異なる場合** に処理する」
- [サンプルコードはこちら](./sample-distinctUntilChanged.md)
