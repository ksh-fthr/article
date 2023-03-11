# はじめに

Angular と RxJS を使っていく上で得た知見を共有することで、次に挙げた内容のブラッシュアップを狙います。
そして個人的な学習の備忘録でもあるので、本ドキュメントの文体も崩れています。予めご承知おきください。

- 知らないことによる忌避感をなくす
  - RxJS を使った実装は個人的に初見殺しもいいとこな実装だと思っているので
  - 知ることで「あ、別に怖がることないじゃん」という感じに持っていきたい
- 知見の向上、また思い込みや間違った理解の是正を狙う
  - ライブラリを改めてみることでより良い実装の方法、テクニックを得る
  - 正しい、最適だと思っていたものが実は間違っていたことも充分あり得るのでその辺が是正できれば御の字

Angular と RxJS の公式、そして cloud でのサンプル実装を試す場は以下の通り。

|            | URL                       | 備考                                       |
| ---------- | ------------------------- | ---------------------------------------- |
| Angular    | https://angular.io/       | 公式(英語)                                   |
|            | https://angular.jp/       | 公式(日本語)                                  |
| RxJS       | https://rxjs.dev/         | 公式                                       |
| Lern RxJS  | https://www.learnrxjs.io/ | リファレンス的な感じの公式学習サイト                       |
| StackBlitz | https://stackblitz.com/   | RxJS だけでなく Angular とか React とかの実装お試しができる |

# Angular と RxJS

