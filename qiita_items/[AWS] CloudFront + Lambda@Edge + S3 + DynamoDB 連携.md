# はじめに

[こちらの記事](https://qiita.com/ksh-fthr/items/e033156756aa182d8eb1) で IP アドレスによるフィルタリングが実現できたので、本記事ではフィルタリングに用いる IP アドレスのリストを

- S3
- DynamoDB

に持たせる方法について見ていく。


# 注意

本記事は 2020年7月10日 時点の情報です｡
ご覧になられた時点で UI が変更されている可能性がありますので､その点ご注意ください｡



# 前提

- [こちらの記事](https://qiita.com/ksh-fthr/items/e033156756aa182d8eb1) の構築が完了していること



# 環境

| サービス                                                     | 概要                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| macOS                                                        | 10.15.x                                                      |
| [Elemental MediaLive](https://aws.amazon.com/jp/medialive/)  | あらゆるデバイスへのブロードキャストおよびストリーミング向けにライブ動画をエンコードする |
| [Elemental MediaStore ](https://aws.amazon.com/jp/mediastore/) | ライブストリーミングによるメディアワークフロー向けにビデオアセットを保存、配信する |
| [CloudFront](https://aws.amazon.com/jp/cloudfront/)          | 高速で安全性が高くプログラム可能なコンテンツ配信ネットワーク (CDN、content delivery network) |
| [Lambda](https://aws.amazon.com/jp/lambda/)                  | サーバーについて検討することなくコードを実行できる           |
| [Lambda@Edge](https://aws.amazon.com/jp/lambda/edge/)        | ユーザーに近いロケーションでコードを実行                     |
| [S3](https://aws.amazon.com/jp/s3/) |  どこからでもお好みの量のデータの保存と取得が簡単に行えるオブジェクトストレージ |
| [DynamoDB](https://aws.amazon.com/jp/dynamodb/) | どんな規模にも対応する高速で柔軟な NoSQL データベースサービス |


# やりたいこと

- CloudFront へのアクセスを Lambda@Edge で IP フィルタリングする
- フィルタリングほホワイトリスト形式で許可するIPアドレスを管理する
- ホワイトリストは S3 / DynamoDB で管理する

# S3 での実現方法
## 想定する構成　

MediaLive + MediaStore のプロダクトを作る場合、CloudFront でキャッシュすると想定すると、以下の図になる。

![S3-01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/9e8dea79-9f7e-8a85-e50c-8e4838711939.png)


## S3 にホワイトリストを登録

### バケットの設定

#### 暗号化

- AES-256 を設定

#### アクセス権限

- ブロックパブリックアクセスを**すべてブロック**に設定

![S3-02.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/56b77956-b225-3830-88f2-3c67394a10f1.png)


### ホワイトリストの内容

次の `JSON` ファイルを S3 のバケットにアップロード。

```json
{
  "white_list": [
      "xxx.xxx.xxx.xxx" // 許可するIPアドレス
  ]
}
```



## Lamdba@Edge の編集

### アクセス権限

- S3 へのアクセス権限をもつロールを付与
    - フルアクション
    - フルアクセス
   
![S3-03.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/d6649da9-684d-bfd1-3714-aeaef4ff20d3.png)


### Lamdba 関数

以下の実装で実現できた。(あくまでサンプル。実運用の際は精査する)

```javascript
'use strict'

// S3 からファイルを読み込む
const aws = require('aws-sdk');
aws.config.region = 'ap-northeast-1';
const s3 = new aws.S3();

const paramsToGet = {
    Bucket: 'ip-white-list',
    Key: 'white_list.json'
};

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
  const clientIp = request.clientIp;
  

  // S3 からファイルを読み込む
  s3.getObject(paramsToGet, (err, response) => {
    if (err) {
      console.log(err);
      callback(err);
    }

    console.log('response: ' + JSON.stringify(response));
    if (!response) {
      callback('response is null or undefined');
    }

    console.log('response.body: ' + response.Body.toString("utf-8"));
    if (!response.Body) {
      callback('response.body is null or undefined');
    }
    
    // ここで指定する `white_list` は S3 のバケットにアップロードされた JSON 中の key
    const permitIp = JSON.parse(response.Body.toString("utf-8"))["white_list"];
    console.log('permitIp: ' + permitIp);

    const isPermittedIp = permitIp.includes(clientIp);
    if (isPermittedIp) {
      // 許可されているIPアドレスなので何もしない( 通常処理へ流れる )
      callback(null, request);
    } else {
      // 許可されていない IP アドレスなのでエラーを返す
      callback(null, errorResponse(httpVersion));
    }
  });
}
```



## 要検討事項

- バケットの設定(暗号化等)とアクセス権限
- Lamdba@Edge のロールに持たせるアクセス権限


---

# DynamoDB での実現方法
## 想定する構成

MediaLive + MediaStore のプロダクトを作る場合、CloudFront でキャッシュすると想定すると、以下の図になる。

![dynamodb-01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/e96d6b78-0894-c2db-9cd0-324c721bd677.png)


## DynamoDB にホワイトリストを登録
### テーブルの作成とデータ登録

[ NoSQL テーブルを作成してクエリを実行する](https://aws.amazon.com/jp/getting-started/hands-on/create-nosql-table/) に従いテーブルを作成する。
今回作成したテーブルと登録したデータの内容は次の通り。

![dynamodb-02.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/9c8b2ac4-bd57-e163-975f-d949a240e741.png)


## Lamdba@Edge の編集

### アクセス権限

- DynamoDB へのアクセス権限をもつロールを付与
  - フルアクション
  - フルアクセス

![dynamodb-03.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/47469e13-2885-49a4-a78a-a6ffd4fe3b4a.png)



### Lamdba 関数

以下の実装で実現できた。(あくまでサンプル。実運用の際は精査する)
ポイントは以下の2つ。

- `aws.DynamoDB.DocumentClient` を使用する
- `scan` メソッドを使用する

```javascript
'use strict'

const aws = require('aws-sdk');
aws.config.region = 'ap-northeast-1';

// DynamoDB のオブジェクトを作る
const ddb = new aws.DynamoDB.DocumentClient({apiVersion: '2012-08-10'});

const params = {
  TableName: 'ip-white-list'
};

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
  const clientIp = request.clientIp;
  
  // DynamoDB からデータ取得
  ddb.scan(params, function(err, response) {
    if (err) {
      console.log(err);
      callback(err);
    }

    console.log('response: ' + JSON.stringify(response));
    if (!response) {
      callback('response is null or undefined');
    }

    console.log('response.Items: ' + response.Items);
    if (!response.Items) {
      callback('response.Item is null or undefined');
    }
    
    const permitIp = response.Items.map((item) => {
      // ここで指定する `ip_address` は DynamoDB のテーブルに作成したカラム
      return item.ip_address;
    });
    console.log('permitIp: ' + permitIp);

    const isPermittedIp = permitIp.includes(clientIp);
    if (isPermittedIp) {
      // 許可されているIPアドレスなので何もしない( 通常処理へ流れる )
      callback(null, request);
    } else {
      // 許可されていない IP アドレスなのでエラーを返す
      callback(null, errorResponse(httpVersion));
    }
  });
}
```


## 要検討事項

- DynamoDB の設定
- Lamdba@Edge のロールに持たせるアクセス権限

# まとめにかえて
以上、S3 と Dynamo DB に IP アドレスのホワイトリストを持たせて、Lambda@Edge から利用する方法を見てきた。
これで

- 別サービスと連携してホワイトリストを実現出来る
- Lambda 関数内にデータを持つ必要はなくなった
- _実装_ と _データ_ で密な関係を持たずに済む

というのが分かった。
要検討事項として

- S3 や DynamoDB の設定
- アクセス権限

といったものはあるが、これらについては実際の運用に合わせて適宜見直す。

# 備考
## エラー時のレスポンス
記事中の Lambda 関数では許可されていない IP アドレスだった場合にレスポンスに `body` を設定しているが、場合によってはリダイレクトを行って任意のページに誘導したいケースもあると思う。

リダイレクトについては [公式の開発ガイド](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/lambda-examples.html) にサンプルが載っているので参照されたい。

一応参考までにサンプルコードを転記しておく。

### [例: HTTP リダイレクトの生成 (生成されたレスポンス)](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/lambda-examples.html#lambda-examples-http-redirect)

```javascript
'use strict';

exports.handler = (event, context, callback) => {
    /*
     * Generate HTTP redirect response with 302 status code and Location header.
     */
    const response = {
        status: '302',
        statusDescription: 'Found',
        headers: {
            location: [{
                key: 'Location',
                value: 'http://docs.aws.amazon.com/lambda/latest/dg/lambda-edge.html',
            }],
        },
    };
    callback(null, response);
};
```

# 参考
## 公式
- [開発ガイド-Lambda@Edge 関数の例](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/lambda-examples.html)
- [Lambda 関数の要件と制限](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/lambda-requirements-limits.html#lambda-requirements-distributions)
- [Lambda@Edge デザインベストプラクティス](https://aws.amazon.com/jp/blogs/news/lambdaedge-design-best-practices/)

