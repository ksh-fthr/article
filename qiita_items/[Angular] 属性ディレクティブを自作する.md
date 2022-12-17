# はじめに
[本家](https://angular-ja.firebaseapp.com/guide/attribute-directives.en) によると､Angular には3つのディレクティブがある｡
以下､抜粋｡

> Directives overview
>
> There are three kinds of directives in Angular:
>
>     1. Components—directives with a template.
>     2. Structural directives—change the DOM layout by adding and removing DOM elements.
>     3. Attribute directives—change the appearance or behavior of an element, component, or another directive.
>
> Components are the most common of the three directives. You saw a component for the first time in the QuickStart guide.
>
> Structural Directives change the structure of the view. Two examples are NgFor and NgIf. Learn about them in the Structural > Directives guide.
>
> Attribute directives are used as attributes of elements. The built-in NgStyle directive in the Template Syntax guide, for > example, can change several element styles at the same time.

3つのディレクティブとは｢**コンポーネント**｣｢**構造ディレクティブ**｣｢**属性ディレクティブ**｣を指し､コンポーネントもディレクティブの一つであることを上記でうたっている｡
で､残る2つの｢構造ディレクティブ｣と｢属性ディレクティブ｣だが､これらは

* 構造ディレクティブ
  * ビューの構造を変更する
  * ```ngIf``` や ```ngFor``` のように DOM の追加や削除を行うことで DOM のレイアウトを変更する
* 属性ディレクティブ
  * 要素の属性として扱われる
  * ```NgStyle``` のように DOM 要素の変更を行うことで見た目などを変更する

ものとなる｡

本記事では簡単な例を示し､｢**属性ディレクティブの自作**｣についてみていく｡

# 更新情報

## 2021/01/09
- 記事内で扱ったコードを Angular `v11.0.5` で確認しました

# 作業環境

| 環境                                          | バージョン          | 備考               |
| --------------------------------------------- | ------------------- | ------------------ |
| [Angular CLI](https://cli.angular.io/)        | ~~v6.0.0~~ v11.0.5  | `$ ng --version`   |
| [Angular](https://angular.io/)                | ~~v6.0.0~~ v11.0.5  | 同上               |
| [TypeScript](https://www.typescriptlang.org/) | v4.0.2              | 同上               |
| [Node.js](https://nodejs.org/ja/)             | ~~v9.2.1~~ v12.18.3 | `$ node --version` |
| [npm](https://www.npmjs.com/)                 | ~~v6.1.0~~ v6.14.6  | `$ npm --version`  |


<details>
<div>
<summary>ng version の結果</summary>

```bash
$ ng version

     _                      _                 ____ _     ___
    / \   _ __   __ _ _   _| | __ _ _ __     / ___| |   |_ _|
   / △ \ | '_ \ / _` | | | | |/ _` | '__|   | |   | |    | |
  / ___ \| | | | (_| | |_| | | (_| | |      | |___| |___ | |
 /_/   \_\_| |_|\__, |\__,_|_|\__,_|_|       \____|_____|___|
                |___/
    

Angular CLI: 11.0.5
Node: 12.18.3
OS: darwin x64

Angular: 11.0.5
... animations, cli, common, compiler, compiler-cli, core, forms
... platform-browser, platform-browser-dynamic, router
Ivy Workspace: Yes

Package                         Version
---------------------------------------------------------
@angular-devkit/architect       0.1100.5
@angular-devkit/build-angular   0.1100.5
@angular-devkit/core            11.0.5
@angular-devkit/schematics      11.0.5
@schematics/angular             11.0.5
@schematics/update              0.1100.5
rxjs                            6.6.0
typescript                      4.0.2
```

</div>
</details>

# ディレクティブの生成

Angular CLI を使用してディレクティブを生成する｡

```bash:ディレクティブを生成するコマンド
$ ng g directive directive/template
  create src/app/directive/template.directive.spec.ts
  create src/app/directive/template.directive.ts
  update src/app/app.module.ts
```

# 本記事で扱う構成

上記で生成したディレクティブを含め､本記事で扱う構成を示す｡

```bash:構成
src/
  `-app/
    `-component/
    |  `-use-directive/
    |   `- use-directive.component.css
    |   `- use-directive.component.html
    |   `- use-directive.component.ts
    `-directive/
    |  `- template.directive.ts
    `- app.component.css
    `- app.component.html
    `- app.component.ts
    `- app.module.ts
```

今回の構成の内容は次のとおり｡

* use-directive.component
  * 自作ディレクティブである template.directive の利用方法を確認するためのコンポーネント
  * テンプレート HTML の中で template.directive を扱う
* template.directive
  * 自作ディレクティブ
  * use-directive.component から利用され､要素の属性として扱われる
* app.component
  * ルートコンポーネント
  * use-directive.component を子コンポーネントとして扱う
* app.module
  * ルートモジュール
  * 本構成であつかうコンポーネントやモジュールを管理する

では実際にディレクティブの中身を実装し､その扱い方についてみていく｡


# 属性ディレクティブの実装

```typescript:template.directive.ts
import { Directive, ElementRef, Input, OnInit } from '@angular/core';

@Directive({
  selector: '[appTemplate]'
})
export class TemplateDirective implements OnInit {

  /**
   * 親コンポーネントから受け取るデータ-1
   *
   * @type {string}
   * @memberof TemplateDirective
   */
  @Input() public greet: string = '';

  /**
   * 親コンポーネントから受け取るデータ-2
   *
   * @type {string}
   * @memberof TemplateDirective
   */
  @Input() public name: string = '';

  /**
   * コンストラクタ
   *
   * @param {ElementRef} elementRef このディレクティブがセットされたDOMへの参照
   * @memberof TemplateDirective
   */
  constructor(
    private elementRef: ElementRef
  ) {
  }

  /**
   * 初期処理
   * 本ディレクティブではこのタイミングで親コンポーネントから受け取ったデータを
   * DOMの innerHTML にセットする
   *
   * @memberof TemplateDirective
   */
  public ngOnInit() {
    const element = this.elementRef.nativeElement;
    element.innerHTML = this.greet + ' ' + this.name + '.';
  }
}
```

以下､ポイントを順にみていく｡

## Directive デコレーターでディレクティブ情報を宣言する

ディレクティブを作成するには `Directive` デコレータの宣言が必要である｡
そのため､まずは次のように `Directive` モジュールを ```import``` しておく｡
( 実際は先に示したコマンドでディレクティブを生成した際に自動で ```import``` されている )

```typescript:Directiveデコレータのimport
import { Directive, ElementRef, Input, OnInit } from '@angular/core';
```

そして宣言は次のコードで行う｡

```typescript:Directiveデコレータでディレクティブ情報を宣言する
@Directive({
  selector: '[appTemplate]'
})
```

上記コードは

* このディレクティブが `selector` パラメータに指定された属性として扱われる

という意味で、これを利用する側から見ると

* `selector` パラメータに指定された値を要素の属性にセットすることで､このディレクティブを適用できる

という意味となる｡

## @Input デコレータで修飾したプロパティで利用する側からデータを受け取る

ディレクティブは｢パラメータつきのディレクティブ｣として定義することができる｡
パラメータとして属性値を受け取る方法は､ ```@Input デコレータ``` を利用すれば良い｡
実際のコードは次のとおり｡

```typescript:@Inputデコレータでプロパティをを修飾
@Input() public greet: string;
@Input() public name: string;
```

```@Input デコレータ``` については [こちら](https://qiita.com/ksh-fthr/items/db6a48d072d5e9a33f0b#input-%E3%83%87%E3%82%B3%E3%83%AC%E3%83%BC%E3%82%BF) を参照いただきたい｡

## ElementRef で要素オブジェクトを取得して利用する

要素オブジェクトを取得するためのモジュールとして ```ElementRef``` がある｡
これを利用することで｢このディレクティブがセットされた DOM 要素を取得できる｣ので､属性ディレクティブとして要素の変更を容易に実現することができる｡

実際にコードをみていくと､次のコードで ```ElementRef``` の ```import``` を行う｡

```typescript:ElementRefのimport
import { Directive, ElementRef, Input, OnInit } from '@angular/core';
```

そしてコンストラクタで DI することで､ ```ElementRef``` の利用できるよう有効化する｡

```typescript:ElementRefをDIする
  constructor(
    private elementRef: ElementRef
  ) {
  }
```

最後に ```nativeElement``` で DOM 要素を取得し､要素の変更を行う｡

```typescript:DOMの取得と要素の変更
public ngOnInit() {
  const element = this.elementRef.nativeElement;
  element.innerHTML = this.greet + ' ' + this.name + '.';
}
```

上記コードで示すとおり､ ```nativeElement``` で取得できるよう要素は通常の DOM 要素そのものなので､```innerHTML```等で要素へのアクセスと変更が簡単に行える｡


# ディレクティブの登録

作成した属性ディレクティブを利用するために app.module.ts に登録する｡
実際には前掲のコマンドでディレクティブを生成した際に自動で登録されているので､その内容の確認となる｡

```typescript:app.module.ts
// 前略...
import { TemplateDirective } from './directive/template.directive';
// 中略...
@NgModule({
  declarations: [
    // 中略...
    TemplateDirective,
  ],
  // 中略...
})
export class AppModule { }
```

このとおり､生成されたディレクティブを ```import``` し､ ```declarations``` にセットすることで､ディレクティブが登録されたことになる｡
これでアプリ全体でディレクティブの利用が可能となる｡


# コンポーネントからディレクティブを利用する

ディレクティブはコンポーネントのテンプレートで要素の属性に指定してやることで利用できる｡
コードで示すと次のとおり。

```html:use-directive.component.html
<!-- ディレクティブを指定してパラメータをセットする例 -->
<div appTemplate [greet]="'Hello'" [name]="'Angular'"></div>
```

`div` 要素の属性として作成したディレクティブである `appTemplate` を指定していて、これだけで **コンポーネントからディレクティブを利用する** コードとなる｡

さらに今回の例では､ `template.directive.ts` が外部から属性値を受け取るパラメータとして `@Input` デコレータで修飾したプロパティを 2つ用意している。
この `@Input` で修飾されたプロパティを利用するにあたり、コンポーネントでは `template.directive.ts` を呼び出す際に

```html:パラメータの指定
[greet]="'Hello'" [name]="'Angular'"
```

とプロパティを 2 つセットしている｡

この例では ```greet``` と ```name``` には文字列を直接セットしているが､プロパティバインディングを使ってコンポーネントで定義した変数をセットしてもよい｡

で､このコードの実行結果は次のとおり｡

# 実行結果

![attribute_directive_result.png](https://qiita-image-store.s3.amazonaws.com/0/193342/fe105cd4-d5f4-a0db-d5b2-0c8982d83c12.png)

```use-directive.component.html``` で ```template.directive.ts``` を呼び出す際にセットした値である ｢Hello｣ と ｢Angular｣ が ```template.directive.ts``` で DOM 要素の変更を経て表示されるのが確認できた｡


# ソースコード

今回の記事で動作確認に使用したコードは [ここ](https://github.com/ksh-fthr/angular-work/tree/feat_directive) にアップしたのでご参考までに。。。


# 参考

* [本家の Attribute Directives](https://angular.jp/guide/attribute-directives.en)

