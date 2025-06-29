## 目次

1. [はじめに](#はじめに)
2. [Breaking Changes](#breaking-changes)
   2.1 [古い構文や属性の非推奨化](#1-古い構文や属性の非推奨化)
   2.2 [`void` 演算子のサポート強化](#2-void-演算子のサポート強化)
   2.3 [html テンプレートで指数演算子を利用可能に](#3-html-テンプレートで指数演算子を利用可能に)
   2.4 [ファイル・クラス命名規則の変更](#4-ファイルクラス命名規則の変更)
   2.5 [`ng-reflect-*` 属性の削除](#5-ng-reflect--属性の削除)
   2.6 [自動 CSP サポート導入](#6-自動-csp-サポート導入)
   2.7 [テンプレート式と診断機能の強化](#7-テンプレート式と診断機能の強化)
   2.8 [`formatDate()` の週年パターン検証](#8-formatdate-の週年パターン検証)
   2.9 [`ViewportScroller` API 拡張](#9-viewportscroller-api-拡張)
3. [まとめ](#まとめ)
   3.1 [主な Breaking Changes 一覧](#主な-breaking-changes)
   3.2 [Deprecations (非推奨)](#deprecations)
   3.3 [破壊的変更・非推奨の一覧表](#表にしてみました)


## はじめに

本記事では、Angular v20 のリリースに伴う主な変更点を簡潔にまとめてみました。

:::note warn

⚠️ 注意 ⚠️
本記事は Angular v20 の公式リリースノートや、各種解説記事を参考に筆者が独自に整理・要約したものであり、誤りが含まれる可能性もあります。
公式情報については [公式 GitHub](https://github.com/angular/angular/releases/tag/20.0.0) を必ずご確認ください。

:::

※ 内容に不備がありましたらコメントにてお知らせください 🙏

次の資料をもとに作成してます。

- **公式リリースノート**:
  -  [Angular 20.0.0](https://github.com/angular/angular/releases/tag/20.0.0)
- **公式ブログ記事**:
  -  [Announcing Angular v20](https://blog.angular.dev/announcing-angular-v20-b5c9c06cf301)
- **コミュニティ記事**:
  - [Angular 20 – What’s New](https://angular.love/angular-20-whats-new)
  - [Angular v20: Key features you need to know](https://medium.com/@satvatiyasin/angular-v20-key-features-you-need-to-know-7e1aa2528625)

## Breaking Changes

### 1. 古い構文や属性の非推奨化
`*ngIf`、`*ngFor`、`*ngSwitch`、`<ng-template>` といった従来の構造ディレクティブは、v20から将来的な非推奨・廃止予定とされ、代替として新構文 `@if`, `@for`, `@switch` が導入されています。  
（出典：Angular v20 のコミュニティ記事要約）[angular.love](https://angular.love/angular-20-whats-new)


**`*ngIf` -> `@if`**

[v19 以前(**非推奨**)]

```html
<div *ngIf="isLoggedIn">
  Welcome!
</div>
```

[v20 以降]

```html
@if(isLoggedIn){
  <div>Welcome!</div>
} @else {
  <div>Please log in.</div>
}
```

**`*ngFor` -> `@for`**
[v19 以前(**非推奨**)]

```html
<ul>
  <li *ngFor="let item of items">{{ item }}</li>
</ul>
```

[v20 以降]

**track関数なし**
```html
<ul>
  @for (item of items) {
    <li>{{ item }}</li>
  }
</ul>
```

**track関数あり**
```html
<ul>
  @for (item of items; track(item) => item.id) {
    <li>{{ item }}</li>
  }
</ul>
```

※ `track(item) => item.id` は従来の `trackBy` に相当する記述方法で、差分レンダリングに使われます。


**`*ngSwitch` -> `@switch`**
[v19 以前(**非推奨**)]

```html
<div [ngSwitch]="status">
  <p *ngSwitchCase="'loading'">Loading...</p>
  <p *ngSwitchCase="'success'">Success!</p>
  <p *ngSwitchDefault>Unknown status</p>
</div>
```

[v20 以降]

```html
@switch(status){
  @case('loading'){ <p>Loading...</p> }
  @case('success'){ <p>Success!</p> }
  @default { <p>Unknown status</p> }
}
```

**`<ng-template>`**

`<ng-template>` はほぼ不要となり、今後の使用は非推奨が予想されます。
（背景説明も同じくコミュニティ記事より要約）[angular.love](https://angular.love/angular-20-whats-new)


:::note warn

⚠️ 注意 ⚠️

- `@if` / `@for` / `@switch` は Angular v20 から 正式にサポートされました
- 従来の構文 ( `*ngIf` など )は v20 ではまだ使用可能ですが、将来的な削除が予定されています
- Angular コンパイラが `@` を識別するため、テンプレート構文の一部として `@...` を特別に扱います

:::

### 2. void 演算子のサポート強化

テンプレート内での **`void`** 演算子サポートが強化されました。

```ts
// component.ts
export class AppComponent {
    doSomething(): void {
        console.log("Clicked!");
    }
}
```

```html
<!-- template.html -->
<button (click)="void doSomething()">Click Me</button>
```

### 3. html テンプレートで指数演算子を利用可能に

html テンプレートで指数演算子 ( `**` )の直接使用が可能になりました。

```ts
// component.ts
export class AppComponent {
    base = 2;
    exponent = 5;
}
```

```html
<!-- template.html -->
<!-- 2 ** 5 となって 32 が出力される -->
<p>{{ base ** exponent }}</p>
```

### 4. ファイル・クラス命名規則の変更

Angular CLI による自動生成で従来末尾についていた `Component`, `Service`, `Directive`, `Pipe` のサフィックスがデフォルトで**廃止**され、クリーンな命名へと変わりました。
例えば、my-card.component.ts の場合は次のようになります。

[v19 以前(**非推奨**)]

```ts
// ファイル名は my-card.component.ts で生成され、クラス名は MyCardComponent だった.
export class MyCardComponent {
}
```

[v20 以降]
```ts
// ファイル名は my-card.ts で生成され、クラス名は MyCard になる.
export class MyCard {
}
```

### 5. `ng-reflect-*` 属性の削除
開発中に出力されていた `ng-reflect-*` 属性はデフォルトで出力されなくなりました。必要な場合は `provideNgReflectAttributes()` を追加してください。

つまり...
HTML に表示されていた `ng-reflect-*` 属性に依存するテストケースがある場合は、それらのテストを修正する必要があります。
が、まだ使用したい場合はアプリケーションの起動時に **`provideNgReflectAttributes()` オプションを追加する** ことで **`**ng-reflect-*` を有効** にすることができます：

```ts
bootstrapApplication(AppComponent, {
    providers:[
        // これを追加することでこれまでと同じ動きとなる
        // -> ng-reflect-* が有効になる
        provideNgReflectAttributes()
    ]
});
```

### 6. 自動 CSP サポート導入

`angular.json` に `security.autoCsp: true` を指定することで、本番ビルド時に **Content Security Policy** が自動的に設定されます。

```json
"options": {
  "security": {
    "autoCsp": true
  }
}
```

### 7. テンプレート式と診断機能の強化

`@angular/compiler` が下記にしめすようなテンプレート式に対するコンパイル時警告／エラーを追加するようになりました。

**`@for` における track関数 の呼び出し忘れ**
track関数が参照されているのに呼び出されず、不必要なDOMの再作成を引き起こすパフォーマンス問題を防ぐ一助になります。

[誤り]

```ts
@Component({
  template: `
    <!-- これだと参照だけで呼び出していない -->
    @for(item of items; track trackByName) {
      <div>{{ item.name }}</div>
    }
  `
})
export class ListComponent {
  items = [{ name: 'Hoge' }, { name: 'Foo' }];
  
  // 呼ばれない
  trackByName(item: any){
    return item.name;
  }
}
```

[正しい]

```ts
@Component({
  template: `
    <!-- track関数をちゃんと呼び出している-->
    @for (item of items; track(item) => trackByName(item)) {
      <div>{{ item.name }}</div>
    }
  `
})
export class ListComponent {
  items = [{ name: 'Hoge' }, { name: 'Foo' }];

  // 呼ばれる
  trackByName(item: any): string {
    return item.name;
  }
}
```

**nullish-coalescing(`??`)と論理演算子(`&&`／`||`)の非括弧混合使用に対する `NG8114` 診断**
新しい拡張診断( `NG8114` )が導入されました。

論理演算子( `&&` と `||` )に `??` が混在している場合に、演算子の優先順位が曖昧になる可能性を検出するようになりました。
解決策として、常に括弧を使用して意図する操作の順序を明示的に定義する、という対応が必要になります。

**構造ディレクティブのインポート漏れに対する `NG8116` 警告**
新しい拡張診断( `NG8116` )が導入されました。

スタンドアロンコンポーネントのカスタム構造ディレクティブのインポート漏れを特定するための診断です。
この診断によりランタイムエラーが防止され、カスタム構造ディレクティブを使用する際の開発者エクスペリエンスが向上します。


### 8. `formatDate()` の週年パターン検証

開発モードで、 `YYYY`( 週年 )を誤って使用した場合にエラーがスローされるようになりました( 週指定なしの `Y` 使用など )。

[誤り]

```ts
// 誤り( 週番号なし )
formatDate(new Date('2024-12-31'), 'YYYY-MM-dd', 'en');
```

[正しい]

```ts
// 正しくは:
formatDate(new Date('2024-12-31'), 'yyyy-MM-dd', 'en');
```

[なぜこれがエラーなのか...]
「**週単位の年( ISO 週番号付きの年 )は、1月1日前後の数日間だけ暦年と異なる**」というのが理由です。
つまり「週単位の年( ISO 週番号付きの年 )」は「その週がどの年の週としてカウントされるか」という概念なので、カレンダー年( 暦年 )と数日間ズレることがあり、特に年末年始の週ではズレが発生しやすくなる、ということです。

[要は...]
`Y` が示す内容を理解して `Y` を使っていればよいが、大抵の場合 `Y` ではなく `y` を意図しているだろうから、開発モードでそれを検知してくれる、というわけですね。

### 9. `ViewportScroller` API 拡張

 `scrollToPosition` と `scrollToAnchor` に **`ScrollOptions`** が追加され、スムーズスクロールなど高度な制御が可能になりました。

[v19以前]

```ts
abstract class ViewportScroller {
  abstract setOffset(offset: [number, number] | (() => [number, number])): void;
  abstract getScrollPosition(): [number, number];
  //
  // scrollToPosition と scrollToAnchor で ScrollOptions は利用できなかった
  //
  abstract scrollToPosition(position: [number, number]): void;
  abstract scrollToAnchor(anchor: string): void;
  abstract setHistoryScrollRestoration(scrollRestoration: "auto" | "manual"): void;
}
```

[v20 以降]

```ts
abstract class ViewportScroller {
  abstract setOffset(offset: [number, number] | (() => [number, number])): void;
  abstract getScrollPosition(): [number, number];
  //
  // scrollToPosition と scrollToAnchor で ScrollOptions が利用可能になった
  //
  abstract scrollToPosition(position: [number, number], options?: ScrollOptions | undefined): void;
  abstract scrollToAnchor(anchor: string, options?: ScrollOptions | undefined): void;
  abstract setHistoryScrollRestoration(scrollRestoration: "auto" | "manual"): void;
}
```

[ScrollOptions の interface と使用例]

```ts
interface ScrollOptions {
  behavior?: 'auto' | 'instant' | 'smooth';
  block?: 'start' | 'center' | 'end' | 'nearest';
  inline?: 'start' | 'center' | 'end' | 'nearest';
}
```

```ts
this.viewportScroller.scrollToAnchor('contact', {
  behavior: 'smooth',
  block: 'start',
  inline: 'nearest'
});
```

---

## まとめ

### 主な Breaking Changes

#### common

- `Y`( 週番号付きの年 )フォーマット使用時、`w`( 週番号 )なしの使用が警告対象に
- `AsyncPipe` が未処理エラーをアプリの `ErrorHandler` に直接伝播するよう変更。Zoneベース環境と動作は同等だが、テスト環境で差異が生じる可能性あり

#### compiler

- テンプレート内の `in` / `void` は、オペレータとして解釈されるように変更
-  `{{void}}` → 無効に( 以前はクラスのプロパティとして参照された )→ `{{this.void}}` に変更必要

#### core

**TypeScript/Node.js サポートの変更**

- TypeScript v5.8未満はサポート終了
- Node.js v18および v22.0〜22.10 はサポート対象外。v20.11.1以上が必要

**TestBed 関連**

- `TestBed.flushEffects()` 削除 → `TestBed.tick()` を使用
- `TestBed.get()` 削除( 代わりに `TestBed.inject()` を使用 )
- `injector.get()` の `any` オーバーロードが削除され、`ProviderToken<T>` のみが許容されるように変更
- `InjectFlags` は削除され、関連 API は対応しなくなった

**API リネーム・仕様変更**

- `provideExperimentalCheckNoChangesForDebug` → `provideCheckNoChangesConfig` にリネーム
- `provideExperimentalZonelessChangeDetection` → `provideZonelessChangeDetection`( Developer Preview に昇格 )
- `afterRender` → `afterEveryRender` に名称変更
- `PendingTasks.run()` は async 関数の戻り値を返さなくなった( 手動再実装が必要な場合あり )

**エラーハンドリング動作の変更**

- イベントリスナの未処理エラーは、ErrorHandlerだけでなくAngular内部でも処理
- `ApplicationRef.tick()` はエラーをキャッチせずスロー → 呼び出し側で明示的な処理が必要

**アニメーションと変更検知**

- アニメーションは変更検知時や `ApplicationRef.tick()` 呼び出し時に確実に flush されるように変更
- DOM 状態に依存するテストに影響する可能性あり

**`ng-reflect-*` 属性の削除**

- `ng-reflect-*` 属性が出力されなくなった( デフォルト )
- 必要に応じて `provideNgReflectAttributes()`( @angular/core から import )で dev モード時に再有効化可

#### router

- `RedirectFn` は `Observable` / `Promise` を返せるように
- いくつかの `Router` API は `readonly array` に対応( ミューテーションしない前提 )
- ルートガードの型定義において `any` は除外( `string` は非推奨ながら残存 )

### Deprecations

#### core

- `ngIf` / `ngFor` / `ngSwitch` は非推奨 → `@if` / `@for` / `@switch` に移行推奨

#### platform-browser

- `@angular/platform-browser-dynamic` のすべてのエントリが非推奨
- `HammerJS` サポートは非推奨( 将来的に削除予定 )

#### platform-server

- `@angular/platform-server/testing` は非推奨 → SSR 確認は E2E テストを使用することが推奨


### 表にしてみました

以下に Angular v20 における **破壊的変更( Breaking Changes )** と **非推奨( Deprecations )となったもの** をまとめました。

**Breaking Changes**

| 変更対象                | 概要                                                                                                                     | 備考・対応方法／注意点                                            |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------- |
| `common`                | `Y`( 週番号付き年 )+ `w`( 週番号 )未使用パターンが警告対象になった                                                       | 日付フォーマット見直し                                            |
| `AsyncPipe`             | 未処理エラーを `ErrorHandler` に直接送出するようになった                                                                 | テスト環境でのエラー捕捉に注意                                    |
| `compiler`              | `in` / `void` はオペレータとして解釈する                                                                                 | `{{void}}` → `{{this.void}}` に変更                               |
| TypeScript/Node.js      | TypeScript 5.8 未満および Node.js v18, v22.0〜22.10 をサポート終了                                                       | Node.js v20.11.1 以上が必要                                       |
| `TestBed` 関連          | `flushEffects()` 削除、`get()` 削除、`inject()` のみサポート                                                             | `tick()` へ移行                                                   |
| `InjectFlags`           | 完全削除( 複数 API で非対応 )                                                                                            | `inject()` や `Injector.get()` で使用不可                         |
| API 名称変更            | `afterRender` → `afterEveryRender`、`provideExperimentalZonelessChangeDetection` → `provideZonelessChangeDetection` など | 旧名称の置換対応が必要                                            |
| `PendingTasks`          | `run()` が戻り値を返さなくなる                                                                                           | `PendingTasks.add()` による手動処理必要                           |
| イベントエラー処理      | イベント中の未処理エラーが Angular 内部でも扱われ、テストで rethrow される                                               | エラーが表面化するため要対応                                      |
| `ApplicationRef.tick()` | エラーを `ErrorHandler` 経由でなく、呼び出し元にスローする                                                                   | 呼び出し元で try-catch 等で制御必要                               |
| アニメーション          | Angular の検出タイミングでアニメーションが確実に flush されるようになった                                                | DOM 依存テストに影響あり                                          |
| `ng-reflect-*` 属性     | デフォルトで出力されなくなった                                                                                           | 必要時 `provideNgReflectAttributes()` により dev モードで再出力可 |
| `router`                | `RedirectFn` で `Observable` / `Promise` 返却可能になった                                                                | 呼び出し元コードの対応が必要                                      |
| `Router` API            | `readonly array` に対応、一部 `any` 型ガードを削除                                                                       | 型安全性向上に伴う変更                                            |

**Deprecations**

| 対象                          | 概要                                             | 備考・移行先                           |
| ----------------------------- | ------------------------------------------------ | -------------------------------------- |
| `ngIf` / `ngFor` / `ngSwitch` | 非推奨 → `@if` / `@for` / `@switch` への移行推奨 | 新しい制御フロー構文の使用が推奨される |
| `platform-browser-dynamic`    | 全エントリが非推奨                               | —                                      |
| `HammerJS`                    | サポート終了予定                                 | —                                      |
| `platform-server/testing`     | 非推奨 → SSR の検証は E2E テスト推奨             | —                                      |


## 参考
- [Angular v20 is here - Reddit](https://www.reddit.com/r/angular/comments/1kxlmhi/angular_v20_is_here/?utm_source=chatgpt.com)
- [What's new - by Amos Isaila - Angular 20](https://www.codigotipado.com/p/angular-20-whats-new?utm_source=chatgpt.com)
- [What's new in Angular 20.0? - Ninja Squad](https://blog.ninja-squad.com/2025/05/28/what-is-new-angular-20.0/?utm_source=chatgpt.com)
- [Stop Using These Angular Features Before It's Too Late(v20 Update](https://learnwithawais.medium.com/%EF%B8%8F-stop-using-these-angular-features-before-its-too-late-v20-update-69fa6f9b7a59?utm_source=chatgpt.com)
- [Angular v20 Has Landed: A Deep Dive into the Future of Frontend ...](https://eraoftech.medium.com/angular-v20-has-landed-a-deep-dive-into-the-future-of-frontend-development-dc0d5a1fa2f9?source=rss------programming-5&utm_source=chatgpt.com)
- [Angular 20.0.0-next.2 Release - GitClear](https://www.gitclear.com/open_repos/angular/angular/release/20.0.0-next.2?utm_source=chatgpt.com)
- [Angular 20.0.0 Release - GitClear](https://www.gitclear.com/open_repos/angular/angular/release/20.0.0?utm_source=chatgpt.com)

