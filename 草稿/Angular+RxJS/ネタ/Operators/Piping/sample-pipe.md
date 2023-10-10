
- `subscribe` で購読する前に、`observable` で観察していたデータを加工するための **繋ぎ** を実現するためのオペレータ
- この中でストリームで流れているデータに対する操作やフィルタリングを行う


## サンプルコード

`map` の中で配列化しただけの単純例だが雰囲気はつかめるはず。

```typescript
import {
  of,
  BehaviorSubject,
} from 'rxjs';

import {
  map,
} from 'rxjs/operators';

const receiver$ = new BehaviorSubject<string>('初期値');
const streamData$ = of('streamData');

receiver$.subscribe((receiver) => {
  // 初期値の購読で1回、 streamData$ の購読で 1回 の 計2回 流れる
  console.log(`receiver=${receiver}`);
});

streamData$
.pipe(
  map((streamData: string) => [streamData, 'map で加工した', '値が流れる'])
)
.subscribe((streamData: string[]) => {
  receiver$.next(`${streamData[0]} を ${streamData[1]} ${streamData[2]}.`);
});
```

```bash
receiver=初期値
receiver=streamData を map で加工した 値が流れる.
```
