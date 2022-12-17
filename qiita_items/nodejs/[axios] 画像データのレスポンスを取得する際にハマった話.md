axios を使って画像データ(PNG画像)を取得する REST-API を叩いた際に画像データが期待通りに取得できなくてハマったのでメモを残す。

# どういう状況か
前述の通り、[axios](https://github.com/axios/axios) を使って画像取得の REST-API を実行～画像取得、それにプラスして base64 エンコードした文字列をフロントエンドに送ることが目的。
構成と一連の流れは次のような感じ。

* 構成

    ```:構成
    フロントエンド ⇔ バックエンドA ⇔ バックエンドB
    ```
* 流れ
  1. フロントエンドからバックエンドAに対して画像取得のAPIを実行
  1. バックエンドAは仲介人のような立ち位置で、バックエンドBに対して画像取得のAPIを実行。ここで axios を使う
  1. バックエンドBからバックエンドAに画像が response で返る
  1. バックエンドAは base64 エンコードしてフロントエンドへ返す
  1. フロントエンドは base64 エンコードされて返却された文字列を dataURL で表示する

で、バックエンドBからバックエンドAに返った response にはバイナリデータっぽい文字化けした文字列があって、これを一生懸命 base64 エンコードしていた。

ここで、この **「バイナリデータっぽい文字化けした文字列」が response で返ってきているのがおかしい**、と気づけなかったのがハマった原因。

# 返却される画像データは arraybuffer が正しい
ということで、結論を書くと axios のインスタンスを create する際に設定するパラメータに問題があった。

以下に駄目だったケースと期待通りに動いたケースを示すが、responseType と Content-Type に json を指定したのが誤り。画像データを取得する際は、それぞれ **arryabuffer** と (PNG画像の場合は) **image/png** を指定する必要があった。

## 画像取得が正常に行えなかったケース

```javascript:設定するパラメータ
var instance = axios.create({
  'responseType': 'json',
  'headers': {
    'Content-Type': 'application/json'
  }
});
```

そのときのシーケンスがこちら。
<img width="804" alt="axios_get_image_ng.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/5f632b40-f16e-8acb-c09b-cb019927d0da.png">


## 画像取得が正常に行えたケース

```javascript:設定するパラメータ
var instance = axios.create({
  'responseType': 'arraybuffer',
  'headers': {
    'Content-Type': 'image/png'
  }
});
```

そのときのシーケンスがこちら。
<img width="804" alt="axios_get_image_ok.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/db73dbfb-2b70-98ac-569d-9e7282a69a99.png">


# どうして気づくのが遅れたか
自分自身の知識/経験不足と、しつこいようだが、前掲の「バイナリデータっぽい文字化けした文字列」が返って来ていたのが原因。
「バイナリデータを取得しているんだから、そりゃあ文字化けしてるよね」と思い込んでいた。
なので、base64 エンコードをする際の実装に問題があるとばかり(ここでも)思い込んで、そっちの処理の見直しと試行錯誤ばかりを繰り返していた。

実際は base64 エンコードの処理は問題なくて、画像取得の REST-API を叩く際に axios に設定するパラメータが間違っていた、という話。

# おまけ
base64 エンコードの処理はググればいくらでも見つかるけれども、下記のような感じ。

```javascript:base64エンコードする処理
/**
 * 画像データを base64 エンコード
 *
 * @param {array} imgData
 */
function base64Encode(imgData) {
  // arraybuffer で渡された imgData を base64 エンコードする
  const base64Encoded = imgData.toString('base64');
  return base64Encoded;
}
```
