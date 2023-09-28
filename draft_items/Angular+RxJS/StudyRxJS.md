# はじめに

RxJS を使っていく上で行った学習の備忘録になります。主に次に挙げた内容のブラッシュアップを狙いました。

- 知らないことによる忌避感をなくす
  - RxJS を使った実装は個人的に初見殺しもいいとこな実装だと思っている
  - 知ることで「あ、別に怖がることないじゃん」という感じに持っていきたい
- 知見の向上、また思い込みや間違った理解の是正を狙う
  - ライブラリを改めてみることでより良い実装の方法、テクニックを得る
  - 正しい、最適だと思っていたものが実は間違っていたことも充分あり得るのでその辺が是正できれば御の字

RxJS の公式、そして cloud でのサンプル実装を試す場は以下の通りです。

|            | URL                       | 備考                                                        |
| ---------- | ------------------------- | ----------------------------------------------------------- |
| RxJS       | https://rxjs.dev/         | 公式                                                        |
| Lern RxJS  | https://www.learnrxjs.io/ | リファレンス的な感じの公式学習サイト                        |
| StackBlitz | https://stackblitz.com/   | RxJS だけでなく Angular とか React とかの実装お試しができる |

なお本記事執筆時の RxJS のバージョンは StackBlitz の DEPENDENCIES を見ると `v7.8.0` でした。

# RxJS とは

## 公式から

[公式のトップページ](https://rxjs.dev/) から引用。

> RxJS is a library for reactive programming using Observables, to make it easier to compose asynchronous or callback-based code. This project is a rewrite of Reactive-Extensions/RxJS with better performance, better modularity, better debuggable call stacks, while staying mostly backwards compatible, with some breaking changes that reduce the API surface
>
> ( Deepl による翻訳 )
> RxJS は Observables を使ったリアクティブプログラミングのためのライブラリで、非同期またはコールバックベースのコードを簡単に構成できるようにするものです。このプロジェクトは、Reactive-Extensions/RxJS を、より良いパフォーマンス、より良いモジュール性、より良いデバッグ可能なコールスタックで、ほぼ後方互換性を保ちつつ、API サーフェイスを縮小するいくつかの破壊的変更で書き直したものである。

とあります。
実際に使う・コードを読むのが簡単なのかという点は脇に置いておいて、**非同期プログラミングを実現するためのライブラリ** 、という風に理解をしておきましょう。

## Angular のドキュメントから

( 蛇足ながら...) RxJS は自分がよく使う Angular とも関係が深いので、Angualr のドキュメントでも確認してみます。
そちらには [RxJS ライブラリ](https://angular.jp/guide/rx-library#rxjs-%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA) の説明に以下の記述があります。
(日本語ドキュメントから抜粋)

> リアクティブプログラミングは、データストリームと変更の伝播 (Wikipedia) に関する非同期プログラミングのパラダイムです。RxJS (Reactive Extensions for JavaScript) は、非同期またはコールバックベースのコード (RxJS Docs) の作成を容易にする observables を使用したリアクティブプログラミング用のライブラリです。

こちらも非同期プログラミングの実現、そして **ストリーム** という単語が出てきています。
非同期プログラミングの実現にあたり **ストリーム** という概念をもって実装していく、と理解しておきます。

## 実際になにをどうするのか

ストリームを「川の流れ」として捉えると理解し易いかと思います。システムで扱うデータが「川が流れる」がごとく延々とそのシステム上で垂れ流されている、というイメージです。
プログラマは、その流れの中で扱いたいデータに対して以下のような作業を行っていくことでアプリケーションを構築していきます。

1. アプリケーションで扱うデータを流す | 流れてくる. ( ストリームを流す | 流れてくる)
2. ストリームは 1 つ とは限らず、システム上に 複数 流れている場合もある
3. データを扱う側はストリームを観察( `Observe` )しておき、
4. ストリームからデータを参照したり
5. そのデータを加工して別のデータにしたり
6. 加工したデータを更にストリームとして扱ったり
7. 等々...オペレータを活用してストリームを目的に応じて利用していく

# Observables, Observer, Subscription, Operators, Subjects

RxJS を学ぶ際に重要な概念として Observables, Observer, Subscription, Operators, Subjects があります。
本記事ではこれら 5つ について触れます。

なお公式のドキュメントにも [Overview](https://rxjs.dev/guide/overview) に下記のとおり記載があり、基本的なコンセプトとして挙がっています。
( Scheduler については不勉強につき割愛します)

> The essential concepts in RxJS which solve async event management are:
>
> - Observable: represents the idea of an invokable collection of future values or events.
> - Observer: is a collection of callbacks that knows how to listen to values delivered by the Observable.
> - Subscription: represents the execution of an Observable, is primarily useful for cancelling the execution.
> - Operators: are pure functions that enable a functional programming style of dealing with collections with operations like map, filter, concat, reduce, etc.
> - Subject: is equivalent to an EventEmitter, and the only way of multicasting a value or event to multiple Observers.
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
> - Subject: EventEmitterに相当し、値やイベントを複数のObserversにマルチキャストする唯一の方法です。
> - Schedulers: 同時実行をコントロールするための集中ディスパッチャで、例えばsetTimeoutやrequestAnimationFrameなどで計算が発生するタイミングを調整できる。

## Observables

- Observables についての記事は [こちら](./Observables/Observables.md)

## Observer

- Observer についての記事は [こちら](./Observer/Observer.md)

## Subjects

- Subjects についての記事は [こちら](./Subject/Subject.md)

## Subscription

- Subscription についての記事は [こちら](./Subscription/Subscription.md)

## Operators

### [Utility Operators](https://rxjs.dev/guide/operators#utility-operators)

- Utility Operators についての記事は [こちら](./Operators/UtilityOperators/UtilityOperators.md)

### [Piping](https://rxjs.dev/guide/operators#piping)

- Piping についての記事は [こちら](./Operators/Piping/Piping.md)

### [Creation Operators](https://rxjs.dev/guide/operators#creation-operators-1)

- Creation Operators についての記事は [こちら](./Operators/CreationOperators/CreationOperators.md)

### [Filtering Operators](https://rxjs.dev/guide/operators#filtering-operators)

- Filtering Operators についての記事は [こちら](./Operators/FitleringOperators/FitleringOperators.md)

### [Transformation Operators](https://rxjs.dev/guide/operators#transformation-operators)

- Transformation Operators についての記事は [こちら](./Operators/TransformationOperators/TransformationOperators.md)

### [Join Creation Operators](https://rxjs.dev/guide/operators#join-creation-operators)

- Join Creation Operators についての記事は [こちら](./Operators/JoinCreationOperators/JoinCreationOperators.md)

### [Join Operators](https://rxjs.dev/guide/operators#join-operators)

- Join Operators についての記事は [こちら](./Operators/JoinOperators/JoinOperators.md)

# Hot / Cold について

- Hot / Cold についての記事は [こちら](./Hot-Cold/Hot-Cold.md)

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

# おわりに

RxJS を利用した実装は初見殺しなコードが多くなりがちなので、しっかり理解して間違いのないように使いたいですね。

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

# おまけ

- [RxJS を覚えない Angular の書き方【2021 年/RxJS7 版】](https://zenn.dev/rdlabo/articles/60b4be5be02704)
