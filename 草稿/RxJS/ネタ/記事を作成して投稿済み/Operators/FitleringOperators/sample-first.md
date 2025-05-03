## サンプルコード

```typescript
import {
  BehaviorSubject
} from 'rxjs';

import { 
  first
} from 'rxjs/operators'

const receiver$ = new BehaviorSubject<string>('初期値');

receiver$
.pipe(
  first(),
)
.subscribe((receiver) => {
  // first で 最初の値だけ 流すようにしているので `初期値` だけが出力される
  console.log(`receiver=${receiver}`);
});

receiver$.next('2回目')
receiver$.next('3回目')
```

```bash
receiver=初期値
```

## fist を外すと

```typescript
import {
  BehaviorSubject
} from 'rxjs';

import { 
  first
} from 'rxjs/operators'

const receiver$ = new BehaviorSubject<string>('初期値');

receiver$
// .pipe(
//   first(),
// )
.subscribe((receiver) => {
  // first で 最初の値だけ 流すようにしているので `初期値` だけが出力される
  console.log(`receiver=${receiver}`);
});

receiver$.next('2回目')
receiver$.next('3回目')
```

```bash
receiver=初期値
receiver=2回目
receiver=3回目
```

# 補足

## Angular の HttpClientModule

Angular では Http クライアントの実装で [HttpClientModule](https://angular.jp/api/common/http/HttpClientModule) に含まれる [HttpClient](https://angular.jp/api/common/http/HttpClient) を利用することが多いと思います。
この HttpClient でも RxJS を利用していますので、それについて補足します。

以下、公式からの転載です。

( 転載もと: [HttpClient のメソッドはひとつの値を返す](https://angular.jp/tutorial/toh-pt6#httpclient%E3%81%AE%E3%83%A1%E3%82%BD%E3%83%83%E3%83%89%E3%81%AF%E3%81%B2%E3%81%A8%E3%81%A4%E3%81%AE%E5%80%A4%E3%82%92%E8%BF%94%E3%81%99) )

> ```text
> HTTPはリクエスト/レスポンスプロトコルです。 リクエストを送信すると、ひとつのレスポンスを返却します。
>
> 一般には、Observableは時間によって複数の値を返すことが 可能 です。 HttpClientが返すObservableは常にひとつの値を発行してから完了するので、再び値を発行することはありません。
> ```

ということで、`HttpClient` を用いた場合、`first()` 等でストリームが流れる回数に対して制限を掛ける必要はありません。
上記を更に補足しますと、 `subscribe` には次のイベントが用意されています。
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

というものですが、Angular の `HttpClient` では **値が取得できたら `next`, 次に `complete` を流し** ます。
ということは、`HttpClient` で流れてきたストリームに対して `first` を使っても、(`complete` が流れてきているので) その前後で動きが変わることがない、となります。

つまり `HttpCliente` 使うときに `first` は **使っても使わなくても結果は変わらず**、よって  **使う必要がない イコール 不要**、となります。
