# はじめに

本記事は RxJS を使っていく上で行った学習の備忘録になります。主に次に挙げた内容を目的とします。

:::note info

- 知らないことによる忌避感をなくす
  - RxJS を使った実装は個人的に初見殺しもいいとこな実装だと思っている
  - 知ることで「あ、別に怖がることないじゃん」という感じに持っていきたい
- 知見の向上
  - ライブラリを改めてみることでより良い実装の方法、テクニックを得る
- 思い込みや間違った理解の是正
  - 正しい、最適だと思っていたものが実は間違っていたことも充分あり得るので、その辺が是正できれば御の字

:::

# これまでと今回

これまでに次の記事を投稿してまいりました。

- [[RxJS] RxJS の学習メモ-Observable と Observer](https://qiita.com/ksh-fthr/items/2933492929bbeccece50)
- [[RxJS] RxJS の学習メモ-Subject](https://qiita.com/ksh-fthr/items/54b19b4160505e2fddd9)
- [[RxJS] RxJS の学習メモ-Subject の派生](https://qiita.com/ksh-fthr/items/6c6cb89acce6fe756c60)
- [[RxJS] RxJS の学習メモ-Operator(1)-PipeとFiltering](https://qiita.com/ksh-fthr/items/0f34da2cc38311ecf9c6)
- [[RxJS] RxJS の学習メモ-Operator(2)-CreationとUtility](https://qiita.com/ksh-fthr/items/3f5ecb5bf47ad0216101)

今回も [Operators](https://rxjs.dev/guide/operators) についての続きです。

# 環境

本記事について扱うライブラリや環境の情報です。

|                                        | 備考                                                        |
| -------------------------------------- | ----------------------------------------------------------- |
| [RxJS](https://rxjs.dev/)              | 公式                                                        |
| [Learn RxJS](https://www.learnrxjs.io/) | リファレンス的な感じの学習サイト                        |
| [StackBlitz](https://stackblitz.com/)  | RxJS だけでなく Angular とか React とかの実装お試しができる |

なお本記事執筆時の RxJS のバージョンは StackBlitz の DEPENDENCIES を見ると `v7.8.0` でした。

# この記事でやること

## [Operators](https://rxjs.dev/guide/operators) について触れてみる(2)

前回の記事は [こちら](https://qiita.com/ksh-fthr/items/0f34da2cc38311ecf9c6) です｡
ご興味あればご覧ください。

今回の記事では次のオペレータについて触れていきます｡

**本記事で扱うオペレータ**

- ストリームから新たなストリームに変換する [Transformation Operators](https://rxjs.dev/guide/operators#transformation-operators)

こちらもすべてを扱うのは量的な面で難しいので、個人的によく使うメソッドに対する理解を深めたいと思います。

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

# 参考

- [RxJS Marbles](https://rxmarbles.com/)
- [マーブル図で怖くない RxJS](https://www.slideshare.net/bitbankink/rxjs-159715695)
- [RxJS の concatMap, mergeMap, switchMap の違いを理解する(中級者向け)](https://qiita.com/ovrmrw/items/b45d7bf29c8d29415bd7)
- [RxJS map と mergeMap の違い](https://zenn.dev/shrek13/articles/rxjs-map-mergemap)
- [RxJS のオペレーターの動きをデモアプリを自作して確認してみた](<https://note.com/shift_tech/n/n7643a684e947#map()>)
- [RxJS を学ぼう #2 - よく使う ( と思う ) オペレータ 15 選](https://blog.recruit.co.jp/rmp/front-end/post-11475/)
- [Angular-RxJS ライブラリ](https://angular.jp/guide/rx-library#rxjs-%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA)
- [Angular のための RxJS](https://learn-rxjs-for-angular.info/)

