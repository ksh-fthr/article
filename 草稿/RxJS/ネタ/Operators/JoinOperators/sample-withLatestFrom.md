## サンプルコード

### トリガーとなる Observable に値が流れていない場合

```typescript
import {
  BehaviorSubject,
  Subject,
} from 'rxjs';

import {
  map,
  withLatestFrom
} from 'rxjs/operators';

const streamData1$ = new Subject(); // 初期データが登録されないのでストリームが流れない
const streamData2$ = new BehaviorSubject<string>('初期データ');

streamData1$.pipe(
  withLatestFrom(streamData2$),
  map(([streamData1, streamData2]) => {
    console.log(`streamData1=${streamData1}`);
    console.log(`streamData2=${streamData2}`);

    return 'withLatestFrom の処理終了.'
  })
)
.subscribe({
  next: (message: string) => {
    console.log(message);
  }
});

// streamData1$.next('streamData1$ を更新'); // トリガーに値を流さない
streamData2$.next('streamData2$ を更新');


// streamData1$.next('streamData1$ を再更新'); // トリガーに値を流さない
streamData2$.next('streamData2$ を再更新');
```

```bash
# なにも出力されない
# これはトリガーである streamData1$ にデータが流れていないため
# 流れてくるものが無いから動きようがない
# streamData2$ はトリガーではないので、こいつにデータを流しても処理されない
```

## トリガーとなる Observable に値を流した場合(1回だけ流す)

```typescript
import {
  BehaviorSubject,
  Subject,
} from 'rxjs';

import {
  map,
  withLatestFrom
} from 'rxjs/operators';

const streamData1$ = new Subject();
const streamData2$ = new BehaviorSubject<string>('初期データ');


streamData1$.pipe(
  withLatestFrom(streamData2$),
  map(([streamData1, streamData2]) => {
    console.log(`streamData1=${streamData1}`);
    console.log(`streamData2=${streamData2}`);

    return 'withLatestFrom の処理終了.'
  })
)
.subscribe({
  next: (message: string) => {
    console.log(message);
  }
});

streamData1$.next('streamData1$ を更新(1回目)'); // 1回目を流す
streamData2$.next('streamData2$ を更新(1回目)');


// streamData1$.next('streamData1$ を再更新(2回目)'); // 2回目は流さない
streamData2$.next('streamData2$ を再更新(2回目)');
```

```bash
# streamData1$ に値を流した1回目だけデータが処理される
# streamData2$ の情報は「初期データ」しか流れていないことに注目
# 前の例でも書いたが「streamData2$ はトリガーではないので、こいつにデータを流しても処理されない」ことを理解する
streamData1=streamData1$ を更新
streamData2=初期データ
withLatestFrom の処理終了.
```

## トリガーとなる Observable に値を流した場合(2回目も流す)

```typescript
import {
  BehaviorSubject,
  Subject,
} from 'rxjs';

import {
  map,
  withLatestFrom
} from 'rxjs/operators';

const streamData1$ = new Subject();
const streamData2$ = new BehaviorSubject<string>('初期データ');


streamData1$.pipe(
  withLatestFrom(streamData2$),
  map(([streamData1, streamData2]) => {
    console.log(`streamData1=${streamData1}`);
    console.log(`streamData2=${streamData2}`);

    return 'withLatestFrom の処理終了.'
  })
)
.subscribe({
  next: (message: string) => {
    console.log(message);
  }
});

streamData1$.next('streamData1$ を更新(1回目)'); // 1回目を流す
streamData2$.next('streamData2$ を更新(1回目)');


streamData1$.next('streamData1$ を再更新(2回目)'); // 2回目は流さない
streamData2$.next('streamData2$ を再更新(2回目)');
```

```bash
# streamData1$ に値を流した1回目と2回目の両方でデータが処理される
# streamData2$ のデータに注目
# - 1回目は「初期データ」
# - 2回目は「streamData2$ を更新(1回目)」
# これは streamData2$ の更新した値が streamData1$ の更新に遅れて流れてきていることが原因
# 1. streamData1$ の1回目が流れる -> 付随して streamData2$ の初期データが処理される
# 2. streamData1$ の2回めが流れる -> 今度は streamData2$ の更新データ(1回目) が処理される
# 3. streamData1$ の処理はもう流れない -> streamData2$ の更新データ(2回目) は処理されないまま終わる
streamData1=streamData1$ を更新(1回目)
streamData2=初期データ
withLatestFrom の処理終了.
streamData1=streamData1$ を再更新(2回目)
streamData2=streamData2$ を更新(1回目)
withLatestFrom の処理終了.
```
