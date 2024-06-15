ng update @angular-eslint/schematics@rc-v18


# 移行手順

公式より [移行手順](https://angular.dev/update-guide?v=17.0-18.0&l=1) が提供されています。
本記事ではこの移行手順に沿って作業を進めます。

# 作業環境

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

## 再度 移行コマンドを実行

```bash
% ng update @angular/core@18 @angular/cli@18
```

今度は無事成功しました。以下はコマンド実行時のログです。
ログ中に記載のとおり、 `use-application-builder` の利用について確認がでました。今回は利用する方向で進めています。

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
差分については こちら をご参照ください。

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

# Angular Material もアップデート

## 移行コマンドを実行

Angular Material を利用しているので、移行手順に従いアップデートします。

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
差分については こちら をご参照ください。

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

```bash
% npm i typescript
```

こちらは単純に `npm i` でパッケージを更新しただけなので、修正ファイルのリストは割愛します。
( package.json, package-log.json が更新されただけです )

# バージョンアップ後の確認

## 各種情報

`ng version` で情報を見てみます。
Angular, Angular Material, TypeScript 等、今回更新対象としたパッケージのバージョンが指定したものであることが確認できます。

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

アプリは無事起動はしましたが、上記 URL にアクセスしたところ画面が真っ白でした。
devtools よりコンソールを見てみると...


```bash
Uncaught Error: Dynamic require of "microphone-stream" is not supported
    at main.js:6:9
    at use-aws-transcribe-streaming.component.ts:19:26
```

と出ています。
たしかに今回対象としたアプリケーションでは `microphone-stream` を利用しています。

この部分ですね。

```ts
// AWS Transcribe Streaming に流す audio データを作るのに必要
// https://github.com/microphone-stream/microphone-stream#readme
const MicrophoneStream = require('microphone-stream').default;
```
