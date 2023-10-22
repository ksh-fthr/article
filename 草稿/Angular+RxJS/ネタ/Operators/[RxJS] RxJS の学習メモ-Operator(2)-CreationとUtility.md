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
- [[RxJS] RxJS の学習メモ-Operator(1)-PipeとFilteringOperators](https://qiita.com/ksh-fthr/items/0f34da2cc38311ecf9c6)

今回は [Operators](https://rxjs.dev/guide/operators) についての続きです。

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
ご興味あればご参照ください。

今回の記事では次のオペレータについて触れていきます｡

**本記事で扱うオペレータ**

- ストリームを生成する [Creation Operators](https://rxjs.dev/guide/operators#creation-operators-1)
- ストリームに補助的な動きを与える [Utility Operators](https://rxjs.dev/guide/operators#utility-operators)

これらのオペレータで特に利用シーンが多いと思うものについて扱います｡

# [Creation Operators](https://rxjs.dev/guide/operators#creation-operators-1)

## [of](https://rxjs.dev/api/index/function/of)

## [from](https://rxjs.dev/api/index/function/from)

# [Utility Operators](https://rxjs.dev/guide/operators#utility-operators)

## [tap](https://rxjs.dev/api/operators/tap)

## [subscribe](https://rxjs.dev/api/index/function/subscribeOn)

## [toArray](https://rxjs.dev/api/operators/toArray)
