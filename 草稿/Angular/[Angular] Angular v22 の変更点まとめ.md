## はじめに

本記事では、2026年6月にリリースされた **Angular v22** の主な変更点や新機能、破壊的変更、非推奨となったAPIについて、簡潔かつ詳細にまとめています。

Angular v22は、これまでに実験的（Experimental）として導入されていた数々の強力な機能（Signal Forms、Resource APIなど）が一挙に安定版（Stable）へと昇格した、モダンAngularの「集大成」とも言える非常に重要なマイルストーンリリースです。また、AI支援開発（Agentic workflows）を見据えた先進的な機能の追加や、パフォーマンスを重視したデフォルト設定の変更が行われています。

:::note warn
⚠️ **重要：環境要件の変更** ⚠️
Angular v22では、**Node.js v20 のサポートが終了し、Node.js v22（またはv26）および TypeScript v6.0 が必須**となります。インフラやCI/CD環境を管理する際は、事前にこれらのアップデートを確認してください。

:::

⚠️ **注意** ⚠️
本記事は Angular v22 の公式リリースノートや各種解説記事をもとに、要点を整理・要約したものです。
アップデートを実際に行う際や、各APIの厳密な仕様については、必ず公式の一次情報をご確認ください。

:::

※ 内容に誤りや不備を見つけられましたら、コメントにてお知らせいただけますと幸いです 🙏

次の資料をもとに作成しています。

