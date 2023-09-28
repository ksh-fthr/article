# [Join Operators](https://rxjs.dev/guide/operators#join-operators)

## [withLatestFrom](https://rxjs.dev/api/index/function/withLatestFrom)

- ベースとなる Observable からストリームが流れたら、それをトリガーに引数に取った Observable から流れてきたストリームとの結合処理を行う
- `combineLatest` に類似。違いは処理が行われるトリガーにある
  - `combineLatest` は引数にとった各 Observable のいずれかの値が流れるタイミングでイベント発火する
  - `withLatestFrom` は最初に起点となる Observable の値が流れるタイミングの時だけイベント発火する
- こちらの `import` 元は `rxjs/operators`
- [サンプルコードはこちら](./sample-withLatestFrom.md)
