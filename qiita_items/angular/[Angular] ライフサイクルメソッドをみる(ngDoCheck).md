
# はじめに
[こちらの記事](https://qiita.com/ksh-fthr/items/ccd9861f919c4aa30ae8) の続き。
今回は ngDoCheck についてみていく。

# 更新情報
## 2021/01/02
- 記事内で扱ったコードを Angular `v11.0.5` で確認しました

# 作業環境

| 環境                                   | バージョン          | 備考               |
| -------------------------------------- | ------------------- | ------------------ |
| [Angular CLI](https://cli.angular.io/) | ~~v1.6.3~~ v11.0.5  | `$ ng --version`   |
| [Angular](https://angular.io/)         | ~~v4.4.6~~ v11.0.5  | 同上                |
| [TypeScript](https://www.typescriptlang.org/) | v4.0.2       | 同上                |
| [Node.js](https://nodejs.org/ja/)      | ~~v9.2.1~~ v12.18.3 | `$ node --version` |
| [npm](https://www.npmjs.com/)          | ~~v5.6.0~~ v6.14.6  | `$ npm --version`  |

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
* [[Angular] ライフサイクルメソッドをみる(ngAfterContentInit と ngAfterContentChecked)](https://qiita.com/ksh-fthr/items/bf8fb8c66cd1d044866e)
* [[Angular] ライフサイクルメソッドをみる(ngAfterViewInit と ngAfterViewChecked)](https://qiita.com/ksh-fthr/items/411d2884875a4a0f7bd6)

# ngDoCheck
ngDoCheck は Change Detection が走るたびに実行される。
[本家のAPI仕様](https://angular.io/api/core/DoCheck) を確認すると、[注意事項](https://angular.io/api/core/DoCheck#description) として

> Note that a directive typically should not use both DoCheck and OnChanges to respond to changes on the same input, as ngOnChanges will continue to be called when the default change detector detects changes.
>(一部訳: *同じ入力値に対して ngOnChanges と併用すべきではない*)

とある。

なぜそんなことを言われているのかを ngOnChanges と ngDoCheck を混在させたコードで確認してみる。
(下記では子コンポーネントのコードのみ記載する。親コンポーネントのコードは [こちらの記事](https://qiita.com/ksh-fthr/items/ccd9861f919c4aa30ae8#%E8%A6%AA%E3%82%B3%E3%83%B3%E3%83%9D%E3%83%BC%E3%83%8D%E3%83%B3%E3%83%88) と同じなので割愛する)

## テンプレート
親コンポーネントから受け渡された値を表示するブロックと子コンポーネント自身で入出力するブロックを用意する。

```html:hoge-hoge.component.html
<p>
  親コンポーネントで入力された値は... {{hogeInputValue}}
</p>

<label for="inputText">入力項目(子だけ):</label>
<input id="inputText" name="inputText" type="text" [(ngModel)]="ngDoCheckValue" />
<p>
  子コンポーネントで入力された値は... {{ngDoCheckValue}}
</p>
```

## クラス
親コンポーネントからデータを受け取るプロパティ ngOnChangesValue に加え、自分自身の入出力で使用するプロパティ ngDoCheckValue を定義する。

```typescript:hoge-hoge.component.ts
// DoCheck は ngDoCheck を実装するためのインターフェース
import { Component, OnInit, Input, OnChanges, DoCheck, SimpleChanges } from '@angular/core';

@Component({
  selector: 'app-hoge-hoge',
  templateUrl: './hoge-hoge.component.html',
  styleUrls: ['./hoge-hoge.component.css']
})
export class HogeHogeComponent implements OnInit, OnChanges, DoCheck {

  /**
   * ngOnChanges の確認のためのプロパティ
   *
   * @type {String}
   * @memberof HogeHogeComponent
   */
  @Input()
  ngOnChangesValue: String = '';

  /**
   * ngDoCheck の確認のためのプロパティ
   *
   * @type {String}
   * @memberof HogeHogeComponent
   */
  ngDoCheckValue: String = 'Initial Value';

  constructor() {
    console.log('[constructor] fired');
  }

  ngOnInit(): void {
    console.log('[ngOnInit] fired');
  }

  ngOnChanges(changes: SimpleChanges): void {
    console.log('[ngOnChanges] fired. ngOnChangesValue={' + this.ngOnChangesValue + '}' );
  }

  ngDoCheck(): void {
    console.log('[ngDoCheck] fired. ngDoCheckValue={' + this.ngDoCheckValue + '}' );
  }
}
```

## 実行結果を確認する
コードを実行するとログには次のように出力される。

### 起動直後
![ngDoCheck-01.png](https://qiita-image-store.s3.amazonaws.com/0/193342/87bb44cf-9d92-abfb-7ee5-ceaa0b2144f5.png)
(オレンジの枠線が ngOnChanges、青の枠線が ngDoCheck、赤の枠線は Angular が出力しているログ)

ngOnChanges のログが出力された後に ngOnInit のログが出力され、その後に ngDoCheck のログが出力されていることから

* **ngOnChanges -> ngOnInit -> ngDoCheck の順に実行**されている

ことが確認できる。これは [本家の Lifecycle Hooks](https://angular.io/guide/lifecycle-hooks) にある図の通り。
気になるのは

```:実行時ログ
Angular is running in the development mode. Call enableProdMode() to enable the production mode.
```

の赤枠で囲ったログ(これは Angular が出力している) で、このログの後にもう一度 ngDoCheck のログが出力されている点である。
このことから DOM に変化があったら ngDoCheck が走っていることがわかる。

### ngDoCheck 単体を走らせたときの動作を確認する
では次に、アプリが起動して画面の表示が完了した後に ngDoCheck が実行される操作をした際の動作を確認する。
(このとき F5 でリローﾄﾞ(アプリの再実行)を行っている)

操作は以下を行った。

1. 「入力項目(子だけ)」 のフォームに「change value」をコピペで入力
1. フォーカスを入力フォームから移す

![ngDoCheck-03.png](https://qiita-image-store.s3.amazonaws.com/0/193342/ce9eab90-5b45-58d9-fd96-6b6e77a01702.png)

ログからは

* 画面起動後に出力されたログの後に、今回の操作によって出力された ```[ngDoCheck] fired. ngDoCheckValue={change value}``` だけが出力されていること

がわかる。
これは **子コンポーネント( つまり自分自身 ) の変更された値だけを検知して ngDoCheck だけが実行されている** といえる｡
ngDoCheck のログの先頭に数字の「2」が表示されていることについては後述する。 

### ngOnChanges を走らせたときの動作を確認する
今度はアプリが起動して画面表示が完了した後、 ngOnChanges が実行される操作をした際の ngDoCheck の動作を確認してみる。
(こちらも F5 でリローﾄﾞ(アプリの再実行)を行っている)

操作は以下を行った。

1. 「入力項目(親から子に渡る)」のフォームに「change value」をコピペで入力
1. フォーカスを入力フォームから移す

![ngDoCheck-02.png](https://qiita-image-store.s3.amazonaws.com/0/193342/de7db735-75e6-88e5-b605-7a5b05b04dc3.png)

ログから

* ngOnChanges の後に ngDoCheck が実行されていること

がわかる。
どういうことかというと、**ngDoCheck は ngOnChanges がフックしているプロパティ( 親コンポーネントから渡されたデータ ) の変更も検知して実行されている**、ということである。
こちらも ngDoCheck のログの先頭に数字の「2」が表示されていることについては後述する。 

### ngDoCheck のログの先頭に数字が表示される理由
この数字はカウンタであり、同じログが出力される際にカウントアップされている。
ではなぜ同じログが出力されているかだが、前掲の操作に

> フォーカスを入力フォームから移す

という項目を記載した。これが同じログが出力された原因である。
詳しくみていくと、

1. ngOnChanges ないし ngDoCheck が実行される操作を行う
 * ここでは各入力値の更新を行った
1. そうすることで ngDoCheck が実行され、ログには ```[ngDoCheck] fired. ngDoCheckValue={change value}``` が出力される
1. 次にフォーカスを入力フォームから移した
 * これは TAB キーによる移動でもマウスクリックによる移動でも良い
1. すると ngDoCheck が実行される
 * このとき入力値は変わっていないから同じログ( ```[ngDoCheck] fired. ngDoCheckValue={change value}``` ) が出力される
 * 出力されるログが同じなので数字がカウントアップされる

という流れとなる。
また入力フォームに再フォーカスしてもログはカウントアップ( ngDoCheck が実行 )された。

## 動作確認からわかること
上記までの動作確認で次のことがわかる。

* アプリ起動時の実行順序は [Lifecycle Hooks](https://angular.io/guide/lifecycle-hooks) の図面の通り
* ngDoCheck は ngOnChanges がフックしているプロパティの変更も検知して実行される
* ngDoCheck は入力フォームからフォーカスが外れたり再フォーカスされても実行される

冒頭で書いた **ngOnChanges と ngDoCheck を併用すべきではない** ことの理由は

* **ngDoCheck は ngOnChanges がフックしているプロパティの変更も検知して実行**される

にある。
**ngOnChanges でプロパティの変更を検知して処理を行うと、その変更がトリガーとなって、続いて ngDoCheck が実行されてしまう**ため、パフォーマンス上余計な負荷が発生する可能性がある。
また ngOnChanges で変更した値を、ngDoCheck で更に更新してしまう、といったバグを生む可能性もある。

上記以外にも

* ngDoCheck は入力フォームからフォーカスが外れたり再フォーカスされても実行される

ことから、 ngDoCheck は頻繁に実行されることがわかる。

# まとめにかえて
ngDoCheck の動作を確認してきたが、ngOnChanges との併用の問題であったり、呼び出し頻度であったりと使い所が難しいとメソッドであると感じた。

[本家の DoCheck](https://angular.io/guide/lifecycle-hooks#docheck) には次の一文がある。

> Use this method to detect a change that Angular overlooked.
> (訳: *Angular が見落とした変更を検知するために使う*)

「Angular が見落とした変更」というのが分かっていないので、まずはそこから理解していこう。

次もライフサイクルメソッドについてみていく。

# ソースコード
今回の記事で動作確認に使用したコードは [ここ](https://github.com/ksh-fthr/angular-work/tree/feat_life_cycle_docheck) にアップしたので、ご参考までに。。。
