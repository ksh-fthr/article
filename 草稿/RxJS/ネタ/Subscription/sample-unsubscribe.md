## unsubscribe
 
```typescript
import { Observable } from 'rxjs';

const observable = new Observable((subscriber) => {
  setTimeout(() => {
    // unsubscribe 後のストリームは流れない
    subscriber.next(3);
  }, 1_500)

  subscriber.next(1);
  subscriber.next(2);
});

const subscription = observable.subscribe({
  next: (x) => {
    console.log('got value ' + x);
  },
  error: (error) => {
    console.log(`error: ` + error)
  },
  complete: () => {
    // unsubscribe の場合 complete は処理されない
    console.log('done');
  },
});

setTimeout(() => {
  // ここをコメントアウトすると 1.5秒後 に next(3) が流れる
  subscription.unsubscribe();
}, 1_000)


// Logs:
// got value 1
// got value 2

```