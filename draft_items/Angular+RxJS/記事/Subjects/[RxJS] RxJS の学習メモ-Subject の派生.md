# はじめに

本記事は [[RxJS] RxJS の学習メモ-Observable と Observer](https://qiita.com/ksh-fthr/items/2933492929bbeccece50) の続きです。
RxJS を使っていく上で行った学習の備忘録になります。主に次に挙げた内容のブラッシュアップを狙いました。

:::note info

- 知らないことによる忌避感をなくす
  - RxJS を使った実装は個人的に初見殺しもいいとこな実装だと思っている
  - 知ることで「あ、別に怖がることないじゃん」という感じに持っていきたい
- 知見の向上、また思い込みや間違った理解の是正を狙う
  - ライブラリを改めてみることでより良い実装の方法、テクニックを得る
  - 正しい、最適だと思っていたものが実は間違っていたことも充分あり得るのでその辺が是正できれば御の字

:::

# 環境

本記事について扱うライブラリや環境の情報です。

|                                         | 備考                                                        |
| --------------------------------------- | ----------------------------------------------------------- |
| [RxJS](https://rxjs.dev/)               | 公式                                                        |
| [Learn RxJS](https://www.learnrxjs.io/) | リファレンス的な感じの学習サイト                            |
| [StackBlitz](https://stackblitz.com/)   | RxJS だけでなく Angular とか React とかの実装お試しができる |

なお本記事執筆時の RxJS のバージョンは StackBlitz の DEPENDENCIES を見ると `v7.8.0` でした。

# この記事でやること

## `Subject` について触れてみる

本記事では `Subject` について触れます。
`Subject` は RxJS の基本コンセプトである 6つ のコンセプトに含まれるものです。

## RxJS の基本コンセプト

RxJS の基本コンセプトは `Observables`, `Observer`, `Subscription`, `Operators`, `Subjects`, `Schedulers` の 6つ があります。
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

# [Subject](https://rxjs.dev/guide/subject)

前回の記事: [[RxJS] RxJS の学習メモ-Observable と Observer](https://qiita.com/ksh-fthr/items/2933492929bbeccece50) で `Observer` と `Observable` を見ました。
が、筆者の経験上( あまり経験豊富とは言えませんが )、 これらはあまり使ったり見たりした記憶がありません。
殆どの場合 [Subject](https://rxjs.dev/guide/subject#subject) だったり、その派生である [BehaviorSubject](https://rxjs.dev/guide/subject#behaviorsubject) を使ってます。

そして、`Subject` とは何かをまとめると下記になります。

:::note info

**`Subject` とはなにか**

- `Subject` ≒ `Observable`
- `Subject` は **マルチキャスト** でストリームを流す
- ( `Observable` はユニキャスト  でストリームを流す )
- `Subject` は `Observable` と `Observer` の両方の性質をもつ

※ 補足: マルチキャスト・ユニキャストとは以下を意味します

**マルチキャストとユニキャスト**

- マルチキャスト
  - 購読している `Observer` が一つの `Observable` 実行を共有する
- ユニキャスト
  - 購読している `Observer` がそれぞれ `Observable` の独立した実行を所有する

:::

次の項目では **`Subject` ≒ `Observable`** と **`Subject` は `Observable` と `Observer` の両方の性質をもつ** が示すところを見ていきます。

## Subject の動きをサンプルコードで確認する

以下のサンプルコードは [前回記事のサンプルコード](https://qiita.com/ksh-fthr/items/2933492929bbeccece50#%E3%82%B5%E3%83%B3%E3%83%97%E3%83%AB%E3%82%B3%E3%83%BC%E3%83%89) `Subject` に置き換えたものです。
このコードで **消費者としての Observer** と **配信者としての Observable** の両方の役割を **Subject** が担っていることが確認できます。

```typescript
import { Subject } from 'rxjs';

// (1) 最初のブロック
// (1-1) 消費者であり配信者でもある `subject`
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

## 詳しく見ていく

### 最初のブロック

```typescript
// (1) 最初のブロック
// (1-1) 消費者であり配信者でもある `subject`
const subject = new Subject();
```

**(1-1) 消費者であり配信者でもある `subject`**
コメントにあるように **消費者であり配信者でもある `subject`** のインスタンスを生成しています。
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
**`Observable` は **subscribe** されることでストリームが購読される** のは `subject` でも同じです。
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

`subject` に流れてくるストリームを購読する準備として、ここで `subscribe` を実行します。

**補足:**
`next` のあとに `subscribe` をしても、その `next` で流したストリームは購読できません。
これは `subscribe` による購読準備を行う前にストリームが流れてしまっているからです。

`subscribe` による購読準備は `next` によるストリームが流れる前に行っておく必要があります。

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
後述の出力結果にあるとおり、ここで遅延実行された `next(5)` は 五番目のブロック のログ出力後に配信され、`subscribe` で処理されます。

### 五番目のブロック

```typescript
// (5) 五番目のブロック
console.log('just after subscribe');
```

**(5) 五番目のブロック**
[二番目のブロック](#二番目のブロック) と同じく、`subscribe` が実行された後に出るログ、ということで項目を分けました。
このログがあることで、[四番目のブロック](#四番目のブロック) で示した遅延実行の `next(4)` の実行結果が分かりやすくなっています。

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

# まとめにかえて

以上、 `Subject` が `Observer` と `Observable` の代替となることをサンプルコードで見てきました。

なお本記事ではベーシックな `Subject` について触れましたが、`Subject` にはそのの派生となる `BehaviorSubject`, `ReplaySubject`, `AsyncSubject` があります。
次の記事ではそれらの派生について触れたいと思います。


# [**BehaviorSubject**](https://rxjs.dev/guide/subject#behaviorsubject)

`BehaviorSubject` についての説明を上記リンクから転載します。

:::note info

> One of the variants of Subjects is the BehaviorSubject, which has a notion of "the current value". It stores the latest value emitted to its consumers, and whenever a new Observer subscribes, it will immediately receive the "current value" from the BehaviorSubject.
>
>> BehaviorSubjects are useful for representing "values over time". For instance, an event stream of birthdays is a Subject, but the stream of a person's age would be a BehaviorSubject.
>
> ( Deepl による翻訳 )
> Subjectsのバリエーションの1つにBehaviorSubjectがあり、これは「現在の値」という概念を持っています。これは、コンシューマーに発行された最新の値を保存し、新しいObserverが購読するたびに、すぐにBehaviorSubjectから「現在の値」を受け取ることができます。  
>
>> BehaviorSubjectは「時間の経過に伴う値」を表現するのに便利です。例えば、誕生日のイベントストリームはSubjectですが、人の年齢のストリームはBehaviorSubjectとなります。

:::

この説明で大事なのは次の部分です

:::note info

> **発行された最新の値を保存し、新しいObserverが購読するたびに、すぐにBehaviorSubjectから「現在の値」を受け取る**

:::

この一文が示す意味をサンプルコードで確認します。

## BehaviorSubject の動きをサンプルコードで確認する

```typescript
import { BehaviorSubject } from 'rxjs';

// (1) 最初のブロック
const subject = new BehaviorSubject(0); // 0 is the initial value

// (2) 二番目のブロック
// この時点で subject を初期化した値である `0` が出力される
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});

// (3) 三番目のブロック
// ここでは `1` が出力される
subject.next(1);
// ここで `2` が ↑ の subscribe と ↓ の subscribe で出力される
// `1` のストリームはこの `2` のストリームで上書きされるので出力されない
subject.next(2);

// (4) 四番目のブロック
// 直前に流れた `2` が出力される
// また ↓ の `3` が実行されたら それも出力される
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

// (5) 五番目のブロック
// 最初の subscribe と 2つめの subscribe で `3` が出力される
subject.next(3);
```

## 詳しく見ていく

### 最初のブロック

```typescript
// (1) 最初のブロック
const subject = new BehaviorSubject(0); // 0 is the initial value
```

### 二番目のブロック

```typescript
// (2) 二番目のブロック
// この時点で subject を初期化した値である `0` が出力される
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});
```

### 三番目のブロック

```typescript
// (3) 三番目のブロック
// ここでは `1` が出力される
subject.next(1);
// ここで `2` が ↑ の subscribe と ↓ の subscribe で出力される
// `1` のストリームはこの `2` のストリームで上書きされるので出力されない
subject.next(2);
```

### 四番目のブロック

```typescript
// (4) 四番目のブロック
// 直前に流れた `2` が出力される
// また ↓ の `3` が実行されたら それも出力される
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});
```

### 五番目のブロック

```typescript
// (5) 五番目のブロック
// 最初の subscribe と 2つめの subscribe で `3` が出力される
subject.next(3);
```

## 実行結果

このコードの実行結果は次のとおりです。

```log
// Logs
// observerA: 0 // 最初の注目点
// observerA: 1
// observerA: 2
// observerB: 2 // 2つ目の注目点
// observerA: 3
// observerB: 3
```

**最初の注目点**
最初に注目したいのは `observerA: 0` の出力です。
[subject の 三番目のブロック](#三番目のブロック) の補足では次のように記載しました。

>
> **補足:**
> `next` のあとに `subscribe` をしても、その `next` で流したストリームは購読できません。
> これは `subscribe` による購読準備を行う前にストリームが流れてしまっているからです。
>
> `subscribe` による購読準備は `next` によるストリームが流れる前に行っておく必要があります。

これに対して、`BehaviorSubject` を使ったこのサンプルコードでは

> ```typescript
> const subject = new BehaviorSubject(0); // 0 is the initial value
> 
> // この時点で subject を初期化した値である `0` が出力される
> subject.subscribe({
>   next: (v) => console.log(`observerA: ${v}`),
> });
> ```

と、 `subscribe` の前に変数宣言と同時に生成している `new BehaviorSubject(0)` で指定された `0` が購読されています。

**二番目の注目点**
次に注目したいのは `observerB: 2` の出力です。
この出力はコード中の

```typescript
// (4) 四番目のブロック
// 直前に流れた `2` が出力される
// また ↓ の `3` が実行されたら それも出力される
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});
```

で処理されたものですが、出力された値が `observerA: 2` と同じ値であることがポイントです。
コメントにも記載してありますように、直前の `subject.next(2)` で流れた値が購読されていることが分かります。

**冒頭で注目した内容を振り返る**
ここで上に挙げた 2つ のポイントから、本項目の冒頭で注目した

:::note info

> **発行された最新の値を保存し、新しいObserverが購読するたびに、すぐにBehaviorSubjectから「現在の値」を受け取る**

:::

の示す動きを振り返りますと、次の 3点 にまとめられます。

- `new BehaviorSubject(0)` と `subject.next(2)` の部分で `subject` には `0` と `2` がストリームとして保持されている
- 最初の `subject.subscribe()` では `new BehaviorSubject(0)` で保持されている `0` を購読して `observerA: 0` が出力された
- 2つ目の `subject.subscribe()` では `subject.next(2)` で保持されている `2` を購読して `observerB:2` が出力された

以上、サンプルコードから `BehaviorSubject` の動きが理解できました。

# [ReplaySubject](https://rxjs.dev/guide/subject#replaysubject "Link to this heading")

`ReplaySubject` についての説明を上記リンクから転載します。

:::note info

> A ReplaySubject is similar to a BehaviorSubject in that it can send old values to new subscribers, but it can also _record_ a part of the Observable execution.
>
>> A ReplaySubject records multiple values from the Observable execution and replays them to new subscribers.
( Deepl による翻訳 )
> ReplaySubjectは、古い値を新しい購読者に送ることができるという点ではBehaviorSubjectと似ていますが、Observableの実行の一部を記録することもできます。  
>
>> ReplaySubjectは、Observableの実行から複数の値を記録し、新しいSubscriberに再生することができます。

:::

次のサンプルコードで挙動を確認します。

```typescript
import { ReplaySubject } from 'rxjs';

