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
