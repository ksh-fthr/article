# [Transformation Operators](https://rxjs.dev/guide/operators#transformation-operators)

## [map](https://rxjs.dev/api/index/function/map)

- ストリームで流れてきたデータに対して同期的に処理を行う
- そして処理を行った結果の値を返し、それが `subscribe` に流れていく。イテレータで言うところの `map`
- [サンプルコードはこちら](./sample-map.md)

## [mergeMap](https://rxjs.dev/api/index/function/mergeMap)

- ストリームで流れてきたデータに対して同期的に処理を行う。ここは `map` と変わらない
  - `map` との違いは **新しくストリームを生成するか否か**
  - `map` は流れてきた値を加工するが、新たにストリームを生成することはない
- **`mergeMap` は流れてきたデータをもとに新たなストリームを生成する**。(これは後述の **`switchMap`** や **`concatMap`** も同じ)
- 新しく生成したストリームは、それを対象に `map` で処理しても良いし、そのまま `subscribe` に流すことも出来る
- `mergeMap` は流れてきた **ストリームの同期処理が片付いた順** に次の処理にデータが流れる
- [サンプルコードはこちら](./sample-mergeMap.md)

## [switchMap](https://rxjs.dev/api/index/function/switchMap)

- 大まかな部分は `mergeMap` と一緒
- `switchMap` は **最後に処理されたストリームだけを対象に処理** をして次の処理にデータが流れる
- つまり **最後に流れてくるストリームの前に処理されていた内容はキャンセル** される
- [サンプルコードはこちら](./sample-switchMap.md)

## [cancatMap](https://rxjs.dev/api/index/function/concatMap)

- こちらも大まかな部分は `mergeMap` と一緒
- `concatMap` は **ストリームが流れてきた 順** に処理を行い次の処理にデータが流れる
- [サンプルコードはこちら](./sample-concatMap.md)
