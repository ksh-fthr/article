# はじめに

[こちらの記事](https://qiita.com/ksh-fthr/items/64d7137409f3199557fd) の ｢[まとめにかえて](https://qiita.com/ksh-fthr/items/64d7137409f3199557fd#%E3%81%BE%E3%81%A8%E3%82%81%E3%81%AB%E3%81%8B%E3%81%88%E3%81%A6)｣で触れたとおり､ CDN として [CloudFront](https://aws.amazon.com/jp/cloudfront/) を利用した構成での Live 動画配信の構成を作っていく｡

# 注意
本記事は 2020年6月21日 時点の情報です｡
ご覧になられた時点で UI が変更されている可能性がありますので､その点ご注意ください｡

# 前提

- [こちらの記事](https://qiita.com/ksh-fthr/items/64d7137409f3199557fd) の構築が完了していること

# 環境

| サービス                                                     | 概要                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| macOS                                                        | 10.15.x                                                      |
| [Elemental MediaLive](https://aws.amazon.com/jp/medialive/) | あらゆるデバイスへのブロードキャストおよびストリーミング向けにライブ動画をエンコードする |
| [Elemental MediaStore ](https://aws.amazon.com/jp/mediastore/) | ライブストリーミングによるメディアワークフロー向けにビデオアセットを保存、配信する |
| [CloudFront](https://aws.amazon.com/jp/cloudfront/)   | 高速で安全性が高くプログラム可能なコンテンツ配信ネットワーク (CDN、content delivery network) |
| [OBS](https://obsproject.com/ja)                             | ビデオ録画と生放送用の無料でオープンソースのソフトウェア。   |

# CloudFront の設定
※ CloudFront は｢グローバル｣リージョンでのみ利用可能

## 1.Distribution の作成
`Create Distribution` で作成を開始｡

![スクリーンショット 2020-06-14 14.42.48.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/f70da9f8-9575-aa85-35b1-78b7b0e581a4.png)

Web Distribution を選択｡
![スクリーンショット 2020-06-14 14.43.14.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/ce0d87be-992f-8274-82bf-02ffacbaaa67.png)


## 2.Origin Origin Settings の設定
### 2.1.Origin Domain Name
MediaStore のコンテナを選択する｡
![スクリーンショット 2020-06-14 14.45.47.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/492427b7-cb45-d555-ace5-3b2b38bf0b18.png)


### 2.2.Origin Domain Name 選択後
- 以下の項目を設定

| 項目 | 設定内容 |
|-----|---------|
|Origin ID|今回は `Origin Domain Name` を選択したときに設定された値のままとした｡|
|Minimum Origin SSL Protocol|今回は初期値の `TLSv1` のままとした｡|
|Origin Protocol Policy|`HTTPS Only` を選択した｡|
|上記以外の設定|すべて初期値のままとした｡|

![スクリーンショット 2020-06-14 14.46.29.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/7e547f07-0fe8-6177-734e-7a6df4763ea3.png)

### 2.6.Distribution の作成
画面最下部右にある `Create Distribution` をクリックして作成する｡
![スクリーンショット 2020-06-14 14.55.49.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/9092655a-43e9-6595-0e02-845181edbc2a.png)

### 2.7.作成中
![スクリーンショット 2020-06-14 14.57.26.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/c23fe88f-b4db-8626-b61c-6cc170f3ae73.png)


### 2.8.作成完了
![スクリーンショット 2020-06-14 15.03.36.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/293eb3d0-93c4-ba08-023b-b1f5c53bdcae.png)


# 3.CloudFront 経由で動画を見る
## 3.1.CloudFrontのURLを確認
 - 作成した Distribution の詳細画面から `Domain Name` を確認する｡
    ![スクリーンショット 2020-06-14 15.10.05.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/3c506c25-ce5c-fcab-0a5e-09f8a5263e5e.png)

## URLをSafariで開く
MediaStore にある配信ファイル(`*.m3u8`) の URL のうち､ドメイン部分を CloudFront のものに置き換えて､Safari から確認する｡

[こちら](https://qiita.com/ksh-fthr/items/64d7137409f3199557fd#332%E9%85%8D%E4%BF%A1%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%81%AE%E7%A2%BA%E8%AA%8D) の配信ファイルを例に取ると､ MediaStore､CloudFront､置き換え後の URL (CloudFront 経由で閲覧する動画の URL)は下記表のようになる｡

|MediaStore のドメイン|CloudFront のURL|置き換え後の URL |
|----|-----|-----|
| hogehoge.data.mediastore.ap-northeast-1.amazonaws.com | piyopiyo.cloudfront.net | https://piyopiyo.cloudfront.net/live/Sample-A.m3u8 |

▼ ドメインが `******.cloudfront.net` で動画配信されていることが確認できる
<img width="1266" alt="スクリーンショット 2020-06-14 15.19.15.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/f715af05-0808-218d-7d81-76af29800d21.png">

# まとめにかえて
これで `MediaLive + MediaStore + CloudFront` の構成で Live 動画配信が確認できた｡
視聴対象を絞らずにアクセスした人が全員見れて良い､というのならばこの構成で良いのだが､視聴対象を絞りたいケースもある｡

ということで､今度は [こちらの記事](https://qiita.com/ksh-fthr/items/e033156756aa182d8eb1) にて以下の構成で IP アドレスで視聴対象を絞る方法について触れてみたい｡

▼Lambda を用いて IP アドレスで視聴対象を絞る

![Lambda-01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/d8f36a41-392f-5fc8-0aa3-447d6ad829e7.png)

