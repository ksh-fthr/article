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

前回に引き続き、今回も [Subject](https://rxjs.dev/guide/subject) について学びます。

# 環境

本記事について扱うライブラリや環境の情報です。

|                                         | 備考                                                        |
| --------------------------------------- | ----------------------------------------------------------- |
| [RxJS](https://rxjs.dev/)               | 公式                                                        |
| [Learn RxJS](https://www.learnrxjs.io/) | リファレンス的な感じの学習サイト                            |
| [StackBlitz](https://stackblitz.com/)   | RxJS だけでなく Angular とか React とかの実装お試しができる |

なお本記事執筆時の RxJS のバージョンは StackBlitz の DEPENDENCIES を見ると `v7.8.0` でした。

# この記事でやること

## Subject の派生について触れてみる

[`Subject`](https://rxjs.dev/guide/subject#subject) の派生である [`BehaviorSubject`](https://rxjs.dev/guide/subject#behaviorsubject), [`ReplaySubject`](https://rxjs.dev/guide/subject#replaysubject), [`AsyncSubject`](https://rxjs.dev/guide/subject#asyncsubject) について触れたいと思います。

冒頭でも触れておりますが、 `Subject` については以下の記事で扱っております。ご興味あればご参照ください。

- [[RxJS] RxJS の学習メモ-Subject](https://qiita.com/ksh-fthr/items/54b19b4160505e2fddd9)

# [BehaviorSubject](https://rxjs.dev/guide/subject#behaviorsubject)

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

この説明で大事なのは次の部分です。

:::note info

> **発行された最新の値を保存し、新しいObserverが購読するたびに、すぐにBehaviorSubjectから「現在の値」を受け取る**

:::

この一文が示す意味をサンプルコードで確認します。

## BehaviorSubject の動きを確認する

```typescript
import { BehaviorSubject } from 'rxjs';

// (1) 最初のブロック
// 初期値として 0 を設定, ここで設定した `0` はこの直後の `subscribe` で購読される
const subject = new BehaviorSubject(0);

// (2) 二番目のブロック
// この時点で subject を初期化した値である `0` が出力される
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});

// (3) 三番目のブロック
// `1` を `subject` のストリームに流す.
// この `1` は ↑ の subscribe で購読される
subject.next(1);
// `2` を `subject` のストリームに流す.
// この `2` は ↑ の subscribe と ↓ の subscribe で出力される
// 
// なお、↓ の subscribe において `1` のストリームは
// この `2` のストリームで上書きされるので出力されない
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

### 各ブロックの説明

要点を実行結果のログで見ていきたいので、各ブロックの説明はコード中のコメントをご確認ください。

### 実行結果

このコードの実行結果は次のとおりです。

```log
// Logs
observerA: 0 // 最初の注目点
observerA: 1
observerA: 2
observerB: 2 // 2つ目の注目点
observerA: 3
observerB: 3
```

**最初の注目点**
最初に注目したいのは `observerA: 0` の出力です。
前回記事 [[RxJS] RxJS の学習メモ-Subject の 三番目のブロック](https://qiita.com/ksh-fthr/items/54b19b4160505e2fddd9#%E4%B8%89%E7%95%AA%E7%9B%AE%E3%81%AE%E3%83%96%E3%83%AD%E3%83%83%E3%82%AF) の補足では次のように記載しました。

>
> **補足:**
> `next` のあとに `subscribe` をしても、その `next` で流したストリームは購読できません。
> これは `subscribe` による購読準備を行う前にストリームが流れてしまっているからです。
>
> `subscribe` による購読準備は `next` によるストリームが流れる前に行っておく必要があります。

これに対して、`BehaviorSubject` を使ったこのサンプルコードでは

```typescript
// (1) 最初のブロック
// 初期値として 0 を設定, ここで設定した `0` はこの直後の `subscribe` で購読される
const subject = new BehaviorSubject(0);

// (2) 二番目のブロック
// この時点で subject を初期化した値である `0` が出力される
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});
```

と、 `subscribe` の前に変数宣言と同時に生成している `new BehaviorSubject(0)` で指定された `0` が購読されています。
この違いが大きなポイントです。
すなわち **BehaviorSubject で配信された「現在の値」を受け取る** ことがこのコードから確認できました。

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

最初の注目点とあわせて **BehaviorSubject で配信された「現在の値」を受け取る** ことを確認できるコードです。

**冒頭で注目した内容を振り返る**
以上、サンプルコードから `BehaviorSubject` の動きを見てきました。
ここでもう一度本項目の冒頭で注目した部分を挙げておきます。

:::note info

> **発行された最新の値を保存し、新しいObserverが購読するたびに、すぐにBehaviorSubjectから「現在の値」を受け取る**

:::

`BehaviorSubject` を使うことで **直前に配信された値** イコール **現在の値** を、**ストリーム配信前に `subscribe` しておかなくとも購読できる** ことが分かりました。

# [ReplaySubject](https://rxjs.dev/guide/subject#replaysubject)

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

## ReplaySubject の動きを確認する

```typescript
import { ReplaySubject } from 'rxjs';

// (1) 最初のブロック
// 3回分 の繰り返し用バッファを用意
// subscribe 前に配信されたストリームは ここで用意したバッファ分 保持され、subscribe で購読される
const subject = new ReplaySubject(3);

// (2) 二番目のブロック
// subscribe 前に 4回 ストリームを配信して動きを確認する
// 用意したバッファは 3つ なので購読されるのは next(2)~next(3) の 3つ
// next(1) はバッファからあぶれるので購読されない
subject.next(1); // 購読されない
subject.next(2); // 購読される
subject.next(3); // 購読される
subject.next(4); // 購読される

// (3) 三番目のブロック
// subscribe 前のストリームは 3回分 購読する
// subscribe 後のストリームはバッファの回数に関係なく新しい subscribe が用意されるまで延々と購読する
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});
console.log('---');

// (4) 四番目のブロック
// 4回、新しくストリームを流す. このとき observerA と observerB で購読するものが異なる
// observerA
//   5, 6, 7, 8 と購読している. これは observerA の subscribe 後に配信されたストリームであることが理由
//   つまり observerA では subscribe 後に配信されたストリームはバッファの有無に関係なく順次購読している
// observerB
//   6, 7, 8 と購読している. これは observerB の subscribe 前に配信されたストリームであることが理由
//   つまり 最初のブロックで指定した 3回分のバッファ が効いている.
//   直前の 3回分 のストリームがバッファとして残っていて、それを購読しているのが observerB の挙動となる
subject.next(5);
subject.next(6);
subject.next(7);
subject.next(8);
console.log('---');

// (5) 五番目のブロック
// ストリームが流れた後の購読では next(6), next(7), next(8) の 3回分 流れる
// ( 繰り返しになるが、3回分のバッファをここで購読している )
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});
console.log('---');

