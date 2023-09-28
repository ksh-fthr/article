[こちらの記事](https://qiita.com/ksh-fthr/items/d040cf8b2d15bd7e507d) で Angular CLI で生成した Angular アプリの構成をみた。
で、Angualr アプリはコンポーネントを作ること、ということで、今回は Component の開発についてみていく。

## Angular CLI でコンポーネントの雛形を作る

Angular CLI を使うと、以下のコマンドを実行することでコマンドラインから雛形を作ることができる。

### コンポーネント生成のコマンド

```:コンポーネントの雛形を作るコマンド(例1)
$ ng generate component hoge-hoge
installing component
  create src/app/hoge-hoge/hoge-hoge.component.css
  create src/app/hoge-hoge/hoge-hoge.component.html
  create src/app/hoge-hoge/hoge-hoge.component.spec.ts
  create src/app/hoge-hoge/hoge-hoge.component.ts
  update src/app/app.module.ts
```

```:コンポーネントの雛形を作るコマンド(例2)
$ ng g component PiyoPiyo
installing component
  create src/app/piyo-piyo/piyo-piyo.component.css
  create src/app/piyo-piyo/piyo-piyo.component.html
  create src/app/piyo-piyo/piyo-piyo.component.spec.ts
  create src/app/piyo-piyo/piyo-piyo.component.ts
  update src/app/app.module.ts
```

```:コンポーネントの雛形を作るコマンド(例3)
$ ng g component fooBar
installing component
  create src/app/foo-bar/foo-bar.component.css
  create src/app/foo-bar/foo-bar.component.html
  create src/app/foo-bar/foo-bar.component.spec.ts
  create src/app/foo-bar/foo-bar.component.ts
  update src/app/app.module.ts
```

```:コンポーネントの雛形を作るコマンド(例4)
$ ng g component bar_bar
installing component
  create src/app/bar-bar/bar-bar.component.css
  create src/app/bar-bar/bar-bar.component.html
  create src/app/bar-bar/bar-bar.component.spec.ts
  create src/app/bar-bar/bar-bar.component.ts
  update src/app/app.module.ts
```

* ポイント
 * ```gengerate``` は ```g``` と省略することもできる
 * コンポーネント名はハイフン区切り、キャメルケース(Upper/Lower)、スネークケース のどれで指定しても良い
 * コンポーネントはハイフンで区切られたファイル名で生成される
 * コンポーネントは ```src/app/``` 配下にファイル名と同名のディレクトリを生成した上でその下に配置される
 * ```*.css, *.html, *.spec.ts, *.ts``` の4ファイルが生成される(```*.spec.ts``` はテストコード)
 * src/app/app.module.ts が更新されている

### *.ts ファイルをみる

 生成したコンポーネントをみると次のようになっている。
 ここでは上記で生成した4つのコンポーネントのうち一つだけをみるが、他のコンポーネントも同じである。

```typescript:hoge-hoge.component.ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-hoge-hoge',
  templateUrl: './hoge-hoge.component.html',
  styleUrls: ['./hoge-hoge.component.css']
})
export class HogeHogeComponent implements OnInit {

  constructor() { }

  ngOnInit() {
  }

}
```

```html:hoge-hoge.component.html
<p>
  hoge-hoge works!
</p>
```

* ポイント
 * OnInit が import されている
  * OnInit はライフサイクル(後述)をフックするインターフェース
 * @Component デコレータの selector には、文字列「app」+ ハイフン区切りでテンプレートのタグが定義されている
 * クラス名は Upper キャメルケース + 文字列「Component」が付与されて定義されている

### src/app/app.module.ts の更新内容をみる

Angular CLI でコンポーネントを生成すると、前掲のとおり src/app/app.module.ts が自動で更新される。
その中身をみてみると

```typescript:src/app/app.module.ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppComponent } from './app.component';

// 生成した4つのコンポーネントが追加されている
import { HogeHogeComponent } from './hoge-hoge/hoge-hoge.component';
import { PiyoPiyoComponent } from './piyo-piyo/piyo-piyo.component';
import { FooBarComponent } from './foo-bar/foo-bar.component';
import { BarBarComponent } from './bar-bar/bar-bar.component';

@NgModule({
  declarations: [
    AppComponent,
    // 生成した4つのコンポーネントが追加されている
    HogeHogeComponent,
    PiyoPiyoComponent,
    FooBarComponent,
    BarBarComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

となっている。
生成した4つのコンポーネントが import 文に追加されていること。また @NgModule の declarations に、本モジュールに属するコンポーネントとして、それらが追加で宣言されていることがわかる。

## 生成したコンポーネントを呼び出す

ここまででコンポーネントの生成と生成されたコンポーネントの定義や src/app/app.module.ts の更新内容をみてきたが、この状態ではまだアプリを起動しても生成したコンポーネントはアプリの中で呼び出されない。

* ここまでの状態でアプリを確認
    ![no-call-added-component.png](https://qiita-image-store.s3.amazonaws.com/0/193342/1e08bb88-a2a1-7d2e-4970-b046df8682c4.png)


* テンプレートを編集
コンポーネントをアプリに追加するには次のように src/app/app.component.html を編集する。

    ```html:src/app/app.component.html
    <!--The content below is only a placeholder and can be replaced.-->
    <div style="text-align:center">
      <h1>
        Welcome to {{title}}!
      </h1>
      <img width="300" 
src="data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz4NCjwhLS0gR2VuZXJhdG9yOiBBZG9iZSBJbGx1c3RyYXRvciAxOS4xLjAsIFNWRyBFeHBvcnQgUGx1Zy1JbiAuIFNWRyBWZXJzaW9uOiA2LjAwIEJ1aWxkIDApICAtLT4NCjxzdmcgdmVyc2lvbj0iMS4xIiBpZD0iTGF5ZXJfMSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxuczp4bGluaz0iaHR0cDovL3d3dy53My5vcmcvMTk5OS94bGluayIgeD0iMHB4IiB5PSIwcHgiDQoJIHZpZXdCb3g9IjAgMCAyNTAgMjUwIiBzdHlsZT0iZW5hYmxlLWJhY2tncm91bmQ6bmV3IDAgMCAyNTAgMjUwOyIgeG1sOnNwYWNlPSJwcmVzZXJ2ZSI+DQo8c3R5bGUgdHlwZT0idGV4dC9jc3MiPg0KCS5zdDB7ZmlsbDojREQwMDMxO30NCgkuc3Qxe2ZpbGw6I0MzMDAyRjt9DQoJLnN0MntmaWxsOiNGRkZGRkY7fQ0KPC9zdHlsZT4NCjxnPg0KCTxwb2x5Z29uIGNsYXNzPSJzdDAiIHBvaW50cz0iMTI1LDMwIDEyNSwzMCAxMjUsMzAgMzEuOSw2My4yIDQ2LjEsMTg2LjMgMTI1LDIzMCAxMjUsMjMwIDEyNSwyMzAgMjAzLjksMTg2LjMgMjE4LjEsNjMuMiAJIi8+DQoJPHBvbHlnb24gY2xhc3M9InN0MSIgcG9pbnRzPSIxMjUsMzAgMTI1LDUyLjIgMTI1LDUyLjEgMTI1LDE1My40IDEyNSwxNTMuNCAxMjUsMjMwIDEyNSwyMzAgMjAzLjksMTg2LjMgMjE4LjEsNjMuMiAxMjUsMzAgCSIvPg0KCTxwYXRoIGNsYXNzPSJzdDIiIGQ9Ik0xMjUsNTIuMUw2Ni44LDE4Mi42aDBoMjEuN2gwbDExLjctMjkuMmg0OS40bDExLjcsMjkuMmgwaDIxLjdoMEwxMjUsNTIuMUwxMjUsNTIuMUwxMjUsNTIuMUwxMjUsNTIuMQ0KCQlMMTI1LDUyLjF6IE0xNDIsMTM1LjRIMTA4bDE3LTQwLjlMMTQyLDEzNS40eiIvPg0KPC9nPg0KPC9zdmc+DQo=">
    </div>

    <!-- もともとはここに各種リンクがあったが邪魔なので削除 -->
    <!-- 生成した4つのコンポーネントを追加する -->
    <app-hoge-hoge></app-hoge-hoge>
    <app-piyo-piyo></app-piyo-piyo>
    <app-foo-bar></app-foo-bar>
    <app-bar-bar></app-bar-bar>
    ```

* 編集後にアプリを再度確認
この状態でアプリをみると、次の通り追加したコンポーネントの内容が表示されている。
    ![call-added-component.png](https://qiita-image-store.s3.amazonaws.com/0/193342/9b30f1ee-7e77-a386-f640-aa768628b7a7.png)
  (オレンジの枠線内が生成した4つのコンポーネント)

## ライフサイクルという概念

話が前後するが、前述のとおり ```*.ts``` ファイルには OnInit というインターフェースが import されていて、クラス定義ではそれを implements し、ngOnInit メソッドの実装を行っている。

コンポーネントには**ライフサイクル**という概念があって、OnInit はその中の一つである。

ライフサイクルについての詳細は別記事に書くとして、ここでは OnInit について簡単な説明を。

### OnInit

OnInit についての簡単な説明を箇条書きで記すと

* ライフサイクルの一つで TypeScript のインターフェース
* コンポーネントの初期化処理を担当する
* ngOnInit というメソッドが宣言されていて、OnInit を implements したクラスは ngOnInit メソッドの実装を強制される
* ngOnInit が実行されるタイミングはコンストラクタのあと

となる。
で、当然「では初期化処理はコンストラクタと ngOnInit とどちらで行うのが良いのか」となるが、一般的に Angular アプリでは **ngOnInit で初期化処理** を行う。
理由は他のライフサイクルが絡んでくるのでここでは割愛する。

## まとめ

長くなったので一旦ここまでを区切りとして、本記事のまとめを。

* Angular CLI では ```ng generate component コンポーネント名``` でコンポーネントを生成できる
* コンポーネントは
 * ハイフン区切りのファイル名で生成される
 * クラス定義は「Upper キャメルケース + Component」で定義される
 * テンプレートのタグは「app + ハイフン区切り」で定義される
 * src/app/app.module.ts は自動で更新され、生成したコンポーネントが追加される
 * ただしコンポーネントをアプリ上で呼び出すには src/app/app.component.html でタグを記載する必要がある
* コンポーネントには*ライフサイクル*という概念があり、Angular CLI で生成したコンポーネントにはデフォルトで OnInit インターフェースの実装として ngOnInit メソッドを定義している
* Angular アプリでは一般的に初期化処理を ngOnInit で実装する

次はライフサイクルについてみていこうと思う。
