# はじめに
「 [[Angular] ngComponentOutlet を使ってタブを実装する](https://qiita.com/ksh-fthr/items/212fe3a1c0308b1fd782) 」 で ```ngComponentOutlet``` を使用したタブの実装例について記事を書いた。本記事では同じく ```ngComponentOutlet``` を使った実装例として、モーダルダイアログの実装を試してみたい｡

モーダルダイアログについては下記を参照｡

* [モーダルダイアログ 【 modal dialog 】 モーダルウィンドウ / modal window](http://e-words.jp/w/%E3%83%A2%E3%83%BC%E3%83%80%E3%83%AB%E3%83%80%E3%82%A4%E3%82%A2%E3%83%AD%E3%82%B0.html)

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
    │    └ modal/                           # モーダルダイアログ
    │         ├ modal.component.css
    │         ├ modal.component.html
    │         └ modal.component.ts
    ├ service/                               # モーダルを管理するサービス
    │    └ modal.service.ts
    ├ app.component.css
    ├ app.component.html
    ├ app.component.ts
    └ app.module.ts
```


# 方針

* モーダルダイアログはヘッダ/ボディ/フッタを持ち､フッタに close ボタンを配置する
  * [modal.component.html](#modalcomponenthtml), [modal.component.ts](#modalcomponentts) で実装
* モーダルダイアログは呼び出し元の画面からボタンクリックで表示する
  * [app.component.html](#appcomponenthtml), [app.component.ts](#appcomponentts) で実装
* モーダルダイアログを表示したらオーバーレイで呼び出し元の画面への操作を抑止する
  * [modal.component.html](#modalcomponenthtml), [modal.component.css](#modalcomponentcss) で実装
* モーダルダイアログの close ボタンクリックで閉じる
  * [app.component.ts](#appcomponentts), [modal.component.ts](#modalcomponentts), [modal.service.ts](#modalservicets) で実装


# 実装

## app.component

app.component ではモーダルダイアログを表示するためのベースとなる画面を作成し、```ngComponentOutlet``` を利用したモーダルダイアログの呼び出しを行う。

以下、[app.component.html](#appcomponenthtml) と [app.component.ts](#appcomponentts) の実装について触れる。
app.component.css は単純なスタイル定義を担当しているだけなので説明を省略する。

### app.component.html

```html:app.component.html
<div class="footer">
  <input id="footer_button" type="button" value="open modal" (click)="onClick($event);">
</div>
<ng-container *ngComponentOutlet="modal"></ng-container>
```

ここでの実装の流れは下記のとおり。

* フッターに配置したボタンクリックでイベントをコンポーネントに送り
* コンポーネントは ```modal``` に ```ModalComponent``` をセット
* ```modal``` に実体( ```ModalComponent``` )がセットされたので ```ngComponentOutlet``` によってモーダルを表示する

つまりテンプレートはイベントトリガーとしての役割を持ち、次の [app.component.ts](#appcomponentts) との連携がポイントとなる。


### app.component.ts

```typescript:app.component.ts
import { Component, OnInit, OnDestroy  } from '@angular/core';
import { Subscription } from 'rxjs';

// モーダルダイアログとして表示するコンポーネント
import { ModalComponent } from './component/modal/modal.component';

// モーダルダイアログを閉じるためのイベントを管理するサービス
import { ModalService } from './service/modal.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit, OnDestroy  {

  // モーダルダイアログが閉じた際のイベントをキャッチするための subscription
  private subscription!: Subscription;

  // ngComponentOutlet にセットするためのプロパティ
  public modal: any = null;

  /**
   * コンストラクタ
   *
   * @memberof AppComponent
   */
  constructor(
    private modalService: ModalService
  ) {}

  /**
   * 初期処理
   *
   * @memberof AppComponent
   */
  ngOnInit() {
    // モーダルダイアログを閉じた際のイベントを処理する
    this.subscription = this.modalService.closeEventObservable$.subscribe(
      () => {
        // プロパティ modal に null をセットすることでコンポーネントを破棄する
        // このタイミングで ModalComponent では ngOnDestroy が走る
        this.modal = null;
      }
    );
  }

  /**
   * 終了処理
   *
   * @memberof AppComponent
   */
  ngOnDestroy() {
    this.subscription.unsubscribe();
  }

  /**
   * クリックイベント
   *
   * @param {*} $event イベント情報
   * @memberof AppComponent
   */
  public onClick($event: any) {
    this.setModal();
  }

  /**
   * モーダルダイアログを表示する
   *
   * @private
   * @memberof AppComponent
   */
  private setModal() {
    this.modal = ModalComponent;
  }
}
```

ポイントは以下の 2 点｡

* クリックイベント ```onClick``` で ```ngComponentOutlet``` にセットするコンポーネント: ```ModalComponent``` をセット
* ```Subscription``` によってモーダルダイアログが閉じた際のイベントをキャッチ

上記 2 点によって

* テンプレートでボタンクリック -> モーダルダイアログを表示する
* モーダルダイアログで close ボタンクリック -> モーダルダイアログを閉じる

という動きを実現している｡


## modal.service

app.modal.service はモーダルダイアログの管理を担当する。本記事の例では close イベントを管理するだけだが、その内容について [modal.service.ts](#modalservicets) で説明する。

### modal.service.ts

```typescript:modal.service.ts
import { Injectable } from '@angular/core';
import { Subject } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class ModalService {

  // データの変更を通知するためのオブジェクト
  private closeEventSubject = new Subject<string>();

  // Subscribe するためのプロパティ( これでイベント通知をキャッチする )
  public closeEventObservable$ = this.closeEventSubject.asObservable();

  /**
   * コンストラクタ
   *
   * @memberof ModalService
   */
  constructor() { }

  /**
   * イベント通知のリクエストを処理する( モーダルダイアログを閉じる )
   *
   * @memberof ModalService
   */
  public requestCloseModal() {
    this.closeEventSubject.next();
  }
}
```

前述のとおり、このサービスの役割はひとつだけ｡

* ```requestCloseModal()``` でモーダルダイアログを閉じるためのイベント処理を受け付け､イベントを発火する

これのみとなる｡

なお､ここで行っていることの詳細( ```Subject``` を使用したイベントのやり取り )については､手前味噌ながら「 [[Angular] サービスを使用してデータをコンポーネント間で共有する](https://qiita.com/ksh-fthr/items/e43dd37bff2e51e95a59) 」で説明しているので､ご興味あればそちらを参照いただきたい｡


## modal.component

modal.component はモーダルダイアログの本体であるコンポーネント。役割はモーダルダイアログを表示して閉じるだけのものとなる。
ただその中でもオーバーレイやダイアログを閉じる際の処理について触れる部分があるので、その説明を [modal.component.html](#modalcomponenthtml), [modal.component.css](#modalcomponentcss), [modal.component.ts](#modalcomponentts) で行う。

### modal.component.html

```html:modal.component.html
<div class="overlay">
  <div class="modal">
    <div class="header">
      <label id="header_label">header title</label>
    </div>
    <div class="contents">
      contents body
    </div>
    <div class="footer">
      <input id="close_button" type="button" value="close" (click)="onClick($event)">
    </div>
  </div>
</div>
```

ここでのポイントは

* ```<div class="overlay">``` でオーバーレイを実装している

こと｡
これと後述の [スタイルシート](#modalcomponentcss) によって呼び出し元の [app.component.html](#appcomponenthtml) とモーダルダイアログとの間にオーバーレイが挟まるので､モーダルダイアログ表示中は呼び出し元の画面に対して操作ができなくなる｡


### modal.component.css

```css:modal.component.css
.overlay {
  margin: 0;
  padding: 0;
  position: absolute;
  width: 100%;
  height: 100%;
  overflow: hidden;
  background: rgba(0, 0, 0, 0.4);
  z-index: 1000;
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal {
  overflow: hidden;
  height: 200px;
  width: 400px;
  background-color: rgb(229, 241, 247);
  border-radius: 15px;
  box-shadow: 10px 10px 10px rgba(0,0,0,0.4);
  border: solid 1px rgb(0, 0, 0);
  z-index: 1001;
}

.header {
  height: 20%;
  width: 100%;
  font-size: 1.2em;
  border-bottom: solid 1px rgb(0, 0, 0);
  display: flex;
  justify-content: center;
  align-items: center;
}

.header > #header_label {
  display: block;
}

.contents {
  height: 60%;
  width: 100%;
  text-align: center;
  font-size: 1.8em;
}

.footer {
  height: 20%;
  width: 100%;
  border-top: solid 1px rgb(0, 0, 0);
  display: flex;
  justify-content: center;
  align-items: center;
}

.footer > #close_button {
  display: block;
  width: 30%;
  height: 70%;
  font-size: 1.2em;
}
```

```.overlay``` は [modal.component.html](#modalcomponenthtml) でポイントとして取り上げた ```<div class="overlay">``` のスタイル定義｡
この要素で ```position: absolute```, ```width:100%```, ```height:100%``` を指定していることで､親要素( ここでは呼び出し元画面 )に対して全体を覆っている｡


### modal.component.ts

```typescript:modal.component.ts
import { Component, OnInit, OnDestroy } from '@angular/core';

