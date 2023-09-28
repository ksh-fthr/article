# はじめに

フロントエンドは Angular、バックエンドが Flask の環境でCSVファイルを出力するときにやったことのメモ。
とはいうものの、やっていることに Angular 要素はほとんどなくて、ほぼ JavaScript ( TypeScript ) と HTML5 で実現している。

# 更新情報

## 2021/01/11
- 記事内で扱ったコードを Angular `v11.0.5` で確認しました

# 作業環境
## フロントエンド

| 環境                                          | バージョン          | 備考               |
| --------------------------------------------- | ------------------- | ------------------ |
| [Angular CLI](https://cli.angular.io/)        | ~~v7.2.2~~ v11.0.5  | `$ ng --version`   |
| [Angular](https://angular.io/)                | ~~v7.2.2~~ v11.0.5  | 同上               |
| [TypeScript](https://www.typescriptlang.org/) | v4.0.2              | 同上               |
| [Node.js](https://nodejs.org/ja/)             | ~~v10.15.3~~ v12.18.3 | `$ node --version` |
| [npm](https://www.npmjs.com/)                 | ~~v6.4.1~~ v6.14.6  | `$ npm --version`  |


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


## バックエンド

|                      | バージョン | 備考           |
| -------------------- | ---------- | ------------------ |
| [Python](https://www.python.org/)            | 3.7.2      | $ python --version |
| [Flask](https://palletsprojects.com/p/flask/)| 1.0.2      | $ flask --version  |



# 前提: CSVファイルの出力にあたって

バックエンドは CSV データとファイル名を返却するに留め、CSVファイルの出力処理そのものは **フロントエンド** で行った。
このときの 流れは次のとおり。

1. フロントエンドからバックエンドへ
   1. CSV データを返却してもらう REST-API を実行する
2. バックエンドからフロントエンドへ
   1. CSV データとファイル名をプロパティとした JSON を返却する
3. フロントエンドで CSV ファイルを出力する
   1. バックエンドから返却された CSV データを `Blob` 型のオブジェクトにする
   2. CSVファイルのファイル名はバックエンドから返却されたファイル名とする
   3. CSVファイルの文字コードは `UTF-8 BOMあり` とする

すごく簡単な図だが、以下のイメージ
<img width="593" alt="スクリーンショット 2020-07-13 12.22.58.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/026cf87a-390e-f698-5dc7-9614d60b7813.png">



# 結論から

フロントエンド、バックエンドともに次の処理で実現できた。

## フロントエンド

### テンプレート

```html:CSV出力のUI
<div>
  <button type="button" class="btn btn1" (click)="outputCsv($event)">CSV出力</button>
  <a id="csv-donwload"></a>
</div>
```

### コンポーネント

```typescript:CSV出力の実行部分
import { Component, OnInit, ElementRef } from '@angular/core';
import { HttpClientService } from '../service/http-client.service';

@Component({
  selector: 'app-http-client',
  templateUrl: './http-client.component.html',
  styleUrls: ['./http-client.component.css']
})
export class HttpClientComponent implements OnInit {

  private element: HTMLElement;

  /**
   * コンストラクタ. HttpClientComponent のインスタンスを生成する
   * 自作した HttpClientService を DI する
   *
   * @param {HttpClientService} httpClientService HTTP通信を担当するサービス
   * @param {ElementRef} elementRef DOM参照のためのモジュール
   * @memberof HttpClientComponent
   */
  constructor(
    private httpClientService: HttpClientService,
    private elementRef: ElementRef
  ) {
    this.element = this.elementRef.nativeElement;
  }

  /**
   * ライフサイクルメソッド｡コンポーネントの初期化で使用する
   * 今回はなにもしない
   *
   * @memberof HttpClientComponent
   */
  ngOnInit() {}

  public async outputCsv(event: any): Promise<any> {
    //-------------------------------------------
    // 1. REST-API を実行して CSV データを取得する
    //-------------------------------------------
    this.httpClientService.getCsv()
    .then(
      (response: any) => {
        const csv = response.csv;
        const filename = response.fileName;

        //-------------------------------------------
        // 2. レスポンスを加工してCSVファイルとURLを作る
        //-------------------------------------------
        // CSV ファイルは `UTF-8 BOM有り` で出力する
        // そうすることで Excel で開いたときに文字化けせずに表示できる
        const bom = new Uint8Array([0xEF, 0xBB, 0xBF]);
        // CSVファイルを出力するために Blob 型のインスタンスを作る
        const blob = new Blob([bom, csv], { type: 'text/csv' });
        const url = window.URL.createObjectURL(blob);

        //-------------------------------------------
        // 3. 出力はリンクタグのDOMを取得してそこから行う
        //-------------------------------------------
        // this.element は `ElementRef.nativeElement` から取得した `HTMLElement`
        const link: HTMLAnchorElement = this.element.querySelector('#csv-donwload') as HTMLAnchorElement;
        link.href = url;
        link.download = filename;
        link.click();
      }
    )
    .catch(
      (error) => console.log(error)
    );
  }
}
```

### サービス

```typescript:バックエンドとの通信部分
import { Injectable } from '@angular/core';

// REST クライアント実装ののためのサービスを import
import { HttpClient, HttpHeaders } from '@angular/common/http';

@Injectable()
export class HttpClientService {

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
    // DELETE 実行時に `body` が必要になるケースがあるのでプロパティとして用意しておく
    // ( ここで用意しなくても追加できるけど... )
    body: null
  };

  /**
   * RST-API 実行時に指定する URL
   *
   * @private
   * @memberof HttpClientService
   * @description
   * バックエンドは Express で実装し、ポート番号「3000」で待ち受けているため、
   * そのまま指定すると CORS でエラーになる
   * それを回避するため、ここではフロントエンドのポート番号「4200」を指定し、
   * Angular CLI のリバースプロキシを利用してバックエンドとの通信を実現する
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
   * @returns {Promise<any>}
   * @memberof HttpClientService
   */
  public getCsv(): Promise<any> {
    return this.http.get(this.host + '/csv', this.httpOptions)
    .toPromise()
    .then((res) => {
      // response の型は any ではなく class で型を定義した方が良いが
      // ここでは簡便さから any としておく
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
  private errorHandler(err: any) {
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

## バックエンド

### CSV データを生成する API

```python:CSVデータをファイル名を返却するREST-API
from flask import Flask
from flask_restful import Resource
from datetime import datetime, timedelta
import csv
from io import StringIO


class Csv(Resource):
  def get(self) -> dict:
    """CSVデータを作って返却する

    Returns:
        Response -- レスポンスオブジェクト

    Description:
        CSVファイルの出力は行わず、作成したデータとファイル名を返却する
    """
    # CSVファイル名
    # ファイル名のフォーマットは ${STR}_${STR}_${YYYYMMDD}. とし、${STR} は任意の文字列が入る
    # ${YYYYMMDD} には西暦での年月日が入る
    date_time = datetime.now().strftime('%Y%m%d')
    output_path = '{}_{}_{}.csv'.format('好きな', '文字', date_time)

    # CSVデータを返却する
    f = StringIO()
    writer = csv.writer(f, quotechar='"', quoting=csv.QUOTE_ALL, lineterminator="\r\n")

    # ヘッダレコードとボディレコードを作る
    header_record = [
        '名前', '年齢', '住所', '電話番号', '備考'
    ]
    body_record = [
        'ほげ', '99歳', 'ほげ県ほげほげ市', '999-9999-9999', ''
    ]
    writer.writerow(header_record)
    writer.writerow(body_record)

    res: dict = {
        'fileName': output_path,
        'csv': f.getvalue()
    }

    return res
```

# 動作確認
## フロントエンドの起動
バックエンドとはオリジンが異なるため、プロキシ経由で起動させる。

```bash:フロントエンドの起動
$ npm start
```

プロキシの設定は以下のとおり。

```json:プロキシの設定
{
  "/app": {
    "target": "http://localhost:5000", # バックエンドはポート: 5000 で待受け
    "pathRewrite": {"^/app": ""}
  }
}
```

## バックエンドの起動

```bash:バックエンドの起動
$ python3 app/run.py
```

## 出力される CSV ファイル
- デフォルトのファイル名: `好きな_文字_20210111.csv` (`20210111` の部分は出力日付)
- ファイルの内容

```csv
"名前","年齢","住所","電話番号","備考"
"ほげ","99歳","ほげ県ほげほげ市","999-9999-9999",""
```

# 具体的な説明

## フロントエンド

フロントエンドは Angular アプリとして実装しているので、html を表現しているテンプレートと処理を実装してるコンポーネントに分けて説明する。

### テンプレートについて

ボタン要素に加えて `  <a id="csv-donwload"></a>` でリンク要素を配置している点がポイント。
ボタン要素の `(click)="outputCsv($event)"` で CSV出力処理が実行された際にこのリンクタグが活きてくる。
詳細は後述の [3. CSVファイルの出力](#3-csv-ファイルの出力) を参照。

### コンポーネント について

コンポーネントにはテンプレートで「CSV出力」ボタンがクリックされたときの処理である `outputCsv` を実装する。
ここでのポイントは大きくわけて 3 つ。

1. ひとつめは REST-API によるCSVファイル出力ための情報の取得
2. ふたつめは CSV ファイル出力のための準備
3. みっつめは CSV ファイルの出力

以下、これらのポイントを細かくみていく。



#### 1. REST-API によるCSVファイル出力ための情報の取得

```typescript:APIの実行部分
  //-------------------------------------------
  // 1. REST-API を実行して CSV データを取得する
  //-------------------------------------------
  const res: any = await this.httpClientService.postCsv();
  const csv = res.csv;
  const filename = res.fileName;
```

細かくみていく、といったがここで語る詳細はあまりない。単純に Angular で Http クライアントを実装して、後述のバックエンドで提供している CSV データ取得のための API を実行するだけ。

Angualr における Http クライアントの実装については、僭越ながら [こちらの記事](https://qiita.com/ksh-fthr/items/840ae54472892a87f48d) で触れているのでご興味あれば参照されたい。



#### 2. CSV ファイル出力のための準備

```typescript:CSVファイルとURLを作る
  //-------------------------------------------
  // 2. レスポンスを加工してCSVファイルとURLを作る
  //-------------------------------------------
  // CSV ファイルは `UTF-8 BOM有り` で出力する
  // そうすることで Excel で開いたときに文字化けせずに表示できる
  const bom = new Uint8Array([0xEF, 0xBB, 0xBF]);
  // CSVファイルを出力するために Blob 型のインスタンスを作る
  const blob = new Blob([bom, csv], { type: 'text/csv' });
  const url = window.URL.createObjectURL(blob);
```

こちらは見るべき箇所が何点かあるので、ポイントごとに説明を。

1. `Blob()` でインスタンスの生成
   1. これは CSV ファイルを生成するためのもので、ここではコンストラクタの引数に注目
   2. コンストラクタ引数にはまず `[bom, csv]` として CSV データと共に **UTF-8** の **B**yte **O**rder **M**ark( BOM )として `[0xEF, 0xBB, 0xBF]` を指定している
   3. こうすることで **BOM付きの UTF-8** で出力できる
   4. BOM 付きの UTF-8 で出力したい理由はコード中のコメントのとおりで、 Excel で開いたときの文字化けを防ぐため
   5. そして `Option` として MIME タイプに `{ type: 'text/csv' }` を指定している
2. `window.URL.createObjectURL()` で CSV ファイルの URL を生成
   1. 生成した Blob インスタンスを引数に設定することで生成した CSV ファイルを指す URL が作られる
   2. これによって `URL == CSVファイル` ということになる
   3. で、ここで生成した URL が後述の [CSV ファイル出力](#3-csv-ファイルの出力) で利用される

#### 3. CSV ファイルの出力

```typescript:csvファイルの出力
  //-------------------------------------------
  // 3. 出力はリンクタグのDOMを取得してそこから行う
  //-------------------------------------------
  // this.element は `ElementRef.nativeElement` から取得した `HTMLElement`
  const link: HTMLAnchorElement = this.element.querySelector('#csv-donwload') as HTMLAnchorElement;
  link.href = url;
  link.download = filename;
  link.click();
```

最後は CSV ファイルの出力処理。ここでは CSV ファイル出力のためにリンク要素である `<a></a>` タグを取得している。

前項までの処理は UI からの「CSV出力」ボタンのクリックイベントを受けて「データを取得して」「CSVファイルの生成とダウンロードURLを生成した」だけで、これだけでは CSVファイルをダウンロードすることは出来ない。

`<a></a>` タグ の `href` 属性に URL を設定し、`download` 属性にファイル名を設定し、`click()`  メソッドを実行させてようやく「**バックエンドから取得したCSVデータを、同じくバックエンドから取得したファイル名で UTF-8 BOM付きでダウンロード(出力)する**」という要件が完了する。

ポイントは次のとおり。

1. `href` 属性に Blob インスタンスから生成した URL をセットする。ただしこれだけでは ダウンロードは始まらない
2. `download` 属性にファイル名をセットする。これを行わないと CSVファイル名に目的のファイル名がセットされない
3. `click()` を実行することでリンククリックアクションの実行を実現する

あとはおまけ程度であるが、DOM ( `<a></a>`  タグ ) の取得を

```html:DOMの取得処理
const link: HTMLAnchorElement = this.element.querySelector('#csv-donwload') as HTMLAnchorElement;
```

で行っている点も挙げておく。
これは一応 Angular アプリ上での実装なので、DOM の取得にあたり `document.getElementById()` ではなく`ElementRef.nativeElement` から行なった。

それからもう一点。`HTMLAnchorElement` でキャストしているのは `TS2339` 対策。

```html:リンク要素の属性を設定
link.href = url;
link.download = filename;
link.click();
```

取得したDOMに対して `href`, `download`, `click()` を行った際に

```bash:typescriptのエラー
error TS2339: Property 'href' does not exist on type 'Element'.
error TS2339: Property 'download' does not exist on type 'Element'.
error TS2339: Property 'click' does not exist on type 'Element'.
```

が出るのを防ぐのが目的である。

#### 4. フロントエンドのまとめ

大きく3つのポイントをあげたが、その中でも [3. CSV ファイルの出力](#3-csv-ファイルの出力) で行っているリンク要素の取得から始まる CSV ファイルのダウンロード処理が重要な点だ。

ボタン要素ではなくリンク要素のイベントとして `(click)="outputCsv($event)"` を配置しても

1. REST-API の実行
2. Blob インスタンスの生成〜URL
3. `download` 属性にファイル名を設定

することまでは行える。
が、前述のとおりこれだけではCSVファイルのダウンロードは実行されない。`click()` して初めて `href` に設定した URL からダウンロードが実行される。

ではここで `click()` を実行するのはどれかというと、すでにクリックされたリンク要素である `<a></a>` タグ自身である。
で、それをするとどうなるかというと、ダウンロードのためのダイアログが表示されたあとに、また REST-API の実行から始まる一連の処理が実行される。

このループを回避する意味もあり、

1. UI にボタン配置
2. ボタンクリックによる CSV データの取得から `<a></a>` タグの生成とダウンロード

で要件を実現させている。

## バックエンド

バックエンドは Python のマイクロフレームワークである Flask を採用していて、REST-API の実装にあたり`flask_restful` を使用している。エンドポイントは `GET` で待ち受け。

※ Flask や flask_restful についての説明はここではしない。

### CSV データの生成

```python:CSVデータをファイル名を返却するREST-API(抜粋)
    # CSVデータを返却する
    f = StringIO()
    writer = csv.writer(f, quotechar='"', quoting=csv.QUOTE_ALL, lineterminator="\r\n")

    # ヘッダレコードとボディレコードを作る
    header_record = [
        '名前', '年齢', '住所', '電話番号', '備考'
    ]
    body_record = [
        'ほげ', '99歳', 'ほげ県ほげほげ市', '999-9999-9999', ''
    ]
    writer.writerow(header_record)
    writer.writerow(body_record)
```

ポイントは上記の CSV データの生成部分で、注目するのは 2 点。

まずひとつめ。
この API では **CSV のフォーマットに整形したデータを返却する** ので、`StringIO` のインスタンスをファイルとして扱い、そこに書き込まれた( 設定された )文字列を CSV データとしてレスポンスに設定している。

次いでふたつめ。
CSVデータの出力は `writerow` メソッドで行うのだが、ここで当該メソッドの引数に設定しているヘッダやボディのレコードを `List` で設定していること。出力するデータを  `List` で渡すことで各項目がカンマ区切りで1レコードに出力される。
ここで `writerow` に渡すデータを`List` に設定せずに **文字列として設定した場合、意図せずに余計なカンマ が付与されてしまう**ので注意。

# CSV ファイルの出力で出来なかったこと

Flask で CSV ファイルを出力するAPIを実装する、という要件について調べていたとき、次のようなサンプルコードをよく目にした。

```python:よくみるCSVファイル出力のAPIサンプル
    #
    # レスポンスデータとして CSVファイルを作成
    #
    f = StringIO()
    writer = csv.writer(f, quotechar='"', quoting=csv.QUOTE_ALL, lineterminator="\r\n")

    # ヘッダレコードとボディレコードを作る
    header_record = [
        '名前', '年齢', '住所', '電話番号', '備考'
    ]
    body_record = [
        'ほげ', '99歳', 'ほげ県ほげほげ市', '999-9999-9999', ''
    ]
    writer.writerow(header_record)
    writer.writerow(body_record)

    # レスポンスのインスタンスを生成
    res = make_response()

    # ファイルインスタンスからデータを取得して UTF-8 BOMあり でレスポンスにセット
    res.data = f.getvalue().encode('utf_8_sig')

    # レスポンスヘッダに CSV ファイルであることを設定する
    # 非ASCIIのファイル名だとエラーになるので `unicode-escape` を指定してエンコードする
    res.headers['Content-Type'] = 'text/csv'
    res.headers['Content-Disposition'] = 'attachment; filename={}'.format('非ASCIIのファイル名'.encode('unicode-escape'))
```

実際のところ、Postman 上で確認した限りでは上記コードでCSVデータは作成され、ヘッダ情報も設定した内容が含まれていた。

だがこれを Angular アプリから実行したときに

- API の実行結果を受けても、そのままではCSVファイルのダウンロードが行われなかった
- そのため、結局は [2. CSV ファイル出力のための準備](#2-csv-ファイル出力のための準備) でやったように BOMありを指定したうえで`Blob`  のインスタンスを生成して URL を作らなければならなかった
- しかもこのケースだと CSV ファイル名をヘッダから取得しなければならず、しかもエンコードされたファイル名をデコードする必要があった ( が、これは面倒だったので試行していない )

 という結果になった。

で、思ったことが、それならば最初から「ファイル名」と「CSVデータ」を JSON で貰って、そのデータをもとにCSVファイル出力をフロントエンドで実装した方が面倒が少ない、ということで本記事の内容となった。


# ソースコード

今回の記事で作成したコードは 以下にアップしてあるのでご参考まで。

* [フロントエンド](https://github.com/ksh-fthr/angular-work/tree/feat_csv)
* [バックエンド](https://github.com/ksh-fthr/flask-work/tree/feat/csv)



# 参考

- [MDN-Blob](https://developer.mozilla.org/ja/docs/Web/API/Blob)
- [MDN-Blob-コンストラクタ](https://developer.mozilla.org/ja/docs/Web/API/Blob/Blob)
- [MDN-Uint8Array](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array)
- [JavaScriptでファイルダウンロード機能を実装する](https://helloworld-blog.tech/javascript/javascriptでファイルダウンロード機能を実装する)

