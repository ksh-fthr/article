# はじめに

掲題の環境を構築する必要があったので、その際に行った手順を備忘録として残します。
未来の自分と類似案件でお悩みの方の一助となれば幸いです。

# 注意
本記事は 2020年6月21日 時点の情報です｡
ご覧になられた時点で UI が変更されている可能性がありますので､その点ご注意ください｡

# 前提

- AWS を利用する
- AWS のアカウントを持っている
- 以下のサービスに対するアクセス権限を持っているユーザで作業する
  - MediaLive
  - MediaStore
  - CloudFront(今回は必要ないが別記事で CloudFront で CDN を実現する予定なので入れておく)
  - IAM
  - CloudWatch

# 環境

| サービス                                                     | 概要                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| macOS                                                        | 10.15.x                                                      |
| [Elemental MediaLive](https://aws.amazon.com/jp/medialive/) | あらゆるデバイスへのブロードキャストおよびストリーミング向けにライブ動画をエンコードする |
| [Elemental MediaStore ](https://aws.amazon.com/jp/mediastore/) | ライブストリーミングによるメディアワークフロー向けにビデオアセットを保存、配信する |
| [OBS](https://obsproject.com/ja)                             | ビデオ録画と生放送用の無料でオープンソースのソフトウェア。   |

# 1.MediaStoreのリソース作成

## 1.1.コンテナの作成
最初は何も作成されていないので､作成から始める｡

### 1.1.1.コンテナの新規作成
1. MediaService のトップページからコンテナ名を入力して `Create container` ボタンをクリック
<img width="1120" alt="スクリーンショット 2020-06-07 8.59.06.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/b606c6e7-39dc-46c7-568b-28a7cd2fe37b.png">

2. 作成中の画面に遷移するので待つ
<img width="1627" alt="スクリーンショット 2020-06-07 8.59.49.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/570dc72d-edb3-8b01-54dc-103f829549df.png">

3. コンテナの作成完了
<img width="1629" alt="スクリーンショット 2020-06-07 9.00.24.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/a10223af-80cb-f4a4-eef9-f3312533b040.png">


## 1.2.コンテナの設定
コンテナの作成が完了したので､今度は設定を行っていく｡

### 1.2.1.コンテナポリシーの設定
[コンテナポリシーの例: HTTPS 経由のパブリック読み取りアクセス](https://docs.aws.amazon.com/ja_jp/mediastore/latest/ug/policies-examples-public-https.html) をもとに設定する｡
設定する内容は

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadOverHttps",
      "Effect": "Allow",
      "Action": ["mediastore:GetObject", "mediastore:DescribeObject"],
      "Principal": "*",
      "Resource": "arn:aws:mediastore:<region>:<owner acct number>:container/<container name>/*",
      "Condition": {
        "Bool": {
            "aws:SecureTransport": "true"
        }
      }
    }
  ]
}
```

で､このうち以下の 2 つ

- `"Sid": "PublicReadOverHttps"`
- `"Resource": "arn:aws:mediastore:<region>:<owner acct number>:container/<container name>/*"`

の部分は､コンテナ作成時に元々設定されていた値をそのまま使うようにした｡
特に後者の `Resource` については､自分自身の環境を指定する必要がある(コピペするだけで完了ではないので注意)

▼設定後の画面
<img width="1514" alt="スクリーンショット 2020-06-07 9.05.57.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/3cc4c214-6233-964c-0fd9-983b7f23c6e9.png">


### 1.2.2.コンテナ CORS ポリシーの設定
異なるドメインから MediaStore リソースの読み込を許可するため､以下を設定しておく｡

```json
[
  {
    "AllowedHeaders": [
      "*"
    ],
    "AllowedMethods": [
      "GET",
      "HEAD"
    ],
    "AllowedOrigins": [
      "*"
    ],
    "MaxAgeSeconds": 3000
  }
]
```

▼設定後の画面
<img width="1545" alt="スクリーンショット 2020-06-07 12.59.58.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/7c835562-d405-b56a-ecc5-460bf56078ae.png">


## 1.3.エンドポイントをメモする
ここまでで `MediaStore` の作成は完了｡
あとで `MediaLive` と連携で必要となるので､ `Data Endpoint` をメモしておく｡
<img width="1512" alt="スクリーンショット 2020-06-07 9.11.46.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/040011ff-fcf0-b1d4-6c0f-5e66e28595bc.png">


# 2.MediaLiveのリソース作成
## 2.1.チャンネルの作成
1. MediaLive のトップページからコンテナ名を入力して `Create channel` ボタンをクリック
<img width="1336" alt="スクリーンショット 2020-06-07 9.26.30.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/6e578b6a-6b1e-a05b-5487-1c9187080af5.png">

## 2.2.Channel and input details の設定
### 2.2.1.Channel template の選択
ここでは `Live Event HLS` を選択
<img width="793" alt="スクリーンショット 2020-06-07 9.30.23.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/cb6ae817-03db-aeea-eb29-253b136d5468.png">

### 2.2.2.General info の設定
#### 2.2.2.1.テンプレートからロールを生成
`Create role from template` を選択して `Create IAM role` ボタンをクリック｡
このとき `Channel name` も忘れずに入力しておく｡
<img width="789" alt="スクリーンショット 2020-06-07 9.30.59.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/bfdb6912-7f64-8f9d-dc8b-f32369f8d5f9.png">

#### 2.2.2.2.生成されたロールを選択
`Use existin role` を選択､また `Remember role` にチェックする
<img width="925" alt="スクリーンショット 2020-06-07 12.46.53.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/2df341f7-0750-c188-dcc3-88fb840e7934.png">

### 2.2.3.Channel and input details と Input specifications
ここはいじらずにそのまま｡
<img width="789" alt="スクリーンショット 2020-06-07 9.32.40.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/4346dec4-66b4-8573-6ce4-8906fe3e974d.png">

## 2.3.Input attachments の設定
最初は何も作成されていないので､作成から始める｡

### 2.3.1.Input の新規作成
`Add` ボタンをクリックすると `Attach input` が表示されるので､`Create Input` から作成する｡
<img width="1133" alt="スクリーンショット 2020-06-07 9.33.54.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/8b71d32c-d48f-8fa6-7116-d69c4a7dcc00.png">

### 2.3.2.Input details の設定
`Input name` の入力と `Input type` に `RTMP(push)` を選択する｡
<img width="915" alt="スクリーンショット 2020-06-07 9.35.05.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/d7b77912-79aa-b130-d590-6839c9d6380f.png">

### 2.3.3.Network mode の設定
ここはいじらずにそのまま｡
<img width="771" alt="スクリーンショット 2020-06-07 13.40.13.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/da324b89-b3c6-7dcd-68b8-fbc2311d73df.png">

### 2.3.4.Input security group の設定
最初は何も作成されていないので､`Create` から作成する｡
このときテキストエリアにアクセスを許可するIPアドレスとサブネットマスクを設定する｡
<img width="768" alt="スクリーンショット 2020-06-07 13.43.08.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/82542cef-00a5-fec9-585d-872626de61b5.png">

### 2.3.5.Input destinations の設定
ここで入力した値が後述の [2.4.2.HLS group destination A/B の設定](#242hls-group-destination-ab-の設定) で使用される｡
ここでは `Destination A`, `Destination B` にそれぞれ `live` と `Sample` を入力している｡
(あくまで本手順における入力値なので､実際の状況に合わせて値は変えて OK )
<img width="892" alt="スクリーンショット 2020-06-07 14.29.49.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/406403d7-d6bc-529e-949d-1844f6aa07aa.png">


### 2.3.6.作成
キャプチャは用意していないが､上記設定が完了したら画面下部にある `Create` ボタンをクリックして作成する｡


## 2.4.Output groups の設定
最初は何も作成されていないので､作成から始める｡

### 2.4.1.Output groups の新規作成
`Add` ボタンをクリックすると `Add ouput groups` が表示されるので､`HLS` を選択して `Confirm` をクリックする｡
<img width="1317" alt="スクリーンショット 2020-06-07 13.49.46.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/b52ac4d5-10f4-7485-cf44-6a702603fe26.png">

### 2.4.2.HLS group destination A/B の設定
[MediaStore のエンドポイント](#13エンドポイントをメモする) をここで設定する｡
<img width="945" alt="スクリーンショット 2020-06-07 10.05.24.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/38c0fce7-174c-3230-08f7-7d4bdffd08f8.png">

このとき後述の [2.4.3.HLS settings の設定](#243hls-settings-の設定) で `Hls media store` を選択する関係上､スキーマは

| 変更前 | 変更後 |
| ----- | ----- |
|`https` |`mediastoressl`|

に変更している｡
また `A` と `B` でユニークにする必要があるので､

| destination | URLの区別 |
| ----------- | -------- |
|destination A| Sample-A |
|destination B| Sample-B |

としている｡
ここで `live/Sample-A` や `live/Sample-B` としているのは､ [2.3.5.Input destinations の設定](#235input-destinations-の設定) で設定した値｡


### 2.4.3.HLS settings の設定
`Hls media store` を選択し､あとはデフォルト値のまま｡
<img width="917" alt="スクリーンショット 2020-06-07 13.56.19.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/8a03f9cc-c07c-4d2e-8924-6372c6316421.png">
<img width="939" alt="スクリーンショット 2020-06-07 9.57.44.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/4afcc2f5-1a5a-f148-f859-caaf4e20c03e.png">

### 2.4.4.それ以外の設定
他の設定はいじらずに､そのままとする｡

## 2.5.チャンネルの作成
画面左の `Create channel` をクリックしチャンネルを作成する｡
<img width="382" alt="スクリーンショット 2020-06-07 9.58.01.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/49e989a4-300c-7f78-85b7-6c652c7faaa1.png">


# 3.配信を試す
## 3.1.OB を使用する
動画配信のクライアントには [OBS](https://obsproject.com/ja) を使用する｡
リンクからダウンロードとインストールを行う｡

## 3.2.動画配信の設定
動画ファイルは任意のものを用意し､以下の設定を行う｡
<img width="980" alt="スクリーンショット 2020-06-07 12.44.51.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/c24ead02-d4cc-6784-fd0e-ef276e3680c2.png">
ここで設定するサーバのアドレスとストリームキーは [2.3.Input attachments の設定](#23input-attachments-の設定) で作成した Input の詳細画面から確認できる下記を元に設定する｡

<img width="1320" alt="スクリーンショット 2020-06-07 15.06.26.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/93947495-bb76-4ba9-c020-6ab5434a8214.png">

| サーバ | ストリームキー |
|-------|-----------|
|rtmp:xxx.xxx.xxx.xxx:1935/live|Sample|

## 3.3.MedilaLive の Channel をスタート
作成したチャンネルを選択した状態で画面上部の `Start` をクリック｡ `Status` が `Running` になるまで待つ｡
<img width="1640" alt="スクリーンショット 2020-06-07 15.15.54.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/8871352c-5576-7dd7-532e-0f97a1a6a092.png">
<img width="1640" alt="スクリーンショット 2020-06-07 15.17.15.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/cecd50ad-5902-6113-8599-50280092bbdc.png">

## 3.5.動画配信開始
OBS から｢動画配信開始｣をクリックして動画配信を開始する｡
<img width="1076" alt="スクリーンショット 2020-06-07 15.20.55.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/1909e9b3-1dd6-57b2-5d2e-4554c252065f.png">

## 3.4.MediaStore のコンテナを確認
### 3.3.1.フォルダの確認
画面右上の `Refresh` ボタンを何度かクリックすると､フォルダが出来上がる｡
<img width="1550" alt="スクリーンショット 2020-06-07 12.45.13.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/b42f2637-3439-9d07-a16a-25fb98e22273.png">
<img width="1556" alt="スクリーンショット 2020-06-07 12.48.50.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/927cd92b-cdbe-da9b-3ad0-1e24741d4cb8.png">

### 3.3.2.配信ファイルの確認
フォルダの中にはいると､配信ファイルが配置されている｡
<img width="1568" alt="スクリーンショット 2020-06-07 12.49.01.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/2cadd009-ce41-ffe8-8533-edacaaf7bc81.png">

このなかから､拡張子 `m3u8` のものをクリック -> 詳細画面を表示する
<img width="1568" alt="スクリーンショット 2020-06-07 15.28.32.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/8373c77d-77e3-494d-39ba-aa1bcbc9ed93.png">

### 3.3.3.URLをSafariで開く
Object nameのURLをコピーしSafariなどのHLS再生できるブラウザで開いて再生確認ができれば成功｡

<img width="1266" alt="スクリーンショット 2020-06-07 12.49.56.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/9590c0ca-0577-a81d-ca6c-ac87a89bf423.png">

# まとめにかえて
できれば CloudFron を利用して次の図のような構成を試してみたかったが､かなり時間が掛かったし記事の内容も長くなったので一旦ここで終了する｡
CloudFront を使った構成は ~~別に記事を作る｡~~  [こちら](https://qiita.com/ksh-fthr/items/ee2b7880a9e6c0fc6f88) をご参考いただければ｡

▼CloudFront を用いた構成
(mermaid.js の `classDiagram` で無理くり書いた構成図なので､ツッコミはご勘弁ください)
<img width="974" alt="スクリーンショット 2020-06-07 15.53.02.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/e903d8bc-6e7f-5a73-c70a-ea7f1bb613df.png">

# MediaLive の料金について
ちょっと気になる一文があったので｡
気をつけよう｡
[AWS Elemental MediaLive の料金](https://aws.amazon.com/jp/medialive/pricing/)
> <img width="703" alt="スクリーンショット 2020-06-07 16.04.15.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/7f141de5-a59a-e056-dcb9-9b9163e81406.png">


# 参考

- [【やってみた】AWS Elemental MediaLiveとAWS Elemental MediaStoreでライブ配信してみた](https://dev.classmethod.jp/articles/reinvent2017-awselemental-medialive-mediastore-livestreaming/)
- [コンテナポリシーの例: HTTPS 経由のパブリック読み取りアクセス](https://docs.aws.amazon.com/ja_jp/mediastore/latest/ug/policies-examples-public-https.html)

