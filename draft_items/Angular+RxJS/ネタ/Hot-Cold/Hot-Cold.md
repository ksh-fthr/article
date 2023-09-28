
## COLD ストリーム

- 特性-1
  - `subscribe` で購読しない限りストリームは流れてこない ===> 処理できない
- 特性-2
  - ストリームは個別に流れる
- COLD となるのはどういうときか
  - 通常、利用しているのはこちら
  - `BehaviorSubject`m `Subject`, `Observable` 等のクラスのインスタンスを処理するとこちらになる
- [サンプルコードはこちら](./sample-cold-hot-stream.md#cold-ストリーム)

## HOT ストリーム

- 特性-1
  - `subscribe` しなくてもストリームが流れる
- 特性-2
  - ストリームは分岐して同じ値がそれぞれのストリームに流れる
- HOT となるのはどういうときか
  - `connectable` クラスのインスタンスを生成し, `connect` オペレータを実行するとこちらになる
- 注意-非推奨オペレータについて
  - HOT ストリームに関する記事でサンプルコードとして頻出する [publish](https://rxjs.dev/api/operators/publish) や [refCount](https://rxjs.dev/api/operators/refCount) は非推奨となった
  - 同じく [ConnectableObservable](https://rxjs.dev/api/index/class/ConnectableObservable) クラスも非推奨となった
  - これらは v8 で廃止予定
  - 今後は [share](https://rxjs.dev/api/operators/share) と [Connectable](https://rxjs.dev/api/index/interface/Connectable) を使用する
    - ( 本項で紹介するサンプルコードはこちらを使用している )
  - API の置き換えについては [Multicasting](https://rxjs.dev/deprecations/multicasting) を参照
- [サンプルコードはこちら](./sample-cold-hot-stream.md#hot-ストリーム)