* **公式リリースノート（GitHub）**: [Angular v22.0.0 Changelog](https://github.com/angular/angular/blob/main/CHANGELOG.md#2200-2026-06-03) / [GitHub Releases](https://github.com/angular/angular/releases/tag/v22.0.0)
* **公式ブログ記事**: [Announcing Angular v22](https://blog.angular.dev/announcing-angular-v22-c52bb83a4664)
* **公式イベント資料**: [Angular v22 Release Event](https://angular.dev/events/v22)
* **コミュニティ記事（Angular.love）**: [Angular 22: Key Features and Changes](https://angular.love/angular-22-key-features-and-changes)

## Major Features & Breaking Changes

### 1. OnPush が新規コンポーネントのデフォルト変更検出ストラテジーに

Angularの歴史において非常に大きな方針転換が行われました。CLI等で新規にコンポーネントを作成する際、デフォルトの変更検出ストラテジーが従来の `Default`（ゾーンベースのフルチェック）から、パフォーマンスに優れた `OnPush` へと変更されました。
これにより、Signalsファーストで構築される現代のAngularアプリケーションにおいて、開発者が意識せずとも最初から高いパフォーマンスの恩恵を受けられるようになります。

**[v21 以前 (以前の挙動)]**

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-my-component',
  templateUrl: './my-component.component.html',
  styleUrls: ['./my-component.component.css']
  // changeDetection の明示的な指定がない場合、自動的に ChangeDetectionStrategy.Default だった
})
export class MyComponent {}

```

**[v22 以降]**

```typescript
import { Component, ChangeDetectionStrategy } from '@angular/core';

@Component({
  selector: 'app-my-component',
  templateUrl: './my-component.component.html',
  styleUrls: ['./my-component.component.css'],
  // 新規生成時のデフォルトが OnPush に。Signalsとの親和性が最大化される
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class MyComponent {}

```

※ 既存のアプリケーションへの影響が出ないよう、自動マイグレーション用のスキーマが提供されます。

### 2. 3つの主要APIが正式に安定版（Stable）へ昇格

これまで実験的（Experimental）ステータスだった以下の3つの重要なAPIが、Google内部での大規模な実証を経て、本番環境で安全に使用できる安定版（Stable）へと昇格しました。

#### ① Signal Forms

冗長な `FormGroup` や `FormControl` のボイラープレートを大幅に削減し、シグナルをベースにしたコンポーザブルでリアクティブな新しいフォームAPIが正式に本番対応となりました。

#### ② Asynchronous Reactivity APIs (Resource API)

非同期データ（特にHTTPリクエスト）の取得をシグナルのグラフ内に綺麗に組み込める `resource`、`rxResource`、`httpResource` が安定版になりました。
もっとも便利な `httpResource` では、シグナルの値が変わると自動的に再リクエストが走り、連続して要求が発生した際のレースコンディション（古いリクエストの自動キャンセル）もRxJSの `switchMap` のように自動でハンドリングしてくれます。

**[v22 以降の httpResource 利用例]**

```typescript
import { Component, signal } from '@angular/core';
import { httpResource } from '@angular/core';

@Component({
  selector: 'app-user-profile',
  template: `
    <input [value]="userId()" (input)="userId.set($any($event.target).value)">
    @if (userResource.value(); as user) {
      <div>ユーザー名: {{ user.name }}</div>
    }
  `
})
export class UserProfileComponent {
  userId = signal(1);

  // userId() が変わると自動でリクエストが再トリガーされ、古いリクエストはキャンセルされる
  userResource = httpResource(() => `/api/users/${this.userId()}`);
}

```

#### ③ Angular Aria

アクセシビリティを考慮したUIを構築するための12のパターン（ボタン、モーダル、ナビゲーションなど）を含む、アクセシブルUIプリミティブが安定版に到達しました。

### 3. 新しい `@Service` デコレータと `injectAsync` による遅延読み込みの導入

依存性の注入（DI）周りで、コードの記述をより簡潔かつ意図を明確にするための新しい仕組みが導入されました。

* **`@Service` デコレータ**: 従来の冗長だった `@Injectable({ providedIn: 'root' })` パターンに代わる、新しい直感的なデコレータです。
* **`injectAsync`**: サービスを必要なタイミングで非同期に遅延読み込み（Lazy-loading）し、コード分割（Code-splitting）を行うための、ファーストクラスのAPIです。
  ※ 従来の `defer()` や `inject()` と比較して、よりクリーンな構文で特定のコンテキストにおいて型安全かつ直感的な実装を可能にします。

**[v12 以降]**

```typescript
import { Service, injectAsync } from '@angular/core';

// よりシンプルで直感的なサービス宣言
@Service()
export class DataService {}

// 呼び出し側での非同期遅延読み込みの例
async function loadFeature() {
  // 必要なときにサービスを動的インポートしてインジェクト
  const lazyService = await injectAsync(DataService);
}

```

### 4. インクリメンタルハイドレーション（Incremental Hydration）がデフォルトで有効化

サーバーサイドレンダリング（SSR）におけるUXとパフォーマンスを飛躍的に向上させる「インクリメンタルハイドレーション」が、`provideClientHydration()` を設定した際にデフォルトで有効になるよう変更されました。不要な場合は明示的にオプトアウトする必要があります。

**[v22 以降]**

```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { provideClientHydration, withNoIncrementalHydration } from '@angular/platform-browser';
import { AppComponent } from './app.component';

bootstrapApplication(AppComponent, {
  providers: [
    // デフォルトで インクリメンタルハイドレーション が有効になる
    provideClientHydration()

    // もし無効化（オプトアウト）したい場合は以下のように記述：
    // provideClientHydration(withNoIncrementalHydration())
  ]
});

```

### 5. テンプレート機能の人間工学的な各種強化

HTMLテンプレートの記述がさらに柔軟に、誠に厳格になり、開発効率を高める多くの機能が追加されました。

* **HTML要素内のコメントサポート**: テンプレートのプロパティやバインディングに対してコメントを付与できるようになりました。VS Codeでのコメントトグル（`Ctrl+/` や `Cmd+/`）にも完全対応しています。
* **無効な `@for` ループのビルド時型チェック**: 従来はランタイムエラーになっていた不適切な `@for` ループの記述が、コンパイル時に型チェック（Build-time type checking）され、ビルドエラーとして検知可能になりました。
* **アロー関数とSpread構文の解禁**: テンプレート内でアロー関数（Arrow Functions）やスプレッド構文（Spread syntax）が直接利用可能になり、柔軟なデータハンドリングがインラインで完結します。
* **`@switch` とホストディレクティブの改善**: `@switch` のボイラープレートがさらに削減され、ホストディレクティブの重複排除（Host Directive de-duplification）も行われています。

**[v22 以降のテンプレート表現例]**

```html
<div [id]="elementId">
  <p>{{ userName }}</p>
</div>

<div>
  <app-child [transform]="(val) => val * 2" [config]="{ ...baseConfig, theme: 'dark' }"></app-child>
</div>
```

### 6. ルーティングにおけるデフォルト挙動の変更（破壊的変更）

ルーター周りで、開発者のコード量を減らすためのデフォルト設定の変更（一部破壊的変更）が2点行われました。

#### ① `paramsInheritanceStrategy` のデフォルトが `'always'` に変更

これまで子ルートから親ルートのパラメータを参照する際、`route.parent?.parent?.snapshot.params` のような冗長な記述を強いられることが多々ありましたが、デフォルトで親ルートのパラメータをすべて自動で引き継ぐ（`always`）ようになりました。

⚠️ **重要：注意点**
この変更には**自動マイグレーション（Schematics）が提供されません**。ネストされたルートを持つアプリケーションでは、意図しない挙動を防ぐために手動での確認と修正が必要です。以前の挙動（子ルートが空の場合のみ引き継ぐ）を維持したい場合は、明示的に `'emptyOnly'` を設定する必要があります。

#### ② `canMatch` ガード関数の引数変更

`canMatch` 関数に、第3引数として `currentSnapshot` が必須（Mandatory）となりました。これに伴う自動マイグレーション（Schematics）が提供されます。

**[v21 以降]**

```typescript
import { RouterStateSnapshot } from '@angular/router';

const routes: Routes = [
  {
    path: 'admin',
    component: AdminComponent,
    canMatch: [
      // 第3引数の currentSnapshot が必須となった
      (route: Route, segments: UrlSegment[], currentSnapshot: RouterStateSnapshot) => inject(AuthService).isAdmin()
    ]
  }
];

```

### 7. AI・Agent対応の強化（WebMCPツールの実験的導入）

Angular v21から導入された、AIがソースコードを理解・編集しやすくするための「Angular MCP (Model Context Protocol) Server」がさらに成熟し、新たに `angular-new-app`（新規プロジェクト作成ガイドライン）や `angular-developer`（アーキテクチャ案内・コード生成）といった「Agent Skills」が追加されました。
さらに、ブラウザ上のAIエージェントがアプリケーションのフォームやサービスを型安全なツール（Typed tools）として直接呼び出せるようにする、実験的な機能 **WebMCP**（`provideExperimentalWebMcpTools`）が追加されています。

---

## まとめ

### 主な Major Features & Breaking Changes 一覧

#### core
* CLI等での新規コンポーネント生成時、デフォルトの変更検出ストラテジーが `ChangeDetectionStrategy.OnPush` に変更。
* `Signal Forms`、`Asynchronous Reactivity APIs (Resource API)`、`Angular Aria` が実験的ステータスを脱し、正式に本番環境対応の安定版（Stable）へ昇格。
* 新しい `@Service` デコレータが導入され、`@Injectable({ providedIn: 'root' })` の代替としてより簡潔に記述可能に。
* DIにおいて、サービスを必要な時に動的・非同期にコード分割読み込みするための `injectAsync` APIが追加（`defer`等よりもクリーンな構文を提供）。

#### compiler / template
* HTMLテンプレート内の要素、バインディング、プロパティに対するコメント記法をサポート（VS Codeのコメントトグル対応）。
* テンプレート内で直接アロー関数（Arrow Functions）およびスプレッド構文（Spread syntax）の利用が可能に。
* 無効な記述の `@for` ループをビルド時に検知するコンパイル時型チェックを導入（ランタイムエラーの防止）。
* ホストディレクティブの重複排除（Host Directive de-duplification）によるテンプレートパイプラインの効率化。
* TypeScriptの `isolatedDeclarations` 下での NgModule コンパイルに対応し、より高速で並列なビルドが可能に。

#### router
* **`paramsInheritanceStrategy`**: デフォルトが `'emptyOnly'` から `'always'` へ変更。**※注意：自動マイグレーションなし。**
* **`canMatch` ガード関数**: 第3引数 `currentSnapshot` の受け取りが必須化（自動マイグレーションあり）。

#### common / platform-browser
* HTTPクライアント（HttpClient）の通信基盤として、従来のXMLHttpRequestに代わり **Fetch API がデフォルトで使用される**ように変更。
* `provideClientHydration()` 使用時、**インクリメンタルハイドレーション（Incremental Hydration）が自動的にデフォルトで有効化**。

#### 環境要件・AI
* **【重要】環境要件の更新**: **Node.js v20 のサポート終了**に伴い、**Node.js v22 (またはv26) および TypeScript v6.0 が必須**となります。
* AIコーディングアシスタント用のエージェントスキル（`angular-new-app`, `angular-developer`）を実装。
* ブラウザ駆動AI向けの実験的API `WebMCP`（`provideExperimentalWebMcpTools` 等）の導入。

### Deprecations (非推奨) / Removals (削除)

#### core
* 従来の `@Injectable({ providedIn: 'root' })` は引き続き動作しますが、今後は新しい `@Service` への移行が推奨されます。

#### compiler / platform-browser
* 過去の古いブラウザ向けのエミュレートされた挙動である、レガシーな「シャドーCSSカプセル化ポリフィル（Shadow CSS encapsulation polyfills）」および「レガシーなシャドーDOMセレクタサポート」が完全に削除されました。これに伴い、`:host` や `:host-context` の内部ハンドリングが大幅にクリーンかつシンプル化されています。古いスタイリングに依存していたアプリケーションは念のため注意してください。

#### build / CLI
* Webpackベースのビルドサポートが非推奨（Deprecated）となりました。Angularチームは今後、超高速な「TSGoベースのapplication builder」への移行を完全に推奨しています。

### 破壊的変更・非推奨の一覧表

| 変更対象 | 区分 | 概要 | 備考・対応方法／注意点 |
| --- | --- | --- | --- |
| **Change Detection デフォルト** | 変更 | 新規作成されるコンポーネントのデフォルト変更検出が `OnPush` に。 | 既存のアプリには影響しないようマイグレーションが走りますが、新規開発時は意識する必要があります。 |
| **paramsInheritanceStrategy** | 破壊的変更 | ルーターのパラメータ引き継ぎ戦略のデフォルトが `'always'` に。 | **【注意】自動マイグレーションなし。** 手動での確認と修正が必要です。 |
| **canMatch 引数** | 破壊的変更 | `canMatch` ガード関数に第3引数 `currentSnapshot` が必須化。 | 自動マイグレーション（Schematics）が提供されるため、アップデートコマンドで自動置換可能です。 |
| **HTTP Client** | 変更 | 通信裏の仕組みがデフォルトで `Fetch API` に。 | 近年のモダンブラウザ標準に合わせた最適化です。 |
| **Incremental Hydration** | 変更 | SSR利用時、インクリメンタルハイドレーションが標準でONに。 | オプトアウトしたい場合は、`provideClientHydration(withNoIncrementalHydration())` を使用します。 |
| **環境要件 (TS / Node)** | 破壊的変更 | TypeScript v6.0 未満、および Node.js v20 のサポート終了。 | **重要：** アップデート前に Node.js v22（またはv26）環境、1つ以上への更新が必要です。 |
| **Webpack サポート** | 非推奨 | Webpackを用いた従来のビルド機構が非推奨に。 | TSGoベースの新しい `application builder` への完全移行・ refactor が推奨されます。 |
| **Shadow CSS ポリフィル** | 削除 | 古いシャドーCSSカプセル化ポリフィル、レガシーセレクタの削除。 | `:host` などのハンドリングがシンプル化。極端に古い emulated-encapsulation スタイルを使っていないか確認してください。 |

## 参考

* [Angular v22.0.0 Changelog (GitHub)](https://github.com/angular/angular/blob/main/CHANGELOG.md#2200-2026-06-03)
* [Announcing Angular v22 (Official Blog)](https://blog.angular.dev/announcing-angular-v22-c52bb83a4664)
* [Angular 22: Key Features and Changes (Angular.love)](https://angular.love/angular-22-key-features-and-changes)
* [What's new in Angular 22.0? (The Ninja Squad)](https://blog.ninja-squad.com/2026/06/03/what-is-new-angular-22.0)
