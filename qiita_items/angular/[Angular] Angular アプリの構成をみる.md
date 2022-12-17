Angular に関する自身の勉強の復習がてらの備忘録記事｡[こちらの記事](https://qiita.com/ksh-fthr/items/6b454264bb1f95434d42)の続き｡
前回で Angular CLI で作成したプロジェクト構成について簡単に見たので､今回は

* src/
 * index.html
 * main.ts
* src/app
 * app.component.css
 * app.component.html
 * app.component.ts
 * app.module.ts

の中身を確認し､Angular アプリ(Anguar で作成したアプリケーション)の構成について見ていく｡
的はずれだったり間違った情報や記載があればご指摘頂けるとありがたい｡

## この記事を実施した環境

* Windows10 Home 64bit
* node v8.2.1
* npm v4.0.5
* angular/cli v1.2.6
* Angular v4.3.2

## 前提条件

* Angular CLI でプロジェクトを作成している
* プロジェクト作成直後の状態である

では次項から上であげた各ファイルをみていく。

## src/index.html
まずは ```index.html``` から見ていく｡
```index.html``` は Angular アプリを起動するためのトップページで､プロジェクト作成直後は次のコードとなっている｡

```html:index.html全体
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>MyApp</title>
  <base href="/">

  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" type="image/x-icon" href="favicon.ico">
</head>
<body>
  <app-root></app-root>
</body>
</html>
```

このコードで注目するポイントは次の部分｡

```html:index.htmlの注目するポイント
  <app-root></app-root>
```

```app-root```のタグは Angular アプリのルートコンポーネントを示しており､これは後述する ```app.component.ts```  で定義されたタグとリンクしている｡
つまり Angular CLI でプロジェクトを作成した直後の状態で Angular アプリを起動すると､

1. まず index.html がトップページとして読み込まれる
1. 次にここで指定された ```app-root``` タグで指定されたコンポーネントとして ```app.component.ts``` が読み込まれる

流れとなる｡

で､じゃあ ｢```app-root``` は固定値として必ず指定しなければならないのか｣ というとそうではなく､ここで指定するタグは別に ```hoge``` でも ```bar``` でもなんでも良い｡
乱暴な言い方になるが､要は **読み込みたいコンポーネントで定義しているタグと index.html で指定するタグが一致すれば良い** ということになる｡

## src/main.ts
```main.ts``` は Angular アプリを起動するためのスタートアップコードであり､中身は次の通り｡

```typescript:main.ts全体
import { enableProdMode } from '@angular/core';
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';

import { AppModule } from './app/app.module';
import { environment } from './environments/environment';

if (environment.production) {
  enableProdMode();
}

platformBrowserDynamic().bootstrapModule(AppModule);
```

ポイントは3つで

* import 文のブロック
* if 文のブロック
* 最終行

となる。以下､順に見ていく｡

### import 文のブロック

```typescript:main.tsのimport文のブロック
import { enableProdMode } from '@angular/core';
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';

import { AppModule } from './app/app.module';
import { environment } from './environments/environment';
```

impot 文で アプリに必要なモジュールをインポートする｡

* enableProdMode
  * 本番環境(product)用でアプリを起動する際に実行される
  * 後述の if 文によるチェックで実行の是非が決まる

* platformBrowserDynamic
  * ブラウザでアプリを起動するために用意された Angular 標準のモジュール

* AppModule
  * Angular アプリの本体となるモジュール
  * 後述の app.module.ts で定義されている

* environment
  * 環境設定ファイル
  * ここで定義されている ```production``` の値を後述の if文 でチェックする
  * true ならば本番環境､false ならば開発環境

### if 文のブロック

```typescript:main.tsのif文のブロック
if (environment.production) {
  enableProdMode();
}
```

アプリケーションを本番環境用に起動するか､開発環境用に起動するかをチェックする｡
チェック内容は前述の ```environment``` で定義されている ```production``` の真偽値の確認で、ここで「真」ならば本番環境(production)として判断し、本番環境としてデプロイするべく ```enableProdMode()``` が実行される。

### 最終行

```typescript:main.tsの最終行
platformBrowserDynamic().bootstrapModule(AppModule);
```

この行が実行されることで Angular アプリが起動する｡このとき､Angular アプリとして起動したいモジュールを ```platformBrowserDynamic().bootstrapModule()``` の引数に指定する｡
ここでは ```AppModule```(実体は src/app/app.module.ts) を指定して実行している｡

## src/app/app.module.ts

```app.module.ts``` は Angular アプリのルートモジュール｡
プロジェクト作成直後のコードは次の通り｡

```typescript:app.module.ts全体
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

こちらもポイントは3つ｡
それぞれ

* import 文のブロック
* @NgModule のブロック
* クラス定義

で､順に見ていく｡

### import 文のブロック

```typescript:app.module.tsのimport文のブロック
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppComponent } from './app.component';
```

import 文で アプリに必要なモジュールをインポートする｡
各モジュールの簡単な説明は以下の通り。

* BrowserModule
  * Angular が提供する標準モジュール
  * アプリをブラウザ上で動作させるためのモジュール
* NgModule
  * Angular が提供する標準モジュール
  * モジュールを定義するためのモジュール
* AppComponent
  * Angular アプリの実体となるルートコンポーネント
  * 実体は src/app/app.component.ts

### @NgModule のブロック

```typescript:app.module.tsの@NgModuleブロック
@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
```

@NgModule は後述の export 文で外部公開の宣言で定義したクラスの **モジュールの構成情報** を宣言するデコレーター｡
デコレーターは Java のアノテーションに相当し､構成情報を付与するための仕組み｡
各要素の説明は次の通り。

* declarations
 * 現在のモジュールに属するコンポーネントを宣言
* imports
 * ここで指定するモジュールは前述の import 文でインポートしていることが前提
 * 現在のモジュールで使用する他のモジュールを宣言
* providers
 * 依存性注入
 * 利用したいサービス(サービスについては本記事では触れない)を指定する
 * ここで指定するサービスは import 文でインポートしていることが前提
 * 指定したサービスはモジュール全体でアクセス可能となる
* bootstrap
 * ここで指定するコンポーネントは前述の import 文でインポートしていることが前提
 * アプリで最初に起動するコンポーネント(ルートコンポーネント)を指定する

### クラス定義

```typescript:app.module.tsのクラス定義
export class AppModule { }
```

モジュールクラスを定義し､且つ外部公開を宣言することで､他のクラスから参照することが可能となる｡
前述の @NgModule で本モジュールの構成情報が付与されている｡

## src/app/app.component.*
最後に app.component.* として html/css/ts の各ファイルを見る｡
各ファイルは

* app.component.html
 * コンポーネントのテンプレート
* app.component.css
 * コンポーネントのデザイン
* app.component.ts
 * コンポーネントのロジック

を担当している｡

### app.component.html

```html:app.component.html全体
<!--The content below is only a placeholder and can be replaced.-->
<div style="text-align:center">
  <h1>
    Welcome to {{title}}!
  </h1>
  <img width="300" src="data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz4NCjwhLS0gR2VuZXJhdG9yOiBBZG9iZSBJbGx1c3RyYXRvciAxOS4xLjAsIFNWRyBFeHBvcnQgUGx1Zy1JbiAuIFNWRyBWZXJzaW9uOiA2LjAwIEJ1aWxkIDApICAtLT4NCjxzdmcgdmVyc2lvbj0iMS4xIiBpZD0iTGF5ZXJfMSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxuczp4bGluaz0iaHR0cDovL3d3dy53My5vcmcvMTk5OS94bGluayIgeD0iMHB4IiB5PSIwcHgiDQoJIHZpZXdCb3g9IjAgMCAyNTAgMjUwIiBzdHlsZT0iZW5hYmxlLWJhY2tncm91bmQ6bmV3IDAgMCAyNTAgMjUwOyIgeG1sOnNwYWNlPSJwcmVzZXJ2ZSI+DQo8c3R5bGUgdHlwZT0idGV4dC9jc3MiPg0KCS5zdDB7ZmlsbDojREQwMDMxO30NCgkuc3Qxe2ZpbGw6I0MzMDAyRjt9DQoJLnN0MntmaWxsOiNGRkZGRkY7fQ0KPC9zdHlsZT4NCjxnPg0KCTxwb2x5Z29uIGNsYXNzPSJzdDAiIHBvaW50cz0iMTI1LDMwIDEyNSwzMCAxMjUsMzAgMzEuOSw2My4yIDQ2LjEsMTg2LjMgMTI1LDIzMCAxMjUsMjMwIDEyNSwyMzAgMjAzLjksMTg2LjMgMjE4LjEsNjMuMiAJIi8+DQoJPHBvbHlnb24gY2xhc3M9InN0MSIgcG9pbnRzPSIxMjUsMzAgMTI1LDUyLjIgMTI1LDUyLjEgMTI1LDE1My40IDEyNSwxNTMuNCAxMjUsMjMwIDEyNSwyMzAgMjAzLjksMTg2LjMgMjE4LjEsNjMuMiAxMjUsMzAgCSIvPg0KCTxwYXRoIGNsYXNzPSJzdDIiIGQ9Ik0xMjUsNTIuMUw2Ni44LDE4Mi42aDBoMjEuN2gwbDExLjctMjkuMmg0OS40bDExLjcsMjkuMmgwaDIxLjdoMEwxMjUsNTIuMUwxMjUsNTIuMUwxMjUsNTIuMUwxMjUsNTIuMQ0KCQlMMTI1LDUyLjF6IE0xNDIsMTM1LjRIMTA4bDE3LTQwLjlMMTQyLDEzNS40eiIvPg0KPC9nPg0KPC9zdmc+DQo=">
