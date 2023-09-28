# はじめに

[こちら](https://qiita.com/ksh-fthr/items/29db7c5c7268ee1802c5) の記事で 「REST-API で CSV データを受け取り、それを CSV ファイルとしてダウンロードする」やり方について触れた。
今回は 「CSV データではなく ZIP データを受け取り、それを ZIP ファイルとしてダウンロードする」方法について触れる。

本記事を書くにあたりバックエンドの実装を一つの記事にまとめるのは流石に長いと思ったので、バックエンドについては別途記事を作成した。ご興味あれば [こちらの記事](https://qiita.com/ksh-fthr/items/df875613d7e36f94a679) を参照されたい。



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



# やりたいこと

冒頭で触れたとおり、今回は以下を目的とする。

- ZIP データを REST-API から受け取り、ZIP ファイルとしてダウンロードする



で、その際の前提として以下を挙げる。



# 前提

- RETS-API からは以下のデータが返却される
  - ファイル名
  - base64 文字列化した ZIP データ
- つまり次のフォーマットであることを期待する

```javascript

{
  'fileName': string, // ファイル名
  'zip': string       // ZIP ファイルのデータを base64文字列 にコンバートしたもの
}
```



# 実装
## フロントエンド

Angular での実装となるので REST-API との通信部分はサービス化、受け取ったデータの ZIP ファイル化 & ダウンロードはコンポーネントで対応する方針とする。

で、サービスの部分については [前回の記事](https://qiita.com/ksh-fthr/items/29db7c5c7268ee1802c5) の 「[サービス](https://qiita.com/ksh-fthr/items/29db7c5c7268ee1802c5#%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9)」で扱っているので今回は触れず、テンプレートとコンポーネントのみ取り上げる。



### テンプレート

```html:テンプレート全体
<div class="button-block">
  <div class="buttons">
    <button type="button" class="btn btn1" (click)="outputZip($event)">ZIPファイル出力</button>
    <a id="zip-donwload"></a>
  </div>
</div>
```



### コンポーネント

```typescript:コンポーネント全体
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

  /**
   * ZIP ファイル出力を行う
   *
   * @param event イベント情報
   * @description
   *  REST-API を実行して取得した ZIP データを元に ZIP ファイル出力を行う
   */
  public async outputZip(event: any): Promise<any> {
    //-------------------------------------------
    // 1. REST-API を実行して ZIP データを取得する
    //-------------------------------------------
    this.httpClientService.getZip()
    .then(
      (response: any) => {
        const zip = response.zip;
        const filename = response.fileName;

        // -------------------------------------------
        // 2. レスポンスを加工して ZIP ファイルと URL を作る
        // -------------------------------------------
        // data はバイナリを文字列化したものなので、これをバイナリに戻してやる必要がある
        // ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        // └─> ZIP を base64エンコード + utf-8 でデコードして文字列化している
        const blob = this.toBlobZip(zip);
        const url = window.URL.createObjectURL(blob);

        //-------------------------------------------
        // 3. 出力はリンクタグの DOM を取得してそこから行う
        //-------------------------------------------
        // this.element は `ElementRef.nativeElement` から取得した `HTMLElement`
        const link: HTMLAnchorElement = this.element.querySelector('#zip-donwload') as HTMLAnchorElement;
        link.href = url;
        link.download = filename;
        link.click();
      }
    )
    .catch(
      (error) => console.log(error)
    );
  }

  /**
   * bas64 文字列になっている ZIP ファイル(バイナリデータ) をバイナリデータに変換する
   *
   * @private
   * @param {string} base64 バイナリデータを base64 エンコードして更に文字列化した文字列
   * @returns {Blob} 引数の文字列をバイナリに戻したバイナリデータ
   * @memberof AggregateMonthlyComponent
   * @description
   *  ZIP ファイルへの変換のみ対応している
   * @see
   *  https://developer.mozilla.org/ja/docs/Web/API/WindowBase64/atob
   *  https://developer.mozilla.org/ja/docs/Web/API/Blob
   *  https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_objects/Uint8Array
   */
  private toBlobZip(base64: string): Blob {
    const bin = atob(base64.replace(/^.*,/, ''));
    const buffer = new Uint8Array(bin.length);
    for (let i = 0; i < bin.length; i++) {
      buffer[i] = bin.charCodeAt(i);
    }
    const blob = new Blob([buffer.buffer], {
      type: 'application/zip'
    });
    return blob;
  }
}
```

では以下、ポイントとなる次の 2 メソッドについて触れていく。

- `outputZip(event: any): Promise<any>`
- `toBlobZip(base64: string): Blob`



### ZIP データの取得とダウンロード

まずは `outputZip(event: any): Promise<any>` から。

```typescript:outputZip
  /**
   * ZIP ファイル出力を行う
   *
   * @param event イベント情報
   * @description
   *  REST-API を実行して取得した ZIP データを元に ZIP ファイル出力を行う
   */
  public async outputZip(event: any): Promise<any> {
    //-------------------------------------------
    // 1. REST-API を実行して ZIP データを取得する
    //-------------------------------------------
    this.httpClientService.getZip()
    .then(
      (response: any) => {
        const zip = response.zip;
        const filename = response.fileName;

        // -------------------------------------------
        // 2. レスポンスを加工して ZIP ファイルと URL を作る
        // -------------------------------------------
        // data はバイナリを文字列化したものなので、これをバイナリに戻してやる必要がある
        // ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        // └─> ZIP を base64エンコード + utf-8 でデコードして文字列化している
        const blob = this.toBlobZip(zip);
        const url = window.URL.createObjectURL(blob);

        //-------------------------------------------
        // 3. 出力はリンクタグの DOM を取得してそこから行う
        //-------------------------------------------
        // this.element は `ElementRef.nativeElement` から取得した `HTMLElement`
        const link: HTMLAnchorElement = this.element.querySelector('#zip-donwload') as HTMLAnchorElement;
        link.href = url;
        link.download = filename;
        link.click();
      }
    )
    .catch(
      (error) => console.log(error)
    );
  }
```

といっても、こちらも [前回の記事](https://qiita.com/ksh-fthr/items/29db7c5c7268ee1802c5) の 「[コンポーネントについて](https://qiita.com/ksh-fthr/items/29db7c5c7268ee1802c5#%E3%82%B3%E3%83%B3%E3%83%9D%E3%83%BC%E3%83%8D%E3%83%B3%E3%83%88-%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6)」で扱っているので特に取り立てる箇所はない。
以下の流れで ZIP データを取得し、ZIP ファイルをダウンロードしている。( カッコ内はコード中のコメント )

1. API を実行して ZIP データを受け取り( `1. REST-API を実行して ZIP データを取得する` )
2. ZIP ファイルの作成とダウンロードURLを作成(`2. レスポンスを加工して ZIP ファイルと URL を作る` )
3. テンプレートにある `<a></a>` タグの DOM を取得してクリックイベントを実行( `3. 出力はリンクタグの DOM を取得してそこから行う` )



### バイナリデータに戻す

次に `toBlobZip(base64: string): Blob` について。

```typescript:toBlobZip
  /**
   * bas64 文字列になっている ZIP ファイル(バイナリデータ) をバイナリデータに変換する
   *
   * @private
   * @param {string} base64 バイナリデータを base64 エンコードして更に文字列化した文字列
   * @returns {Blob} 引数の文字列をバイナリに戻したバイナリデータ
   * @memberof AggregateMonthlyComponent
   * @description
   *  ZIP ファイルへの変換のみ対応している
   * @see
   *  https://developer.mozilla.org/ja/docs/Web/API/WindowBase64/atob
   *  https://developer.mozilla.org/ja/docs/Web/API/Blob
   *  https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_objects/Uint8Array
   */
  private toBlobZip(base64: string): Blob {
    const bin = atob(base64.replace(/^.*,/, ''));
    const buffer = new Uint8Array(bin.length);
    for (let i = 0; i < bin.length; i++) {
      buffer[i] = bin.charCodeAt(i);
    }
    const blob = new Blob([buffer.buffer], {
      type: 'application/zip'
    });
    return blob;
  }
```

今回の記事の最重要な点はこちら。
といってもやっていることは単純で、**base64 文字列をバイナリに変換している** だけである。

ただそのための手順が重要なので、それを追ってみていく。



#### 1. atob によるデコード

ここは [MDN の atob の説明](https://developer.mozilla.org/ja/docs/Web/API/WindowBase64/atob) をそのまま引用する。

> base64 形式でエンコードされたデータの文字列をデコードします。

とあるとおり、base64 文字列化されたデータをデコードしている。
( なおここで返却されたデータである bin は `typeof bin` で型を確認すると `string` である )

またここで `replace(/^.*,/, '')` とあるのは、ファイル情報として必要な情報が始まる `,` までの情報をトリムするため。
たとえば `data:application/zip;base64,iVBORw0KGgoAAAANSUhEUgAA` というデータだったら、`data:application/zip;base64,` をトリムする。



#### 2. Uint8Array による配列の生成

ここでは後述の Blob オブジェクトを生成するための **元データとなる配列** を生成する。
そのために以下を行っている。

1. Unit8Array のコンストラクタ引数に `atob` でデコードしたバイナリのレングスを指定してオブジェクトを生成
2. バイナリを一バイトずつループして `charCodeAt()` で **整数値** に変換した値をセット

特に2つ目の `charCodeAt()` による整数値への変換がキモで、これを行うことで初めて `atob` で変換したバイナリデータが Blob オブジェクトの生成元データとなる。



#### 3. Blob オブジェクトの生成

ここで生成したオブジェクトがメソッドの返り値になり、更にいうと戻った先で `URL.createObjectURL` の引数として扱われる。つまりこれが ZIP ファイルの本体となる。

さて、ここの処理ではコンストラクタとその引数に注目。

- [コンストラクタ](https://developer.mozilla.org/ja/docs/Web/API/Blob/Blob)
  - `var aBlob = new Blob( array[, options]);`

- 第一引数
  - 新たに Blob オブジェクトを生成するための元データとして「[2.Uint8Array による配列の生成](#2.-Uint8Array-による配列の生成)」 で生成したオブジェクトから `[buffer.buffer]` で `ArrayBuffer` オブジェクトを指定
- 第二引数
  - MIME タイプとして ZIP ファイルを扱うために `type: 'application/zip'` を指定

## バックエンド
### ZIP ファイルを生成して返却する API

```python:ZIPファイル生成して返却するREST-API
from flask import Flask
from flask_restful import Resource
from datetime import datetime, timedelta
import csv
from io import StringIO
import zipfile
import base64
import os

TMP_PATH = './tmp'


class Zip(Resource):
  def get(self) -> dict:
    """CSVファイルを複数作ってZIPに固めて返却する

    Returns:
      Response -- レスポンスオブジェクト

    Description:
      CSVファイルの出力を行った上でZIPに固めて返却する
      ZIPは base64エンコードした上で文字列化する
    """
    # CSVファイル出力のための準備
    os.makedirs(TMP_PATH, exist_ok=True)

    res: dict = self.__create_zip_file()

    # 後始末. 作成した CSV ファイルや ZIP ファイルを削除する
    # CSV生成処理である `create_csv_monthly` の中でやると zip ファイルが掴まれたままで
    # `PermissionError` が発生するので、仕方なくメソッドを抜けたあとに後始末を行う
    self.__delete_files(TMP_PATH)

    return res

  def __create_zip_file(self) -> dict:
    """[summary]

    Returns:
      dict -- ファイル名と base64文字列化したZIPファイルのデータをセットした dict
    """
    #
    #  サンプルコードなのでヘッダもデータも各ファイルで使いまわす
    #

    # ヘッダレコードとボディレコードを作る
    header_record = [
        '名前', '年齢', '住所', '電話番号', '備考'
    ]
    body_record = [
        'ほげ', '99歳', 'ほげ県ほげほげ市', '999-9999-9999', ''
    ]

    # CSVファイル名
    # ファイル名のフォーマットは ${STR}_${STR}_${YYYYMMDD}. とし、${STR} は任意の文字列が入る
    # ${YYYYMMDD} には西暦での年月日が入る
    date_time = datetime.now().strftime('%Y%m%d')
    output_path1 = '{}_{}_{}.csv'.format('好きな', '文字1', date_time)
    output_path2 = '{}_{}_{}.csv'.format('好きな', '文字2', date_time)
    output_path3 = '{}_{}_{}.csv'.format('好きな', '文字3', date_time)

    # CSVファイルを作成する
    with open(self.__make_file_path(TMP_PATH, output_path1), 'w') as f1:
      writer = csv.writer(f1, quotechar='"', quoting=csv.QUOTE_ALL, lineterminator="\n")
      writer.writerow(header_record)
      writer.writerow(body_record)

    with open(self.__make_file_path(TMP_PATH, output_path2), 'w') as f2:
      writer = csv.writer(f2, quotechar='"', quoting=csv.QUOTE_ALL, lineterminator="\n")
      writer.writerow(header_record)
      writer.writerow(body_record)

    with open(self.__make_file_path(TMP_PATH, output_path3), 'w') as f3:
      writer = csv.writer(f3, quotechar='"', quoting=csv.QUOTE_ALL, lineterminator="\n")
      writer.writerow(header_record)
      writer.writerow(body_record)

    # ZIPファイル名の例
    # テスト店_月次集計_20190520.zip
    file_name_zip = '{}_{}_{}.zip'.format('ZIP', 'ファイル', date_time)

    # ZIP ファイルを生成
    with zipfile.ZipFile(self.__make_file_path(TMP_PATH, file_name_zip), 'w', compression=zipfile.ZIP_DEFLATED) as new_zip:
      new_zip.write(self.__make_file_path(TMP_PATH, output_path1), arcname=output_path1)
      new_zip.write(self.__make_file_path(TMP_PATH, output_path2), arcname=output_path2)
      new_zip.write(self.__make_file_path(TMP_PATH, output_path3), arcname=output_path3)

    # ZIP ファイルを base64 エンコード
    # ただしそのままだとバイナリなので JSON 形式でレスポンスを返せない
    # -> decode することで文字列として扱うことで JSON 形式に対応させる
    #
    # つまり
    #   binary ファイル読み込み -> base64encode -> decode で文字列化
    # している
    fzip = open(self.__make_file_path(TMP_PATH, file_name_zip), 'br')
    fzip_64encoded = base64.b64encode(fzip.read())
    res: dict = {
        'fileName': file_name_zip,
        'zip': fzip_64encoded.decode('utf-8')
    }

    return res

  def __make_file_path(self, dir_path: str, file_name: str) -> str:
    """ファイルパスを作成する

    Arguments:
      dir_path {str} -- ディレクトリパス
      file_name {str} -- ファイル名

    Returns:
      str -- ファイル名まで含めたパス
    """
    return '{}/{}'.format(dir_path, file_name)

  def __delete_files(self, dir_path: str) -> None:
    """CSVファイル出力後にできたファイルを削除する

    Arguments:
      dir_path {str} -- ディレクトリパス

    Returns:
      None -- なし
    """
    files: list = os.listdir(dir_path)
    for file in files:
      try:
        # tmp ファイルの下は zip と csv しかないのでディレクトリのケアは必要ない
        target = self.__make_file_path(dir_path, file)
        os.remove(target)
      except:
        # 本来こないハズのルート
        # os.remove() ではディレクトリの削除で例外(`PermissionError`)が発生するが
        # まあ発生しても 数kb 程度のゴミが残るだけなので放っておく
        continue

    return None
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

## 出力される ZIP ファイル

- デフォルトのファイル名: `ZIP_ファイル_20210111.zip` (`20210111` の部分は出力日付)
- ファイルの内容
    - CSV ファイルが 3 つ
        - 好きな_文字1_20210111.csv
        - 好きな_文字2_20210111.csv
        - 好きな_文字3_20210111.csv
    - 出力される内容は 3 ファイルすべて同じ

```
"名前","年齢","住所","電話番号","備考"
"ほげ","99歳","ほげ県ほげほげ市","999-9999-9999",""
```

# まとめ

結局のところやっていることは

- [バイナリデータに戻す](#バイナリデータに戻す) の
  1. base64 文字列のデコード(バイナリ変換)
  2. バイナリデータの整数値配列化
  3. Blob オブジェクトの生成

と、そこからの

- [ZIP データの取得とダウンロード](#zip-データの取得とダウンロード) でやっている
  1. URL の生成
  2. ダウンロードファイルの生成
  3. ダウンロード

なのだが、[バイナリデータに戻す](#バイナリデータに戻す) の「1.」～「3.」で何をやっているのかを理解するのが大変だった。

今回の実装にあたり「[JS: base64文字列をBlob形式のFileに変換する](https://qiita.com/rc_code/items/150003f016287750cf34)」の記事が大変参考になりました( というよりコードはそのままですが )。
ありがとうございました。



# ソースコード

今回の記事で作成したコードは 以下にアップしてあるのでご参考まで。

- [フロントエンド](https://github.com/ksh-fthr/angular-work/tree/feat_zip)
- [バックエンド](https://github.com/ksh-fthr/flask-work/tree/feat/zip)


# 参考

- [MDN-window.atob](https://developer.mozilla.org/ja/docs/Web/API/WindowBase64/atob)
- [MDN-Uint8Array](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array)
- [MDN-Blob](https://developer.mozilla.org/ja/docs/Web/API/Blob)
- [MDN-Blob()](https://developer.mozilla.org/ja/docs/Web/API/Blob/Blob)( こちらはコンストラクタの説明 )
- [MDN-String.prototype.charCodeAt()](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/String/charCodeAt)
- [MDN-URL.createObjectURL](https://developer.mozilla.org/ja/docs/Web/API/URL/createObjectURL)
- [MDN-MIME タイプの不完全な一覧](https://developer.mozilla.org/ja/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types)
- [JS: base64文字列をBlob形式のFileに変換する](https://qiita.com/rc_code/items/150003f016287750cf34)
