# はじめに
Angular では標準で入力フォームの入力値をチェックする validation 機能が搭載されている。
本記事ではこの validation 機能を使用して

* 必須入力チェック
* 長さチェック
* パターンチェック
* 最小値, 最大値チェック

を行う方法を確認する。

# 更新情報

## 2022/07/10
- [Angular v12 で追加された `min`, `max` の validation](https://github.com/angular/angular/pull/39063) の検証コードを追加しました
- 記事内で扱ったコードを Angular `v13.2.3` で確認しました

## 2021/01/06
- 記事内で扱ったコードを Angular `v11.0.5` で確認しました

# 作業環境

| 環境                                          | バージョン                       | 備考               |
| --------------------------------------------- | -------------------------------- | ------------------ |
| [Angular CLI](https://cli.angular.io/)        | ~~v6.0.0~~ ~~v11.0.5~~ v13.2.4   | `$ ng --version`   |
| [Angular](https://angular.io/)                | ~~v6.0.0~~ ~~v11.0.5~~ v13.2.3   | 同上               |
| [TypeScript](https://www.typescriptlang.org/) | ~~v4.0.2~~ v4.5.5                | 同上               |
| [Node.js](https://nodejs.org/ja/)             | ~~v9.2.1~~ ~~v12.18.3~~ v14.17.0 | `$ node --version` |
| [npm](https://www.npmjs.com/)                 | ~~v6.1.0~~ ~~v6.14.6~~ v6.14.13  | `$ npm --version`  |


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
    

Angular CLI: 13.2.4
Node: 14.17.0
Package Manager: npm 6.14.13
OS: darwin x64

Angular: 13.2.3
... animations, common, compiler, compiler-cli, core, forms
... platform-browser, platform-browser-dynamic, router

Package                         Version
---------------------------------------------------------
@angular-devkit/architect       0.1302.4
@angular-devkit/build-angular   13.3.7
@angular-devkit/core            13.2.4
@angular-devkit/schematics      13.2.4
@angular/cdk                    13.3.9
@angular/cli                    13.2.4
@angular/material               13.3.9
@schematics/angular             13.2.4
rxjs                            6.6.0
typescript                      4.5.5
```

</div>
</details>


# ```FormsModule``` の import

Angular ではフォームを作成する際、```FormsModule``` を ```app.module.ts``` で import しておく必要がある。

```typescript:app.module.ts
// 省略
import { FormsModule } from '@angular/forms';

// 省略
@NgModule({
  declarations: [
    // 省略
    ValidationComponent,
  ],
  // 省略
})
export class AppModule { }
```
これでフォーム作成の準備が整った｡以降は実際に Angular の標準機能を利用した validation の実現方法についてみていく｡


# validation を設定

validation は基本的にテンプレート側で設定する。
次のコードは テキストボックスに対して validation を設定したテンプレート。

```html:validation-verification.component.html(validationを設定したテンプレート)
<form #formCheck="ngForm" class="formCheck">
  <div class="input-network-address">
    <label>IPアドレス:</label>
    <input
      type="text"
      class="input-data-column"
      name="input-ip-address"
      [(ngModel)]="inputIP"
      #inputIPinfo="ngModel"
      required
      minlength="{{minNetworkAddressLength}}"
      maxlength="{{maxNetworkAddressLength}}"
      pattern="{{networkAddressPattern}}"
    >
    <div class="note" *ngIf="inputIPinfo.errors && (inputIPinfo.dirty || inputIPinfo.touched)">
      <div [hidden]="!inputIPinfo.errors.required">※ 項目が未入力です</div>
      <div [hidden]="!inputIPinfo.errors.minlength">※ 入力した内容が短すぎます</div>
      <div [hidden]="!inputIPinfo.errors.maxlength">※ 入力した内容が長すぎます</div>
      <div [hidden]="!inputIPinfo.errors.pattern">※ 入力した内容に誤りがあります</div>
    </div>
  </div>
  <input type="button" class="button_ok" name="button_ok" value="OK" (click)="onClickOK($event)" [disabled]="formCheck.invalid">
</form>
```

以下、内容を具体的に見ていく。


## フォームの設置

まずは ```form``` タグによるフォーム要素を設置する。
ポイントは

* **```#formCheck="ngForm"```**

とすることで ```テンプレート参照変数``` に ```ngForm``` ディレクティブをセットしていること。
これによって **フォーム全体の状態を監視** することができる。
ここで宣言したテンプレート参照変数 ```formCheck``` は、後述の ```button``` の有効/無効を制御する際に利用する。


## validation のための属性を設定

次にテキスト入力を行うための ```input``` タグの属性について。
この ```input``` タグでは validation のための属性を複数設定しているので、各項目について細かくみていく。
なお minlength、 maxlength、 pattern にはクラスファイルで設定したプロパティを片方向データバインドでセットしている。

* **```name="input-ip-address"```**
  * この属性の設定は必須
  * フォーム要素を識別するためのキーとしてセットする
* **```[(ngModel)]="inputIP"```**
  * この属性の設定は必須
  * フォーム要素とクラスファイルのプロパティを紐付ける
* **```#inputIPinfo="ngModel"```**
  * この属性の設定は必須
  * フォーム要素を参照するために ```テンプレート参照変数``` にフォーム要素(ngModel) をセットする
* **```required```**
  * この属性の設定は任意
  * 必須入力項目であることを示す
  * これを指定することで未入力の際にエラーを出すことができる
* **```minlength="{{minNetworkAddressLength}}"```**
  * この属性の設定は任意
  * 最小文字数をチェックする
  * これを指定することで入力された文字数が最小文字数以下の際にエラーを出すことができる
* **```maxlength="{{maxNetworkAddressLength}}"```**
  * この属性の設定は任意
  * 最大文字数をチェックする
  * これを指定することで入力された文字数が最大文字数以上の際にエラーを出すことができる
* **```pattern="{{networkAddressPattern}}"```**
  * この属性の設定は任意
  * 入力された文字列のパターンをチェックする
  * 正規表現を用いることも可能
  * これを指定することで入力された文字列がパターンに合致しない際にエラーを出すことができる

## エラー表示

次に ```class="note"``` の ```div``` 要素について。
ここでは ```input``` タグで設定した validation の検証結果によるエラーを表示する。

* **```*ngIf="inputIPinfo.errors"```**
  * テンプレート参照変数 ```inputIPinfo``` から ```errors``` を参照
  * 且つ ```dirty``` と ```touched``` を参照
  * ```errors``` が ```true``` 且つ ```dirty``` と ```touched``` のいずれかが ```true``` の場合に ```class="caution"``` の ```div``` 要素が有効になる
  * ```dirty``` と ```touched``` を参照/判定している理由は、<font color="red">初期表示時にエラーにならないようにする</font>ため
  * 逆に初期表示時に未入力エラーを明示したい場合、 ```&& (inputIPinfo.dirty || inputIPinfo.touched)``` の部分を削除すれば良い
* **```[hidden]="!inputIPinfo.errors.required"```**
  * 未入力の際にエラーを表示する
* **```[hidden]="!inputIPinfo.errors.minlength"```**
  * 入力された文字数が最小文字数以下の際にエラーを表示する
* **```[hidden]="!inputIPinfo.errors.maxlength```**
  * 入力された文字数が最大文字数以以上の際にエラーを表示する
* **```[hidden]="!inputIPinfo.errors.pattern"```**
  * 入力された文字列がパターンに合致しない際にエラーを表示する

## ボタンの有効/無効

最後に OK ボタンの有効/無効について確認する。
ここでのポイントは

* **```[disabled]="formCheck.invalid"```**

の部分で、ボタンの有効/無効の制御に ```テンプレート変数参照変数``` である ```formCheck``` を使用していること。
前述のとおり ```form``` 要素に ```#formCheck="ngForm"``` をセットしたことで、フォーム全体の状態を ```formCheck``` が持っている。
これによって ```formCheck``` の ```invalid``` プロパティを参照することで

* フォーム全体に問題がなければ false
* フォーム全体に問題があれば true

をチェックすることが可能になる。
で、それを ```[disable]``` にセットすることで、

* 入力された内容に問題がなければ OK ボタンを有効(押下可能)にし、問題があれば無効(押下不可)にする

制御を実現させている。


# validation の条件を設定

validation の条件はクラスファイルに記述する。
とはいえ、ここではテンプレートにおいて双方向、もしくは片方向データーバインドによって参照していたプロパティを用意しているだけで、詳解することもないのでコード中のコメントをご覧いただければと思う。

```typescript:validation-verification.component.ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-validation-verification',
  templateUrl: './validation-verification.component.html',
  styleUrls: ['../../../style/common.css'],
})
export class ValidationVerificationComponent implements OnInit {

  /**
   * validaion のための双方向データバインドを行うプロパティ
   *
   * @type {string}
   */
  public inputIP: string = '';

  /**
   * 入力された内容の最小文字数チェック
   * IPアドレスの最小文字数である x.x.x.x の ７ を定義
   *
   * @type {number}
   */
  public readonly minNetworkAddressLength: number = 7;

  /**
   * 入力された内容の最大文字数チェック
   * IPアドレスの最大文字数である xxx.xxx.xxx.xxx の 15 を定義
   *
   * @type {number}
   */
  public readonly maxNetworkAddressLength: number = 15;

  /**
   * 入力された内容のパターンチェック
   * 正規表現で x.x.x.x ~ xxx.xxx.xxx.xxx のパターンマッチングを実現させる
   *
   * @type {string}
   */
  public readonly networkAddressPattern: string = '^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])[¥.]){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])$';

  constructor() { }

  ngOnInit() {
  }

  /**
   * OKボタンがクリックされた時のイベントハンドラ
   * ここでは単純にアラートを出すだけ
   *
   * @param {any} $event イベント情報
   * @memberof ValidationComponent
   */
  public onClickOK($event: any) {
    alert('OK button had clicked.');
  }
}
```

# 複数の入力項目があるケースで最後に入力した項目のエラーだけ表示したい

上記までは入力項目をひとつだけ設定している状態での説明となる｡
だが実際に入力項目を設ける場合､複数の項目を設定することが殆どだろう｡

本項目では **入力項目を複数設定** し､そのエラー内容が共通のケースにおいて､項目ごとにエラー情報を表示するのではなく､ **最後にエラーが発生した項目のエラー情報を表示したい** ケースの対応例を載せる｡
また本項目では Angular v12 で追加された `min`, `max` の validation についても記載する｡

## 複数の入力項目を設置

```html:validation.component.html(複数の入力項目を設定)
<form #formCheck="ngForm" class="form-check">
  <div class="validation-form-area">
    <p>require, minLength, maxLength, pattern のチェック</p>
    <div class="input-network-address">
      <label class="label-common">IPアドレス: </label>
      <input
        type="text"
        class="input-data-column"
        name="input-ip-address"
        [(ngModel)]="inputIP"
        #inputIPinfo="ngModel"
        required
        minlength="{{minNetworkAddressLength}}"
        maxlength="{{maxNetworkAddressLength}}"
        pattern="{{networkAddressPattern}}"
        (keyup)="onKeyUp('inputIPinfo', inputIPinfo.errors)"
      >
    </div>
    <div class="input-network-address">
      <label class="label-common">サブネットマスク: </label>
      <input
        type="text"
        class="input-data-column"
        name="input-subnet-mask"
        [(ngModel)]="inputSubnetMask"
        #inputSubnetMaskInfo="ngModel"
        required
        minlength="{{minNetworkAddressLength}}"
        maxlength="{{maxNetworkAddressLength}}"
        pattern="{{networkAddressPattern}}"
        (keyup)="onKeyUp('inputSubnetMaskInfo', inputSubnetMaskInfo.errors)"
      >
    </div>
  </div>
  <div class="validation-form-area">
    <p>数値の最小/最大値のチェック</p>
    <div class="input-number">
      <label class="label-common">数値: </label>
      <input
        type="number"
        class="input-data-column"
        name="input-number"
        [(ngModel)]="inputNumber"
        #inputNumberInfo="ngModel"
        min="0"
        max="10"
        (keyup)="onKeyUp('inputNumberInfo', inputNumberInfo.errors)"
      >
    </div>
  </div>
  <div class="validation-error-area">
    <p>
      入力に誤りがある場合は下記にエラー内容が表示されます。
    </p>
    <div class="validation-error-information">
      <div class="note" *ngIf="validationError">
        <div [hidden]="!validationError?.required">※ 項目が未入力です</div>
        <div [hidden]="!validationError?.minlength">※ 入力した内容が短すぎます</div>
        <div [hidden]="!validationError?.maxlength">※ 入力した内容が長すぎます</div>
        <div [hidden]="!validationError?.pattern">※ 入力した内容に誤りがあります</div>
        <div [hidden]="!validationError?.min">※ 入力された数値が小さすぎます</div>
        <div [hidden]="!validationError?.max">※ 入力された数値が大きすぎます</div>
      </div>
    </div>
  </div>
</form>
```

[validation を設定](#validation-を設定) で載せたコードとの変更点は

* サブネットマスクの入力項目を設けていること
* 数値の最小値､最大値のチェックとして `min`, `max` を設けていること
* エラーの表示領域で用いている( ```*ngIf``` で参照している )変数が ```テンプレート参照変数: inputIPinfo``` ではなく ```validationError``` となっていること( ```validationError``` はクラスで定義しているプロパティ )

となる｡
あとは細かな HTML の構成や CSS セレクタの変更を行っているが､そこの説明は割愛する｡


## `min`, `max` の説明( 2022/07/10 追記 )
### validation のための属性を設定

* **```min="0"```**
  * この属性の設定は任意
  * 入力された数値の最小値をチェックする
  * これを指定することで入力された数値が最小値を下回った際にエラーを出すことができる
* **```max=10"```**
  * この属性の設定は任意
  * 入力された数値の最大値をチェックする
  * これを指定することで入力された数値が最大値を上回った際にエラーを出すことができる

### エラー表示

* **```[hidden]="!validationError?.min"```**
  * 入力された数値が最小値を下回った際にエラーを表示する
* **```[hidden]="!validationError?.max"```**
  * 入力された数値が最大値を上回った際にエラーを表示する


## エラー情報をリストで管理する

```typescript:validation-verification.component.ts(抜粋)

  /**
   * 入力エラー情報を画面に表示するためのプロパティ
   *
   * @type {*}
   */
  public validationError: any;

  /**
   * 入力エラー情報を管理するためのリスト
   *
   * @private
   * @type {*}
   */
  private validationErrorList: any = [];

  /**
   * keyup イベントのイベントハンドラ
   * このイベントをトリガーに入力エラー情報を管理する
   *
   * @param validationKey
   * @param errorInformation
   */
  public onKeyUp(validationKey: any, errorInformation: any) {
    this.manageValidationError(validationKey, errorInformation);
  }

  /**
   * 入力エラー情報を管理する
   * 具体的には次の処理を行う
   * # 引数の validationKey を元にエラー情報を一意に管理する
   * # ビューに表示するためのエラー情報にリストの最後の情報をセットする
   *
   * @private
   * @param validationKey エラーが発生した入力フォーム
   * @param errorInformation バリデーションエラー情報
   */
  private manageValidationError(validationKey: any, errorInformation: any) {
    for (const target in this.validationErrorList) {
      if (this.validationErrorList.hasOwnProperty(target) && this.validationErrorList[target].key === validationKey) {
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
```

ポイントは次のとおり｡

* ```onKeyUp``` でテンプレートの入力項目から発火された ```keyup``` イベントをハンドリングする
* エラー情報を管理するリストにイベント発火時のエラー情報をセットする
  * このときイベント発火時に受け取った ```validationKey``` で一意に管理する
* テンプレートの ```*ngIf``` で参照しているプロパティ ```validationError``` にリストの最後のエラー情報をセットする
  * このときリストがらデータが取得できなかった場合は ```undefined``` がセットされる( validation エラーが発生していないということで画面上からエラー情報が消える )


## 実行結果
### 1. IP アドレスで未入力エラー
![スクリーンショット 2022-07-10 19.23.21.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/ec152c13-fb92-179d-4fa4-8a1c06c0106a.png)

### 2. サブネットマスクでデータ不正エラー
![スクリーンショット 2022-07-10 19.24.08.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/4b0fae5e-6b46-450f-6c6b-70c8bb2d1a44.png)

### 3. IP アドレスで再度未入力エラー
![スクリーンショット 2022-07-10 19.24.55.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/f12827cd-2fb8-b63e-eae9-392812305cb5.png)

### 4. 数値入力の最小値でエラー
![スクリーンショット 2022-07-10 19.25.39.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/fa5a7d64-44d1-0d61-c4aa-69bb96896eda.png)

### 5. 数値入力の最大値でエラー
![スクリーンショット 2022-07-10 19.25.56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/d9df493c-ee87-698e-6f0c-2fa1f8f54666.png)

### 6. エラー解消
![スクリーンショット 2022-07-10 19.26.12.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/c5642798-1c6b-77a7-2a1a-688fe6b3b16a.png)


## 本項目についての備考

なお上記の方法はエラー情報の管理をコンポーネントで行っているため､ [[Angular] 属性ディレクティブにvalidation機能を設ける](https://qiita.com/ksh-fthr/items/22fe5be4ff3c3467cb85) で示したコンポーネントにも適用できる｡ [そちらのコード](https://github.com/ksh-fthr/angular-work/tree/feat_directive_validation) もご興味あれば参照いただきたい｡

もっとスマートなやり方や､そもそもロジックでゴリゴリやるのではなく､ Angular で仕組みが用意されているかもしれないので､そのあたりは今後も調べていき､より良いやり方が見つかったら更新する｡

## 上記コードの不具合について

```(keyup)``` をトリガーに validation のエラー情報を管理しているが､これだと全角文字を1文字だけ入力した際に **未入力エラー** となってしまう｡( 期待値はパターン不整合のエラーとなること )
これは [statusChanges](https://angular.io/api/forms/AbstractControlDirective#statusChanges) を用いれば解決できたので､その方法を [こちら](https://qiita.com/ksh-fthr/items/b5546c50129c60b883ba) で記事にした｡
ご興味あれば参照されたい｡

# 参考

* [NgForm](https://angular.io/api/forms/NgForm)
* [Form Validation](https://angular.io/guide/form-validation)

また、本記事では Angular 標準の validation 機能を利用した入力値チェックについて確認したが、それだけではカバーできないケースの場合、本家の次のセクションが参考になると思う。

* [Custom validators](https://angular.io/guide/form-validation#custom-validators)


# ソースコード

今回の記事で作成したコードのブランチは [こちら](https://github.com/ksh-fthr/angular-work/tree/feature/validation)､ コードは同ブランチの [こちら](https://github.com/ksh-fthr/angular-work/tree/feature/validation/src/app/component/validation/validation-verification) にアップしてあるのでご参考まで。

