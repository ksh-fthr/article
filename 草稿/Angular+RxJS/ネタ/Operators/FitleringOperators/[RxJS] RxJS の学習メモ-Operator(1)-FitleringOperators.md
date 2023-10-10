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

今回は [Operator](https://rxjs.dev/guide/subject) について学びます。

# 環境

本記事について扱うライブラリや環境の情報です。

|                                        | 備考                                                        |
| -------------------------------------- | ----------------------------------------------------------- |
| [RxJS](https://rxjs.dev/)              | 公式                                                        |
| [Learn RxJS](https://www.learnrxjs.io/) | リファレンス的な感じの学習サイト                        |
| [StackBlitz](https://stackblitz.com/)  | RxJS だけでなく Angular とか React とかの実装お試しができる |

なお本記事執筆時の RxJS のバージョンは StackBlitz の DEPENDENCIES を見ると `v7.8.0` でした。

# この記事でやること

## [`Subject`](https://rxjs.dev/guide/subject) について触れてみる

本記事では `Subject` について触れます。
`Subject` は RxJS の基本コンセプトである 6つ のコンセプトに含まれるものです。

## RxJS の基本コンセプト

RxJS の基本コンセプトは `Observable`, `Observer`, `Subscription`, `Operators`, `Subject`, `Schedulers` の 6つ があります。
これは公式のドキュメントにも [Overview](https://rxjs.dev/guide/overview) に下記のとおり記載されています。
( 太字については本記事にて加工しました )

:::note info

> The essential concepts in RxJS which solve async event management are:
>
> - Observable: represents the idea of an invokable collection of future values or events.
> - Observer: is a collection of callbacks that knows how to listen to values delivered by the Observable.
> - Subscription: represents the execution of an Observable, is primarily useful for cancelling the execution.
> - Operators: are pure functions that enable a functional programming style of dealing with collections with operations like map, filter, concat, reduce, etc.
> - **Subject: is equivalent to an EventEmitter, and the only way of multicasting a value or event to multiple Observers.**
> - Schedulers: are centralized dispatchers to control concurrency, allowing us to coordinate when computation happens on e.g. setTimeout or requestAnimationFrame or others.
>
> (Deepl による翻訳)
>
> 非同期イベント管理を解決するRxJSの本質的な概念は以下の通りである：
>
> - Observable（オブザーバブル）：将来の値やイベントの呼び出し可能なコレクション。
> - Observer：Observableによって配信される値をリッスンする方法を知っているコールバックのコレクションです。
> - サブスクリプション: Observableの実行を表し、主に実行をキャンセルするのに役立つ。
> - Operators: map、filter、concat、reduceなどの操作でコレクションを扱う関数型プログラミングスタイルを可能にする純粋な関数です。
> - **Subject: EventEmitterに相当し、値やイベントを複数のObserversにマルチキャストする唯一の方法です。**
> - Schedulers: 同時実行をコントロールするための集中ディスパッチャで、例えばsetTimeoutやrequestAnimationFrameなどで計算が発生するタイミングを調整できる。

:::

前置きが長くなりましが、これから実際に `Subject` を見ていきます。

# [Operator](https://rxjs.dev/guide/subject)

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

# 補足

## Angular の HttpClientModule

Angular では Http クライアントの実装で [HttpClientModule](https://angular.jp/api/common/http/HttpClientModule) に含まれる [HttpClient](https://angular.jp/api/common/http/HttpClient) を利用することが多いと思います。
この HttpClient でも RxJS を利用していますので、それについて補足します。

以下、公式からの転載です。

( 転載もと: [HttpClient のメソッドはひとつの値を返す](https://angular.jp/tutorial/toh-pt6#httpclient%E3%81%AE%E3%83%A1%E3%82%BD%E3%83%83%E3%83%89%E3%81%AF%E3%81%B2%E3%81%A8%E3%81%A4%E3%81%AE%E5%80%A4%E3%82%92%E8%BF%94%E3%81%99) )

> ```text
> HTTPはリクエスト/レスポンスプロトコルです。 リクエストを送信すると、ひとつのレスポンスを返却します。
>
> 一般には、Observableは時間によって複数の値を返すことが 可能 です。 HttpClientが返すObservableは常にひとつの値を発行してから完了するので、再び値を発行することはありません。
> ```

ということで、`HttpClient` を用いた場合、`first()` 等でストリームが流れる回数に対して制限を掛ける必要はありません。
上記を更に補足しますと、 `subscribe` には次のイベントが用意されています。
それぞれ

- `next`
  - 複数回流れる
  - データを伴って流れる
- `error`
  - 一回だけ流れる
  - データを伴って流れる
- `complete`
  - 一回だけ流れる
  - データを伴って流れない

というものですが、Angular の `HttpClient` では **値が取得できたら `next`, 次に `complete` を流し** ます。
ということは、`HttpClient` で流れてきたストリームに対して `first` を使っても、(`complete` が流れてきているので) その前後で動きが変わることがない、となります。

つまり `HttpCliente` 使うときに `first` は **使っても使わなくても結果は変わらず**、よって  **使う必要がない イコール 不要**、となります。

# 参考

- [RxJS Marbles](https://rxmarbles.com/)
- [マーブル図で怖くない RxJS](https://www.slideshare.net/bitbankink/rxjs-159715695)
- [RxJS の concatMap, mergeMap, switchMap の違いを理解する(中級者向け)](https://qiita.com/ovrmrw/items/b45d7bf29c8d29415bd7)
- [RxJS map と mergeMap の違い](https://zenn.dev/shrek13/articles/rxjs-map-mergemap)
- [RxJS のオペレーターの動きをデモアプリを自作して確認してみた](<https://note.com/shift_tech/n/n7643a684e947#map()>)
- [CombineLatest vs withLatestFrom](https://medium.com/@vinothinikings/combinelatest-vs-withlatestfrom-5003377b766f)
- [RxJS を学ぼう #2 - よく使う ( と思う ) オペレータ 15 選](https://blog.recruit.co.jp/rmp/front-end/post-11475/)
- [RxJS を学ぼう #4 - COLD と HOT について学ぶ / ConnectableObservable](https://blog.recruit.co.jp/rmp/front-end/post-11558/)
- [【Rxjs のすゝめ】Observable の COLD と HOT って結局なんなの？](https://deep.tacoskingdom.com/blog/25)
- [Angular-RxJS ライブラリ](https://angular.jp/guide/rx-library#rxjs-%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA)
- [Angular のための RxJS](https://learn-rxjs-for-angular.info/)
