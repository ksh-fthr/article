## complete

```typescript
//
// このコードは next(1)~next(3) まで流れる( ログ出力される )
//
import { Observable } from 'rxjs';

const observable = new Observable((subscriber) => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  subscriber.complete();
});

observable.subscribe({
  next: (x) => {
    console.log('got value ' + x);
  },
  error: (error) => {
    console.log(`error: ` + error)
  },
  complete: () => {
    console.log('done');
  },
});


// Logs:
// got value 1
// got value 2
// got value 3
// done
```

```typescript
//
// このコードは next(1)~next(2) まで流れる( ログ出力される )が next(3) は流れない
//
import { Observable } from 'rxjs';

const observable = new Observable((subscriber) => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.complete();

  // complete 後のストリームは流れない
  subscriber.next(3);
});

observable.subscribe({
  next: (x) => {
    console.log('got value ' + x);
  },
  error: (error) => {
    console.log(`error: ` + error)
  },
  complete: () => {
    console.log('done');
  },
});


// Logs:
// got value 1
// got value 2
// done
```
