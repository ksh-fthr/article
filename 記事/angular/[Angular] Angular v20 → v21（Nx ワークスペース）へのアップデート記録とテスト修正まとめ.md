## 概要（TL;DR）

Angular を v20 → v21 にアップデートし、Nx との互換調整・テスト修正（Karma/Jasmine）を行って全テストをパスさせました。
この記事は「アップデート前後の環境」「Nx 固有の注意点」「テスト修正の背景と具体例」「公式チェック項目の適用状況」をまとめたものです。

※ 本記事は AI による要約をもとに筆者が確認・編集したものです。
※ nx 環境でのマイグレーションで毎回つまづくので、備忘録として残します。

**PR:**

実際に作業を行った PR はこちらです。

- https://github.com/ksh-fthr/angular-work/pull/1017 （マージ済み）

## 目次

1. [アップデート前の環境](#before-environment)
2. [アップデート後の環境](#after-environment)
3. [Nx 環境での Angular v21 へアップデートの注意点](#nx-angular-v21-notes)
4. [テスト修正 — 背景と具体的事例](#test-fixes)
5. [v20 → v21 の主要な変更点](#changes-v20-to-v21)
6. [公式チェック項目（2点）の状況](#official-checks)
7. [再現コマンド / 推奨作業](#how-to-reproduce)

<a id="before-environment"></a>

## 1) アップデート前の環境 (主要パッケージ)

- @angular/core 等: 20.3.16
- @angular/cli: 20.3.14
- @angular/material: 20.2.14
- @angular/compiler-cli: 20.3.16
- @nx/angular: 21.6.10
- nx: 21.6.10
- typescript: ^5.8.2
- rxjs: 6.6.7
- zone.js: 0.15.1

<a id="after-environment"></a>

## 2) アップデート後の環境 (主要パッケージ)

- @angular/core 等: **21.0.5**
- @angular/cli: **21.0.5**
- @angular/material: **21.0.5**
- @angular/compiler-cli: **21.0.5**
- @nx/angular: **22.3.3**
- nx: **22.3.3**
- typescript: ^5.8.2（変更なし）
- rxjs: 6.6.7（変更なし）
- zone.js: 0.15.1（変更なし）

:::note warn

注: 現在このリポジトリでは TypeScript は **^5.8.2** のままです。
[公式のアップデート手順](https://angular.dev/update-guide?v=20.0-21.0&l=1) は TypeScript **5.9 以降** を推奨していますが、本アップデートでは **Nx や他の依存パッケージとの互換性確認を優先** し、また現時点でビルド/テストに重大な影響が出ていなかったため自動で TypeScript を上げていません。

:::

<a id="nx-angular-v21-notes"></a>

## 3) Nx 環境での Angular v21 へアップデートの注意点 🔧

- Nx / `@nx/angular` のバージョン整合が必要（Angular のメジャー上げに合わせて `nx` 側も上げました）
- 公式の自動マイグレーションで全てが解決しないケースがある（テストランタイム差や追加の小修正が必要）
- テスト関連の API 変化（`async` → `waitForAsync`、`runInInjectionContext` 等）に注意
- テスト実行時の polyfill（`global`, `process` など）の準備が必要な場合あり（[`src/polyfills.ts`](https://github.com/ksh-fthr/angular-work/blob/develop/src/polyfills.ts) を確認）

<a id="test-fixes"></a>

## 4) テスト修正 — 背景と代表的な修正例 🧪

**背景**

- v21 に伴うテスト周辺の振る舞い変化で、既存テストがランタイム例外や型エラーを吐くケースが発生

**代表的な対応例**

- `AfterContentParentComponent`：`ngAfterContentInit` / `ngAfterContentChecked` にガードを追加して TypeError を回避
- `HttpClientVerificationComponent`：テストモックを「コール可能な値（関数）」に変更してコンポーネントの期待形に合わせた
- `switch-tab` Service：内部配列／フィールドを `beforeEach` で初期化
- テストユーティリティ：`async` → `waitForAsync` に置換、必要に応じて `runInInjectionContext` を使用
- polyfills：テスト環境で `process`/`global` がなかったため簡易 shim を `src/polyfills.ts` に追加

**実施手順（実務）**

1. `npm test` → 失敗箇所を特定
2. テスト/モック/小さな実装修正
3. 再実行 → 全テストが通るまで繰り返し（最終: 56/56 passed）

<a id="changes-v20-to-v21"></a>

## 5) Angular v20 → v21 の注目変更点（抜粋）

- ホストバインディングの型チェックがデフォルトで有効に（`angularCompilerOptions.typeCheckHostBindings`）
- `provideZoneChangeDetection()` の導入（ゾーンベースアプリ用の推奨プロバイダー）
- テストユーティリティの更新（`async` → `waitForAsync`、`runInInjectionContext` の必要性など）
- signals に関する挙動改善（モックの形を合わせる必要が出るケースあり）

<a id="official-checks"></a>

## 6) 公式チェック項目（2点）とリポジトリの状況 ✅/❌

[公式の手順](https://angular.dev/update-guide?v=20.0-21.0&l=1) では次のチェックが促されていました。

> - [ ] Zone-based applications should add `provideZoneChangeDetection()` to your application's root providers. For standalone apps, add it to the `bootstrapApplication` call. For NgModule-based apps, add it to your root `AppModule`'s `providers` array. An automated migration should handle this.
> - [ ] Host binding type checking is now enabled by default and may surface new build errors. Resolve any new type errors or set `typeCheckHostBindings: false` in your `tsconfig.json`'s `angularCompilerOptions`.

といった感じで、

- `provideZoneChangeDetection` を追加すること
- ホストバインディング型チェックに注意すること

と記載されていますが、今回はいずれも対応していません。
理由は以下になります。

**1) provideZoneChangeDetection()**

- 状況: **未追加**（`src/app/app.module.ts` の `providers` に `provideZoneChangeDetection()` は入っていません）
- 理由: 自動マイグレーションで追加されず、現状は動作に明確な問題が出ていないため手動追加していません

**2) `typeCheckHostBindings`（ホストバインディングの型チェック）**

- 状況: **設定は変更していません（デフォルトのまま）**
- 理由: ビルド/テストで目立つ型エラーは発生しなかったため、無効化せず型エラーを個別に修正する方針です

**それぞれを対応する場合**

下記は `provideZoneChangeDetection` と `typeCheckHostBindings` の対応を行う場合のコード例です。

**`provideZoneChangeDetection()` の例**

```ts
import { provideZoneChangeDetection } from '@angular/core';

@NgModule({
  providers: [
    // ...既存 providers
    provideZoneChangeDetection()
  ]
})
export class AppModule {}
```

**`tsconfig.json` による一時回避（必要なら）**

```json
{
  "angularCompilerOptions": {
    "typeCheckHostBindings": false
  }
}
```

<a id="how-to-reproduce"></a>

## 8) 再現コマンド / 推奨作業 🛠

今回行った作業の流れを記しておきます。

**1) Nx に対して Angular 関連パッケージのマイグレートを作成**

```bash
npx nx migrate @angular/cli@21 @angular/core@21
```

**2) 依存関係をインストール**

```bash
npm install
```

**3) マイグレーションを実行**

```bash
npx nx migrate --run-migrations=migrations.json
```

**4) テスト実行**

```bash
npm run test

## Nx 経由の場合は...
nx test
```
