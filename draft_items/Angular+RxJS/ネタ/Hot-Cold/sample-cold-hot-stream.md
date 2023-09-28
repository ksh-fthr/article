## COLD ストリーム

```typescript
import {
  take,
  interval,
} from 'rxjs';

const streamData$ = interval(1000);

// Observer A
streamData$.pipe(
  take(5),
)
.subscribe(x => console.log(`A: ${x}`));;


// Observer B
setTimeout(() => {
  streamData$.pipe(
    take(5)
  )
  .subscribe(x => console.log(`B: ${x}`));
}, 2500);
```

```bash
A: 0
A: 1
A: 2
#
# A が流れてから 2.5 秒後に B が流れはじめた
# このとき A のストリームに対して干渉していない
# 以降、A と B が交互に処理されている
#
B: 0
A: 3
B: 1
A: 4
B: 2
B: 3
B: 4
```

## HOT ストリーム

### subscribe なしでも流れる

```typescript
import {
  take,
  interval,
  tap,
  share,
  connectable,
  map,
} from 'rxjs';

// Observer A
const streamData$ = connectable(interval(1_000)
.pipe(
  take(5),
  tap(x => console.log(`A: ${x}`)),
));

// これで HOT ストリームにデータが流れる
streamData$.connect();
```

```bash
# このとおり subscribe しなくても tap が処理されている
A: 0
A: 1
A: 2
A: 3
A: 4
```

### 別のストリームに流れる

```typescript
import {
  take,
  interval,
  tap,
  share,
  connectable,
  map,
} from 'rxjs';

// Observer A, オリジナルのストリーム
const streamData$ = connectable(interval(1_000)
.pipe(
  take(5),
  tap(x => console.log(`A: ${x}`)),
  // オリジナルで変更したデータは共有先に影響する
  map(x => x + 20),
  // ↓ Observable B にデータを共有する...と思ったのだが、これがなくても Observable B で購読できる, ???
  share(),
));

// これで HOT ストリームにデータが流れる
streamData$.connect();

// Observer B, 共有されたデータを購読する
setTimeout(() => {
  streamData$
  .pipe(
    // 共有先でありこちらで変更したデータはオリジナルに影響しない
    map(x => x + 10)
  )
  .subscribe(x => console.log(`B: ${x}`));
}, 2500);
```

```bash
A: 0
A: 1
A: 2
#
# A が流れてから 2.5 秒後に B が流れはじめた
# このとき A のストリームは B に分岐するかたちで流れ始める
# 以降、A と B は同じストリームを処理する
#   このとき A で編集したデータは共有さきである B に影響する
#   が、 B で編集したデータはオリジナルの A に影響しない
B: 32
A: 3
B: 33
A: 4
B: 34
```
