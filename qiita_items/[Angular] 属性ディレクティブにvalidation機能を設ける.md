# はじめに
[こちらの記事](https://qiita.com/ksh-fthr/items/ee9b026da40cae96ac38) でコンポーネントでの検証機能の実現について触れたが､ディレクティブでも検証機能を実現できる｡
本記事ではその方法についてみていく｡

# 更新情報

## 2021/01/09
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


# 関連記事

* [[Angular] 標準機能を利用して validation を実現する](https://qiita.com/ksh-fthr/items/ee9b026da40cae96ac38)
* [[Angular] 属性ディレクティブを自作する](https://qiita.com/ksh-fthr/items/b8e3577f47483f5685e2)

# 本記事で扱う構成

```bash:構成
src/
  `-app/
    `-component/
    |  `-use-directive/                           # ディレクティブを適用するコンポーネント
    |  `- use-directive.component.css
    |  `- use-directive.component.html
    |  `- use-directive.component.ts
    `-directive/
    |  `- network-address-validator.directive.ts  # 本記事で扱うディレクティブ
    `- app.component.css
    `- app.component.html
    `- app.component.ts
    `- app.module.ts
```

# 属性ディレクティブで validation を実装

```typescript:network-address-validator.directive.ts
import { Directive, Input } from '@angular/core';
import { AbstractControl, NG_VALIDATORS, Validator } from '@angular/forms';

@Directive({
  selector: '[appNetworkAddressValidator][ngModel]',
  providers: [
    {provide: NG_VALIDATORS, useExisting: NetworkAddressValidatorDirective, multi: true}
  ]
})
export class NetworkAddressValidatorDirective implements Validator {

  /**
   * 入力された内容の最小文字数チェック
   * アドレスの最小文字数である x.x.x.x の ７ を定義
   *
   * @type {number}
   * @memberof ValidationComponent
   */
  public readonly minLength: number = 7;

  /**
   * 入力された内容の最大文字数チェック
   * アドレスの最大文字数である xxx.xxx.xxx.xxx の 15 を定義
   *
   * @type {number}
   * @memberof ValidationComponent
   */
  public readonly maxLength: number = 15;

  /**
   * 入力された内容のパターンチェック
   * 正規表現で x.x.x.x ~ xxx.xxx.xxx.xxx のパターンマッチングを実現させる
   *
   * @type {string}
   * @memberof ValidationComponent
   */
  public readonly pattern: string = '^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])[¥.]){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])$';

  /**
   * 入力値に対する validation を実施する
   *
   * @param {AbstractControl} control 検証対象の form 要素
   * @returns {{[key: string]: any}} 検証結果
   * @memberof ValidationDirective
   */
  validate(control: AbstractControl): {[key: string]: any} {

    const address: string = control.value;
    if (!address) {
      return { required: true };
    }

    if (address.length < this.minLength) {
      return { minLength: true };
    }

    if (address.length > this.maxLength) {
      return { maxLength: true };
    }

    const patternRegex: any = new RegExp(this.pattern);
    if (!patternRegex.test(address)) {
      return { pattern: true };
    }

    return {};
  }
}
```

ポイント大きく分けて 3つ｡

1. validation に必要なモジュールの import と implements
1. @Directive デコレータでディレクティブ情報を定義する
1. validation の実装

以下､上から順にみていく｡


## validation に必要なモジュールの import と implements

```typescript:モジュールのimportとimplements
import { AbstractControl, NG_VALIDATORS, Validator } from '@angular/forms';

// 中略
export class NetworkAddressValidatorDirective implements Validator {
  // 省略
}
```

```import``` している各モジュールに関する簡単な説明は以下のとおり。

* AbstractControl
  * 検証対象のコントロールが継承している抽象クラスで
  * 後述の ```validate``` メソッドで受け取る引数の型として使用している
* NG_VALIDATORS
  * 標準の検証ルール
  * 独自の検証ルールを儲けたい場合は後述の ```@Directive``` デコレータで行なっているように、これに検証クラスを追加する
* Validator
  * 検証ルールを定義するためのインターフェイス
  * これを implements することで後述の ```validate``` メソッドで検証ルールを実装できる


## @Directive デコレータでディレクティブ情報を定義する

```typescript:@Directiveの定義
@Directive({
  selector: '[appNetworkAddressValidator][ngModel]',
  providers: [
    {provide: NG_VALIDATORS, useExisting: NetworkAddressValidatorDirective, multi: true}
  ]
})
```

```@Directive``` デコレータで検証機能に関する情報を定義する。
各パラメータに関する説明は次のとおり。

* selector
  * ```appNetworkAddressValidator``` はコンポーネントのテンプレートで指定する文字列を定義する
  * ポイントは次の ```ngModel`` で、これが指定されていないと検証機能が動作しない
* providers
  * ここで設定している意味は「標準の検証ルールである ```NG_VALIDATORS``` に ```NetworkAddressValidatorDirective``` で定義して **検証ルールを追加** する」となる
  * ```multi``` パラメータに ```true``` がセットされている点に注目で、これが定義されていないと「**標準の検証ルールを上書き**」する動きとなってしまう


## validation の実装

```typescript:validationの実装

  /**
   * 入力値に対する validation を実施する
   *
   * @param {AbstractControl} control 検証対象の form 要素
   * @returns {{[key: string]: any}} 検証結果
   * @memberof ValidationDirective
   */
  validate(control: AbstractControl): {[key: string]: any} {

    const address: string = control.value;
    if (!address) {
      return { required: true };
    }

    if (address.length < this.minLength) {
      return { minLength: true };
    }

    if (address.length > this.maxLength) {
      return { maxLength: true };
    }

    const patternRegex: any = new RegExp(this.pattern);
    if (!patternRegex.test(address)) {
      return { pattern: true };
    }

    return {};
  }
```

検証機能の実装部分。処理の流れは次のとおり。

* 引数: ```control: AbstractControl```
  * 本ディレクティブを適用しているテンプレートのフォーム要素を受け取る
* 検証処理
  * 引数で受け取った ```control``` から ```control.value``` で値を取り出して検証処理を実行する
  * この例では「未入力チェック」「最小文字列チェック」「最長文字列チェック」「正規表現によるパターンチェック」を行っている
  * エラー時はここで返却したパラメータを利用しているテンプレートでチェックすることで、エラー内容に応じたメッセージを出力することになる


# コンポーネントからディレクティブを利用する

コンポーネントからディレクティブの検証機能を利用するコードは次のようになる。

## テンプレート

```html:use-directive.component.html
<form #formCheck="ngForm" class="formCheck">
  <div class="input_ip_address">
    <label>IPアドレス:</label>
    <input
      type="text"
      class="input_text"
      name="input_ip_address"
      [(ngModel)]="inputAddress"
      #validateAddress="ngModel"
      required
      appNetworkAddressValidator
    >
    <div class="caution">
      <div *ngIf="validateAddress.errors && (validateAddress.dirty || validateAddress.touched)">
        <div [hidden]="!validateAddress.errors.required">
          ※ 項目が未入力です
        </div>
        <div [hidden]="!validateAddress.errors.minLength">
          ※ 入力した内容が短すぎます
        </div>
        <div [hidden]="!validateAddress.errors.maxLength">
          ※ 入力した内容が長すぎます
        </div>
        <div [hidden]="!validateAddress.errors.pattern">
          ※ 入力した内容に誤りがあります
        </div>
      </div>
    </div>
  </div>
  <input type="button" class="button_ok" name="button_ok" value="OK" (click)="onClickOK($event)" [disabled]="formCheck.invalid">
</form>
```

ポイントは　```input``` タグの部分。
ここで ```appNetworkAddressValidator``` とディレクティブを指定することで、```input フォーム``` の内容が　```NetworkAddressValidatorDirective``` に送られる。

```input``` タグの各属性の説明については関連記事にあげた [[Angular] 標準機能を利用して validation を実現する#valildation のための属性を設定](https://qiita.com/ksh-fthr/items/ee9b026da40cae96ac38#valildation-%E3%81%AE%E3%81%9F%E3%82%81%E3%81%AE%E5%B1%9E%E6%80%A7%E3%82%92%E8%A8%AD%E5%AE%9A) を参照いただきたい。

なお上記参考記事にある「minlength」「maxlength」「pattern」については、今回の記事ではディレクティブ側で実装しているので ```input``` タグの属性から取り除いている。


## クラス

```typescript:use-directive.component.ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-use-directive',
  templateUrl: './use-directive.component.html',
  styleUrls: ['./use-directive.component.css']
})
export class UseDirectiveComponent implements OnInit {

  /**
   * 入力されたアドレス
   *
   * @type {string}
   * @memberof UseDirectiveComponent
   */
  public inputAddress: string = '';

  /**
   * コンストラクタ ( 本コンポーネントではなにもしない )
   *
   * @memberof UseDirectiveComponent
   */
  constructor() { }

  /**
   * 初期処理 ( 本コンポーネントではなにもしない )
   *
   * @memberof UseDirectiveComponent
   */
  ngOnInit() { }

  /**
   * OKボタンがクリックされた時のイベントハンドラ
   * ここでは単純にアラートを出すだけ
   *
   * @param {any} $event イベント情報
   * @memberof UseDirectiveComponent
   */
  public onClickOK($event) {
    alert('OK button had clicked.');
  }
}
```

ここではテンプレートの　```[(ngModel)]="inputAddress"``` と紐付けるためのプロパティ ```inputAddress``` を定義していることと、validation による検証エラーが無い場合に ```OKボタン``` をクリックした時の動作を実装しているだけで、ディレクティブに関することで特筆することはなし。


# 検証結果による描画( エラー表示 )

関連記事であげた [[Angular] 標準機能を利用して validation を実現する#エラー表示](https://qiita.com/ksh-fthr/items/ee9b026da40cae96ac38#%E3%82%A8%E3%83%A9%E3%83%BC%E8%A1%A8%E7%A4%BA) と同じ内容なので説明は割愛する｡


# 実行結果

## 起動直後
![directive_validation_boot.png](https://qiita-image-store.s3.amazonaws.com/0/193342/130d8b2c-27f4-1b37-b1a2-090186aabff3.png)


## 入力文字数が短い
![directive_validation_minlength.png](https://qiita-image-store.s3.amazonaws.com/0/193342/5d69544c-34f7-f46f-e3d5-2c3c169b642c.png)


## 入力文字数が長い
![directive_validation_maxlength.png](https://qiita-image-store.s3.amazonaws.com/0/193342/67a91d14-541b-22de-b1aa-f4f0c170c0e9.png)


## 入力内容がパターンに一致しない
![directive_validation_pattern.png](https://qiita-image-store.s3.amazonaws.com/0/193342/98c64ed3-5f15-98ff-84d5-dbf42cec4adc.png)


## 未入力
![directive_validation_nodata.png](https://qiita-image-store.s3.amazonaws.com/0/193342/b09ccf68-b191-daf5-a9df-8b60635260c0.png)

## 正常
![directive_validation_normal.png](https://qiita-image-store.s3.amazonaws.com/0/193342/ca11c278-9d7a-8122-a330-5f3cf3f0284e.png)


# ソースコード

今回の記事で作成したコードは [こちら](https://github.com/ksh-fthr/angular-work/tree/feat_directive_validation) にアップしてあるのでご参考まで。

# 参考

* [Adding to template-driven forms](https://angular.jp/guide/form-validation.en#adding-to-template-driven-forms)
* [AbstractControl](https://angular.io/api/forms/AbstractControl)
* [NG_VALIDATORS](https://angular.io/api/forms/NG_VALIDATORS)