// (6) 六番目のブロック
// subscribe 後のストリーム配信の動きを確認する.
// ここ以降はストリームが配信されるたびに observerA, observerB で購読される.
// 今回は subject.next(9) の 1回 しかストリームが流れないのでログも next(9) だけが流れるが、next(10), nextt(11)... とすれば
// その分 observerA, observerB にストリームが配信され購読される.
// ( これは新しく subscribe が行われるまで続く )
subject.next(9);

// このコメントアウトを外すと obeserverA, observerB には next(9)~next(14) の配信が購読される
// そして observerC には next(12), next(13), next(14) が購読される
// subject.next(10);
// subject.next(11);
// subject.next(12);
// subject.next(13);
// subject.next(14);
// console.log('---');

// subject.subscribe({
//   next: (v) => console.log(`observerC: ${v}`),
// });
```

### 各ブロックの説明

要点を実行結果のログで見ていきたいので、各ブロックの説明はコード中のコメントをご確認ください。

### 実行結果(1)

このコードの実行結果は次のとおりです。
実行結果を細かく見ていくことでコードで実装されている内容がどういうものか、冒頭の `ReplaySubject` の説明は何を言っているのかを理解したいと思います。

```log
// Logs:
observerA: 2 // 最初の注目点
observerA: 3
observerA: 4
---
observerA: 5 // 二番目の注目点
observerA: 6
observerA: 7
observerA: 8
---
observerB: 6 // 三番目の注目点
observerB: 7
observerB: 8
---
observerA: 9 // 四番目の注目点
observerB: 9
```

**最初の注目点**

`observerA` の購読において `next(1)` で配信されたストリームが購読されていないことがログから分かります。
これは

```typescript
// (1) 最初のブロック
// 3回分 の繰り返し用バッファを用意
// subscribe 前に配信されたストリームは ここで用意したバッファ分 保持され、subscribe で購読される
const subject = new ReplaySubject(3);
```

で用意した バッファ が効いていることの証明です。
つまり、コードコメントにある

> subscribe 前に配信されたストリームは ここで用意したバッファ分 保持され、subscribe で購読される

ことをこのログでは示しています。

**二番目の注目点**

バッファの有無に関係なく `next(5)~next(8)` が購読されています。
これは `subscribe` 後に配信されたストリームについては **用意されたバッファに関係なく購読される** ことを示しています。
つまり 通常の `Subject` と同じ動きです｡ `ReplaySubject` を使うと用意したバッファ分しか読み込まれないのではなく、通常の `Subject` 的な動きもすることに注目です。

`ReplaySubject` を使った購読の場合、

- `subscribe` **前** に配信されたストリームは **用意したバッファ分を最大回数として購読** する
- `subscribe` **後** に配信されたストリームは **用意したバッファに関係なく配信されたストリームを購読** する
  つまり 通常の `Subject` と同じ動きをする

と捉えておきます。

**三番目の注目点**

`observerB` の購読において `next(5)` で配信されたストリームが購読されていないことがログから分かります。
これは **最初の注目点** で確認したのと同じ動きです。ここの動きからも用意した バッファ が効いていることが分かります。

前掲の **二番目の注目点** と関係しますが、`next(5)~next(8)` で配信されたストリームの購読の仕方が `observerA` と `observerB` で異なることに充分注意してください。

- `observerA`
  - `subscribe` **後** に配信されたストリームなのでバッファに関係なく配信されたストリームをすべて購読している
  - なので `observerA` では `next(5)~next(8)` の値がログに出ている
- `observerB`
  - `subscribe` **前** に配信されたストリームなのでバッファ分だけ配信されたストリームを購読している
  - なので `observerB` では `next(6)~next(8)` の値しかログに出ていない( `next(5)` は購読されていない )

**四番目の注目点**

`observerA`, `observerB` ともに `subscribe` 後の配信なので、バッファの有無に関係なく両方で購読されていることが分かります。

**冒頭の説明を振り返る**
以上、サンプルコードから `ReplaySubject` の動きを見てきました。
ここでもう一度本項目の冒頭で注目した部分を挙げておきます。

:::note info

> ReplaySubjectは、古い値を新しい購読者に送ることができるという点ではBehaviorSubjectと似ていますが、Observableの実行の一部を記録することもできます。  
>
>> ReplaySubjectは、Observableの実行から複数の値を記録し、新しいSubscriberに再生することができます。

:::

`ReplaySubject` を使うことで **直前に配信された値** を指定回数記録し、それを **新しい subscribe で購読できる** ことが分かりました。

### おまけ(実行結果(2))

コメントアウト部分を外したときの実行結果も載せておきます。
実行結果についての説明はコード中のコメントをご参照ください。

```log
observerA: 2
observerA: 3
observerA: 4
---
observerA: 5
observerA: 6
observerA: 7
observerA: 8
---
observerB: 6
observerB: 7
observerB: 8
---
observerA: 9
observerB: 9
observerA: 10
observerB: 10
observerA: 11
observerB: 11
observerA: 12
observerB: 12
observerA: 13
observerB: 13
observerA: 14
observerB: 14
---
observerC: 12
observerC: 13
observerC: 14
```

# [AsyncSubject](https://rxjs.dev/guide/subject#asyncsubject)

`AsyncSubject` についての説明を上記リンクから転載します。

:::note info

> The AsyncSubject is a variant where only the last value of the Observable execution is sent to its observers, and only when the execution completes.
>
>( Deepl による翻訳 )
> AsyncSubjectは、Observableの実行の最後の値だけが、実行が完了したときだけ、そのオブザーバーに送信される変種である。

:::

サンプルコードで挙動を確認します。

## AsyncSubject の動きを確認する

```typescript
import { AsyncSubject } from 'rxjs';

