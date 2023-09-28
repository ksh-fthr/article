## サンプルコード

```typescript
import { of, take, toArray } from 'rxjs';


const stream$ = of('one', 'two', 'three', 'foure', 'five');
const streamObserver = stream$.pipe(
  // ストリームから流れてくる値のうち、指定した数( 回数 )だけ処理するフィルタ
  // 指定回数分処理すると complete イベントが発火する
  take(5),

  // complete まで流れてきた値を配列にまとめて返す
  toArray()
);

streamObserver.subscribe(value => console.log(value));
```

```bash
["one", "two", "three", "foure", …]
0: "one"
1: "two"
2: "three"
3: "foure"
4: "five"
```

## 補足

### サンプルコードの出力結果について

```
- take(5) で 5回 指定したので 'one', 'two', 'three', 'foure', 'five' がセットされた配列が出力される
- take(2) を指定すれば `one`, `two` までが出力される
```
