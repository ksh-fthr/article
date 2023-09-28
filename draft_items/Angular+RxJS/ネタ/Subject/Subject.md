# サンプルコード

## [Subject](https://rxjs.dev/guide/subject)

ここまでで `Observer` と `Observable` を見てきましたが、( あまり経験豊富とは言えませんが ) これらはあまり使った記憶がありません。
殆どが  `Subject` だったり、その派生である `BehaviorSubject` を使ってます。

そして、`Subject` とは何かをまとめると下記になります。

- `Subject` ≒ `Observable`
- `Subject` は **マルチキャスト** でストリームを流す
- ( `Observable` はユニキャスト  でストリームを流す )
- `Subject` は `Observable` と `Observer` の両方の性質をもつ

## 補足

- マルチキャスト
  - 購読している `Observer` が一つの `Observable` 実行を共有する
- ユニキャスト
  - 購読している `Observer` がそれぞれ `Observable` の独立した実行を所有する

## Observer としての Subject

```typescript
import { Subject } from 'rxjs';

const subject = new Subject<number>();

subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

subject.next(1);
subject.next(2);

// Logs:
// observerA: 1
// observerB: 1
// observerA: 2
// observerB: 2
```

## Observable としての Subject

```typescript
import { Subject, from } from 'rxjs';

const subject = new Subject<number>();

subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

const observable = from([1, 2, 3]);

// observable => subject にストリームが流れて subject の subscribe で購読される
observable.subscribe(subject); // You can subscribe providing a Subject

// Logs:
// observerA: 1
// observerB: 1
// observerA: 2
// observerB: 2
// observerA: 3
// observerB: 3
```

ここまでがベーシックな Subject の話。以下は Subject の派生となる `BehaviorSubject`, `ReplaySubject`, `AsyncSubject` について触れます。

## Subject の派生

### [**BehaviorSubject**](https://rxjs.dev/guide/subject#behaviorsubject)

( 上記リンクから転載 )
> One of the variants of Subjects is the BehaviorSubject, which has a notion of "the current value". It stores the latest value emitted to its consumers, and whenever a new Observer subscribes, it will immediately receive the "current value" from the BehaviorSubject.
>
>> BehaviorSubjects are useful for representing "values over time". For instance, an event stream of birthdays is a Subject, but the stream of a person's age would be a BehaviorSubject.

( Deepl による翻訳 )
> Subjectsのバリエーションの1つにBehaviorSubjectがあり、これは「現在の値」という概念を持っています。これは、コンシューマーに発行された最新の値を保存し、新しいObserverが購読するたびに、すぐにBehaviorSubjectから「現在の値」を受け取ることができます。  
> 
>> BehaviorSubjectは「時間の経過に伴う値」を表現するのに便利です。例えば、誕生日のイベントストリームはSubjectですが、人の年齢のストリームはBehaviorSubjectとなります。

この説明で大事なのは次の部分です

> **発行された最新の値を保存し、新しいObserverが購読するたびに、すぐにBehaviorSubjectから「現在の値」を受け取る**

次は上記を確認するサンプルコードです。

```typescript
import { BehaviorSubject } from 'rxjs';
const subject = new BehaviorSubject(0); // 0 is the initial value

// この時点で subject を初期化した値である `0` が出力される
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});

// ここでは `1` が出力される
subject.next(1);
// ここで `2` が ↑ の subscribe と ↓ の subscribe で出力される
// `1` のストリームはこの `2` のストリームで上書きされるので出力されない
subject.next(2);

// 直前に流れた `2` が出力される
// また ↓ の `3` が実行されたら それも出力される
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

// 最初の subscribe と 2つめの subscribe で `3` が出力される
subject.next(3);

// Logs
// observerA: 0
// observerA: 1
// observerA: 2
// observerB: 2
// observerA: 3
// observerB: 3
```

### [ReplaySubject](https://rxjs.dev/guide/subject#replaysubject "Link to this heading")

( 上記リンクから転載 )
> A ReplaySubject is similar to a BehaviorSubject in that it can send old values to new subscribers, but it can also _record_ a part of the Observable execution.
>
>> A ReplaySubject records multiple values from the Observable execution and replays them to new subscribers.
>
( Deepl による翻訳 )
> ReplaySubjectは、古い値を新しい購読者に送ることができるという点ではBehaviorSubjectと似ていますが、Observableの実行の一部を記録することもできます。  
 > 
>> ReplaySubjectは、Observableの実行から複数の値を記録し、新しいSubscriberに再生することができます。

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

### [AsyncSubject](https://rxjs.dev/guide/subject#asyncsubject "Link to this heading")

( 上記リンクから転載 )

> The AsyncSubject is a variant where only the last value of the Observable execution is sent to its observers, and only when the execution completes.

( Deepl による翻訳 )

> AsyncSubjectは、Observableの実行の最後の値だけが、実行が完了したときだけ、そのオブザーバーに送信される変種である。

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


///////////// 
使うかわからないが Observer, Observable のサンプルコードを subject で書き換えたもの

```typescript
console.log('just before subscribe');

const subject = new Subject();

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


subject.next(1);
subject.next(2);
subject.next(3);

  
setTimeout(() => {
  subject.next(4);
  subject.complete();
}, 1000);


console.log('just after subscribe');

// Logs:
// just before subscribe
// got value 1
// got value 2
// got value 3
// just after subscribe
// got value 4
// done
```