## サンプルコード

```typescript
import {
  BehaviorSubject
} from 'rxjs';

import { 
  filter
} from 'rxjs/operators'

const receiver$ = new BehaviorSubject<string>('初期値');

receiver$
.pipe(
  filter((stream: string) => stream === '初期値' ),
)
.subscribe((receiver) => {
  // fileter で `初期値` だけ流すようにしているので `初期値` だけが出力される
  console.log(`receiver=${receiver}`);
});

receiver$.next('2回目')
receiver$.next('3回目')
```

```bash
receiver=初期値
```

## filter を外すと

```typescript
import {
  BehaviorSubject
} from 'rxjs';

import { 
  filter
} from 'rxjs/operators'

const receiver$ = new BehaviorSubject<string>('初期値');

receiver$
// .pipe(
//   filter((stream: string) => stream === '初期値' ),
// )
.subscribe((receiver) => {
  // fileter で `初期値` だけ流すようにしているので `初期値` だけが出力される
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
