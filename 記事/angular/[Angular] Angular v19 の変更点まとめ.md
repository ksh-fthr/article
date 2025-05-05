## はじめに

本記事では、Angular v19 のリリースに伴う主な変更点を簡潔にまとめてみました。
新機能や破壊的変更( Breaking Changes )について、コード例とともに解説します。v18 から移行する方、これから試す方の参考になれば幸いです。

※ 内容に不備がありましたらコメントにてお知らせください🙏


次の資料をもとに作成してます。

- release note
  - https://github.com/angular/angular/releases/tag/19.0.0

- blog
  - https://blog.angular.dev/meet-angular-v19-7b29dfd05b84


## Breaking Changes( 破壊的変更 )
### compiler

html テンプレートのコンテキスト変数( `#foo` で定義したもの )に対して、下記のような `this` を使った参照ができなくなります。
Angular のテンプレートでは `this` はコンポーネントのコンテキストにバインドされているため、テンプレート変数にアクセスするには `this` を使わず、直接 `foo` と書く必要があります。


[v18以前はできた]
```html
<ng-template let-foo="someValue">
  {{ this.foo }}  <!-- これは foo を参照できた( 意図しない動作 ) -->
</ng-template>
```

[v19ではできない]
```html
<ng-template let-foo="someValue">
  {{ this.foo }}  <!-- エラー！ -->
  {{ foo }}       <!-- 正しい書き方 -->
</ng-template>
```

- this.foo はコンポーネントのプロパティ `foo` を探しに行く( テンプレート変数 `foo` は無視される )
- テンプレート変数を参照する場合は、 `this.` をつけずに `foo` と書くのが正解


### core　　

**新しく生成される directives, components, pipes はデフォルトで `standalone` になるようになった**

`@NgModule` に含まれる directives, components, pipes は自動的に `standalone: false` と見なされます。これは `ng update` 時に自動で付与されます。
この変更は `ng update` 実行時に自動変換されますが、既存の NgModule ベースの構成も引き続き利用可能です。


**TypeScript v5.5 未満をサポートしなくなった**

単純にサポートするバージョンの話です。


**effect API のタイミングが変わる**

**1. ChangeDetection 外でトリガーされた effect は、ChangeDetection 中に実行される**

[v18以前]
ChangeDetection 外で Signal を変更すると、effect は **microtask( `Promise.resolve().then(...)` )として非同期で実行** されていました。

[v19]
ChangeDetection 外からトリガーされた場合も、**次の ChangeDetection の中で effect が実行されるように** なりました。

[結果として...]
アプリやテストの構成によっては、effect の実行タイミングが早くなったり遅くなったりします。
特にテストでは、effect の発火を待つ必要がある場合に `fixture.detectChanges()` や `fakeAsync + tick()` が追加で必要になることがあります。

**2. ChangeDetection 中でトリガーされた effect はテンプレートより先に実行される**

サンプルを示します。

```ts
@Component({
  selector: 'app-demo',
  template: '{{ mySignal() }}',
})
export class DemoComponent {
  mySignal = signal(0);

  constructor() {
    effect(() => {
      console.log('Effect sees:', this.mySignal());
    });
  }
}
```

[v18以前]
ChangeDetection の最中にテンプレートが更新されてから effect が走っていました。

[v19]
ChangeDetection 中に値が変わった場合、テンプレートが描画される前または同時に effect が走る可能性があります。

[意図と影響]
**副作用が常に最新の状態で実行される**ことを保証しやすくなります。
ただし、副作用で状態を変更するような場合には、**レンダリング順序**の影響に注意が必要です。

**テストでの影響( 補足 )**

v19 のこのタイミング変更により、ユニットテストで次のような違いが出ることがあります。

[v18以前]
```ts
mySignal.set(5);
// microtask の後に effect が実行された
```

[v19]
```ts
mySignal.set(5);
// ChangeDetection のタイミングで effect が実行される
```

したがって、**`detectChanges()` を挟まないと effect が走らない場合がある** ということです。


