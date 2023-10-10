# はじめに

[こちらの記事](https://qiita.com/ksh-fthr/items/ee2b7880a9e6c0fc6f88) の ｢[まとめにかえて](https://qiita.com/ksh-fthr/items/ee2b7880a9e6c0fc6f88#%E3%81%BE%E3%81%A8%E3%82%81%E3%81%AB%E3%81%8B%E3%81%88%E3%81%A6)｣で触れたとおり､ CloudFront へのアクセスに対して IP アドレスで制限をかける方法について触れる｡
実現にあたり､本記事では CloudFront と Lambda@Edge を利用する｡



# 注意

本記事は 2020年6月21日 時点の情報です｡
ご覧になられた時点で UI が変更されている可能性がありますので､その点ご注意ください｡



# 前提

- [こちらの記事](https://qiita.com/ksh-fthr/items/ee2b7880a9e6c0fc6f88) の構築が完了していること



# 環境

| サービス                                                     | 概要                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| macOS                                                        | 10.15.x                                                      |
| [Elemental MediaLive](https://aws.amazon.com/jp/medialive/)  | あらゆるデバイスへのブロードキャストおよびストリーミング向けにライブ動画をエンコードする |
| [Elemental MediaStore ](https://aws.amazon.com/jp/mediastore/) | ライブストリーミングによるメディアワークフロー向けにビデオアセットを保存、配信する |
| [CloudFront](https://aws.amazon.com/jp/cloudfront/)          | 高速で安全性が高くプログラム可能なコンテンツ配信ネットワーク (CDN、content delivery network) |
| [Lambda](https://aws.amazon.com/jp/lambda/)                  | サーバーについて検討することなくコードを実行できる           |
| [Lambda@Edge](https://aws.amazon.com/jp/lambda/edge/)        | ユーザーに近いロケーションでコードを実行                     |
| [OBS](https://obsproject.com/ja)                             | [AWS Lambda の使用開始](https://console.aws.amazon.com/lambda/home?region=us-east-1)ビデオ録画と生放送用の無料でオープンソースのソフトウェア。 |



# 想定する構成　

MediaLive + MediaStore のプロダクトを作りたい。
で、CloudFront でキャッシュし、かつ Lambda@Edge でアクセス元の IP アドレスによるフィルタリングも行いたい。

というのをイメージしたのが以下の図。


![Lambda-01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/c6ff6080-ebbd-4c3a-5900-df5030e5b90f.png)

# フィルタリング

ホワイトリスト方式を採用する。指定した IP アドレスのみ許可し、リストにない IP アドレスからのアクセスはエラーとする。



# 制限事項
## ホワイトリストの持ち方

本記事ではホワイトリストは Lambda 関数で保持する方法を示す。
ホワイトリストを DB や S3 等で保持する方法について、本記事では扱わない。
( 手前味噌ではありますが、[こちら](https://qiita.com/ksh-fthr/items/4c7ccd5c9d5b09e5a36e) の記事で _**S3**_ や _**DynamoDB**_ でホワイトリストを持つ方法について触れております。ご興味有ればご参照ください )



# トリガーとするイベント

- [Lambda 関数をトリガーできる CloudFront イベント](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/lambda-cloudfront-trigger-events.html) には下記の4つがあり、本記事では **ビューワーリクエスト** をトリガーとする。
  - **ビューワーリクエスト**
  - **オリジンリクエスト**
  - **オリジンレスポンス**
  - **ビューワーレスポンス**



## ビューワーリクエスト

[開発ガイド](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/lambda-cloudfront-trigger-events.html) から抜粋。

> CloudFront がビューワーからのリクエストを受け取ると、リクエストされたオブジェクトが CloudFront キャッシュ内にあるかどうかを確認する前に、関数が実行されます。

## 注意事項

こちらも [開発ガイド](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/lambda-cloudfront-trigger-events.html) から抜粋。

> CloudFront イベントによって Lambda  関数の実行がトリガーされると、その関数が終了するまで CloudFront は続行できません。たとえば、CloudFront ビューワーリクエストイベントによって Lambda  関数がトリガーされた場合、Lambda 関数の実行が終了するまでは、CloudFront  はビューワーにレスポンスを返したり、オリジンにリクエストを転送したりしません。つまり、Lambda 関数をトリガーするリクエストごとにリクエストのレイテンシーが長くなるため、関数をできるだけ速く実行する必要があります。                                       

## Lambda@Edge の制限
[公式のこちら](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/cloudfront-limits.html#limits-lambda-at-edge) から転載。
これによると Lambda@Edge では通常の Lambda に比べて設定項目に対する制限が厳しくなっているとのこと。

| イベントタイプによって異なるクォータ                         |                                                              |                                                          |
| ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------------------------- |
| エンティティ                                                 | オリジンのリクエストおよびレスポンスイベントのクォータ       | ビューワーのリクエストおよびレスポンスイベントのクォータ |
| 関数のメモリサイズ                                           | [Lambda のクォータ](https://docs.aws.amazon.com/lambda/latest/dg/limits.html)と同じ | 128 MB                                                   |
| 関数タイムアウト。関数は AWS リージョンの Amazon S3 バケット、DynamoDB テーブル、Amazon EC2 インスタンスなどのリソースに対してネットワーク呼び出しを実行できます。 | 30 秒                                                        | 5 秒                                                     |
| ヘッダーと本文を含む、Lambda 関数によって生成されたレスポンスのサイズ | 1 MB                                                         | 40 KB                                                    |
| Lambda 関数および組み込みライブラリの最大圧縮サイズ          | 50 MB                                                        | 1 MB                                                     |


# Lambda@Edge 関数での IP 制限
## Lambda 関数を作成する
### 注意事項
#### Lambda 関数を作成するリージョン
CloudFront のリージョンは ~~現状だと~~ グローバルしか選択できない。それに関係してか、Lambda@Edge 用の関数は **バージニア北部** のリージョンで作成する必要がある。(そうしないとトリガーに CloudFront を指定できない )

### 実行時のリージョンと出力されるログ
作成した関数は _**世界中のAWSローケーション( リージョン )にレプリケートされる**_。
で、Lambda@Edge が実行されるとき、_**実際に実行されるのはクライアントから最寄りのリージョンにレプリケートされた関数**_ となる。
実行時のログは _**実行されたリージョンの CloudWatch Logs**_ に出力される。


参考: [Lambda@Edge 関数の作成と使用の開始](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/lambda-edge-how-it-works.html)

## 手順

1. リージョンに **バージニア北部** を選択する
   <img width="407" alt="スクリーンショット 2020-06-21 9.01.40.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/4952c181-3785-d529-cf60-da0b82d3424b.png">


1. Lambda > 関数 より「関数の作成」をクリック
   ![スクリーンショット 2020-05-25 11.29.25.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/d3f22141-8d40-75d0-6081-e2498e20f269.png)


1. 「一から作成」で「基本的な情報」を埋めていく

   1. 関数名: 実行する処理に応じた関数名を入力
   2. ランタイム: デフォルトのまま(今回は `Node.js 12.x`)
   3. アクセス権限
      - 「実行ロールの選択または作成」で「基本的な Lambda アクセス権限で新しいロールを作成」を選択
   4. 「関数の作成」をクリック
      ![スクリーンショット 2020-05-25 12.07.36.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/a381f716-ff42-aef3-1199-aa4060aa0460.png)
       ![スクリーンショット 2020-05-25 12.07.47.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/3ebf85a2-53f5-bc0f-67e3-a51c4f93acef.png)

2. Lambda 関数の設定画面が表示される(これで Lambda 関数そのものは作成できた)
   <img width="1297" alt="スクリーンショット 2020-05-25 15.42.09.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/76210495-d144-acd6-58b8-a8fe77d8dcea.png">


1. 一旦完了


## IAM ロールの設定

CloudFront へのアクセスに対して Lambda@Edge を適用する場合、**Lambda@Edge 用の IAM ロール** が必要なので、作成したロールに設定を追加する。

### アクセス権限の設定

1. IAM > ロール から「[Lambda 関数を作成する](#Lambda-関数を作成する)」で作成したロールを選択
   ![スクリーンショット 2020-05-25 12.27.16.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/7db05a0c-eeda-eead-dd45-4d02ae174b59.png)

2. 表示された概要の「アクセス権限」タブからポリシーを選択し「ポリシーの編集」をクリック
   ![スクリーンショット 2020-05-25 12.30.28.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/6afcdbef-99e5-ff33-fe80-04764150d5a7.png)
   ![スクリーンショット 2020-05-25 12.32.44.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/b51871ce-aa4e-067f-9b3b-081e36070d54.png)


3. 編集画面が表示されるので「JSON」タブをクリック
   ![スクリーンショット 2020-05-25 12.33.19.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/081f6031-4a93-4e26-e046-d879358cfd58.png)


4. JSONを編集

   次の内容を設定する｡

   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "VisualEditor0",
               "Effect": "Allow",
               "Action": [
                   "logs:CreateLogStream",
                   "iam:CreateServiceLinkedRole",
                   "lambda:GetFunction",
                   "cloudfront:UpdateDistribution",
                   "cloudfront:CreateDistribution",
                   "logs:PutLogEvents",
                   "lambda:EnableReplication*"
               ],
               "Resource": "*"
           },
           {
               "Sid": "VisualEditor1",
               "Effect": "Allow",
               "Action": "logs:CreateLogGroup",
               "Resource": "*"
           }
       ]
   }
   ```

5. 「ポリシーの確認」をクリック
   ![スクリーンショット 2020-05-25 15.54.09.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/ab61b58e-9662-f390-7a4f-9e04e3195a6a.png)

6. 確認画面で設定した項目が追加されていることを確認し､「変更の保存」をクリック
   ![スクリーンショット 2020-05-25 12.39.22.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/7778cef3-920f-8916-4e6c-e1aa551fe536.png)


### 信頼関係の設定

1.  再度､IAM > ロール から「[Lambda 関数を作成する](#Lambda-関数を作成する)」で作成したロールを選択

2. 「信頼関係」のタブから「信頼関係の編集」をクリック
   <img width="1598" alt="スクリーンショット 2020-05-25 14.21.34.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/524e9a40-6109-7b5b-113b-0f62cb07770d.png">

3. JSONの編集画面になるので、`Service` プロパティに以下を追記

   ```javascript
   edgelambda.amazonaws.com
   ```

   `Service` プロパティの部分が配列ではない場合は配列に変更する。変更後の JSON が下記。

   ```javascript
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "Service": [
               "lambda.amazonaws.com",
               "edgelambda.amazonaws.com"
           ]
         },
         "Action": "sts:AssumeRole"
       }
     ]
   }
   ```

   ▼JSON編集後の画面
   <img width="1271" alt="スクリーンショット 2020-05-25 14.30.21.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/075a3ca4-dd49-5b16-24c9-7a3685e56476.png">

4. 設定した項目が追加されていることを確認し､「信頼ポリシーの更新」をクリック
   <img width="1599" alt="スクリーンショット 2020-05-25 14.31.07.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/74bcdc4c-2efa-e992-9c11-e53ac3f47c21.png">



# Lambda に戻って

IAM ロールの設定が完了したので､もう一度 Lambda に戻って Lambda 関数の実装を行う｡

## Lambda 関数の処理を実装する

1. 「[Lambda 関数を作成する](#Lambda-関数を作成する)」で作成した関数を選択〜表示し「関数コード」を編集する
   IPアドレスのホワイトリスト、並びにフィルタリング処理の例を以下に示す。

   ```javascript
   'use strict'
   
   // IPアドレスのホワイトリスト
   const IP_WHITE_LIST = [
     'xxx.xxx.xxx.xxx', //IPアドレスは各自設定すること！
   ];
   
   const errorResponse = (httpVersion) => {
     const body =
       '<!DOCTYPE html>\n' +
       '<html>\n' +
       '<head><title>Lambda@Edge からのエラー</title></head>\n' +
       '<body>\n' +
       '許可されていない IP アドレスです\n' +
       '</body>\n' +
       '</html>'
   
     return  {
       status: '403',
       statusDescription: 'Forbidden',
       httpVersion: httpVersion,
       body: body,
       headers: {
         'cache-control': [{
           key: 'Cache-Control',
           value: 'max-age=100'
         }],
         'content-type': [{
           key: 'Content-Type',
           value: 'text/html; charset=utf-8'
         }],
       },
     };
   };
   
   exports.handler = (event, context, callback) => {
     const request = event.Records[0].cf.request;
     const httpVersion = request.httpVersion;
     const clientIp = request.clientIp; // リクエスト情報からアクセス元のIPアドレスを取得できる
     const isPermittedIp = IP_WHITE_LIST.includes(clientIp);
   
     if (isPermittedIp) {
       // 許可されているIPアドレスなので何もしない( 通常処理へ流れる )
       callback(null, request);
     } else {
       // 許可されていない IP アドレスなのでエラーを返す
       callback(null, errorResponse(httpVersion));
     }
   }
   ```

2. 「保存」をクリックして編集内容を保存する

## Lambda関数を Lambda@Edge として CloudFront に紐付ける

Lambda 関数の実装が終わったら､次は下記の手順で **Lambda@Edge** として CloudFront と紐付けを行う｡

1. 「アクション」から「新しいバージョンを発行」を選択
   <img width="1498" alt="スクリーンショット 2020-05-25 14.03.52.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/fec62e8a-7a98-131b-cea0-d635b5f93d49.png">

2. モーダル上で「発行」をクリック(バージョンの説明はなくてもOK)
   <img width="847" alt="スクリーンショット 2020-05-25 14.04.03.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/f5e9e133-ce9b-301f-e3ee-11174ff5ebed.png">

3. 「トリガーを追加」をクリック
   <img width="1503" alt="スクリーンショット 2020-05-25 14.04.25.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/b6af8b6b-019b-cc22-1fd6-3be8450b3873.png">

4. 編集画面で「CloudFront」を選択

   このとき Lmabda関数を **バージニア北部** のリージョンで作成していないと CloudFront が選択肢に出てこないので注意｡

   <img width="860" alt="スクリーンショット 2020-05-25 14.04.40.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/070dac25-0b53-aa94-75c7-5aaec752cf44.png">

5. 「CloudFront トリガーの設定」を編集

   1. 「ディストリビューション」に紐付ける CloudFront のディストリビューションを設定
   2. 「キャッシュ動作」はいじらない
   3. 「CloudFront イベント」には「ビューアーリクエスト」を選択
   4. 「ボディを含める」にチェック
   5. 「関数のこのバージョンが〜」にチェック

    <img width="842" alt="スクリーンショット 2020-05-25 14.05.18.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/fa739a8e-0d88-95b5-c041-bc0bec5238f4.png">

6. 「追加」をクリック
   <img width="837" alt="スクリーンショット 2020-05-25 14.05.28.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/cf8c50dc-67c6-b3a8-c073-ebb337524b5d.png">

7. トリガーに **CloudFront** が追加されていることを確認
  ![スクリーンショット 2020-05-25 15.04.27.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/3152ebd9-bdea-cb2f-4b18-c1f286e191b3.png)


# 動作確認

## ホワイトリスに登録されていないIPアドレスからのアクセス
Lambda 関数で設定したエラーメッセージが表示されていることが確認できる｡
<img width="1267" alt="スクリーンショット 2020-06-21 10.11.25.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/354b17f3-faf5-c32d-5f25-5aa13a893a1d.png">

# まとめ

- IPアドレスのフィルタリングはホワイトリスト方式とする
- Lambda 関数での視聴者の IP アドレス取得方法

  ```javascript
  exports.handler = (event, context, callback) => {
    // 省略  
    const clientIp = request.clientIp; // リクエスト情報からアクセス元のIPアドレスを取得できる
    // 省略
  }
  ```

- **Lambda@Edge 用の IAM ロール**が必要
- Lambda@Edge 用の関数は **バージニア北部** のリージョンで作成する必要がある( そうしないとトリガーに CloudFront を指定できない )
- Lambda@Edge で作成した関数は _**世界中のAWSローケーション( リージョン )にレプリケートされる**_
- _**実際に実行されるのはクライアントから最寄りのリージョンにレプリケートされた関数**_
- 実行時のログは _**実行されたリージョンの CloudWatch Logs**_ に出力される。
- CloudFront のイベントには **ビューワーリクエスト** を指定する

# 参考
## 公式

- [開発ガイド-Lambda@Edge 関数の例](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/lambda-examples.html)
- [Lambda 関数の要件と制限](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/lambda-requirements-limits.html#lambda-requirements-distributions)
- [Lambda@Edge デザインベストプラクティス](https://aws.amazon.com/jp/blogs/news/lambdaedge-design-best-practices/)
- [Lambda コンソールで Lambda@Edge 関数を作成する](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/lambda-edge-create-in-lambda-console.html)
- [Lambda@Edge 用の Lambda 関数の編集](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/lambda-edge-edit-function.html) 
- [Amazon CloudFront と AWS Lambda@Edge を用いたプライベートコンテンツの提供](https://aws.amazon.com/jp/blogs/news/serving-private-content-using-amazon-cloudfront-aws-lambdaedge/)
- [Lambda@Edge 関数の作成と使用の開始](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/lambda-edge-how-it-works.html)
- [Lambda@Edge のクォータ](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/cloudfront-limits.html#limits-lambda-at-edge)

## その他
- [Lambda@EdgeでCloudFrontへのアクセスをいい感じに振り分ける](https://techblog.zozo.com/entry/lambda-edge)
- [Amazon CloudFrontとAWS Lambda@EdgeでSPAのBasic認証をやってみる](https://dev.classmethod.jp/articles/cloudfront-lambdaedge-basic-spa/)
