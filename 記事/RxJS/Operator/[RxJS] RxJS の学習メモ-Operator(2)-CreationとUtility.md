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
ご興味あればご覧ください。

今回の記事では次のオペレータについて触れていきます｡

**本記事で扱うオペレータ**

- ストリームを生成する [Creation Operators](https://rxjs.dev/guide/operators#creation-operators-1)
- ユーティリティである [Utility Operators](https://rxjs.dev/guide/operators#utility-operators)

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

実行結果から `of` の引数に指定された `1`, `2`, `3` がそれぞれ独立して `subscirbe` に流れて行っているのが分かります｡

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
これは冒頭で記載した

> 生成されるストリームはあくまで指定した引数の単位ごと

となっていることを  しています｡

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
  - 指定したオブジェクトの要素一つに対してストリームを一つ流す

`of` とはかなり勝手が異なります｡

以下のサンプルコードで `from` を使った動きを見ていきます｡
( 本記事では Array オブジェクトである 配列 を用いて動きを確認します )

### 配列を一つ指定

```typescript
import { from } from 'rxjs';

from([1, 2, 3]).subscribe({
  next: (x) => console.log(`stream=${x}`)
});

// Logs:
// 配列の要素毎にストリームが流れているのが分かる
// stream=1
// stream=2
// stream=3
```

実行結果のログから

>  - 指定したオブジェクトの要素一つに対してストリームを一つ流す

上記のとおり､配列の要素である `1`, `2`, `3` がそれぞれ個別に `subscribe` に流れてきているのが分かります｡ 

### 配列を複数指定

次のサンプルコードではエラーが発生することを確認します｡

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
しかしながら､ `from` を使って **配列をまるごとストリームとして流したい**, **複数の配列を流したい** ケースもあるかもしれません｡
そんなときは **配列を配列でくくる** ことで実現できます｡

```typescript
import { from } from 'rxjs';

// 配列の中に配列を含める
from([[1, 2, 3], [4, 5, 6]]).subscribe({
  next: (x) => console.log(`stream=${x}`)
});

// Logs:
// 配列の要素毎にストリームが流れているのが分かる
// stream=1,2,3
// stream=4,5,6
```

`subscribe` に配列の要素ではなく､配列そのものである `[1,2,3]` と `[4,5,6]` が流れてきていることが確認できました｡

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

:::note info
本記事では扱っていませんが､ `scheduler` パラメータ付きで使用することは `of` と `from` 双方ともに非推奨となりました｡
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

Utility Operators では以下の Operators について見ていきます｡

- [tap](https://rxjs.dev/api/index/function/tap)
- [toArray](https://rxjs.dev/api/operators/toArray)

## [tap](https://rxjs.dev/api/operators/tap)

ストリームで流れてきたデータに対して同期的に処理を行います｡
[公式の tap ページ](https://rxjs-dev.firebaseapp.com/api/operators/tap) には

> Used to perform side-effects for notifications from the source observable

とあり、副作用について扱うメソッドと説明があります。具体的にはログ出したり 事前にデータ検証して例外を発生させたりといった､本来の目的ではない処理を実行させるために用いられるオペレータになります。

そして重要なポイントですが **`tap` では 新たに値を生成して返すことはしません。**

ちょっとニュアンスが異なりますが､イテレータで言うところの `forEach` をイメージすると分かりやすいかと思います。
逆にストリームで流れてきたデータを処理し､その処理した結果を次の処理をつなげたい場合は `map` 等の [Transformation Operators](https://rxjs.dev/guide/operators#transformation-operators) を使います。

今回の記事では [Transformation Operators](https://rxjs.dev/guide/operators#transformation-operators) について詳しくは触れませんが､ `tap` と `map` の違いを明示したかったのでサンプルコードでは両方を扱いました。

以下のサンプルコードから `tap` で返却した値が `map` に渡っていないことが確認できます。

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
const streamData$ = of('加工対象の値');

receiver$.subscribe((receiver) => {
  // 初期値の購読で1回、 streamData$ の購読で 1回 の 計2回 流れる
  console.log(`receiver=${receiver}`);
});

streamData$
.pipe(
  tap((streamData: string) => {
    console.log(`[tap] に入ってきたときの値=${streamData}`);

    // return で 値を返却しても何も返らない
    // 出力結果で map に加工した値が渡っていないことを確認したかったので return で値を返しているが、
    // ここの return　は無くても文法的にエラーにならない
    // 
    // なお 本来 tap は副作用について記すもので､ tap で何かを処理した結果を返すものではない
    // そういうわけで tap で return 文を書くのはコーディングルール等で NG としたほうが良いと思う
    return 'tap で返却した値';
  }),
  map((streamData: string) =>  {
    // tap で加工した値は渡ってこず、streamData$ の初期値に設定した `streamData` が出力される
    console.log(`[map] に入ってきたときの値=${streamData}`);

    // map は戻り値が必要なのでこの return 文は必須
    return `${streamData} + map で加工した値`;
  })
)
.subscribe({
  next: (streamData: string) => {
    receiver$.next(streamData);
  }
});

// Logs:
// receiver=初期値
// [tap] に入ってきたときの値=加工対象の値
// [map] に入ってきたときの値=加工対象の値
// receiver=加工対象の値 + map で加工した値
```

`tap` で行っている返却処理

> ```typescript
> return 'tap で返却した値';
> ```

で返却した値が `map` に渡されていないのがお分かりいただけるでしょうか｡
`tap` の返却値が `map` に渡っているのならば､ `map` で出力するログには

```text
[map] に入ってきたときの値=tap で返却した値
```

と出ているはずです｡
しかし実際に出力されているのは

```text
[map] に入ってきたときの値=加工対象の値
```

です｡
このことから前掲の記述である

> 新たに値を生成して返すことはしません。

が示す動きとなっていることが分かります｡

`map` はストリームで流れてきた値を加工し､新たなストリームを返却するオペレータです｡
`map` で返却された値は `subscribe` に流れて `receiver$.next(streamData);` が実行され､その情報( `streamData` )は

```typescript
receiver$.subscribe((receiver) => {
  // 初期値の購読で1回、 streamData$ の購読で 1回 の 計2回 流れる
  console.log(`receiver=${receiver}`);
});
```

で購読され次のログとなって出力されています｡

```text
receiver=加工対象の値 + map で加工した値
```

ちなみに､サンプルコードの `tap` を `map` に変更すると､ログは以下のように変わります｡

```logs
receiver=初期値
[tap] に入ってきたときの値=加工対象の値
[map] に入ってきたときの値=tap で返却した値
receiver=tap で返却した値 + map で加工した値
```

ご興味あればお試しください｡


## [toArray](https://rxjs.dev/api/operators/toArray)

`toArray` は `complete` が来るまでの `next` を Array に詰めた値として返すオペレータです｡
例によってサンプルコードで具体的な動きを見ていきます｡

```typescript
import { of, take, toArray } from 'rxjs';

const stream$ = of('one', 'two', 'three', 'foure', 'five');
const streamObserver = stream$.pipe(
  // take はストリームから流れてくる値のうち、指定した数( 回数 )だけ処理するフィルタ
  // 指定回数分処理すると complete イベントが発火する
  take(5),

  // complete まで流れてきた値を配列にまとめて返す
  toArray()
);

streamObserver.subscribe({
  next: (value) => console.log(value)
});

// Logs:
// ["one", "two", "three", "foure", …]
// 0: "one"
// 1: "two"
// 2: "three"
// 3: "foure"
// 4: "five"
```

ログには `["one", "two", "three", "foure", 'five']` と出力されています｡
このことから

> `complete` が来るまでの `next` を Array に詰めた値として返す

ことが確認できました｡

なおこのサンプルコードでは `take(5)` で扱う回数を **5回** と指定したので 'one', 'two', 'three', 'foure', 'five' がセットされた配列が出力されました｡
`take(2)` を指定すれば `one`, `two` までがセットされた配列が出力されます｡

:::note info
コード中の [take](https://rxjs.dev/api/operators/take) は [Filtering Operators](https://rxjs.dev/guide/operators#filtering-operators) のオペレータです｡
当該オペレータの詳細についてはリンク先をご参照ください｡
:::

# 参考

- [RxJS Marbles](https://rxmarbles.com/)
- [マーブル図で怖くない RxJS](https://www.slideshare.net/bitbankink/rxjs-159715695)
- [RxJS を学ぼう #2 - よく使う ( と思う ) オペレータ 15 選](https://blog.recruit.co.jp/rmp/front-end/post-11475/)
- [RxJS のオペレーターの動きをデモアプリを自作して確認してみた](<https://note.com/shift_tech/n/n7643a684e947#map()>)