なんていうか切っても切れない関係。Angular でバックエンド( RestAPI ) との疎通で使用する [`HttpClientModule > HttpClient`](https://angular.io/api/common/http/HttpClient) からして、各 Http メソッドで Observable が返る。
( [HttpClientModule についての補足](#Angular%20%E3%81%AE%20HttpClientModule)を後述します )

必然、Angular をやっていく上では RxJS も学ばざるを得なくなる。

# RxJS とは

## 公式から

では RxJS とはなにか。公式のトップには

> RxJS is a library for reactive programming using Observables, to make it easier to compose asynchronous or callback-based code. This project is a rewrite of Reactive-Extensions/RxJS with better performance, better modularity, better debuggable call stacks, while staying mostly backwards compatible, with some breaking changes that reduce the API surface
>
> ( Deepl による翻訳 )
> RxJSはObservablesを使ったリアクティブプログラミングのためのライブラリで、非同期またはコールバックベースのコードを簡単に構成できるようにするものです。このプロジェクトは、Reactive-Extensions/RxJSを、より良いパフォーマンス、より良いモジュール性、より良いデバッグ可能なコールスタックで、ほぼ後方互換性を保ちつつ、APIサーフェイスを縮小するいくつかの破壊的変更で書き直したものである。

とある。実際に使う・コードを読むのが簡単なのかという点は脇に置いておいて、非同期プログラミングを実現するためのライブラリ、という理解をしておく。

## Angular のドキュメントから

Angular のドキュメントにある [RxJSライブラリ](https://angular.jp/guide/rx-library#rxjs-%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA) の説明には以下のように記述がある。(日本語ドキュメントから抜粋)

> リアクティブプログラミングは、データストリームと変更の伝播 (Wikipedia) に関する非同期プログラミングのパラダイムです。RxJS (Reactive Extensions for JavaScript) は、非同期またはコールバックベースのコード (RxJS Docs) の作成を容易にする observables を使用したリアクティブプログラミング用のライブラリです。

こちらも非同期プログラミングの実現、そして **ストリーム** という単語が出てきている。
非同期プログラミングの実現にあたり **ストリーム** という概念をもって実装していく、と理解しておく。

## 実際になにをどうするのか

RxJS -> Observable を使用することで何が起こるのかを簡単に見ていく。

[資料を拝借]
以下の資料が分かりやすかったので転載させていただく。

- [マーブル図で怖くないRxJS](https://www.slideshare.net/bitbankink/rxjs-159715695)

ストリームはまんま「川の流れ」として理解すればよい。システムで扱うデータが「川が流れる」がごとく延々とそのシステム上で垂れ流されている、という感じ。

その流れの中で扱いたいデータにたいして以下のような作業を行っていく。

- 観察( `Observe` )しておいて
- 必要に応じてデータを参照したり
- 加工して別のデータにしたり
- またそのデータをストリームとして扱ったり

# 本記事執筆時のバージョン

Angular: `v14.2.12`, RxJS: `6.6.7` を使用している、

<details>
    <summary>ng version の結果</summary>

```bash
$ ng version

     _                      _                 ____ _     ___
    / \   _ __   __ _ _   _| | __ _ _ __     / ___| |   |_ _|
   / △ \ | '_ \ / _` | | | | |/ _` | '__|   | |   | |    | |
  / ___ \| | | | (_| | |_| | | (_| | |      | |___| |___ | |
 /_/   \_\_| |_|\__, |\__,_|_|\__,_|_|       \____|_____|___|
                |___/


Angular CLI: 14.2.10
Node: 16.19.0
Package Manager: npm 8.19.3
OS: darwin x64

Angular: 14.2.12
... animations, common, compiler, compiler-cli, core, forms
... platform-browser, platform-browser-dynamic, router

Package                         Version
---------------------------------------------------------
@angular-devkit/architect       0.1402.10
@angular-devkit/build-angular   14.2.10
@angular-devkit/core            14.2.10
@angular-devkit/schematics      14.2.10
@angular/cdk                    14.2.7
@angular/cli                    14.2.10
@angular/material               14.2.7
@schematics/angular             14.2.10
rxjs                            6.6.7
typescript                      4.7.4
```

</details>

# よく使われているメソッド

( 個人的な主観込みですが )
RxJS の中でも知っていなければならないメソッド、またポピュラーなメソッドについて本記事では触れていきます。

## [Utility Operators](https://rxjs.dev/guide/operators#utility-operators)

### [subscribe](https://rxjs.dev/api/index/function/subscribeOn)

- データの購読
- Observable で観察していたデータを購読することで、流れてきたデータに対してアレコレする
- そのデータを加工して別のストリームに流したり、そのデータを別の変数から参照させて同期的に扱うようにしたり、とか色々
- [サンプルコードはこちら]()

### [tap](https://rxjs.dev/api/index/function/tap)

ストリームで流れてきたデータに対して同期的に処理を行う。
[公式の tap ページ](https://rxjs-dev.firebaseapp.com/api/operators/tap) には

> Used to perform side-effects for notifications from the source observable

とあり、副作用について扱うメソッドとのこと。具体的にはログ出したり、事前にデータ検証して例外を発生させたり。
新たに値を生成して返すことはしない。イテレータで言うところの `forEach`。

データに対して加工 & それをベースに処理をつなげたい場合は `map` を使う。

- [サンプルコードはこちら]()

### [toArray](https://rxjs.dev/api/operators/toArray)

-  `complete` が来るまでの `next` を Array に詰めた値を取得する( 加藤さんからのコメントより )
- [サンプルコードはこちら]()
  - 一緒に [Filtering Operators](https://rxjs.dev/guide/operators#filtering-operators) の [take](https://rxjs.dev/api/operators/take) も扱っている

## [Piping](https://rxjs.dev/guide/operators#piping)

### [pipe](https://rxjs.dev/api/index/function/pipe)

- `subscribe` で購読する前に、`observable` で観察していたデータを加工するための **繋ぎ** を実現するためのメソッド
- この中で後述のデータ操作やフィルタリングを行う
- [サンプルコードはこちら]()

## [Creation Operators](https://rxjs.dev/guide/operators#creation-operators-1)

### [of](https://rxjs.dev/api/index/function/of)

- 引数に指定したデータからストリームを作り出す
  - 複数指定可能
  - 配列もオブジェクトも指定できるが、生成されるのはあくまで指定した引数の単位ごと
- [サンプルコードはこちら]()

### [from](https://rxjs.dev/api/index/function/from)

- 引数に指定した配列からストリームを作り出す
  - 配列の要素一つに対してストリームを1つ流す
  - 複数指定不可
- [サンプルコードはこちら]()

### of と from の補足

- `scheduler` パラメータ付きで使用する場合、両方とも非推奨
- RxJS v7.x で実装された [`scheduled`](https://rxjs.dev/api/index/function/scheduled) への置き換えを推奨されている
- [こちら](https://rxjs.dev/deprecations/scheduler-argument#scheduler-argument) を参照。

> ```
> This deprecation was introduced in RxJS 6.5 and will become breaking with RxJS 8.
> ```

## [Filtering Operators](https://rxjs.dev/guide/operators#filtering-operators)

### [first](https://rxjs.dev/api/operators/first)

- ストリームが流れてきたとき **最初の値だけ** を次の処理に回す
- [サンプルコードはこちら]()

### [filter](https://rxjs.dev/api/index/function/filter)

- ストリームが流れてきたとき 条件に合致した値を次の処理に回す
- [サンプルコードはこちら]()

### [distinctUntilChanged](https://rxjs.dev/api/index/function/distinctUntilChanged)

- ストリームが流れてきたとき  `comparator` に指定した条件に合致した値だけを処理する
- `comparator` は省略可能
- デフォルトの動きは 「**前回流れてきたデータと異なる場合** に処理する」
- [サンプルコードはこちら]()

## [Transformation Operators](https://rxjs.dev/guide/operators#transformation-operators)

### [map](https://rxjs.dev/api/index/function/map)

- ストリームで流れてきたデータに対して同期的に処理を行う
- そして処理を行った結果の値を返し、それが `subscribe` に流れていく。イテレータで言うところの `map`
- [サンプルコードはこちら]()

### [mergeMap](https://rxjs.dev/api/index/function/mergeMap)

- ストリームで流れてきたデータに対して同期的に処理を行う。ここは `map` と変わらない
  - `map` との違いは **新しくストリームを生成するか否か**
  - `map` は流れてきた値を加工するが、新たにストリームを生成することはない
- **`mergeMap` は流れてきたデータをもとに新たなストリームを生成する**。(これは後述の **`switchMap`** や **`concatMap`** も同じ)
- 新しく生成したストリームは、それを対象に `map` で処理しても良いし、そのまま `subscribe` に流すことも出来る
- `mergeMap` は流れてきた **ストリームの同期処理が片付いた順** に次の処理にデータが流れる
- [サンプルコードはこちら]()

### [switchMap](https://rxjs.dev/api/index/function/switchMap)

- 大まかな部分は `mergeMap` と一緒
- `switchMap` は **最後に処理されたストリームだけを対象に処理** をして次の処理にデータが流れる
- つまり **最後に流れてくるストリームの前に処理されていた内容はキャンセル** される
- [サンプルコードはこちら]()

### [cancatMap](https://rxjs.dev/api/index/function/concatMap)

- こちらも大まかな部分は `mergeMap` と一緒
- `concatMap` は **ストリームが流れてきた 順** に処理を行い次の処理にデータが流れる
- [サンプルコードはこちら]()

## [Join Creation Operators](https://rxjs.dev/guide/operators#join-creation-operators)

### [combineLatest](https://rxjs.dev/api/index/function/combineLatest)

- 複数の Observable 引数に取り、それぞれの Observable から 新しいストリームが流れるたびに流れてきたストリームに対して処理する
- 複数の支流が一つの川に流れ込んで合流するイメージ
- ちなみに `import` 元は `rxjs`
- [サンプルコードはこちら]()

## [Join Operators](https://rxjs.dev/guide/operators#join-operators)

### [withLatestFrom](https://rxjs.dev/api/index/function/withLatestFrom)

- ベースとなる Observable からストリームが流れたら、それをトリガーに引数に取った Observable から流れてきたストリームとの結合処理を行う
- `combineLatest` に類似。違いは処理が行われるトリガーにある
  - `combineLatest` は引数にとった各 Observable のいずれかの値が流れるタイミングでイベント発火する
  - `withLatestFrom` は最初に起点となる Observable の値が流れるタイミングの時だけイベント発火する
- こちらの `import` 元は `rxjs/operators`
- [サンプルコードはこちら]()

# Hot / Cold について

## COLD ストリーム

- 特性-1
  - `subscribe` で購読しない限りストリームは流れてこない ===> 処理できない
- 特性-2
  - ストリームは個別に流れる
- COLDとなるのはどういうときか
  - 通常、利用しているのはこちら
  - `BehaviorSubject`m `Subject`, `Observable` 等のクラスのインスタンスを処理するとこちらになる
- [サンプルコードはこちら]()

## HOT ストリーム

- 特性-1
  - `subscribe` しなくてもストリームが流れる
- 特性-2
  - ストリームは分岐して同じ値がそれぞれのストリームに流れる
- HOT となるのはどういうときか
  - `connectable` クラスのインスタンスを生成し, `connect` オペレータを実行するとこちらになる
- 注意-非推奨オペレータについて
  - HOT ストリームに関する記事でサンプルコードとして頻出する [publish](https://rxjs.dev/api/operators/publish) や [refCount](https://rxjs.dev/api/operators/refCount) は非推奨となった
  - 同じく [ConnectableObservable](https://rxjs.dev/api/index/class/ConnectableObservable) クラスも非推奨となった
  - これらは v8 で廃止予定
  - 今後は [share](https://rxjs.dev/api/operators/share) と [Connectable](https://rxjs.dev/api/index/interface/Connectable) を使用する
    - ( 本項で紹介するサンプルコードはこちらを使用している )
  - API の置き換えについては [Multicasting](https://rxjs.dev/deprecations/multicasting) を参照
- [サンプルコードはこちら]()

# 補足

## Angular の HttpClientModule

[HttpClientModule](https://angular.jp/api/common/http/HttpClientModule) に含まれる [HttpClient](https://angular.jp/api/common/http/HttpClient) について補足する。
以下、公式からの転載。

( 転載もと: [HttpClientのメソッドはひとつの値を返す](https://angular.jp/tutorial/toh-pt6#httpclient%E3%81%AE%E3%83%A1%E3%82%BD%E3%83%83%E3%83%89%E3%81%AF%E3%81%B2%E3%81%A8%E3%81%A4%E3%81%AE%E5%80%A4%E3%82%92%E8%BF%94%E3%81%99) )

> ```text
> HTTPはリクエスト/レスポンスプロトコルです。 リクエストを送信すると、ひとつのレスポンスを返却します。
> 
> 一般には、Observableは時間によって複数の値を返すことが 可能 です。 HttpClientが返すObservableは常にひとつの値を発行してから完了するので、再び値を発行することはありません。
> ```

ということで、`HttpClient` を用いた場合、`first()` 等でストリームが流れる回数に対して制限を掛ける必要はない。

## 上記の意図について更に補足

subscribe にはイベントとして次がある。

- next
  - 複数回流れる
  - データを伴って流れる
- error
  - 一回だけ流れる
  - データを伴って流れる
- complete
  - 一回だけ流れる
  - データを伴って流れない

`HttpClient` では値が取得できたら `next`, 次に `complete` を流す。
なので、ここで `HttpClient` で流れてきたストリームに対して `first` を使っても、その前後で動きが変わることがない。
よって これを使うときに `first` は不要、となる。

# おわりに

RxJS を利用した実装は初見殺しなコードが多くなりがちなので、しっかり理解して間違いのないように使いたいですね。

# 参考

- [Angular-RxJS ライブラリ](https://angular.jp/guide/rx-library#rxjs-%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA)
- [Angular のための RxJS](https://learn-rxjs-for-angular.info/)
- [マーブル図で怖くないRxJS](https://www.slideshare.net/bitbankink/rxjs-159715695)
- [RxJSのconcatMap, mergeMap, switchMapの違いを理解する(中級者向け)](https://qiita.com/ovrmrw/items/b45d7bf29c8d29415bd7)
- [RxJS mapとmergeMapの違い](https://zenn.dev/shrek13/articles/rxjs-map-mergemap)
- [RxJSのオペレーターの動きをデモアプリを自作して確認してみた](https://note.com/shift_tech/n/n7643a684e947#map())
- [CombineLatest vs withLatestFrom](https://medium.com/@vinothinikings/combinelatest-vs-withlatestfrom-5003377b766f)
- [RxJS を学ぼう #2 - よく使う ( と思う ) オペレータ15選](https://blog.recruit.co.jp/rmp/front-end/post-11475/)
- [RxJS を学ぼう #4 - COLD と HOT について学ぶ / ConnectableObservable](https://blog.recruit.co.jp/rmp/front-end/post-11558/)
- [【Rxjsのすゝめ】ObservableのCOLDとHOTって結局なんなの？](https://deep.tacoskingdom.com/blog/25)

# おまけ

- [RxJSを覚えないAngularの書き方【2021年/RxJS7版】](https://zenn.dev/rdlabo/articles/60b4be5be02704)
