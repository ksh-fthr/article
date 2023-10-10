# はじめに
親子関係にない複数コンポーネント間において **サービスを利用したデータ共有** の方法についてみていく｡

# 更新情報

## 2021/01/04
- 記事内で扱ったコードを Angular `v11.0.5` で確認しました

## 2018/05/26
* Angular のバージョンが `6` に上がったことで､ RxJS のバージョンも更新されました
* 上記に伴い、 RxJS の import パスも変更されました。つきましては、記事中のコードと [github のコード](https://github.com/ksh-fthr/angular-work/tree/feat_share_service) をあわせて修正しました


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

* [[Angular] Angular CLI によるサービスの生成](https://qiita.com/ksh-fthr/items/900baee52b80e6ed1b66)
* [[Angular] 親子関係にあるコンポーネント間でデータの受け渡しを行う](https://qiita.com/ksh-fthr/items/db6a48d072d5e9a33f0b)
* [[Angular] 子コンポーネントや外部コンテンツの参照を取得する](https://qiita.com/ksh-fthr/items/00341b3b12f7048c9575)

# 構成

本記事で扱う構成は次のとおり｡

```bash:構成
src/
  `-app/
       `-sample1/
       |   `- sample1.component.css
       |   `- sample1.component.html
       |   `- sample1.component.ts
       `-sample2/
       |   `- sample2.component.css
       |   `- sample2.component.html
       |   `- sample2.component.ts
       `-service/
       |   `- common.service.ts
       `- app.component.css
       `- app.component.html
       `- app.component.ts
       `- app.module.ts
```

今回の構成におけるポイントは次の 2点｡

* サービスによるデータの共有を確認するため次の2コンポーネントを配置
 * sample1.component
 * sample2.component
* コンポーネント間でデータ共有を実現するため次のサービスを配置
 * common.service

これらのコンポーネントとサービスで､コンポーネント間のデータ共有について確認する｡

# サービスの実装

## common.service

```typescript:common.service.ts
import { Injectable } from '@angular/core';

// イベント発火のための Subject を import
// Angular Ver.6.x.x では rxjs から直接importするように変更された
import { Subject } from 'rxjs';

// こちらは Angular 6.x.x だとビルドエラーとなる
//import { Subject } from 'rxjs/Subject';

@Injectable()
export class CommonService {

  /**
   * データの変更を通知するためのオブジェクト
   *
   * @private
   * @memberof CommonService
   */
  private sharedDataSource = new Subject<string>();

  /**
   * Subscribe するためのプロパティ
   * `- コンポーネント間で共有するためのプロパティ
   *
   * @memberof CommonService
   */
  public sharedDataSource$ = this.sharedDataSource.asObservable();

  /**
   * コンストラクタ. CommonService のインスタンスを生成する
   *
   * @memberof CommonService
   */
  constructor() {}

  /**
   * データの更新イベント
   *
   * @param {string} updateed 更新データ
   * @memberof CommonService
   */
  public onNotifySharedDataChanged(updateed: string) {
    console.log('[CommonService] onNotifySharedDataChanged fired.');
    this.sharedDataSource.next(updateed);
  }
}
```

以下､ポイントを順にみていく｡

1. Subject の import

    ```typescript:Subjectのimport
    import { Subject } from 'rxjs';
    ```

    次の ｢Subject型のプロパティを宣言｣ で必要となる Subject を import する｡
    本記事で確認するデータ共有の仕組みはこの ```Subject``` と､後述の コンポーネントで扱う ```Subscription``` で実現する｡

1. Subject 型のプロパティを宣言

    ```typescript:Subject型のプロパティを宣言
    private sharedDataSource = new Subject<string>();
    ```

    以降の ｢Subscribe のためのプロパティを宣言｣ と ｢データの更新イベント｣ で使用するプロパティを宣言し､ ```Subject``` のインスタンスを生成する｡
    このプロパティを用いて共有データ の変更通知を行う｡

1. Subscribe のためのプロパティを宣言

    ```typescript:Subscribeのためのプロパティを宣言
    public sharedDataSource$ = this.sharedDataSource.asObservable();
    ```

    先に生成したオブジェクト ```sharedDataSource``` から ```asObservable()``` でオブジェクトを生成する｡
    ```asObservable()``` で生成されたオブジェクトは､データ共有を行うコンポーネント側で ```subscribe``` (後述) することでデータ共有の仕組みを実現する｡

1. データの更新イベント

    ```typescript:データの更新イベント
    public onNotifySharedDataChanged(updateed: string) {
      this.sharedDataSource.next(updateed);
    }
    ```

    こちらは先に生成したオブジェクト ```sharedDataSource``` から ```next()``` を実行している｡
    このメソッドはデータ共有を行うコンポーネントから実行されることを想定したもので､引数の ```updated``` をそのまま ```next()``` の引数にセットしている｡
    このメソッドで ```this.sharedDataSource.next(updateed);``` が実行されることで､ ```subscribe``` (後述) で待ち受けているコンポーネントに引数のデータが展開される｡


# コンポーネントの実装

## sample1.component

```typescript:sample1.component.ts
import { Component, OnInit, OnDestroy } from '@angular/core';

// subscribe を保持するための Subscription を import
// Angular Ver.6.x.x では rxjs から直接importするように変更された
import { Subscription } from 'rxjs';

// こちらは Angular 6.x.x だとビルドエラーとなる
//import { Subscription } from 'rxjs/Subscription';

// サービスを登録するための import
// アプリ全体でのサービスの共有､コンポーネント単位でのサービスの共有に関わらず､ここの import は必要
import { CommonService } from '../service/common.service';

@Component({
  selector: 'app-sample1',
  templateUrl: './sample1.component.html',
  styleUrls: ['./sample1.component.css'],
})
export class Sample1Component implements OnInit, OnDestroy {

  /**
   * CommonService の変数の参照を取得するプロパティ
   *
   * @type {String}
   * @memberof Sample1Component
   */
  public serviceProp: String = 'Initialized by Sample1Component';

  /**
   * subscribe を保持するための Subscription
   *
   * @private
   * @type {Subscription}
   * @memberof Sample1Component
   */
  private subscription!: Subscription;

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

    // イベント登録
    // サービスで共有しているデータが更新されたら発火されるイベントをキャッチする
    this.subscription = this.commonService.sharedDataSource$.subscribe(
      msg => {
        console.log('[Sample1Component] shared data updated.');
        this.serviceProp = msg;
      }
    );
  }

  /**
   * コンポーネント終了時の処理
   *
   * @memberof Sample1Component
   */
  ngOnDestroy() {
    //  リソースリーク防止のため CommonService から subcribe したオブジェクトを破棄する
    this.subscription.unsubscribe();
  }

  /**
   * ボタンクリック時のイベントハンドラ
   *
   * @memberof Sample1Component
   */
  onClicSendMessage() {
    // CommonService のデータ更新を行う
    console.log('[Sample1Component] onClicSendMessage fired.');
    this.commonService.onNotifySharedDataChanged('Updated by Sample1Component.');
  }
}
```

```html:sample1.component.html
<p>
  CommonService で設定されている値は｢ {{this.serviceProp}} ｣です｡
</p>
<input type="button" value="send message" (click)="onClicSendMessage()" />
```

以下､ポイントを順にみていく｡
なお CommonService の DI については関連記事の [こちら](https://qiita.com/ksh-fthr/items/900baee52b80e6ed1b66) にあるので割愛する｡

1. Subscription の import

    ```typescript:Subscriptionのimport
    // 省略
    import { Subscription } from 'rxjs';
    // 省略
    private subscription!: Subscription
    ```

    ```subscribe``` を保持するため､```Subscription``` を import し､そのプロパティを宣言する｡
    

1. データ更新イベントをsubscribeするイベントハンドラを登録

    ```typescript:データ更新イベントをsubscribeするイベントハンドラを登録
    this.subscription = this.commonService.sharedDataSource$.subscribe(
      msg => {
        this.serviceProp = msg;
      }
    );
    ```

    CommonService における ```データの更新イベント``` で ```next()``` によって発火されたイベントを ```subscribe``` するイベントハンドラを登録している｡
    ここで ```msg``` が ```next()``` の引数にセットされたオブジェクトであり､これを自身のプロパティである ```serviceProp``` にセットすることで､ CommonService 経由でデータの共有と更新を実現している｡
    また ```subscription``` に ```subscribe``` のオブジェクトをセットしているのは､後述の ```ngOnDestroy``` で ```subscribe``` の内容を破棄するためである｡

1. データ更新を CommonService へ通知

    ```typescript:データ更新をCommonServiceへ通知
    onClicSendMessage() {
      this.commonService.onNotifySharedDataChanged('Updated by Sample1Component.');
    }
    ```

    画面上で入力されたデータを受け取るイベントハンドラ｡
    ここでは入力されたデータをサービスに送るため､ ```CommonService``` で用意されたメソッド ```onNotifySharedDataChanged()``` をコールしている｡
    ここで引数にセットしたデータが **コンポーネント間で共有するデータ** として扱われる｡

1. コンポーネント破棄時の後処理

    ```typescript:コンポーネント破棄時の後処理
    ngOnDestroy() {
      this.subscription.unsubscribe();
    }
    ```
 
    ```subscribe``` した内容をコンポーネントが破棄されるタイミングで ```unsubscribe``` することで破棄する｡
    これを行わないと､登録したイベントハンドラが延々と生き続けることになる｡

1. テンプレートについて
    テンプレートについては片方向データバインドと入力フォームがあるだけなので特筆する点はなし｡


## sample2.component

```sample2.component``` の実装は ```sample1.component``` とほぼ同じなため､差分箇所のみを抜粋し､説明は割愛する｡
```sample2.component``` のコード詳細については [こちら](https://github.com/ksh-fthr/angular-work/tree/feat_share_service) にアップしたものを参照されたい｡

```typescript:sample2.component.ts(差分のみ抜粋)
// 省略
@Component({
  selector: 'app-sample2',
  templateUrl: './sample2.component.html',
  styleUrls: ['./sample2.component.css']
})
export class Sample2Component implements OnInit, OnDestroy {

  /**
   * CommonService の変数の参照を取得するプロパティ
   *
   * @type {String}
   * @memberof Sample2Component
   */
  public serviceProp: String = 'Initialized by Sample2Component';

  // 省略

  /**
   * ボタンクリック時のイベントハンドラ
   *
   * @memberof Sample2Component
   */
  onClicSendMessage() {
    console.log('[Sample2Component] onClicSendMessage fired.');
    this.commonService.onNotifySharedDataChanged('Updated by Sample2Component.');
  }
}
```

# 実行結果

## 1. 起動直後

![share-service01.png](https://qiita-image-store.s3.amazonaws.com/0/193342/56d7d75e-101e-9b3c-0573-675718837f52.png)

枠で囲った文言は **```sample1.component``` で ｢Initialized by Sample1Component｣ (オレンジ枠) が表示** され､ **```sample2.component``` で ｢Initialized by Sample2Component｣ (青枠) が表示** されている｡

## 2. sample1.component の send message を実行

![share-service02.png](https://qiita-image-store.s3.amazonaws.com/0/193342/70412cc1-bcb3-15c3-ec42-408059a5740e.png)

send message ボタンをクリックした結果､ CommonService 経由で ｢｣ 内の文言が更新されている｡
このとき枠で囲った文言は ```sample1.component```､ ```sample2.component``` 共に **```sample1.component``` でセットしたメッセージである ｢Updated by Sample1Component.｣ が表示** されていることが確認できる｡

## 3. sample2.component の send message を実行

![share-service03.png](https://qiita-image-store.s3.amazonaws.com/0/193342/0ace667a-628a-8eef-9bc1-fa726a619ce6.png)

send message ボタンをクリックした結果､ CommonService 経由で ｢｣ 内の文言が更新されている｡
このとき枠で囲った文言は ```sample1.component```､ ```sample2.component``` 共に **```sample1.component``` でセットしたメッセージである ｢Updated by Sample2Component.｣ が表示**されていることが確認できる｡


# まとめ

本記事で実装したデータ共有の仕組みをまとめると次の流れとなる｡

1. コンポーネントでデータを変更する
1. コンポーネントは変更したデータをメソッドコールによってサービスに渡す
1. サービスは変更したデータを受取り､そのデータをイベント発火することで展開する
1. 各コンポーネントは発火されたイベントを ```subscribe``` し自身のプロパティを更新する

つまり ```サービス <-> コンポーネント``` 間で **メソッドコールとイベントによってデータ共有を実現** していることになる｡

**データ共有** というワードと今回の記事における実装とでは伝わるイメージにズレが感じられるが以上となる｡

関連記事で上げたコンポーネントの参照や @Input/@Output によるデータの受け渡しでは親子関係が必要となるが､本記事の方法では親子関係がなくともデータの共有ができる｡
これはコンポーネント設計時に親子関係の縛りを気にせずに済む､という点でメリットになると思う｡


# ソースコード

今回の記事で動作確認に使用したコードは [ここ](https://github.com/ksh-fthr/angular-work/tree/feat_share_service) にアップしてあるのでご参考まで。