**[ExperimentalPendingTasks](https://v18.angular.jp/api/core/ExperimentalPendingTasks) が [PendingTasks](https://next.angular.dev/api/core/PendingTasks) に名称変更した**

SSR で効果を発揮するものになります。
`ExperimentalPendingTasks` は **正式 API として昇格** し、名称も `PendingTasks` に変更されました。


**createComponent は、projectableNodes が空の場合にデフォルトのフォールバックをレンダリングするようになった**

createComponent API で `projectableNodes` に空の配列を渡すと `ng-content` のデフォルトのフォールバック・コンテンツが存在する場合にレンダリングされます。
デフォルト・コンテンツをレンダリングしないようにするには、`projectableNode` として `document.createTextNode('')` を渡せばよいです。

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
`[]`( 空の配列 ) を指定した場合 `ng-content` に挿入する内容がないとみなされ、デフォルトコンテンツは表示されませんでした。

[v19]
`[]`( 空の配列 ) を指定した場合でもデフォルトコンテンツを表示するようになった。
ただし、「本当に空の内容で表示したい」場合には `document.createTextNode('')` を使って次のようにすれば良いです。

```ts
createComponent(MyComponent, { projectableNodes: [[document.createTextNode('')]] });
```


**KeyValueDiffers の 非推奨プロパティである `factories` が削除された**

オブジェクトの変更検知や差分処理で使うケースがあるようです。


### router

**Angular 19 では、Router.errorHandler プロパティが削除された**


[v18以前]
```ts
import { Router } from '@angular/router';

// これが v19 で使えなくなる
router.errorHandler = (error) => {
  console.error('Navigation error:', error);
};
```

[v19]
**方法-1. 代替案の 1つ目は provideRouter でwithNavigationErrorHandler を設定する**

```ts
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

```ts
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


**ナビゲーションエラーで Promise の reject を防ぎたい場合は resolveNavigationPromiseOnError を活用する**

```ts
import { provideRouter, withNavigationErrorHandler, resolveNavigationPromiseOnError } from '@angular/router';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes,
      withNavigationErrorHandler((error) => {
        console.error('Navigation error:', error);
      }),

      // Promise の reject を防ぐ
      // つまり ナビゲーションに失敗しても Promise を reject せず、成功( resolve )したものとして扱うときは true を設定する
      resolveNavigationPromiseOnError(true)
    )
  ]
});
```

なお `Router.resolveNavigationPromiseOnError` は オプションであって必須ではありません。
Promise の `reject` を防ぐために使うが、必要なケースは限定的かと思います。


**Resolve インターフェイスの戻り値の型に RedirectCommand が追加された**

Angular 19 では `Resolve<T>` インターフェースの戻り値として `RedirectCommand` がサポートされるようになったとのことです。

従来、 `Resolver` はデータを取得するために使われ、ルートガードのようにリダイレクトすることはできませんでした。
しかし、Angular 19 からは `Resolver` の結果として `RedirectCommand` を返すことで、リダイレクトが可能になります。

[v18以前]
Resolver の戻り値として `Observable<T>` や `Promise<T>` を返すことができました。
またリダイレクトは Router Guard( `canActivate` )を使う必要がありました。

```ts
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
Resolver の `resolve` メソッドで `RedirectCommand` を返すことで、条件に応じてリダイレクト できるようになりました。

```ts
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

ルート設定で上記を利用するコード( resolve を適用 )はこちらです。

```ts
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

### elements

カスタムCD( ChangeDetection ) から hybrid scheduler の切り替えの一環としての変更です。
変更検知の最適化が行われました。

で、最適化したことでエレメントはより効率的になるものの、テストに影響が出る可能性があるので v19 にアップデートしたらテストが通らなくなるかもしれない。


### platform-browser

**SSR に関する変更点**

サーバレンダリングした HTML をブラウザに渡すときに `BrowserModule.withServerTransition` を使っていましたが、これが v19 で削除されました。
v19 では代わりに `APP_ID` DIトークンを使用してアプリケーションIDを設定するようになります。

[v18以前]
```ts
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
```ts
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


### localize

`name` オプションが廃止されて `project` オプションに統合されました。

[v18以前]
```bash
ng add @angular/localize --name=hoge
```

[v19]
```bash
ng add @angular/localize --project=hoge
```


### テスト関連での変更( v19での挙動変更 )

v19 へのアップデートにより、これまで通っていた一部のテストが失敗する可能性があります。

**1. `ComponentFixture` の `autoDetectChanges` の挙動が変更され、 `ErrorHandler` に報告されるように。**

`ComponentFixture` の `autoDetect` 機能が `ApplicationRef` にアタッチされるようになりました。
その結果、自動変更検知中に発生したエラーが `ErrorHandler` に報告されるようになりました。
これにより、 `カスタム ErrorHandler` を設定している場合、今まで報告されなかったエラーが検出される可能性があります。

つまり `autoDetectChanges(true)` を設定すると、テスト中に Angular の変更検知( ChangeDetection )が自動的に実行されます。
これによって変更検知中に発生したエラーが `ErrorHandler` に正しく報告されるようになったので、今まで検出されなかったエラーが `ErrorHandler` で拾われる可能性があるということです。

というわけで、テストの信頼性向上に寄与する変更といえると思います。

**2. `ApplicationRef.tick()` の例外が `TestBed` 実行時に再スローされるように。**

`ApplicationRef.tick()` の実行中にエラーが発生した場合、 `TestBed` でテスト実行時にエラーが再スロー( rethrow )されるようになりました。
これにより変更検知中に発生するエラーがより明示的になり、テストで検出しやすくなりました。

**3. Zone の動作変更により、fakeAsync の制御対象が変わる可能性あり。**

Angular のゾーン外で動作するタイマー( `zone coalescing` や `hybrid mode scheduling` に使われるもの )の動作が変わりました。
これらのタイマーは、"root zone"( グローバルな JavaScript 実行環境 )ではなく、"Angular の 1 つ上のゾーン" で動作するようになったとのことです。
影響があるのは主に `fakeAsync` を使ったテスト で、これらのタイマーが `tick()` や `flush()` によって制御可能になりました。

ただ正直

> Angular のゾーン外で動作するタイマー( zone coalescing や hybrid mode scheduling に使われるもの )の動作が変わった。

この辺がまだ理解があやふやなので、こういう変更があったのだな、という程度の理解度です。

> v19 にアップデートすることでこれまで通っていたテストが通らなくなる可能性がある。

を意識しておこうと思います。


## おわりに

Angular v19 の変更は、全体として standalone と SSR サポートの強化、古い構文の整理が中心です。 また RouterModule などの「将来的な非推奨」も明言されており、移行方針を考える良い機会になりそうです。

テスト周りの変更で意図しないエラーに遭遇する可能性があるため、CI で早めにキャッチできるようにしておくのがおすすめです。