</div>
<h2>Here are some links to help you start: </h2>
<ul>
  <li>
    <h2><a target="_blank" href="https://angular.io/tutorial">Tour of Heroes</a></h2>
  </li>
  <li>
    <h2><a target="_blank" href="https://github.com/angular/angular-cli/wiki">CLI Documentation</a></h2>
  </li>
  <li>
    <h2><a target="_blank" href="http://angularjs.blogspot.com/">Angular blog</a></h2>
  </li>
</ul>
```

ポイントは ```h1タグ``` 内の ```{{title}}``` ｡

```html:app.component.htmlのh1タグ
  <h1>
    Welcome to {{title}}!
  </h1>
```

```{{title}}``` は **片方向データバインド** と呼ばれる仕組みを表している｡
```{{}}``` で囲まれた文字列は後述の ```app.component.ts``` で宣言されたプロパティとリンクしており､ ```tsファイル``` でセットされた値が表示される｡
この場合 ```app.component.ts``` で ```app``` という文字列がセットされているので､ブラウザ上では ```Welcome to app!``` と表示される｡

### app.component.css

```css:app.component.css全体
/* 空(このコメントは本記事用に記載) */
```

今回は空ファイルだが､ここで定義したデザインがテンプレートである ```app.component.html``` に反映される｡
前記事で少し触れたとおり､ここで定義したデザインは原則として定義したコンポーネントにのみ適用され､他のコンポーネントで定義されたスタイルと競合･衝突することは無い｡

### app.component.ts

```typescript:app.component.ts全体
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'app';
}
```

ポイントは次の3つ｡

* import 文
* @Component ブロック
* クラス定義


それぞれを順に見ていく｡

#### import 文

```typescript:app.component.tsのimport文
import { Component } from '@angular/core';
```

* Component
 * Angular が提供する標準モジュール
 * コンポーネントを定義するために必要なモジュール


#### @Component ブロック

```typescript:app.component.tsの@Componentブロック
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
```

@Component は後述の export 文で外部公開の宣言で定義したクラスの **コンポーネントの構成情報** を宣言するデコレーター｡
各構成要素の意味は次の通り｡

* selector
 * 本コンポーネントが外部から参照される際に指定されるタグを定義する
 * ここで定義されている ```app-root``` は ```index.html``` で指定され､読み込まれる
* templateUrl
 * 本コンポーネントの html テンプレートファイルのファイルパスを指定する
* styleUrls
 * 本コンポーネントの css ファイルパスを配列で指定する

#### クラス定義

```typescript:app.component.tsのクラス定義
export class AppComponent {
  title = 'app';
}
```

このコンポーネントの本体で、```app.module.ts``` で宣言した ```AppComponent``` コンポーネントの実体である｡
コンポーネントクラスを定義し､且つ外部公開を宣言することで､他のクラスからの参照が可能となる｡
説明が前後するが､コンポーネントとして扱われるために､前述の ```@Component``` デコレータでコンポーネントの構成情報を宣言しておく｡
また本コンポーネントの html テンプレートである ```app.component.html``` で扱う片方向データバインドのためのプロパティ ```title``` の宣言と値のセットを行っている｡

## 終わりに
以上､[前回の記事の｢終わりに｣](https://qiita.com/ksh-fthr/items/6b454264bb1f95434d42#%E7%B5%82%E3%82%8F%E3%82%8A%E3%81%AB) で示した各ファイルについて簡単にみてきた｡
ここでみてきたのは Angular CLI でプロジェクトを作成した直後の状態だが、私見ながら、Angular アプリについて次のことが言えると思う。

* Angular アプリはモジュールを作成して起動する
* モジュールはコンポーネントによって形成される
* コンポーネントは以下の3つによって形成される
 * テンプレート(*.html)
 * デザイン(*.css)
 * ロジック(*.ts)

本記事では Angular アプリを構成するファイルの構成と各ファイルの内容を見ただけで､実際に開発を進めていく点については触れなかった｡
例えば、Angular がフロントエンドのフレームワークである以上バックエンドとのやり取りは発生するし、コンポーネントを複数作成したケースにおいて、各コンポーネント間でデータを共有したい、といった要求も発生するだろう。
そういったケースについても今後の記事で触れていきたいが、とりあえずこの次はコンポーネントの開発について触れてみたい。
