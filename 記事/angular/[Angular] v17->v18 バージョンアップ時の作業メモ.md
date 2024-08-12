# はじめに

Angular v18 がリリースされました。( [Angular v18 is now available!](https://blog.angular.dev/angular-v18-is-now-available-e79d5ac0affe) )
この記事では v18 への移行手順と、自分の実施したときにおきたトラブルとその対応についての備忘録になります。

# 対象アプリ

Angular 学習用のアプリケーションを移行対象とします。
リポジトリはこちらです。

- [ksh-fthr/angular-work](https://github.com/ksh-fthr/angular-work)

# 移行作業にあたって

公式より [移行手順](https://angular.dev/update-guide?v=17.0-18.0&l=1) が提供されています。
本記事ではこの移行手順に沿って作業を進めます。

## 作業前の環境

Angualr v18 に更新前の環境です。

| 環境                                                        | バージョン | 備考                     |
| ----------------------------------------------------------- | ---------- | ------------------------ |
| [Angular CLI](https://cli.angular.io/)                      | v17.3.8    | `ng version` で確認      |
| [Angular](https://angular.io/)                              | v17.3.8    | 同上                     |
| [Angular Material](https://material.angular.io/)            | v17.3.7    | 同上                     |
| [Angular CDK](https://github.com/angular/components#readme) | v17.3.7    | 同上                     |
| [RxJS](https://rxjs.dev/)                                   | v6.6.7     | 同上                     |
| [TypeScript](https://www.typescriptlang.org/)               | v5.4.5     | 同上                     |
| [zone.js](https://www.npmjs.com/package/zone.js)            | v0.14.5    | 同上                     |
| [Node.js](https://nodejs.org/ja/)                           | v20.14.0   | 同上                     |
| [npm](https://www.npmjs.com/)                               | v10.7.0    | 同上                     |

## 注意点

移行手順には次の記述があります。

> Make sure that you are using a supported version of node.js before you upgrade your application. Angular v18 supports node.js versions: v18.19.0 and newer

node.js のバージョンは `v18.19.0` 以上をサポート対象としています。
もしご利用の環境で node.js のバージョンがこれより低かったらバージョンアップしておきましょう。

# Angular v18 にアップデート

## エラー発生

に沿って実行したところ次のエラーが発生。

```bash
% ng update @angular/core@18 @angular/cli@18
The installed Angular CLI version is outdated.
Installing a temporary Angular CLI versioned 18.0.4 to perform the update.
✔ Packages successfully installed.
Using package manager: npm
Collecting installed dependencies...
Found 50 dependencies.
Fetching dependency metadata from registry...
                  Package "@angular-eslint/schematics" has an incompatible peer dependency to "@angular/cli" (requires ">= 17.0.0 < 18.0.0", would install "18.0.4").
✖ Migration failed: Incompatible peer dependencies found.
Peer dependency warnings when installing dependencies means that those dependencies might not work correctly together.
You can use the '--force' option to ignore incompatible peer dependencies and instead address these warnings later.
  See "/private/var/folders/_8/1kwkmmvn2136hjlg0h2r68l00000gn/T/ng-s0Ef98/angular-errors.log" for further details.
```

## このときの pakcage.json

```json
  "devDependencies": {
    ..(略)..
    "@angular-eslint/eslint-plugin": "17.5.2",
    "@angular-eslint/eslint-plugin-template": "17.5.2",
    "@angular-eslint/schematics": "17.5.2",
    "@angular-eslint/template-parser": "17.5.2",
    ..(略)..
  }
```

上記のうち `@angular-eslint/schematics` でエラーが出ていたので、一旦これらの `@angular-eslint` 関連のパッケージを package.json から削除しました。

## 移行コマンドを再実行

```bash
% ng update @angular/core@18 @angular/cli@18
```

今度は無事成功しました。以下はコマンド実行時のログです。
ログ中に記載のとおり、 `use-application-builder` の利用について確認がでました。

※ `use-application-builder` についてこちらをご参照ください
- [Getting started with the Angular CLI's new build system](https://angular.jp/guide/esbuild)
- [use-application-builder](https://robert-isaac.medium.com/angular-v17-the-application-builder-2482979648bf)
- [ビルドが100倍速くなると噂の Angular 17 の新ビルドシステムを使ってみた](https://qiita.com/sakakig/items/bd4a3a6892a524f9a060)
- [Speeding up our Angular app with esbuild](https://medium.com/@mathieu.schnoor/speeding-up-our-angular-app-with-esbuild-3f7b0b716bef)

これまではビルドツールに [webpack](https://webpack.js.org/) が使われていましたが、今後は [esbuild](https://esbuild.github.io/) が標準になるそうです。
というわけで、ここは利用する方向で進めていきます。

```bash
% ng update @angular/core@18 @angular/cli@18
The installed Angular CLI version is outdated.
Installing a temporary Angular CLI versioned 18.0.4 to perform the update.
✔ Packages successfully installed.
Using package manager: npm
Collecting installed dependencies...
Found 46 dependencies.
Fetching dependency metadata from registry...
    Updating package.json with dependency @angular-devkit/build-angular @ "18.0.4" (was "17.3.8")...
    Updating package.json with dependency @angular/cli @ "18.0.4" (was "17.3.8")...
    Updating package.json with dependency @angular/compiler-cli @ "18.0.3" (was "17.3.11")...
    Updating package.json with dependency @angular/animations @ "18.0.3" (was "17.3.11")...
    Updating package.json with dependency @angular/common @ "18.0.3" (was "17.3.11")...
    Updating package.json with dependency @angular/compiler @ "18.0.3" (was "17.3.11")...
    Updating package.json with dependency @angular/core @ "18.0.3" (was "17.3.11")...
    Updating package.json with dependency @angular/forms @ "18.0.3" (was "17.3.11")...
    Updating package.json with dependency @angular/platform-browser @ "18.0.3" (was "17.3.11")...
    Updating package.json with dependency @angular/platform-browser-dynamic @ "18.0.3" (was "17.3.11")...
    Updating package.json with dependency @angular/router @ "18.0.3" (was "17.3.11")...
UPDATE package.json (1900 bytes)
✔ Packages successfully installed.
** Optional migrations of package '@angular/cli' **

This package has 1 optional migration that can be executed.
Optional migrations may be skipped and executed after the update process, if preferred.

# use-application-builder を選択するために space を押下して継続する
Select the migrations that you'd like to run (Press <space> to select, <a> to toggle all, <i> to invert selection, and <enter> to proceed)
❯◉ [use-application-builder] Migrate application projects to the new build system. (https://angular.dev/tools/cli/build-system-migration)

# 後処理が走る

❯ Migrate application projects to the new build system.
  Application projects that are using the '@angular-devkit/build-angular' package's 'browser' and/or 'browser-esbuild' builders will be migrated to use the new 'application' builder.
  You can read more about this, including known issues and limitations, here: https://angular.dev/tools/cli/build-system-migration
    The output location of the browser build has been updated from "dist/angular-app" to "dist/angular-app/browser". You might need to adjust your deployment pipeline or, as an alternative, set outputPath.browser to "" in order to maintain the previous functionality.
UPDATE angular.json (4230 bytes)
UPDATE tsconfig.json (772 bytes)
  Migration completed (2 files modified).

** Executing migrations of package '@angular/core' **

❯ Updates two-way bindings that have an invalid expression to use the longform expression instead.
  Migration completed (No changes made).

❯ Replace deprecated HTTP related modules with provider functions.
UPDATE src/app/app.module.ts (5107 bytes)
  Migration completed (1 file modified).
```

## 移行コマンドによって修正されたファイル

次のファイルがコマンドによって修正されました。
差分については [こちら](https://github.com/ksh-fthr/angular-work/pull/655/commits/71fa85121f9115a98e7080506bfcb411f8a49a23) をご参照ください。

```bash
% git status
On branch feature/update/angular-v18
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   angular.json
        modified:   package-lock.json
        modified:   package.json
        modified:   src/app/app.module.ts
no changes added to commit (use "git add" and/or "git commit -a")
```

## ビルドツールが変わっています

先にあげたビルドツールは angular.json のこの部分で指定されます。

```json
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:application",
        }
      }
```

`application` か `browser-esbuild` を指定できます。デフォルトは `application` です。
[Angular 公式](https://angular.jp/guide/esbuild) に詳細がありますのでご参照ください。


# Angular Material もアップデート

## 移行コマンドを実行

Angular Material を利用しているので、こちらも移行手順に従いアップデートします。

```bash
% ng update @angular/material@18
```

とくに何事もなく無事終了。
実行ログはこちらです。

```bash
% ng update @angular/material@18
Using package manager: npm
Collecting installed dependencies...
Found 46 dependencies.
Fetching dependency metadata from registry...
    Updating package.json with dependency @angular/cdk @ "18.0.3" (was "17.3.10")...
    Updating package.json with dependency @angular/material @ "18.0.3" (was "17.3.10")...
UPDATE package.json (1900 bytes)
✔ Packages successfully installed.
** Executing migrations of package '@angular/cdk' **

❯ Updates the Angular CDK to v18.

      ✓  Updated Angular CDK to version 18

  Migration completed (No changes made).

** Executing migrations of package '@angular/material' **

❯ Updates Angular Material to v18.

      ✓  Updated Angular Material to version 18

  Migration completed (No changes made).
```

## 移行コマンドによって修正されたファイル

package.json と package-lock.json が修正されたのみでした。
差分については [こちら](https://github.com/ksh-fthr/angular-work/pull/655/commits/d8b23382fe8bca7b743a564195e4d189d2eaa2d1) をご参照ください。

```bash
 % git status
On branch feature/update/angular-v18
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   package-lock.json
        modified:   package.json

no changes added to commit (use "git add" and/or "git commit -a")
```

# TypeScript をアップデート

## `npm i` で最新版をインストール

TypeScript も移行手順に従い `v5.4` にアップデートします。
( [作業前の環境](#作業前の環境) にあるとおり既に `v5.4.5` が入っていますが、明示的に実行いて package.json も更新しておきます )

```bash
% npm i typescript
```

こちらは単純に `npm i` でパッケージを更新しただけなので、修正ファイルのリストは割愛します。
( [package.json, package-log.json が更新されただけ](https://github.com/ksh-fthr/angular-work/pull/655/commits/28fb58277627ae5ce08239e6fde0291f33fc7197) です )

# 移行作業後の確認

## 作業後の環境

Angualr v18 に更新後の環境です。

| 環境                                                        | バージョン | 備考                     |
| ----------------------------------------------------------- | ---------- | ------------------------ |
| [Angular CLI](https://cli.angular.io/)                      | v18.0.4    | `ng version` で確認      |
| [Angular](https://angular.io/)                              | v18.0.3    | 同上                     |
| [Angular Material](https://material.angular.io/)            | v18.0.3    | 同上                     |
| [Angular CDK](https://github.com/angular/components#readme) | v18.0.3    | 同上                     |
| [RxJS](https://rxjs.dev/)                                   | v6.6.7     | 同上                     |
| [TypeScript](https://www.typescriptlang.org/)               | v5.4.5     | 同上                     |
| [zone.js](https://www.npmjs.com/package/zone.js)            | v0.14.7    | 同上                     |
| [Node.js](https://nodejs.org/ja/)                           | v20.14.0   | 同上                     |
| [npm](https://www.npmjs.com/)                               | v10.7.0    | 同上                     |

<details>
<summary>ng version の情報</summary>

```bash
% ng version

     _                      _                 ____ _     ___
    / \   _ __   __ _ _   _| | __ _ _ __     / ___| |   |_ _|
   / △ \ | '_ \ / _` | | | | |/ _` | '__|   | |   | |    | |
  / ___ \| | | | (_| | |_| | | (_| | |      | |___| |___ | |
 /_/   \_\_| |_|\__, |\__,_|_|\__,_|_|       \____|_____|___|
                |___/


Angular CLI: 18.0.4
Node: 20.14.0
Package Manager: npm 10.7.0
OS: darwin x64

Angular: 18.0.3
... animations, cdk, common, compiler, compiler-cli, core, forms
... material, platform-browser, platform-browser-dynamic, router

Package                         Version
---------------------------------------------------------
@angular-devkit/architect       0.1800.4
@angular-devkit/build-angular   18.0.4
@angular-devkit/core            18.0.4
@angular-devkit/schematics      18.0.4
@angular/cli                    18.0.4
@schematics/angular             18.0.4
rxjs                            6.6.7
typescript                      5.4.5
zone.js                         0.14.7
```

</details>

## 起動確認

さて、バージョンアップが無事終了したので起動確認です。

```bash
% npm run start

> angular-app@0.0.0 start
> ng serve --proxy-config proxy.conf.json

Initial chunk files | Names         |  Raw size
main.js             | main          | 438.26 kB |
polyfills.js        | polyfills     | 157.85 kB |
styles.css          | styles        | 100.95 kB |

                    | Initial total | 697.07 kB

Application bundle generation complete. [4.928 seconds]

Watch mode enabled. Watching for file changes...
NOTE: Raw file sizes do not reflect development server per-request transformations.
  ➜  Local:   http://localhost:4200/
  ➜  press h + enter to show help
```

アプリのビルドは成功しているようです。

...が、上記 URL にアクセスしたところ画面が真っ白でした。
devtools よりコンソールを見てみると...

```bash
Uncaught Error: Dynamic require of "microphone-stream" is not supported
    at main.js:6:9
    at use-aws-transcribe-streaming.component.ts:19:26
```

と出ています。
たしかに今回対象としたアプリケーションでは `microphone-stream` を利用しています。

[この部分](https://github.com/ksh-fthr/angular-work/blob/develop/src/app/component/speech-to-text/use-aws-transcribe-streaming/use-aws-transcribe-streaming.component.ts#L17-L20) ですね。

```ts
// AWS Transcribe Streaming に流す audio データを作るのに必要
// https://github.com/microphone-stream/microphone-stream#readme
const MicrophoneStream = require('microphone-stream').default;
```

## エラーの原因

どうも [移行コマンドを再実行](#移行コマンドを再実行) において `use-application-builder` を選択したことで angular.json が更新されたことが原因のようです。
angular.json の差分はこちらです。 angular.json の修正を戻したら起動後の動作検証も無事行えました。

<details>
<summary>angular.json 差分</summary>

```bash
diff --git a/angular.json b/angular.json
index c959573..8534952 100644
--- a/angular.json
+++ b/angular.json
@@ -15,12 +15,15 @@
       "prefix": "app",
       "architect": {
         "build": {
-          "builder": "@angular-devkit/build-angular:browser",
+          "builder": "@angular-devkit/build-angular:application",
           "options": {
-            "outputPath": "dist/angular-app",
+            "outputPath": {
+              "base": "dist/angular-app"
+            },
             "index": "src/index.html",
-            "main": "src/main.ts",
-            "polyfills": "src/polyfills.ts",
+            "polyfills": [
:...skipping...
diff --git a/angular.json b/angular.json
index c959573..8534952 100644
--- a/angular.json
+++ b/angular.json
@@ -15,12 +15,15 @@
       "prefix": "app",
       "architect": {
         "build": {
-          "builder": "@angular-devkit/build-angular:browser",
+          "builder": "@angular-devkit/build-angular:application",
           "options": {
-            "outputPath": "dist/angular-app",
+            "outputPath": {
+              "base": "dist/angular-app"
+            },
             "index": "src/index.html",
-            "main": "src/main.ts",
-            "polyfills": "src/polyfills.ts",
+            "polyfills": [
+              "src/polyfills.ts"
+            ],
             "tsConfig": "tsconfig.app.json",
             "assets": [
               "src/favicon.ico",
:...skipping...
diff --git a/angular.json b/angular.json
index c959573..8534952 100644
--- a/angular.json
+++ b/angular.json
@@ -15,12 +15,15 @@
       "prefix": "app",
       "architect": {
         "build": {
-          "builder": "@angular-devkit/build-angular:browser",
+          "builder": "@angular-devkit/build-angular:application",
           "options": {
-            "outputPath": "dist/angular-app",
+            "outputPath": {
+              "base": "dist/angular-app"
+            },
             "index": "src/index.html",
-            "main": "src/main.ts",
-            "polyfills": "src/polyfills.ts",
+            "polyfills": [
+              "src/polyfills.ts"
+            ],
             "tsConfig": "tsconfig.app.json",
             "assets": [
               "src/favicon.ico",
@@ -32,9 +35,7 @@
               "src/styles.css"
             ],
             "scripts": [],
-            "vendorChunk": true,
             "extractLicenses": false,
-            "buildOptimizer": false,
             "sourceMap": true,
             "optimization": false,
             "namedChunks": true,
@@ -45,7 +46,8 @@
               "@aws-crypto/sha256-browser",
               "@aws-crypto/crc32",
               "microphone-stream"
-            ]
+            ],
+            "browser": "src/main.ts"
           },
           "configurations": {
             "production": {
@@ -60,8 +62,6 @@
               "sourceMap": false,
               "namedChunks": false,
               "extractLicenses": true,
-              "vendorChunk": false,
-              "buildOptimizer": true,
               "budgets": [
                 {
                   "type": "initial",
```
</details>

## 対応

とはいえ、今後は esbuild を使う `application` が標準になるとのことなので、angular.json を戻さずにできる方法が必要です。
以下、自分なりに調べたこと・試したことです。

### import に変更する

今回はトップレベルでの定義でしたので、そのまま `import` への置き換えが可能です。
( 参考: https://azukiazusa.dev/blog/vite-require/#require-%E3%82%92-import-%E3%81%AB%E7%BD%AE%E3%81%8D%E6%8F%9B%E3%81%88%E3%82%8B )

```ts
// AWS Transcribe Streaming に流す audio データを作るのに必要
// https://github.com/microphone-stream/microphone-stream#readme
// const MicrophoneStream = require('microphone-stream').default;
import MicrophoneStream from 'microphone-stream';
```

が、こうしたところ次のエラーになりました。

```bash
✘ [ERROR] TS7016: Could not find a declaration file for module 'readable-stream'. '/Users/ksh-fthr/workspace/angular-work/node_modules/readable-stream/readable.js' implicitly has an 'any' type.
  Try `npm i --save-dev @types/readable-stream` if it exists or add a new declaration (.d.ts) file containing `declare module 'readable-stream';` [plugin angular-compiler]

    node_modules/microphone-stream/types/microphone-stream.d.ts:2:25:
      2 │ import { Readable } from "readable-stream";
        ╵                          ~~~~~~~~~~~~~~~~~
```

型定義ファイルが無いと怒っているので、指示に従い下記で型定義ファイルをインストールします。

```bash
% npm i --save-dev @types/readable-stream
```

エラーになることなく無事インストールできました。
これで型定義の問題も解決したと思いましたが、今度は別のエラーが発生しました。

```bash
✘ [ERROR] TS2416: Property 'pipe' in type 'Duplex' is not assignable to the same property in base type '_Writable'.
  Type '<S extends _IWritable>(dest: S, pipeOpts?: { end?: boolean | undefined; } | undefined) => S' is not assignable to type '<T extends WritableStream>(destination: T, options?: { end?: boolean | undefined; } | undefined) => T'.
    Types of parameters 'dest' and 'destination' are incompatible.
      Type 'T' is not assignable to type '_IWritable'.
        Type 'WritableStream' is not assignable to type '_IWritable'.
          The types returned by 'end(...)' are incompatible between these types.
            Type 'void' is not assignable to type '_IWritable'. [plugin angular-compiler]

    node_modules/@types/readable-stream/index.d.ts:355:8:
      355 │         pipe<S extends _IWritable>(dest: S, pipeOpts?: { end?: bo...
          ╵         ~~~~
```

ここまでくると、ライブラリの中の話になるので追うのはやめました。

## 残念

すぐの解決は難しいと判断し、今回は [該当するコンポーネントをコメントアウト](https://github.com/ksh-fthr/angular-work/pull/655/commits/6a704eaf51c5d5803b0ce38045d2bc8103ea640b) することで回避しました。

# まとめにかえて

一部使用しているライブラリによってスムーズに移行を進めることはできませんでしが、とりあえず v18 に上げることができました。
Angular v18 では zonless が experimental ですが導入されてます。 [Signals](https://angular.jp/guide/signals) とともに今後の標準になっていくと思われますので、学習を進めていたいですね。
