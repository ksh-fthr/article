
## release note

https://github.com/angular/angular/releases/tag/19.0.0


## blog

https://blog.angular.dev/meet-angular-v19-7b29dfd05b84


## Breaking Changes
### compiler

html テンプレートのコンテキスト変数（`#var` で定義したもの）に対して、下記のような `this` を使った参照ができなくなる。

[v18以前はできた]

```html
<ng-template let-foo="someValue">
  {{ this.foo }}  <!-- これは foo を参照できた (意図しない動作) -->
</ng-template>
```

[v19ではできない]

```html
<ng-template let-foo="someValue">
  {{ this.foo }}  <!-- エラー！ -->
  {{ foo }}       <!-- 正しい書き方 -->
</ng-template>
```

- this.foo はコンポーネントのプロパティ foo を探しに行く（テンプレート変数 foo は無視される）。
- テンプレート変数を参照する場合は、this. をつけずに foo と書くのが正解。


### core　　


　**directives, components, pipe はデフォルトで `standalone` になった**

現在 @NgModules で宣言されている宣言に対して standalone: false を指定するようになる。
この変更は`ng update` で v19 にあげると自動で適用される。


　**TypeScript v5.5 未満をサポートしなくなった**

単純にサポートするバージョンの話。TypeScript v5.6 以降の機能についてもキャッチアップしないとね。


　**effect API のタイミングが変わる**

このブログにくわしい。
https://zenn.dev/lacolaco/articles/angular-v19-effect-changes




　**[ExperimentalPendingTasks](https://v18.angular.jp/api/core/ExperimentalPendingTasks) が [PendingTasks](https://next.angular.dev/api/core/PendingTasks) に名称変更した**

SSR で効果を発揮するものっぽい。これは単純にリネームされただけ。


　**createComponent は、projectableNodes が空の場合にデフォルトのフォールバックをレンダリングするようになった**

createComponent API で `projectableNodes` に空の配列を渡すと `ng-content` のデフォルトのフォールバック・コンテンツが存在する場合にレンダリングされる。
デフォルト・コンテンツをレンダリングしないようにするには、`projectableNode` として `document.createTextNode('')` を渡せばよい。

[サンプルコード]

```ts
createComponent(MyComponent, { projectableNodes: [[]] });
```

というコードがあって html 側で `ng-content` を次のように指定した場合...

```html
<div>
  <ng-content>デフォルトのコンテンツ</ng-content>
</div>
```

[v18以前]
`デフォルトのコンテンツ` が表示されなかった。

[v19]
`デフォルトのコンテンツ` が表示される。
v19 で表示したくない場合は次のようにする。

```ts
createComponent(MyComponent, { projectableNodes: [[document.createTextNode('')]] });
```

ってことだけれど、まぁ `ng-content` 指定したときに ts で createComponent することってそうないと思うのだがどうか。
単体テストでは書きまくっているけれど。


　**KeyValueDiffers の 非推奨プロパティである `factories` が削除された**

ということだけれど、普段使いでみたことない。
オブジェクトの変更検知や差分処理で使うケースがあったらしい。


　**テスト関連での変更**

v19 にアップデートすることでこれまで通っていたテストが通らなくなる可能性がある。

[そのいち]
ComponentFixture の autoDetect 機能が ApplicationRef にアタッチされるようになった。
その結果、自動変更検知中に発生したエラーが ErrorHandler に報告されるようになった。
これにより、カスタム ErrorHandler を設定している場合、今まで報告されなかったエラーが検出される可能性がある。

つまり autoDetectChanges(true) を設定すると、テスト中に Angular の変更検知 (Change Detection) が自動的に実行される。
で、変更検知中に発生したエラーが ErrorHandler に正しく報告されるようになったので、今まで検出されなかったエラーが ErrorHandler で拾われる可能性がある。

というわけで、テストの信頼性向上に寄与するっぽい。

[そのに]
ApplicationRef.tick() の実行中にエラーが発生した場合、TestBed でテスト実行時にエラーが再スロー（rethrow）されるようになった。
これにより、変更検知中に発生するエラーがより明示的になり、テストで検出しやすくなった。

[そのさん]
Angular のゾーン外で動作するタイマー（zone coalescing や hybrid mode scheduling に使われるもの）の動作が変わった。
これらのタイマーは、"root zone"（グローバルな JavaScript 実行環境）ではなく、"Angular の 1 つ上のゾーン" で動作するようになった。
影響があるのは主に fakeAsync を使ったテスト で、これらのタイマーが tick() や flush() によって制御可能になった。

でも正直

> Angular のゾーン外で動作するタイマー（zone coalescing や hybrid mode scheduling に使われるもの）の動作が変わった。

この辺、まだ理解があやふや。
まぁ

> v19 にアップデートすることでこれまで通っていたテストが通らなくなる可能性がある。

を意識しておこう。


### elements

カスタムCD(ChangeDetection) から hybrid scheduler の切り替えの一環としての変更。
変更検知の最適化が行われたのだけれども、変更検知のタイミングが微妙に変更されたことでもある。

最適化したことでエレメントはより効率的になるが、テストに影響が出る可能性があるので v19 にアップデートしたらテストが通らなくなるかもしれない。

> カスタムCD(ChangeDetection)

これよくわからなかった。普段使っていないからなぁ。
調べます。


### localize


`name` オプションが廃止されて `project` オプションに統合された。

[v18以前]

```
ng add @angular/localize --name=hoge
```

[v19]

```
ng add @angular/localize --project=hoge
```




### platform-browser

SSR の話。
サーバレンダリングした HTML をブラウザに渡すときに `BrowserModule.withServerTransition` を使っていたが、これが v19 で削除された。
v19 では代わりに `APP_ID` DIトークンを使用してアプリケーションIDを設定する。

[v18以前]

```
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule.withServerTransition({ appId: 'my-app' }) // サーバーからクライアントへの遷移を有効にする
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

[v19]

```
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';
import { APP_ID } from '@angular/core';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule], // 変更点: withServerTransition は不要に
  providers: [{ provide: APP_ID, useValue: 'my-app' }], // APP_ID トークンで appId を設定
  bootstrap: [AppComponent]
})
export class AppModule {}
```


### router


　**Angular 19 では、Router.errorHandler プロパティが削除された**


[v18以前]

```
import { Router } from '@angular/router';

