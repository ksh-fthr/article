# 目次

- [目次](#目次)
- [はじめに](#はじめに)
- [Breaking Changes](#breaking-changes)
  - [1. Zoneless のデフォルト化と変更検知の刷新](#1-zoneless-のデフォルト化と変更検知の刷新)
  - [2. Experimental Signal Forms の導入](#2-experimental-signal-forms-の導入)
  - [3. 新UIライブラリ「Angular Aria」の登場 ( Developer Preview )](#3-新uiライブラリangular-ariaの登場--developer-preview-)
  - [4. 安定版テストランナーとしての Vitest のデフォルト化](#4-安定版テストランナーとしての-vitest-のデフォルト化)
  - [5. SimpleChanges のジェネリック化による型安全性の向上](#5-simplechanges-のジェネリック化による型安全性の向上)
  - [6. HttpClient のデフォルトプロバイド化](#6-httpclient-のデフォルトプロバイド化)
  - [7. ngClass / ngStyle からの移行を支援する新シマティクス](#7-ngclass--ngstyle-からの移行を支援する新シマティクス)
  - [8. KeyValue パイプのオプショナルキー対応](#8-keyvalue-パイプのオプショナルキー対応)
  - [9. HttpResponse / HttpErrorResponse への `responseType` 追加](#9-httpresponse--httperrorresponse-への-responsetype-追加)
  - [10. テンプレートでの正規表現サポートと @defer の拡張](#10-テンプレートでの正規表現サポートと-defer-の拡張)
  - [11. Angular MCP サーバーの安定版 ( Stable ) 昇格](#11-angular-mcp-サーバーの安定版--stable--昇格)
- [まとめ](#まとめ)
  - [主な Breaking Changes 一覧](#主な-breaking-changes-一覧)
  - [Deprecations (非推奨)](#deprecations-非推奨)
  - [破壊的変更・非推奨の一覧表](#破壊的変更非推奨の一覧表)
- [参考](#参考)

# はじめに

本記事では、Angular v21 のリリースに伴う主な変更点を簡潔にまとめてみました。
( Angular v22 がリリースされる直前で今更感がありますが、自分への備忘録として )

:::note warn

⚠️ **注意** ⚠️
本記事は Angular v21 の公式リリースノートや、各種解説記事を参考に筆者が独自に整理・要約したものであり、誤りが含まれる可能性もあります。
公式情報については [公式 GitHub](https://github.com/angular/angular/releases/tag/21.0.0) や [公式ドキュメント](https://blog.angular.dev/announcing-angular-v21-57946c34f14b) を必ずご確認ください。

:::

※ 内容に不備がありましたらコメントにてお知らせください 🙏

次の資料をもとに作成してます。

- **公式リリースノート:**
  - [Angular 21.0.0](https://github.com/angular/angular/releases/tag/21.0.0)
- **公式ブログ記事:**
  - [Announcing Angular v21](https://blog.angular.dev/announcing-angular-v21-57946c34f14b)
- **コミュニティ記事:**
  - [Angular 21 – What's New](https://angular.love/angular-21-whats-new)

---

# Breaking Changes

## 1. Zoneless のデフォルト化と変更検知の刷新

Angular v18 で実験的に導入され、更新されてきた **Zoneless ( `zone.js` に依存しない変更検知 )** が、v21 より新規アプリケーションにおけるデフォルトの挙動となりました。
これにより、Core Web Vitals の向上、ネイティブな `async/await` の完全サポート、バンドルサイズの削減、そしてシンプルなデバッグ環境が標準で提供されます。

これに伴い、従来の `zone.js` ベースのアプリケーション運用方法に一部破壊的変更が加わっています。
デフォルトでは変更検知スケジューラが自動提供されなくなるため、既存の ZoneJS ベースのプロジェクトでは `provideZoneChangeDetection` を明示的に指定する必要があります。
なお、この変更は自動マイグレーションスクリプトによってカバーされます。また、`ignoreChangesOutsideZone` オプションは完全に削除されました。

**[v20 以前 ( ZoneJSベース )]**

```typescript
// 自動的にZoneJSベースのスケジューラが有効になっていた
bootstrapApplication(AppComponent);
```

**[v21 以降 ( ZoneJSを引き続き使う場合 )]**

```typescript
import { provideZoneChangeDetection } from "@angular/core";

bootstrapApplication(AppComponent, {
  providers: [provideZoneChangeDetection({ eventCoalescing: true })],
});
```

## 2. Experimental Signal Forms の導入

Signals を基盤とした新しいフォーム管理ライブラリ **Signal Forms** が実験的機能 ( Experimental ) として導入されました。
フォームのモデル状態が Signal となり、テンプレートに配置した新しい `[formField]` ディレクティブと双方向で自動同期します。
型安全性が100%保証されるほか、従来の複雑な `ControlValueAccessor` を実装する必要がなくなり、カスタムコンポーネントとのバインディングが劇的に容易になります。
また、スキーマベースの集中バリデーションロジックが組み込まれています。

なお、従来の Reactive Forms とも100%完全な互換性があり、同じアプリケーション内で画面ごとに段階的に移行することが可能です。

**[コンポーネント実装例]**

```typescript
import { Component, signal } from "@angular/core";
import { form, FormField } from "@angular/forms/signals";

@Component({
  selector: "app-login-form",
  standalone: true,
  imports: [FormField],
  template: `
    <form (ngSubmit)="onSubmit()">
      Email: <input [formField]="loginForm.email" type="text" /> Password:
      <input [formField]="loginForm.password" type="password" />
      <button type="submit" [disabled]="loginForm().invalid()">Submit</button>
    </form>
  `,
})
export class LoginForm {
  login = signal({
    email: "",
    password: "",
  });

  loginForm = form(this.login);

  onSubmit() {
    console.log("Form Submitted:", this.login());
  }
}
```

スキーマ関数を用いた柔軟なエラーハンドリングやカスタムバリデーションも以下のようにすっきりと定義できます。

```typescript
import { required, minLength } from '@angular/forms/signals';

protected readonly personForm = form(this.person, (path) => {
  required(path.name);
  minLength(path.name, 3);
});

```

## 3. 新UIライブラリ「Angular Aria」の登場 ( Developer Preview )

アクセシビリティ ( A11y ) を最優先事項として設計されたモダンなUIコンポーネントライブラリ **[Angular Aria](https://angular.jp/guide/aria/overview)** が開発者プレビュー ( Developer Preview ) としてリリースされました。

第一弾として、Accordion, Combobox, Grid, Listbox, Menu, Tabs, Toolbar, Tree の 8 つのUIパターンにまたがる 13 のコンポーネントが提供されます。
これらは完全にアンスタイルド ( Headless ) な状態で提供されるため、アクセシビリティの厳しい国際基準を満たした強固なマークダウン構造の上に、Tailwind CSS や独自のスタイルを開発者が完全に自由に上乗せすることができます。

**[インストールコマンド]**

```bash
npm install @angular/aria
```

## 4. 安定版テストランナーとしての Vitest のデフォルト化

2023 年の Karma 非推奨化以降、次世代のテストソリューションの検証が進められてきましたが、v21 にて **Vitest サポートが正式に安定版 ( Stable )** となり、新規プロジェクトのデフォルトテストランナーに指定されました。
新しくアプリケーションを作成した場合は、おなじみの `ng test` コマンドを叩くだけで超高速な Vitest によるテストが起動します。

既存プロジェクトを Jasmine から Vitest へ自動リファクタリングするためのマイグレーションシマティクスも提供されています。

**[マイグレーションコマンド]**

```bash
ng g @schematics/angular:refactor-jasmine-vitest
```

※ Vitest の安定版昇格に伴い、これまで実験的に提供されていた Web Test Runner および Jest の Angular CLI による組み込みサポートは **非推奨** となり、v22 で削除される予定です ( コミュニティ製の `jest-preset-angular` 等は引き続き利用可能です ) 。
※ 複雑な Jasmine 固有の Spy やアサーションについては、一部手動でのリファクタリングが必要になる場合があります。
※ Angular v21 の Vitest によるユニットテストは **デフォルトで Zoneless環境 で実行** されます。そのため、従来の `ZoneJS` に依存していた `fakeAsync` や `tick` などのテストユーティリティが使えなくなります。既存プロジェクトから移行する際は **非同期処理を同期的なモックに置き換えるリファクタリング** が必要になります。

## 5. SimpleChanges のジェネリック化による型安全性の向上

`ngOnChanges` ライフサイクルホックで変更検知を受け取る `SimpleChanges` インターフェースがジェネリック型に対応しました。
これにより、従来 `any` 型としてダウングレードされていた `previousValue` や `currentValue` が、コンポーネントが持つ本来のプロパティ型として厳密に型チェックされるようになります。

**[v21 以降の型セーフな実装]**

```typescript
import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';

@Component({
  selector: 'app-profile',
  standalone: true,
  template: `...`
})
export class ProfileComponent implements OnChanges {
  @Input({ required: true }) userName!: string;
  @Input({ required: true }) age!: number;

  // コンポーネントのクラス名 (または this) を渡すことで型を安全にする
  ngOnChanges(changes: SimpleChanges<ProfileComponent>) {
    if (changes.age) {
      const newAge = changes.age.currentValue;  // 自動的に number 型として推論される
      const oldAge = changes.age.previousValue; // 自動的に number 型として推論される
      console.log(`年齢が ${newAge - oldAge} 歳上がりました`);
    }
  }
}
```

## 6. HttpClient のデフォルトプロバイド化

Angular v21 より、特別なインターセプターの設定などを行わないプレーンな利用において、`appConfig` 内で明示的に `provideHttpClient()` を呼び出してプロバイダを追加する必要がなくなりました。
フレームワーク内部でデフォルトで自動的に提供されるため、構成コードがより洗練され、シンプルになります。

**[v20 以前 ( 必須 )]**

```typescript
export const appConfig: AppConfig = {
  providers: [
    provideHttpClient(), // 必須だった
  ],
};
```

**[v21 以降]**

```typescript
export const appConfig: AppConfig = {
  providers: [
    // 特別な構成が必要なければ provideHttpClient() の記述を省略可能！
  ],
};
```

※ `provideHttpClient()` の記述を省略できるようになりましたが、それはプレーンに使う場合のみです。
※ [lacolacoさんの記事](https://zenn.dev/lacolaco/articles/angular-provide-http-client-by-default#providehttpclient%E3%81%AF%E5%BC%95%E3%81%8D%E7%B6%9A%E3%81%8D%E5%BF%85%E8%A6%81) にある通り、「カスタムインターセプターを通したい場合 ( `withInterceptors([...])` などを使う時 ) 」は、従来通り `appConfig` での明示的なプロバイドが必要になります。

## 7. ngClass / ngStyle からの移行を支援する新シマティクス

パフォーマンスの最適化およびコードの簡潔化の観点から、従来の `ngClass` や `ngStyle` ディレクティブの使用を避け、ネイティブのオブジェクト指定による `[class]` や `[style]` バインディングの利用が強力に推奨されるようになりました。

これらを既存コードから自動検出して一括変換し、不要になった `NgClass` / `NgStyle` のインポートを自動で削除する強力なマイグレーションコマンドが追加されています。

**[ngClass の一括移行シマティクス]**

```bash
ng generate @angular/core:ngclass-to-class

```

**[ngStyle の一括移行シマティクス]**

```bash
ng generate @angular/core:ngstyle-to-style

```

## 8. KeyValue パイプのオプショナルキー対応

テンプレート内でオブジェクトをループ処理する際に重宝する `keyvalue` パイプが、オプショナルなキー ( `?` がついたプロパティ ) を持つデータ構造を公式にサポートしました。
これまで TypeScript の厳格な型チェックによってコンパイルエラーを引き起こしていたケースでも、型定義を崩すことなく安全にループを回すことが可能になりました。

```typescript
export interface OptionalUser {
  name: string;
  surname?: string; // オプショナル
  age?: number; // オプショナル
}
```

```html
@for (prop of user | keyvalue; track $index) {
<p>Key: {{ prop.key }}, Value: {{ prop.value }}</p>
}
```

## 9. HttpResponse / HttpErrorResponse への `responseType` 追加

`HttpClient` から返される `HttpResponse` および `HttpErrorResponse` クラスに、Fetch API に準拠した生のレスポンス種別を示す `responseType` プロパティ ( `basic`, `cors`, `opaque`, `opaqueredirect` など ) が新設されました。
これにより、HttpClient の従来の挙動を一切破壊することなく、CORS ( Cross-Origin Resource Sharing ) 起因による接続エラーなどの原因究明・デバッグが格段に容易になります。

```typescript
this.http.get("/api/data", { observe: "response" }).pipe(
  tap((response) => {
    console.log("Response type:", response.responseType);
    if (response.responseType === "opaque") {
      console.warn(
        "CORSの不具合が検出されました ( 不透明なレスポンスです ) 。",
      );
    }
  }),
);
```

## 10. テンプレートでの正規表現サポートと @defer の拡張

テンプレート内の開発体験を向上させる細かな機能追加も多く実施されています。

- **テンプレート内での正規表現 ( RegExp ) の直接サポート**
  - 新機能の `@let` 構文の強化と合わさり、テンプレート内でインラインで正規表現のテストメソッド ( `.test()` ) などを記述できるようになりました。
  - 従来はコンポーネント側にわざわざ readonly regex = /.../ と定義して公開する必要があったため、テンプレートで完結できるのは大きなメリットです。

```html
@let isValidNumber = /\d+/.test(someValue); @if (!isValidNumber) {
<p>{{someValue}} は有効な数値ではありません！</p>
}
```

- **@defer の `on viewport` オプションの拡張**
  遅延読み込みを制御する `@defer` 構文の `on viewport` トリガーにおいて、`IntersectionObserver` の `rootMargin` ( 例: `'100px'` ) などの高度なオプションをカスタム指定できるようになりました。

## 11. Angular MCP サーバーの安定版 ( Stable ) 昇格

AIエージェントやLLMと連携し、コンテキスト情報を橋渡しする「Model Context Protocol」サーバーが正式に安定版となり、AIを活用した OnPush / Zoneless 移行のコード生成が大幅に強化されました。
これにより、Cursor や VS Code などのAIツールが、プロジェクト全体の依存関係を正確に把握した上で、最適なリファクタリングを提案してくれるようになります。

---

# まとめ

## 主な Breaking Changes 一覧

- **TypeScript 5.9 未満のサポート終了**：TypeScript 5.9 への移行が必須となります。
- **ZoneJSの自動変更検知スケジューラの提供終了**：ZoneJSベースのアプリケーションでは明示的に `provideZoneChangeDetection` を追加する必要があります ( 自動移行あり ) 。
- **Component メタデータの削減**：`moduleId` 属性、および独自構文を指定できた `interpolation` オプションが削除され、標準の `{{ ... }}` のみとなりました。
- **Custom Elements 内における Signal Inputs 参照方法の変更**：カスタムエレメント ( `@angular/elements` ) において、Signal Input の値を読み込む際の関数実行の括弧 ( `()` ) が不要となり、プロパティのように直接アクセスする形式に統一されました ( 例：`elementRef.inputName` ) 。
- **Router の `lastSuccessfulNavigation` の Signal 化**：呼び出しの際に関数として実行 ( `lastSuccessfulNavigation()` ) する必要があります。

## Deprecations (非推奨)

- **Web Test Runner および Jest の実験的サポート非推奨**：デフォルトのテストランナーが Vitest (Stable) に一本化されたため、Angular CLI によるこれら2つの実験的構成サポートは非推奨となりました。v22 で削除される予定です。

## 破壊的変更・非推奨の一覧表

| 変更対象                      | 区分            | 概要                                               | 備考・対応方法／注意点                                                                       |
| ----------------------------- | --------------- | -------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| **TypeScript**                | Breaking Change | TypeScript 5.9 未満のサポート終了                  | v5.9 以上へのアップデートが必要です。                                                        |
| **ZoneJS / 変更検知**         | Breaking Change | デフォルトの変更検知スケジューラ無効化             | `provideZoneChangeDetection` を明示的に `providers` に追加 ( 自動マイグレーションあり ) 。   |
| **ZoneJS / 変更検知**         | Breaking Change | `ignoreChangesOutsideZone` オプションの削除        | 該当オプションは利用できなくなりました。                                                     |
| **Component**                 | Breaking Change | `moduleId` および `interpolation` オプションの削除 | 独自のインターポレーション構文は廃止され、`{{ }}` のみに限定されます。                       |
| **@angular/elements**         | Breaking Change | Custom Elements 内の Signal Inputs 参照方法変更    | ゲッター呼び出しの括弧 `()` が不要になり、`elementRef.inputName` で値を取得可能に。          |
| **@angular/router**           | Breaking Change | `lastSuccessfulNavigation` の Signal 化            | 値を取得する際に関数実行の括弧 `()` が必要になります。                                       |
| **@angular/router**           | Breaking Change | ナビゲーション完了のマイクロタスク追加             | テストなどでナビゲーション完了のタイミングを待つアサーションの修正が必要なケースがあります。 |
| **@angular/platform-browser** | Breaking Change | `ApplicationConfig` のエクスポート削除             | 今後は `@angular/core` からのインポートに統一する必要があります。                            |
| **テストランナー**            | Deprecation     | Web Test Runner と Jest の実験的サポート非推奨     | v22 で削除予定。新規プロジェクトはデフォルトの Vitest へのリファクタリングを推奨。           |

---

# 参考

- **公式リリースノート:** [Angular v21.0.0 Release Notes](https://github.com/angular/angular/releases/tag/21.0.0)
- **公式ブログ記事:** [Announcing Angular v21](https://blog.angular.dev/announcing-angular-v21-57946c34f14b)
- **公式ドキュメント ( Aria ) :** [Angular Aria Overview](https://angular.dev/guide/aria/overview)
- **公式ドキュメント ( バージョン互換性 ) :** [Angular Actively Supported Versions](https://angular.jp/reference/versions)
- **コミュニティ解説記事:** [Angular 21 – What's New](https://angular.love/angular-21-whats-new)
- **コミュニティ解説記事 ( HttpClient ) :** [Angular: provideHttpClient はもう不要 ( でも必要 )](https://zenn.dev/lacolaco/articles/angular-provide-http-client-by-default)
- **コミュニティ解説記事 ( MCP ) :** [Angular v21 リリース、何か変わったのか](https://zenn.dev/suhyoenbae/articles/c6d8b06fba8ab3)
