# はじめに
[こちらの記事](https://qiita.com/ksh-fthr/items/9f73fa161a1d7798def2) でコンポーネントの作成について触れたので、本記事で Angular CLI でサービスの作り方についてみていく。

# 更新情報

## 2021/01/04
- 記事内で扱ったコードを Angular `v11.0.5` で確認しました

# 作業環境

| 環境                                          | バージョン          | 備考               |
| --------------------------------------------- | ------------------- | ------------------ |
| [Angular CLI](https://cli.angular.io/)        | ~~v1.6.3~~ v11.0.5  | `$ ng --version`   |
| [Angular](https://angular.io/)                | ~~v4.4.6~~ v11.0.5  | 同上               |
| [TypeScript](https://www.typescriptlang.org/) | v4.0.2              | 同上               |
| [Node.js](https://nodejs.org/ja/)             | ~~v9.2.1~~ v12.18.3 | `$ node --version` |
| [npm](https://www.npmjs.com/)                 | ~~v5.6.0~~ v6.14.6  | `$ npm --version`  |


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


# 前提条件

* Angular CLI でプロジェクトを作成している


# Angular CLI でサービスの雛形を作る

Angular CLI から次のコマンドで雛形を作ることができる。


## サービス生成のコマンド

```bash:サービスの雛形を作るコマンド
$ ng generate service service/common
  create src/app/service/common.service.spec.ts (374 bytes)
  create src/app/service/common.service.ts (112 bytes)
```

コンポーネント生成と同じく､ ```generate``` は ```g``` と省略することもできる｡
それ以外のポイントとしては

* ```ディレクトリ/サービス名``` とすることでサービスを配置するディレクトリを生成した上でファイルが生成される
 * コンポーネントの場合は単純に ```ng g component コンポーネント名``` でディレクトリが生成されたが､**サービスの場合はディレクトリを明示的に指定してやらなければいけない**点に注意
* サービスは **```app.module.ts```に自動で登録されない**
 * コンポーネントの場合は自動で登録されていたので､ここもコンポーネント-サービスとで異なる点として注意

といった項目があげられる｡


## *.ts ファイルをみる

生成したサービスをみると次のようになっている。

```typescript:common.service.ts
import { Injectable } from '@angular/core';

@Injectable()
export class CommonService {

  constructor() { }

}
```

ポイントは

* ```Injectable``` が ```import``` されていること
* ```@Injectable``` でクラスが修飾されていること

の 2点 で､これによって **DI( Dependency Injection )** が行えることを宣言することになる｡


# サービスの利用

サービスを利用するには､アプリ全体でインスタンスを共有するために ```app.module.ts``` で ```providers``` に登録してやる必要がある｡

と､その前に ```CommonService``` を修正し､サービスとして DI された際に使用するプロパティを用意しておく｡

## サービスを修正する

```typescript:common.service.ts
import { Injectable } from '@angular/core';

@Injectable()
export class CommonService {

  /**
   * 本サービスを DI したコンポーネントで参照されるプロパティ
   *
   * @type {String}
   * @memberof CommonService
   */
  public commonProp: String = '';

  /**
   * コンストラクタ. CommonService のインスタンスを生成する
   *
   * @memberof CommonService
   */
  constructor() {
    this.commonProp = 'このプロパティは CommonService のもの';
  }
}
```

ただプロパティを定義しただけの単純なコードだが､ポイントは

 * DI された際に参照されるプロパティとして ```commonProp``` を定義
 * コンストラクタで初期化

となる｡


## サービスを登録する

前述の通り､サービスの利用には

* ```app.module.ts``` で ```import``` ～ ```providers``` に登録

する必要がある｡以下､実際にコードをみていく｡

```typescript:app.module.ts
// 省略

// サービスを登録するための import
import { CommonService } from './service/common.service';

@NgModule({
  declarations: [
    // 省略
  ],
  imports: [
    // 省略
  ],
  // サービスを登録する
  providers: [
    CommonService
  ],
  // 省略
})
export class AppModule { }
```

これで

* ```CommonService``` のインスタンスがアプリ全体で共有されるサービスとして登録された

ことになる｡
あとは利用したいコンポーネントでサービスを DI してやれば良い｡


## サービスを DI する

```app.module.ts``` で登録したサービスを DI するには

* 使用したいコンポーネントで ```import``` してコンストラクタで引数にセット

することで実現できる｡実際のコードは次の通り｡


### クラス

```typescript:sample1.component.ts
import { Component, OnInit } from '@angular/core';

// サービスを登録するための import
import { CommonService } from '../service/common.service';

@Component({
  selector: 'app-sample1',
  templateUrl: './sample1.component.html',
  styleUrls: ['./sample1.component.css']
})
export class Sample1Component implements OnInit {

  /**
   * CommonService のプロパティの参照を取得するプロパティ
   *
   * @type {String}
   * @memberof Sample1Component
   */
  public serviceProp: String = '';

  /**
   * コンストラクタ. ServiceSample1Component のインスタンスを生成する
   *
   * @param {CommonService} commonService 共通サービス
   * @memberof Sample1Component
   */
  constructor(private commonService: CommonService) { }

  /**
   * ライフサイクルメソッド｡コンポーネントの初期化で使用する
   *
   * @memberof Sample1Component
   */
  ngOnInit() {
    this.serviceProp = this.commonService.commonProp;
  }
}
```

ポイントは次の 3点｡

* ```import``` したサービスをコンストラクタの引数にセット
* サービスで定義されたプロパティの値を取得するためのプロパティの定義
* ngOnInit でサービスから値を取得

これによって ```CommonService``` で定義､セットされたプロパティの値を DI したコンポーネントで取得できる｡


### テンプレート

取得したサービスの値を表示するためのテンプレートは次の通り｡

```html:sample1.component.html
<p>
  CommonService で設定されている値は｢ {{this.serviceProp}} ｣です｡
</p>
```

こちらはクラスで定義したプロパティに対して片方向データバインドしただけの単純なもの｡


# 実行結果

上記までのコードを実行するとこうなる｡
![confirm_service_propaty.png](https://qiita-image-store.s3.amazonaws.com/0/193342/6abed0e3-4a56-f79e-3fe1-ebb1afd5a9fa.png)

オレンジ枠で囲った部分に ```CommonService``` でセットした文字列が表示されていることが確認できる｡


# まとめ

サービスを利用するにあたり､次の点を確認できた｡

* ```ng g service サービス名``` でサービスを生成できる
* ディレクトリを指定する場合は ```ng g service ディレクトリ/サービス名``` とすればよい
 * このときディレクトリがなければディレクトリを生成した上でサービスが生成される
* 生成されたサービスは ```@Injectable``` で修飾されている
 * これによって **DI( Dependency Injection )** が行えることを宣言している
* サービスは( コンポーネントと異なり )生成しただけではアプリに登録されない
* サービスは ```app.module.ts``` で登録することでアプリ全体でインスタンスを共有することになる
* サービスの利用にあたっては､利用するコンポーネント側で ```import``` し､コンストラクタの引数にセットする必要がある


# 補足

## コンポーネントでサービスを DI する

なおサービスの DI は ```app.module.ts``` だけではなく､任意のコンポーネントで行うこともできる｡
実装方法は次の通り｡

```typescript:sample1.component.ts
import { Component, OnInit } from '@angular/core';

// サービスを登録するための import
import { CommonService } from '../service/common.service';

@Component({
  selector: 'app-sample1',
  templateUrl: './sample1.component.html',
  styleUrls: ['./sample1.component.css'],
  providers: [
    // サービスを登録する
    CommonService
  ]
})
export class Sample1Component implements OnInit {

  /**
   * CommonService のプロパティの参照を取得するプロパティ
   *
   * @type {String}
   * @memberof Sample1Component
   */
  public serviceProp: String = '';

  /**
   * コンストラクタ. ServiceSample1Component のインスタンスを生成する
   *
   * @param {CommonService} commonService 共通サービス
   * @memberof Sample1Component
   */
  constructor(private commonService: CommonService) { }

  /**
   * ライフサイクルメソッド｡コンポーネントの初期化で使用する
   *
   * @memberof Sample1Component
   */
  ngOnInit() {
    this.serviceProp = this.commonService.commonProp;
  }
}
```

この場合､ 

* ```app.module.ts``` でサービスの ```import``` と ```providers``` への登録は不要

である｡
またコンポーネントでサービスを DI した場合､

* そのサービスはアプリ全体での共有は行われない
* DI したコンポーネント､及びその子コンポーネントでのみ共有される

ので注意すること｡


# ソースコード

今回の記事で動作確認に使用したコードは [ここ](https://github.com/ksh-fthr/angular-work/tree/feat_create_service) にアップしたので、ご参考までに。。。
