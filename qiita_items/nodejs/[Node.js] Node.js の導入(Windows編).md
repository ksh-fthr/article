# 注意

2018/09/20 現在､ nodist で npm をインストールした環境で次のコマンドを実行し､ npm のアップデートを行うとエラーが発生します｡

```bash:問題となるコマンド(npmのアップデートを行う)
$ nodist npm match
```

エラーが発生する原因や対応については下記をご参照ください｡

* 参考
  * [nodist の pull request](https://github.com/marcelklehr/nodist/pull/217)
  * [nodistでnpm6.2.0以降にアップデート出来なかった話](https://qiita.com/TalkWithWater/items/aa7b1becd8fc1344170d)



## nodist を使った npm のアップデート

上記で挙げた `$ nodist npm match` は Node.js と npm のバージョンを対応させてくれるコマンドですが、これを使わずに npm のバージョンアップを行うこともできます。
その場合、後述の [Node.js の導入 (nodist を介して)](#nodejs-の導入-nodist-を介して) に記載の「6.npm のアップデートも行っておく」で示した方法で実現できますが、この方法だと Node.js と npm のバージョンを自動で対応させてくれません。
両者のバージョンを合わせたい場合は下記の一覧をもとにバージョンを指定してコマンドを実行してください。

* [リリース一覧](https://nodejs.org/ja/download/releases/)



# 更新情報(2020年12月31日）

以下のバージョンで動作確認したので､それにあわせて記事を修正しました｡

## 確認環境

| 環境                 | バージョン                     | 備考 |
| -------------------- | ------------------------------ | ---- |
| Windows10 Home 64bit | Version 1909 用の 4.8          |      |
| nodist               | ~~v0.8.8~~ v0.9.1              |      |
| Node.js              | ~~v8.2.1~~ ~~v9.2.1~~ v14.15.3 |      |
| npm                  | ~~v4.0.5~~ ~~v5.6.0~~ v6.14.9  |      |
| Git                  | ~~v2.16.0~~ v2.30.0            |      |



# 更新情報(2019年7月8日）

2019/07/08 現在、 nodist は `v0.9.1` が最新です。
後述の [nodist の導入](#nodist-の導入) でもリンクを貼っておりますが、[こちら](https://github.com/nullivex/nodist/releases) から最新版のダウンロードができます。

~~なお `v0.9.1` での動作確認を行なっておりませんので、本記事では本項目での「更新情報」としての紹介に留めさせていただきます。~~

~~本記事における以下の内容の動作確認環境は [確認環境](#確認環境) でも記しましたとおり、あくまで `v0.8.8` となります。ご承知おきください。~~



# Node.js

フロントエンド､バックエンド問わず､最近の JavaScript を使用した開発では [Node.js](https://nodejs.org/ja/) の導入がデファクトとなっている｡
Node.js を導入することで npm が利用できるようになり､ npm を利用することで Node.js で提供される JavaScript フレームワークやライブラリの導入が容易となる｡

# nodist

nodist は Node.js のバージョンを管理するツール｡
これを入れることでバージョン単位で Node.js を導入することができ､状況に応じてバージョンを切り替えることができる｡



# バージョンを切り替えることができるメリット

npm パッケージは Node.js のバージョンに依存することが多い｡
あるバージョンでは動いていたパッケージがあるバージョンでは動かない､ということがある｡
このケースにおいて､ Node.js のバージョンを､ Node.js をインストールし直すことなく切り替えることができることは大きなメリットである｡

# nodist の導入

## 公式ページからインストーラをダウンロード

1. 公式ページ(GitHub)
   https://github.com/marcelklehr/nodist

2. インストーラのリンクをクリックして
   ![nodist01.png](https://qiita-image-store.s3.amazonaws.com/0/193342/bd701185-8e46-7995-aa9d-914897a07c76.png)

3. 最新版( `v0.9.1` )をダウンロードする
   ![NodeJS_091.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/bd9ca1db-a6f4-1a74-f40f-fc4e060d6f7b.png)


## nodist のインストール

1. インストーラに従ってインストール

2. バージョン確認

   ```bash
   $ nodist --version
   0.9.1
   ```



# Node.js の導入 (nodist を介して)

1. インストール可能な Node.js のバージョンを確認

     ```bash
     $ nodist dist
       0.1.14
       0.1.15
       ～中略～
       14.15.3
       15.0.0
       15.0.1
       15.1.0
       15.2.0
       15.2.1
       15.3.0
       15.4.0
       15.5.0
     ```

2. 任意のバージョンをインストール

     ここでは [2020年12月現在の LTS](https://nodejs.org/ja/) である `v14.15.3` を選択した｡

     ```bash
     $ nodist + v14.15.3
     14.15.3 [===============] # ここの KiB はバイナリによる
     14.15.3
     ```

3. インストールされている Node.js を確認

     ```bash
     $ nodist
       (x64)
       10.11.0
     > 11.13.0  (global: v11.13.0)
       14.15.3
     ```

4. Node.js のバージョンを切り替える

     ```bash
     $ nodist v14.15.3
     v14.15.3
     v14.15.3 (global)
     ```

5. 現在利用対象となっている Node.js を確認

     ```bash
     $ nodist
       (x64)
       10.11.0
       11.13.0
     > 14.15.3  (global: v14.15.3)
     $ 
     $ node --version
     v14.15.3
     ```



# npm のマッチング

1. 現在のバージョンを確認

     ```bash
     $ npm --version
     6.4.1
     ```

1. nodist から Node.js と npm のマッチングを行う

     [ リリース一覧](https://nodejs.org/ja/download/releases/) を見ると `Node.js v14.15.3` にマッチングするのは `npm v6.14.9` )

     ```bash
     $ nodist npm match
     npm match
     https://codeload.github.com/npm/cli/tar.gz/v6.14.9 [============== ] # ここの KiB はバイナリによる
     ```

1. インストールした npm のバージョンに切り替える

     ```bash
     $ nodist npm 6.14.9
     npm 6.14.9
     ```

1. 現在利用対象となっている Node.js を確認

     ```bash
     $ npm --version
     6.14.9
     ```



# npm を単体でアップデートしたい場合

1. 現在のバージョンを確認

     ```bash
     $ npm --version
     4.0.5
     ```

2. アップデート

     ```bash:最新版にする場合
     $ npm install npm -g
     ```

     ```bash:バージョンを指定する場合
     $ npm install npm@5.6.0 -g
     ```

3. 再度バージョンを確認

     ```bash
     $ npm --version
     5.6.0
     ```

## npm をアップデートしたときの挙動に関する備忘録

1. 通常のコマンドプロンプトと Git Bash とで挙動が異なる
   * [A] 通常のコマンドプロンプトの実行結果
     * アップデート処理終了後の `npm -v` の結果は `4.0.5` のままだった
   * [B] [確認環境](#確認環境) に示した `Git Bash` の実行結果
     * アップデート処理終了後の `npm -v` の結果は更新したバージョン( 今回は `5.6.0` ) となった

2. `npmv` フォルダを削除したときの挙動が異なる
   通常のコマンドプロンプトでは nodist のインストール先にある `npmv` ( デフォルトだと C:\Program Files (x86)\Nodist\npmv )を参照しているようで､このフォルダを削除したときの `npm -v` は以下の結果となった｡

  * [A]のケースでは 

      ```bash
      $ npm --version
      Sorry, there's a problem with nodist. Couldn't resolve npm version spec 4.0.5 : open C:\Program Files (x86)\Nodist/npmv: The system cannot find the file specified.
      ```

  * [B] のケースでは

      ```bash
      $ npm --version
      5.6.0
      ```



# プロキシの設定(2017/08/28 追記)

`nodist dist` を実行した際に次のエラーが出るケースがある｡

```bash:エラー
$ nodist dist
Could not read response for https://nodejs.org/dist/index.json
Could not read response for https://iojs.org/dist/index.json
Could not read response for https://nodejs.org/dist/index.json.
Sorry.
```

ネットワークに繋がっているにもかかわらずこういったエラーがでるのは、プロキシが通っていないケースが多い｡
次のコマンドを実行してプロンプト上でプロキシを設定してやる｡

```bash:プロキシの設定
$ set HTTP_PROXY=http://proxy.server.co.jp:8080

# ホスト名の proxy.server.co.jp は自分の環境にあわせて編集
# ポート番号の 8080 も自分の環境にあわせて編集
```

なお上記の設定は、実行しているコマンドプロンプト上でのみ有効な一時的な設定なので留意しておくこと｡


# ```Use the `--scripts-prepend-node-path` ```の対応

npm 実行時に

```bash:WARN
npm WARN lifecycle The node binary used for scripts is
C:\Program Files (x86)\Nodist\bin\node.exe but npm is using C:\Program Files (x86)\Nodist\v-x64\9.4.0\node.exe itself.
Use the `--scripts-prepend-node-path` option to include the path for the node binary npm was executed with.
```

上記 `WARN` が発生したら( 改行は筆者が行っている )

```bash:オプションセット
$ npm config set scripts-prepend-node-path true
```

してやれば良い｡

* 参考
  * [npm runコマンドを実行したら警告が出るようになった](https://qiita.com/isamusuzuki/items/738a2736a746b67bd977)



# 終わり

ここまでの手順で Node.js とそれに付随して npm を利用する環境が構築できた｡
あとは npm を使ってフロントエンドなりバックエンドなりの開発環境を整えれば良い｡
