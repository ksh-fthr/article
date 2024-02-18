[Express](https://expressjs.com/) を使用した開発環境とデバック環境の構築についてのメモです｡
本記事では次の手順で作業を進めます。

1. Express はグローバルでインストールします
2. その後ジェネレータを使用して雛形を生成します
3. そしてその雛形をもとにデバッグ環境を構築します

:::note info
(注釈)

開発環境、並びにデバッグ環境の構築が主眼のため上記としております。
( 公式の Getting Started に則った手順で Express を導入しておりません)

Express の導入にあたり、より確かな情報をお求めの際は Express のページより [Getting Started](https://expressjs.com/en/starter/installing.html) をご参照ください。
:::

# この記事を実施した環境(本記事執筆当時)

|                      | バージョン | 備考                    |
| -------------------- | ---------- | ----------------------- |
| Windows10 Home 64bit |            |                         |
| node.js              | `v8.2.1`   | `node --version` で確認 |
| npm                  | `v4.0.5`   | `npm --version` で確認  |
| express              | `v4.15.0`  | `package.json` で確認   |
| Visual Studio Code   | `v1.14.2`  |                         |

# この記事を実施した環境(2024/02/18 更新時)

|                    | バージョン | 備考                    |
| ------------------ | ---------- | ----------------------- |
| macOS              | 14.2.1     |                         |
| node.js            | `v18.19.0` | `node --version` で確認 |
| npm                | `v10.2.3`  | `npm --version` で確認  |
| express            | `v4.18.2`  | `package.json` で確認   |
| Visual Studio Code | `v1.86.2`  |                         |

※ 記事中のキャプチャは 本記事執筆当時 のものをそのまま使用しております。

# Express

Node.js 環境下で動作するフレームワークです。
公式の日本語訳は [ここ](http://expressjs.com/ja/) ですが古くなっている可能性があります。
最新の情報は [こちら](https://expressjs.com/) をご参照ください。

# 前提

[Node.js](https://nodejs.org/ja/) がインストールされていること。
Node.js のインストールについては、手前味噌で恐縮ですが [Windows 版はこちらの記事](http://qiita.com/ksh-fthr/items/fc8b015a066a36a40dc2) を、 [Linux版はこちらの記事](http://qiita.com/ksh-fthr/items/c272384f73f8e319733c) を参照してください。

# インストール

## 1. Express本体のインストール

次のコマンドを実行して express 本体のグローバルインストールを行います｡

```bash
% npm install express -g
```

## 2. Expressジェネレータのインストール

次のコマンドを実行してジェネレータをインストールします。

```bash
% npm install express-generator -g
```

# 雛形の生成

ジェネレータを使用してアプリケーションの雛形を生成します｡

## 1. ジェネレータの実行

次のコマンドを実行して雛形を生成します｡

```bash
% express my-app
```

そうしますとカレントディレクトリ直下に my-app ディレクトリが生成されているので､ my-app に移動して中を確認します｡

```bash
% ls -l
total 5
-rw-r--r-- 1 hogehoge hogehoge 1257 8月  11 22:50 app.js
drwxr-xr-x 1 hogehoge hogehoge    0 8月  11 22:50 bin/
-rw-r--r-- 1 hogehoge hogehoge  326 8月  11 22:50 package.json
drwxr-xr-x 1 hogehoge hogehoge    0 8月  11 22:50 public/
drwxr-xr-x 1 hogehoge hogehoge    0 8月  11 22:50 routes/
drwxr-xr-x 1 hogehoge hogehoge    0 8月  11 22:50 views/
```

## 2. 依存関係の解決

ジェネレータで生成しただけではライブラリの依存関係でアプリケーションの起動時にエラーとなるので､次のコマンドを実行して依存関係を解決します｡

```bash
% npm install
```

# アプリケーションの起動と確認

## 1. アプリケーションの起動

次のコマンドを実行してアプリケーションを起動します｡

```bash
% npm start
```

上記は `package.json` の次のブロックに記載されている `start` の内容を実行するコマンドです｡
`start` コマンドの内容を変更したかったり `start` 以外のコマンドを追加したい場合は､この `"scripts"` の要素内の情報を編集していくことになります。

```js:package.json
{
  ...
  "scripts": {
    "start": "node ./bin/www"
  },
  ...
}
```

## 2. アプリケーションの起動確認

デフォルトでは `port:3000` で起動しているはずなので､ブラウザで `localhost:3000` を確認します｡
次の画面が出れば express のアプリケーションが起動されています。

![express-app01.png](https://qiita-image-store.s3.amazonaws.com/0/193342/6acd3193-e416-4c65-37a5-097d5ace467b.png)

## 3. port 番号の変更

アプリケーション起動時のポート番号を変更したい場合は `my-app/bin/www` の次のコードを編集します｡

```js:my-app/bin/www
// 3000 以外のポートにしたければ 3000 の箇所を任意のポートに変更する
var port = normalizePort(process.env.PORT || '3000');
```

# デバッグ環境の構築

Visual Studio Code でのデバッグ環境を構築します｡

## 1. launch.json の生成

1. デバッグペインを開く
クモマークをクリックしてデバッグペインを開きます。
![vscode-debug01.png](https://qiita-image-store.s3.amazonaws.com/0/193342/4470b01e-5077-9ec4-ebb4-3a6121f92cd3.png)

2. 環境の選択
歯車マークをクリックし、｢環境の選択｣から｢node.js｣を選択します。
![vscode-debug02.png](https://qiita-image-store.s3.amazonaws.com/0/193342/8a9ff1bf-ee9c-29de-7b29-8ecbe45d59fb.png)

3. launch.json が生成される
`my-app/.vscode/launch.json` が生成されます。
なおデバッグ実行時に指定する名前は `launch.json` の `configurations` のリストにある `name` で指定したものが表示されます。
環境に応じてわかりやすい名前を設定すると良いです。

```js:生成されたlaunch.json
{
  // Use IntelliSense to learn about possible Node.js debug attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Program",  // ここに指定した名前がデバッグ時に指定する名前
      "program": "${workspaceRoot}\\bin\\www"
    }
  ]
}
```

## 2. デバッグ実行

1. デバッグ環境の選択  
デバッグペインより `launch.json` の `configurations` から実行したい環境を選択します。
ここでは `Launch Program` しか設定していないので、それを選択します｡  
![vscode-debug04.png](https://qiita-image-store.s3.amazonaws.com/0/193342/8cb3f219-7e38-8881-5dbc-3bf3d0ec2966.png)

2. F5実行でデバッグ開始
上のオレンジ枠で囲った部分がデバッグ実行メニューです｡  
下のオレンジ枠で囲った部分はブレイクポイントで止まった状態です｡
![vscode-debug05.png](https://qiita-image-store.s3.amazonaws.com/0/193342/ee72d483-4fe2-9d17-9c45-a3a57a57deed.png)

# まとめにかえて

以上で Express における開発環境とデバッグ環境の構築は終わりです。

冒頭でも触れましたが、本記事は環境構築が主眼のため公式の Getting Started に則った手順で Express を導入しておりません。
より確かな情報をお求めの際は是非公式の [Getting Started](https://expressjs.com/en/starter/installing.html) をご参照ください。

また REST-API を Express で実装し、そのデバッグを行いたい場合も本記事の手順で実施できますのでお試しください。

# 参考

- [Express(英語)](https://expressjs.com/)
- [Express(日本語)](https://expressjs.com/ja/)
