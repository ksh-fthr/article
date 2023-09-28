## はじめに

本記事は Angular v4.x.x を元に記載しておりますが、現在の Angular のバージョンは **v6.x.x** (2018/08/07現在) であり、記載している内容は古くなっております。

最新の情報につきましては [Angular 日本語ドキュメンテーション](https://angular.jp/) から [プロジェクトファイルについて](https://angular.jp/guide/quickstart#%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6) をご確認ください。

## 本記事の主旨

[[Angular] 開発環境の構築](https://qiita.com/ksh-fthr/items/68c5e08a9cdac7c6b9ee) の記事のあと､ Angular に関する記事をあまり書いていなかったので､備忘録も兼ねて少しづつ記事を増やしていこうと思う｡ 
まずは導入編というところで､ Angular CLI で生成したプロジェクトの構成についての説明をば｡｡｡

かなり冗長､というかディレクトリやファイルについての説明なので退屈な記事だが､Angular で作成するアプリの構成は抑えておいた方が良いと考えた次第｡

ディレクトリの中も後述で説明する場合は､その項目を色付きで記載する｡

## そもそも Angular とは

プロジェクト構成の説明に入る前に､そもそも Angular とは何かというところから｡

Angular は Google が提供する JavaScript のフレームワークであり､ **コンポーネント指向** であるという特徴を持つ｡ ここでいうコンポーネントとはページを構成する UI 部品を指し､ロジック､ビュー､スタイルを定義する｡

Angular ではこのコンポーネントを組み合わせることでアプリを構築していくことになる｡

コンポーネントは基本的に独立した部品であり､コンポーネントのスタイルは **原則として定義したコンポーネントにのみ適用** される｡ つまり他のコンポーネントで定義されたスタイルと競合･衝突することは無い｡

開発言語は **TypeScript** が推奨されている｡ 本家のチュートリアルも TypeScript で説明されているので､特に嫌がる理由が無ければ TypeScript で開発を進めるのが良い｡

では Angular CLI で生成したプロジェクトの構成について次の項から説明を始める｡

## この記事を実施した環境

- Windows10 Home 64bit
- node v8.2.1
- npm v4.0.5
- angular/cli v1.2.6
- Angular v4.3.2

## プロジェクトディレクトリ
Angulr CLI で作成されたプロジェクトディレクトリ配下は次の通り｡
まずはプロジェクトディレクトリ配下にあるディレクトリやファイルの説明から｡

```bash:プロジェクトの作成
$ ng new my-app
```

```bash:ディレクトリの中を確認
$ cd my-app
$ ls -l
total 281
drwxr-xr-x 1 hogehoge 197121    0 9月  24 16:03 e2e/
-rw-r--r-- 1 hogehoge 197121  924 9月  24 16:03 karma.conf.js
drwxr-xr-x 1 hogehoge 197121    0 9月  24 16:06 node_modules/
-rw-r--r-- 1 hogehoge 197121 1311 9月  24 16:03 package.json
-rw-r--r-- 1 hogehoge 197121  722 9月  24 16:03 protractor.conf.js
-rw-r--r-- 1 hogehoge 197121 1075 9月  24 16:03 README.md
drwxr-xr-x 1 hogehoge 197121    0 9月  24 16:03 src/
-rw-r--r-- 1 hogehoge 197121  363 9月  24 16:03 tsconfig.json
-rw-r--r-- 1 hogehoge 197121 3040 9月  24 16:03 tslint.json
```

- e2e/
    - End To End テスト関連のディレクトリ｡

- karma.conf.js
    - ユニットテストのテストランナー Karma の設定ファイル｡ユニットテストの設定はここに記述する｡

- node_modules/
    - このプロジェクトで利用する npm ライブラリの配置先｡
    - `npm install` を `-g` をつけないで実行すると､このディレクトリ配下にインストールされる｡

- package.json
    - npm の設定ファイル｡このプロジェクトで利用するライブラリ情報を管理する｡
    - `npm install` で `--save` や `--save-dev` をつけて実行した場合､このファイルに自動で情報が追記される｡
    - 逆にこのファイルに記載されてさえいれば､ `npm install` を引数無しで実行した場合にプロジェクトに必要なライブラリがインストールされる｡

- protractor.conf.js
    - E2E テストのテストランナー Protractor の設定ファイル｡
    - E2E テストの設定はここに記述する｡

- README.md
    - このプロジェクトの説明ファイル｡
    - 具体的には作成するアプリについての説明を記述する｡

- <font color="red">src/</font>
    - <font color="red">このプロジェクトで作成するアプリ本体を格納するディレクトリ｡</font>
    - <font color="red">実装していくコードはここに配置していく｡</font>

- tsconfig.json
    - TypeScript を JavaScript にトランスパイルための設定情報ファイル｡
    - (Angular は基本的に TypeScript で実装していく)

- tslint.json
    - TSLint の設定情報｡

## src/ 配下

プロジェクト直下の構成は見たので､次に作成するアプリ本体である src/ 配下を確認する｡

```bash:src/配下の構成
$ cd src/
$ ls -l
total 30      
drwxr-xr-x 1 hogehoge 197121    0 9月  24 16:03 app/
drwxr-xr-x 1 hogehoge 197121    0 9月  24 16:03 assets/
drwxr-xr-x 1 hogehoge 197121    0 9月  24 16:03 environments/
-rw-r--r-- 1 hogehoge 197121 5430 9月  24 16:03 favicon.ico
-rw-r--r-- 1 hogehoge 197121  292 9月  24 16:03 index.html
-rw-r--r-- 1 hogehoge 197121  336 9月  24 16:03 main.ts
-rw-r--r-- 1 hogehoge 197121 2480 9月  24 16:03 polyfills.ts
-rw-r--r-- 1 hogehoge 197121   80 9月  24 16:03 styles.css
-rw-r--r-- 1 hogehoge 197121 1085 9月  24 16:03 test.ts
-rw-r--r-- 1 hogehoge 197121  211 9月  24 16:03 tsconfig.app.json
-rw-r--r-- 1 hogehoge 197121  304 9月  24 16:03 tsconfig.spec.json
-rw-r--r-- 1 hogehoge 197121  104 9月  24 16:03 typings.d.ts
```    

- <font color="red">app/</font>
    - <font color="red">Angular アプリのコード一式</font>

- assets/
    - 画像ファイルや npm 以外で導入する JavaScript ライブラリ等を配置する｡

- environments/
    - 本番(Product)環境とか開発(Development)環境とか､環境設定情報を記述したファイルを配置する｡

- favicon.ico
    - favicon｡ブックマーク時のアイコン｡

- index.html
    - アプリのトップページ｡
    - ここで読み込まれた Angular モジュールを元にアプリが構成されていく｡

- main.ts
    - スタートアップファイル｡
    - アプリを起動するためのスタートアップコードを記述する｡

- polyfills.ts
    - polyfill の設定情報を管理する｡
    - Angular が提供する機能に対応できていないブラウザにアプリを適応させたい場合､ここを編集する｡
    - 例えば Angular アプリを [IE11 に対応させたい](https://qiita.com/ksh-fthr/items/68c5e08a9cdac7c6b9ee#ie11%E5%AF%BE%E5%BF%9C20170902-%E8%BF%BD%E8%A8%98)場合など｡

- styles.css
    - スタイルシート｡
    - グローバルで適用させたい CSS はここに記述する｡

- test.ts
    - ユニットテストの設定ファイル｡

- tsconfig.app.json
    - TypeScript を JavaScript にトランスパイルための設定情報ファイル｡
    - アプリ用｡

- tsconfig.spec.json
    - TypeScript を JavaScript にトランスパイルための設定情報ファイル｡
    - テストコード用｡

- typings.d.ts
    - TypeScript の型定義情報を管理する｡

## src/app/ 配下
最後に Angular アプリのコード一式を配置する src/app/ 配下を見る｡

```bash:src/app/配下の構成
$ cd app/
$ ls -l
total 10
-rw-r--r-- 1 hogehoge 197121    0 9月  24 16:03 app.component.css
-rw-r--r-- 1 hogehoge 197121 1785 9月  24 16:03 app.component.html
-rw-r--r-- 1 hogehoge 197121  991 9月  24 16:03 app.component.spec.ts
-rw-r--r-- 1 hogehoge 197121  207 9月  24 16:03 app.component.ts
-rw-r--r-- 1 hogehoge 197121  314 9月  24 16:03 app.module.ts
```

- app.component.css
    - app.component という Angular コンポーネントの css テンプレートを記述する｡

- app.component.html
    - app.component という Angular コンポーネントの html テンプレートを記述する｡

- app.component.spec.ts
    - app.component という Angular コンポーネントのユニットテストを記述する｡

- app.component.ts
    - app.component という Angular コンポーネントの実装コードを記述する｡

- app.module.ts
    - この Angular プロジェクトで作成するアプリの ルートモジュールを記述する｡

## 終わりに
プロジェクト構成として各ディレクトリやファイルを見てきたので､次回はアプリを作成していく上で必要なファイルの中身を見ていく｡
具体的には

- src/
    - index.html
    - main.ts
- src/app/
    - app.component.css
    - app.component.html
    - app.component.ts
    - app.module.ts

を見ていく｡