// モーダルダイアログを閉じるためのイベントを管理するサービス
import { ModalService } from '../../service/modal.service';

@Component({
  selector: 'app-modal',
  templateUrl: './modal.component.html',
  styleUrls: ['./modal.component.css']
})
export class ModalComponent implements OnInit, OnDestroy {

  /**
   * コンストラクタ
   *
   * @param {ModalService} modalService
   * @memberof ModalComponent
   */
  constructor(
    private modalService: ModalService
  ) {}

  /**
   * 初期処理
   *
   * @memberof ModalComponent
   */
  ngOnInit() {}

  /**
   * 終了処理
   *
   * @memberof ModalComponent
   */
  ngOnDestroy() {
    // モーダルダイアログが閉じたタイミングで出力される
    console.log('destroyed');
  }

  /**
   * クリックイベント
   *
   * @param {*} $event イベント情報
   * @memberof ModalComponent
   */
  public onClick($event: any) {
    this.notifyCloseModal();
  }

  /**
   * モーダルダイアログを閉じる
   *
   * @private
   * @memberof ModalComponent
   */
  private notifyCloseModal() {
    this.modalService.requestCloseModal();
  }
}
```

こちらはモーダルダイアログで描画している ```close``` ボタンの制御に関する内容となる｡
ポイントは

```typescript:closeボタンクリック時の処理(コメントは除去済み)
  public onClick($event) {
    this.notifyCloseModal();
  }

  private notifyCloseModal() {
    this.modalService.requestCloseModal();
  }
