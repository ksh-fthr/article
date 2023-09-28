## サンプルコードの前に

- 参照
  - [pipe のサンプルコード]()
  - [tap のサンプルコード]()

## サンプルコード

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
  console.log(`receiver=${receiver}`);
});

// 文字列を返す例をみる
streamData$
.pipe(
  map((streamData: string) =>  {
    console.log(`map に入ってきたときの値-1回目=${streamData}`);

    streamData = `初期値は全く別の値で上書きして値を返す.`
    return streamData;
  })
)
.subscribe((streamData: string) => {
  receiver$.next(streamData);
});

// 配列を返す例をみる
streamData$
.pipe(
  map((streamData: string) =>  {
    console.log(`map に入ってきたときの値-2回目=${streamData}`);
    console.log(`(蛇足) 2回目の streamData$ に対する処理だが、 1回目 の処理で streamData$ に対して next　で値の更新は行っていないので ↑ のログには初期値が出力されているはず`);

    return [
      'map では',
      '文字列ではなく',
      '配列や Object を返すこともできる.'
    ];
  })
)
.subscribe((streamData: string[]) => {
  receiver$.next(streamData.join(' ')); // 配列を連結して receiver$ に流す
});
```

```bash
receiver=初期値
map に入ってきたときの値-1回目=streamData
receiver=初期値は全く別の値で上書きして値を返す.
map に入ってきたときの値-2回目=streamData
(蛇足) 2回目の streamData$ に対する処理だが、 1回目 の処理で streamData$ に対して next　で値の更新は行っていないので ↑ のログには初期値が出力されているはず
receiver=map では 文字列ではなく 配列や Object を返すこともできる.
```
