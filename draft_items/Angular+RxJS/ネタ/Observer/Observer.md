# サンプルコード

## [Observer](https://rxjs.dev/guide/observer)

- Observable によって配信されたストリームを受けてアレコレするための受け皿

( 上記リンクから転載 )
> **What is an Observer?** An Observer is a consumer of values delivered by an Observable. Observers are simply a set of callbacks, one for each type of notification delivered by the Observable: `next`, `error`, and `complete`. The following is an example of a typical Observer object:

( Deepl による翻訳 )
> Observerとは何ですか？Observerとは、Observableによって配信される値の消費者のことです。Observerはコールバックのセットで、Observableによって配信される通知の各タイプ（next、error、complete）に対して1つずつあります。以下は、典型的なObserverオブジェクトの例です

`Observable` は次のインターフェースを持ちます。

```typescript
const observer = {
  next: x => console.log('Observer got a next value: ' + x),
  error: err => console.error('Observer got an error: ' + err),
  complete: () => console.log('Observer got a complete notification'),
};
```

# 参考

- [RxJS における Subject の活用]( https://zenn.dev/mikakane/articles/rxjs_5_subject)
- [RxJS を学ぼう #5 - Subject について学ぶ / Observable × Observer](https://blog.recruit.co.jp/rmp/front-end/post-11951/)
