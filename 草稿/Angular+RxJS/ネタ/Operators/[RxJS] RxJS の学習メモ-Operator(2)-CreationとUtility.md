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

ただすべてを扱うのは量的な面で難しいので、個人的によく使うメソッドに対する理解を深めたいと思います。


# [Creation Operators](https://rxjs.dev/guide/operators#creation-operators-1)

Creation Operators では `of` と `from` を扱います｡
どちらも **引数で渡された情報からストリームを生成する** オペレータですが､扱う引数や生成されるストリームに違いがあります｡

## [of](https://rxjs.dev/api/index/function/of)

`of` はストリームを生成するオペレータで､以下の特徴があります｡

- 引数に指定したデータからストリームを作り出す
  - 複数指定可能
  - 配列もオブジェクトも指定できるが、生成されるストリームはあくまで指定した引数の単位ごと

それではそれぞれのケースをサンプルコードで見ていきます｡

### 数値を複数指定したケース

```typescript
import { of } from 'rxjs';

of(1, 2, 3).subscribe({
  next: (x) => console.log(`stream=${x}`)
});

// Logs:
// stream=1
// stream=2
// stream=3
```

実行結果から `of` の引数に指定された `1`, `2`, `3` がそれぞれ独立して `subscirbe` に流れて行っているのがわかります｡

### 配列を指定したケース

```typescript
import { of } from 'rxjs';

of([1, 2, 3], [4,5,6]).subscribe({
  next: (x) => console.log(`stream=${x}`)
});

// Logs
// stream=1,2,3
// stream=4,5,6
```

こちらのケースでは `of` の引数を配列で指定していますが､その要素である `1,2,3` がまとめて `subscribe` に流れています｡
つまり冒頭で記載した

> 生成されるストリームはあくまで指定した引数の単位ごと

となっていることがわかります｡

### オブジェクトを指定したケース

```typescript
import { of } from 'rxjs';

of(
  {'id': 1, 'name': 'hoge', 'birthday': '2020/04/08'},
  {'id': 2, 'name': 'piyo', 'birthday': '2020/06/18'}
).subscribe({
  next: (x) => console.log(`stream=${JSON.stringify(x)}`)
});

// Logs:
// stream={"id":1,"name":"hoge","birthday":"2020/04/08"}
// stream={"id":2,"name":"piyo","birthday":"2020/06/18"}
```

