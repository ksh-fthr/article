# はじめに

Angular v17 で新しい制御構文が導入されました｡
本記事では既存の制御構文を新しい制御構文で書き換えてみます｡

# 環境

今回の記事の内容は次の環境で実施したものです。

| 環境                                                        | バージョン | 備考                     |
| ----------------------------------------------------------- | ---------- | ------------------------ |
| [Angular CLI](https://cli.angular.io/)                      | v17.0.10   | `ng version` で確認      |
| [Angular](https://angular.io/)                              | v17.0.9    | 同上                     |
| [TypeScript](https://www.typescriptlang.org/)               | v5.2.2     | 同上                     |
| [zone.js](https://www.npmjs.com/package/zone.js)            | v0.14.3    | 同上                     |
| [Node.js](https://nodejs.org/ja/)                           | v18.19.0   | 同上                     |
| [npm](https://www.npmjs.com/)                               | v10.2.3    | 同上                     |

# 今回扱うソースコード

対象のソースコードは私の学習リポジトリである [こちら](https://github.com/ksh-fthr/angular-work) になります｡

# とはいってもコマンド一発たたくだけ

書き換えると言ったものの､移行のためのマイグレーションコマンドが用意されております｡
今回はそれを叩いてみました｡

```bash:マイグレーションコマンド
% ng generate @angular/core:control-flow
```

コマンドを実行したときのログが下記になります｡
ログ中の `#` によるコメントは私が記載したものです｡

```bash:実行時のログ
# 対象のディレクトリを指定します. こんかいはリポジトリルートで実行したので `./` としました
? Which path in your project should be migrated? ./

# html テンプレートのマイグレーションを行うか確認されましたので Yes としました
? Should the migration reformat your templates? Yes
    IMPORTANT! This migration is in developer preview. Use with caution.

# 実際にマイグレーションが行われました
UPDATE src/app/component/use-attribute-directive/attribute-directive-validator-verification/attribute-directive-validator-verification.component.html (1736 bytes)
UPDATE src/app/component/validation/validation-verification/validation-verification.component.html (3024 bytes)
UPDATE src/app/component/reactive-form/reactive-form-verification/reactive-form-verification.component.html (2637 bytes)
UPDATE src/app/component/http-client/http-client-verification/http-client-verification.component.html (2561 bytes)
UPDATE src/app/component/use-angular-material/autocomplete/highlight-firs-item/highlight-firs-item.component.html (1747 bytes)
UPDATE src/app/component/use-angular-material/control-functions/control-functions.component.html (341 bytes)
UPDATE src/app/component/tab/switch-tab/switch-tab.component.html (341 bytes)
```

# 差分をみてみる

今回マイグレーションが行われたのは 7ファイル でした｡すべての差分を出すのは冗長ですので､対象を絞って差分を以下に示します｡
( 対象のリポジトリでは `*ngFor` と `*ngIf` しか扱っていなかったので `[ngSwitch]` の例はお見せすることができません｡ご容赦ください )

## `*ngFor` から `@for` に移行

### 対象ソースコード

- [src/app/component/tab/switch-tab/switch-tab.component.html](https://github.com/ksh-fthr/angular-work/blob/develop/src/app/component/tab/switch-tab/switch-tab.component.html)

### 差分

```bash
% git diff src/app/component/tab/switch-tab/switch-tab.component.html
diff --git a/src/app/component/tab/switch-tab/switch-tab.component.html b/src/app/component/tab/switch-tab/switch-tab.component.html
index 970a4a2..13c7231 100644
--- a/src/app/component/tab/switch-tab/switch-tab.component.html
+++ b/src/app/component/tab/switch-tab/switch-tab.component.html
@@ -1,8 +1,8 @@
 <div class="tab-area-base">
   <ul class="tab-menu-base">
-    <ng-container *ngFor="let tab of tabs">
-        <li [class.current]="tab.current" (click)="onClick($event)">{{tab.name}}</li>
-    </ng-container>
+    @for (tab of tabs; track tab) {
+      <li [class.current]="tab.current" (click)="onClick($event)">{{tab.name}}</li>
+    }
   </ul>

   <!-- ngComponentOutlet で動的にコンポーネントを読み込む -->
```

以下､ポイントだけ抜き出します｡

```html:移行前のコード
-    <ng-container *ngFor="let tab of tabs">
-        <li [class.current]="tab.current" (click)="onClick($event)">{{tab.name}}</li>
-    </ng-container>
```

が

```html:移行後のコード
+    @for (tab of tabs; track tab) {
+      <li [class.current]="tab.current" (click)="onClick($event)">{{tab.name}}</li>
+    }
```

に書き換えられました｡
面白いのは `ng-container` が除去されていることですね｡

もうひとつ差分を見てみます｡

### 対象ソースコード

- [src/app/component/use-angular-material/autocomplete/highlight-firs-item/highlight-firs-item.component.html](https://github.com/ksh-fthr/angular-work/blob/develop/src/app/component/use-angular-material/autocomplete/highlight-firs-item/highlight-firs-item.component.html)

### 差分

```bash
% git diff src/app/component/use-angular-material/autocomplete/highlight-firs-item/highlight-firs-item.component.html
diff --git a/src/app/component/use-angular-material/autocomplete/highlight-firs-item/highlight-firs-item.component.html b/src/app/component/use-angular-material/autocomplete/highlight-firs-item/highlight-firs-item.component.html
index 5199bd5..1259022 100644
--- a/src/app/component/use-angular-material/autocomplete/highlight-firs-item/highlight-firs-item.component.html
+++ b/src/app/component/use-angular-material/autocomplete/highlight-firs-item/highlight-firs-item.component.html
@@ -6,15 +6,15 @@
     <!-- autoCompleteInput はフォーカスをあてるために使用する､本要素を指定するためのテンプレート変数 -->
     <!-- trigger は autocomplete パネルを開く処理: openPanel を実行するための MatAutocompleteTrigger オブジェクト -->
     <input type="text"
-          placeholder="Pick one"
-          aria-label="Number"
-          #autoCompleteInput
-          matInput
-          #trigger="matAutocompleteTrigger"
-          (focus)="openPanel($event, trigger)"
-          [formControl]="autocompleteControl"
-          [matAutocomplete]="auto"
-    >
+      placeholder="Pick one"
+      aria-label="Number"
+      #autoCompleteInput
+      matInput
+      #trigger="matAutocompleteTrigger"
+      (focus)="openPanel($event, trigger)"
+      [formControl]="autocompleteControl"
+      [matAutocomplete]="auto"
+      >
     <!-- mat-autocomplete についてのメモ -->
     <!-- * autoActiveFirstOption を設定することで autocomplete のリストの最初のアイテムにハイライトを当てることができる -->
     <!--   * これは [autoActiveFirstOption]="true" と書いても同様の効果が得られる -->
@@ -23,10 +23,12 @@
       autoActiveFirstOption
       #auto="matAutocomplete"
       (optionSelected)="onItemSelected($event)"
-    >
-      <mat-option *ngFor="let autocompleteItem of filteredAutocompleteItemList | async" [value]="autocompleteItem">
-        {{autocompleteItem}}
-      </mat-option>
+      >
+      @for (autocompleteItem of filteredAutocompleteItemList | async; track autocompleteItem) {
+        <mat-option [value]="autocompleteItem">
+          {{autocompleteItem}}
+        </mat-option>
+      }
     </mat-autocomplete>
   </mat-form-field>
 </form>
```

最初のブロックの差分はフォーマッタによる差分です｡お恥ずかしながらフォーマット不正があったのですが､これもマイグレーションコマンドで修正してくれてます｡
そしてこちらもポイントだけ抜き出します｡

```html:移行前のコード
-      <mat-option *ngFor="let autocompleteItem of filteredAutocompleteItemList | async" [value]="autocompleteItem">
-        {{autocompleteItem}}
-      </mat-option>
```

が

```html:移行後のコード
+      @for (autocompleteItem of filteredAutocompleteItemList | async; track autocompleteItem) {
+        <mat-option [value]="autocompleteItem">
+          {{autocompleteItem}}
+        </mat-option>
+      }
```

に書き換えてくれています｡ 先の例では `ng-container` が除去されてましたが､今回は `mat-option` を残したまま書き換えが行われてます｡
この辺､ものすごく賢いですね｡助かります｡

## `*ngIf` から `@if` に移行

### 対象ソースコード

- [src/app/component/validation/validation-verification/validation-verification.component.html](https://github.com/ksh-fthr/angular-work/blob/develop/src/app/component/validation/validation-verification/validation-verification.component.html)

### 差分

```bash
% git diff src/app/component/validation/validation-verification/validation-verification.component.html
diff --git a/src/app/component/validation/validation-verification/validation-verification.component.html b/src/app/component/validation/validation-verification/validation-verification.component.html
index aad39f9..f3c1a49 100644
--- a/src/app/component/validation/validation-verification/validation-verification.component.html
+++ b/src/app/component/validation/validation-verification/validation-verification.component.html
@@ -18,7 +18,7 @@
           maxlength="{{maxNetworkAddressLength}}"
           pattern="{{networkAddressPattern}}"
           (keyup)="onKeyUp('inputIPinfo', inputIPinfo.errors)"
-        >
+          >
       </div>
       <div class="input-network-address">
         <label class="label-common">サブネットマスク: </label>
@@ -33,7 +33,7 @@
           maxlength="{{maxNetworkAddressLength}}"
           pattern="{{networkAddressPattern}}"
           (keyup)="onKeyUp('inputSubnetMaskInfo', inputSubnetMaskInfo.errors)"
-        >
+          >
       </div>
     </div>
     <div class="validation-form-area">
@@ -49,7 +49,7 @@
           min="0"
           max="10"
           (keyup)="onKeyUp('inputNumberInfo', inputNumberInfo.errors)"
-        >
+          >
       </div>
     </div>
     <div class="validation-error-area">
@@ -57,14 +57,16 @@
         入力に誤りがある場合は下記にエラー内容が表示されます。
       </p>
       <div class="validation-error-information">
-        <div class="note" *ngIf="validationError">
-          <div [hidden]="!validationError?.required">※ 項目が未入力です</div>
-          <div [hidden]="!validationError?.minlength">※ 入力した内容が短すぎます</div>
-          <div [hidden]="!validationError?.maxlength">※ 入力した内容が長すぎます</div>
-          <div [hidden]="!validationError?.pattern">※ 入力した内容に誤りがあります</div>
-          <div [hidden]="!validationError?.min">※ 入力された数値が小さすぎます</div>
-          <div [hidden]="!validationError?.max">※ 入力された数値が大きすぎます</div>
-        </div>
+        @if (validationError) {
+          <div class="note">
+            <div [hidden]="!validationError?.required">※ 項目が未入力です</div>
+            <div [hidden]="!validationError?.minlength">※ 入力した内容が短すぎます</div>
+            <div [hidden]="!validationError?.maxlength">※ 入力した内容が長すぎます</div>
+            <div [hidden]="!validationError?.pattern">※ 入力した内容に誤りがあります</div>
+            <div [hidden]="!validationError?.min">※ 入力された数値が小さすぎます</div>
+            <div [hidden]="!validationError?.max">※ 入力された数値が大きすぎます</div>
+          </div>
+        }
       </div>
     </div>
   </form>
```

こちらも前半部分はフォーマッタによる差分です｡
後半に `*ngIf` -> `@if` への書き換えとして

```html:移行前のコード
-        <div class="note" *ngIf="validationError">
-          <div [hidden]="!validationError?.required">※ 項目が未入力です</div>
-          <div [hidden]="!validationError?.minlength">※ 入力した内容が短すぎます</div>
-          <div [hidden]="!validationError?.maxlength">※ 入力した内容が長すぎます</div>
-          <div [hidden]="!validationError?.pattern">※ 入力した内容に誤りがあります</div>
-          <div [hidden]="!validationError?.min">※ 入力された数値が小さすぎます</div>
-          <div [hidden]="!validationError?.max">※ 入力された数値が大きすぎます</div>
-        </div>
```

が

```html:移行後のコード
+        @if (validationError) {
+          <div class="note">
+            <div [hidden]="!validationError?.required">※ 項目が未入力です</div>
+            <div [hidden]="!validationError?.minlength">※ 入力した内容が短すぎます</div>
+            <div [hidden]="!validationError?.maxlength">※ 入力した内容が長すぎます</div>
+            <div [hidden]="!validationError?.pattern">※ 入力した内容に誤りがあります</div>
+            <div [hidden]="!validationError?.min">※ 入力された数値が小さすぎます</div>
+            <div [hidden]="!validationError?.max">※ 入力された数値が大きすぎます</div>
+          </div>
+        }
```

に変更されています｡

# まとめにかえて

[マイグレーションコマンド](#とはいってもコマンド一発たたくだけ) による移行を行っただけですが､特にトラブルもなく移行できました｡
移行後のコードに対して `npm run start` を実行した際もエラーなく Angular アプリが起動してます｡

```bash
% npm run start

> angular-app@0.0.0 start
> ng serve --proxy-config proxy.conf.json

✔ Browser application bundle generation complete.

Initial Chunk Files   | Names         |  Raw Size
vendor.js             | vendor        |   4.96 MB |
main.js               | main          |   1.34 MB |
polyfills.js          | polyfills     | 405.92 kB |
styles.css, styles.js | styles        | 323.52 kB |
runtime.js            | runtime       |   6.52 kB |

                      | Initial Total |   7.03 MB

Build at: 2024-01-13T09:06:55.280Z - Hash: 4e74de41bdd101a3 - Time: 10534ms

** Angular Live Development Server is listening on localhost:4200, open your browser on http://localhost:4200/ **
```

新しい制御構文は [チュートリアル](https://angular.dev/tutorials/learn-angular) でも扱われており､今後はこちらが主流になるようです｡
またこの新しい制御構文を使うことで次のメリットがあるとのこと｡

- メリット
  - パフォーマンスの向上
    - [こちらのスライド](https://docs.google.com/presentation/d/e/2PACX-1vSdSos0klysPqneN59S2m54Mw75sWAx8UP7Ju5kk-yut5ixYkOTOVWuoa8ix6IFwn5dTjJWE2_r5-HD/pub?resourcekey=0-fQRaaQ2q31dV7tJj8oDzyg&slide=id.g261707052bb_0_114) によると､ `@for` を使うことで最大 `90%` の高速化が見込める
  - バンドルサイズの削減
    - `Built-in(組み込み)` ということで､ 以前の制御構文で使用していた [`CommonModule`](https://angular.jp/api/common/CommonModule)( `*ngIf` や `*ngFor`, `*ngSwitch` を使う際に import していたモジュール ) が不要となる
    - これによりバンドルサイズの削減も見込める
    - ↑ は [こちらの放送](https://www.youtube.com/watch?v=s1rlAeSeO8c) の `11:06` あたりからのやり取りで触れてます
    - とはいえ､ `CommonModule` は制御構文以外の Directive を使う際に必要なモジュールです｡手動で削除する際はご注意ください

というわけで､新しい制御構文に慣れていこうと思います｡

# 補足

今回扱ったリポジトリではモジュール分割していたためか マイグレーションコマンドで `CommonModule` が削除されませんでした｡
従いまして `CommonModule` を必要としているモジュールを除き､ `CommonModule` の import は手動で削除しています｡

# 参考

- [公式ドキュメント](https://angular.dev/overview)
  - [Built-in control flow](https://angular.dev/guide/templates/control-flow)
- [チュートリアル](https://angular.dev/tutorials/learn-angular)
  - [Control Flow in Components - @if](https://angular.dev/tutorials/learn-angular/control-flow-if)
  - [Control Flow in Components - @for](https://angular.dev/tutorials/learn-angular/control-flow-for)
- [Angular v17 で新しい制御フロー構文がやってきた！マイグレーションコマンドを試してみたよ](https://qiita.com/kozy4324/items/356fd8e2429ae5142641)
- [Angular Control Flow: the Complete Guide](https://www.codemotion.com/magazine/frontend/angular-control-flow-the-complete-guide/)
- [Angularの新しいテンプレート構文 if/for/switch ブロックを解説！【OnAir切り抜き】](https://www.youtube.com/watch?v=s1rlAeSeO8c)
  - [スライド](https://docs.google.com/presentation/d/e/2PACX-1vSdSos0klysPqneN59S2m54Mw75sWAx8UP7Ju5kk-yut5ixYkOTOVWuoa8ix6IFwn5dTjJWE2_r5-HD/pub?resourcekey=0-fQRaaQ2q31dV7tJj8oDzyg&slide=id.g260298bad6d_0_77)
- [Angularの新しいチュートリアルをやってみよう！【ng-japan OnAir #74】](https://www.youtube.com/watch?v=cXa0NvAcSV4)
