## サンプルコード

### 配列を一つ指定

```typescript
import { from } from 'rxjs';

from([1, 2, 3]).subscribe((x) => console.log(`stream=${x}`));
```

```bash
# 配列の要素毎にストリームが流れているのがわかる
stream=1
stream=2
stream=3
```

### 配列を複数指定

```typescript
import { from } from 'rxjs';

// このやり方はエラーになる
// -> Error: scheduler.schedule is not a function
// from([1, 2, 3], [4, 5, 6]).subscribe((x) => console.log(`stream=${x}`));

// 配列の中に配列を含める
from([[1, 2, 3], [4, 5, 6]]).subscribe((x) => console.log(`stream=${x}`));
```

```bash
# 配列の要素毎にストリームが流れているのがわかる
stream=1,2,3
stream=4,5,6
```

## schduler を指定したい場合

### 非推奨となった書き方

- 前掲のソースコードの `from` の最終引数に scheduler を指定する
- 例
  
  ```typescript
  import { from, asyncScheduler } from 'rxjs';
  ```

from([1, 2, 3], asyncScheduler).subscribe((x) => console.log(`stream=${x}`));

### 今後の書き方

#### 配列を一つ指定
[of の こちら]() と一緒。

```typescript
import { asyncScheduler, scheduled } from 'rxjs';

scheduled([1,2,3], asyncScheduler).subscribe((x) => console.log(`stream=${x}`));
```

```bash
stream=1
stream=2
stream=3
```

#### 配列を複数指定

[of の こちら]() と一緒。

```typescript
import { asyncScheduler, scheduled } from 'rxjs';

// 配列の配列を指定する
scheduled([[1,2,3], [4,5,6]], asyncScheduler).subscribe((x) => console.log(`stream=${x}`));
```

```bash
stream=1,2,3
stream=4,5,6
```
