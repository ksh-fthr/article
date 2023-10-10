Express を使用したサーバーサイドの開発環境構築についてのメモ｡
Express はグローバルでインストール｡その後ジェネレータを使用して雛形を生成する｡
で､その雛形をもとにサーバサイドの実装を進めていく感じ｡

あくまで Express はサーバサイドを実装するために使用していて､フロントエンドは Angular とか｡｡｡

## この記事を実施した環境
* Windows10 Home 64bit
* node v8.2.1
* npm v4.0.5
* express v4.15.0
* Visual Studio Code v1.14.2

## Express
Node.js 環境下で動作するフレームワーク｡
公式の日本語訳は[ここ](http://expressjs.com/ja/)｡

## 前提
[Node.js](https://nodejs.org/ja/) がインストールされていること。
Node.js のインストールについては [Windows 版はこちらの記事](http://qiita.com/ksh-fthr/items/fc8b015a066a36a40dc2) を、 [Linux版はこちらの記事](http://qiita.com/ksh-fthr/items/c272384f73f8e319733c) を参照。 

## インストール
1. Express本体のインストール
    次のコマンドを実行して express 本体のグローバルインストールを行う｡

    ```
    $ npm install express -g
    ```

1. Expressジェネレータのインストール
    次のコマンドを実行してジェネレータをインストールする｡

    ```
    $ npm install express-generator -g
    ```

## 雛形の生成
ジェネレータを使用してアプリケーションの雛形を生成する｡

1. ジェネレータの実行
    次のコマンドを実行して雛形を生成する｡

    ```
    $ express my-app
    ```

    カレントディレクトリ直下に my-app ディレクトリが生成されているので､ my-app に移動して中を確認｡

    ```
    $ ls -l
    total 5
    -rw-r--r-- 1 hogehoge hogehoge 1257 8月  11 22:50 app.js
    drwxr-xr-x 1 hogehoge hogehoge    0 8月  11 22:50 bin/
    -rw-r--r-- 1 hogehoge hogehoge  326 8月  11 22:50 package.json
    drwxr-xr-x 1 hogehoge hogehoge    0 8月  11 22:50 public/
    drwxr-xr-x 1 hogehoge hogehoge    0 8月  11 22:50 routes/
    drwxr-xr-x 1 hogehoge hogehoge    0 8月  11 22:50 views/
    ```

1. 依存関係の解決
ジェネレータで生成しただけではライブラリの依存関係でアプリケーションの起動時にエラーとなるので､次のコマンドを実行して依存関係を解決する｡

    ```
    $ npm install
    ```

## アプリケーションの起動と確認
1. アプリケーションの起動
    次のコマンドを実行してアプリケーションを起動する｡

    ```
    $ npm start
    ```

    上記は package.json の次のブロックに記載されている内容を実行するコマンドである｡
    start コマンドの内容を変更したかったり､start 以外のコマンドを追加したい場合は､この "scripts" の要素に追加していけばよい｡

    ```javascript:package.json
    {
      ...
      "scripts": {
        "start": "node ./bin/www"
      },
      ...
    }
    ```

1. アプリケーションの起動確認
    デフォルトでは port:3000 で起動しているはずなので､ブラウザで localhost:3000 を確認する｡
    次の画面が出れば express のアプリケーションが起動されている｡
    ![express-app01.png](https://qiita-image-store.s3.amazonaws.com/0/193342/6acd3193-e416-4c65-37a5-097d5ace467b.png)

1. port 番号の変更
    アプリケーション起動時のポート番号を変更したい場合は my-app/bin/www の次のコードを編集する｡

    ```javascirpt:my-app/bin/www
    // 3000 以外のポートにしたければ 3000 の箇所を任意のポートに変更する
    var port = normalizePort(process.env.PORT || '3000');
    ```

## デバッグ環境の構築
Visual Studio Code でのデバッグ環境を構築する｡

1. launch.json の生成

  1. クモマークをクリックしてデバッグペインを開く  
    ![vscode-debug01.png](https://qiita-image-store.s3.amazonaws.com/0/193342/4470b01e-5077-9ec4-ebb4-3a6121f92cd3.png)

  1. 歯車マークをクリック～｢環境の選択｣から｢node.js｣を選択する
    ![vscode-debug02.png](https://qiita-image-store.s3.amazonaws.com/0/193342/8a9ff1bf-ee9c-29de-7b29-8ecbe45d59fb.png)

  1. my-app/.vscode/launch.json が生成される 
  後述するデバッグ実行時に指定する名前は､下記の｢name｣に指定したものが表示される｡環境に応じてわかりやすい名前を設定すると良い｡

        ```javascript:生成されたlaunch.json
        {
          // Use IntelliSense to learn about possible Node.js debug attributes.
          // Hover to view descriptions of existing attributes.
          // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
          "version": "0.2.0",
          "configurations": [
            {
              "type": "node",
              "request": "launch",
              "name": "Launch Program",
              "program": "${workspaceRoot}\\bin\\www"
            }
          ]
        }
        ```

1. デバッグ実行

  1. デバッグ環境の選択  
  デバッグペインより launch.json の configurations から実行したい環境を選択する｡
  ここでは｢Launch Program｣しか設定していないのでそちらを選択｡  
  ![vscode-debug04.png](https://qiita-image-store.s3.amazonaws.com/0/193342/8cb3f219-7e38-8881-5dbc-3bf3d0ec2966.png)

  1. F5実行でデバッグ開始  
  上のオレンジ枠で囲った部分がデバッグ実行メニュー｡  
  下のオレンジ枠で囲った部分はブレイクポイントで止まった状態｡
  ![vscode-debug05.png](https://qiita-image-store.s3.amazonaws.com/0/193342/ee72d483-4fe2-9d17-9c45-a3a57a57deed.png)

