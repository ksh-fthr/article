# [Angular] HttpClient で取得したデータを Signal で扱う際にやったこと

# はじめに

 タイトルのとおり [HttpClient](https://angular.io/api/common/http/HttpClient) で取得したデータを Angular Signal で扱ってみる試みです｡
 [HttpClient では Observable を返します](https://angular.io/guide/observables-in-angular#http) が､それを Signal で扱うにはどうすればよいか､がこの記事の主眼です｡

なお本記事で扱うコードにはエラー処理を施しておりません｡
あらかじめご承知おきください｡

# 環境

今回の記事の内容は次の環境で実施したものです。

| 環境                                                        | バージョン | 備考                     |
| ----------------------------------------------------------- | ---------- | ------------------------ |
| [Angular CLI](https://cli.angular.io/)                      | v17.3.6    | `ng version` で確認      |
| [Angular](https://angular.io/)                              | v17.3.7    | 同上                     |
| [TypeScript](https://www.typescriptlang.org/)               | v5.4.5     | 同上                     |
| [zone.js](https://www.npmjs.com/package/zone.js)            | v0.14.5    | 同上                     |
| [Node.js](https://nodejs.org/ja/)                           | v18.19.0   | 同上                     |
| [npm](https://www.npmjs.com/)                               | v10.5.1    | 同上                     |


# 修正したコードと役割

修正したコードは `service.ts`, `component.ts`, `compoonent.html` です｡
それぞれの役割は次のとおりです｡

- `service.ts`
  - HttpClient を使って API を実行
  - API の実行結果( レスポンス )をコール元に返却する
- `component.ts`
  - service 経由で API から返却されたレスポンスを html テンプレートで参照する変数にセットする
- `compoonent.html`
  - component.ts でセットされた変数を参照して描画する

# 本記事の構成

本記事では `service.ts`, `component.ts`, `compoonent.html` の修正前後のコードを挙げて差分を見ていきます｡

# 修正前後のコード比較(service.ts)

## GET

**修正前**

```ts
  public get$(): Observable<MessageModel[]> {
    return this.http.get<HttpResponseBodyModel>(this.host + '/message/get', this.httpOptions).pipe(
      map((response) => {
        const res: any = response;
        const resBody: HttpResponseBodyModel = res.body;
        return resBody.messages;
      })
    );
  }
```

**修正後**

```ts
  public get$(): Signal<MessageModel[] | undefined> {
    const observable = this.http.get<HttpResponseBodyModel>(this.host + '/message/get', this.httpOptions).pipe(
      map((response) => {
        const res: any = response;
        const resBody: HttpResponseBodyModel = res.body;
        return resBody.messages;
      })
    );

    return toSignal(observable);
  }
```

大きな違いは `Observable` を返却するか `Signal` を返却するかです｡
修正前では HttpClient の実行結果( レスポンス )を一部加工して､そのままメソッドの戻り値としているのに対し､修正後では [`toSignal`](https://angular.io/api/core/rxjs-interop/toSignal) で [Signal](https://angular.io/api/core/Signal) に変換して返却しています｡

これにより､呼び出しもとでは `GET` で得た値を `Signal` で扱うことができるようになりました｡

以降の `POST`, `PUT`, `DELETE` についても基本的な修正内容は同じですので､それぞれの項目での説明は省きます｡

## POST

**修正前**

```ts
  public register$(body: MessageModel): Observable<MessageModel[]> {
    return this.http.post(this.host + '/message/post', body, this.httpOptions).pipe(
      map((response) => {
        const res: any = response;
        const resBody: HttpResponseBodyModel = res.body;
        return resBody.messages;
      })
    );
  }
```

**修正後**

```ts
  public register$(body: MessageModel): Signal<MessageModel[] | undefined> {
    const observable = this.http.post(this.host + '/message/post', body, this.httpOptions).pipe(
      map((response) => {
        const res: any = response;
        const resBody: HttpResponseBodyModel = res.body;
        return resBody.messages;
      })
    );

    return toSignal(observable);
  }
```

## PUT

**修正前**

```ts
  public update$(body: MessageModel): Observable<MessageModel[]> {
    return this.http.put(this.host + '/message/put', body, this.httpOptions).pipe(
      map((response) => {
        const res: any = response;
        const resBody: HttpResponseBodyModel = res.body;
        return resBody.messages;
      })
    );
  }
```

**修正後**

```ts
  public update$(body: MessageModel): Signal<MessageModel[] | undefined> {
    const observable = this.http.put(this.host + '/message/put', body, this.httpOptions).pipe(
      map((response) => {
        const res: any = response;
        const resBody: HttpResponseBodyModel = res.body;
        return resBody.messages;
      })
    );

    return toSignal(observable);
  }
```

## DELETE

**修正前**

```ts
  public delete$(body: MessageModel): Observable<MessageModel[]> {
    this.httpOptions.body = body;
    return this.http.delete(this.host + '/message/delete', this.httpOptions).pipe(
      map((response) => {
        const res: any = response;
        const resBody: HttpResponseBodyModel = res.body;
        return resBody.messages;
      })
    );
  }
```

**修正後**

```ts
  public delete2$(body: MessageModel): Signal<MessageModel[] | undefined> {
    this.httpOptions.body = body;
    const observable = this.http.delete(this.host + '/message/delete', this.httpOptions).pipe(
      map((response) => {
        const res: any = response;
        const resBody: HttpResponseBodyModel = res.body;
        return resBody.messages;
      })
    );

    return toSignal(observable);
  }
```

# 修正前後のコード比較(component.ts)

## 共通

**修正後のコードだけにある処理**

```ts
export class HttpClientVerificationComponent implements OnInit {
  injector = inject(EnvironmentInjector);
  // ...(略)...
}
```

こちらは後述の [補足-2](#補足-2) で触れている [`runInInjectionContext`](https://angular.io/api/core/runInInjectionContext) で使用するために必要な処理を追加しています｡


## GET

**修正前後で差分なし**

```ts
  public messageInfoList$ = this.httpClientService.get$();
```

`GET` における差分はありません｡ `Observable` を扱っていたときと同じコードで､そのまま `Signal` が返却されるコードにも対応できました｡

## POST

**修正前**

```ts
  public onClickRegister(event: any): void {
    // ..(略)..
    this.messageInfoList$ = this.httpClientService.register$(body);
  }
```

**修正後**

```ts
  public onClickRegister(event: any): void {
    // ..(略)..
    runInInjectionContext(this.injector, () => {
      this.messageInfoList$ = this.httpClientService.register$(body);
    });
  }
```

この次に続く `PUT` や `DELETE` も同じなのですが､これらの処理はクリックイベントから実行されます｡( [補足-1](#補足-1) )
そのためか 修正前のコード では [`NG0203`](https://angular.io/errors/NG0203) が発生しました｡ この修正は当該エラーに対応するためのものです｡
詳しくは [補足-2](#補足-2) をご参照ください｡

なお `PUT`, `DELETE` についても基本的な修正内容は同じですので､それぞれの項目での説明は省きます｡


## PUT

**修正前**

```ts
  public onClickUpdate(event: any): void {
    // ..(略)..
    this.messageInfoList$ = this.httpClientService.update$(body);
  }
```

**修正後**

```ts
  public onClickUpdate(event: any): void {
    // ..(略)..
    runInInjectionContext(this.injector, () => {
      this.messageInfoList$ = this.httpClientService.update$(body);
    });
  }
```

## DELETE

**修正前**

```ts
  public onClickDelete(event: any): void {
    // ..(略)..
    this.messageInfoList$ = this.httpClientService.delete$(body);
  }
```

**修正後**

```ts
  public onClickDelete(event: any): void {
    // ..(略)..
    runInInjectionContext(this.injector, () => {
      this.messageInfoList$ = this.httpClientService.delete$(body);
    });
  }
```

## 補足-1

`GET` 以外の HTTP メソッドは UI からのボタンクリックイベントで実行されます｡


## 補足-2

[修正前後のコード比較(service.ts)](#修正前後のコード比較servicets) に挙げたコードのとおり､本記事の `POST`, `PUT`, `DELETE` では､ API 実行後のレスポンスを `Signal` で返却しています｡
そして､それを利用している component 側では返却されたレスポンスを `messageInfoList$` にセットします｡
( `messageInfoList$` は `GET` 時も参照している変数です )

`messageInfoList$` へ値を設定するにあたり `GET` と同じように修正前後で差分なし､つまり修正なしで対応できるかと思ったのですが､実行時に次のエラーが発生しました｡

```text
NG0203: toSignal() can only be used within an injection context such as a constructor, a factory function, a field initializer, or a function used with `runInInjectionContext`.
```

どうやら **`toSignal` をクリックイベントで扱うことは NG** のようです｡
このエラーに対応するために `POST`, `PUT`, `DELETE` の API を実行する際に次の修正を施しました｡
( 下記は `POST` の例 )

> ```ts
>  public onClickRegister(event: any): void {
>    runInInjectionContext(this.injector, () => {
>      this.messageInfoList$ = this.httpClientService.register$(body);
>    });
>  }
> ```

修正の際は [こちら](https://github.com/angular/angular/issues/50947) を参考にしました｡

# 修正前後のコード比較(component.html)

**修正前**

```html
    @for ((messageInfo of messageInfoList$ | async); track messageInfo; let i = $index) {
      <tr>
        <td>{{messageInfo.id}}</td>
        <td>{{messageInfo.message}}</td>
      </tr>
    }
```

**修正後**

```html
    <!-- messageInfoList$ を参照するさいに `()` を付与している点に注目 -->
    @for ((messageInfo of messageInfoList$()); track messageInfo; let i = $index) {
      <tr>
        <td>{{messageInfo.id}}</td>
        <td>{{messageInfo.message}}</td>
      </tr>
    }
```

手前味噌で恐縮ですが､ [こちら](https://qiita.com/ksh-fthr/items/54f5e63cdeade8b9c474#%E5%B7%AE%E5%88%86childcomponenthtml) で触れておりますとおり､ Signal に対して参照する際は `()` を付与しないとエラーになります｡コンパイルも通りません｡
以下は `()` を付与しなかったときのエラー内容です｡

```text
Type 'Signal<MessageModel[] | undefined>' must have a '[Symbol.iterator]()' method that returns an iterator.ngtsc(2488)
http-client-verification.component.ts(9, 44): Error occurs in the template of component HttpClientVerificationComponent.
```


# まとめにかえて

以上､ HttpClient で取得したデータを Signal で扱う試みでした｡
キモとなるポイントは以下になるかと思います｡

- [`toSginal`](https://angular.io/api/core/rxjs-interop/toSignal) での変換
- クリックイベントで `toSignal` を扱うと [`NG0203`](https://angular.io/errors/NG0203) が発生する
- その対応のために [`runInInjectionContext`](https://angular.io/api/core/runInInjectionContext) でラップする
- Signal を html テンプレートで参照する際は `()` を付与してメソッド呼び出しとする

対応方法が分かってしまえば ( 修正差分をご覧のとおり ) 大きな修正は必要とせず､ Signal に変換可能であることが分かりました｡


# ソースコード

修正差分やソースコードは以下からご覧になれます｡
ご興味あればどうぞご確認ください｡

## 差分( PR )
- [http-client のレスポンス(Observable) を Signal で扱う. #626](https://github.com/ksh-fthr/angular-work/pull/626)

## リポジトリ内のコード

- [service](https://github.com/ksh-fthr/angular-work/blob/develop/src/app/service/http-client/http-client.service.ts)
- [component.ts](https://github.com/ksh-fthr/angular-work/blob/develop/src/app/component/http-client/http-client-verification/http-client-verification.component.ts)
- [component.html](https://github.com/ksh-fthr/angular-work/blob/develop/src/app/component/http-client/http-client-verification/http-client-verification.component.html)

# 参考

HttpClient を Signal で扱うにあたり､次の記事を参考にさせていただきました｡
大変助かりました｡ 深く感謝申し上げます｡

- Angular Signals
  - [Angular Signals スタートガイド](https://codelabs.developers.google.com/angular-signals?hl=ja#0)
  - [Angular Signals: Complete Guide](https://blog.angular-university.io/angular-signals/)

- HttpClient を Signal で扱うために参考にしたページ
  - [Analyze ways to retrieve data with signals and HttpClient in Angular](https://dev.to/railsstudent/analyze-ways-to-retrieve-data-with-signals-and-httpclient-in-angular-2je7)
  - [Angular Signals and HttpClient](https://www.techiediaries.com/angular-signals-httpclient/)

- エラー解決にあたり参考にしたページ
  - [Can toSignal be used in ngOnInit? #50947](https://github.com/angular/angular/issues/50947)
  - [NG0203: `inject()` must be called from an injection context such as a constructor, a factory function, a field initializer, or a function used with `runInInjectionContext`.](https://angular.io/errors/NG0203)
