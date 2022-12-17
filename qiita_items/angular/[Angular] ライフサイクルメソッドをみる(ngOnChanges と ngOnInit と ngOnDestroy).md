コンポーネントには **ライフサイクル** という概念があって、それはコンポーネントの生成直後から破棄されるまでの流れを示している。
で、コンポーネントにはこのライフサイクルの変化、つまりコンポーネントの状態の変化にそって実行される **ライフサイクルメソッド** が用意されている。

本記事では Angular コンポーネントのライフサイクルメソッドについてみていく。

## 関連記事
* [[Angular] Angular CLI によるコンポーネントの生成](https://qiita.com/ksh-fthr/items/9f73fa161a1d7798def2)
* [[Angular] ライフサイクルメソッドをみる(ngDoCheck)](https://qiita.com/ksh-fthr/items/f1adea56c17f8c7f6c0d)
* [[Angular] ライフサイクルメソッドをみる(ngAfterContentInit と ngAfterContentChecked)](https://qiita.com/ksh-fthr/items/bf8fb8c66cd1d044866e)
* [[Angular] ライフサイクルメソッドをみる(ngAfterViewInit と ngAfterViewChecked)](https://qiita.com/ksh-fthr/items/411d2884875a4a0f7bd6)

## ライフサイクルメソッドの種類

[本家のLifecycle Hooks](https://angular.io/guide/lifecycle-hooks) より、ライフサイクルメソッドには次の種類がある。

![hooks-in-sequence.png](https://qiita-image-store.s3.amazonaws.com/0/193342/13b7c36e-a9ae-15b7-f62e-223d21cb3313.png)

この図は上から順に実行される順序を示していて、それぞれを簡単に説明すると...

| ライフサイクル          | 説明                                                           | 備考                                                             |
|:------------------------|:---------------------------------------------------------------|:-----------------------------------------------------------------|
| ngOnChanges             | @Input 経由で入力値が設定されたときに実行される                | @Input は別のコンポーネントとのデータバインド示すデコレータ(注1) |
| ngOnInit                | コンポーネントの初期化時に実行される                           | 最初に ngOnChanges が実行されたあとに一度だけ実行される(注2)     |
| ngDoCheck               | コンポーネントの状態が変わったことを検知したら実行される       | いわゆる Change Detection が走るごとに実行される(注3)            |
| ngAfterContentInit      | 外部コンテンツを初期化したときに実行される                     |                                                                  |
| ngAfterContentChecked   | 外部コンテンツの変更を検知したときに実行される                 |                                                                  |
| ngAfterViewInit         | 自分自身と子コンポーネントのビューの初期化時に実行される       |                                                                  |
| ngAfterViewChecked      | 自分自身と子コンポーネントのビューが変更されたときに実行される |                                                                  |
| ngOnDestroy             | コンポーネントが破棄されるときに実行される                     |                                                                  |

* 注1
 * コンポーネントは入れ子にすることができて、@Input デコレータは別のコンポーネント(親コンポーネント)から値を受け取りたいケースで使用する
* 注2
 * [こちら](https://qiita.com/ksh-fthr/items/9f73fa161a1d7798def2#oninit) で初期化処理は ngOnInit で行うと記載したのはこれが理由
 * @Input で設定された値が処理されたあとに ngOnInit が実行されるので、もし @Input に関係する処理を初期化時に行いたい場合、コンストラクタでは早すぎる(コンストラクタでは @Input の処理が行われていない)
 * 上記より Angular コンポーネントでは初期化処理はコンストラクタではなく ngOnInit で行うのが一般的である
* 注3
 * Change Detection については [こちらの記事](https://qiita.com/lacolaco/items/523d96ddbfe55c4e6949) が詳しい

以降、各ライフサイクルメソッドについてみていく。

## ngOnChanges

ngOnChanges は 子コンポーネントで @Input デコレータで修飾されたプロパティが親コンポーネントで変更されるたびに実行される。
実際にコードを書いてみる。

### 親コンポーネント

#### テンプレート

ブラウザからの入力フォームを用意する。

```html:親コンポーネントのテンプレート(app.component.html)
 <label for="inputText">入力項目:</label>
 <input id="inputText" name="inputText" type="text" [(ngModel)]="childHogeValue" />

 <app-hoge-hoge [hogeInputValue]="childHogeValue"></app-hoge-hoge>
```

ポイントは

```html:子コンポーネントの指定とデータの受け渡し
 <app-hoge-hoge [hogeInputValue]="childHogeValue"></app-hoge-hoge>
```

で、```<app-hoge-hoge></app-hoge-hoge>``` で子コンポーネントを指定していることと、```[hogeInputValue]="childHogeValue"``` で子コンポーネントにデータを渡していること。
ここで ```hogeInputValue``` は子コンポーネントで定義されている @Input デコレータで修飾されたプロパティで、```childHogeValue``` は親コンポーネントである自分自身が定義しているプロパティである。

#### クラス
こちらはテンプレートで用意した入力フォームで入力された値を受け取るプロパティを用意するだけの単純なコードとなる。

```typescript:親コンポーネント(app.component.ts)
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  childHogeValue: String = 'initial value';
}
```

#### 注意
```app.module.ts``` で　```FormsModule``` を import していないとビルドエラーとなる。
コードでいうと以下のような感じ。

```typescript:app.module.ts
// これが必要
import { FormsModule } from '@angular/forms';

// で、imports にもセットする
@NgModule({
　　　　...
  imports: [
  　　　　...
    FormsModule
  ],
　　　　...
})
```

### 子コンポーネント

#### テンプレート
子コンポーネントであるこちらのテンプレートは親コンポーネントから渡されたデータを表示するだけ。

```html:子コンポーネントのテンプレート(hoge-hoge.component.html)
<p>
  親コンポーネントで入力された値は... {{hogeInputValue}}
</p>
```

#### クラス
ngOnChanges を確認するためのキモとなる部分。ここで親コンポーネントからデータを受け取るためのプロパティを定義し、ngOnChanges の確認を行う。

```typescript:子コンポーネント(hoge-hoge.component.ts)
import { Component, Input, OnInit, OnChanges, SimpleChanges } from '@angular/core';

@Component({
  selector: 'app-hoge-hoge',
  templateUrl: './hoge-hoge.component.html',
  styleUrls: ['./hoge-hoge.component.css']
})
export class HogeHogeComponent implements OnInit, OnChanges {

  @Input() hogeInputValue: String;

  constructor() {
    console.log('[constructor] execute');
  }

  ngOnChanges(changes: SimpleChanges) {

    console.log('[ngOnChanges] execute');
    // SimpleChanges を使って変更前の値と変更後の値、そして変更されているかをログ出力する
    for (const prop in changes) {
      const change = changes[prop];
      console.log(`${prop}: ${change.firstChange}, ${change.previousValue} => ${change.currentValue}`);
    }
  }

  ngOnInit() {
    console.log('[ngOnInit] execute');
  }
}
```

ポイントを順にみていく。
まずは

```typescript:OnChangesとSimpleChangesのimport
import { Component, Input, OnInit, OnChanges, SimpleChanges } from '@angular/core';
```

で、ライフサイクルメソッドである ngOnChanges のインターフェスを import する。
また `SimpleChanges` は ngOnChanges メソッドで **新旧のプロパティ値を取得する** ためのオブジェクトである。

次は

```typescript:@Inputデコレータを設定
  @Input() hogeInputValue: String;
```

で、こちらは親コンポーネントからデータを受け取るためのプロパティとして @Input デコレータを設定して定義している。

最後に

```typescript:ngOnChangesで状態の変化をみる
  ngOnChanges(changes: SimpleChanges) {

    console.log('[ngOnChanges] execute');
    // SimpleChanges を使って変更前の値と変更後の値、そして変更されているかをログ出力する
    for (const prop in changes) {
      const change = changes[prop];
      console.log(`${prop}: ${change.firstChange}, ${change.previousValue} => ${change.currentValue}`);
    }
  }
```

となる。
前述の SimpleChanges を引数として受け取っており、これには ```{プロパティ名:SimpleChangeオブジェクト}``` のオブジェクトがセットされて渡ってくる。
今回のコードでいうと、実行時には

```javascript:SimpleChangesにセットされたオブジェクト
{
  'hogeInputValue':SimpleChangeオブジェクト
}
```

となる。
そして SimpleChangeオブジェクト には次のプロパティがセットされている。

```javascript:SimpleChangeオブジェクトにセットされたプロパティ
{
  'currentValue': 変更後の値,
  'previousValue': 変更前の値,
  'firstChange': 最初の値設定であるかの真偽値
}
```

ngOnChanges 内の for 文ではこの SimpleChanges をループで回し、SimpleChangeオブジェクトのプロパティ値をログ出力している。


### 実行結果を確認する
実際にアプリを起動して確認した結果を示す。

* まず初期値として設定されている「initial value」が表示されている
![ngOnChanges-inital-value.JPG](https://qiita-image-store.s3.amazonaws.com/0/193342/e503b1e4-f904-8e3f-1193-ff648a096bed.jpeg)

* 次に入力ボックスに「change value」を入力
  (後述のログ出力時に見栄えをよくするため、ここで「change value」は一文字ずつの入力ではなくコピー&ペーストで入力した)
![ngOnChanges-change-value.JPG](https://qiita-image-store.s3.amazonaws.com/0/193342/454ed3df-cca8-4286-2d17-274d9cdcbd06.jpeg)

* 実行時のログ
![ngOnChanges_logs.JPG](https://qiita-image-store.s3.amazonaws.com/0/193342/d2755400-3686-bd31-962f-468fc31d7ea5.jpeg)

* 上記ログからわかること
 * アプリ起動時は **constructor -> ngOnChanges -> ngOnInit の順で実行**されている
 * 起動直後、@Input デコレータが設定された hogeInputValue は 「undefined」 から 「initial value」 に変更されている
 * 上記は最初の値設定であることから、firstChange には true がセットされている
 * 「change value」 を入力すると ngOnChanges が実行されている
 * 値は「initial value」から「change value」に変更されている
 * 2回目以降の設定となるので firstChange には false がセットされている
 
## ngOnInit
ngOnInit については 前項の ngOnChanges のログからわかるように

* constructor -> ngOnChanges のあとに実行される
* ただし一度しか実行されない

というのが全てだと思う。
つまり @Input デコレータで修飾されたプロパティに関連する初期処理を行いたい場合、

* constructor では早すぎるため、ngOnChanges か ngOnInit で行う必要がある
* だが ngOnChanges だと値が変更されると毎回実行されてしまうため初期処理にはそぐわない
* したがって、当該プロパティに関連する初期処理は ngOnInit で行う

ことになる。
で、扱うプロパティによって初期処理を constructor や ngOnInit で振り分けるのは煩雑だし気持ち悪い。
というところから、

* **コンポーネントにおける初期処理は ngOnInit で統一する**

のがシンプルで良いと思う。

## ngOnDestroy

ngOnDestroy については手抜きながら [本家のOnDestroy()](https://angular.io/guide/lifecycle-hooks#ondestroy) を抜粋させていただく。

> Put cleanup logic in ngOnDestroy(), the logic that must run before Angular destroys the directive.
>
> This is the time to notify another part of the application that the component is going away.
>
> This is the place to free resources that won't be garbage collected automatically. Unsubscribe from Observables and DOM events. Stop interval timers. Unregister all callbacks that this directive registered with global or application services. You risk memory leaks if you neglect to do so.
>
> (意訳: ngOnDestroy() では Angular がディレクティブを破壊する前に、メモリリークのリスクを回避するための処理、例えば Observables から subscribe を解除したり DOM イベントの解放、interval timers の停止、ディレクティブに登録されたコールバック処理の解除といったクリーンアップを実装する。)

## まとめにかえて
ngOnChanges のメソッドが実行される流れをみたことで、constructor から ngOnInit が実行されるまでの順序を確認できた。また ngOnInit で初期処理を記述する理由についても確認できた。
ngOnDestroy については今後活用するケースがあれば、その際に記事にしたい。

長くなったので今回はここまでで区切りとして、他のライフサクルメソッドについては別の記事で扱う。
