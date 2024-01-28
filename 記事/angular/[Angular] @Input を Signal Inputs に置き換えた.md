# はじめに

Angular v17.1 で **Signal Inputs** がリリースされました。
本記事では従来の `@Input` で記述された構文を Signal Inputs で書き換えてみます。

# Signal Inputs について

**Signal Inputs** については [こちら](https://zenn.dev/lacolaco/articles/angular-signal-inputs/) をご参照ください。
ここで見出しを拝借しますと

- [`ngOnChanges` が不要になる](https://zenn.dev/lacolaco/articles/angular-signal-inputs#ngonchanges%E3%81%8C%E4%B8%8D%E8%A6%81%E3%81%AB%E3%81%AA%E3%82%8B)
- [TypeScriptのstrictPropertyInitializationを常に有効化できる](https://zenn.dev/lacolaco/articles/angular-signal-inputs#%E3%82%A4%E3%83%B3%E3%83%97%E3%83%83%E3%83%88%E3%81%8C%E3%82%A4%E3%83%9F%E3%83%A5%E3%83%BC%E3%82%BF%E3%83%96%E3%83%AB%E3%81%AB%E3%81%AA%E3%82%8B)
- [インプットがイミュータブルになる](https://zenn.dev/lacolaco/articles/angular-signal-inputs#%E3%82%A4%E3%83%B3%E3%83%97%E3%83%83%E3%83%88%E3%81%8C%E3%82%A4%E3%83%9F%E3%83%A5%E3%83%BC%E3%82%BF%E3%83%96%E3%83%AB%E3%81%AB%E3%81%AA%E3%82%8B)

といった内容で Signal Inputs に変えることによるメリットを簡潔にわかりやすく説明してくださっています。

# 環境

今回の記事の内容は次の環境で実施したものです。

| 環境                                                        | バージョン | 備考                     |
| ----------------------------------------------------------- | ---------- | ------------------------ |
| [Angular CLI](https://cli.angular.io/)                      | v17.1.0    | `ng version` で確認      |
| [Angular](https://angular.io/)                              | v17.1.0    | 同上                     |
| [TypeScript](https://www.typescriptlang.org/)               | v5.3.3     | 同上                     |
| [zone.js](https://www.npmjs.com/package/zone.js)            | v0.14.3    | 同上                     |
| [Node.js](https://nodejs.org/ja/)                           | v18.19.0   | 同上                     |
| [npm](https://www.npmjs.com/)                               | v10.2.3    | 同上                     |

# 今回扱うソースコード

対象のソースコードは私の学習リポジトリである [こちら](https://github.com/ksh-fthr/angular-work) になります｡

# 作業は手動で行う

`@Input` から Signal Inputs へのマイグレーションコマンドは用意されていないようです。
従いまして今回は手動で移行を行いました。

# 差分をみてみる

## 単純な置き換え

### 対象ソースコード

- [src/app/component/parent-child/child](https://github.com/ksh-fthr/angular-work/tree/develop/src/app/component/parent-child/child)

### 差分(`child.component.ts`)

```typescript:差分
% git diff src/app/component/parent-child/child/child.component.ts
diff --git a/src/app/component/parent-child/child/child.component.ts b/src/app/component/parent-child/child/child.component.ts
index d97c8bc..7394a23 100644
--- a/src/app/component/parent-child/child/child.component.ts
+++ b/src/app/component/parent-child/child/child.component.ts
@@ -1,7 +1,4 @@
-import { Component, OnInit } from '@angular/core';
-
-// @Input デコレータを使用するための import
-import { Input } from '@angular/core';
+import { Component, OnInit, input } from '@angular/core';
 
 // @Output デコレータを使用するための import
 import { Output, EventEmitter } from '@angular/core';
@@ -14,10 +11,8 @@ import { Output, EventEmitter } from '@angular/core';
 export class ChildComponent implements OnInit {
   /**
    * 親コンポーネントから受け取るデータ(文字列)をセットするパラメータ
-   *
-   * @type {string}
    */
-  @Input() dataFromParent = '';
+  dataFromParent = input<string>('');
 
   /**
    * 親コンポーネントに対してイベントを発火するためのパラメータ
```

以下､ポイントを抜き出します｡( import 文やコメントの差分は除きます )

```typescript:移行前のコード
-  @Input() dataFromParent = '';
```

を

```typescript:移行後のコード
+  dataFromParent = input<string>('');
```

に書き換えています。
これにより `dataFromParent` の型も `string` から `InputSignal<string, string>` に変わり、`computed` や `effect` によるリアクティブな処理ができるようになる、とのことです。

( 移行前後のキャプチャ )

![スクリーンショット 2024-01-28 16.39.31.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/6492bd81-7be7-5319-f092-5798279f7a4e.png)

![スクリーンショット 2024-01-28 17.08.44.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/4037246f-95e7-89d4-2193-ec5ded09feef.png)

### 補足-`required` の指定

なお親コンポーネントから値を必ず受け取りたい場合は `required` を指定します。
上記のコードを例に取ると

```typescript:requiredを使う例
  dataFromParent = input.required<string>();
```

となります。
このとき引数の値に `''` を指定すると次のエラーになります。

```text
Argument needs to be an object literal that is statically analyzable.(-991010)
```

( こちらはキャプチャ )
![スクリーンショット 2024-01-28 16.30.06.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/aa7d3273-1cbb-b511-4773-2cf4acd1cf19.png)

## 差分(`child.component.html`)

```html
% git diff src/app/component/parent-child/child/child.component.html
diff --git a/src/app/component/parent-child/child/child.component.html b/src/app/component/parent-child/child/child.component.html
index 997180c..2cdd85b 100644
--- a/src/app/component/parent-child/child/child.component.html
+++ b/src/app/component/parent-child/child/child.component.html
@@ -1,7 +1,7 @@
 <div id="child-wrap">
   <!--
     親コンポーネントから受け取ったデータを表示する
-    @Inputデコレータで修飾されたパラメータに対して「親コンポーネント側のテンプレートでセット済み」なので
+    Signal Inputs であるパラメータに対して「親コンポーネント側のテンプレートでセット済み」なので
     ここでは単純にパラメータを単方向データバインドで扱うだけで良い
   -->
   <div id="parent-to-child">
@@ -9,7 +9,7 @@
       <strong>子コンポーネント</strong>
     </p>
     <p id="sent-data">
-      親コンポーネントから渡された文字列は 「<strong>{{dataFromParent}}</strong>」です。
+      親コンポーネントから渡された文字列は 「<strong>{{dataFromParent()}}</strong>」です。
     </p>
   </div>
```

以下､ポイントを抜き出します｡( コメントの差分は除きます )

```html:移行前のコード
-      親コンポーネントから渡された文字列は 「<strong>{{dataFromParent}}</strong>」です。
```

を

```html:移行後のコード
+      親コンポーネントから渡された文字列は 「<strong>{{dataFromParent()}}</strong>」です。
```

に書き換えています。
`dataFromParent` の呼び出しが **`()` をつけた関数呼び出し** になっていることに注目です。
`()` をつけないと次の警告とエラーがでるのでご注意ください。( 私の環境で出力されたログをそのまま出しています )

```text:警告とエラー
Warning: src/app/component/parent-child/child/child.component.html:12:37 - warning NG8109: dataFromParent is a function and should be invoked: dataFromParent()

12       親コンポーネントから渡された文字列は 「<strong>{{dataFromParent}}</strong>」です。
                                       ~~~~~~~~~~~~~~

  src/app/component/parent-child/child/child.component.ts:8:16
    8   templateUrl: './child.component.html',
                     ~~~~~~~~~~~~~~~~~~~~~~~~
    Error occurs in the template of component ChildComponent.
```

次は Directive の例です。

## Directiv での置き換え

### 対象ソースコード

- [src/app/directive/attribute/template/template.directive.ts](https://github.com/ksh-fthr/angular-work/blob/develop/src/app/directive/attribute/template/template.directive.ts)

### 差分

```typescript:差分
% git diff
diff --git a/src/app/directive/attribute/template/template.directive.ts b/src/app/directive/attribute/template/template.directive.ts
index e649704..0475414 100644
--- a/src/app/directive/attribute/template/template.directive.ts
+++ b/src/app/directive/attribute/template/template.directive.ts
@@ -1,4 +1,4 @@
-import { Directive, ElementRef, Input, OnInit } from '@angular/core';
+import { Directive, ElementRef, OnInit, input } from '@angular/core';
 
 @Directive({
   selector: '[appTemplate]',
@@ -6,22 +6,16 @@ import { Directive, ElementRef, Input, OnInit } from '@angular/core';
 export class TemplateDirective implements OnInit {
   /**
    * 親コンポーネントから受け取るデータ-1
-   *
-   * @type {string}
    */
-  @Input() public greet = '';
+  public greet = input<string>('');
 
   /**
    * 親コンポーネントから受け取るデータ-2
-   *
-   * @type {string}
    */
-  @Input() public name = '';
+  public name = input<string>('');
 
   /**
    * コンストラクタ
-   *
-   * @param elementRef このディレクティブがセットされたDOMへの参照
    */
   constructor(private elementRef: ElementRef) {}
 
@@ -32,6 +26,6 @@ export class TemplateDirective implements OnInit {
    */
   ngOnInit(): void {
     const element = this.elementRef.nativeElement;
-    element.innerHTML = this.greet + ' ' + this.name + '.';
+    element.innerHTML = this.greet() + ' ' + this.name() + '.';
   }
 }
```

この差分のうち

```typescript:差分
-  @Input() public greet = '';
+  public greet = input<string>('');
```

や

```typescript:差分
-  @Input() public name = '';
+  public name = input<string>('');
```

は [前項](#単純な置き換え) で挙げたものと同じです。
Directive におけるポイントは次の差分です。

```typescript:差分
-    element.innerHTML = this.greet + ' ' + this.name + '.';
+    element.innerHTML = this.greet() + ' ' + this.name() + '.';
```

Signal Inputs に置き換えたことにより、`greet` や `name` を関数呼び出しに書き換えました。
移行前のコードのままですと、次のエラーが出ます。

```text
function inputValueFn() { // Record that someone looked at this signal. (0,_angular_core_primitives_signals__WEBPACK_IMPORTED_MODULE_0__.producerAccessed)(node); if (node.value === REQUIRED_UNSET_VALUE) { throw new RuntimeError(-950 /* RuntimeErrorCode.REQUIRED_INPUT_NO_VALUE */, ngDevMode && 'Input is required but no value is available yet.'); } return node.value; } function inputValueFn() { // Record that someone looked at this signal. (0,_angular_core_primitives_signals__WEBPACK_IMPORTED_MODULE_0__.producerAccessed)(node); if (node.value === REQUIRED_UNSET_VALUE) { throw new RuntimeError(-950 /* RuntimeErrorCode.REQUIRED_INPUT_NO_VALUE */, ngDevMode && 'Input is required but no value is available yet.'); } return node.value; }.
```

[前項](#単純な置き換え) のテンプレートの書き換えでも類似のエラーが出ていますが、これは型が `Signal` に変わったことによるものです。
Signal Inputs に置き換えた場合、その変数を参照する場合は `()` を付与した関数呼び出しに注意しましょう。

これは [こちらのガイド](https://blog.angular-university.io/angular-signal-inputs/) で説明されています。

## ngOnChagnes を含む置き換え

### 対象ソースコード

- [src/app/component/lifecycle/on-change/on-change-verification](https://github.com/ksh-fthr/angular-work/tree/develop/src/app/component/lifecycle/on-change/on-change-verification/)

### 差分(`on-change-verification.component.ts`)

(html テンプレートの差分はこれまでに挙げたものと同じなので割愛します)

```typescript:差分
% git diff src/app/component/lifecycle/on-change/on-change-verification/on-change-verification.component.ts
diff --git a/src/app/component/lifecycle/on-change/on-change-verification/on-change-verification.component.ts b/src/app/component/lifecycle/on-change/on-change-verification/on-change-verification.component.ts
index 953c69b..13be5f9 100644
--- a/src/app/component/lifecycle/on-change/on-change-verification/on-change-verification.component.ts
+++ b/src/app/component/lifecycle/on-change/on-change-verification/on-change-verification.component.ts
@@ -1,4 +1,4 @@
-import { Component, ElementRef, Input, OnChanges, OnInit, SimpleChanges } from '@angular/core';
+import { AfterViewInit, Component, ElementRef, SimpleChanges, effect, input } from '@angular/core';
 import { Logging } from 'src/app/utils/logging';

 @Component({
@@ -6,8 +6,8 @@ import { Logging } from 'src/app/utils/logging';
   templateUrl: './on-change-verification.component.html',
   styleUrls: ['../../../../style/common.css', './on-change-verification.component.css'],
 })
-export class OnChangeVerificationComponent implements OnInit, OnChanges {
-  @Input() ngOnChangesValue = '';
+export class OnChangeVerificationComponent implements AfterViewInit {
+  signalInputValue = input<string>('');

   /**
    * ログ出力を行うテキストエリアの HTML エレメント
@@ -16,56 +16,32 @@ export class OnChangeVerificationComponent implements OnInit, OnChanges {

   constructor(private element: ElementRef) {
     // コンストラクタは画面描画の前に実行されるので画面上の要素を取得することができない
-    // よって､ここではコンソールログに出力するだけになる
     console.log('[constructor] execute');
-  }

-  ngOnInit(): void {
-    // こちらはコンソールログと画面上の両方に出力する
-    const message = '[ngOnInit] fired';
-    console.log(message);
-    Logging.info(this.textAreaElement, message);
+    effect(() => {
+      // コンソールログと画面上の両方に出力する
+      let message = '[effect] fired';
+      console.log(message);
+      Logging.info(this.textAreaElement, message);
+
+      // effect を使って変更前の値と変更後の値、そして変更されているかをログ出力する
+      const messageJson = {
+        currentValue: this.signalInputValue(),
+      };
+      message = `[ngOnChanges] ${JSON.stringify(messageJson)}`;
+      console.log(message);
+      Logging.info(this.textAreaElement, message);
+    });
   }

-  ngOnChanges(changes: SimpleChanges) {
-    // ngOnchanges はライフサイクルの順序上､ ( @Input で修飾されたデータが存在する場合は )ngOninit の先に実行される.
-    // なので､ちょっと気持ち悪いがここで
-    //
-    // - 画面上のログ出力先となる HTML 要素の取得
-    //
-    // を行う
+  ngAfterViewInit(): void {
+    // 画面上のログ出力先となる HTML 要素の取得を行う
     // ( ngOninit で行うと HTML 要素が取得される前にログ出力を行おうとするので例外が発生する )
     this.textAreaElement = this.element.nativeElement.querySelector('.log-text-area');

-    // こちらもコンソールログと画面上の両方に出力する
-    let message = '[ngOnChanges] fired';
+    // コンソールログと画面上の両方に出力する
+    const message = '[ngOnInit] fired';
     console.log(message);
     Logging.info(this.textAreaElement, message);
-
-    // SimpleChanges を使って変更前の値と変更後の値、そして変更されているかをログ出力する
-    for (const prop in changes) {
-      if (!changes.hasOwnProperty(prop)) {
-        continue;
-      }
-
-      const change = changes[prop];
-      const messageJson = {
-        property: prop,
-        isFirstChange: change.firstChange,
-        previousValue: ((previousValue): string => {
-          // 初回実行時、`previousValue` は「前回の値が存在しない」ので `undefined` が設定される
-          // だが JSON.stringfy で文字列に変換する際、 `undefined` は文字列化の対象から除外されてしまうので
-          // 文字列としての `undefined` を返却したい
-          if (previousValue === undefined) {
-            return 'undefined';
-          }
-          return previousValue;
-        })(change.previousValue),
-        currentValue: change.currentValue,
-      };
-      message = `[ngOnChanges] ${JSON.stringify(messageJson)}`;
-      console.log(message);
-      Logging.info(this.textAreaElement, message);
-    }
   }
 }
```

この差分ではわけがわからないので、注目したい部分について移行前後のコードを示します。
※ Signal Ipnuts への置き換え部分については、前項までに挙げたものと同じなので割愛します
※ またコード中のコメントはここでは冗長なので削除してます

```typescript:移行前
  constructor(private element: ElementRef) {
    console.log('[constructor] execute');
  }

  ngOnInit(): void {
    const message = '[ngOnInit] fired';
    console.log(message);
    Logging.info(this.textAreaElement, message);
  }

  ngOnChanges(changes: SimpleChanges) {
    this.textAreaElement = this.element.nativeElement.querySelector('.log-text-area');

    let message = '[ngOnChanges] fired';
    console.log(message);
    Logging.info(this.textAreaElement, message);

    for (const prop in changes) {
      if (!changes.hasOwnProperty(prop)) {
        continue;
      }

      const change = changes[prop];
      const messageJson = {
        property: prop,
        isFirstChange: change.firstChange,
        previousValue: ((previousValue): string => {
          if (previousValue === undefined) {
            return 'undefined';
          }
          return previousValue;
        })(change.previousValue),
        currentValue: change.currentValue,
      };
      message = `[ngOnChanges] ${JSON.stringify(messageJson)}`;
      console.log(message);
      Logging.info(this.textAreaElement, message);
    }
  }
```

- 移行前のコードのポイント
  - `constructor` と `ngOnInit` はログ出力だけで他には何もしていない
  - `ngOnChanges` では `SimpleChanges` オブジェクトを参照して以下を行っている
    - `firstChange` を参照して最初の変更か否かをチェック
    - `previousValue` を参照して更新前の値を取得
    - `currentValue` を参照して更新後( 現在 )の値を取得

これが `effect` に置き換えることで次のようになります。

```typescript:移行後
  constructor(private element: ElementRef) {
    console.log('[constructor] execute');

    effect(() => {
      let message = '[effect] fired';
      console.log(message);
      Logging.info(this.textAreaElement, message);

      const messageJson = {
        currentValue: this.signalInputValue(),
      };
      message = `[ngOnChanges] ${JSON.stringify(messageJson)}`;
      console.log(message);
      Logging.info(this.textAreaElement, message);
    });
  }

  ngAfterViewInit(): void {
    this.textAreaElement = this.element.nativeElement.querySelector('.log-text-area');

    const message = '[ngOnInit] fired';
    console.log(message);
    Logging.info(this.textAreaElement, message);
  }
```

- 移行後のコードのポイント
  - **`effect` を実装**
    - `ngOnChanges` で行っていた値が更新されたときのハンドラを `constructor` 内の `effect` に移した
    - `SimpleChanges` オブジェクトは参照しなくなったので、それに関する処理を削除
    - `firstChange` の参照を削除
    - `previousValue` の参照を削除
    - `currentValue` の参照を削除
  - 副次的な変更として
    - `ngOnChanges` で取得していた HTMLElement の参照が取得できなくなったので、 `ngAfterViewInit` を用意してそちらに移した

`effect` にすることでコードはかなりスッキリしました。`SimpleChanges` オブジェクトから対象となる値を抜き出すことなく、直接 Signal Inputs で定義されたインプット変数を参照できるのは嬉しいです。
なお `effect` では古い値を提供してくれないようです。

- 参考: [Possible to access the old value in Angular 16's effect() function (similar to Vue.js watch)?](https://stackoverflow.com/questions/76200595/possible-to-access-the-old-value-in-angular-16s-effect-function-similar-to-v)

反面、`SimpleChanges` にあった `firstChange` や `previousValue` を参照した処理が書けなくなる、もしくは別の方法を模索しなければならなくなりました。
その点において若干の不便さはあると感じました。

### 補足

`effect` を `ngOnInit` とか `ngAfterViewInit` で実装すると次のエラーが出ます。

```text
ERROR Error: NG0203: effect() can only be used within an injection context such as a constructor, a factory function, a field initializer, or a function used with `runInInjectionContext`. Find more at https://angular.io/errors/NG0203
```

# まとめにかえて

Angular v16 で搭載された Signal がベースになっているので、そちらをまず理解しておくことが大事ですね。
[こちらのようなスタートガイド](https://codelabs.developers.google.com/angular-signals?hl=ja#0) もありますので活用していきます。

あとは記事中でも触れていますが、 `ngOnChanges` を使わないことでコードがかなりスッキリと、かつ見やすくなります。これは嬉しいですね。

また Signal Inputs に置き換えることで、それらのインプットプロパティは **Signal型のオブジェクトで初期化される** のも嬉しいです。
本記事では試していませんが、参考記事の [こちら](https://zenn.dev/lacolaco/articles/angular-signal-inputs) によりますと、これによって [`strictPropertyInitialization`](https://typescriptbook.jp/reference/tsconfig/strictpropertyinitialization) を常に有効化することが可能になるそうです。
そうすると変数宣言時の初期化が必須となり、より安全にプログラムに臨めるということで、これも嬉しいメリットです。

# 課題

次が今回の記事を書く中で分からなかったので今後の課題として挙げます。

- `@Input` から Signal Inputs への置き換えのように `@Output` の置き換えはあるのか、とか
- `ngOnChanges` が `effect` に置き換わったように `ngDocheck` についても置き換えはあるのか、とか

# 参考

- Angualr Signals
  - [Angular Signals: Complete Guide](https://blog.angular-university.io/angular-signals/)
  - [Angular Signalsとコンポーネント間通信](https://zenn.dev/lacolaco/articles/angular-signals-and-component-communication)
- Angular Signal Inputs
  - [Angular Signal Inputs: Complete Guide](https://blog.angular-university.io/angular-signal-inputs/)
  - [Angular Signal Inputs](https://angularexperts.io/blog/angular-signal-inputs)
  - [Angular: Signal Inputsで何が変わるのか](https://zenn.dev/lacolaco/articles/angular-signal-inputs)
- Monthly Angular 1月号
  - [Angularの最新情報がわかる！Monthly Angular 1月号 【ng-japan OnAir #75】](https://github.com/ng-japan/onair/discussions/86)
  - [#75 Angularの最新情報がわかる！Monthly Angular 1月号 #86](https://github.com/ng-japan/onair/discussions/86)
- [What's new in Angular 17.1?](https://blog.ninja-squad.com/2024/01/17/what-is-new-angular-17.1/)
- [Possible to access the old value in Angular 16's effect() function (similar to Vue.js watch)?](https://stackoverflow.com/questions/76200595/possible-to-access-the-old-value-in-angular-16s-effect-function-similar-to-v)
