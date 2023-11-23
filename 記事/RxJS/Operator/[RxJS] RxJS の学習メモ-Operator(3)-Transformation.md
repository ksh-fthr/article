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
- [[RxJS] RxJS の学習メモ-Operator(1)-PipeとFiltering](https://qiita.com/ksh-fthr/items/0f34da2cc38311ecf9c6)
- [[RxJS] RxJS の学習メモ-Operator(2)-CreationとUtility](https://qiita.com/ksh-fthr/items/3f5ecb5bf47ad0216101)

今回も [Operators](https://rxjs.dev/guide/operators) についての続きです。

# 環境

本記事について扱うライブラリや環境の情報です。

|                                        | 備考                                                        |
| -------------------------------------- | ----------------------------------------------------------- |
| [RxJS](https://rxjs.dev/)              | 公式                                                        |
| [Learn RxJS](https://www.learnrxjs.io/) | リファレンス的な感じの学習サイト                        |
| [StackBlitz](https://stackblitz.com/)  | RxJS だけでなく Angular とか React とかの実装お試しができる |

なお本記事執筆時の RxJS のバージョンは StackBlitz の DEPENDENCIES を見ると `v7.8.0` でした。

# この記事でやること

## [Operators](https://rxjs.dev/guide/operators) について触れてみる(3)

前回の記事は [こちら](https://qiita.com/ksh-fthr/items/3f5ecb5bf47ad0216101) です｡
ご興味あればご覧ください。

今回の記事では次のオペレータについて触れていきます｡

**本記事で扱うオペレータ**

- ストリームから新たなストリームに変換する [Transformation Operators](https://rxjs.dev/guide/operators#transformation-operators)

こちらもすべてを扱うのは量的な面で難しいので、個人的によく使うメソッドに対する理解を深めたいと思います。

# [Transformation Operators](https://rxjs.dev/guide/operators#transformation-operators)

## [map](https://rxjs.dev/api/index/function/map)

:::note info
**`map` の特徴**

- ストリームで流れてきたデータに対して同期的に処理を行う
- そして処理を行った結果の値を返し、それが `subscribe` に流れていく
- イテレータで言うところの `map` をイメージするとわかりやすい
:::

```typescript
import {
  of,
  BehaviorSubject,
} from 'rxjs';

import {
  map,
} from 'rxjs/operators';

// ストリームを受信する受け皿
const receiver$ = new BehaviorSubject<string>('初期値');

// 配信するストリーム
const streamData$ = of('streamData');

receiver$.subscribe((receiver) => {
  console.log(`receiver=${receiver}`);
});

// 文字列を返す例をみる
streamData$
.pipe(
  map((streamData: string) =>  {
    console.log(`map に入ってきたときの値-1回目=${streamData}`);
    streamData = `${streamData} に文字列を加える.`
    return streamData;
  })
)
.subscribe((streamData: string) => {
  // receiver$ には `streamData に文字列を加える.` が流れていく
  receiver$.next(streamData);
});

// 配列を返す例をみる
streamData$
.pipe(
  map((streamData: string) =>  {
    console.log(`map に入ってきたときの値-2回目=${streamData}`);
    return [
      '文字列ではなく',
      '配列や Object を返すこともできる.'
    ];
  })
)
.subscribe((streamData: string[]) => {
  // 配列を連結して receiver$ に流す
  // receiver$ には `文字列ではなく 配列や Object を返すこともできる.` が流れていく
  receiver$.next(streamData.join(' '));
});
```

この処理を実行すると下記が出力されます。

```log
receiver=初期値
map に入ってきたときの値-1回目=streamData
receiver=streamData に文字列を加える.
map に入ってきたときの値-2回目=streamData
receiver=文字列ではなく 配列や Object を返すこともできる.
```

出力された内容から `receriver$` に `next` した際に流れたストリームが同期的に処理されていること、また `map` は `string` でも `配列` でも `Object` でも返せることが確認できました。

## [mergeMap](https://rxjs.dev/api/index/function/mergeMap)

:::note info
**`mergeMap` の特徴**

- ストリームで流れてきたデータに対して同期的に処理を行う。ここは `map` と変わらない
  - `map` との違いは **新しくストリームを生成するか否か**
  - `map` は流れてきた値を加工するが、新たにストリームを生成することはない
- **`mergeMap` は流れてきたデータをもとに新たなストリームを生成する**。(これは後述の **`switchMap`** や **`concatMap`** も同じ)
- 新しく生成したストリームは、それを対象に `map` で処理しても良いし、そのまま `subscribe` に流すことも出来る
- `mergeMap` は流れてきた **ストリームの同期処理が片付いた順** に次の処理にデータが流れる
:::

```typescript
import { BehaviorSubject, Observable, of } from 'rxjs';
import { delay, mergeMap } from 'rxjs/operators';

interface StreamData {
  sentence: string;
  delay: number;
}

// 初期データなので一番最初に出力される
const receiver$ = new BehaviorSubject<StreamData>({
  sentence: '最初に流すストリーム',
  delay: 0,
});

receiver$
  .pipe(
    mergeMap((streamData: StreamData, index: number) => {
      return delayProcess$(index, streamData.sentence, streamData.delay)
    }),
  )
  .subscribe({
    next: (value: string): void => {
      console.log(value);
    }
  }
);

// receiver$ に対してストリームを流す
// receiver$　内の mergeMap では新たなストリームを生成している
//
// で、 mergeMap は処理が終わった順に流れるので、このケースは delay の小さな順に処理が実行される
receiver$.next({ sentence: '最後に流れるストリーム', delay: 500 }); // 初期データが流れたあと、最後に出力される
receiver$.next({ sentence: '3番目に流れるストリーム', delay: 300 }); // 初期データが流れたあと、3番めに出力される
receiver$.next({ sentence: '2番目に流れるストリーム', delay: 100 }); // 初期データが流れたあと、2番めに出力される

function delayProcess$(index: number, sentence: string, delayTime: number): Observable<string> {
  return of(`[${index}] resultData: ${sentence} -> resolved`).pipe(
    // delay は指定した時間( msec )遅延してくれる
    delay(delayTime)
  )
}
```

この処理を実行すると下記が出力されます。

```log
[0] resultData: 最初に流すストリーム -> resolved
[3] resultData: 2番目に流れるストリーム -> resolved
[2] resultData: 3番目に流れるストリーム -> resolved
[1] resultData: 最後に流れるストリーム -> resolved
```

`[]` 内の数値は実行順序です。
実行順序と出力結果から、 `next` で流れたストリームは指定した遅延時間にそって処理されていることがわかります。
つまり

> - `mergeMap` は流れてきた **ストリームの同期処理が片付いた順** に次の処理にデータが流れる

の動きをこのサンプルコードから確認できました。

## [switchMap](https://rxjs.dev/api/index/function/switchMap)

:::note info
**`switchMap` の特徴**

- 大まかな部分は `mergeMap` と一緒
- `switchMap` は **最後に処理されたストリームだけを対象に処理** をして次の処理にデータが流れる
- つまり **最後に流れてくるストリームの前に処理されていた内容はキャンセル** される
:::

```typescript
import { BehaviorSubject, Observable, of } from 'rxjs';
import { delay, switchMap } from 'rxjs/operators';

interface StreamData {
  sentence: string;
  delay: number;
}

// 初期データなので一番最初に出力される
const receiver$ = new BehaviorSubject<StreamData>({
  sentence: '最初に流すストリーム',
  delay: 0,
});

receiver$
  .pipe(
    switchMap((streamData: StreamData, index: number) => {
      return delayProcess$(index, streamData.sentence, streamData.delay)
    }),
  )
  .subscribe({
    next: (value: string): void => {
      console.log(value);
    }
  }
);

// receiver$ に対してストリームを流す
// receiver$　内の switchMap では新たなストリームを生成している
//
// で、 switchMap は最後に流れてきたストリームだけを処理するので、 3番目の next だけが出力される
receiver$.next({ sentence: 'このストリームは出力されない', delay: 500 }); // 出力されない
receiver$.next({ sentence: 'このストリームは出力されない', delay: 300 }); // 出力されない
receiver$.next({ sentence: 'このストリームだけが出力される', delay: 100 }); // 出力される

function delayProcess$(index: number, sentence: string, delayTime: number): Observable<string> {
  return of(`[${index}] resultData: ${sentence} -> resolved`).pipe(
    // delay は指定した時間( msec )遅延してくれる
    delay(delayTime)
  )
}
```

この処理を実行すると下記が出力されます。

```log
[3] resultData: このストリームだけが出力される -> resolved
```

`[]` 内の数値は実行順序です。
実行順序と出力結果から、最後に実行された `next` のストリームだけが処理されていることがわかります。
というわけで

> - `switchMap` は **最後に処理されたストリームだけを対象に処理** をして次の処理にデータが流れる
> - つまり **最後に流れてくるストリームの前に処理されていた内容はキャンセル** される

動きをこのサンプルコードから確認できました。

## [cancatMap](https://rxjs.dev/api/index/function/concatMap)

:::note info
**`concatMap` の特徴**

- こちらも大まかな部分は `mergeMap` と一緒
- `concatMap` は **ストリームが流れてきた 順** に処理を行い次の処理にデータが流れる
:::

```typescript
import { BehaviorSubject, Observable, of } from 'rxjs';
import { delay, concatMap } from 'rxjs/operators';

interface StreamData {
  sentence: string;
  delay: number;
}

// 初期データなので一番最初に出力される
const receiver$ = new BehaviorSubject<StreamData>({
  sentence: '最初に流すストリーム',
  delay: 0,
});

receiver$
  .pipe(
    concatMap((streamData: StreamData, index: number) => {
      return delayProcess$(index, streamData.sentence, streamData.delay)
    }),
  )
  .subscribe({
    next: (value: string): void => {
      console.log(value);
    }
  }
);

// receiver$ に対してストリームを流す
// receiver$　内の concatMap では新たなストリームを生成している
//
// で、 concatMap はストリームが流れた順に処理されるので、 delay の大小に関係なく上から出力される
receiver$.next({ sentence: '2番目に流れるストリーム', delay: 500 }); // 初期データが流れたあと、2番目に出力される
receiver$.next({ sentence: '3番目に流れるストリーム', delay: 300 }); // 初期データが流れたあと、3番目に出力される
receiver$.next({ sentence: '最後に流れるストリーム', delay: 100 }); // 初期データが流れたあと、最後に出力される

function delayProcess$(index: number, sentence: string, delayTime: number): Observable<string> {
  return of(`[${index}] resultData: ${sentence} -> resolved`).pipe(
    // delay は指定した時間( msec )遅延してくれる
    delay(delayTime)
  )
}
```

この処理を実行すると下記が出力されます。

```log
[0] resultData: 最初に流すストリーム -> resolved
[1] resultData: 2番目に流れるストリーム -> resolved
[2] resultData: 3番目に流れるストリーム -> resolved
[3] resultData: 最後に流れるストリーム -> resolved
```

`[]` 内の数値は実行順序です。
実行順序と出力結果から、`next` の実行順にストリームが処理されていることがわかります。
というわけで

> - `concatMap` は **ストリームが流れてきた 順** に処理を行い次の処理にデータが流れる

動きをこのサンプルコードから確認できました。

## 補足

:::note info
コード中の [delay](https://rxjs.dev/api/operators/delay) は [Utility Operators](https://rxjs.dev/guide/operators#utility-operators) のオペレータです｡
当該オペレータの詳細についてはリンク先をご参照ください｡
:::

# 参考

- [RxJS Marbles](https://rxmarbles.com/)
- [マーブル図で怖くない RxJS](https://www.slideshare.net/bitbankink/rxjs-159715695)
- [RxJS の concatMap, mergeMap, switchMap の違いを理解する(中級者向け)](https://qiita.com/ovrmrw/items/b45d7bf29c8d29415bd7)
- [RxJS map と mergeMap の違い](https://zenn.dev/shrek13/articles/rxjs-map-mergemap)
- [RxJS のオペレーターの動きをデモアプリを自作して確認してみた](<https://note.com/shift_tech/n/n7643a684e947#map()>)
- [RxJS を学ぼう #2 - よく使う ( と思う ) オペレータ 15 選](https://blog.recruit.co.jp/rmp/front-end/post-11475/)
- [【RxJS】指定した秒数遅らせてデータを処理](https://kakkoyakakko2.hatenablog.com/entry/2018/06/02/003000)
- [Angular-RxJS ライブラリ](https://angular.jp/guide/rx-library#rxjs-%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA)
- [Angular のための RxJS](https://learn-rxjs-for-angular.info/)
