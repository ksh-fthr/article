# はじめに
Angular でコンポーネントを開発していく際、複数のコンポーネントを親子として関連付けて扱うことができる。
例えば

* comp-parent コンポーネント
* comp-child コンポーネント

という2つのコンポーネントがあり、comp-parent が親、comp-child を子として関連付ける場合、次のようにテンプレートを定義することで親子関係を作ることができる。

```html:app.component.html(サンプル)
<!-- ルートコンポーネントでは親コンポーネントのタグをセット -->
<app-comp-parent>
</app-comp-parent>
```

```html:app.parent.component.html(サンプル)
<!-- 親コンポーネントで子コンポーネントのタグをセット -->
<app-comp-child>
</app-comp-child>
```

今回はこうした親子関係をもつコンポーネント間における値の受け渡しについて確認する。
具体的には @Input、@Output デコレータを用いた確認をしていく。



# 更新情報

## 2021/01/03
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

* [[Angular] 子コンポーネントや外部コンテンツの参照を取得する](https://qiita.com/ksh-fthr/items/00341b3b12f7048c9575)

こちらは @ViewChild、@ViewChildren、@ContentChild、@ContentChildren デコレータを使用した子コンポーネントや外部コンテンツの**参照**を取得して、参照先のプロパティの値を取得する方法についての記事。
上記記事と本記事の違いは、上記記事が**コンポーネントの参照を取得する**方法に対して、本記事は**コンポーネントが扱うプロパティを取得する**点である。

では実際にコードで確認していく。

# @Input デコレータ

@Input デコレータを使用するにあたり、 **import が必要なのは子コンポーネント側**となる。
というわけで、子コンポーネントのテンプレートとクラスからみていく。


## 子コンポーネント

### テンプレート

この例では子コンポーネントでは親コンポーネントから受け取ったデータを表示するが、

* @Inputデコレータで修飾されたプロパティへデータをセットする処理は後述する親コンポーネント側のテンプレートで行う

ので、ここでは単純にプロパティを片方向データバインドで扱うだけで良い。

```html:comp-child.component.html
<div>
  <p>
    親コンポーネントから渡された文字列は「{{dataFromParent}}」です。
  </p>
</div>
```

### クラス

クラスでは単純に

* @Input デコレータを使用するために必要な import 宣言
* @Input デコレータで修飾したプロパティを宣言

するだけで良い。
これで親コンポーネントからデータを受け取るための準備が整ったので、後は親コンポーネント側でデータを渡すための処理を書けば、親コンポーネントから子コンポーネントへデータを渡すことができる。

```typescript:comp-child.component.ts
// ...
// 省略
// ...

// @Input デコレータを使用するための import
import { Input } from '@angular/core';

// ...
// 省略
// ...

export class CompChildComponent implements OnInit {

  /**
   * 親コンポーネントから受け取るデータ(文字列)をセットするプロパティ
   *
   * @type {String}
   * @memberof CompParentComponent
   */
  @Input() dataFromParent: String = '';
}
```

## 親コンポーネント

親コンポーネント側では @Input デコレータ を使ってデータを渡す際に import を行う必要はなく、単純に子コンポーネントで用意されたプロパティにデータをセットしてやるだけで良い。

### テンプレート

子コンポーネントのタグに対して

* 子コンポーネントの属性として、子コンポーネント側で@Inputデコレータを付与されたプロパティを指定
* そのプロパティに対して親コンポーネントで定義したプロパティを代入

することで、親コンポーネントが持つプロパティの値を子コンポーネントに渡すことができる。

```html:comp-parent.component.html
<div>
  <app-comp-child [dataFromParent]="parentData"></app-comp-child>
</div>
```

### クラス

親コンポーネントのクラスで行う特別な処理はなく、子コンポーネントに渡すためのデータのセットだけ行っている。

```typescript:comp-parent.component.ts
// ...
// 省略
// ...

export class CompParentComponent implements OnInit {

  /**
   * 子コンポーネントに渡すための文字列をセットするプロパティ
   *
   * @public
   * @type {String}
   * @memberof CompParentComponent
   */
  public parentData: String = '';

  /**
   * コンポーネントの初期化処理
   *
   * @memberof CompParentComponent
   */
  ngOnInit() {
    this.parentData = '親コンポーネントから文字列を渡します';
  }
}
```


## 実行結果

上記を実行すると次の図となる。

![use_@input.png](https://qiita-image-store.s3.amazonaws.com/0/193342/d2e69117-71a7-b906-0cad-521e824448f3.png)

オレンジ枠で囲った部分が @Input デコレータで修飾したプロパティを片方向データバインドした部分。
親コンポーネントでセットした文字列が表示されていることが確認できる。


# @Output デコレータ

前掲の @Input デコレータ同様 @Output デコレータでも**必要な import は子コンポーネント側**で行う。
ここでも先に子コンポーネントのテンプレートとクラスをみる。

なお @Input デコレータとは異なり、 @Output デコレータを扱った処理ではデータを渡すのではなく、 **イベントの発火** によって子コンポーネントから親コンポーネントに対して**通知を行う**動きとなる。
それに伴い @Output デコレータでは @Input デコレータよりも下準備が少々面倒である点に注意すること。

## 子コンポーネント

### テンプレート

この例ではボタンがクリックされたイベントをトリガーに親コンポーネントへのイベント通知を行う。
そのための準備として

* 親コンポーネントに対してイベントを発火するためのボタンを設置
* click イベントをキャッチする onClick イベントハンドラ内で親コンポーネントに対してイベントを発火する

ための設定をテンプレートとして定義している。

```html:comp-child.component.html
<div>
  <input type="button" value="今度は親コンポーネントへイベント発火" (click)="onClick()"/>
</div>
```


### クラス

クラスでは @Output デコレータを使用するための import、そして イベント通知のための import を行っている。
これらの import で宣言したモジュールを使用して親コンポーネントへのイベント通知を実現する。
具体的には

* @Output デコレータで修飾したプロパティを宣言し、そのプロパティに EventEmitter のインスタンスをセット
* ボタンクリック時のイベントハンドラ内で、@Output デコレータで修飾したプロパティを使用して emit によるイベントを発火

している。これで親コンポーネントへのイベント通知が実装されたことになる。

```typescript:comp-child.component.ts
// ...
// 省略
// ...

// @Output デコレータを使用するための import
import { Output, EventEmitter } from '@angular/core';

// ...
// 省略
// ...

export class CompChildComponent implements OnInit {

  /**
   * 親コンポーネントに対してイベントを発火するためのプロパティ
   *
   * @memberof CompChildComponent
   */
  @Output() event = new EventEmitter<String>();

  /**
   * イベントハンドラ
   * ボタンクリック時のイベントをキャッチして親コンポーネントへのイベントを発火する
   *
   * @memberof CompChildComponent
   */
  onClick() {
    this.event.emit('子コンポーネントから親コンポーネントへデータを渡す際はイベントを経由します。');
  }
}
```


## 親コンポーネント

### テンプレート

親コンポーネントのテンプレートでは子コンポーネントから発火されたイベントを受け取るために次のことを行っている。

* 子コンポーネントのタグの属性として「子コンポーネントで @Output デコレータで修飾されたプロパティ」をセット
* そのプロパティに対して、親コンポーネントのクラスで定義したイベントハンドラを代入
* その際、子コンポーネントから渡されてくるイベント情報を受け取るために $event を引数にセット

そしてイベント受信によって受け取ったデータを表示するために

* `{{eventData}}` として親コンポーネント自身で宣言したプロパティを片方向データバインド

している。

```html:comp-parent.component.html
<div>
  <app-comp-child (event)="onReceiveEventFromChild($event)"></app-comp-child>
</div>
<hr />
<div>
  <p>{{eventData}}</p>
</div>
```


### クラス

親クラスのコンポーネントでは

* 子コンポーネントからのイベントをキャッチするためのイベントハンドラを用意
* その中でテンプレートで扱うためのプロパティに受信したイベント情報をセット

し、ここでセットした受信情報をテンプレートで片方向データバインドによって表示している。

```typescript:comp-parent.component.ts
// ...
// 省略
// ...

export class CompParentComponent implements OnInit {

  /**
   * 子コンポーネントからイベント発火で渡された文字列をセットするプロパティ
   *
   * @public
   * @type {String}
   * @memberof CompParentComponent
   */
  public eventData: String = '';

  /**
   * イベントハンドラ
   * 子コンポーネントから発火されたイベントをキャッチして文字列を受け取る
   *
   * @param {String} eventData 子コンポーネントから渡される文字列
   * @memberof CompParentComponent
   */
  onReceiveEventFromChild(eventData: String) {
    this.eventData = eventData;
  }
}
```


## 実行結果

上記を実行すると

![usr_@output_01.png](https://qiita-image-store.s3.amazonaws.com/0/193342/e09cef94-9f13-6e2d-2175-97ca2bf99313.png)

のように、まずボタン(オレンジ枠の部分)だけが設置された画面が表示される。
このボタンをクリックすると

![usr_@output_02.png](https://qiita-image-store.s3.amazonaws.com/0/193342/58d29b8e-c10b-2c72-261b-915f9e1fbf14.png)

文字列が表示された(オレンジ枠の部分)。
この文字列は親コンポーネントで宣言したプロパティを片方向データバインドによって表示したものである。
そしてそのパタメータへは子コンポーネントからのイベントをキャッチした際に受信したデータをセットしている。

この結果から、表示された文字列が子コンポーネントでセットした文字列で、それが @Output デコレータと EventEmitter によって親コンポーネントに通知されたことが確認できた。

# まとめにかえて

@Input デコレータと @Output デコレータを使用したデータの受け渡しについてみてきた。
@Input デコレータでは

* **プロパティへデータをセット**する処理

なので直感的にわかりやすい。
@Output デコレータは

* **イベントによる通知**、かつ **$event プロパティから受信したデータを取り出す**必要がある

ので少々わかりづらい、というのが感想である。
また両者ともに言えることとして

* 親コンポーネントが子コンポーネントのプロパティを知っていなければならない

というのがある。これが非常に違和感を感じた。

親子関係にあるコンポーネント、もしくは外部コンテンツとして取り込んだコンポーネントに対してデータをやり取りする方法として、[関連記事](https://qiita.com/ksh-fthr/items/00341b3b12f7048c9575) で扱った @ViewChild や @ContentChild 等があるが、あちらは対象となるコンポーネントの参照を取得するために、そのコンポーネントを import し、プロパティとして宣言している。従って扱うデータは「そのコンポーネントを対象としたものだ」というのが明示されるのでコードリーディングもし易い。

対して @Input デコレータや @Output デコレータで修飾されたプロパティは、それを扱う親コンポーネントにおいて明示されずに扱っているのでコードリーディング時、すぐに分からないのでは、と思った。
が、ここまで書いて「そもそも対象のプロパティは子コンポーネントのタグ要素に属性としてセットしているから、どのコンポーネントに属しているかは自明なのか」と考え直した。

少々とっちらかってしまったが、@Input デコレータ、@Output デコレータともに

* テンプレートとクラス双方での準備が必要であることからちょっと扱いが面倒であること
* 慣れてこないとソースコードをみたときに直感的にわかりづらい

というところでまとめに代えたい。


# ソースコード

今回の記事で動作確認に使用したコードは [ここ](https://github.com/ksh-fthr/angular-work/tree/feat_parent_child) にアップしたので、ご参考までに。。。