// これが v19 で使えなくなる
router.errorHandler = (error) => {
  console.error('Navigation error:', error);
};
```

[v19]
**方法-1. 代替案の 1つ目は provideRouter でwithNavigationErrorHandler を設定する**

```
import { provideRouter, withNavigationErrorHandler } from '@angular/router';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes, withNavigationErrorHandler((error) => {
      console.error('Navigation error:', error);
    }))
  ]
});
```

**方法-2. 代替案の 2つ目は RouterModule.forRoot の追加オプションで errorHandler プロパティを設定する**

```
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [ /* ルート定義 */ ];

@NgModule({
  imports: [
    RouterModule.forRoot(routes, {
      errorHandler: (error) => {
        console.error('Navigation error:', error);
      }
    })
  ],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

**promise のエラーをキャンセルしたい場合は resolveNavigationPromiseOnError を活用する**

```
import { provideRouter, withNavigationErrorHandler, resolveNavigationPromiseOnError } from '@angular/router';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes,
      withNavigationErrorHandler((error) => {
        console.error('Navigation error:', error);
      }),

      // Promise の reject を防ぐ
      // つまり ナビゲーションに失敗しても Promise を reject せず、成功（resolve）したものとして扱うときは true を設定する
      resolveNavigationPromiseOnError(true)
    )
  ]
});
```


　**Resolveインターフェイスの戻り値の型にRedirectCommandが追加された**


Angular 19 では Resolve<T> インターフェースの戻り値として RedirectCommand がサポートされるようになったとのこと。

従来、Resolver はデータを取得するために使われ、ルートガードのようにリダイレクトすることはできませんでした。
しかし、Angular 19 からは Resolver の結果として RedirectCommand を返すことで、リダイレクトが可能になります。

[v18以前]

- Resolver の戻り値として Observable<T> や Promise<T> を返すことができた
- リダイレクトは Router Guard (canActivate) を使う必要があった

```
import { Resolve } from '@angular/router';
import { Injectable } from '@angular/core';
import { Observable, of } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class DataResolver implements Resolve<string> {
  resolve(): Observable<string> {
    return of('Resolved data');
  }
}
```

[v19]

- Resolver の resolve メソッドで RedirectCommand を返すことで、条件に応じてリダイレクト できるようになった

```
import { Resolve, RedirectCommand } from '@angular/router';
import { Injectable } from '@angular/core';
import { Observable, of } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class DataResolver implements Resolve<string | RedirectCommand> {
  resolve(): string | RedirectCommand {
    const userLoggedIn = false; // 例：ユーザーがログインしているか
    if (!userLoggedIn) {
      return { navigationCommands: ['/login'] }; // ログインページへリダイレクト
    }
    return 'Resolved data'; // 通常のデータ取得
  }
}
```

- ルート設定で上記を利用するコード( resolve を適用 )

```
import { Routes } from '@angular/router';
import { DataResolver } from './data.resolver';

const routes: Routes = [
  {
    path: 'protected',
    component: ProtectedComponent,
    resolve: { data: DataResolver } // Resolver にリダイレクト機能を追加
  }
];
```
