# はじめに
Angular でバックエンドとの疎通を行う方法について見ていく。
ここでは ```Http``` サービスを使用し、REST-API による実現方法を確認する。

# 更新情報
## 2021/08/14
- [補足-3](#補足-3) として､レスポンスにヘッダ情報を含める設定について追記しました

## 2021/01/04
- 記事内で扱ったコードを Angular `v11.0.5` で確認しました

## 2019/03/22
* Angular 7.2.2 の現在、`Headers` が<font color="red">非推奨</font>となっているため、REST-API のヘッダ情報を `HttpHeaders` を使用するよう修正しました

## 2018/05/13

* Angular 5.0.0 で ```@angular/http``` が<font color="red">非推奨</font>( [出典:Version 5.0.0 of Angular Now Available](https://blog.angular.io/version-5-0-0-of-angular-now-available-37e414935ced) )となったため､Httpクライアントモジュールを ```@angular/common/http``` に変更しました
* ｢登録｣｢更新｣｢削除｣ の動作検証を行えるよう UI を変更しました

# 作業環境
## バックエンド
( Node.js と npm のバージョンはフロントエンドと同じ )

| 環境                                          | バージョン          | 備考                    |
| --------------------------------------------- | ------------------- | ----------------------- |
| [express](https://expressjs.com/)             | v4.14.1             | package.json の記載から |


## フロントエンド
| 環境                                          | バージョン          | 備考                    |
| --------------------------------------------- | ------------------- | ----------------------- |
| [Angular CLI](https://cli.angular.io/)        | ~~v6.1.3~~ v11.0.5  | `$ ng --version`        |
| [Angular](https://angular.io/)                | ~~v6.1.2~~ v11.0.5  | 同上                    |
| [TypeScript](https://www.typescriptlang.org/) | v4.0.2              | 同上                    |
| [Node.js](https://nodejs.org/ja/)             | ~~v9.4.0~~ v12.18.3 | `$ node --version`      |
| [npm](https://www.npmjs.com/)                 | ~~v6.1.0~~ v6.14.6  | `$ npm --version`       |


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


# 構成

* バックエンド
 * Express( http://localhost:3000 )
* フロンドエンド
 * Angular( http://localhost:4200 )

# バックエンド

Angular による Http クライアントの実装が本記事の目的だが､その相手であるバックエンドも必要なので、Express を使用したリクエストを受け付けるだけの単純なサーバを本項で実装する。
( Express については [こちらの記事](https://qiita.com/ksh-fthr/items/c22dedbc0952bfdcc808) に開発環境の構築について記載している。ご参考まで )

```javascript:index.js
var express = require('express');
var router = express.Router();

var strage = {
  id: 0,
  message: 'デフォルトメッセージ'
};

const strages = [strage];

/**
 * HTTP の GET メソッドを待ち受けてステータスコードと文字列, メッセージリストを返す
 * レスポンスは下記のJSONフォーマットで返却する
 * {
 *   status: 200,
 *   response: 'メッセージリストを返却',
 *   messages: {{メッセージリスト}}
 * }
 * といった JSON が返却される
 */
router.get('/message/get', function(req, res, next) {
  res.status(200);
  res.json({
    status: 200,
    response: 'メッセージリストを返却',
    messages: strages
  });
});

module.exports = router;
```

実装に関してポイントは特になく､コード中のコメントに記載のとおりである｡
ソースコメントの繰り返しとなるが､ Router によって HTTP の GET メソッドを待ち受けるハンドラを登録しただけの単純なもので､

```javascript:返却するレスポンス
{
  status: 200,
  response: 'メッセージリストを返却',
  messages: {{メッセージリスト}}
}
```

というレスポンスを返す｡


# フロントエンド

冒頭で記載のとおり､こちらは Angular 標準で提供されている ```Http``` サービスを利用した HTTP クライアントを実装する。
以下の項目では「 GET リクエストを発行してバックエンドから取得した文字列を表示する処理 」の実装についてみていく。

## ```Http``` サービス利用のために ```HttpModule``` を import

```Http``` サービスを利用するには ```HttpModule``` が必要なので､```app.module.ts``` で import する｡

```typescript:app.module.ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';

// HTTP クライアントのための import ( 下記は Angular 5.0.0 で非推奨となった )
//import { HttpModule } from '@angular/http';

// HTTP クライアントのための import ( Angular 5.0.0 以降はこちらを使う )
import { HttpClientModule } from '@angular/common/http';

import { AppComponent } from './app.component';

// HTTP クライアントとしてのコンポーネント
import { HttpClientComponent } from './http-client/http-client.component';

// バックエンドとの通信を実際に担当するサービス
import { HttpClientService } from './service/http-client.service';

@NgModule({
  declarations: [
    AppComponent,
    HttpClientComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    // モジュールを利用を宣言する
    // 下記は Angular 5.0.0 で非推奨となった
    //HttpModule
    // Angular 5.0.0 以降はこちらを使う
    HttpClientModule
  ],
  providers: [
    // 自作サービスをアプリ全体で DI するために登録する
    HttpClientService
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

```app.module.ts``` でみるポイントは次の2点｡

1. HttpModule の import 宣言
1. HttpModule を imports プロパティの要素に登録して利用を宣言

この2点でアプリ全体で ```Http``` サービスを利用するための準備となる｡


## バックエンドとの通信を担当するサービス(GETの例)

バックエンドとの実際の通信を担当する役割をもったサービスを実装する｡
実装する内容としては大まかに

* ```Http``` サービスの import
* バックエンドから提供されている REST-API の実行

となる｡以下､GETの例についてコードを示す｡

```typescript:http-client.service.ts
import { Injectable } from '@angular/core';

// REST クライアント実装ののためのサービスを import ( 下記は Angular 5.0.0 で非推奨となった )
//import { Http } from '@angular/http';

// REST クライアント実装ののためのサービスを import ( Angular 5.0.0 以降はこちらを使う )
import { HttpClient, HttpHeaders } from '@angular/common/http';

@Injectable()
export class HttpClientService {

  // 下記のコードは非推奨
  /**
   * リクエストヘッダを定義
   *
   * @private
   * @memberof HttpClientService
   */
  //private headers: any = new Headers({'Content-Type': 'application/json'});

  // `Headers` の代わりに `HttpHeaders` を利用する
  // またコメントにあるとおり、 `Authorization` を設定できるように `httpOptions` としてオブジェクトでラップしている
  // 参考
  // https://angular.jp/guide/http#%E3%83%98%E3%83%83%E3%83%80%E3%83%BC%E3%82%92%E8%BF%BD%E5%8A%A0%E3%81%99%E3%82%8B
  /**
   * Http クライアントを実行する際のヘッダオプション
   * @private
   * @type {*}
   * @memberof HttpClientService
   * @description
   * 認証トークンを使用するために `httpOptions` としてオブジェクトを用意した。
   */
  private httpOptions: any = {
    // ヘッダ情報
    headers: new HttpHeaders({
      'Content-Type': 'application/json'
    }),
    //
    // レスポンスにヘッダ情報を入れるための設定
    // https://angular.io/guide/http#reading-the-full-response
    //
    observe: 'response',  // ⇐ これを追加
    //
    // DELETE 実行時に `body` が必要になるケースがあるのでプロパティとして用意しておく
    // ( ここで用意しなくても追加できるけど... )
    body: null
  };

  /**
   * RST-API 実行時に指定する URL
   *
   * バックエンドは Express で実装し、ポート番号「3000」で待ち受けているため、
   * そのまま指定すると CORS でエラーになる
   * それを回避するため、ここではフロントエンドのポート番号「4200」を指定し、
   * Angular CLI のリバースプロキシを利用してバックエンドとの通信を実現する
   *
   * @private
   * @memberof HttpClientService
   */
  private host: string = 'http://localhost:4200/app';

  /**
   * コンストラクタ. HttpClientService のインスタンスを生成する
   *
   * @param {Http} http Httpサービスを DI する
   * @memberof HttpClientService
   */
  constructor(private http: HttpClient) {
    // `Authorization` に `Bearer トークン` をセットする
    this.setAuthorization('my-auth-token');
  }

  /**
   * HTTP GET メソッドを実行する
   * (toPromise.then((res) =>{}) を利用する場合のコード)
   *
   * @returns {Promise<any[]>}
   * @memberof HttpClientService
   */
  public get(): Promise<any[]> {
    return this.http.get(this.host + '/message/get', this.httpOptions)
    .toPromise()
    .then((res) => {
      // response の型は any ではなく class で型を定義した方が良いが
      // ここでは簡便さから any としておく

      // @angular/http では json() でパースする必要があったが､ @angular/common/http では不要となった
      //const response: any = res.json();
      const response: any = res;
      return response;
    })
    .catch(this.errorHandler);
  }

  /**
   * REST-API 実行時のエラーハンドラ
   * (toPromise.then((res) =>{}) を利用する場合のコード)
   *
   * @private
   * @param {any} err エラー情報
   * @memberof HttpClientService
   */
  private errorHandler(err) {
    console.log('Error occured.', err);
    return Promise.reject(err.message || err);
  }

  /**
   * Authorizatino に認証トークンを設定しする
   *
   * @param {string} token 認証トークン
   * @returns {void}
   * @memberof HttpClientService
   * @description
   * トークンを動的に設定できるようメソッド化している
   * Bearer トークンをヘッダに設定したい場合はこのメソッドを利用する
   */
  public setAuthorization(token: string = ''): void {
    if (!token) {
      return;
    }
    const bearerToken: string = `Bearer ${token}`;
    this.httpOptions.headers = this.httpOptions.headers.set('Authorization', bearerToken);
  }
}
```

コードの補足ポイントについて記載する｡

1. RST-API 実行時に指定する URL
    フロントエンドとバックエンドではポート番号が異なるために <font color="red">CORS によるエラー</font> が発生する｡
    後述するが､本記事では Angular CLI によるリバースプロキシを利用してこの問題を回避しており､ここではリバースプロキシに投げるURLとして ```http://localhost:4200/app``` を指定している｡

1. RESTR-API の実行
    ```Http``` サービスの ```get``` メソッドを実行することで､バックエンドから提供されている REST-API を実行している｡
    このときレスポンスは **```toPromise.then((res) => {})```** のアロー関数で受けている点に注目｡
    <font color="red">非同期で実行されている REST-API を ```toPromise()``` で待受け､レスポンスが返ってきたら ```then((res) => {})``` で処理する</font>､といった流れである｡

    また `toPromise()` は、本来 `Observable` オブジェクトである `http.get()` の戻り値を強制的に `Promise` オブジェクトに変換している処理で、これによって後続の処理が `Promise` オブジェクトのメソッドで処理できるようになっている。

    そして返却されてきたレスポンスの扱いについて。
    ~~バックエンドはレスポンスを ```JSON``` フォーマットで返却しているため､ここではそのレスポンスを ```json()``` で ```JSON``` フォーマットに復元している｡~~
    ```@angular/http``` では json() でパースする必要があったが､ ```@angular/common/http``` では不要となったため､ここでは <font color="red">返却されたレスポンスをそのままセット</font> している｡

1. Authorization の設定
    `Bearer`トークンを動的に設定できるよう、`setAuthorization()` としてトークンの設定処理をメソッド化している。
    今回はコンストラクタで実行しているが、必ず時もコンストラクタでなければならない訳ではない。トークンを取得したタイミングでセットしても良いし、他のタイミングでも良い。要件にあったタイミングでどうぞ。

1. 補足
    コメントにも記載してあるが､バックエンドから返却されるデータについては ```any``` ではなく､返却された値を保持するためのクラスなりインターフェイスを用意するのが望ましいが､本記事ではサンプルコードということで ```any``` で処理している。


## 作成したサービスを利用するコンポーネント

作成した HttpClientService を使ってバックエンドから情報を取得するコンポーネントを実装する。

```typescript:http-client.component.ts
import { Component, OnInit } from '@angular/core';
import { HttpClientService } from '../service/http-client.service';

@Component({
  selector: 'app-http-client',
  templateUrl: './http-client.component.html',
  styleUrls: ['./http-client.component.css']
})
export class HttpClientComponent implements OnInit {

  /**
   * バックエンドから返却されたレスポンスをセットするプロパティ
   *
   * 型は any ではなく class で型を定義した方が良いが
   * ここでは簡便さから any としておく
   *
   * @private
   * @type {string}
   * @memberof HttpClientComponent
   */
  public param: any = {};

  /**
   * バックエンドから返却されたたメッセージをセットするプロパティ
   *
   * @type {*}
   * @memberof HttpClientComponent
   */
  public messageInfo: any = {
    id: null,
    message: null
  };

  /**
   * バックエンドから返却されたたメッセージを保持するリストプロパティ
   *
   * @type {*}
   * @memberof HttpClientComponent
   */
  public messageInfoList: any = [this.messageInfo];

  /**
   * メッセージ登録回数
   *
   * @private
   * @type {number}
   * @memberof HttpClientComponent
   */
  public messageId: number = 1;

  /**
   * 入力メッセージ
   *
   * @type {string}
   * @memberof HttpClientComponent
   */
  public message: string = '';

  /**
   * コンストラクタ. HttpClientComponent のインスタンスを生成する
   * 自作した HttpClientService を DI する
   *
   * @param {HttpClientService} httpClientService HTTP通信を担当するサービス
   * @memberof HttpClientComponent
   */
  constructor(private httpClientService: HttpClientService) { }

  /**
   * ライフサイクルメソッド｡コンポーネントの初期化で使用する
   * 今回はコンポーネントの初期化時にバックエンドから情報を取得してビューに表示する
   *
   * @memberof HttpClientComponent
   */
  ngOnInit() {
    // ------
    // toPromise.then((res) =>{}) を利用する場合のコード
    // ------
    this.httpClientService.get()
    .then(
      (response: any) => {
        this.param = response.body;
        this.messageInfoList = this.param.messages;
      }
    )
    .catch(
      (error) => console.log(error)
    );
  }
}
```

ポイントは

* ```ngOnInit()``` で ```HttpClientService``` の ```get()``` メソッドを実行している

箇所で、このとき ```then()``` で処理結果を待ち受けてバックエンドから返却されたレスポンスを処理している｡

返却されたレスポンスの内容だが､ ```response.body``` にはバックエンドから返却された次の ```JSON``` オブジェクトがセットされているので､

```javascript:paramsにセットされているオブジェクト
params = {
    status: 200,
    response: 'メッセージリストを返却',
    messages: [
      {
        id: 0,
        message: 'デフォルトメッセージ'
      }
    ]
  }
]
```

ビュー( ```http-client.html``` )でメッセージの一覧を ```*ngFor``` で表示するために､返却されたレスポンスを ```this.messageInfoList``` にセットしている｡

```html:http-client.html
<!-- 片方向データバインドでバックエンドから取得した情報を表示する -->
<table class="table">
  <caption>バックエンドから取得したメッセージリスト</caption>
  <tr>
    <th id="th-id">ID</th>
    <th id="th-message">メッセージ</th>
  </tr>
  <!--インデックス番号（index）に変数iでアクセスできるように-->
  <tr *ngFor="let messageInfo of messageInfoList; index as i">
    <td>{{messageInfo.id}}</td>
    <td>{{messageInfo.message}}</td>
  </tr>
</table>

<form #formCheck="ngForm" class="formCheck">
  <input type="text" id="form-id" class="input-text" name="message-id" [(ngModel)]="messageId">
  <input type="text" id="form-message" class="input-text" name="message" [(ngModel)]="message">
</form>

<input type="button" id="register" value="登録" (click)="onClickRegister($event)">
<input type="button" id="update" value="更新" (click)="onClickUpdate($event)">
<input type="button" id="delete" value="削除" (click)="onClickDelete($event)">
```

前述のとおりメッセージの一覧を表示するために､ビューの処理は ```*ngFor``` を用いて前掲のクラスでセットしたプロパティ ```messageInfoList``` から 要素を一つずつ取り出し､ ```id``` と ```message``` を参照している｡

なお ```form``` タグ以降のブロックはメッセージの｢登録｣｢更新｣｢削除｣を行うためのインターフェース部分となる｡｢登録｣｢更新｣｢削除｣の処理については後述の [補足-2](#補足-2) で実装例を挙げているので、そちらも参照いただければ。。｡


# Angular CLI によるリバースプロキシ設定

冒頭の繰り返しとなるが、今回の構成は次のようになっていて､

* フロントエンドが Angular でポート番号:4200 で起動する
* バックエンドが Express でポート番号:3000 で起動する

この時、両者はポート番号が異なるためにフロントエンドからバックエンドに対してリクエストを送っても <font color="red">CORS でエラー</font> となる。
これを回避するために Angular CLI でリバースプロキシの設定を行う。
( リバースプロキシの設定については [こちらの記事](https://qiita.com/ksh-fthr/items/a462a96de7080092b73c) に記載している。ご参考まで )

設定した内容は次の通り。

* リバースプロキシ設定ファイル(proxy.conf.json)

```javascript:proxy.conf.json
{
  // localhost:4200/app を localhost:3000 にフォワード
  "/app": {
    "target": "http://localhost:3000",
    "pathRewrite": {"^/app": ""}
  }
}
```

* リバースプロキシ有りで起動するためのスクリプト

```javascript:package.json(抜粋)
  // 省略
  "scripts": {
    // 省略
    // リバースプロキシ設定ファイルを読み込ませて起動する
    "start": "ng serve --proxy-config proxy.conf.json",
    // 省略
  },
  // 省略
```

で、リバースプロキシの設定を行った場合、フロントエンド⇔バックエンド間は次の図のようになる。

* リバースプロキシ後のフロントエンド⇔バックエンド間

```:フロントエンド⇔バックエンド間
 フロントエンド ------> Angular CLI によるリバースプロキシ ------> バックエンド
(locahost:4200)                                        (localhost:3000)
```

ここで前掲の ```HttpClientService``` とバックエンドの ```GET``` メソッドを再度確認すると、

* フロントエンド

```typescript:http-client.service.ts(抜粋)
  // 省略
  private host: string = 'http://localhost:4200/app';

  // 省略

  /**
   * HTTP GET メソッドを実行する
   * (toPromise.then((res) =>{}) を利用する場合のコード)
   *
   * @returns {Promise<any[]>}
   * @memberof HttpClientService
   */
  public get(): Promise<any[]> {
    return this.http.get(this.host + '/message/get', this.httpOptions)
    .toPromise()
    .then((res) => {
      // response の型は any ではなく class で型を定義した方が良いが
      // ここでは簡便さから any としておく
      const response: any = res;
      return response;
    })
    .catch(this.errorHandler);
  }
```

* バックエンド

```javascript:index.js(抜粋)
/**
 * HTTP の GET メソッドを待ち受けてステータスコードと文字列, メッセージリストを返す
 * レスポンスは下記のJSONフォーマットで返却する
 * {
 *   status: 200,
 *   response: 'メッセージリストを返却',
 *   messages: {{メッセージリスト}}
 * }
 * といった JSON が返却される
 */
router.get('/message/get', function(req, res, next) {
  res.status(200);
  res.json({
    status: 200,
    response: 'メッセージリストを返却',
    messages: strages
  });
});
```

と実装してある。

上記までのリバースプロキシの設定､フロントエンドの実装、バックエンドの実装が意図するところをまとめると､

1. リバースプロキシは ```localhost:4200/app``` を ```localhost:3000``` にフォワードする設定である
1. バックエンドは ```localhost:3000/get``` で待ち受けている
1. フロントエンドは ```localhost:4200/app/get``` でリクエストを発行する
1. リバースプロキシの設定により､上記リクエストは ```localhost:3000/get``` にフォワードされる
1. バックエンドは無事リクエストを受け付けることができる

となる。

( 前掲の [参考記事](https://qiita.com/ksh-fthr/items/a462a96de7080092b73c) にも記載してあるが ) リバースプロキシありで Angular を起動する際は

```:リバースプロキシありでAngularを起動
$ npm start
```

とする必要があるので注意すること。
```ng serve``` でもアプリは起動するが､リバースプロキシありでアプリが起動していないので､期待通りの動作とならない｡


#  実行結果

バックエンド､フロントエンドを次のコマンドで起動する｡

```:バックエンドの起動
$ npm start
```

```:フロントエンドの起動
$ npm start
```

ともに起動した状態でブラウザから実行結果を見てみると

![http-client-get-result.png](https://qiita-image-store.s3.amazonaws.com/0/193342/9eda49df-feb4-3331-03ab-16091a2d520d.png)


となり､バックエンドから返却された値である ｢**デフォルトメッセージ**｣ が表示されていることが確認できた｡


# 補足-1

この記事では ```HttpClientService``` の ```get()``` メソッドで ```toPromise.then()``` を使用した例を挙げたが､その代わりに ```subscribe()``` を利用することもできる｡

その場合､該当する各コードは次のような実装になる｡

## toPromise().then() を subscribe() で書き換え

* サービス

```typescript:HttpClientService.ts
  /**
   * HTTP GET メソッドを実行する
   * (subscribe((res) =>{}) を利用する場合のコード)
   *
   * @param {*} callback HTTP GET の実行結果を受け取って処理するためのコールバック処理
   * @memberof HttpClientService
   */
  public get(callback: any) {
    this.http.get(this.host + '/message/get', this.httpOptions)
    .subscribe(
      (res) => {
        const response: any = res;
        callback(response);
      },
      (error) => {
        // subscribe の実装のときに this.errorHandler でエラー処理を
        // 行うと Uncaught (in promise) が発生するので、
        // ここではコンソールにログを出すだけにする
        console.log(error);
      }
    );
  }
```

* コンポーネント

```typescript:HttpClientComponent.ts
  /**
   * ライフサイクルメソッド｡コンポーネントの初期化で使用する
   * 今回はコンポーネントの初期化時にバックエンドから情報を取得してビューに表示する
   *
   * @memberof HttpClientComponent
   */
  ngOnInit() {
    // ------
    // subscribe((res) =>{}) を利用する場合のコード
    // ------
    // HTTP GET の実行結果を受け取るためのコールバックを引数に､ get() を呼び出す
    this.httpClientService.get((response: any) => {
      this.param = response.body;
      this.messageInfoList = this.param.messages;
    });
  }
```


# 補足-2

POST, PUT, DELETE の例を以降に記載する。
メモリ上で管理しているメッセージ情報に対する処理だが､これで雰囲気を掴んでいただければ｡

なおバックエンドからエラーを返却するケースもあるが､フロントエンド側ではそのエラーに対するケアは ```console.log(error)``` でログ出力するに留めている｡

## POST
### バックエンド

```javascript:index.js(postの例)
/**
 * HTTP の POST メソッドを待ち受けてメッセージを登録する
 * 成功時､ステータスコードと文字列､登録後のメッセージリストを返す
 *
 * レスポンスは下記のJSONフォーマットで返却する
 * {
 *   status: 200,
 *   response: 'メッセージを登録',
 *   messages: {{メッセージリスト}}
 * }
 * といった JSON が返却される
 */
router.post('/message/post', function(req, res, next) {
  if (!req.body.id || req.body.id === '') {
    res.status(400);
    res.json({status: 400, response: 'IDが指定されていない'})
    return;
  }

  for (var i = 0; i < strages.length; i++) {
    if (req.body.id === strages[i].id) {
      res.status(409);
      res.json({
        status: 409,
        response: 'IDが重複',
        messages: req.body
      })
      return;
    }
  }

  const localStorage = {
    id: req.body.id,
    message: req.body.message
  };
  strages.push(localStorage);

  res.status(200);
  res.json({
    status: 200,
    response: 'メッセージを登録',
    messages: strages
  })
});
```

### フロントエンド

* サービス

```typescript:http-client.service.ts(postの例)
  /**
   * メッセージ登録
   *
   * @param {*} body リクエストボディ
   * @returns {Promise<any[]>} バックエンドからのレスポンス
   * @memberof HttpClientService
   */
  public register(body: any): Promise<any[]> {
    return this.http.post(this.host + '/message/post', body, this.httpOptions)
    .toPromise()
    .then((res) => {
      const response: any = res;
      return response;
    })
    .catch(this.errorHandler);
  }
```

* コンポーネント

```typescript:http-client.component.ts(postの例)
  /**
   * メッセージ登録
   *
   * @private
   * @memberof HttpClientComponent
   */
  private doRegister() {
    const body: any = {
      id: this.messageId,
      message: this.message
    };
    this.httpClientService.register(body)
    .then(
      (response: any) => {
        this.param = response.body;
        this.messageInfoList = this.param.messages;
      }
    )
    .catch(
      (error) => console.log(error)
    );
  }
```

## PUT
### バックエンド

```javascript:index.js(putの例)
/**
 * HTTP の PUT メソッドを待ち受けてメッセージを更新する
 * 成功時､ステータスコードと文字列､ 更新後のメッセージリストを返す
 *
 * レスポンスは下記のJSONフォーマットで返却する
 * {
 *   status: 200,
 *   response: 'メッセージを更新',
 *   messages: {{メッセージリスト}}
 * }
 * といった JSON が返却される
 */
router.put('/message/put', function(req, res, next) {
  if (!req.body.id || req.body.id === '') {
    res.status(400);
    res.json({
      status: 400,
      response: 'IDが指定されていない',
      message: req.body
    })
    return;
  }

  for (var i = 0; i < strages.length; i++) {
    if (req.body.id === strages[i].id) {
      strages[i].message = req.body.message;
      res.status(200);
      res.json({
        status: 200,
        response: 'メッセージを更新',
        messages: strages
      })
      return;
    }
  }

  res.status(404);
  res.json({
    status: 404,
    response: '対象のメッセージが存在しない',
    messages: req.body
  })
});
```

### フロントエンド

* サービス

```typescript:http-client.service.ts(putの例)
  /**
   * メッセージ更新
   *
   * @param {*} body リクエストボディ
   * @returns {Promise<any[]>} バックエンドからのレスポンス
   * @memberof HttpClientService
   */
  public update(body: any): Promise<any[]> {
    return this.http.put(this.host + '/message/put', body, this.httpOptions)
    .toPromise()
    .then((res) => {
      const response: any = res;
      return response;
    })
    .catch(this.errorHandler);
  }
```

* コンポーネント

```typescript:http-client.component.ts(putの例)
  /**
   * メッセージ更新
   *
   * @private
   * @memberof HttpClientComponent
   */
  private doUpdate() {
    const body: any = {
      id: this.messageId,
      message: this.message
    };
    this.httpClientService.update(body)
    .then(
      (response: any) => {
        this.param = response.body;
        this.messageInfoList = this.param.messages;
      }
    )
    .catch(
      (error) => console.log(error)
    );
  }
```

## DELETE
### バックエンド

```javascript:index.js(deleteの例)
/**
 * HTTP の DELETE メソッドを待ち受けてメッセージを削除する
 * 成功時､ステータスコードと文字列､ 削除後のメッセージリストを返す
 *
 * レスポンスは下記のJSONフォーマットで返却する
 * {
 *   status: 200,
 *   response: 'メッセージを削除',
 *   messages: {{メッセージリスト}}
 * }
 * といった JSON が返却される
 */
router.delete('/message/delete', function(req, res, next) {
  if (!req.body.id || req.body.id === '') {
    res.status(400);
    res.json({
      status: 400,
      response: 'IDが指定されていない',
      message: req.body
    })
    return;
  }

  for (var i = 0; i < strages.length; i++) {
    if (req.body.id === strages[i].id) {
      strages.splice(i, 1);
      res.status(200);
      res.json({
        status: 200,
        response: 'メッセージを削除',
        messages: strages
      })
      return;
    }
  }

  res.status(404);
  res.json({
    status: 404,
    response: '対象のメッセージが存在しない',
    messages: req.body
  })
});
```

### フロントエンド

* DELETE については次の点に注意
    * Http サービスの delete メソッドでは body を引数にセットできない
    * DELETE 時に body をセットしたい場合、例に示したように **Header Option に含める** 必要がある

* サービス

```typescript:http-client.service.ts(deleteの例)
  /**
   * メッセージ削除
   *
   * @param {*} body リクエストボディ
   * @returns {Promise<any[]>} バックエンドからのレスポンス
   * @memberof HttpClientService
   */
  public delete(body: any): Promise<any[]> {
    this.httpOptions.body = body;
    return this.http.delete(this.host + '/message/delete', this.httpOptions)
    .toPromise()
    .then((res) => {
      const response: any = res;
      return response;
    })
    .catch(this.errorHandler);
  }
```

* コンポーネント

```typescript:http-client.component.ts(deleteの例)
  /**
   * メッセージ削除
   *
   * @private
   * @memberof HttpClientComponent
   */
  private doDelete() {
    const body: any = {
      id: this.messageId
    };
    this.httpClientService.delete(body)
    .then(
      (response: any) => {
        this.param = response.body;
        this.messageInfoList = this.param.messages;
      }
    )
    .catch(
      (error) => console.log(error)
    );
  }
```

## POST, PUT, DELETE の実行結果

### POST

* 実行前
![http-client-post-execute.png](https://qiita-image-store.s3.amazonaws.com/0/193342/fe968078-fa4b-1404-2f18-19743f43d068.png)

* 実行後
![http-client-post-result.png](https://qiita-image-store.s3.amazonaws.com/0/193342/960471c5-5e69-6ae9-f434-a668b4b92c0e.png)

### PUT

* 実行前
      ![http-client-put-execute.png](https://qiita-image-store.s3.amazonaws.com/0/193342/309429fd-e383-de23-1206-17255e7a72cb.png)

* 実行後
![http-client-put-result.png](https://qiita-image-store.s3.amazonaws.com/0/193342/ce2ae0c3-4530-8d0d-7ba6-3b2dde245607.png)


### DELETE

* 実行前
![http-client-delete-execute.png](https://qiita-image-store.s3.amazonaws.com/0/193342/017afb4d-9b8c-6c59-96cb-e90edc82aa9f.png)

* 実行後
![http-client-delete-result.png](https://qiita-image-store.s3.amazonaws.com/0/193342/213f3dff-e29e-d8e5-f115-ac7208c18d80.png)

# 補足-3
## レスポンスにヘッダ情報を含めるには?

`HttpClient` の各メソッド実行時に指定するオプションに `observe: 'response` を追加する｡
本記事のコードでいうと次の箇所となる｡

```typescript:http-client.service.ts
  private httpOptions: any = {
    // ヘッダ情報
    headers: new HttpHeaders({
      'Content-Type': 'application/json'
    }),
    //
    // レスポンスにヘッダ情報を入れるための設定
    // https://angular.io/guide/http#reading-the-full-response
    //
    observe: 'response',  // ⇐ これを追加
    //
    // DELETE 実行時に `body` が必要になるケースがあるのでプロパティとして用意しておく
    // ( ここで用意しなくても追加できるけど... )
    body: null
  };
```

## 上記設定をした場合のレスポンスの内容
### get のレスポンス
```bash:getのログ
[get] response:
{
    "headers":{
       "normalizedNames":{},
       "lazyUpdate":null
    },
    "status":200,
    "statusText":"OK",
    "url":"http://localhost:4200/app/get",
    "ok":true,
    "type":4,
    "body":{
        "status":200,
        "response":"メッセージリストを返却",
        "messages":[
            {
                "id":0,
                "message":"デフォルトメッセージ"
            }
        ]
    }
}
```

### post のレスポンス
```bash:postのログ
[post] response: {
    "headers":{
        "normalizedNames":{},
        "lazyUpdate":null
    },
    "status":200,
    "statusText":"OK",
    "url":"http://localhost:4200/app/post",
    "ok":true,
    "type":4,
    "body":{
        "status":200,
        "response":"メッセージを登録",
        "messages":[
            {
                "id":0,
                "message":"デフォルトメッセージ"
            },
            {
                "id":1,
                "message":"hogehoge"
            }
        ]
    }
}
```

### put のレスポンス
```bash:putのログ
[put] response: {
    "headers":{
        "normalizedNames":{},
        "lazyUpdate":null
    },
    "status":200,
    "statusText":"OK",
    "url":"http://localhost:4200/app/put",
    "ok":true,
    "type":4,
    "body":{
        "status":200,
        "response":"メッセージを更新",
        "messages":[
            {
                "id":0,
                "message":"デフォルトメッセージ"
            },
            {
                "id":1,
                "message":"hogehogehogehoge"
            }
        ]
    }
}
```

### delete のレスポンス
```bash:deleteのログ
[delete] response: {
    "headers":{
        "normalizedNames":{},
        "lazyUpdate":null
    },
    "status":200,
    "statusText":"OK",
    "url":"http://localhost:4200/app/delete",
    "ok":true,
    "type":4,
    "body":{
        "status":200,
        "response":"メッセージを削除",
        "messages":[
            {
                "id":0,"message":"デフォルトメッセージ"
            }
        ]
    }
}
```

# ソースコード

今回の記事で動作確認に使用したコードは下記にアップしてあるのでご参考まで。

* [フロントエンド](https://github.com/ksh-fthr/angular-work/tree/feat_http_client)
* [バックエンド](https://github.com/ksh-fthr/express-work/tree/feat_http_method)

# 参考

* [Version 5.0.0 of Angular Now Available](https://blog.angular.io/version-5-0-0-of-angular-now-available-37e414935ced)
* [Angular 5.0.0がリリースされました( 上記の日本語訳 )](https://medium.com/angular-japan-user-group/version-5-0-0-of-angular-now-available-9746ef966c7d)
* [HttpClient - 公式の日本語ドキュメント](https://angular.jp/guide/http#%E3%83%98%E3%83%83%E3%83%80%E3%83%BC%E3%82%92%E8%BF%BD%E5%8A%A0%E3%81%99%E3%82%8B)
* [Communicating with backend services using HTTP > Reading the full response](https://angular.io/guide/http#reading-the-full-response)
