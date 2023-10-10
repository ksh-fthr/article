## サンプルコード

### complete を投げない場合

```typescript
import {
  Subject,
  combineLatest,
} from 'rxjs';

import {
  map
} from 'rxjs/operators';

const streamData1$ = new Subject();
const streamData2$ = new Subject();

combineLatest([streamData1$, streamData2$])
.pipe(
  map(([streamData1, streamData2]) => {
    console.log(`streamData1=${streamData1}`);
    console.log(`streamData2=${streamData2}`);

    return 'combineLatest の処理終了.'
  })
)
.subscribe({
  next: (message: string) => {
    console.log(message);
  }
});

streamData1$.next('streamData1$ に流す(1回目)');
streamData2$.next('streamData2$ に流す(1回目)');

streamData1$.next('streamData1$ に流す(2回目)');
streamData2$.next('streamData2$ に流す(2回目)');
```

```bash
# 1回目
# -> StreamData$1, StreamData$2 ともにデータが流れてきた
streamData1=streamData1$ に流す(1回目)
streamData2=streamData2$ に流す(1回目)
combineLatest の処理終了.
#
# 2回目-1
# -> StreamData$1 にデータが流れてきた( 2回目 )
# -> StreamData$2 にはまだデータが流れていない( 1回目のまま )
streamData1=streamData1$ に流す(2回目)
streamData2=streamData2$ に流す(1回目)
combineLatest の処理終了.
#
# 2回目-2
# -> StreamData$1 は直前のデータがそのまま
# -> StreamData$2 にデータが流れてきた( 2回目 )
streamData1=streamData1$ に流す(2回目)
streamData2=streamData2$ に流す(2回目)
combineLatest の処理終了.
```

### complete を投げた場合( streamData1$ だけに copmlete 投げる )

```typescript
import {
  Subject,
  combineLatest,
} from 'rxjs';

import {
  map
} from 'rxjs/operators';

const streamData1$ = new Subject();
const streamData2$ = new Subject();

combineLatest([streamData1$, streamData2$])
.pipe(
  map(([streamData1, streamData2]) => {
    console.log(`streamData1=${streamData1}`);
    console.log(`streamData2=${streamData2}`);

    return 'combineLatest の処理終了.'
  })
)
.subscribe({
  next: (message: string) => {
    console.log(message);
  }
});

streamData1$.next('streamData1$ に流す(1回目)');
// complete を投げてストリームを終了する
streamData1$.complete();
streamData2$.next('streamData2$ に流す(1回目)');

streamData1$.next('streamData1$ に流す(2回目)'); // complete でストリームが終了しているので流れない
streamData2$.next('streamData2$ に流す(2回目)'); // こちらはデータが流れる
```

```bash
# 1回目
# -> StreamData$1, StreamData$2 ともにデータが流れてきた
# -> ( これは complete が実行される前なので前掲のサンプルコードと同じ動きとなる )
streamData1=streamData1$ に流す(1回目)
streamData2=streamData2$ に流す(1回目)
combineLatest の処理終了.
#
# 2回目
# -> StreamData$1 にはデータが流れてこない,
#    complete によってストリームが終了しているために 1回目 のデータのまま
# -> StreamData$2 にはデータが流れてきた( 2回目 )
streamData1=streamData1$ に流す(1回目)
streamData2=streamData2$ に流す(2回目)
combineLatest の処理終了.
```

### complete を投げた場合( streamData2$ にも copmlete 投げる )

```typescript
import {
  Subject,
  combineLatest,
} from 'rxjs';

import {
  map
} from 'rxjs/operators';

const streamData1$ = new Subject();
const streamData2$ = new Subject();

combineLatest([streamData1$, streamData2$])
.pipe(
  map(([streamData1, streamData2]) => {
    console.log(`streamData1=${streamData1}`);
    console.log(`streamData2=${streamData2}`);

    return 'combineLatest の処理終了.'
  })
)
.subscribe({
  next: (message: string) => {
    console.log(message);
  }
});

streamData1$.next('streamData1$ に流す(1回目)');
// complete を投げてストリームを終了する
streamData1$.complete();

streamData2$.next('streamData2$ に流す(1回目)');
// complete を投げてストリームを終了する
streamData2$.complete();


streamData1$.next('streamData1$ に流す(2回目)'); // complete でストリームが終了しているので流れない
streamData2$.next('streamData2$ に流す(2回目)'); // complete でストリームが終了しているので流れない
```

```bash
# 1回目
# -> StreamData$1, StreamData$2 ともにデータが流れてきた
# -> ( これは complete が実行される前なので前掲のサンプルコードと同じ動きとなる )
streamData1=streamData1$ に流す(1回目)
streamData2=streamData2$ に流す(1回目)
combineLatest の処理終了.
#
# 2回目のデータは流れてこない
# -> StreamData1$, StreamData$2 ともに complete でストリームが終了しているため
```

## combineLatestで監視対象のストリームでcompleteが発火したときのイメージ<img width="2964" alt="combineLatestでcompleteが発火したときのイメージ図.png (359.2 kB)" src="https://img.esa.io/uploads/production/attachments/17638/2022/12/02/102830/46dfe244-2e57-4a1a-b932-04ca8029f0fb.png">