```

の部分で､ ```close``` ボタンがクリックされたら [modal.service.ts](#modalservicets) の ```requestCloseModal()``` メソッドを実行している｡その後は [app.component.ts](#appcomponentts) に処理が流れてモーダルダイアログが閉じる動きとなる｡

で､このときの処理の流れを図にしたものが以下のシーケンス｡

![modal04.png](https://qiita-image-store.s3.amazonaws.com/0/193342/50573f37-004f-9f3d-3dc7-96dac06f30d6.png)

ちなみにシーケンス上で Service から App に対して発火されている｢ イベント通知 ｣は､厳密には App に対してのみ発火されているのではない｡
｢ **Service が対象を指定せずに発火したイベントを  App がキャッチしている** ｣ というのが正しい解釈になるのだが､シーケンス図として表現するには図のようにしたほうがイメージしやすいためそうしている｡

またこのシーケンスの登場人物と各ファイルの対応は以下の表を参照｡

* 登場人物と各ファイルの対応表

    | 登場人物  | ファイルとの対応     |
    |-----------|----------------------|
    | View      | modal.component.html |
    | Component | modal.component.ts   |
    | Service   | modal.service.ts     |
    | App       | app.component.ts     |



# 実行結果

1. 初期表示

    ![modal01.png](https://qiita-image-store.s3.amazonaws.com/0/193342/3819e1fd-2c32-ff86-1fd1-e65c8197935c.png)

    ｢open modal｣ ボタンをクリックすることでモーダルダイアログが表示される

1. モーダルダイアログを表示

    ![modal02.png](https://qiita-image-store.s3.amazonaws.com/0/193342/22fa8947-9796-88ae-de41-7ca7dae1d1fc.png)

    ｢close｣ボタンをクリックすることでモーダルダイアログが閉じる


1. モーダルダイアログを閉じたときのログ

    ![modal03.png](https://qiita-image-store.s3.amazonaws.com/0/193342/311e726c-d2c0-fa78-ecad-e80b0a94cd64.png)

    モーダルダイアログが閉じたときのログ｡｢ destroyed ｣が出力されていることから､ [modal.component.ts](#modalcomponentts) の ```ngOnDestroy()``` が実行されていることがわかる｡


# まとめ

ひとつのボタンにひとつのモーダルダイアログを紐付けただけのなんのひねりもない例だが､ ```ngComponentOutlet``` を使用してモーダルダイアログが簡単に実装できた｡
やっていることは ```ngIf``` や ```ngSwitch``` でも実現できるが､ ```ngComponentOutlet``` を使うことの強みはやはり **抽象化** だと思う｡

「 [[Angular] ngComponentOutlet を使ってタブを実装する](https://qiita.com/ksh-fthr/items/212fe3a1c0308b1fd782) 」でもそうだったが､ ```ngComponentOutlet``` セットされたコンポーネントによって表示する内容を動的に切り替えることができるのはコードがスッキリして気持ちが良い｡


# ソースコード

今回の記事で作成したコードは [こちら](https://github.com/ksh-fthr/angular-work/tree/feat_modal) にアップしてあるのでご参考まで。
