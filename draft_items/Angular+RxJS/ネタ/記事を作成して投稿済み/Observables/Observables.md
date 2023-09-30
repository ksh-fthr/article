## [Observable](https://rxjs.dev/guide/observable)

( 上記リンクから転載 )
> Observables are lazy Push collections of multiple values. They fill the missing spot in the following table:

( Deepl による翻訳 )
> Observablesは、複数の値を集めたLazy Pushの集合体です。以下の表の欠落している部分を埋めるものです：

なにを言っているかわからないのでサンプルコードを見ていきます。

### サンプルコード

```typescript
import { Observable } from 'rxjs';

// 次のコードは **消費者たる observer** と **配信者たる observable** を同時に定義している
// - observable => 消費者
// - new Observable() => 配信者
const observable = new Observable((subscriber) => {
  // next(1)~next(3) は同期的に実行される
  // ( 公式サイトの説明では `push` という単語で説明されているので、本コードでも以下、同じ用語で説明する )
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);

  // next(4)は実行後から 1秒後 に push される
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});

// Observable は **subscribe** されることでストリームが購読されるので、
// このコードの処理ではまず最初にこのログが出力される
console.log('just before subscribe');

// **subscribe** によるストリームの購読を行う
observable.subscribe({
  // 成功時: 流れてきたストリームを出力する
  next(x) {
    console.log('got value ' + x);
  },
  // エラー処理
  error(err) {
    console.error('something wrong occurred: ' + err);
  },
  // 完了処理: **complete** が実行されるとこのハンドラが実行される
  complete() {
    console.log('done');
  },
});
console.log('just after subscribe');


// Logs:
// just before subscribe
// got value 1           // subscribe によって出力されたログ
// got value 2           // 同上
// got value 3           // 同上
// just after subscribe
// got value 4           // subscribe によって出力されたログ
// done                  // complete によって出力されたログ
```

上記コードの `next(1)` ~ `nex(4)` の間に `subscriber.error()` や `subscriber.complete()` を入れるとログの出方が変わります。

**pull  と push について**
本項のはじめに `pull` と `push` の表を示しています。公式サイトでは それに関する [説明の項目](https://rxjs.dev/guide/observable#pull-versus-push) がありますが、ここでは割愛します。

# 参考

- [RxJS における Subject の活用]( https://zenn.dev/mikakane/articles/rxjs_5_subject)
- [RxJS を学ぼう #5 - Subject について学ぶ / Observable × Observer](https://blog.recruit.co.jp/rmp/front-end/post-11951/)
