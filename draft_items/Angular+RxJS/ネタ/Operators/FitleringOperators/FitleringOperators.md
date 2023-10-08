# [Filtering Operators](https://rxjs.dev/guide/operators#filtering-operators)
  
## [first](https://rxjs.dev/api/operators/first)

- ストリームが流れてきたとき **最初の値だけ** を次の処理に回す
- [サンプルコードはこちら](./sample-first.md

## [filter](https://rxjs.dev/api/index/function/filter)

- ストリームが流れてきたとき 条件に合致した値を次の処理に回す
- [サンプルコードはこちら](./sample-filter.md)

## [distinctUntilChanged](https://rxjs.dev/api/index/function/distinctUntilChanged)

- ストリームが流れてきたとき `comparator` に指定した条件に合致した値だけを処理する
- `comparator` は省略可能
- デフォルトの動きは 「**前回流れてきたデータと異なる場合** に処理する」
- [サンプルコードはこちら](./sample-distinctUntilChanged.md)

# 補足

## Angular の HttpClientModule

Angular では Http クライアントの実装で [HttpClientModule](https://angular.jp/api/common/http/HttpClientModule) に含まれる [HttpClient](https://angular.jp/api/common/http/HttpClient) を利用することが多いと思います。
この HttpClient でも RxJS を利用していますので、それについて補足します。

以下、公式からの転載です。

( 転載もと: [HttpClient のメソッドはひとつの値を返す](https://angular.jp/tutorial/toh-pt6#httpclient%E3%81%AE%E3%83%A1%E3%82%BD%E3%83%83%E3%83%89%E3%81%AF%E3%81%B2%E3%81%A8%E3%81%A4%E3%81%AE%E5%80%A4%E3%82%92%E8%BF%94%E3%81%99) )

> ```text
> HTTPはリクエスト/レスポンスプロトコルです。 リクエストを送信すると、ひとつのレスポンスを返却します。
>
> 一般には、Observableは時間によって複数の値を返すことが 可能 です。 HttpClientが返すObservableは常にひとつの値を発行してから完了するので、再び値を発行することはありません。
> ```

ということで、`HttpClient` を用いた場合、`first()` 等でストリームが流れる回数に対して制限を掛ける必要はありません。
上記を更に補足しますと、 `subscribe` には次のイベントが用意されています。
それぞれ

- `next`
  - 複数回流れる
  - データを伴って流れる
- `error`
  - 一回だけ流れる
  - データを伴って流れる
- `complete`
  - 一回だけ流れる
  - データを伴って流れない

というものですが、Angular の `HttpClient` では **値が取得できたら `next`, 次に `complete` を流し** ます。
ということは、`HttpClient` で流れてきたストリームに対して `first` を使っても、(`complete` が流れてきているので) その前後で動きが変わることがない、となります。

つまり `HttpCliente` 使うときに `first` は **使っても使わなくても結果は変わらず**、よって  **使う必要がない イコール 不要**、となります。

# 参考

- [RxJS Marbles](https://rxmarbles.com/)
- [マーブル図で怖くない RxJS](https://www.slideshare.net/bitbankink/rxjs-159715695)
- [RxJS の concatMap, mergeMap, switchMap の違いを理解する(中級者向け)](https://qiita.com/ovrmrw/items/b45d7bf29c8d29415bd7)
- [RxJS map と mergeMap の違い](https://zenn.dev/shrek13/articles/rxjs-map-mergemap)
- [RxJS のオペレーターの動きをデモアプリを自作して確認してみた](<https://note.com/shift_tech/n/n7643a684e947#map()>)
- [CombineLatest vs withLatestFrom](https://medium.com/@vinothinikings/combinelatest-vs-withlatestfrom-5003377b766f)
- [RxJS を学ぼう #2 - よく使う ( と思う ) オペレータ 15 選](https://blog.recruit.co.jp/rmp/front-end/post-11475/)
- [RxJS を学ぼう #4 - COLD と HOT について学ぶ / ConnectableObservable](https://blog.recruit.co.jp/rmp/front-end/post-11558/)
- [【Rxjs のすゝめ】Observable の COLD と HOT って結局なんなの？](https://deep.tacoskingdom.com/blog/25)
- [Angular-RxJS ライブラリ](https://angular.jp/guide/rx-library#rxjs-%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA)
- [Angular のための RxJS](https://learn-rxjs-for-angular.info/)
