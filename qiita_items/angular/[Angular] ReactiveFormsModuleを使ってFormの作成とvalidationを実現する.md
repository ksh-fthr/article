# まえおき

[こちらの記事](https://qiita.com/ksh-fthr/items/ee9b026da40cae96ac38#%E8%A4%87%E6%95%B0%E3%81%AE%E5%85%A5%E5%8A%9B%E9%A0%85%E7%9B%AE%E3%81%8C%E3%81%82%E3%82%8B%E3%82%B1%E3%83%BC%E3%82%B9%E3%81%A7%E6%9C%80%E5%BE%8C%E3%81%AB%E5%85%A5%E5%8A%9B%E3%81%97%E3%81%9F%E9%A0%85%E7%9B%AE%E3%81%AE%E3%82%A8%E3%83%A9%E3%83%BC%E3%81%A0%E3%81%91%E8%A1%A8%E7%A4%BA%E3%81%97%E3%81%9F%E3%81%84) で validation のエラー情報を管理する際に、そのトリガーとして ```(keyup)``` を使用する方法を記載したが、これだと期待通りに動作しないケースがあった。
具体的には次のとおり。

## 前提
* 複数の入力項目で発生したエラー情報を管理し、最後に発生したエラーを表示したい
* そのために  ```(keyup)``` をトリガーにエラー情報を管理している

## ```(keyup)``` でうまくいかないケース
* 操作
  * 全角入力で1文字入力 -> Enterで確定する
* 結果
  * 未入力を示す validation エラーとなる
  * 2文字目の入力を行うとパターン不整合による validation エラーとなる
* 期待値
  * 1文字目の入力でパターン不整合による validation エラーとなる

## 対策
上記で発生した問題は [statusChanges](https://angular.io/api/forms/AbstractControlDirective#statusChanges) を利用することで解決､期待値に示す結果となった。
で、```statusChanges``` を利用するには ```ReactiveFormsModule``` を使用してフォームをコンポーネントで作成する必要がある。

本記事では以下の内容で ```statusChanges``` と ```ReactiveFormsModule``` を使った方法について紹介する。

* 本記事で紹介する内容
  * [ReactiveFormsModule を使用したフォームを作成する](#reactiveformsmodule-を使用したフォームを作成する)
  * [statusChanges を使用して validation の状態を監視する](#statuschanges-を使用して-validation-の状態を監視する)

# 更新情報

## 2021/01/10
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


# ReactiveFormsModule を使用したフォームを作成する

```ReactiveFormsModule``` でフォームを作るには次の作業を行う。

1. 前準備
  1. ```app.module.ts``` に ```ReactiveFormsModule``` を ```import``` して ```@NgModule``` の ```imports``` に登録する
  2. フォームを作るコンポーネントで ```FormGroup```, ```FormControl``` を ```import``` する
  3. validation も行いたい場合は ```Validators``` も ```import``` する
2. 実装
  1. ```FormGroup``` から テンプレートで使用するフォームのインスタンスを生成する
  2. このとき、テンプレートのフォームで設定したい項目のインスタンスを ```FormControl``` を指定して生成する
  3. validation を行う場合は、```FromControl``` のインスタンスを生成する際に、コンストラクタの引数に指定することで設定する
  4. 生成した ```FormGourp``` のインスタンスをセットしたプロパティはテンプレートの ```<form>``` タグで ```[formGroup]="form"``` と指定する

以下、具体的な内容をコードでみていく。


## 前準備

1. ```app.module.ts``` の編集

    ```typescript:app.module.ts
    // 省略
    import { ReactiveFormsModule } from '@angular/forms'; // <-- ここを追加
    // 省略
    @NgModule({
      // 省略
      imports: [
        // 省略
        ReactiveFormsModule // <-- ここを追加
      ],
      // 省略
    })
    export class AppModule { }
    ```

1. コンポーネントの編集

    ```typescript:use-directive.component.ts
    // 省略
    import { FormGroup, FormControl, Validators } from '@angular/forms'; // <-- ここを追加

    // 省略
    export class UseDirectiveComponent implements OnInit, OnDestroy {
      // 省略
    }
    ```

## 実装

1. Form の生成

    ```typescript:コンポーネントの編集例.ts
    // 省略
    import { FormGroup, FormControl, Validators } from '@angular/forms';
    import { Subscription } from 'rxjs';
    import { stringify } from 'querystring';

    // 省略
    export class exampleComponent implements OnInit, OnDestroy {

      public form!: FormGroup;  // テンプレートで使用するフォームを宣言

      public ngOnInit() {

        // フォームの生成
        this.form = new FormGroup({

          // フォーム内で使用する項目の生成
          numberControl: new FormControl(
            '',
            [
              Validators.required,            // 必須入力をチェック
              Validators.minLength(7),        // 最少入力桁数をチェック
              Validators.maxLength(15),       // 最大入力桁数をチェック
              Validators.pattern('^[0-9]+$')  // 入力パターンをチェック
            ]
          )
        });
      }

      public onClickOK($event) {
        const inputValue: any = {
          // 入力項目へのアクセスは FormGroup のインスタンスの controls から行う
          // このとき FormControl を生成したときの名前を指定する
          input: this.form.controls['numberControl'].value
        };
        alert('input value: ' + JSON.stringify(inputValue));
      }
    }
    ```

1. テンプレートへの登録

    ```html:テンプレートの編集例.html

    <!--
      [formGroup]="form" でコンポーネントで生成した FormGroup をセット
    -->
    <form #formCheck="ngForm" [formGroup]="form" class="form-check">
      <div class="form-area">
        <div class="input-network-address">
          <label class="label-common">IPアドレス:</label>

          <!--
            [formControl]="form.controls['numberControl']" でコンポーネントで生成した FormControl をセット
          -->
          <input
            type="text"
            class="input-text"
            name="input-ip-address"
            formControlName="numberControl"
          >
        </div>

        <div class="validation-error-information">

          <!--
            *ngIf=form.controls['numberControl'].errors で FormControl のエラー有無をチェックしてエラーが発生していれば表示する
          -->
          <div class="caution" *ngIf="form.controls['numberControl'].errors">
            <div [hidden]="!form.controls['numberControl'].errors?.required">※ 項目が未入力です</div>
            <div [hidden]="!form.controls['numberControl'].errors?.minlength">※ 入力した内容が短すぎます</div>
            <div [hidden]="!form.controls['numberControl'].errors?.maxlength">※ 入力した内容が長すぎます</div>
            <div [hidden]="!form.controls['numberControl'].errors?.pattern">※ 入力した内容に誤りがあります</div>
          </div>
        </div>
      </div>

      <div class="button-area">
        <!--
          form タグの属性にセットしていた #formCheck="ngForm" から formCheck.invalid を判定することでボタンの有効/無効を制御
        -->
        <input type="button" class="button-ok" name="button-ok" value="OK" (click)="onClickOK($event)" [disabled]="formCheck.invalid">
      </div>
    </form>
    ```

## 小まとめ

以上で ```ReactiveFormsModule``` を使用したフォームの作成となる。
ただここまでの内容ではテンプレートで validation を実現する方法とやっていることは変わりなく、ただ入力項目に対する validation チェックを行ってエラーを出しているだけである。

[まえおき](#まえおき) で記した要件を実現するための実装を以降に記載する。


# statusChanges を使用して validation の状態を監視する

[本家のリファレンス](https://angular.io/api/forms/AbstractControlDirective#statusChanges) をみると ```statusChanges``` の型は ```Observable<any> | null``` となっている。
ということで、```statusChanges の subscribe``` を利用すれば、 **validation の状態を監視** できる。

コード例を挙げると次のような感じになる。

## statusChanges を利用したコード例

1. validation の状態を監視するサンプル

    ```typescript:コンポーネントの編集例.ts
    // 省略
    export class exampleComponent implements OnInit, OnDestroy {

      public form!: FormGroup;  // テンプレートで使用する form を宣言
      private statusChangesSubscription!: Subscription; // statusChanges イベントを保持する subscription

      public ngOnInit() {

        // form の生成
        this.form = new FormGroup({

          // form 内で使用する項目の生成
          numberControl: new FormControl(
            '',
            [
              Validators.required,            // 必須入力をチェック
              Validators.minLength(7),        // 最少入力桁数をチェック
              Validators.maxLength(15),       // 最大入力桁数をチェック
              Validators.pattern('^[0-9]+$')  // 入力パターンをチェック
            ]
          )
        });

        // -------------------------------------------//
        // 入力フォームの validation の状態を監視する    //
        // -------------------------------------------//
        this.statusChangesSubscription = this.networkForm.controls['ipControl'].statusChanges.subscribe((data) => {
          // エラー情報に対して行いたい処理...
          // なおエラー情報は
          //   form Group のインスタンス.controls['FormControl を生成したときの名前'].errors
          // で取得できる
        });

        // 省略
      }

      public ngOnDestroy() {
        // イベント情報が残り続けるのを防ぐためにコンポーネントの終了処理で破棄する
        this.statusChangesSubscription.unsubscribe();
      }
    }
    ```

## statusChanges を利用してエラー情報を管理する

要件は [まえおき](#まえおき) に記載したとおり 「**複数の入力項目で発生したエラー情報を管理し、最後に発生したエラーを表示したい**」 となる。
エラー管理の実装内容は、これも [まえおき](#まえおき) で挙げた参照記事と変わらない。

変更点は

* **状態の監視に ```statusChanges``` を利用する**
* **それに合わせて ```(keyup)``` の利用を廃止する**

といったところ。

では、実際のコードを示す。
( 上記までで挙げたサンプルコードとはプロパティ名やメソッド名が異なるので注意 )

1. コンポーネント

    ```typescript:reactive-form.component.ts
    import { Component, OnInit, OnDestroy } from '@angular/core';
    import { FormGroup, FormControl, Validators } from '@angular/forms';
    import { Subscription } from 'rxjs';
    import { stringify } from 'querystring';

    @Component({
      selector: 'app-reactive-form',
      templateUrl: './reactive-form.component.html',
      styleUrls: ['./reactive-form.component.css']
    })
    export class ReactiveFormComponent implements OnInit, OnDestroy {

      public networkForm!: FormGroup; // ビューで定義する Form の本体

      // validation の属性
      public readonly minNetworkAddressLength: number = 7;
      public readonly maxNetworkAddressLength: number = 15;
      public readonly networkAddressPattern: string = '^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])[¥.]){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])$';

      public validationError: any; // 入力エラー情報を画面に表示するためのプロパティ
      private validationErrorList: any = []; // 入力エラー情報を管理するためのリスト

      // 入力 form で発生した statusChanges イベントを保持する subscription
      private ipControlSubscription!: Subscription;
      private subnetmaskSubscription!: Subscription;

      constructor() { }

      public ngOnInit() {

        // 複数の入力項目を設置する form を生成する
        this.networkForm = new FormGroup({
          // IPアドレスの入力項目
          ipControl: new FormControl(
            '',
            [
              Validators.required,                                 // 必須入力をチェック
              Validators.minLength(this.minNetworkAddressLength),  // 最少入力桁数をチェック
              Validators.maxLength(this.maxNetworkAddressLength),  // 最大入力桁数をチェック
              Validators.pattern(this.networkAddressPattern)       // 入力パターンをチェック
            ]
          ),
          // サブネットマスクの入力項目
          subnetmaskControl: new FormControl(
            '',
            [
              Validators.required,                                 // 必須入力をチェック
              Validators.minLength(this.minNetworkAddressLength),  // 最少入力桁数をチェック
              Validators.maxLength(this.maxNetworkAddressLength),  // 最大入力桁数をチェック
              Validators.pattern(this.networkAddressPattern)       // 入力パターンをチェック
            ]
          ),
        });

        // form 生成時に指定した入力項目( FormControl ) 分、validation の状態を監視する
        // 状態が変化したらイベントをキャッチするので、それをトリガーにエラー情報の管理を行う
        this.ipControlSubscription = this.networkForm.controls['ipControl'].statusChanges.subscribe((data) => {
          this.manageValidationError('ipControl', this.networkForm.controls['ipControl'].errors);
        });

        this.subnetmaskSubscription = this.networkForm.controls['subnetmaskControl'].statusChanges.subscribe((data) => {
          this.manageValidationError('subnetmaskControl', this.networkForm.controls['subnetmaskControl'].errors);
        });

        // おまけ: 入力 form の有効/無効を制御する
        this.changeFormEnabledDisabled();
      }

      public ngOnDestroy() {
        // イベント情報が残り続けるのを防ぐためにコンポーネントの終了処理で破棄する
        this.ipControlSubscription.unsubscribe();
        this.subnetmaskSubscription.unsubscribe();
      }

      private changeFormEnabledDisabled() {

        // この例では常に「有効」となる

        // 有効にしたい場合
        this.networkForm.controls['ipControl'].enable();
        this.networkForm.controls['subnetmaskControl'].enable();
        // 無効にしたい場合
        // this.networkForm.controls['ipControl'].disable();
        // this.networkForm.controls['subnetmaskControl'].disable();
      }

      private manageValidationError(validationKey, errorInformation) {

        // validation のエラー情報を管理する
        // 常に最後に発生したエラー情報をビューに表示するため、リストから最後の要素を取得して
        // テンプレートから参照されるプロパティにセットしている

        for (const target in this.validationErrorList) {
          if (this.validationErrorList.hasOwnProperty(target) &&
              this.validationErrorList[target].key === validationKey) {
            this.validationErrorList.splice(target, 1);
            break;
          }
        }

        if (errorInformation) {
          this.validationErrorList.push({ key: validationKey, error: errorInformation });
        }

        const errorData: any = this.validationErrorList[this.validationErrorList.length - 1];
        this.validationError = errorData ? errorData.error : undefined;
      }
    }
    ```

1. テンプレート

    ```html:reactive-form.component.html
    <!--
      [formGroup]="networkForm" でコンポーネントで生成した FormGroup をセット
    -->
    <form #formCheck="ngForm" [formGroup]="networkForm" class="form-check">
      <div class="form-area">
        <div class="input-network-address">
          <label class="label-common">IPアドレス:</label>
          <!--
            [formControl]="networkForm.controls['ipControl']" でコンポーネントで生成した FormControl をセット
          -->
          <input
            type="text"
            class="input-text"
            name="input-ip-address"
            formControlName="ipControl"
          >
        </div>
        <div class="input-network-address">
          <label class="label-common">サブネットマスク:</label>
          <!--
            [formControl]="networkForm.controls['subnetmaskControl']" でコンポーネントで生成した FormControl をセット
          -->
          <input
            type="text"
            class="input-text"
            name="input-subnet-mask"
            formControlName="subnetmaskControl"
          >
        </div>
        <div class="validation-error-information">

          <!--
            *ngIf=validationError で validation のエラー有無をチェックしてエラーが発生していれば表示する
            (networkForm.dirty || networkForm.touched) は一回でも入力があったかを判断する
          -->
          <div class="caution" *ngIf="validationError && (networkForm.dirty || networkForm.touched)">
            <div [hidden]="!validationError?.required">※ 項目が未入力です</div>
            <div [hidden]="!validationError?.minlength">※ 入力した内容が短すぎます</div>
            <div [hidden]="!validationError?.maxlength">※ 入力した内容が長すぎます</div>
            <div [hidden]="!validationError?.pattern">※ 入力した内容に誤りがあります</div>
          </div>
        </div>
      </div>

      <div class="button-area">
        <!--
          form タグの属性にセットしていた #formCheck="ngForm" から formCheck.invalid を判定することでボタンの有効/無効を制御
        -->
        <input type="button" class="button-ok" name="button-ok" value="OK" (click)="onClickOK($event)" [disabled]="formCheck.invalid">
      </div>
    </form>
    ```

# 実行結果

ここでは [まえおき](#まえおき) の要件の確認ということで､全角文字1文字の確認結果のみを示す｡

1. IP アドレスでデータ不正エラー( 1文字目を全角文字 )
    ![01.ip-zen-error.png](https://qiita-image-store.s3.amazonaws.com/0/193342/73b43fe6-c83c-6f34-9147-b9cf2e60f8c8.png)

1. サブネットマスクでデータ不正エラー( 1文字目を全角文字 )
    ![02.subnet-zen.error.png](https://qiita-image-store.s3.amazonaws.com/0/193342/37ad1401-8537-63aa-cc9a-7adfa4797449.png)


# まとめ

本記事の内容を簡単にまとめると

* ```statusChanges``` で validation の状態を監視することができる
* ```statusChanges``` を使うには ```ReactiveFormsModule``` でフォームを作成する必要がある
* フォームを作る際は ```FormGroup``` と ```FromControl``` のインスタンスを生成してテンプレートにセットする
* 生成したフォームに対して validation の状態を監視するには､ ```FromControl``` に対して ```statusChanges``` を ```subscribe``` してイベント登録する
* フォームの有効/無効を切り替える際は ```enable()``` と ```disable()``` を使用する

となる｡


# ソースコード

今回の記事で作成したコードは [こちら](https://github.com/ksh-fthr/angular-work/tree/feat_reactive_form) にアップしてあるのでご参考まで。


# 謝辞

validation のエラーハンドリングについて [こちら](https://spectrum.chat/?t=a28cf5d5-f1aa-421b-a735-5edacf5a4ba6) で質問をしたところ､ @Quramy 様に [statusChanges](https://angular.io/api/forms/AbstractControlDirective#statusChanges) をご教示いただきました｡

あらためて御礼申し上げます｡
ありがとうございました｡