// 3回分 の繰り返し用バッファを用意
const subject = new ReplaySubject(3); // buffer 3 values for new subscribers

// ストリームが流れる前の購読ではバッファ有無に関係なく 後述の next(1)~next(4) まで流れる
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});

subject.next(1);
subject.next(2);
subject.next(3);
subject.next(4);

// ストリームが流れたあとの購読では next(2)~next(3) の 3回分 流れる
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

// 購読が終わったあとにストリームを流す
// バッファは 3回分 用意しているが、ここでは 1回 しかストリームが流れないのでログも next(5) だけが流れる
subject.next(5);

// Logs:
// observerA: 1
// observerA: 2
// observerA: 3
// observerA: 4
// observerB: 2
// observerB: 3
// observerB: 4
// observerA: 5
// observerB: 5
```

# [AsyncSubject](https://rxjs.dev/guide/subject#asyncsubject "Link to this heading")

`AsyncSubject` についての説明を上記リンクから転載します。

:::note info

> The AsyncSubject is a variant where only the last value of the Observable execution is sent to its observers, and only when the execution completes.
>
>( Deepl による翻訳 )
> AsyncSubjectは、Observableの実行の最後の値だけが、実行が完了したときだけ、そのオブザーバーに送信される変種である。

:::

サンプルコードで挙動を確認します。

```typescript
import { AsyncSubject } from 'rxjs';
const subject = new AsyncSubject();

// AsyncSubject では最後に発信されたストリームだけが流れてくるので `next(5)` の値だけが出力される
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});

subject.next(1); // ストリームが流れない
subject.next(2); // 同上
subject.next(3); // 同上
subject.next(4); // 同上

// 最初の購読と同じく、こちらも `next(5)` の値だけが出力される
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

subject.next(5);

// ここをコメントアウトすると subscribe にストリームが流れない
subject.complete();

// Logs:
// observerA: 5
// observerB: 5
```

※ 補足
重要なポイントが  `complete()` の存在です。
`AsyncSubject` では `subscribe` にストリームが流れるために `complete` による通知が必要です。
上記コードで `subject.complete()` をコメントアウトすると `subscribe` にストリームが流れません。

# 参考

- [RxJS における Subject の活用]( https://zenn.dev/mikakane/articles/rxjs_5_subject)
- [RxJS を学ぼう #5 - Subject について学ぶ / Observable × Observer](https://blog.recruit.co.jp/rmp/front-end/post-11951/)
