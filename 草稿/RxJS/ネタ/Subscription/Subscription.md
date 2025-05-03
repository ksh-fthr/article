## [subscribe](https://rxjs.dev/api/index/function/subscribeOn)

- データの購読
- Observable で観察していたデータを購読することで、流れてきたデータに対してアレコレする
- そのデータを加工して別のストリームに流したり、そのデータを別の変数から参照させて同期的に扱うようにしたり、とか色々
- [サンプルコードはこちら](./sample-subscribe.md)

## unsubscribe

- ストリームを **完了** させるための仕組み
- `unsubscribe()` 後にストリームを流そうとしても流れない
- `unsubscribe()` を実行しても購読している側の `subscribe` で `complete` のハンドラが実行されない
- [サンプルコードはこちら](./sample-unsubscribe.md)

## complete

- ストリームを **完了** させるための仕組み
- `complete()` 後にストリームを流そうとしても流れない
- `complete()` を実行すると購読している側の `subscribe` で `complete` のハンドラが実行される
- [サンプルコードはこちら](./sample-complete.md)


# サンプルコード

## [Subscription](https://rxjs.dev/guide/subscription)

```typescript
import { interval } from 'rxjs';

const observable = interval(1000);
const subscription = observable.subscribe(x => console.log(x));
// Later:
// This cancels the ongoing Observable execution which
// was started by calling subscribe with an Observer.
subscription.unsubscribe();
```

```typescript
import { interval } from 'rxjs';
 
const observable1 = interval(400);
const observable2 = interval(300);
 
const subscription = observable1.subscribe(x => console.log('first: ' + x));
const childSubscription = observable2.subscribe(x => console.log('second: ' + x));
 
subscription.add(childSubscription);
 
setTimeout(() => {
  // Unsubscribes BOTH subscription and childSubscription
  subscription.unsubscribe();
}, 1000);
```