// 最初のブロック
// AsyncSubject のコンストラクタは引数を受け取らない
const subject = new AsyncSubject();

// 二番目のブロック
// AsyncSubject では最後に発信されたストリームだけが流れてくるので `next(5)` の値だけが出力される
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});

// 三番目のブロック
subject.next(1); // ストリームが流れない
subject.next(2); // 同上
subject.next(3); // 同上
subject.next(4); // 同上

// 四番目のブロック
// 最初の購読と同じく、こちらも `next(5)` の値だけが出力される
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

// 五番目のブロック
subject.next(5);

// 六番目のブロック
// ここをコメントアウトすると subscribe にストリームが流れない
subject.complete();
```

### 各ブロックの説明

要点を実行結果のログで見ていきたいので、各ブロックの説明はコード中のコメントをご確認ください。

### 実行結果

このコードの実行結果は次のとおりです。

```log
// Logs:
observerA: 5
observerB: 5
```

**`AsyncSubject` におけるポイント**
**出力されたストリームが `next(5)` の値だけ** であることが重要なポイントです。
コード中のコメントにも記載してありますが、 `AsyncSubject` では **最後に発信されたストリームだけが流れてくる** ので、このサンプルコードでは **`next(5)` の配信だけが購読** されました。

またもう一つ重要なポイントが  `complete()` の存在です。
`AsyncSubject` では `subscribe` にストリームを流すために **`complete` による通知が必要** です。
上記コードで `subject.complete()` をコメントアウトすると `subscribe` にストリームが流れません。

**冒頭の説明を振り返る**
以上、サンプルコードから `AsyncSubject` の動きを見てきました。
ここでもう一度本項目の冒頭で注目した部分を挙げておきます。

:::note info

> AsyncSubjectは、Observableの実行の最後の値だけが、実行が完了したときだけ、そのオブザーバーに送信される変種である。

:::

`AsyncSubject` では `complete()` を使うことで **最後に配信されたストリームだけを購読する** ことが分かりました。

# 各 Subject の違いを振り返る

ここで改めて `Subject` とその派生の違いについて確認します｡

| オペレータ      | 特徴                                                                               |
| --------------- | ---------------------------------------------------------------------------------- |
| Subject         | 初期値を設定できない                                                               |
|                 | 配信されてきた値を保持できない                                                     |
|                 | 直前に配信された値はストリーム配信前に subscribe しておかないと購読できない        |
| BehaviorSubject | 初期値を設定できる                                                                 |
|                 | 配信されてきた値を保持できる                                                       |
|                 | 直前に配信された値をストリーム配信前に subscribe しておかなくとも購読できる        |
| ReplaySubject   | 直前に配信された値を指定回数記録できる                                             |
|                 | 記録した値は新しい subscribe で購読できる                                          |
|                 | subscribe 後に配信されたストリームについては用意されたバッファに関係なく購読される |
|                 | ( 通常の Subject と同じ動きをする )                                                |
| AsyncSubject    | 初期値を設定できない                                                               |
|                 | subscribe にストリームを流すために complete による通知が必要                       |
|                 | 最後に配信されたストリームだけを購読する                                           | 

# まとめにかえて

[`Subject`](https://rxjs.dev/guide/subject#subject) の派生である [`BehaviorSubject`](https://rxjs.dev/guide/subject#behaviorsubject), [`ReplaySubject`](https://rxjs.dev/guide/subject#replaysubject), [`AsyncSubject`](https://rxjs.dev/guide/subject#asyncsubject) について、[こちらの記事](https://qiita.com/ksh-fthr/items/54b19b4160505e2fddd9) と本記事の 2回 に分けて見てきました。
経験上、`subject` と `BehaviorSubject` を使う機会、見る機会が多く、他 2つ についてはほとんど扱う機会がなかったのですが、今回の記事はそれらを知る・学習するよい機会になりました。

記事の内容に不備や誤りがありましたらお知らせください。

# 参考

- [RxJS における Subject の活用]( https://zenn.dev/mikakane/articles/rxjs_5_subject)
- [RxJS を学ぼう #5 - Subject について学ぶ / Observable × Observer](https://blog.recruit.co.jp/rmp/front-end/post-11951/)
