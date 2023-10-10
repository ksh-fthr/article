# はじめに
Angular コンポーネントのライフサイクルについて、本記事では ngAfterViewInit と ngAfterViewChecked についてみていく。

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
* [[Angular] ライフサイクルメソッドをみる(ngAfterContentInit と ngAfterContentChecked)](https://qiita.com/ksh-fthr/items/bf8fb8c66cd1d044866e)
* [[Angular] 子コンポーネントや外部コンテンツの参照を取得する](https://qiita.com/ksh-fthr/items/00341b3b12f7048c9575)

# ngAfterViewInit と ngAfterViewChecked

これらのメソッドはメソッド名から推測しやすい。つまりビューの初期化と変更のタイミングでそれぞれが実行されるもので、

- ngAfterViewInit は **ビューの初期化後に一度だけ**
- ngAfterViewChecked は **ビューが変更される度** に

それぞれ実行される。
以下、実際にコードで確認してみる。

## ルートコンポーネント

ここでは単純に今回の確認対象となるコンポーネントを登録するだけ。

```html:app.component.html
<!-- 親コンポーネントを登録 -->
<app-view-parent>
</app-view-parent>
```

## 親コンポーネント

親コンポーネントである view-parent.component では

* 入力フォームの設置
* テンプレートで子コンポーネントを登録
* クラスでは子コンポーネントへの参照を取得して ngAfterViewChecked で更新

を行う。

```html:view-parent.component.html
<p>
  親コンポーネントの入力フォームからビューの変更を確認する
</p>

<label for="inputText">入力項目(親):</label>
<input id="inputText" name="inputText" type="text" [(ngModel)]="ngAfterViewCheckValue" />

<!-- 子コンポーネントを登録 -->
<app-view-child>
</app-view-child>
```

```typescript:view-parent.component.ts
import { Component, OnInit } from '@angular/core';

// ngAferViewInit と ngAfterViewChecked を利用するための import
import { AfterViewInit, AfterViewChecked } from '@angular/core';

// 子コンポーネントで定義されたプロパティを取得するための import
import { ViewChild } from '@angular/core';

// 子コンポーネントを import
import { ViewChildComponent} from '../view-child/view-child.component';

@Component({
  selector: 'app-view-parent',
  templateUrl: './view-parent.component.html',
  styleUrls: ['./view-parent.component.css']
})
export class ViewParentComponent implements OnInit, AfterViewInit, AfterViewChecked {

  /**
   * ngAfterViewInit と ngAfterViewChecked の確認のためのプロパティ
   *
   * @type {String}
   * @memberof ViewChildComponent
   */
  public ngAfterViewCheckValue: String = '';

  /**
   * 子コンポーネントを参照
   *
   * @type {ViewChildComponent}
   * @memberof ViewParentComponent
   */
  @ViewChild(ViewChildComponent) viewChild!: ViewChildComponent;

  constructor() { }

  /**
   * コンポーネントの初期化処理
   *
   * @memberof ViewParentComponent
   */
  ngOnInit() {
    this.ngAfterViewCheckValue = 'ngOnInitで初期化した';
  }

  /**
   * ビューの初期化をフックする
   *
   * @memberof ViewParentComponent
   */
  ngAfterViewInit() {
    console.log('[ViewParentComponent][ngAfterViewInit] fired');
  }

  /**
   * ビューの変更をフックする
   *
   * @memberof ViewParentComponent
   */
  ngAfterViewChecked() {
    console.log('[ViewParentComponent][ngAfterViewChecked] fired. ngAfterViewCheckValue={' + this.ngAfterViewCheckValue + '}');
  }
}
```

## 子コンポーネント

子コンポーネントである view-child.component では

* 入力フォームの設置

を行う。

```html:view-child.component.html
<p>
  子コンポーネントの入力フォームからビューの変更を確認する
</p>

<label for="inputText">入力項目(子):</label>
<input id="inputText" name="inputText" type="text" [(ngModel)]="ngAfterViewCheckValue" />
```

```typescript:view-child.component.ts
import { Component, OnInit } from '@angular/core';

// ngAferViewInit と ngAfterViewChecked を利用するための import
import { AfterViewInit, AfterViewChecked } from '@angular/core';

@Component({
  selector: 'app-view-child',
  templateUrl: './view-child.component.html',
  styleUrls: ['./view-child.component.css']
})
export class ViewChildComponent implements OnInit, AfterViewInit, AfterViewChecked {

  /**
   * ngAfterViewInit と ngAfterViewChecked の確認のためのプロパティ
   *
   * @type {String}
   * @memberof ViewChildComponent
   */
  public ngAfterViewCheckValue: String = '';

  constructor() { }

  /**
   * コンポーネントの初期化処理
   *
   * @memberof ViewChildComponent
   */
  ngOnInit() {
    this.ngAfterViewCheckValue = 'ngOnInitで初期化した';
  }

  /**
   * ビューの初期化をフックする
   *
   * @memberof ViewChildComponent
   */
  ngAfterViewInit() {
    console.log('[ViewChildComponent][ngAfterViewInit] fired.');
  }

  /**
   * ビューの変更をフックする
   *
   * @memberof ViewChildComponent
   */
  ngAfterViewChecked() {
    console.log('[ViewChildComponent][ngAfterViewChecked] fired. ngAfterViewCheckValue={' + this.ngAfterViewCheckValue + '}');
  }
}
```

## このサンプルコードの目的

上記では親コンポーネントと子コンポーネントの両方で入力フォームを設置した。
その目的は

* 親コンポーネントでビューを変更した際に子コンポーネントで検知するのか
* 逆に子コンポーネントでビューを変更した際に親コンポーネントで検知するのか

を確認するためである。

## 実行結果を確認する

アプリを起動して実行結果をログから確認する。

### 起動直後

![ngAfterView-boot.png](https://qiita-image-store.s3.amazonaws.com/0/193342/4885988d-011f-db51-efa2-39667d0c84ab.png)

起動直後のログでは下記が確認できる。

* 子コンポーネントで ngAfterViewInit -> ngAfterViewChecked の順で実行され (オレンジ色の枠)
* 次に親コンポーネントで ngAfterViewInit -> ngAfterViewChecked の順で実行された (青色の枠)

各メソッドの実行順序が

- 子コンポーネントの ngAfterViewInit -> 親コンポーネントの ngAfterViewInit の順、
- あるいは子コンポーネントの ngAfterViewChecked -> 親コンポーネントの ngAfterViewChecked の順

**ではなく**、**そのコンポーネント内で連続して実行**されている点に注目。
ビューの初期化、変更を検知して実行された後に次のコンポーネントの処理に移っていることから

- **コンポーネント単位で Init と Checked が行われる**

ことが分かる。

### 親子のコンポーネントでの入力による更新

親コンポーネントの入力フォームで「親コンポーネントでの入力による更新」と入力した際のログが下図 (紫色の枠)
 である。

![ngAfterView-input-parent.png](https://qiita-image-store.s3.amazonaws.com/0/193342/d7841853-d11e-ea80-05ea-13a510b4b167.png)

そして子コンポーネントの入力フォームで「親コンポーネントでの入力による更新」と入力した際のログが下図 (緑色の枠) である。このとき一度 F5 でリロードしてから操作を行った。

![ngAfterView-input-child.png](https://qiita-image-store.s3.amazonaws.com/0/193342/bdb3af30-abe2-8a24-b60f-175db7ef7de0.png)

両者のログからは

* 子コンポーネントの ngAfterViewChecked が実行され
* 次に親コンポーネント ngAfterViewChecked が実行された

ことが確認できる。また親と子のコンポーネントのどちらで入力しても

* 親コンポーネントの ngAfterViewChecked と 子コンポーネントの ngAfterViewChecked が実行された

ことも確認できる。なお ngAfterViewInit はアプリ起動後一度しか実行されないため、本操作時は  ngAfterViewChecked だけが実行されている。

注目は

- **親コンポーネントと子コンポーネントの両方の ngAfterViewChecked が実行**

されている点で、このことからビューの変更が行われたら、

- **コンポーネント単位の変更ではなくそのページ全体の変更として検知**されていること
- また **ngAfterViewChecked を実装したコンポーネント全てで変更をフック**している

ことが分かる。

# まとめにかえて

実行結果の確認で記載した内容と重複するが、次のことを確認できた。

* ngAfterViewInit と ngAfterViewChecked は**コンポーネント単位で Init と Checked が行われる**こと
* ビューの変更が行われたら**コンポーネント単位の変更ではなくそのページ全体の変更として検知**されていること
* **ngAfterViewChecked を実装したコンポーネント全てで変更をフック**していること

上記の関連記事でみてきたライフサイクルメソッドについても言えることだが、コンポーネントやビューの初期化や変更があった場合、ライフサクルメソッドはそれをフックして処理が行われる。このとき Angular は「どのコンポーネントで初期化や更新が行われたか」ではなく「単純に初期化や更新が行われた」ことに対してフックしているのだと思う。

ということは、ライフサイクルメソッドは単純に全てのコンポーネントで実装すれば良いわけではなく、**各ライフサイクルメソッドの特性を理解**した上で**それが必要なケースや箇所を見極めて**利用する必要がある、と捉えることができる。

特に ngOnChanges や ngDoCheck、ngAfterContentChecked や ngAfterViewChecked などは変更を検知するたびに実行されるので、何も考えずに全てのコンポーネントで実装していたら実行回数がとんでもないことになる。充分に注意したい。

# 補足

ngAfterViewInit や ngAfterViewChecked で、**ビューに絡むプロパティの更新**を行うと次のエラーが発生する。

> ERROR Error: ExpressionChangedAfterItHasBeenCheckedError: Expression has changed after it was checked

その回避策については

* https://qiita.com/tomonari-t/items/3fd6d3c30b6b007b0f14

に詳しい。ご興味あれば参考にされたい。

# ソースコード

今回の記事で動作確認に使用したコードは [ここ](https://github.com/ksh-fthr/angular-work/tree/feat_life_cycle_afterview) にアップしたので、ご参考までに。。。
