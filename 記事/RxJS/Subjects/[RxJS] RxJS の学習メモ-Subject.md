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

今回は [Subject](https://rxjs.dev/guide/subject) について学びます。

# 環境

本記事について扱うライブラリや環境の情報です。

|                                        | 備考                                                        |
| -------------------------------------- | ----------------------------------------------------------- |
| [RxJS](https://rxjs.dev/)              | 公式                                                        |
| [Learn RxJS](https://www.learnrxjs.io/) | リファレンス的な感じの学習サイト                        |
| [StackBlitz](https://stackblitz.com/)  | RxJS だけでなく Angular とか React とかの実装お試しができる |

なお本記事執筆時の RxJS のバージョンは StackBlitz の DEPENDENCIES を見ると `v7.8.0` でした。

# この記事でやること

## [Subject](https://rxjs.dev/guide/subject) について触れてみる

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

前置きが長くなりましたが、これから実際に `Subject` を見ていきます。

# [Subject](https://rxjs.dev/guide/subject)

とはいうものの、 [`Subject`](https://rxjs.dev/guide/subject) のページをみますと `Subject` と、その派生である `BehaviorSubject`, `ReplaySubject`, `AsyncSubject` が紹介されています。
それらをすべて扱うのはかなり重たいので、本記事ではベースとなる [`Subject`](https://rxjs.dev/guide/subject#subject) を扱おうと思います。
そして `Subject` とは何かをまとめると下記になります。

:::note info

**`Subject` とはなにか**

- `Subject` ≒ `Observable`
- `Subject` は `Observable` と `Observer` の両方の性質をもつ
- `Subject` は **マルチキャスト** でストリームを流す
- ( `Observable` はユニキャスト  でストリームを流す )

※ 補足: マルチキャスト・ユニキャストとは以下を意味します

**マルチキャストとユニキャスト**

- マルチキャスト
  - 購読している `Observer` が一つの `Observable` 実行を共有する
- ユニキャスト
  - 購読している `Observer` がそれぞれ `Observable` の独立した実行を所有する

:::

以降の項目で `Subject` の動きを見ていきます。

## Observer と Observable 両方の性質をもつことを確認する

このサンプルコードでは次の 2点 を確認します。

> - `Subject` ≒ `Observable`
> - `Subject` は `Observable` と `Observer` の両方の性質をもつ

以下のサンプルコードは [前回記事のサンプルコード](https://qiita.com/ksh-fthr/items/2933492929bbeccece50#%E3%82%B5%E3%83%B3%E3%83%97%E3%83%AB%E3%82%B3%E3%83%BC%E3%83%89) `Subject` に置き換えたものです。
このコードで **消費者としての Observer** と **配信者としての Observable** の両方の役割を **Subject** が担っていることが確認できます。

```typescript
import { Subject } from 'rxjs';

// (1) 最初のブロック
// (1-1) 消費者であり配信者でもある `Subject`
const subject = new Subject();

// (2) 二番目のブロック
// (2-1) `subscribe` の実行前であることを示す
console.log('just before subscribe');

// (3) 三番目のブロック
// (3-1) `subscribe` によるストリームの購読を行う
subject.subscribe({
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

// (4) 四番目のブロック
// (4-1) `next` でストリームを配信する
subject.next(1);
subject.next(2);
subject.next(3);

// (4-2) `next` によるストリーム配信を遅延実行する
setTimeout(() => {
  subject.next(4);
  subject.complete();
}, 1000);

// (5) 五番目のブロック
console.log('just after subscribe');
```

### 最初のブロック

```typescript
// (1) 最初のブロック
// (1-1) 消費者であり配信者でもある `Subject`
const subject = new Subject();
```

**(1-1) 消費者であり配信者でもある `Subject`**
コメントにあるように **消費者であり配信者でもある `Subject`** のインスタンスを生成しています。
これで

- ストリームを配信する準備
- ストリームを購読する準備

が出来ました。
以降の処理は変数 `subject` に対して `next()` による配信と `subscribe()` による購読を行います。

### 二番目のブロック

```typescript
// (2) 二番目のブロック
// (2-1) `subscribe` の実行前であることを示す
console.log('just before subscribe');
```

**(2-1) `subscribe` の実行前であることを示す**
このブロックは `subscribe` の前に実行されるログ、ということで項目を分けました。
**`subscribe` されることでストリームが購読される** のは `Subject` でも同じです。
従いまして、サンプルコードの処理ではまず最初にこのログが出力されます。

### 三番目のブロック

```typescript
// (3) 三番目のブロック
// (3-1) `subscribe` によるストリームの購読を行う
subject.subscribe({
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
[前項](#二番目のブロック) で触れていますとおり、ストリームは `subscribe` をして初めて配信されてきます。
逆に言えば `subscribe` をしていない限りストリームは流れてこない、ということです。

そういうわけで `subject` に流れてくるストリームを購読する準備として、ここで `subscribe` を実行します。

**補足:**
`next` のあとに `subscribe` をしても、`next` で流したストリームは購読できません。
これは `subscribe` による購読準備を行う前にストリームが流れてしまっているからです。

**`subscribe` による購読準備は `next` によるストリームが流れる前に行っておく** 必要があります。

### 四番目のブロック

```typescript
// (4) 四番目のブロック
// (4-1) `next` でストリームを配信する
subject.next(1);
subject.next(2);
subject.next(3);

// (4-2) `next` によるストリーム配信を遅延実行する
setTimeout(() => {
  subject.next(4);
  subject.complete();
}, 1000);
```

**(4-1) `next` でストリームを配信する**
ストリームの配信です。ここの `next(1)~next(3)` は同期的に順番にストリームが流れます。

**(4-2) `next` によるストリーム配信を遅延実行する**
同じくストリームの配信ですが、ここでは `setTimeout` により遅延実行しています。
後述の出力結果にあるとおり、ここで遅延実行された `next(4)` は 五番目のブロック のログ出力後に配信され、`subscribe` で処理されます。

### 五番目のブロック

```typescript
// (5) 五番目のブロック
console.log('just after subscribe');
```

**(5) 五番目のブロック**
[二番目のブロック](#二番目のブロック) と同じく、`subscribe` が実行された後に出るログ、ということで項目を分けました。
このログがあることで、[四番目のブロック](#四番目のブロック) で示した遅延実行の `next(4)` の実行結果が分かりやすくなっています。

### 実行結果

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

[前回記事( [RxJS] RxJS の学習メモ-Observable と Observer )の実行結果](https://qiita.com/ksh-fthr/items/2933492929bbeccece50#%E5%AE%9F%E8%A1%8C%E7%B5%90%E6%9E%9C) と同じ出力結果となりました。
以上、下記について確認できました。

> - `Subject` ≒ `Observable`
> - `Subject` は `Observable` と `Observer` の両方の性質をもつ

## マルチキャストとユニキャストの動きを確認する

では次に、下記についてサンプルコードを交えて確認していきます。

> - `Subject` は **マルチキャスト** でストリームを流す
> - ( `Observable` はユニキャスト  でストリームを流す )
>
> ※ 補足: マルチキャスト・ユニキャストとは以下を意味します
>
> **マルチキャストとユニキャスト**
>
> - マルチキャスト
>   - 購読している `Observer` が一つの `Observable` 実行を共有する
> - ユニキャスト
>   - 購読している `Observer` がそれぞれ `Observable` の独立した実行を所有する

### マルチキャストの動きを確認する( `Subject` )

まずは **マルチキャスト** である `Subject` のサンプルコードです。

```typescript
import { Subject } from 'rxjs';
const subject = new Subject();

subject.subscribe({
  next: (x) => {
    console.log('got value-A ' + x);
  },
});

subject.subscribe({
  next: (x) => {
    console.log('got value-B ' + x);
  },
});

subject.next(1);
subject.next(2);
subject.next(3);

// Logs: 最初と次の subscribe で出力したログが交互にでている
// 
// 一回目の next(1)
// got value-A 1
// got value-B 1
// 二回目の next(2)
// got value-A 2
// got value-B 2
// 三回目の next(3)
// got value-A 3
// got value-B 3
```

**一回のストリーム配信ごとにそれぞれの `subscirbe` で個別に処理されている** こと、つまり

> **購読している `Observer` が一つの `Observable` 実行を共有する**

ということがログから分かります。

### ユニキャストの動きを確認する( `Observer` )

次に **ユニキャスト** である `Observer` のサンプルコードです。

```typescript
import { Observable } from 'rxjs';

const observer = new Observable((subscriber) => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
});

observer.subscribe({
  // (3-2) ストリームに対して next, error, complete でそれぞれのイベントを処理する
  next: (x) => {
    console.log('got value-A: ' + x);
  },
});

observer.subscribe({
  next: (x) => {
    console.log('got value-B: ' + x);
  },
})

// Logs: 最初の subscribe のログが出た後に次の subscribe のログが出ている
// got value-A: 1
// got value-A: 2
// got value-A: 3
// got value-B: 1
// got value-B: 2
// got value-B: 3
```

**それぞれの `subscirbe` でストリーム配信をまとめて処理している** こと、つまり

> **購読している `Observer` がそれぞれ `Observable` の独立した実行を所有する**

ということがログから分かります。

# まとめにかえて

以上、 `Subject` が `Observer` と `Observable` の代替となることをサンプルコードで見てきました。
なお本記事ではベーシックな `Subject` について触れましたが、冒頭申し上げたとおり、 `Subject` にはその派生となる `BehaviorSubject`, `ReplaySubject`, `AsyncSubject` があります。

- [Subject](https://rxjs.dev/guide/subject#subject) の派生
  - [BehaviorSubject](https://rxjs.dev/guide/subject#behaviorsubject)
  - [ReplaySubject](https://rxjs.dev/guide/subject#replaysubject)
  - [AsyncSubject](https://rxjs.dev/guide/subject#asyncsubject)

次の記事ではそれらの派生について触れたいと思います。

# 参考

- [RxJS における Subject の活用]( https://zenn.dev/mikakane/articles/rxjs_5_subject)
- [RxJS を学ぼう #5 - Subject について学ぶ / Observable × Observer](https://blog.recruit.co.jp/rmp/front-end/post-11951/)
