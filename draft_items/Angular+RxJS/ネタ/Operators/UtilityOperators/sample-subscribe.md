## サンプルコード

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
