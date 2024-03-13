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

今回は [Operators](https://rxjs.dev/guide/operators) について学びます。

# 環境

本記事について扱うライブラリや環境の情報です。

|                                        | 備考                                                        |
| -------------------------------------- | ----------------------------------------------------------- |
| [RxJS](https://rxjs.dev/)              | 公式                                                        |
| [Learn RxJS](https://www.learnrxjs.io/) | リファレンス的な感じの学習サイト                        |
| [StackBlitz](https://stackblitz.com/)  | RxJS だけでなく Angular とか React とかの実装お試しができる |

なお本記事執筆時の RxJS のバージョンは StackBlitz の DEPENDENCIES を見ると `v7.8.0` でした。

# この記事でやること

## [Operators](https://rxjs.dev/guide/operators) について触れてみる

`Operators` は RxJS の基本コンセプトである 6つ のコンセプトに含まれるものです。

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
> - **Operators: are pure functions that enable a functional programming style of dealing with collections with operations like map, filter, concat, reduce, etc.**
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
> - **Operators: map、filter、concat、reduceなどの操作でコレクションを扱う関数型プログラミングスタイルを可能にする純粋な関数です。**
> - Subject: EventEmitterに相当し、値やイベントを複数のObserversにマルチキャストする唯一の方法です。
> - Schedulers: 同時実行をコントロールするための集中ディスパッチャで、例えばsetTimeoutやrequestAnimationFrameなどで計算が発生するタイミングを調整できる。

:::

前置きが長くなりました。
これから実際に `Operators` を見ていこうと思うのですが、`Operetors` は [こちら](https://rxjs.dev/guide/operators#categories-of-operators) で示されているように多くのオペレータがあります。
そのすべてを扱うことは量的に難しいので、本記事ではそのなかでも特によく使うことになる、もしくは知っておくと便利だと思われる次のオペレータについて触れていきます。

**本記事で扱うオペレータ**

- `subscribe` する前に各オペレータを繋ぐ役割をもつ [`pipe`](https://rxjs.dev/guide/operators#piping)
- ストリームに対してフィルタリングをしてくれる [`Filtering Operators`](https://rxjs.dev/guide/operators#filtering-operators)

# [Pipe](https://rxjs.dev/guide/operators#piping)

`subscribe` で購読する前に、`observable` で観察していたデータを加工するための **繋ぎ** を実現するためのオペレータとして [`pipe`](https://rxjs.dev/api/index/function/pipe) があります。
`pipe` では、その中でストリームで流れているデータに対する操作やフィルタリングを行います。

以下、サンプリコードを見つつ動きを確認していきます。

## Pipe の動きを確認する

以下のコードは [`map`](https://rxjs.dev/api/operators/map) の中で配列化しただけの単純な例ですが、`pipe` がどういう動きをするものか、という雰囲気はつかめると思います。
なお、ここでは詳細は触れませんが、サンプルコード中にでてくる `map` は [`Transformation Operators`](https://rxjs.dev/guide/operators#filtering-operators) に属するオペレータ、[`of`](https://rxjs.dev/api/index/function/of) は [`Creation Operators`](https://rxjs.dev/guide/operators#creation-operators-1) に属するオペレータです。

```typescript
import {
  of,
  BehaviorSubject,
} from 'rxjs';

import {
  map,
} from 'rxjs/operators';

// (1) 最初のブロック
const receiver$ = new BehaviorSubject<string>('初期値');
const streamData$ = of('streamData');

// (2) 二番目のブロック
receiver$.subscribe({
  next: (receiver) => {
    // 初期値の購読で1回、 streamData$ の購読で 1回 の 計2回 流れる
    console.log(`receiver=${receiver}`);
  }
});

// (3) 三番目のブロック
streamData$
.pipe(
  map((streamData: string) => [streamData, 'map で加工した', '値が流れる'])
)
.subscribe({
  next: (streamData: string[]) => {
    receiver$.next(`${streamData[0]} を ${streamData[1]} ${streamData[2]}.`);
  }
});
```

### 最初のブロック

```typescript
// (1) 最初のブロック
const receiver$ = new BehaviorSubject<string>('初期値');
const streamData$ = of('streamData');
```

ここでは [`BehaviorSubject`](https://rxjs.dev/guide/subject#behaviorsubject) のオブジェクト `receiver$` と [`Observable`](https://rxjs.dev/guide/observable) のオブジェクト `streamData$` を生成します。
それぞれの役割は次のとおりです。

- `receiver$`
  - 流れてきたストリームを購読してログ出力するだけ
  - 初期化時と `streamData$` の購読時の 2回、ストリームが流れる
- `streamData$`
  - 生成したストリームは 三番目のブロック で `pipe` によって `subscribe` 前に加工される
  - 加工されたデータは `subscribe` によって購読され、`receiver$` に新しいストリームとして流される

### 二番目のブロック

```typescript
// (2) 二番目のブロック
receiver$.subscribe({
  next: (receiver) => {
    // 初期値の購読で1回、 streamData$ の購読で 1回 の 計2回 流れる
    console.log(`receiver=${receiver}`);
  }
});
```

`receiver$` に対して `subscribe` するだけの単純なものです。このブロック自体に特筆することはありません。
コードコメントのとおりの動きをします。

### 三番目のブロック

```typescript
// (3) 三番目のブロック
streamData$
.pipe(
  map((streamData: string) => [streamData, 'map で加工した', '値が流れる'])
)
.subscribe({
  next: (streamData: string[]) => {
    receiver$.next(`${streamData[0]} を ${streamData[1]} ${streamData[2]}.`);
  }
});
```

このサンプルコードにおけるキモの部分です。
`streamData$` に対して `subescribe` で購読する前に `pipe` でストリームを加工しています。
そして加工したストリームは `subscribe` によって購読されます。つまり、この処理では以下の動きとなります。

1. `streamData$` のストリームとして 文字列: `streamData` が流れてくる
2. `pipe` では `map` を経由することで 次の **3つ の文字列を 配列 の要素とした新しいストリーム** を生成した
    - `1.` で流れてきた文字列: `streamData`
    - 文字列: `map で加工した`
    - 文字列: `値が流れる` 
3. `subscribe` では `2` で生成された `string[]` のストリームを購読している
4. 最後、購読した `string[]` の各要素を [テンプレートリテラル](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Template_literals) で連結し､ `receiver$` に対して `next` で流す

このコードの実行結果が次項で示すログです。

### 実行結果

```log
receiver=初期値
receiver=streamData を map で加工した 値が流れる.
```

2行目の `receiver=streamData を〜` が示すとおり、`receiver$` の 2回目 に購読された情報が `streamData$` のストリームにおいて `pipe` と `map` で加工されたデータであることが分かります。

# [Filtering Operators](https://rxjs.dev/guide/operators#filtering-operators)

ここまでで `pipe` でできることがわかったので、今度は `pipe` 内でストリームをフィルタリングするオペレータ、`Filtering Operators` を見ていきます。
本記事では `first`, `filter`, `distinctUntilChanged` の 3つ を扱います。

# [first](https://rxjs.dev/api/operators/first)

`first` は ストリームが流れてきたとき **最初の値だけを次の処理に回す** オペレータです。
サンプルコードでその動きを確認します。

## first の動きを確認する

```typescript
import {
  BehaviorSubject
} from 'rxjs';

import { 
  first
} from 'rxjs/operators'

// (1) 最初のブロック
// 最初のストリームとして `初期値` を流す
const receiver$ = new BehaviorSubject<string>('初期値');

// (2) 二番目のブロック
receiver$
.pipe(
  first(),
)
.subscribe({
  next: (receiver) => {
    // first で 最初の値だけ流すようにしているので `初期値` だけが出力される
    console.log(`receiver=${receiver}`);
  },
  complete: () => {
    // first が実行されると `complete` も自動的に発火される
    console.log(`complete`);
  }
});

// (3) 三番目のブロック
// 後続のストリームを流す ( けれど、first を実行しているので実際には流れない )
receiver$.next('2回目')
receiver$.next('3回目')
```

## 実行結果(1)

このサンプルコードの実行結果は次のとおりです。

```log
receiver=初期値
complete
```

ポイントは 2つ あります。

**ひとつ目**
コード中の `(2) 二番目のブロック` の `next` ブロックのコメントにある通り、`pipe` 内で `first` を実行したことで **最初のストリームだけが処理された** ことが分かります。

**2つ目**
同じくコード中の `(2) 二番目のブロック` です。`complete` ブロックのコメントにある通り、**`first` を使用すると `complete` が自動的に発火されます**。
ひとつ目のポイントと合わせ、 **`first` を指定すると後続のストリームが流れない** 、というのがここからも理解できると思います。

## first を外すとどうなる

```typescript
import {
  BehaviorSubject
} from 'rxjs';

import { 
  first
} from 'rxjs/operators'

// (1) 最初のブロック
// 最初のストリームとして `初期値` を流す
const receiver$ = new BehaviorSubject<string>('初期値');

// (2) 二番目のブロック
receiver$
.pipe(
//  first(),
)
.subscribe({
  next: (receiver) => {
    // first を外しているので 初回云々関係なく ストリームが処理される
    console.log(`receiver=${receiver}`);
  },
  complete: () => {
    // first を外しているので `complete` は自動的に発火されない
    // complete を発火するには明示的に `receiver$.complete()` とする必要がある
    console.log(`complete`);
  }
});

// (3) 三番目のブロック
// 後続のストリームを流す ( first を実行していないのでこちらもちゃんと流れる )
receiver$.next('2回目')
receiver$.next('3回目')
```

### 実行結果(2)

このサンプルコードの実行結果は次のとおりです。
初回のストリームである `初期値` だけではなく、後続のストリームが `2回目`, `3回目` と流れています。

```log
receiver=初期値
receiver=2回目
receiver=3回目
```

また `complete` がログに出ていないことにも注目です。
これは `complete` が発火されなかったことを示しています。つまりストリームは活きている状態です。
ストリームを終了させるには明示的に `complete` を実行する必要があります。
サンプルコードの最後に `receiver$.complete()` を記述するとログには `complete` が出力されます。ご興味あればお試しください。

なお `subscribe` 中の `next` や `complete` については [こちらの記事](https://qiita.com/ksh-fthr/items/2933492929bbeccece50#%E4%B8%89%E7%95%AA%E7%9B%AE%E3%81%AE%E3%83%96%E3%83%AD%E3%83%83%E3%82%AF) でも触れております。
ご興味あればそちらも合わせてご参照ください。

# [filter](https://rxjs.dev/api/index/function/filter)

ストリームが流れてきたときに **条件に合致した値** を次の処理に回すオペレータです。
サンプルコードでその動きを確認します。

## filter の動きを確認する

```typescript
import {
  BehaviorSubject
} from 'rxjs';

import { 
  filter
} from 'rxjs/operators'

// (1) 最初のブロック
// 最初のストリームとして `初期値` を流す
const receiver$ = new BehaviorSubject<string>('初期値');

// (2) 二番目のブロック
receiver$
.pipe(
  // 文字列: `初期値` に合致するストリームを subscribe に流す
  filter((stream: string) => {
    console.log(`filter: ${stream}`)
    return stream === '初期値'
  }),
)
.subscribe({
  next: (receiver) => {
    // filter で `初期値` を条件に指定しているのでこのブロックで出力されるログには `初期値` だけが出力される
    console.log(`subscribe=${receiver}`);
  }
});

// (3) 三番目のブロック
// 後続のストリームを流す ( ストリームは流れるものの、filter の条件に合致しないので subscribe されない )
receiver$.next('2回目')
receiver$.next('3回目')
```

## 実行結果

このサンプルコードの実行結果は次のとおりです。

```log
filter: 初期値
subscribe=初期値
filter: 2回目
filter: 3回目
```

`初期値` のストリームを流したときには `filter` 並びに `subscribe` の各処理でログが出力されていること。
`2回目`、 `3回目` のストリームを流したときには `filter` でのログのみが出ていて、`subscribe` のログが出ていないことから、`filter` の役割を確認できました。

# [distinctUntilChanged](https://rxjs.dev/api/index/function/distinctUntilChanged)

ストリームが流れてきたとき **`comparator` に指定した条件に合致した値だけ** を処理するオペレータです。
`comparator` は省略可能で、デフォルトの動きは 「**前回流れてきたデータと異なる場合** に処理する」です。
サンプルコードでその動きを確認します。

## distinctUntilChanged の動きを確認する( comparator で条件指定なし )

### 同じ値が連続しているケース

```typescript
import {
  of,
  distinctUntilChanged,
} from 'rxjs';

// (1) 最初のブロック
// 指定した文字列がストリームとなって順次流れていく
const streamData$ = of('同じ', '同じ', 'データ', 'データ', 'は', 'は', '流れない', '流れない');

// (2) 二番目のブロック
streamData$
.pipe(
  // distinctUntilChanged を引数なしで実行すると、同じ値が連続した場合 はストリームが後続処理に流れない
  // このケースでは「流れない」が連続しているので、2つ目の「流れない」は subscribe に流れていかない
  distinctUntilChanged()
)
.subscribe({
  next: (streamData) => {
    // 「同じデータは流れない」と出力される
    console.log(`receiver=${streamData}`);
  }
});
```

### 実行結果(1)

このサンプルコードの実行結果は次のとおりです。
コード中のコメントのとおり、最後に連続している「流れない」は両方処理されずに片方だけ `subscribe`` に流れていることが分かります。

```log
receiver=同じ
receiver=データ
receiver=は
receiver=流れない
```

### 同じ値ではあるが連続していないケース

```typescript
import {
  of,
  distinctUntilChanged,
} from 'rxjs';

// (1) 最初のブロック
// 指定した文字列がストリームとなって順次流れていく
const streamData$ = of('同じ', '同じ', 'データ', 'だけど', '連続していないから', '同じ', '値でも', '流れる', 'は', '流れる',);

// (2) 二番目のブロック
streamData$
.pipe(
  // 連続した同じデータは処理されないが、間に別の値が挟まった場合は同じデータでも処理される
  distinctUntilChanged()
)
.subscribe({
  next: (streamData) => {
    // 「同じデータでも連続していないから流れるは流れる」と出力される
    console.log(`receiver=${streamData}`);
  }
});
```

### 実行結果(2)

このサンプルコードの実行結果は次のとおりです。
「同じ」は 2つ 続いているために処理されていないこと、「流れる」は同じ文字列ではあるものの間に「は」を挟んでいるので `subscribe` まで流れていることが分かります。

```log
receiver=同じ
receiver=データ
receiver=だけど
receiver=連続していないから
receiver=同じ
receiver=値でも
receiver=流れる
receiver=は
receiver=流れる
```

## distinctUntilChanged の動きを確認する( comparator で条件指定あり )

`comparator` は `distinctUntilChanged` の第一引数に指定する関数です。
シグネチャは次のとおりです。

```typescript
/**
 * @param
 *  第一引数: 前回のストリーム
 *  第二引数: 今回のストリーム 
 * 
 * @returns 真偽値
 *  true が返った場合は処理を後続に流さない
 *  false が返った場合は処理を後続に流す
 */
((previous: K, current: K) => boolean)
```

詳しくは公式ドキュメントの [こちら](https://rxjs.dev/api/index/function/distinctUntilChanged#comparator) をご参照ください。

### 前後で違うものは出力しない( 前後で同じものを出力する )

```typescript
import {
  of,
  distinctUntilChanged,
} from 'rxjs';

// (1) 最初のブロック
// 指定した文字列がストリームとなって順次流れていく
const streamData$ = of('同じ', '同じ', 'データ', 'だけど', '連続していないから', '同じ', '値でも', '流れる', 'は', '流れる');

// (2) 二番目のブロック
streamData$
.pipe(
  distinctUntilChanged((prev, current) => {
    // 前後で違うものは出力しない( 前後で同じものを出力する )
    return prev !== current;
  })
)
.subscribe({
  next: (streamData) => {
    // 「同じ同じ同じ」と出力される
    console.log(`receiver=${streamData}`);
  }
});
```

### 実行結果(3)

このサンプルコードの実行結果は次のとおりです。

```log
receiver=同じ
receiver=同じ
receiver=同じ
```

ストリームには `'同じ', '同じ', 'データ', 'だけど', '連続していないから', '同じ', '値でも', '流れる', 'は', '流れる'` が順に流れてきます。
`comparator` は最初のストリームである `同じ` から順次値をチェックします。
`comparator` の条件は `prev !== current` で **前後で値が異なる場合は `true`** としているので、**前後で違うものは出力しない** ( つまり **前後で同じものを出力する** ) 動きとなります。

**注目**
ここで **前後で違うものは出力しない** という点とログの出方に注目です。
ログには 「同じ同じ同じ」と出ましたが、ストリームでは「同じ」が 2つ 続いた後、次の「同じ」までいくつかのストリームが流れています。
つまり、ストリーム上では 3つ目 の「同じ」は前のストリームとは異なります。にも関わらず、**前後で違うものは出力しない** に従い 3つ目 の「同じ」が出力されています。
これは次の動きとなっていることを示しています。

:::note info
`distinctUntilChanged` で `comparator` を指定した場合、`comparator` の 第一引数である `prev` には **前回処理時に後続のストリームに流れた値が保持されている**
:::

`distinctUntilChanged` のブロックで `comparator` 内に次のようにログを仕込んでみてください。
`prev` には 「同じ」が延々と出力されます。

```typescript
  distinctUntilChanged((prev, current) => {
    console.log({
      prev,
      current
    })
    // 前後で違うものは出力しない
    return prev !== current;
  })

// Logs:
// receiver=同じ
// {prev: "同じ", current: "同じ"}
// receiver=同じ
// {prev: "同じ", current: "データ"}
// {prev: "同じ", current: "だけど"}
// {prev: "同じ", current: "連続していないから"}
// {prev: "同じ", current: "同じ"}
// receiver=同じ
// {prev: "同じ", current: "値でも"}
// {prev: "同じ", current: "流れる"}
// {prev: "同じ", current: "は"}
// {prev: "同じ", current: "流れる"}
```

### 前後で同じものは出力しない( 前後で違うものを出力する )( 通常と同じうごき )

```typescript
import {
  of,
  distinctUntilChanged,
} from 'rxjs';

// (1) 最初のブロック
// 指定した文字列がストリームとなって順次流れていく
const streamData$ = of('同じ', '同じ', 'データ', 'だけど', '連続していないから', '同じ', '値でも', '流れる', 'は', '流れる',);

// (2) 二番目のブロック
streamData$
.pipe(
  distinctUntilChanged((prev, current) => {
    // 前後で同じものは出力しない( 前後で違うものを出力する )
    return prev === current;
  })
)
.subscribe({
  next: (streamData) => {
    // 「同じデータでも連続していないから流れるは流れる」と出力される
    console.log(`receiver=${streamData}`);
  }
});
```

### 実行結果(4)

このサンプルコードの実行結果は次のとおりです。

```log
receiver=同じ
receiver=データ
receiver=だけど
receiver=連続していないから
receiver=同じ
receiver=値でも
receiver=流れる
receiver=は
receiver=流れる
```

先程のコードと同じく、ストリームには `'同じ', '同じ', 'データ', 'だけど', '連続していないから', '同じ', '値でも', '流れる', 'は', '流れる'` が順に流れてきます。
`comparator` は最初のストリームである `同じ` から順次値をチェックします。
`comparator` の条件は `prev === current` で **前後で値が同じ場合は `true`** としているので、**前後で同じものは出力しない** ( **前後で違うものを出力する** ) 動きとなります。
つまり `comperator` を指定しないときと同じ動きです。

# 補足

本記事で `first` を使用することで、ストリームが流れる回数を最初の一回のみに制限できることを見てきました。
この **ストリームが流れる回数を最初の一回のみに制限** する動きについて **Angular の HttpClientModule** が似た動きをするので、それについて触れておきます。

## Angular の HttpClientModule

Angular では Http クライアントの実装で [HttpClientModule](https://angular.jp/api/common/http/HttpClientModule) に含まれる [HttpClient](https://angular.jp/api/common/http/HttpClient) を利用することが多いと思います。
この `HttpClient` でも RxJS を利用していますので、それについて補足します。

`HttpClient` は REST-API をコールするときに利用するモジュールで、REST-API からのレスポンスとして `RxJS` の `Observable` を返してくれます。
ということは 当然ストリームが流れてくるわけですが、REST-API からのストリームが延々と流れてくるのでは困ります。
が、`HttpClient` ではそのことを気にする必要はありません。また `first` を使って制限をかける必要もありません。

以下、公式からの転載です。

( 転載もと: [HttpClient のメソッドはひとつの値を返す](https://angular.jp/tutorial/tour-of-heroes/toh-pt6#httpclient%E3%81%AE%E3%83%A1%E3%82%BD%E3%83%83%E3%83%89%E3%81%AF%E3%81%B2%E3%81%A8%E3%81%A4%E3%81%AE%E5%80%A4%E3%82%92%E8%BF%94%E3%81%99) )

:::note info

( 太字については本記事にて加工しました )

> HTTPはリクエスト/レスポンスプロトコルです。 リクエストを送信すると、ひとつのレスポンスを返却します。
>
> 一般には、Observableは時間によって複数の値を返すことが 可能 です。 **HttpClientが返すObservableは常にひとつの値を発行してから完了する** ので、再び値を発行することはありません。

:::

ということで 繰り返しになりますが、 `HttpClient` を用いた場合、`first` でストリームが流れる回数に対して制限を掛ける必要はありません。

## 補足の補足 - HttpClient における subscribe のイベント

上記を更に補足します。RxJS の `subscribe` には次のイベントが用意されています。
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

というものです。これに対して Angular の `HttpClient` では **値が取得できたら `next`, 次に `complete` を流し** ます。
ということは、 `HttpClient` で流れてきたストリームに対して `first` を使っても、(`complete` が流れてきているので) その前後で動きが変わることがない、となります。

つまり `HttpCliente` 使うときに `first` は **使っても使わなくても結果は変わらず**、よって  **使う必要がない イコール 不要**、となります。

# まとめにかえて

本記事では [`Operators`](https://rxjs.dev/guide/operators) から [`Pipe`](https://rxjs.dev/guide/operators#piping) と [`Filtering Operators`](https://rxjs.dev/guide/operators#filtering-operators) について触れました。
Operators にはまだまだたくさんのオペータがあります。それらについては次回以降で触れていきます。

# 参考

- [RxJS Marbles](https://rxmarbles.com/)
- [マーブル図で怖くない RxJS](https://www.slideshare.net/bitbankink/rxjs-159715695)
- [RxJS を学ぼう #2 - よく使う ( と思う ) オペレータ 15 選](https://blog.recruit.co.jp/rmp/front-end/post-11475/)
- [RxJS のオペレーターの動きをデモアプリを自作して確認してみた](<https://note.com/shift_tech/n/n7643a684e947#map()>)
- [Angular-RxJS ライブラリ](https://angular.jp/guide/rx-library#rxjs-%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA)
- [Angular のための RxJS](https://learn-rxjs-for-angular.info/)
