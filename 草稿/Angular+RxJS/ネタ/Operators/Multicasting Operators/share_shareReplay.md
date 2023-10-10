 [shareReplay](https://rxjs.dev/api/index/function/shareReplay) と [share](https://rxjs.dev/api/index/function/share) はともに [multicasting](https://rxjs.dev/deprecations/multicasting) に属するオペレータです。

そして `shareReplay` には multicasting のページに次の説明があります。

> And [shareReplay](https://rxjs.dev/api/operators/shareReplay) \- which is a thin wrapper around the now highly-configurable [share](https://rxjs.dev/api/operators/share) operator.
> (deepl で翻訳)
> また、ShareReplayは、現在では高度に設定可能なシェアオペレーターを薄く包んだものである。

本項では 先に [share](https://rxjs.dev/api/operators/share) が提供する機能について確認し、その後に [shareReplay](https://rxjs.dev/api/operators/shareReplay) を見ることで両者が提供する機能の違いを確認します。

## 用語
`share` と `shareReplay` の説明には **マルチキャスト** という単語がでてきます。
ここでは **マルチキャスト** と、その対となる **ユニキャスト** について触れておきます。

- **マルチキャスト**
  - 購読している `Observer` が一つの `Observable` **実行を共有** する
- **ユニキャスト**
  - 購読している `Observer` がそれぞれ `Observable` の **独立した実行を所有** する

## サンプルコード
###  [share](https://rxjs.dev/api/operators/share)

公式の説明を転載します。

> Returns a new Observable that multicasts (shares) the original Observable. As long as there is at least one Subscriber this Observable will be subscribed and emitting data. When all subscribers have unsubscribed it will unsubscribe from the source Observable. Because the Observable is multicasting it makes the stream `hot`. This is an alias for `[multicast](https://rxjs.dev/api/index/function/multicast)(() => new [Subject](https://rxjs.dev/api/index/class/Subject)()), refCount()`.
---
> (Deepl で翻訳)
元のObservableをマルチキャスト（共有）する新しいObservableを返します。少なくとも1人のSubscriberが存在する限り、このObservableは購読され、データを発信します。すべての購読者が購読を解除すると、ソースObservableから購読を解除する。Observableがマルチキャストであるため、ストリームがホットな状態になります。これは、multicast(() => new Subject())、refCount()のエイリアスです。

```typescript
import { interval, tap, map, take, share, shareReplay } from 'rxjs';

// オリジナル
// インターバル 2秒 x take(5) で 10秒間 ストリームが流れる
const source$ = interval(2_000)
.pipe(
  tap(x => {
    console.log('▼▼▼▼▼▼')
    console.log('Processing: ', x)
  }),
  take(5),
  share() // これの有無でストリームの流れ方が変わる
);

// 購読-1
source$.subscribe(x => console.log('subscription 1'));

// 購読-2
source$.subscribe(x => console.log('subscription 2'));

// 購読-3
// 11秒後に購読を開始する
setTimeout(() => {
  source$.subscribe(y => console.log('subscription 3'));
}, 11_000);
```

```bash
# -----------------------
# share をつけなかった場合のログ
# -----------------------

# １０秒間　オリジナルと各購読のログが交互に 合計5回 出る
#
# １回目
▼▼▼▼▼▼
Processing: 0
subscription 1
▼▼▼▼▼▼
Processing: 0
subscription 2
#
# 2回め
▼▼▼▼▼▼
Processing: 1
subscription 1
▼▼▼▼▼▼
Processing: 1
subscription 2
#
# 3回め
▼▼▼▼▼▼
Processing: 2
subscription 1
▼▼▼▼▼▼
Processing: 2
subscription 2
#
# 4回め
▼▼▼▼▼▼
Processing: 3
subscription 1
▼▼▼▼▼▼
Processing: 3
subscription 2
#
# 5回め
▼▼▼▼▼▼
Processing: 4
subscription 1
▼▼▼▼▼▼
Processing: 4
subscription 2
#
# そのあと、11秒後 に 購読-3 のログが 5回 流れる
▼▼▼▼▼▼
Processing: 0
subscription 3
▼▼▼▼▼▼
Processing: 1
subscription 3
▼▼▼▼▼▼
Processing: 2
subscription 3
▼▼▼▼▼▼
Processing: 3
subscription 3
▼▼▼▼▼▼
Processing: 4
subscription 3
```

```bash
# -----------------------
# share をつけた場合のログ
# -----------------------

# １０秒間　オリジナルが流れたあと、購読-1, 2 のログが綺麗に並ぶ
#
# 1回め
▼▼▼▼▼▼
Processing: 0
subscription 1
subscription 2
#
# 2回め
▼▼▼▼▼▼
Processing: 1
subscription 1
subscription 2
#
# 3回め
▼▼▼▼▼▼
Processing: 2
subscription 1
subscription 2
#
# 4回め
▼▼▼▼▼▼
Processing: 3
subscription 1
subscription 2
#
# 5回め
▼▼▼▼▼▼
Processing: 4
subscription 1
subscription 2
#
# 実行から 11秒後 に 購読-3 のログが 5回 流れる
▼▼▼▼▼▼
Processing: 0
subscription 3
▼▼▼▼▼▼
Processing: 1
subscription 3
▼▼▼▼▼▼
Processing: 2
subscription 3
▼▼▼▼▼▼
Processing: 3
subscription 3
▼▼▼▼▼▼
Processing: 4
subscription 3
```

また `share` には引数にオプションを取ることが出来ます。(省略可能)
オプションについては こちら ( [ShareConfig](https://rxjs.dev/api/operators/ShareConfig) ) をご参照ください。


### [shareReplay](https://rxjs.dev/api/operators/shareReplay) 
公式の説明を転載します。

> Share source and replay specified number of emissions on subscription.
---
> (Deepl で翻訳)
ソースを共有し、指定された数のエミッションを再生することができます。

何を言っているかわかりませんが、サンプルコードを見ていきます。
サンプルコードは `share` → `shareReplay` に置き換えているだけで、ほかはすべて同じです。

```typescript
import { interval, tap, map, take, share, shareReplay } from 'rxjs';

// オリジナル
// インターバル 2秒 x take(5) で 10秒間 ストリームが流れる
const source$ = interval(2_000)
.pipe(
  tap(x => {
    console.log('▼▼▼▼▼▼')
    console.log('Processing: ', x)
  }),
  take(5),
  shareReplay(3)
);

// 購読-1
source$.subscribe(x => console.log('subscription 1'));

// 購読-2
source$.subscribe(x => console.log('subscription 2'));

// 購読-3
// 11秒後に購読を開始する
setTimeout(() => {
  source$.subscribe(y => console.log('subscription 3'));
}, 11_000);
```

```bash
# -----------------------
# shareReplay のログ
# -----------------------

# オリジナルが流れたあと、購読-1, 2 のログが綺麗に並ぶ
# ここは share と同じ動き
▼▼▼▼▼▼
Processing: 0
subscription 1
subscription 2
▼▼▼▼▼▼
Processing: 1
subscription 1
subscription 2
▼▼▼▼▼▼
Processing: 2
subscription 1
subscription 2
▼▼▼▼▼▼
Processing: 3
subscription 1
subscription 2

# 実行から 11秒後、 購読-3 が動く
# その時点で「最後に実行されたストリームのキャッシュ」を購読している
▼▼▼▼▼▼
Processing: 4
subscription 1
subscription 2
subscription 3
subscription 3
subscription 3
```

ということで、

> ソースを共有し、指定された数のエミッションを再生することができます。

`shareReplay` の引数で指定した回数分、キャッシュが効いていることが確認できました。


## share vs shareReplay

`share` と `shareReplay` はともに multicast するオペレータですが、サンプルコードで見たように挙動が異なります。

どちらも同じように一つの stream が 購読-1, 2 の subscribe で同時に購読されてますが、購読-3 の挙動が異なります。

- `share` による 購読-3 の挙動
    - `setTimeout` によってオリジナルのストリームが **改めて** 流れてきます
    - そのストリームを 購読-3 で購読してます
- `shareReplay` による購読-3 の挙動
    - `setTimeout` 時にオリジナルのストリームは流れていません
    - 購読-3 ではオリジナルの **最後に流れたストリームのキャッシュ** を購読してます

すいません。`share vs shareReplay` と大仰な見出しをつけましたが、どちらを使うのが良いのか、明確に回答をだせません。
ここでは上記のとおり両者の挙動を理解することで、既存コードが意図する動きも理解できるようになることを着地点とさせてください。

## 参考
- [Demystifying RxJS: 'share' vs 'shareReplay' operators](https://www.linkedin.com/pulse/demystifying-rxjs-share-vs-sharereplay-operators-tomas-mikula)
- [Always Know When to Use Share vs. ShareReplay](https://www.bitovi.com/blog/always-know-when-to-use-share-vs.-sharereplay)


