## サンプルコード

[mergeMap]() のコードから `switchMap` に変えただけのコード。
`mergeMap` と同じく [こちらのコード](https://qiita.com/ovrmrw/items/b45d7bf29c8d29415bd7#concatmap-mergemap-switchmap%E3%81%AE%E9%81%95%E3%81%84) が分かりやすかったので参照させて頂いた。
( 説明しやすいように、また自分が理解しやすいように一部改変している )

```typescript
import {
  BehaviorSubject,
} from 'rxjs';

import {
  mergeMap,
  concatMap,
  switchMap,
} from 'rxjs/operators';


interface StreamData {
  url: string;
  delay: number;
}

const initialData: StreamData = {
  url: 'http://initial',
  delay: 0,
};

const receiver$ = new BehaviorSubject<StreamData>(initialData); // switchMap を用いた本例では出力されない

receiver$
  .pipe(
    switchMap((streamData: StreamData) => {
      return httpGet(streamData.url, streamData.delay)
    }),
  )
  // 関数: httpGet の結果が流れてくる
  .subscribe((value: string) => {
    console.log(value)
  }
);

// receiver$ に対してストリームを流す
// receiver$　内の switchMap では httpGet を呼び出して新たなストリームを生成している
//
// で、 switchMap は最後に流れてきたストリームだけを処理するので、 `stream-3` が出力される
receiver$.next({ url: 'http://stream-1', delay: 500 }); // 出力されない
receiver$.next({ url: 'http://stream-2', delay: 300 }); // 出力されない
receiver$.next({ url: 'http://stream-3', delay: 100 }); // 出力される

function httpGet(url, delay) {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve(`resultData: ${url} -> resolved`)
    }, delay)
  })
}
```

```bash
resultData: http://stream-3 -> resolved
```
