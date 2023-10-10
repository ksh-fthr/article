# はじめに
Angular には URL の移動を伴う画面遷移の手段として ルーティング という仕組みが用意されている。
本記事では簡単な例を元にルーティングの基本的な使い方についてみていく。


# 更新情報

## 2021/01/11
- 記事内で扱ったコードを Angular `v11.0.5` で確認しました

# 作業環境

| 環境                                          | バージョン          | 備考               |
| --------------------------------------------- | ------------------- | ------------------ |
| [Angular CLI](https://cli.angular.io/)        | ~~v6.1.3~~ v11.0.5  | `$ ng --version`   |
| [Angular](https://angular.io/)                | ~~v6.1.2~~ v11.0.5  | 同上               |
| [TypeScript](https://www.typescriptlang.org/) | v4.0.2              | 同上               |
| [Node.js](https://nodejs.org/ja/)             | ~~v9.4.0~~ v12.18.3 | `$ node --version` |
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


# ルーティングによる画面遷移1

冒頭に記載のとおり、Angular では URL の移動を伴う画面遷移をルーティングによって実現するが、そのために [@angular/router](https://angular.jp/api/router) から提供される

* [Routes](https://angular.jp/api/router/Routes)
* [RouterModule](https://angular.jp/api/router/RouterModule)
* [RouterOutlet](https://angular.io/api/router/RouterOutlet)
* [RouterLink](https://angular.jp/api/router/RouterLink)

を利用する。
実際にコードを見た方がわかりやすいと思うので、これらを利用した簡単なサンプルコードを挙げる。


## サンプルコード1

```typescript:app.module.ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

// ルーティングによる画面遷移のために必要なモジュール
import { RouterModule, Routes } from '@angular/router';

import { AppComponent } from './app.component';
import { PageAComponent } from './component/page-a/page-a.component';
import { PageBComponent } from './component/page-b/page-b.component';
import { PageCComponent } from './component/page-c/page-c.component';

// Routing を行う対象のコンポーネントを管理する
// path にセットした文字列にマッチしたURLが指定されると、対になっているコンポーネントが表示される
// 下記のように明示する以外にも
//    '' で [/] のルートパスを指定できる
//    '**' でワイルドカードを指定できる
const ROUTE_TABLE: Routes = [
  { path: 'page-a', component: PageAComponent },
  { path: 'page-b', component: PageBComponent },
  { path: 'page-c', component: PageCComponent }
];

@NgModule({
  declarations: [
    AppComponent,
    PageAComponent,
    PageBComponent,
    PageCComponent,
  ],
  imports: [
    BrowserModule,
    RouterModule.forRoot(ROUTE_TABLE), // 追加. routing の情報を登録する
  ],
  providers: [
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

```html:app.component.html
<div>
  <div class="block-header">
    <p>Routing によるページ切り替えの動作確認を行う</p>
  </div>

  <div class="block-body">
    <ul>
      <!-- routing のためのリンク. routerLink に app.module.ts で定義したテーブル: ROUTE_TABLE に登録した path をセットする -->
      <li><a routerLink="/page-a">Page-Aに遷移</a></li>
      <li><a routerLink="/page-b">Page-Bに遷移</a></li>
      <li><a routerLink="/page-c">Page-Cに遷移</a></li>
    </ul>
  </div>
</div>

<!-- routing で呼び出されたページは router-outlet で確保された領域に反映される -->
<router-outlet></router-outlet>
```

```html:page-a.component.html
<p>
  page-a works!
</p>
```

```html:page-b.component.html
<p>
  page-b works!
</p>
```

```html:page-c.component.html
<p>
  page-c works!
</p>
```


## 説明

コード中のコメントが全てなのだが、補足すると

* app.module.ts
    * ```ROUTE_TABLE``` に画面遷移の対象となるコンポーネントと path のオブジェクトを登録する -> **ルーティング情報の作成**
    * 作成したルーティング情報を ```RouterModule.forRoot``` でアプリに登録する
* app.component.html
    * ```<router-outlet>``` タグは ```RouterOutlet``` で提供されているセレクタで、```routerLink``` は ```RouterLink``` から提供されているセレクタ
    * **```<router-outlet>``` セットした領域に ```routerLink```に指定したコンポーネントのテンプレート内容が表示される**
    * ```routerLink``` にセットする URL は、 **```ROUTE_TABLE``` にセットした path と同じ** でなければならない
* 他のテンプレート
    * 文字列を表示するだけの単純なもの

となる。
で、```ROUTE_TABLE``` についてもう少し補足を。

* ROUTE_TABLE の補足

    ```ROUTE_TABLE``` で path　を登録する際、以下の動きとなるので覚えておくとデフォルトページやエラーページの表示に役立つと思う。

    * ```''```　をセットすれば URL のルートパスで表示するコンポーネントを登録できる
    * ```**``` をセットすると、どの URL にも当てはまらない場合に表示するコンポーネントを登録できる

さて、上記のサンプルを実行すると


## 実行結果

* Page-Aに遷移をクリックした
    * ![routing-01.png](https://qiita-image-store.s3.amazonaws.com/0/193342/cc388278-6af8-1808-be30-9ec801228623.png)

* Page-Bに遷移をクリックした
    * ![routing-02.png](https://qiita-image-store.s3.amazonaws.com/0/193342/9a664a4a-1681-ac4e-ff64-dd493485d3e1.png)

* Page-Cに遷移をクリックした
    * ![routing-03.png](https://qiita-image-store.s3.amazonaws.com/0/193342/63cfa471-6c77-c7b7-c5e1-f2ed770e18c1.png)


となる。
これで各コンポーネントへのリンクをクリックすることで

* URL が移動していること
* 表示される画面が切り替わっていること

が確認できた。


# ルーティングによる画面遷移2

上記の例では URL の移動による画面遷移は確認できたものの ```app.component.html``` でセットした ```routerLink``` によって画面が切り替わる「サイトナビゲーション」のような実装となっていた。

以下に示すサンプルでは、```app.component.html``` に ```routerLink``` をセットするのではなく、各画面にセットすることで、画面そのものの切り替えを見てみる。

## サンプルコード2

```typescript:app.module.ts
// 前掲のコードとの差分だけ

// Routing を行う対象のコンポーネントを管理する
// path にセットした文字列にマッチしたURLが指定されると、対になっているコンポーネントが表示される
// 下記のように明示する以外にも
//    '' で [/] のルートパスを指定できる
//    '**' でワイルドカードを指定できる
const ROUTE_TABLE: Routes = [
  // [/] パス指定時は page-a を表示する
  { path: '', component: PageAComponent },
  { path: 'page-a', component: PageAComponent },
  { path: 'page-b', component: PageBComponent },
  { path: 'page-c', component: PageCComponent }
];
```

```html:app.component.html
<!-- routing で呼び出されたページは router-outlet で確保された領域に反映される -->
<router-outlet></router-outlet>
```

```html:page-a.component.html
<div>
  <div class="block-header">
    <p>Routing によるページ切り替えの動作確認を行う</p>
  </div>

  <div class="block-body">
    <ul>
      <!-- routing のためのリンク. routerLink に app.module.ts で定義したテーブル: ROUTE_TABLE に登録した path をセットする -->
      <li><a routerLink="/page-b">Page-Bに遷移</a></li>
      <li><a routerLink="/page-c">Page-Cに遷移</a></li>
    </ul>
  </div>
</div>
<p>
  page-a works!
</p>
```

```html:page-b.component.html
<div>
  <div class="block-header">
    <p>Routing によるページ切り替えの動作確認を行う</p>
  </div>

  <div class="block-body">
    <ul>
      <!-- routing のためのリンク. routerLink に app.module.ts で定義したテーブル: ROUTE_TABLE に登録した path をセットする -->
      <li><a routerLink="/page-a">Page-Aに遷移</a></li>
      <li><a routerLink="/page-c">Page-Cに遷移</a></li>
    </ul>
  </div>
</div>
<p>
  page-b works!
</p>
```

```html:page-c.component.html
<div>
  <div class="block-header">
    <p>Routing によるページ切り替えの動作確認を行う</p>
  </div>

  <div class="block-body">
    <ul>
      <!-- routing のためのリンク. routerLink に app.module.ts で定義したテーブル: ROUTE_TABLE に登録した path をセットする -->
      <li><a routerLink="/page-a">Page-Aに遷移</a></li>
      <li><a routerLink="/page-b">Page-Bに遷移</a></li>
    </ul>
  </div>
</div>
<p>
  page-c works!
</p>
```


## 説明

このサンプルでのポイントは 3つ。

* ```app.module.ts``` で定義している ```ROUTE_TABLE``` で、ルートパス( [```/```] ) で表示するコンポーネントに ```page-a.component``` を指定している
* ```app.component.html``` には ```<router-outlet>``` しかセットしていない
* ```page-a```, ```page-b```, ```page-c``` に各コンポートへの ```routerLink``` をセットしている

こうすることで、```app.component.html``` には

* アプリ起動時は ```page-a.component.html``` の内容が表示される
* リンククリックされる毎に該当するコンポーネントのビューだけが表示される

動きとなる。


## 実行結果

* アプリ起動直後
    * <img width="520" alt="routing-04.png" src="https://qiita-image-store.s3.amazonaws.com/0/193342/7971728f-be69-5591-bcc9-430f2e5aa6b3.png">

* Page-Bに遷移をクリックした
    * <img width="520" alt="routing-05.png" src="https://qiita-image-store.s3.amazonaws.com/0/193342/94e4a6b2-05b8-6041-7ae3-dc7964efff5a.png">

* Page-Cに遷移をクリックした
    * <img width="520" alt="routing-06.png" src="https://qiita-image-store.s3.amazonaws.com/0/193342/615ef62c-f603-d9e5-813c-e07b219055fe.png">

* Page-Aに遷移をクリックした
    * <img width="521" alt="routing-07.png" src="https://qiita-image-store.s3.amazonaws.com/0/193342/2fd5f971-f7ae-3478-e395-360f9e40eae0.png">


以上の動きから

* 起動直後はルートパスに指定した ```page-a.component.html``` が表示されること
* 各画面のリンクに応じて画面そのものが切り替わっていること

が確認できた。


# 参考

* [@angular/router](https://angular.jp/api/router)
* [Routes](https://angular.jp/api/router/Routes)
* [RouterModule](https://angular.jp/api/router/RouterModule)
* [RouterOutlet](https://angular.io/api/router/RouterOutlet)
* [RouterLink](https://angular.jp/api/router/RouterLink)


# ソースコード

今回の記事で作成したコードは以下にアップしてあるのでご参考まで。

* [ルーティングによる画面遷移1のコード](https://github.com/ksh-fthr/angular-work/tree/feat_routing)
* [ルーティングによる画面遷移2のコード](https://github.com/ksh-fthr/angular-work/tree/feat_routing_another)
