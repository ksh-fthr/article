
# はじめに
Angular でタブによるコンポーネントの切り替えを実装したい。
要件は

* なるべくタブの管理を抽象化したい
* つまり 「```ngIf``` とかでコンポーネントを囲って切り替える」といったことはしたくない

で、これを実現するために [NgComponentOutlet](https://angular.jp/api/common/NgComponentOutlet) を使ってタブを実装する。
```NgComponentOutlet``` の特徴は **コンポーネントを動的に切り替えることができる** ことで、今回これで得られるメリットとしては

* メリット
  * 要件にある「タブ切り替えを ```ngIf``` を使って実装しない」を満たせる
  * 上記が満たせるということは、タブが増えるたびに ```ngIf``` を増やさなくて済む
  * 関連して、タブ切り替えのための HTML タグも単純化できる

逆にデメリットは

* デメリット
  * ```NgComponentOutlet``` は普段使うことのないであろう( と勝手に思っている )ディレクティブなのでとっつきづらい
  * 抽象化することで、パッと見ただけでは「なにをしているのか」を把握するのが若干難しくなる

といった点が挙げられる。

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

# 本記事で扱う構成

作成にあたってのアプリの構成は次の通り。

```bash:アプリの構成
src/
  └ app/
    ├ component/
    │    ├ switch-tab/                       # タブによて読み込むコンポーネントを動的に切り替える
    │    │    ├ switch-tab.component.css
    │    │    ├ switch-tab.component.html
    │    │    └ switch-tab.component.ts
    │    ├ tab-a/                            # タブとして扱われるコンポーネント-A
    │    │    ├ tab-a.component.css
    │    │    ├ tab-a.component.html
    │    │    └ tab-a.component.ts
    │    └ tab-b/                            # タブとして扱われるコンポーネント-B
    │         ├ tab-b.component.css
    │         ├ tab-b.component.html
    │         └ tab-b.component.ts
    ├ model/                                 # switch-tab.service で管理する配列の型となるモデル
    │    └ tab-model.ts
    ├ service/                               # タブを管理するサービス
    │    └ switch-tab.service.ts
    ├ app.component.css
    ├ app.component.html
    ├ app.component.ts
    └ app.module.ts
```


# 実現したいこと

冒頭に書いた通り **コンポーネントの切り替えをタブで行う** こと。
つまり次の図を実現したい。

![switch_tab_01.png](https://qiita-image-store.s3.amazonaws.com/0/193342/18387327-4b5f-d077-32ba-5b5c6e2910b1.png)

* Tab-A, Tab-B はコンポーネント
* タブをクリックすることで Tab-A と Tab-B が切り替わる


# 実現にあたって

大まかな実装方針は下記の通り。

* 選択されたタブのコンポーネントは ```ngComponentOutlet``` を使って動的に読み込む
  * [switch-tab.component.html](#switch-tabcomponenthtml) で実装
* タブは ```<ul>タグ``` と ```<li>タグ``` で実現する
  * [switch-tab.component.html](#switch-tabcomponenthtml) と [switch-tab.component.css](#switch-tabcomponentcss) で実装
* タブが増えるたびに ```<li>タグ``` を記述することは避けたいので、ここは ```ngFor``` ディレクティブを使う
  * [switch-tab.component.html](#switch-tabcomponenthtml) で実装
* 上記を満たすためにタブとして扱われるコンポーネントは配列で管理する
  * [switch-tab.component.ts](#switch-tabcomponentts), [switch-tab.service](#switch-tabservice) で実装
* 配列で管理するにあたり、配列の型を明示するためのモデルクラスを用意する
  * [tab-model](#tab-model) で実装


# NgComponentOutlet の使い方

さて、実際の実装の前にまずは ```NgComponentOutlet``` の使い方について確認を｡
( 少々しつこいが ) ```NgComponentOutlet``` は動的にコンポーネントを読み込むことができるディレクティブで､テンプレートで ```ng-container``` と共に使用する｡


## NgComponentOutletを使うテンプレート

```html:NgComponentOutletを使うテンプレート
<ng-container *ngComponentOutlet="hoge"></ng-container>
```

ここで、 ```hoge``` は動的に読み込みたいコンポーネント｡
```ngComponentOutlet``` にセットされているのはコンポーネントで定義されている変数 ```hoge``` だが､この変数に動的に読み込む対象のコンポーネントがセットされている｡

テンプレートでやることはこれだけ｡
上記コードのように対象のコンポーネントを ```*ngComponentOutlet="hoge"``` という形で指定してやるだけでよい｡

なお､上記コードでコンポーネントを動的に読み込むためには

* app.module.ts
* NgComponentOutlet を使うコンポーネント

で下準備が必要となる｡


## app.module.ts における準備

```typescript:app.module.ts
// 省略
import { HogeComponent } from './hoge/hoge.component';

// 省略
@NgModule({
  declarations: [
    // 省略
    HogeComponent
  ],
  // ngComponentOutlet で動的にコンポーネントを読み込むにはこの宣言が必要
  entryComponents: [
    HogeComponent
  ],

  // 省略
})
// 省略
```

通常､コンポーネントを使用する場合は ```declarations``` に設定してやるだけでよいが､今回のケースでは ```entryComponents``` にも設定してやる必要がある｡
これを怠ると､次のエラーが出てブラウザに何も表示されない｡

### entryComponentsへの設定を怠った場合のエラー情報

```:entryComponentsへの設定を怠った場合のエラー情報
ERROR Error: "No component factory found for hoge. Did you add it to @NgModule.entryComponents?"
```

逆に上記エラーが出たら ```entryComponents``` がセットされていない可能性があるので、app.module を見直してみると良い。


## NgComponentOutletを使うコンポーネント

```typescript:NgComponentOutletを使うコンポーネント
// 省略
import { HogeComponent } from './hoge/hoge.component';

// 省略
export class AppComponent implements OnInit {
  // 省略

  hoge = HogeComponent;

  // 省略
}
```

```import``` 宣言したコンポーネントを､直接変数 ```hoge``` にセットしている点がポイント｡

1. コンポーネントそのものを変数にセット
1. セットした変数をテンプレートで ```ngComponentOutlet``` で参照
1. このとき ```ng-container``` と共に使用する

これらを [app.module.tsにおける準備](#appmodulets-における準備) とあわせて行うことで､コンポーネント ```HogeComponent``` を動的に読み込むという要件が実現される｡

以上､簡単ながらサンプルコードで ```NgComponentOutlet``` の使い方について見てきた｡

それでは以降の項目で、今回の要件である ｢```NgComponentOutlet``` を使ったタブの切り替え処理の実装｣ について実際のコードを見ていく。


# Component の実装

## switch-tab.component

switch-tab.component については *ts, *.html, *.css についてコードを載せた。
それぞれについてポイントを見ていく。

### switch-tab.component.ts

```typescript:switch-tab.component.ts
import { Component, OnInit } from '@angular/core';

import { SwitchTabService } from '../../service/switch-tab.service';

import { TabModel } from '../../model/tab-model';

@Component({
  selector: 'app-switch-tab',
  templateUrl: './switch-tab.component.html',
  styleUrls: ['./switch-tab.component.css']
})
export class SwitchTabComponent implements OnInit {

  private _currentTab: any;
  private _tabs!: Array<TabModel>;

  // 次のブロックは setter/getter
  // 今回の実装ではプロパティに対してセット/ゲットするだけの単純なもの

  public get currentTab(): any {
    return this._currentTab;
  }

  public get tabs(): Array<TabModel> {
    return this._tabs;
  }

  /**
   * コンストラクタ ( 本コンポーネントではなにもしない )
   *
   * @memberof SwitchTabComponent
   */
  constructor(
    private switchTabService: SwitchTabService
  ) {}

  /**
   * 初期処理
   *
   * @memberof SwitchTabComponent
   */
  ngOnInit() {
    // view に表示するための情報をここでセットする
    this._tabs = this.switchTabService.tabs;
    this._currentTab = this.switchTabService.getCurrentContents();
  }

  /**
   * ボタンがクリックされた時のイベントハンドラ
   *
   * @param {any} $event イベント情報
   * @memberof SwitchTabComponent
   */
  public onClick($event: any) {
    // クリックされたタブに応じて表示するコンテンツ( component ) を切り替える
    this._currentTab = this.switchTabService.changeCurrentContents($event.target.innerHTML);
  }
}
```

ポイントは switch-tab.service を DI していること。
このコンポーネントの役割はタブによって表示するコンポーネントを切り替えることだが、その切り替えるコンポーネントを DI している switch-tab.service から取得している。

そして取得したデータを次の2つのプロパティを介してテンプレートで参照することで、コンポーネントの動的読み込みによるタブの切り替えを実現している。

* currentTab プロパティ
  * 現在表示しているコンポーネントを扱う
  * 後述のテンプレートで ```ngComponentOutlet``` にセットされることで、このプロパティにセットされたコンポーネントが動的に読み込まれる

* tabs プロパティ
  * 本コンポーネントで扱うコンポーネントがセットされる配列
  * 後述のテンプレートで ```ngFor``` で扱われ、配列にセットされているコンポーネントがタブ表示される

### switch-tab.component.html

```html:switch-tab.component.html
<div class="tab-area-base">
  <ul class="tab-menu-base">
    <ng-container *ngFor="let tab of tabs">
      <li [class.current]="tab.current" (click)="onClick($event)">{{tab.name}}</li>
    </ng-container>
  </ul>

  <!-- ngComponentOutlet で動的にコンポーネントを読み込む -->
  <ng-container *ngComponentOutlet="currentTab"></ng-container>
</div>
```

ポイントは 2点。

* ```ngFor``` で DI した switch-tab.service から取得したコンポーネントの配列を回してタブ表示していること
* ```ngComponentOutlet``` で現在表示対象となるコンポーネントの動的読み込みを行っていること

```ngFor``` でループ処理をしていることで ```<li>タグ``` を何度も記述せずに済んでいるし、```ngComponentOutlet``` を使用することで、切り替え対象のコンポーネントを ```ngIf``` で管理せずに済んでいる。


### switch-tab.component.css

```css:switch-tab.component.css
div.tab-area-base {
  height: 50px;
  width: 90%;
  margin: auto;
}

ul.tab-menu-base {
  overflow: hidden;
  list-style: none;
  margin: 0px auto;
  padding: 0;
  border-bottom: 5px solid rgb(240, 240, 240);
}

ul.tab-menu-base li {
  float: left;
  margin: 20px 8px 0 0 ;
  padding: 0;
  background: rgb(180, 180, 180);
  border-radius: 10px 10px 0 0;
  height: 100%;
  width: 200px;
  font-size: 1.5em;
  text-align: center;
}

ul.tab-menu-base li.current {
  background: rgb(240, 240, 240);
}
```

こちらについて特筆する点は無し。
CSS でタブUI を実現するための一般的なスタイルを記述している(ハズ)。
前掲の [switch-tab.component.html](#switch-tabcomponenthtml) とあわせて見ていただければ。。。


## tab-a.component, tab-b.component

```tab-a.component```, ```tab-b.component``` は ```switch-tab.component``` でタブ切り替えに使われるだけのコンポーネントなので説明を割愛する。


# Model の実装

## tab-model

```typescript:tab-model.ts
export class TabModel {
  private _name: string;
  private _contents: any;
  private _current: boolean;

  /**
   * コンストラクタ
   *
   * @param {string} _name タブ名
   * @param {*} _contents コンテンツ( 実態はコンポーネントそのもの )
   * @param {boolean} _current 現在表示中か否かを示すフラグ
   * @memberof TabModel
   */
  constructor(_name: string, _contents: any, _current: boolean) {
    this._name = _name;
    this._contents = _contents;
    this._current = _current;
  }

  // 以降のブロックは setter/getter
  // 今回の実装ではプロパティに対してセット/ゲットするだけの単純なもの

  public set name(_name: string) {
    this._name = _name;
  }

  public get name(): string {
    return this._name;
  }

  public set contents(_contents: any) {
    this._contents = _contents;
  }

  public get contents(): any {
    return this._contents;
  }

  public set current(_current: boolean) {
    this._current = _current;
  }

  public get current(): boolean {
    return this._current;
  }
}
```

tab-model でのポイントは

* タブとして扱うコンポーネントを管理する型

であるという点。
そのためのプロパティとして

* name: 表示するタブ名
* contents: 表示するコンテンツ( コンポーネント )
* current: 現在表示していることを示すフラグ

を用意している。
これを後述の switch-tab.service で配列の型として指定することで、 switch-tab.service 経由で扱う際に型推論のメリットを享受できる。


# サービスの実装

## switch-tab.service

```typescript:switch-tab.service.ts
import { Injectable } from '@angular/core';

import { TabModel } from '../model/tab-model';

@Injectable({
  providedIn: 'root'
})
export class SwitchTabService {

  private _tabs!: Array<TabModel>;

  // 次のブロックは setter/getter
  // 今回の実装ではプロパティに対してセット/ゲットするだけの単純なもの

  public get tabs(): Array<TabModel> {
    return this._tabs;
  }

  public set tabs(tabs: Array<TabModel>) {
    this._tabs = tabs;
  }

  /**
   * コンストラクタ
   *
   * @memberof SwitchTabService
   */
  constructor() { }

  /**
   * 現在表示中のコンテンツを取得する
   *
   * @returns {any} 表示中のコンテンツ( 実態はコンポーネント... これも抽象クラス作ってやると型指定できるけど今回はそこまでしない )
   * @memberof SwitchTabService
   */
  public getCurrentContents(): any {
    // this._tabs は Array 型なので「for-in」は使わないで無難に「for文」で回す
    // 「for-of」でも良いけれど、それだとデバッグ時に「this」やら「_tabs」が何故か「undefined」になって気持ち悪い...
    for (let index = 0; index < this._tabs.length; index++) {
      const target = this._tabs[index];
      if (target.current) {
        return target.contents;
      }
    }
  }

  /**
   * 表示するコンテンツを切り替える
   *
   * @param {string} name クリックされたタブのタブ名
   * @returns {any} 切り替え先のコンテンツ( 実態はコンポーネント... これも抽象クラス作ってやると型指定できるけど今回はそこまでしない )
   * @memberof SwitchTabService
   */
  public changeCurrentContents(name: string): any {
    let contents: any;

    // this._tabs は Array 型なので「for-in」は使わないで無難に「for文」で回す
    // 「for-of」でも良いけれど、それだとデバッグ時に「this」やら「_tabs」が何故か「undefined」になって気持ち悪い...
    for (let index = 0; index < this._tabs.length; index++) {
      const target = this._tabs[index];
      target.current = false;

      if (target.name === name) {
        target.current = true;
        contents = target.contents;
      }
    }
    return contents;
  }
}
```

switch-tab.service でのポイントは 3点。

* 一つ目は タブとして扱うコンポーネントを管理するためのプロパティとして tab-model 型の配列 ```_tabs``` を用意していること
* 二つ目、三つ目として、```_tabs``` に対して以下のメソッドを用意していること
  * 現在表示しているコンテンツを取得するメソッド
  * タブクリック時に現在表示しているコンテンツを切り替えるメソッド


# app.module と app.component

## app.module

```typescript:app.module.ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppComponent } from './app.component';
import { SwitchTabComponent } from './component/switch-tab/switch-tab.component';
import { TabAComponent } from './component/tab-a/tab-a.component';
import { TabBComponent } from './component/tab-b/tab-b.component';

import { SwitchTabService } from './service/switch-tab.service';

@NgModule({
  declarations: [
    AppComponent,
    SwitchTabComponent,
    TabAComponent,
    TabBComponent,
  ],
  // ngComponentOutlet で動的にコンポーネントを読み込むにはこの宣言が必要
  entryComponents: [
    TabAComponent,
    TabBComponent
  ],
  imports: [
    BrowserModule,
  ],
  providers: [
    SwitchTabService
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

app.module でのポイントは 2点。

* 一つ目は [app.module.tsにおける準備](#appmodulets-における準備) で触れたとおり、 ```entryComponents``` へのセットすること
* 二つ目は SwitchTabService を DI のため ```providers``` へセットすること



## app.component

```app.component.ts
import { Component, OnInit } from '@angular/core';

import { TabAComponent } from './component/tab-a/tab-a.component';
import { TabBComponent } from './component/tab-b/tab-b.component';

import { SwitchTabService } from './service/switch-tab.service';

import { TabModel } from './model/tab-model';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {

  private _tabs!: Array<TabModel> = new Array();

  /**
   * コンストラクタ ( 本コンポーネントではなにもしない )
   *
   * @param {SwitchTabService} switchTabService
   * @memberof AppComponent
   */
  constructor(
    private switchTabService: SwitchTabService
  ) {
  }

  /**
   * 初期処理
   *
   * @memberof AppComponent
   */
  ngOnInit() {
    // タブの情報をここでセットする
    // サービスにセットされたタブ情報は、それを利用するコンポーネントで共有される
    this._tabs.push(new TabModel('Tab-A', TabAComponent, true));
    this._tabs.push(new TabModel('Tab-B', TabBComponent, false));
    this.switchTabService.tabs = this._tabs;
  }
}
```

app.component でのポイントは 1点。

* switch-tab.component で扱うコンポーネントを switch-tab.service にセットすること

switch-tab.service は DI によってインスタンスがアプリ全体で共有されるので、ここでセットしたコンポーネント情報が switch-tab.component で扱える、という流れになる。
もしタブとして扱いたいコンポーネントが増えた場合はここでセットしてやるだけで良い。それだけで switch-tab.component は扱う対象のコンポーネントが増えたことも意識せずに動作する。


# 実行結果

## 初期表示: Tab-A を表示

アプリ起動直後は Tab-A が表示される。
これは [app.component](#appcomponent) の ```ngOnInit()``` で TabModel のインスタンス生成時に true をセットしているため。

![switch_tab_02.png](https://qiita-image-store.s3.amazonaws.com/0/193342/b13a1058-147f-c87d-faf1-ec53203cc709.png)


## 切り替え: Tab-B を表示

Tab-B をクリックすることで、Tab-B がちゃんと表示されている。
これは [switch-tab.service](#switch-tabservice) の ```changeCurrentContents()``` で選択したタブへの切り替え処理を行うことで実現している。

![switch_tab_03.png](https://qiita-image-store.s3.amazonaws.com/0/193342/f47eee42-b208-1ad4-2b98-001a202b7f80.png)


# 参考

* 本家の [NgComponentOutlet](https://angular.jp/api/common/NgComponentOutlet) のドキュメント(まだ日本語訳されていない)
* 本家の [Structural Directives](https://angular.jp/guide/structural-directives) のドキュメント(まだ日本語訳されていない)
* [動的コンポーネントローダー](https://angular.jp/guide/dynamic-component-loader)(本家の日本語ドキュメント)

# ソースコード

今回の記事で作成したコードは [こちら](https://github.com/ksh-fthr/angular-work/tree/feat_switch_tab) にアップしてあるのでご参考まで。
