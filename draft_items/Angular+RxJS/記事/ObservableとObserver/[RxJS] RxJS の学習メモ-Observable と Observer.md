# はじめに

RxJS を使っていく上で行った学習の備忘録になります。主に次に挙げた内容を目的とします。

:::note info

- 知らないことによる忌避感をなくす
  - RxJS を使った実装は個人的に初見殺しもいいとこな実装だと思っている
  - 知ることで「あ、別に怖がることないじゃん」という感じに持っていきたい
- 知見の向上
  - ライブラリを改めてみることでより良い実装の方法、テクニックを得る
- 思い込みや間違った理解の是正
  - 正しい、最適だと思っていたものが実は間違っていたことも充分あり得るので、その辺が是正できれば御の字

:::

# 環境

本記事について扱うライブラリや環境の情報です。

|                                        | 備考                                                        |
| -------------------------------------- | ----------------------------------------------------------- |
| [RxJS](https://rxjs.dev/)              | 公式                                                        |
| [Learn RxJS](https://www.learnrxjs.io/) | リファレンス的な感じの学習サイト                        |
| [StackBlitz](https://stackblitz.com/)  | RxJS だけでなく Angular とか React とかの実装お試しができる |

なお本記事執筆時の RxJS のバージョンは StackBlitz の DEPENDENCIES を見ると `v7.8.0` でした。

# RxJS とは

## 公式から

この記事でやることについて触れる前に、まず RxJS とはなにか、から見ていきます。
以下、[公式のトップページ](https://rxjs.dev/) からの引用です。

:::note info

> RxJS is a library for reactive programming using Observables, to make it easier to compose asynchronous or callback-based code. This project is a rewrite of Reactive-Extensions/RxJS with better performance, better modularity, better debuggable call stacks, while staying mostly backwards compatible, with some breaking changes that reduce the API surface
>
> ( Deepl による翻訳 )
> RxJS は Observables を使ったリアクティブプログラミングのためのライブラリで、非同期またはコールバックベースのコードを簡単に構成できるようにするものです。このプロジェクトは、Reactive-Extensions/RxJS を、より良いパフォーマンス、より良いモジュール性、より良いデバッグ可能なコールスタックで、ほぼ後方互換性を保ちつつ、API サーフェイスを縮小するいくつかの破壊的変更で書き直したものである。

:::

とあります。
実際に使う・コードを読むのが簡単なのかという点は脇に置いておいて、**非同期プログラミングを実現するためのライブラリ** 、という風に理解をしておきましょう。

## Angular のドキュメントから

( 蛇足ながら...) RxJS は自分がよく使う Angular とも関係が深いので、Angualr のドキュメントでも確認してみます。
そちらには [RxJS ライブラリ](https://angular.jp/guide/rx-library#rxjs-%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA) の説明に以下の記述があります。
(日本語ドキュメントから抜粋)

:::note info

> リアクティブプログラミングは、データストリームと変更の伝播 (Wikipedia) に関する非同期プログラミングのパラダイムです。RxJS (Reactive Extensions for JavaScript) は、非同期またはコールバックベースのコード (RxJS Docs) の作成を容易にする observables を使用したリアクティブプログラミング用のライブラリです。

:::

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

# この記事でやること

## `Observable` と `Observer` について触れてみる

本記事では `Observable` と `Observer` について触れます。
この 2つ は RxJS の基本コンセプトである 6つ のコンセプトに含まれるものです。

## RxJS の基本コンセプト

RxJS の基本コンセプトは `Observable`, `Observer`, `Subscription`, `Operators`, `Subject`, `Schedulers` の 6つ があります。
これは公式のドキュメントにも [Overview](https://rxjs.dev/guide/overview) に下記のとおり記載されています。
( 太字については本記事にて加工しました )

:::note info

> The essential concepts in RxJS which solve async event management are:
>
> - **Observable: represents the idea of an invokable collection of future values or events.**
> - **Observer: is a collection of callbacks that knows how to listen to values delivered by the Observable.**
> - Subscription: represents the execution of an Observable, is primarily useful for cancelling the execution.
> - Operators: are pure functions that enable a functional programming style of dealing with collections with operations like map, filter, concat, reduce, etc.
> - Subject: is equivalent to an EventEmitter, and the only way of multicasting a value or event to multiple Observers.
> - Schedulers: are centralized dispatchers to control concurrency, allowing us to coordinate when computation happens on e.g. setTimeout or requestAnimationFrame or others.
>
> (Deepl による翻訳)
>
> 非同期イベント管理を解決するRxJSの本質的な概念は以下の通りである：
>
> - **Observable（オブザーバブル）：将来の値やイベントの呼び出し可能なコレクション。**
> - **Observer：Observableによって配信される値をリッスンする方法を知っているコールバックのコレクションです。**
> - サブスクリプション: Observableの実行を表し、主に実行をキャンセルするのに役立つ。
> - Operators: map、filter、concat、reduceなどの操作でコレクションを扱う関数型プログラミングスタイルを可能にする純粋な関数です。
> - Subject: EventEmitterに相当し、値やイベントを複数のObserversにマルチキャストする唯一の方法です。
> - Schedulers: 同時実行をコントロールするための集中ディスパッチャで、例えばsetTimeoutやrequestAnimationFrameなどで計算が発生するタイミングを調整できる。

:::

前置きが長くなりました。では実際に `Observable` と `Observer` を見ていきます。

# [Observer](https://rxjs.dev/guide/observer)

順序が前後しますが、まずは `Observer` から。
上記のリンクをみますと以下の記述があります。

:::note info

( 上記リンクから転載 )
> **What is an Observer?** An Observer is a consumer of values delivered by an Observable. Observers are simply a set of callbacks, one for each type of notification delivered by the Observable: `next`, `error`, and `complete`. The following is an example of a typical Observer object:
>
> ( Deepl による翻訳 )
> Observerとは何ですか？Observerとは、Observableによって配信される値の消費者のことです。Observerはコールバックのセットで、Observableによって配信される通知の各タイプ（next、error、complete）に対して1つずつあります。以下は、典型的なObserverオブジェクトの例です

```typescript
const observer = {
  next: x => console.log('Observer got a next value: ' + x),
  error: err => console.error('Observer got an error: ' + err),
  complete: () => console.log('Observer got a complete notification'),
};
```

:::

[前掲のコンセプト](#rxjs-の基本コンセプト) での説明や上記の説明文から次のように理解しておきます。

:::note info

Observer とは Observable によって配信されたストリームを受けてアレコレするための受け皿である。
このとき Observer は次のオブジェクトとして表現される。

Observer には next, error, complete の通知タイプがあり、

- next: 配信で流されてきたデータを処理する
- error: 配信でエラーが発生したときの処理を行う
- complete: 配信が終了したときの処理を行う

これらはそれぞれ次のコードで示す形で処理される。

```typescript
// Observable によって配信された値を扱うオブジェクト === Observer
const observer = {
  next: x => console.log('Observer got a next value: ' + x),
  error: err => console.error('Observer got an error: ' + err),
  complete: () => console.log('Observer got a complete notification'),
};
```

:::

# [Observable](https://rxjs.dev/guide/observable)

今度は `Observable` です。上記リンクには次の記述があります。

:::note info

( 上記リンクから転載 )
> Observables are lazy Push collections of multiple values. They fill the missing spot in the following table:
>
> ( Deepl による翻訳 )
> Observablesは、複数の値を集めたLazy Pushの集合体です。以下の表の欠落している部分を埋めるものです：

:::

これら [Observer](https://rxjs.dev/guide/observer) と [Observable](https://rxjs.dev/guide/observer) の説明に対する理解をサンプルコードによって深めていきます。

# Observer と Observable の動きを確認する

こちらのサンプルコードは [公式のサンプルコード](https://rxjs.dev/guide/observer#observer) のベースにしたものです。
コード中のコメントや変数、`subscribe` のコールバック処理の記法等はこちらで変更・追加しています。

```typescript
import { Observable } from 'rxjs';

// (1) 最初のブロック
// (1-1) 消費者である `observer` と 配信者である `subscriber` 
const observer = new Observable((subscriber) => {
  // (1-2) `next` でストリームを配信する
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);

  // (1-3) `next` によるストリーム配信を遅延実行する
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});

// 二番目のブロック
// (2-1) `subscribe` の実行前であることを示す
console.log('just before subscribe');

// 三番目のブロック
// (3-1) `subscribe` によるストリームの購読を行う
observer.subscribe({
  // (3-2) ストリームに対して next, error, complete でそれぞれのイベントを処理する
  next: (x) => {
    console.log('got value ' + x);
  },
  error: (err) => {
    console.error('something wrong occurred: ' + err);
  },
  complete: () => {
    console.log('done');
  },
});

// 四番目のブロック
// (4-1) `subscribe` の実行後であることを示す
console.log('just after subscribe');
```

ではサンプルコードで何を行っているかを見ていきます。

## 最初のブロック

```typescript
// 最初のブロック
// (1-1) 消費者である `observer` と 配信者である `subscriber` 
const observer = new Observable((subscriber) => {
  // (1-2) `next` でストリームを配信する
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);

  // (1-3) `next` によるストリーム配信を遅延実行する
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});
```

**(1-1) 消費者である `observer` と 配信者である `subscriber`**
ここでは **消費者たる observer** と **配信者たる new Observable() のコールバック処理** を同時に定義しています。

- `observer` => 消費者
- `new Observable() のコールバック処理` => 配信者

です。
こうすることで `observer` は コールバック処理によって配信されるデータを購読するオブジェクト として生成されました。

**(1-2) `next` でストリームを配信する**
`next(1)~next(3)` ではストリームの配信処理が同期的に実行されます。後述の処理で遅延実行される `next(4)` との対比で抑えておきます。

**(1-3) `next` によるストリーム配信を遅延実行する**
`next(4)` はストリームが遅延配信されます。前述の `next(1)~next(4)` との対比で抑えておきます。

## 二番目のブロック

```typescript
// 二番目のブロック
// (2-1) `subscribe` の実行前であることを示す
console.log('just before subscribe');
```

**(2-1) `subscribe` の前であることを示す**
このブロックは `subscribe` の前に実行されるログ、ということで項目を分けました。
[リファレンス](https://rxjs.dev/guide/observer#creating-observables) に次の記述があるように

:::note info

> To invoke the Observable and see these values, we need to subscribe to it:
> ( Deepl による翻訳 )
> Observableを起動してこれらの値を見るには、サブスクライブする必要がある：

:::

**`Observable` は **subscribe** されることでストリームが購読される** ので、サンプルコードの処理ではまず最初にこのログが出力されます。

## 三番目のブロック

```typescript
// 三番目のブロック
// (3-1) `subscribe` によるストリームの購読を行う
observer.subscribe({
  // (3-2) ストリームに対して next, error, complete でそれぞれのイベントを処理する
  next: (x) => {
    console.log('got value ' + x);
  },
  error: (err) => {
    console.error('something wrong occurred: ' + err);
  },
  complete: () => {
    console.log('done');
  },
});
```

**(3-1) `subscribe` によるストリームの購読を行う**
消費者である `observer` が流れてきたストリームを処理 ( === **ストリームを購読** ) します。
なお前述したようにストリームは `subscribe` しない限り配信されませんので、 ここで `observer` は `subscribe` して配信者からストリームが流れるようにしています。

**(3-2) ストリームに対して next, error, complete でそれぞれのイベントを処理する**
配信者からのストリームは `subscribe` のコールバックで処理されます。
すなわち

- next: 配信で流されてきたデータを処理する
- error: 配信でエラーが発生したときの処理を行う
- complete: 配信が終了したときの処理を行う

です。これは前掲の [Observer のできること](#observer-のできること) で示したとおりです。
このサンプルコードではそれぞれ次の処理を行います。

1. next: 通常処理, 流れてきたストリームを出力する
2. error: エラー処理, エラー内容をログ出力する
3. complete: 完了処理( **complete** が実行されるとこのハンドラが実行される), `done` をログ出力する

## 四番目のブロック

```typescript
// 四番目のブロック
// (4-1) `subscribe` の実行後であることを示す
console.log('just after subscribe');
```

**(4-1) `subscribe` の実行後であることを示す**
[二番目のブロック](#二番目のブロック) と同じく、`subscribe` が実行された後に出るログ、ということで項目を分けました。
このログがあることで、[最初のブロック](#最初のブロック) で示した遅延実行の `next(4)` の実行結果が分かりやすくなっています。

(再掲: 配信者である `new Observable()` のコールバックで実施している遅延処理)

```typescript
  // (1-3) `next` によるストリーム配信を遅延実行する
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
```

## 実行結果

このコードの実行結果は次のとおりです。

```log
// Logs:
// just before subscribe
// got value 1           // subscribe によって出力されたログ
// got value 2           // 同上
// got value 3           // 同上
// just after subscribe
// got value 4           // subscribe によって出力されたログ
// done                  // complete によって出力されたログ
```

コメントに記載しておりますが、

1. `subscribe` 前にログが出力され
2. `next(1)~next(3)` がその後に連続して( 同期的に )処理されたことを示すログが出力された
3. ついで `subscribe` 後であることを示すログが出力され
4. 最後に `setTimeout` で遅延実行された `next(4)` と `complete` によるログが出力された

ことが分ります。

# Appendix

サンプルコードの `next(1)` ~ `nex(4)` の間に `subscriber.error()` や `subscriber.complete()` を入れるとログの出方が変わります。

## error が通知されたときの動きを確認する

```typescript
import { Observable } from 'rxjs';

const observer = new Observable((subscriber) => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);

  // エラーを発生させて後続の動きを確認する
  subscriber.error('happening!');
  
  // エラーが発生しているのでこの遅延処理部分のログは出力されない
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});

console.log('just before subscribe');

observer.subscribe({
  next: (x) => {
    console.log('got value ' + x);
  },
  error: (err) => {
    console.error('something wrong occurred: ' + err);
  },
  complete: () => {
    console.log('done');
  },
});

console.log('just after subscribe');

// Logs:
// just before subscribe
// got value 1           // subscribe によって出力されたログ
// got value 2           // 同上
// got value 3           // 同上
// something wrong occurred: happening! // エラー処理でログが出力された
// just after subscribe
```

`next(3)` の後に `subscriber.error('happening!');` を挟んだ例です。
エラー処理によって処理が中断され、 `just after subscribe` の後に実行されるはずだった遅延処理が実行されませんでした。

## complete が通知されたときの動きを確認する

```typescript
import { Observable } from 'rxjs';

const observer = new Observable((subscriber) => {
  subscriber.next(1);
  subscriber.next(2);

  // 配信処理を終了させて後続の動きを確認する
  subscriber.complete();

  // next(2) の後に `complete` で配信が終了しているので、next(3) と 遅延処理部分のログは出力されない
  subscriber.next(3);
 
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});

console.log('just before subscribe');

observer.subscribe({
  next: (x) => {
    console.log('got value ' + x);
  },
  error: (err) => {
    console.error('something wrong occurred: ' + err);
  },
  complete: () => {
    console.log('done');
  },
});

console.log('just after subscribe');

// Logs:
// just before subscribe
// got value 1           // subscribe によって出力されたログ
// got value 2           // 同上
// done                  // complete によって出力されたログ
// just after subscribe
```

`next(2)` の後に `subscriber.complete();` を挟んだ例です。
`complete` が途中で実行されたことで、配信が終了となりました。
結果、`just after subscribe` の後に実行されるはずだった遅延処理が実行されませんでした。

# まとめにかえて

`Obseravable` と `Observer` について見ました。
記事中に何度も出てきましたが

- `Observer` は **消費者** として `Observable` から配信されてきたストリームを購読し、ストリーム中のデータを処理する
- `Observable` は **配信者** としてストリームを配信する

ということがサンプルコードからも理解できたと思います。
またサンプルコードが

```typescript
// `observer` は 消費者
// `new Observable((subscriber)` の subscriber は配信者
const observer = new Observable((subscriber) => {
  // 配信のための処理を記述する
});

// 消費者として配信されたストリームを購読してアレコレする処理を記述する
observer.subscribe({
  next: (x) => {
    // 通常時の処理
  },
  error: (err) => {
    // エラー処理
  },
  complete: () => {
    // 完了時の処理
  },
});
```

という実装であることからも、 **消費者** と **配信者** は別々に独立したものであり、それぞれの役割も異なるものである、ということが分かります。

( 蛇足ながら...)
`observer` には `next()` が存在しないため、`observer` からストリームを配信することはできませんし、
コールバックの引数である配信者たる `subscriber` には `subscribe()` が存在しないため、配信されたストリームを購読することもできません。

ですが、RxJS には [Subject](https://rxjs.dev/guide/subject) というものがあり、 `Subject` は `Observable` と `Observer` 両方の性質を持ちます。
すなわち、`Observable` と `Observer` は `Subject` によって代替できます。
次はこの [Subject](https://rxjs.dev/guide/subject) について触れたいと思います。

# 参考

- [RxJS における Subject の活用]( https://zenn.dev/mikakane/articles/rxjs_5_subject)
- [RxJS を学ぼう #5 - Subject について学ぶ / Observable × Observer](https://blog.recruit.co.jp/rmp/front-end/post-11951/)
