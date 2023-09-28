## サンプルコード

### 数値を複数指定したケース

```typescript
import { of } from 'rxjs';

of(1, 2, 3).subscribe((x) => console.log(`stream=${x}`));
```

```bash
stream=1
stream=2
stream=3
```

### 配列を指定したケース

```typescript
import { of } from 'rxjs';

of([1, 2, 3]).subscribe((x) => console.log(`stream=${x}`));
```

```bash
stream=1,2,3
```

### オブジェクトを指定したケース

```typescript
import { of } from 'rxjs';

of({'id': 1, 'name': 'hoge', 'birthday': '2020/04/08'}).subscribe((x) => console.log(`stream=${JSON.stringify(x)}`));
```

```bash
stream={"id":1,"name":"hoge","birthday":"2020/04/08"}
```

## schduler を指定したい場合

### 非推奨となった書き方

- 前掲のソースコードの `of` の最終引数に scheduler を指定する
- 例
  
  ```typescript
  import { of, asyncScheduler } from 'rxjs';
  ```

of(1, 2, 3, asyncScheduler).subscribe((x) => console.log(`stream=${x}`));

```
### 今後の書き方
#### 数値を複数指定したケース
```typescript
import { asyncScheduler, scheduled } from 'rxjs';

// 配列で指定する
scheduled([1, 2, 3], asyncScheduler).subscribe((x) => console.log(`stream=${x}`));
```

```bash
stream=1
stream=2
stream=3
```

#### 配列を指定したケース

```typescript
import { asyncScheduler, scheduled } from 'rxjs';

// 配列の配列を指定する
scheduled([[1, 2, 3]], asyncScheduler).subscribe((x) => console.log(`stream=${x}`));
```

```bash
stream=1,2,3
```

#### オブジェクトを指定したケース

```typescript
import { asyncScheduler, scheduled } from 'rxjs';

// 配列の中にオブジェクトを指定する
scheduled([{'id': 1, 'name': 'hoge', 'birthday': '2020/04/08'}], asyncScheduler).subscribe((x) => console.log(`stream=${JSON.stringify(x)}`));
```

```bash
stream={"id":1,"name":"hoge","birthday":"2020/04/08"}
```
