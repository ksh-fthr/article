## サンプルコード

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
