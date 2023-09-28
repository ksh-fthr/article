# Parcel

[Parcel](https://ja.parceljs.org/) は Webアプリケーションバンドラー で、設定不要を特徴としています。
ドキュメントは [こちら](https://parceljs.org/getting-started/webapp/)。

## 導入

`npm` と `yarn` でのインストールをサポートしていて、次のどちらでもインストールできます。

```bash
% yarn global add parcel
```

```bash
% npm install -g parcel
```

これだけで OK。あとは package.json を作成して Parcel プロジェクトを作っていきます。

※ 注

- なお [日本語ドキュメント](https://ja.parceljs.org/getting_started.html) では `parcel-bundler` を指定していますが、これは v1 の内容になります。ここでは [v2 のドキュメント](https://parceljs.org/getting-started/webapp/) の方法に倣い `parcel` を指定しています。
- parcel と parcel-bundler については [こちら](https://zenn.dev/aumy/articles/parcel-vs-parcel-bundler) を参照

## プロジェクトの作成

package.json は次のコマンドで作成されます。コマンドの実行はプロジェクトフォルダ直下( `プロジェクトルート` )で実行します。

```bash
% cd ${プロジェクトルート}/
```

```bash
% yarn init -y
% yarn add parcel --dev
```

```bash
% npm init -y
% npm install parcel --save-dev
```

## package.json サンプル

学習用リポジトリから [package.json](https://github.com/ksh-fthr/html-javascript-work/blob/master/package.json) 。

<details>
    <summary>package.json</summary>

```json
{
  "name": "html-javascript-work",
  "version": "1.0.0",
  "description": "本リポジトリは HTML5, CSS3, JavaScript(ES2015 以降) の学習用リポジトリです。",
  "browserslist": "> 0.5%, last 2 versions, not dead",
  "source": "./work/index.html", // parcel コマンドでターゲットとするソースを指定
  "scripts": {
    "start": "parcel --open",  // parcel コマンドでアプリを起動
    "build": "parcel build",   // parcel コマンドでアプリをビルド
    "test-prepare": "bash ./prepareTest.sh",
    "test:ch": "karma start --browsers Chrome",
    "test:fx": "karma start --browsers Firefox",
    "lint": "eslint ./**/*.js --fix"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/ksh-fthr/html-javascript-work.git"
  },
  "author": "ksh-fthr",
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/ksh-fthr/html-javascript-work/issues"
  },
  "homepage": "https://github.com/ksh-fthr/html-javascript-work#readme",
  "dependencies": {
    "karma": "6.4.1",
    "tw-elements": "^1.0.0-beta1"
  },
  "devDependencies": {
    "eslint": "8.38.0",
    "eslint-config-standard": "^17.0.0",
    "eslint-plugin-import": "^2.26.0",
    "eslint-plugin-node": "^11.1.0",
    "eslint-plugin-promise": "^6.1.1",
    "jasmine-core": "3.99.1",
    "karma-chrome-launcher": "3.1.1",
    "karma-firefox-launcher": "2.1.2",
    "karma-html2js-preprocessor": "1.1.0",
    "karma-jasmine": "4.0.2",
    "parcel": "latest",
    "parcel-bundler": "^1.12.5",
    "postcss": "^8.4.21",
    "tailwindcss": "^3.2.6"
  }
}
```

</details>

## アプリを起動

次のコマンドでアプリを起動します。

```bash
% npm run start

> html-javascript-work@1.0.0 start
> parcel --open

Server running at http://localhost:1234
✨ Built in 2.76s
```

## フレームワーク等々との組み合わせ

公式ドキュメントでは React, Preact, Vue 等々と Parcel の組み合わせについて紹介されてます。
それぞれのリンクを貼っておきますのでご興味あれば御覧ください。

- **[React との組み合わせ](https://ja.parceljs.org/recipes.html#react)**
- **[Preact との組み合わせ](https://ja.parceljs.org/recipes.html#preact)**
- **[Vue との組み合わせ](https://ja.parceljs.org/recipes.html#vue)**
- **[Svelt との組み合わせ](https://ja.parceljs.org/recipes.html#svelte)**

( Preact というものを知らなかったので公式と関連記事のリンクも貼っておきます)

- [Preact](https://preactjs.com/)
- [Preactの特徴](https://www.codegrid.net/articles/2020-preact-1/)

( Svelt についても一応貼っておきます )

- [Svelt](https://svelte.jp/)
- [Svelte をはじめる](https://developer.mozilla.org/ja/docs/Learn/Tools_and_testing/Client-side_JavaScript_frameworks/Svelte_getting_started) ( mdn で記事が用意されていることに驚きました )

## 補足

### ホットリロード

設定不要を謳っている Parcel だけあり、ソースコード修正時のホットリロードも設定不要で行ってくれます。
ただたまにホットリロードが効かないときがありますので、そんなときは Parcel を `Ctrl-C` で終了して再起動させます。

### キャッシュクリア

またアプリを起動しても修正した内容が反映されないケースもたまにあります。そんなときは Parcel のキャッシュをクリアすることで解消されます。
キャッシュクリアは下記を実行します。

```bash
% pwd
${プロジェクトルート}
% rm -rf .parcel-cache/
```

# Tailwind CSS

公式は [こちら](https://tailwindcss.com/)。[チートシート](https://nerdcave.com/tailwind-cheat-sheet)なんかもあります。
そして Tailwind CSS を用いたコンポーネント群もあります。 -> [Tailwind Elements](https://tailwind-elements.com/)

## どんなもの？

使いやすさ重視の CSS フレームワーク。提供された CSS クラスを適用させることで Web ページのデザインを簡単に実現させることができる、とのことです。

( 公式 TOP から引用 )

> A utility-first CSS framework packed with classes like flex, pt-4, text-center and rotate-90 that can be composed to build any design, directly in your markup.

## Tailwind CSS の導入

[Get started](https://tailwindcss.com/docs/installation) には CDN 経由での利用や CLI 、フレームワーク経由でのインストールについて手順が載っています。
今回は [Parcel 経由での導入](https://tailwindcss.com/docs/guides/parcel) に則って導入していきます。

### 手順にそってインストール

```bash
% npm install -D parcel
% npm install -D tailwindcss postcss
% npx tailwindcss init
```

### postcss と tailwind の設定

※ [postcss](https://postcss.org/) については [こちら](https://qiita.com/morishitter/items/4a04eb144abf49f41d7d) の説明が良さげ

```json:.postcssrc
{
  "plugins": {
    "tailwindcss": {}
  }
}
```

```js:tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    // ここは実際の環境にあわせて設定する
    "./src/**/*.{html,js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

### Tailwind CSS の読み込み

```css:index.css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Tailwind Elements の導入

前述しましたが [Tailwind Elements](https://tailwind-elements.com/) は Tailwind CSS を用いた UI コンポーネント群です。
BootStrap をベースに作られているとのことです。

( 公式 TOP から引用 )

> Bootstrap components recreated with Tailwind CSS, but with better design and more functionalities

[Get started](https://tailwind-elements.com/docs/standard/getting-started/quick-start/) を参考に、今回は `npm` を用いた方法で進めます。

### 手順にそってインストール

```bash
% npm install tw-elements
```

### Tailwind の設定を更新

```js:tailwind.config.js
module.exports = {
  content: [
    // ここは実際の環境にあわせて設定する
    "./src/**/*.{html,js,ts,jsx,tsx}",
    "./node_modules/tw-elements/dist/js/**/*.js"
  ],
  plugins: [require("tw-elements/dist/plugin.cjs")],
};
```

### index.js で読み込む

```js:index.js
import 'tw-elements';
```

参考: [JSモジュール化するしくみの規格はCommonJS、AMD、UMD、ES6(ES2015)の4つ](https://ytyaru.hatenablog.com/entry/2019/03/29/000000)

### index.html で読み込む

```html:index.html
<!doctype html>
<html lang='ja'>
  <head>
    <title>Tailwind CSS</title>
    <meta charset='utf-8'>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel='stylesheet' href='./css/use-tailwind.css'>
    <!-- `Browser scripts cannot have imports or exports` に対応するために `type="module"` が必要. -->
    <!-- 参考: https://github.com/parcel-bundler/parcel/discussions/6490 -->
    <script type="module" src="./use-tailwind.js"></script>
  </head>
  <body>
  </body>
</html>
```

参考: [HTML にモジュールを適用する](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Modules#html_%E3%81%AB%E3%83%A2%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AB%E3%82%92%E9%81%A9%E7%94%A8%E3%81%99%E3%82%8B)

## お試し

[Tailwind CSS の Font Size](https://tailwindcss.com/docs/font-size) を試します。

**フォント**
<img width="588" alt="スクリーンショット 2023-04-18 20.01.15.png (138.4 kB)" src="https://img.esa.io/uploads/production/attachments/17638/2023/04/18/102830/9f08cb00-4809-4e42-99d2-3747a8f39ac9.png">

```html
          <h1 class="text-2xl font-bold underline"> Hello world! (Tailwind CSS) </h1>
          <h1 class="text-4xl font-bold underline"> Hello world! (Tailwind CSS) </h1>
          <h1> Hello world! (no style) </h1>
```

**グリッド**

[Tailwind CSS の Grid](https://tailwindcss.com/docs/display#grid) を試します。

<img width="1572" alt="スクリーンショット 2023-04-18 20.02.35.png (209.8 kB)" src="https://img.esa.io/uploads/production/attachments/17638/2023/04/18/102830/ac8b8452-7649-45fc-8874-04deb5a524f6.png">

```html
        <div
          class="hidden opacity-0 transition-opacity duration-150 ease-linear data-[te-tab-active]:block"
          id="tabs-profile"
          role="tabpanel"
          aria-labelledby="tabs-profile-tab"
        >
          <!-- grid: 1列4カラム -->
          <div class="grid grid-cols-4 gap-4 bg-indigo-100">
            <div>01</div>
            <div>02</div>
            <div>03</div>
            <div>04</div>
            <div>05</div>
            <div>06</div>
            <div>07</div>
            <div>08</div>
          </div>
          <!-- grid: 1列2カラム -->
          <div class="grid grid-cols-2 gap-4 bg-indigo-50">
            <div>01</div>
            <div>02</div>
            <div>03</div>
            <div>04</div>
            <div>05</div>
            <div>06</div>
            <div>07</div>
            <div>08</div>
          </div>
        </div>
```

**コンテナ**
[Tailwind CSS の Container](https://tailwindcss.com/docs/container) を試します。

```html
    <div class="md:container md:mx-auto">
    </div>
```

**タブ**
こちらは [Tailwind Elements の Tabs](https://tailwind-elements.com/docs/standard/navigation/tabs/) を利用して実現します。

<img width="437" alt="スクリーンショット 2023-04-18 20.07.05.png (38.6 kB)" src="https://img.esa.io/uploads/production/attachments/17638/2023/04/18/102830/d0354e67-b3c0-4f51-a45f-b8c8f6d74366.png">

```html
      <ul
        class="mb-5 flex list-none flex-col flex-wrap border-b-0 pl-0 md:flex-row"
        role="tablist"
        data-te-nav-ref
      >
        <li role="presentation">
          <a
            href="#tabs-home"
            class="my-2 block border-x-0 border-b-2 border-t-0 border-transparent px-7 pb-3.5 pt-4 text-xs font-medium uppercase leading-tight text-neutral-500 hover:isolate hover:border-transparent hover:bg-neutral-100 focus:isolate focus:border-transparent data-[te-nav-active]:border-primary data-[te-nav-active]:text-primary dark:text-neutral-400 dark:hover:bg-transparent dark:data-[te-nav-active]:border-primary-400 dark:data-[te-nav-active]:text-primary-400"
            data-te-toggle="pill"
            data-te-target="#tabs-home"
            data-te-nav-active
            role="tab"
            aria-controls="tabs-home"
            aria-selected="true"
          >
            文字サイズの調整
          </a>
        </li>
        <li role="presentation">
          <a
            href="#tabs-profile"
            class="focus:border-transparen my-2 block border-x-0 border-b-2 border-t-0 border-transparent px-7 pb-3.5 pt-4 text-xs font-medium uppercase leading-tight text-neutral-500 hover:isolate hover:border-transparent hover:bg-neutral-100 focus:isolate data-[te-nav-active]:border-primary data-[te-nav-active]:text-primary dark:text-neutral-400 dark:hover:bg-transparent dark:data-[te-nav-active]:border-primary-400 dark:data-[te-nav-active]:text-primary-400"
            data-te-toggle="pill"
            data-te-target="#tabs-profile"
            role="tab"
            aria-controls="tabs-profile"
            aria-selected="false"
          >
            グリッド
          </a>
        </li>
      </ul>

      <!--Tabs content-->
      <div class="mb-6">
        <div
          class="hidden opacity-0 opacity-100 transition-opacity duration-150 ease-linear data-[te-tab-active]:block"
          id="tabs-home"
          role="tabpanel"
          aria-labelledby="tabs-home-tab"
          data-te-tab-active
        >
          フォントサイズ
        </div>

        <div
          class="hidden opacity-0 transition-opacity duration-150 ease-linear data-[te-tab-active]:block"
          id="tabs-profile"
          role="tabpanel"
          aria-labelledby="tabs-profile-tab"
        >
          グリッド
        </div>
      </div>
```

# 所感

## Parcel

Parcel はちょっとしたアプリを作って試してみたい、というときに便利です。
なんせ導入と設定のハードルが低いので、思いついたらすぐにプロジェクト作成して立ち上げができます。
フレームワークとの連携も公式で手順が示されているのも良いですね。

## Tailwind CSS

Tailwind CSS も便利は便利です。
用意された CSS Class を使うだけで面倒な Grid も簡単に実現できました。
反面、CSS を学習することなく利用・実装できてしまうので、その点はエンジニアの育成という点で不安を感じます。
また指定する Class が多いために html が汚くなります。このへんは [こちら](https://coliss.com/articles/build-websites/operation/css/why-tailwind-css-is-not-for-me.html) で記事がありますのでご興味あれば。
