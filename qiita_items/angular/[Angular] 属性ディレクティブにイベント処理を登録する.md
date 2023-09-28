# はじめに
属性ディレクティブにはイベント処理を登録することができる。
本記事ではその方法について示す。

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

# 関連記事
* [[Angular] 属性ディレクティブを自作する](https://qiita.com/ksh-fthr/items/b8e3577f47483f5685e2)


# 本記事で扱う構成
本記事で扱う構成を以下に示す｡

```:構成
src/
  `-app/
    `-component/
    |  `-use-directive/                       # ディレクティブを適用するコンポーネント
    |    `- use-directive.component.css
    |    `- use-directive.component.html
    |    `- use-directive.component.ts
    `-directive/
    |  `- event.directive.ts                  # 本記事で扱うディレクティブ
    `- app.component.css
    `- app.component.html
    `- app.component.ts
    `- app.module.ts
```

# 属性ディレクティブでイベント処理

```typescript:event.directive.ts
import { Directive, ElementRef, HostListener } from '@angular/core';

@Directive({
  selector: '[appEvent]'
})
export class EventDirective {

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
   * イベントリスナー
   * マウスイベントをキャッチして背景色を変更する
   *
   * @param {*} target このディレクティブが適用されているhtml要素
   * @memberof EventDirective
   */
  @HostListener('mouseenter', ['$event.target']) changeBackgroundColor(target: any) {
    console.log(target);
    this.elementRef.nativeElement.style.backgroundColor = 'rgb(255, 0, 0)';
  }

  /**
   * イベントリスナー
   * マウスリーブイベントをキャッチして背景色を変更する
   *
   * @param {*} target このディレクティブが適用されているhtml要素
   * @memberof EventDirective
   */
  @HostListener('mouseleave', ['$event.target']) restoreBackgroundColor(target: any) {
    console.log(target);
    this.elementRef.nativeElement.style.backgroundColor = 'rgb(219, 210, 224)';
  }
}
```

以下､ポイントを順にみていく｡

## @HostListener で修飾する

ディレクティブでイベント処理を登録するには @HostListener でメソッドを修飾してやればよい｡
具体的には

```typescript:HostListenerのimport
import { HostListener } from '@angular/core';
```

で ```HostListener``` を import するのが一点目｡

```typescript:@HostListenerでメソッドを修飾
  @HostListener('イベント名', '[イベント引数]') メソッド名(target: any) {
    // イベント処理を実装する
  }
```

としてメソッドを ```@HostListener``` で修飾してやるのが二点目｡
以上の二点でイベント登録が完了する｡

なお ```@HostListener``` で修飾する際､第一引数にイベント名､第二引数にイベント情報を指定することができる｡
そして第二引数で指定したイベント情報が実際のメソッド引数として渡されることになる｡

では実際のコードでイベント登録の箇所をみていく｡


## mouseenter のイベントリスナー

```typescript:mouseenter
  /**
   * イベントリスナー
   * マウスイベントをキャッチして背景色を変更する
   *
   * @param {*} target このディレクティブが適用されているhtml要素
   * @memberof EventDirective
   */
  @HostListener('mouseenter', ['$event.target']) changeBackgroundColor(target: any) {
    console.log(target);
    this.elementRef.nativeElement.style.backgroundColor = 'rgb(255, 0, 0)';
  }
```

こちらは ```mouseenter``` のイベントリスナー｡
本記事でみているディレクティブ ```event.directive``` を適用しているコンポーネントの要素にマウスカーソルが当たると発火する｡
ここでは<font color="Red">背景色を赤色</font>に変更する単純な処理を行っている｡
なおログ出力を行っているが､出力している内容は **イベント発火時の要素の内容** となる｡


## mouseleave のイベントリスナー

```typescript:mouseleave
  /**
   * イベントリスナー
   * マウスリーブイベントをキャッチして背景色を変更する
   *
   * @param {*} target このディレクティブが適用されているhtml要素
   * @memberof EventDirective
   */
  @HostListener('mouseleave', ['$event.target']) restoreBackgroundColor(target: any) {
    console.log(target);
    this.elementRef.nativeElement.style.backgroundColor = 'rgb(219, 210, 224)';
  }
```

こちらは ```mouseleave``` のイベントリスナー｡
```mouseenter``` と同じく､ディレクティブ ```event.directive``` を適用しているコンポーネントの要素からマウスカーソルが外れると発火する｡
処理している内容は背景色を元に戻すだけの単純なもの｡
出力しているログも ```mouseenter``` と同じで､ **イベント発火時の要素の内容** を出力している｡


# コンポーネントからディレクティブを利用する

単純にディレクティブを適用するだけのコードとなるが､利用する側であるコンポーネントのテンプレートを示すと次のとおり｡

```html:use-directive.component.html
<p>
  属性ディレクティブにイベント処理を登録するサンプル。<br>
  <span appEvent>ここの背景色</span> がマウスエンターとマウスリーブのイベントで切り替わる。
</p>
```

```<span>``` タグで囲った部分が ```event.directive``` が適用される箇所で､前掲のイベントリスナーで引数に渡され､かつログ出力される情報は､この ```<span>``` タグで囲われた部分となる｡


# 実行結果

## 起動直後

![directive_event01.png](https://qiita-image-store.s3.amazonaws.com/0/193342/d88ee9d1-673f-f081-0d8a-5ba398fd8d7d.png)

起動直後なのでログ出力もなく､単純にページを表示しただけ｡


## ｢ここの背景色｣ にマウスエンター

![directive_event02.png](https://qiita-image-store.s3.amazonaws.com/0/193342/81fcf8c3-a6c7-0b16-5ccc-d27928462f7a.png)

```mouseenter``` イベントが発火され､<font color="Red">背景色が赤色</font>に変更されている｡
またログには ( ```background-color: rgb(219, 210, 224)``` となっていることから ) イベント発火時の情報が出力されていることがわかる｡


## ｢ここの背景色｣からマウスリーブ

![directive_event03.png](https://qiita-image-store.s3.amazonaws.com/0/193342/e696bf25-e283-2bf3-b938-d7a1dd53c774.png)

```mouseleave``` イベントが発火され､背景色が元に戻っていることが確認できる｡
こちらもログには ( ```background-color: rgb(255, 0, 0)``` となっていることから ) イベント発火時の情報が出力されていることがわかる｡


# ソースコード

今回の記事で動作確認に使用したコードは [ここ](https://github.com/ksh-fthr/angular-work/tree/feat_directive_event) にアップしたのでご参考までに。。。


# 参考

* [本家の Attribute Directives](https://angular.jp/guide/attribute-directives.en)

