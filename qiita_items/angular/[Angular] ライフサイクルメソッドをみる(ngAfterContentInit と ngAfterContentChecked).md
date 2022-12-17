
# はじめに
Angular コンポーネントのライフサイクルについてみていく。
過去の記事は [こちら](https://qiita.com/ksh-fthr/items/ccd9861f919c4aa30ae8) と [こちら](https://qiita.com/ksh-fthr/items/f1adea56c17f8c7f6c0d)。
今回は ngAfterContentInit と ngAfterContentChecked を対象とする。

# 更新情報

## 2021/01/03

- 記事内で扱ったコードを Angular `v11.0.5` で確認しました

# 作業環境

| 環境                                          | バージョン          | 備考                                |
| --------------------------------------------- | ------------------- | ----------------------------------- |
| [Angular CLI](https://cli.angular.io/)        | ~~v1.6.3~~ v11.0.5  | `$ ng --version`                    |
| [Angular](https://angular.io/)                | ~~v4.4.6~~ v11.0.5  | 同上                                |
| [TypeScript](https://www.typescriptlang.org/) | v4.0.2       | 同上                |
| [Node.js](https://nodejs.org/ja/)             | ~~v9.2.1~~ v12.18.3 | `$ node --version`                  |
| [npm](https://www.npmjs.com/)                 | ~~v5.6.0~~ v6.14.6  | `$ npm --version`                   |

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
* [[Angular] ライフサイクルメソッドをみる(ngOnChanges と ngOnInit と ngOnDestroy)](https://qiita.com/ksh-fthr/items/ccd9861f919c4aa30ae8)
* [[Angular] ライフサイクルメソッドをみる(ngDoCheck)](https://qiita.com/ksh-fthr/items/f1adea56c17f8c7f6c0d)
* [[Angular] ライフサイクルメソッドをみる(ngAfterViewInit と ngAfterViewChecked)](https://qiita.com/ksh-fthr/items/411d2884875a4a0f7bd6)
* [[Angular] 子コンポーネントや外部コンテンツの参照を取得する](https://qiita.com/ksh-fthr/items/00341b3b12f7048c9575)

# ng-content

ngAfterContentInit や ngAfterContentChecked をみていくにあたり、まず ng-content というものがキーワードになる。
両者に関する説明は [本家の AfterContent](https://angular.io/guide/lifecycle-hooks#aftercontent) にかかれていて、そのなかで ng-content について説明されている。

大雑把にいうと、

* ng-content は外部コンテンツをコンポーネントのコンテンツとして埋め込むために利用できる
* ngAfterContentInit や ngAfterContentChecked では、それらのコンテンツが埋め込まれたタイミングでフックされて実行される

といったニュアンスで捉えることができる。

ここで「**外部コンテンツ**とはなにか」となるのだが、[こちらの記事](https://codezine.jp/article/detail/10046) をみると[用語説明]にこうある。

> 「外部コンテンツ」は、リスト2のpタグ要素のように、コンポーネントタグで囲まれた内容です。

引用した文中の「リスト2のpタグ要素」云々については記事を見ていただくとして、(オウム返しになるが)単純にコンポーネントタグで囲った部分が**外部コンテンツ**となるようだ。

さて、[本家](https://angular.io/guide/lifecycle-hooks#aftercontent) では例を示しつつの説明となっているので、それを参考にしつつ本記事でも ng-content を使用した例を挙げてみたい。
まずは単純に ng-content を利用したコンテンツの埋め込みの例から。

## 文字列をコンテンツとして埋め込む

ここでは ルートコンポーネント -> 親コンポーネント -> 子コンポーネント の順でコンポーネントを呼び出していき、 **親コンポーネントで記載した文字列を子コンポーネントのコンテンツとして埋め込む** 例を示す。

### ルートコンポーネント

```html:app.component.html
<app-content-parent>
</app-content-parent>
```

ルートコンポーネントは

* 親コンポーネントである ```content-parent.component.html``` を読み込むためのタグを設定

しているだけ。

### 親コンポーネント

```html:content-parent.component.html
<app-content-child>
  「Child コンポーネントにコンテンツとして文字列をセットする」
</app-content-child>
```

親コンポーネントでは

* 子コンポーネントである ```content-child.component.html``` を読み込むためのタグを設定し、
* そのタグのなかで ```content-child.component.html``` に埋め込むためのコンテンツとして「Child コンポーネントの呼び出し」
を記載

している。

### 子コンポーネント

```html:content-child.component.html
Parent コンポーネントから <ng-content></ng-content> が設定されました。
```

子コンポーネントでは親コンポーネントで設定されたコンテンツを読み込むために ng-content を使用している。

### 実行結果

ではここで実行結果を見てみると。。。

![ng-content-01.png](https://qiita-image-store.s3.amazonaws.com/0/193342/0a5e73dd-47bd-e6ec-1f8e-dbadf073e2fc.png)

このとおり、親コンポーネントである ```content-parent.component.html``` で記載したコンテンツが ```content-child.component.html``` に埋め込まれて表示されているのがわかる。

## コンポーネントをコンテンツとして埋め込む

先程の例では**親コンポーネントから子コンポーネント**に対して ng-content でコンテンツを埋め込んでいたが、今度は親コンポーネント側に ng-content を記載し、**子コンポーネントを親コンポーネントのコンテンツをとして埋め込む**例を示す。

### ルートコンポーネント

```html:app.component.html
<app-piyo-parent>
  <app-piyo-child>
  </app-piyo-child>
</app-piyo-parent>
```

ルートコンポーネントである app.component.html では

* 親コンポーネントの ```piyo-parent.component.html```
* 子コンポーネントの ```piyo-child.component.html```

の両者を埋め込むためのタグを設定している。
ここで、**子コンポーネントを親コンポーネントの入れ子に設定**している点に注目。

### 親コンポーネント

```html:piyo-parent.component.html
<ng-content></ng-content>
```

親コンポーネントでは ng-content タグを設定し、外部コンテンツを読み込めるようにしている。
ここが先のルートコンポーネントとあわせて今回の例で重要な点で、こうすることで**外部コンテンツとして子コンポーネントを埋め込む**動きとなる。

### 子コンポーネント

```html:piyo-child.component.html
<p>
  piyo-child works!
</p>
```

子コンポーネントは通常のコンポーネントと同じように、単純に自分自身のコンポーネント内容を実装するだけなので、特筆すべき点はなし。

### 実行結果

本例の実行結果とみると。。。

![ng-content-02.png](https://qiita-image-store.s3.amazonaws.com/0/193342/e40ebbab-0fba-8395-469d-dd144fe5d53a.png)

といった感じで、子コンポーネントそのものが親コンポーネントのコンテンツとして表示されているのがわかる。

# ngAfterContentInit と ngAfterContentChecked

次に ng-content で埋め込まれたコンテンツをフックする処理である ngAfterContentInit と ngAfterContentChecked についてみる。
先の [ng-content を利用してコンポーネントをコンテンツとして埋め込む](https://qiita.com/ksh-fthr/items/bf8fb8c66cd1d044866e#ng-content-%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%A6%E3%82%B3%E3%83%B3%E3%83%9D%E3%83%BC%E3%83%8D%E3%83%B3%E3%83%88%E3%82%92%E3%82%B3%E3%83%B3%E3%83%86%E3%83%B3%E3%83%84%E3%81%A8%E3%81%97%E3%81%A6%E5%9F%8B%E3%82%81%E8%BE%BC%E3%82%80) の例をもとに ngAfterContentInit と ngAfterContentChecked を確認するコードを書いていく。

## 親コンポーネント

まずテンプレートだが、こちらは前掲の例と変わらず、ng-content タグを設定することで子コンポーネントを外部コンテンツとして埋め込んでいる。

```html:piyo-parent.component.html
<ng-content></ng-content>
```

次にロジックである ts ファイルだが、これが本例における重要な部分となる。
コード中のコメントにも記述してあるが、ポイントみていくと

* import 部分で
 * AfterContentInit, AfterContentChecked を指定してライフサイクルメソッドのインターフェースを import
 * ContentChild を指定して外部コンテンツである子コンポーネントを参照するためのデコレータを import
 * PiyoChildComponent を指定して子コンポーネントを import
* クラス定義で
 * @ContentChild デコレータを使用して PiyoChildComponent の参照を取得
 * ngAfterContentInit, ngAfterContentChecked のライフサイクルメソッドを実装
 * 上記のために AfterContentInit, AfterContentChecked を implements

となる。

```typescript:piyo-parent.component.ts
import { Component, OnInit } from '@angular/core';

// ngAfterContentInit と ngAfterContentChecked を利用するための import
import { AfterContentInit, AfterContentChecked } from '@angular/core';

// 外部コンテンツとして埋め込まれた子コンポーネントを参照するための import
import { ContentChild } from '@angular/core';

// 外部コンテンツである子コンポーネントを import
import { PiyoChildComponent } from '../piyo-child/piyo-child.component';

@Component({
  selector: 'app-piyo-parent',
  templateUrl: './piyo-parent.component.html',
  styleUrls: ['./piyo-parent.component.css']
})
export class PiyoParentComponent implements OnInit, AfterContentInit, AfterContentChecked {

  // 外部コンテンツである子コンポーネントを参照
  @ContentChild(PiyoChildComponent) child!: PiyoChildComponent;

  // 子コンポーネントのプロパティの値をセットするプロパティ
  private contents: String = '';

  constructor() {
    console.log('[PiyoParentComponent][constructor] fired');
  }

  /**
   * コンポーネントの初期化
   */
  ngOnInit() {
    console.log('[PiyoParentComponent][ngOnInit] fired');
  }

  /**
   * 外部コンテンツが初期化された後に処理
   */
  ngAfterContentInit() {
    this.contents = this.child.contents;
    console.log('[PiyoParentComponent][ngAfterContentInit] fired. conents={' + this.contents + '}');
  }

  /**
   * 外部コンテンツの確認後に処理
   */
  ngAfterContentChecked() {
    this.contents = this.child.contents;
    console.log('[PiyoParentComponent][ngAfterContentChecked] fired. contents={' + this.contents + '}');
  }
}
```

## 子コンポーネント

子コンポーネントのテンプレートでは親コンポーネントで実装した ngAfterContentChecked の動きをわかりやすくするため、input フォームを設定した。

```html:piyo-child.component.html
<p>
    <input id="inputText" name="inputText" type="text"  size="50" [(ngModel)]="contents" />
</p>
```

ロジックとなる ts ファイルでは

* テンプレートの input と連動させるためのプロパティとして contents を定義
* ライフサイクルの動きをみるためのログを constructor と ngOnInit に実装

をしている。

```typescript:piyo-child.component.ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-piyo-child',
  templateUrl: './piyo-child.component.html',
  styleUrls: ['./piyo-child.component.css']
})
export class PiyoChildComponent implements OnInit {

  public contents: String = '';

  constructor() {
    console.log('[PiyoChildComponent][constructor] fired');
  }

  /**
   * コンポーネントの初期化
   */
  ngOnInit() {
    this.contents = '子コンポーネント で初期化しました。';
    console.log('[PiyoChildComponent][ngOnInit] fired. contents={' + this.contents + '}');
  }
}
```

## 実行結果

本例の実行結果は、起動直後が

![ng-content-03.png](https://qiita-image-store.s3.amazonaws.com/0/193342/1ba41795-5f9c-66a5-f024-49e3bf86760e.png)

となって、ngAfterContentChecked が走ったときは

![ng-content-04.png](https://qiita-image-store.s3.amazonaws.com/0/193342/e402c82d-8d8d-925a-c000-971d5fc65f52.png)

となる。
このときの操作は次の手順のみを行った。

* 入力項フォームに「コンテンツを更新したよ」をコピー&ペーストで入力

ログの状況を追っていくと、起動直後には

* PiyoParentComponent, PiyoChildComponent の順にコンストラクタのログが出力
* PiyoParentComponent, PiyoChildComponent の順に ngOnInit のログが出力

となっており、コンポーネントの初期化処理がここまでで完了していることがわかる。
ここまでは [こちら](https://qiita.com/ksh-fthr/items/ccd9861f919c4aa30ae8) と [こちら](https://qiita.com/ksh-fthr/items/f1adea56c17f8c7f6c0d) の記事のおさらいのようなもの。
で、その次をみると

* PiyoChildComponent の ngAfterInit と ngAfterContentChecked のログが出力

されている。
ここが最初のポイントで、

* PiyoChildComponent、つまり**外部コンテンツである子コンポーネントの初期化が完了した後に ngAfterInit が実行**されていること
* 次に**外部コンテンツの初期化が完了し、外部コンテンツが埋め込まれたことを検知したことで ngAfterContentChecked が実行**されたこと

を示している。
最後に入力フォームをコピー&ペーストで更新したときのログとして

* PiyoChildComponent の ngAfterContentChecked のログが出力

されている。これは

* **外部コンテンツが更新されたことを検知したことで ngAfterContentChecked が実行**された

ことを示している。

# まとめにかえて

本記事では次のことを確認した。

* ng-content を用いることで外部コンテンツをコンポーネントに埋め込むことができる
* 外部コンテンツにはコンポーネントも対象とすることができる
* ngAfterContentInit や ngAfterContentChecked はコンポーネントの初期化完了後に実行される

正直なところ ng-content と ngAfterContentInit, ngAfterContentChecked の動きを確認できただけで、具体的にこれがアプリを作っていく上でどう有効なのかまでは理解できていない。
このあとは ngAfterViewInit, ngAfterViewChecked, ngOnDestroy が残っているので、それらの確認が終わったら改めて考えてみたい。

# 補足

ng-content の利用にあたっての注意点と便利な利用法について記載しておく。

## 注意点
ng-content はルートコンポーネントで使用することはできない。
[ここ](https://github.com/angular/angular/issues/4946) とか [ここ](https://github.com/angular/angular/issues/1858) をみると、そのやりとりが確認できる。

## 便利な利用法
**class 属性と select 属性の対応付けで任意のコンテンツを取得**できる。
どういうことかというと、

```html:app.component.html
<div>
  <div class="block-header">
    <p>AfterContent の動きを理解するためのブロック</p>
  </div>

  <div class="block-body">
    <app-piyo-parent>
      <app-piyo-child class="external-contents-component">
      </app-piyo-child>
      <p class="external-contents-sentence">
        外部コンテンツとして文を埋め込む
      </p>
    </app-piyo-parent>
  </div>
</div>
```

外部コンテンツとなる要素に対して class 属性を設定し、

```html:piyo-parent.component.html
<ng-content select=".external-contents-component"></ng-content>
<ng-content select=".external-contents-sentence"></ng-content>
```

外部コンテンツを埋め込む側で select 属性を設定する。
このとき **class 属性の値と select 属性の値を対応させる**ことで、**任意の場所にコンテンツを埋め込む**ことができる。

上記コードの実行結果も貼っておく。

![ng-content-05.png](https://qiita-image-store.s3.amazonaws.com/0/193342/b0e7b2ca-cf65-79cd-edd4-f34764451f69.png)

(オレンジ枠が「external-contents-component」で指定したコンテンツ、青枠が「external-contents-sentence」で指定したコンテンツ)

# ソースコード

今回の記事で動作確認に使用したコードは [ここ](https://github.com/ksh-fthr/angular-work/tree/feat_life_cycle_aftercontent) にアップしたので、ご参考までに。。。