前項の [配列を指定したケース](#配列を指定したケース) に同じです｡
こちらも引数で指定したオブジェクトの内容がひとまとめで `subscribe` に流れてきています｡


## [from](https://rxjs.dev/api/index/function/from)

`of` と同様､ `from` もストリームを生成するオペレータです｡
以下の特徴があります｡

- 引数に指定できるのは下記のみ
  - Array、array-likeオブジェクト
  - Promise、iterableオブジェクト
  - Observable-likeオブジェクト
- 引数に指定した 上記オブジェクトから ストリームを作り出す
  - 複数指定不可
  - 指定したオブジェクトの要素一つに対してストリームを1つ流す

`of` とはかなり勝手が異なります｡
それでは `from` を使った動きをサンプルコードで見ていきます｡
( 本記事では Array オブジェクトである 配列 を用いて動きを確認します )

### 配列を一つ指定

```typescript
import { from } from 'rxjs';

from([1, 2, 3]).subscribe({
  next: (x) => console.log(`stream=${x}`)
});

// Logs:
// 配列の要素毎にストリームが流れているのがわかる
// stream=1
// stream=2
// stream=3
```

>  - 指定したオブジェクトの要素一つに対してストリームを1つ流す

で示したとおり､配列の要素である `1`, `2`, `3` がそれぞれ個別に `subscribe` に流れてきているのがわかります｡ 

### 配列を複数指定

```typescript
import { from } from 'rxjs';

// このやり方はエラーになる
from([1, 2, 3], [4, 5, 6]).subscribe({
  next: (x) => console.log(`stream=${x}`)
});

// Error:
// -> Error: scheduler.schedule is not a function
```

`from` の引数に複数の配列を指定したところ `Error: scheduler.schedule is not a function` が発生しました｡
これは先に記した

>  - 複数指定不可

のとおりです｡
`from` を使って **配列をまるごとストリームとして流したい** 場合､ **配列を配列でくくる** ことで実現できます｡

```
import { from } from 'rxjs';

// 配列の中に配列を含める
from([[1, 2, 3], [4, 5, 6]]).subscribe({
  next: (x) => console.log(`stream=${x}`)
});

// Logs:
// 配列の要素毎にストリームが流れているのがわかる
// stream=1,2,3
// stream=4,5,6
```

`subscribe` に配列の要素ではなく､配列そのものである `[1,2,3]` と `[4,5,6]` が流れてきていることがわかります｡

### オブジェクトやプリミティブを指定するとどうなるか

単純なオブジェクトやプリミティブを指定した場合のエラーを確認してみます｡

**単純なオブジェクトを指定した例**

```typescript
import { from } from 'rxjs';

from(
  {'id': 1, 'name': 'hoge', 'birthday': '2020/04/08'}
).subscribe({
  next: (x) => console.log(`stream=${JSON.stringify(x)}`)
});

// Logs:
// Error: You provided an invalid object where a stream was expected. You can provide an Observable, Promise, ReadableStream, Array, AsyncIterable, or Iterable.
```

**プリミティブ型を指定した例**

```typescript
import { from } from 'rxjs';

from(1, 2, 3).subscribe({
  next: (x) => console.log(`stream=${JSON.stringify(x)}`)
});

// Logs:
// Error: You provided '1' where a stream was expected. You can provide an Observable, Promise, ReadableStream, Array, AsyncIterable, or Iterable.
```

いずれも指定可能な型ではないということでエラーになります｡
なお蛇足ながら､上記コードに対して `from` の引数を `of` で囲ってやりますと､

>  - Observable-likeオブジェクト

を満たすことになるので､コードは正常に実行されます｡

```typescript
import { of, from } from 'rxjs';

from(
  // `of` で囲ってやれば Observable-likeオブジェクト が from の引数になるので無事実行される
  of({'id': 1, 'name': 'hoge', 'birthday': '2020/04/08'})
).subscribe({
  next: (x) => console.log(`stream=${JSON.stringify(x)}`)
});

// Logs:
// stream={"id":1,"name":"hoge","birthday":"2020/04/08"}
```

このとおり､ オブジェクト `{'id': 1, 'name': 'hoge', 'birthday': '2020/04/08'}` が `subscribe` に流れました｡

## of と from の補足

:::not info
`scheduler` パラメータ付きで使用することは `of` と `from` 双方ともに非推奨となりました｡
`scheduler` を利用したコードは `RxJS v7.x` で実装された [`scheduled`](https://rxjs.dev/api/index/function/scheduled) への置き換えを推奨されています｡
詳しくは [こちら](https://rxjs.dev/deprecations/scheduler-argument#scheduler-argument) をご参照ください｡
以下は 上記リンク からの抜粋です｡

> (抜粋)
>
> ```text
> This deprecation was introduced in RxJS 6.5 and will become breaking with RxJS 8.
> ```
:::

# [Utility Operators](https://rxjs.dev/guide/operators#utility-operators)

## [tap](https://rxjs.dev/api/operators/tap)

`tap` と `map` の違いを明示したかったので、ここのサンプルコードでは 両方を扱った。
`tap` で行った加工処理が `map` に渡っていないことが確認できる。

```typescript
import {
  of,
  BehaviorSubject,
} from 'rxjs';

import {
  map,
  tap,
} from 'rxjs/operators';

const receiver$ = new BehaviorSubject<string>('初期値');
const streamData$ = of('streamData');

receiver$.subscribe((receiver) => {
  // 初期値の購読で1回、 streamData$ の購読で 1回 の 計2回 流れる
  console.log(`receiver=${receiver}`);
});

streamData$
.pipe(
  tap((streamData: string) => {
    console.log(`tap に入ってきたときの値=${streamData}`);

    // 出力結果で map に加工した値が渡っていないことを明示したかったので return で配列を返しているが、
    // ここの return　は無くても文法的にエラーにならない
    // というか、本来は tap で return 文を書くのは混乱のもとになるので書くのは NG としたほうが良いと思う
    streamData = `${streamData} を tap で加工して return で返す`
    return streamData;
  }),
  map((streamData: string) =>  {
    // tap で加工した値は渡ってこず、streamData$ の初期値に設定した `streamData` が出力される
    console.log(`map に入ってきたときの値=${streamData}`);

    // map は戻り値が必要なのでこの return 文は必須
    streamData = `${streamData} には map で加工した値が流れる. tap で加工しても値は流れてこない`
    return streamData;
  })
)
.subscribe((streamData: string) => {
  receiver$.next(streamData);
});
```

```bash
receiver=初期値
tap に入ってきたときの値=streamData
map に入ってきたときの値=streamData
receiver=streamData には map で加工した値が流れる. tap で加工しても値は流れてこない
```

## [subscribe](https://rxjs.dev/api/index/function/subscribeOn)

```typescript
import {
  of,
  BehaviorSubject,
} from 'rxjs';

const receiver$ = new BehaviorSubject<string>('初期値');
const streamData$ = of('streamData');

receiver$.subscribe((receiver) => {
  // 初期値の購読で1回、 streamData$ の購読で 1回 の 計2回 流れる
  console.log(`receiver=${receiver}`);
});

streamData$.subscribe((streamData) => {
  receiver$.next(`${streamData} を購読して加工したものを別のストリームに流す.`);
});
```

```bash
# 宣言時に指定した初期値が表示される
receiver=初期値
# ストリームが流れてきたので、その値が表示される
receiver=streamData を購読して加工したものを別のストリームに流す.
```

## 補足-1

### `BehaviorSubject` と `Subject` の違い

- `BehaviorSubject`
  
  - 初期値を設定できる
  - ストリームで流れてきた値を購読する
  - 且つ、ストリームで流れてきた値を保持できる

- `Subject`
  
  - 初期値を設定できない
  - ストリームで流れてきた値を購読する
  - ストリームで流れてきた値を保持できない

前掲のコードーを `Subject` で書き直すとこうなる。

```typescript
import {
  of,
  Subject,
  BehaviorSubject,
} from 'rxjs';

// const receiver$ = new BehaviorSubject<string>('初期値');
const receiver$ = new Subject<string>();
const streamData$ = of('streamData');

receiver$.subscribe((receiver) => {
  // streamData$ の購読で 1回 流れる
  console.log(`receiver=${receiver}`);
});

streamData$.subscribe((streamData) => {
  receiver$.next(`${streamData} を購読して加工したものを別のストリームに流す.`);
});
```

```bash
# 初期値がないので購読は一回だけ
receiver=streamData を購読して加工したものを別のストリームに流す.
```

## 補足-2

正常時、エラー時、完了時の文法について。
次の書き方は OK。

```typescript
hoge$.subscribe({
    next: (response: any) => { // 実施はちゃんと型定義しましょう
        this.accounts = response;
    },
    error: (error: HttpErrorResponse) => {
        alert(error.message);
    },
    complete: () => {
        // do something when the observable completes
        // まぁ、書かなくても良い( 本当はちゃんと決めておくのが行儀が良いとは思う )
    }
}
```

次の書き方は[非推奨](https://rxjs.dev/deprecations/subscribe-arguments#what-signature-is-affected)。

```typescript
hoge$.subscribe(
    (response: any) => { // 実施はちゃんと型定義しましょう
        this.accounts = response;
    },
    (error: HttpErrorResponse) => {
        alert(error.message);
    }
)
```

## 蛇足

前掲のコードは次のように書くことでストリームを流し続けることが出来る。
でもうまく制御しないとオーバーフローが発生する。

```typescript
import {
  of,
  Subject,
  BehaviorSubject,
} from 'rxjs';

const receiver$ = new BehaviorSubject<string>('初期値');
const streamData$ = receiver$.asObservable(); // 流す値は `reciver$` の値

receiver$.subscribe((receiver) => {
  // receiver$ を streamData$ の購読対象としていて、streamData$ の中で receiver$ にストリームを流してるので無限ループに...
  console.log(`receiver=${receiver}`);
});

streamData$.subscribe((streamData) => {
  // streamData$ には receiver$ の値が流れてきて
  // 購読したら `receiver$` に値を流す
  // という無限ループに陥る
  receiver$.next(`${streamData} を購読して加工したものを別のストリームに流す.`);
});
```

```bash
receiver=初期値
receiver=初期値 を購読して加工したものを別のストリームに流す.
receiver=初期値 を購読して加工したものを別のストリームに流す. を購読して加工したものを別のストリームに流す.
receiver=初期値 を購読して加工したものを別のストリームに流す. を購読して加工したものを別のストリームに流す. を購読して加工したものを別のストリームに流す.
.
.
.
(以下、繰り返し。最終的に↓が発生)
Error: Maximum call stack size exceeded
```

## 蛇足-2

ちなみに...。↑ は `BehaviorSubject` の例だが、これを `Subject` にするとこうなる。

```typescript
import {
  of,
  Subject,
  BehaviorSubject,
} from 'rxjs';

// const receiver$ = new BehaviorSubject<string>('初期値');
const receiver$ = new Subject<string>();
const streamData$ = receiver$.asObservable();


receiver$.subscribe((receiver) => {
  // receiver$ を streamData$ の購読対象としているけれども初期値が設定されていないので、そもそも streamData$ にストリームが流れないので 1回 も流れてこない
  console.log(`receiver=${receiver}`);
});

streamData$.subscribe((streamData) => {
  receiver$.next(`${streamData} を購読して加工したものを別のストリームに流す.`);
});
```

```bash
# なにも流れてこない -> 初期値を設定していないから最初の receiver$.subscribe で購読されない
# なので後続の streamData$.subscribe にも値が流れてこない
# 結果、なにも処理されない
```

## [toArray](https://rxjs.dev/api/operators/toArray)

```typescript
import { of, take, toArray } from 'rxjs';


const stream$ = of('one', 'two', 'three', 'foure', 'five');
const streamObserver = stream$.pipe(
  // ストリームから流れてくる値のうち、指定した数( 回数 )だけ処理するフィルタ
  // 指定回数分処理すると complete イベントが発火する
  take(5),

  // complete まで流れてきた値を配列にまとめて返す
  toArray()
);

streamObserver.subscribe(value => console.log(value));
```

```bash
["one", "two", "three", "foure", …]
0: "one"
1: "two"
2: "three"
3: "foure"
4: "five"
```

## 補足

### サンプルコードの出力結果について

```
- take(5) で 5回 指定したので 'one', 'two', 'three', 'foure', 'five' がセットされた配列が出力される
- take(2) を指定すれば `one`, `two` までが出力される
```
