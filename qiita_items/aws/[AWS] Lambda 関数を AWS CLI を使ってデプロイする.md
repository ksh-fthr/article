# はじめに

[AWS Lambda](https://aws.amazon.com/jp/lambda/) を使ってサーバレスな環境での処理を実施出来るのは良いが、毎回 [AWS マネジメントコンソール](https://aws.amazon.com/jp/console/) からメンテナンスするのも億劫だし、できれば Lambda 関数のコードは Git 等で管理したい。

というわけで、Lambda 関数のコードはローカルでメンテナンスを行い、[AWS CLI](https://aws.amazon.com/jp/cli/) を使ってデプロイする方法について見ていく。



# 前提条件

本記事における前提条件は以下の 4 点。

1. デプロイ対象の Lambda 関数がすでに存在していること
   - つまり
     -  `aws iam create-role` でロール作成を事前に行うのではなく
     -  `aws lambda create-function` で Lambda 関数を作成するのでもない
2. 既存の Lambda 関数に対する **_更新_** を行うことが目的となる
3. AWS CLI がインストールされていること
4. AWS CLI を実行するためのユーザがいること


# 作業環境

本記事の作業は EC2 インスタンス上で実施している。



# 前準備

AWS CLI を使うには、[前提条件](#前提条件) でも挙げた次の 2 点が必要なので、そのための確認と前準備を行う。

- AWS CLI がインストールされていること
- AWS CLI を実行するためのユーザがいること



## AWS CLI の確認

繰り返しとなるが、本記事では AWS CLI の実行と確認を EC2 インスタンス上で行っている。
EC2 インスタンスでは予め AWS CLI がインストールされているため、当該ツールのインストールは行っていない。
念の為、インストールされているかの確認とバージョンの確認をしておく。

```bash
# 所在確認
$ which aws
/usr/bin/aws

# (一応)バージョン確認
$ aws --version
aws-cli/1.14.9 Python/2.7.13 Linux/4.9.81-35.56.amzn1.x86_64 botocore/1.8.13
```

- AWS CLI のインストールについては [こちらのドキュメント](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-chap-install.html) を参照。



## AWS CLI の設定

### AWS CLI ユーザ

「アクセスの権限」に「プログラムによるアクセス」のみを選択してユーザを作成。ユーザ名は `aws-cli` とした。
以下のロールを持つグループ `Aws-Cli` を作成し、作成したユーザに紐付けた。権限は持たせ過ぎかも。

| アカウント | グループ | ロール                                        |
| ---------- | -------- | --------------------------------------------- |
| aws-cli    | Aws-Cli  | EC2FullAccess, S3FullAccess, LambdaFullAccess |

で、大事なポイント。
***作成後に発行されるクレデンシャルは AWS CLI の設定を行う際に必要になる。必ずメモ、もしくは csv でダウンロードしておくこと。***
ここで保存しておかないと、もう手に入れる術が無いらしい。

#### 補足
- 紛失したり忘れたりした場合は再作成できる
- [IAM ユーザーのアクセスキーの管理](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/id_credentials_access-keys.html) の 「アクセスキーの管理 (コンソール) > **別の IAM ユーザーのアクセスキーを作成、変更、または削除するには (コンソール)**」 を参照



### AWS CLI の設定

`aws configure` コマンドで設定を行う。設定項目は多くないので敷居は全然高くない。
設定については公式の [設定ファイルと認証情報ファイルの設定](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-configure-files.html) を参照。

```bash
$ aws configure
AWS Access Key ID [None]: ${Access Key}            # <- AWS CLI ユーザ作ったときの Access Key
AWS Secret Access Key [None]: ${Secret Access Key} # <- AWS CLI ユーザ作ったときの Secret Access Key
Default region name [None]: ${Region Name}         # <- デプロイ対象のリージョン, 東京の場合は `ap-northeast-1`
Default output format [None]: json                 # <- json/yaml/txt/table を選択できる
```

ここで  _Access Key_ と _Secret Access Key_ を指定することでデプロイ先が決まる、つまり _**指定したキーに紐づくユーザが存在する環境に対してデプロイされる**_、ということになる。この辺はしっかりと理解しておきたい。


# AWS CLI によるデプロイ

前準備が整ったので、実際に AWS CLI を使って Lambda 関数のデプロイを行っていく。



## 環境変数の設定
### 環境変数の適用コマンド

もし Lambda 関数に対して環境変数を設定したい場合、以下のコマンドで設定できる。

```bash
$ aws lambda update-function-configuration --function-name Hogehoge --environment Variables={KeyName1=string,KeyName2=string}
```

参考: [update-function-configuration](https://docs.aws.amazon.com/cli/latest/reference/lambda/update-function-configuration.html#update-function-configuration)



## デプロイ
### デプロイコマンド

デプロイは対象のリソースを `zip` で固めてコマンドを実行するだけで良い。

```bash
$ zip -r lambda.zip .
$ aws lambda update-function-code --function-name Hogehoge --zip-file fileb://lambda.zip
```
このとき `zip` で固めるのは Lambda 関数のコードがあるディレクトリ。

参考: [update-function-code](https://docs.aws.amazon.com/cli/latest/reference/lambda/update-function-code.html)




### デプロイの例 

ディレクトリ構造が以下の場合( Python で Lambda 関数を実装 )

```
git-repository/
+ src/
    + functions/
        + Hogehoge/
        |   + conf/
        |   |   + Hogehoge.conf
        |   + lambda_function.py
        + Piyopiyo/
            + conf/
            |   + Piyopiyo.conf
            + lambda_function.py
```

`Hogehoge` をデプロイしたければ

1. `cd git-repository/src/function/Hogehoge` で `Hogehoge/` まで移動
2. `zip -r lambda.zip .` で `Hogehoge` 以下を固める
3. で、`aws lambda update-function-code ~` を実行してデプロイする

流れとなる。



### 環境変数の適用例

説明が前後するが、`conf/Hogehoge.cong` の中身に Lambda 関数の環境変数を次のような形で定義していた場合、

```conf
HOGE=hogeeeeee
PIYO=poiyooooo
Bar=Barrrrrrrr

```

[環境変数の設定](#環境変数の設定) で挙げたコマンドを以下のようにすることで、ファイルの中身を環境変数に反映させることも出来る。

```bash
cd git-repository/src/function/Hogehoge

# 1. ファイル末尾の空行を削除
# 2. ファイルの改行コードを `,` に置換
# 3. ファイル末尾にできた `,` を `` に置換(トリムする)
# 4. 上記工程を `Variables={}` の中で行った後、環境変数を Lambda に適用
aws lambda update-function-configuration  --function-name Hogehoge --environment Variables={`cat conf/Hogehoge.conf | tr -s "\n" | tr '\n' ',' | sed s/,$//`}
```



# おまけ

## Jenkins からのデプロイ

AWS CLI を使って Jenkins ジョブから Lambda 関数のデプロイも簡単にできたので、その手順も備忘録としてあげておく。

なお今回、 Jenkins は EC2 で構築しているので、AWS CLI のインストールは行っていない。



### Jenkins ジョブの作成

任意のジョブ名を入力して「フリースタイル・プロジェクトのビルド」を選択する。
![新規ジョブ作成.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/ec94b2d0-a3cf-dabb-2ab5-a6c8803b1de5.png)

### ビルドパラメータの設定

「説明」には本ジョブに関する説明を記入する。

それから、後述の [ソースコード管理の選択](#ソースコード管理の選択) で Git を選択するので、__**入力された文字列をデプロイ対象のブランチとする**__ べく

- 「ビルドのパラメータ化」にチェックを入れ
- 「名前」と「デフォルト値」を入力する
![ビルドパラメータ.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/db01032b-6d1f-12aa-8841-df2b16c69b02.png)


### ソースコード管理の選択

ここでは GitHub のリポジトリを扱いたいので

- Git を選択
- リポジトリ URL には GitHub のURL を入力
- 認証情報は、有れば入力
- ブランチ指定子は [ビルドパラメータの設定](#ビルドパラメータの設定) で入力した文字列を `$` を接頭辞につけて指定する
![ソースコード管理でGitを指定.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/a993a86b-58f2-0879-77e8-4ab6cfc2b4fb.png)


### ビルド処理

#### シェルの実行を選択

AWS CLI を使用して Lambda 関数をデプロイしたいので、ビルド処理では「シェルの実行」を選択する。
![ビルド処理にシェルの追加を指定.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/50957934-d400-f621-a30a-40df0a09cb6a.png)

#### ビルドシェル

ビルドシェルは以下の通り。

 で指定したリポジトリはジョブ実行時に `$WORKSPACE` 配下にクローンされてくる。
で、`$WORKSPACE` 配下の __**デプロイしたい Lambda 関数のソースコードがあるところまで移動** し、[AWS CLI によるデプロイ](#aws-cli-によるデプロイ) で示したように `aws lamda` コマンドから環境変数の適用とデプロイを行う。

- デプロイ対象のディレクトリ構成は [デプロイの例](#デプロイの例) で挙げた `Hogehoge` とする
- 環境変数の中身も [環境変数の適用例](#環境変数の適用例) で挙げたものと同じ

```bash
#!/bin/bash
#
# Hogehoge
#

# ここで Lambda 関数としてデプロイしたいディレクトリまで移動する
cd $WORKSPACE/src/function/Hogehoge

# 環境変数の適用
## 1. ファイル末尾の空行を削除
## 2. ファイルの改行コードを `,` に置換
## 3. ファイル末尾にできた `,` を `` に置換(トリムする)
## 4. 上記工程を `Variables={}` の中で行った後、環境変数を Lambda に適用
aws lambda update-function-configuration  --function-name Hogehoge --environment Variables={`cat conf/Hogehoge.conf | tr -s "\n" | tr '\n' ',' | sed s/,$//`}

# デプロイ
## デプロイ対象のリソースを zip で固めたうえで
zip -r lambda.zip .

## AWS CLI でデプロイする
### function-nameはlambda上の上書き対象の関数名
aws lambda update-function-code --function-name Hogehoge --zip-file fileb://lambda.zip

exit
```



# まとめにかえて

## 所感

AWS CLI からコマンドを実行する際、_**どこに対してデプロイするのか**_ を CLI を実行するときに指定しないのが気持ち悪かった。

当初、「デプロイ先を指定しないでどこにデプロイするんだ ?」と疑問だったのだが、手順を調べていくなかで `aws configure` で CLI の設定を行うことを知った。

で、`aws configure` を実行した際に AWS CLI ユーザの _Access Key_ と _Secret Access Key_ を設定することでデプロイ先が決まるんだな、ということで腹に落ちた。

この辺、なんとなくで使っているとモヤモヤ感が残るので、しっかり理解して使っていきたい。



## このあと試したいこと

今回は Lambda 関数が既に AWS 上に存在する前提で、それに対するデプロイを AWS CLI から行えることを確認した。
が、Lambda 関数が存在しない状態でもデプロイが出来るようにしてみたい。

 に書いたように `aws iam create-role` と `aws lambda create-function` を組み合わせることで対応はできるが、以下の記事によると [SAM](https://aws.amazon.com/jp/serverless/sam/) を利用することでも実現できる様子。

なので、そのうちこちらも試してみたい。

- [AWS SAMでLambda関数を作成](https://www.wakuwakubank.com/posts/640-aws-sam/)



# 参考

## AWS 本家

- [AWS CLI のインストール](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-chap-install.html)
- [AWS CLI の設定](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-chap-configure.html)
- [設定ファイルと認証情報ファイルの設定](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-configure-files.html)
- [IAM ユーザーのアクセスキーの管理](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/id_credentials_access-keys.html)
- [AWS Command Line Interface での AWS Lambda の使用](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/gettingstarted-awscli.html)
- [update-function-configuration](https://docs.aws.amazon.com/cli/latest/reference/lambda/update-function-configuration.html)
- [update-function-code](https://docs.aws.amazon.com/cli/latest/reference/lambda/update-function-code.html)



## 上記以外

- [AWS CLIをインストールしてコマンド操作しよう](https://avinton.com/academy/aws-cli-install-setting/)

