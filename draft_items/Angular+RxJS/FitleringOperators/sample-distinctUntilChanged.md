## サンプルコード

### 同じ値が連続しているケース

```typescript
import {
  of,
  distinctUntilChanged,
} from 'rxjs';

const streamData$ = of('同じ', '同じ', 'データ', 'データ', 'は', 'は', '流れない', '流れない');

streamData$
.pipe(
  distinctUntilChanged()
)
.subscribe((streamData) => {
  // 「同じデータは流れない」と出力される
  console.log(`receiver=${streamData}`);
});
```

```bash
receiver=同じ
receiver=データ
receiver=は
receiver=流れない
```

### 同じ値ではあるが連続していないケース

```typescript
import {
  of,
  distinctUntilChanged,
} from 'rxjs';

const streamData$ = of('同じ', '同じ', 'データ', 'だけど', '連続していないから', '同じ', '値でも', '流れる', 'は', '流れる',);

streamData$
.pipe(
  distinctUntilChanged()
)
.subscribe((streamData) => {
  // 「同じデータでも連続していないから流れるは流れる」と出力される
  console.log(`receiver=${streamData}`);
});
```

```bash
receiver=同じ
receiver=データ
receiver=だけど
receiver=連続していないから
receiver=同じ
receiver=値でも
receiver=流れる
receiver=は
receiver=流れる
```

## comparator  で条件指定もできる

`comparator` については [こちら](https://rxjs.dev/api/index/function/distinctUntilChanged#comparator)。

### 前後で違うものは出力しない

```typescript
import {
  of,
  distinctUntilChanged,
} from 'rxjs';

const streamData$ = of('同じ', '同じ', 'データ', 'だけど', '連続していないから', '同じ', '値でも', '流れる', 'は', '流れる',);

streamData$
.pipe(
  distinctUntilChanged((prev, current) => {
    return prev !== current // 前後で違うものは出力しない
  })
)
.subscribe((streamData) => {
  // 「同じ同じ同じ」と出力される
  console.log(`receiver=${streamData}`);
});
```

```bash
receiver=同じ
receiver=同じ
receiver=同じ
```

### 前後で違うものを出力する( 通常と同じうごき )

```typescript
import {
  of,
  distinctUntilChanged,
} from 'rxjs';

const streamData$ = of('同じ', '同じ', 'データ', 'だけど', '連続していないから', '同じ', '値でも', '流れる', 'は', '流れる',);

streamData$
.pipe(
  distinctUntilChanged((prev, current) => {
    return prev === current // 前後で違うものは出力しない
  })
)
.subscribe((streamData) => {
  // 「同じデータでも連続していないから流れるは流れる」と出力される
  console.log(`receiver=${streamData}`);
});
```

```bash
receiver=同じ
receiver=データ
receiver=だけど
receiver=連続していないから
receiver=同じ
receiver=値でも
receiver=流れる
receiver=は
receiver=流れる
```
