# [Angular] Angular v17 の変更点と新機能まとめ

## はじめに

⚠️ **この記事は Angular v17.0 〜 v17.3 のリリース時に ng-japan OnAir で扱われた内容を、自分用の視聴メモを整理したものです。**
Built-in control flow (`@if`/`@for`) の登場や、Signal 系 API (`input()` / `model()` / `output()`) の本格導入など、v17 は Angular の書き方が変わる大きな転換点でした。

プロダクトに入れたい欲は強いですが、既存コードベースの設計思想と混在させると認知負荷が上がるので、入れ方は要設計です。趣味プロダクトで先に慣らしておくのがオススメ。

## 参考資料

* **Angular Blog**: Introducing Angular v17
* **ng-japan OnAir #72**: Angular v17リリース！ここだけは抑えておきたいポイント解説
* **ng-japan OnAir #76**: Monthly Angular 2月号
* **ng-japan OnAir #77**: Monthly Angular 3月号
* **Built-in control flow**
* **Deferrable Views**
* **Angular: Model Inputsで何が変わるのか**
* **Angular's Output Symphony: Introducing the Output Function**

## 目次

* [v17.0] 新しいドキュメントサイト angular.dev
* [v17.0] Built-in control flow (@if / @for / @switch)
* [v17.0] Deferrable Views (@defer)
* [v17.0] Signal が stable に
* [v17.2] NgOptimizedImage の placeholder
* [v17.1 / v17.2] Signal Inputs / Model Inputs / Signal Queries
* [v17.2] CLI まわり (bun / Vite / Define / postcss)
* [v17.3] Output Function (output())
* [v17.3] その他の改善 (HostAttributeToken / Router Guard型 / 双方向バインディング migration)
* [v17.3] Angular Material 3 への切り替え
* NG Conf 2024 で語られた将来トピック
* 個人的な感想
* まとめ

## [v17.0] 新しいドキュメントサイト angular.dev

公式ドキュメントが `angular.io` から `angular.dev` に移行しました。ブラウザ上でそのまま Tutorial を進められるようになっています。

⚠️ **Tutorial では @if などの新構文が積極的に使われており、`*ngIf` はほぼ登場しません。これから Angular を学ぶ人は `*ngIf` を知らない、ということになりそうです。**

## [v17.0] Built-in control flow (@if / @for / @switch)

これまでの `*ngIf` や `*ngFor` などの構造ディレクティブを置き換える、新しい組み込み構文（Control Flow）が導入されました。
新規導入の変更検知や Zoneless 化に向けた地ならしが目的とのこと。

### [16 以前]

```html
<div *ngIf="user; else loading">{{ user.name }}</div>
<ng-template #loading><p>Loading...</p></ng-template>

<div *ngFor="let item of items; trackBy: trackById">
  {{ item.name }}
</div>
```

### [17 以降]

```html
@if (user) {
  <div>{{ user.name }}</div>
} @else if (loading) {
  <p>Loading...</p>
} @else {
  <p>No user</p>
}

@for (item of items; track item.id) {
  <div>{{ item.name }}</div>
} @empty {
  <p>No items found!</p>
}
```

### ポイント

* **`@for` では `track` 式が必須になり、`@empty` ブロックも組み込みで使えるようになりました。**
* `@if` の `@else if` がサポートされ、`ng-template` のたらい回しが不要に。
* `@for` は内部最適化により、**`*ngFor` 比でパフォーマンス約 90% 向上**します。
* ※ ビルトイン化により、`*ngIf` / `*ngFor` が依存していた `CommonModule` の `import` を外せるケースがあります。`NgComponentOutlet` や `AsyncPipe` を使っていない箇所であれば、バンドルサイズの削減も期待できます。

### マイグレーションコマンド

公式の自動マイグレーションが用意されているので、ほぼ機械的に置き換え可能です。

```bash
ng generate @angular/core:control-flow
```

```text
? Which path in your project should be migrated? ./
? Should the migration reformat your templates? Yes

UPDATE src/app/.../validation-verification.component.html (3824 bytes)
UPDATE src/app/.../reactive-form-verification.component.html (2637 bytes)
...
```

実体験として、上記コマンドでほぼ機械的に置換できました。テンプレート再フォーマットも併せて行ってくれるので、既存コードベースでも十分実用的です。

## [v17.0] Deferrable Views (@defer)

v17の目玉機能のひとつ。コンポーネントの一部を遅延読み込み（Lazy Load）できる新構文です。

```html
@defer (on viewport) {
  <heavy-chart [data]="data" />
} @placeholder (minimum 500ms) {
  <p>準備中...</p>
} @loading (after 100ms; minimum 1s) {
  <p>読み込み中...</p>
} @error {
  <p>読み込みに失敗しました</p>
}
```

* **`@placeholder`**: 読み込み開始前の初期表示
* **`@loading`**: ロード中に表示する内容
* **`@error`**: エラー時のフォールバック
* **トリガ**: `on viewport` / `on idle` / `on hover` / `on interaction` など。
* ※ `@placeholder` には `minimum 500ms` のような最小表示時間も指定できるので、ロード完了が早すぎるときの画面ちらつきを抑えることができます。

## [v17.0] Signal が stable に

Signal API が v17 で stable（安定版）になりました。一方で、v16 までに使えていた一部の古い API は廃止されているので、Signal の変更状況には注意が必要です。

そして「`zone.js` をなくす方向の動きが v18 に向けて進んでいる」というアナウンスも v17 リリース時にされていました（実際、v18 で experimental zoneless が登場しています）。これは Signal ベースの変更検知への移行を前提としています。

## [v17.2] NgOptimizedImage の placeholder

画像最適化ディレクティブ `ngSrc` に `placeholder` 属性が追加されました。遅延読み込みなどをいい感じにしてくれ、`width` / `height` で粗く指定しておいても、正規の画像が読み込まれれば自動で置き換えてくれます。

```html
<img ngSrc="hero.jpg" width="800" height="600" placeholder />
```

## [v17.1 / v17.2] Signal Inputs / Model Inputs / Signal Queries

### Signal Inputs (v17.1)

`@Input()` の代替として、Signal ベースの `input()` 関数が追加されました。

#### [v17.0 以前]
```typescript
@Input() hoge!: string;

ngOnChanges(changes: SimpleChanges): void {
  // 変更検知
}
```

#### [v17.1 以降]
```typescript
// 引数に初期値を渡せる。input.required() で必須指定も可能
hoge = input<string>();

constructor() {
  effect(() => {
    console.log(this.hoge()); // テンプレートでも `{{ hoge() }}` のように関数呼び出しで参照
  });
}
```

* **⚠️ ※ Signal 型（InputSignal）として初期化されるため、`strictPropertyInitialization` を常に有効にできる恩恵があります。**
* **※ ただし、`SimpleChanges` の `previousValue` / `firstChange` が使えなくなるため、古い値が必要な場合は RxJS 経由で取る必要があります。**

### Model Inputs (v17.2)

`Model Inputs` は `Signal Inputs` を拡張したもので、名前の通り `ngModel` 系の双方向バインディングを Signal で扱えるようにします。

```typescript
@Component({
  selector: 'app-counter',
  template: `<input [(value)]="count" />`,
})
export class CounterComponent {
  count = model(0); // 双方向バインディングできる Signal
}
```

* **Signal Inputs が「読み取り専用の入力」であるのに対し、Model Inputs は「双方向の同期」をサポートする設計になっています。**
* **Reactive Form の代替、というか置き換えが推進される程に便利らしい。**

> 参考: Angular: Model Inputs で何が変わるのか

### Signal Queries (v17.2)

`@ViewChild` / `@ViewChildren` / `@ContentChild` / `@ContentChildren` を Signal で扱えるようになりました（`viewChild` / `viewChildren` / `contentChild` / `contentChildren`）。

```typescript
import { viewChild } from '@angular/core';

@Component({ /* ... */ })
export class MyComponent {
  // 戻り値が Signal なので、初期化タイミングを effect() 内で安全に扱える
  child = viewChild<ChildComponent>('child');

  constructor() {
    effect(() => {
      const c = this.child();
      if (c) {
        // 参照確定後の処理
      }
    });
  }
}
```

* ※ うまく使うと `afterViewChecked` 等のライフサイクルが不要になるほど強力です。

## [v17.2] CLI まわり (bun / Vite / Define / postcss)

開発体験を底上げするアップデートが入りました。

* **bun 対応**: package manager として `bun` を選択可能に。
* **build オプションに `define` 追加**: webpack の `DefinePlugin` 相当。v17 から `ng new` で `environment.ts` が作られなくなったので、その代替。
* **`ng serve` の進化**: Vite ベースの dev-server オプションも持ちます。
* **postcss config**: `postcss.config.json` / `.postcssrc.json` をプロジェクトに置けます。

## [v17.3] Output Function (output())

`@Output()` も関数ベースに移行しました。今後 `@Output` デコレータは消えていく方向です。

#### [v17.2 以前]
```typescript
@Output() page = new EventEmitter<number>();
```

#### [v17.3 以降]
```typescript
import { output } from '@angular/core';

@Component({
  selector: 'app-foo',
  template: ``,
  standalone: true,
})
export class FooComponent {
  page = output<number>();
  // Or use an alias
  // page = output<number>({ alias: 'currentPage' });
}
```

```html
<app-foo (page)="doSomething($event)" />
```

### ポイント

* `emit` は残っている。`subscribe` と `emit` のみ公開。
* **`next` はなくなった。**（内部的に EventEmitter を使わなくなったため）
* `@Output` を Observable のインスタンスとして使っていた場合は、`outputFromObservable()` を挟めて互換性を取る。
* 逆に、新しい `output()` を Observable として扱いたい場合は `outputToObservable()` を使う。

## [v17.3] その他の改善

### HostAttributeToken で host node の attribute を inject

```typescript
import { HostAttributeToken, inject } from '@angular/core';

@Directive({ /* ... */ })
class MyDir {
  someAttr = inject(new HostAttributeToken('some-attr'));
}
```

### 無効な双方向バインディングのマイグレーション

`[(ngModel)]` に複雑な式を書いている場合、それを自動で `getter`/`setter` 分解してくれるマイグレーションが追加されました。

#### // Before
```html
<input [(ngModel)]="a && b" />
```

#### // After (ng update で自動変換される)
```html
<input [ngModel]="a && b" (ngModelChange)="a && (b = $event)" />
```

⚠️ **※ 変換後のコードがそんなに読みやすくないのはご愛嬌。最新バージョンだと Before のような書き方自体が compile error になります。**

### Router Guard 型

抽象的な型 `MaybeAsync<GuardResult>` が用意され、Guard の型定義がしやすくなりました。

## [v17.3] Angular Material 3 への切り替え

`material.angular.io` が Material 3 ベースに切り替わりました（v17.2 で実験的サポート、v17.3.1 で正式サポート）。

⚠️ **デザイン・カラーリングの変化が大きいので段階対応推奨。**

## NG Conf 2024 で語られた将来トピック

v17.3 のリリース時期に開催された NG Conf 2024 で、Angular の今後の方向性についていくつかトピックが共有されていました。v17自体の機能ではないですが、v18以降への伏線として参考になります。

* **Merging Angular and Wiz**: Wiz は Google の社内フレームワーク（OSSではないが Google 検索や Gmail などの実装に使われている）。Angular の Signals を Wiz に取り込んだものを YouTube のモバイル Web 実装で使い始めているとのこと。
* **Partial Hydration on Angular SSR**: SSR の機能強化。後の v18 で議論。
* **Future: Component Authoring**: コンポーネントの書き方をさらに楽にできないか模索中。
* **Future: AI for Angular developers**: `ng g` 実行時に AI とインタラクティブな質疑応答ができるようにしていく構想。

## 個人的な感想

### バンドルサイズ

新機能と引き換えに、v17 のランタイムは決して重くありません。`vanilla.js` のような超軽量ライブラリと比較すると、改めて Angular のリッチさ（ペイロードのトレードオフ）を実感しますが、Built-in control flow による `CommonModule` 除外や `@defer` の遅延読み込みを使いこなしてバンドルサイズを意識的に削っていく運用が、v17以降は特に重要になりそうです。

### Signals を実プロダクトで使いたい

`input()` / `model()` / `output()` / `Signal Queries` が一通り揃って、RxJS に頼らずに状態を表現できる選択肢が増えました。プロダクトに入れたい欲は強いですが、既存コードベースの設計思想と混在させると認知負荷が上がるので、入れ方は要設計です。趣味プロダクトで先に慣らしておくのがオススメ。

## まとめ

### v17.x系の主要トピック（一覧表）

| バージョン | 機能 / 概念 | 種別 | 概要 |
| --- | --- | --- | --- |
| **v17.0** | `@if` / `@for` / `@switch` | 新機能 | Built-in control flow。`*ngIf` / `*ngFor` の置き換え |
| **v17.0** | `@defer` | 新機能 | コンポーネント・ビューの遅延読み込み |
| **v17.0** | `Signal stable` | API 安定化 | 一部の旧 API は廃止 |
| **v17.0** | `angular.dev` | ドキュメント | 新公式サイト。Tutorial も刷新 |
| **v17.1** | `input()` | 新機能 | Signal ベースの Input |
| **v17.2** | `model()` / Signal Queries | 新機能 | 双方向バインディング / クエリの Signal 化 |
| **v17.2** | `NgOptimizedImage placeholder` | 新機能 | 画像最適化ディレクティブの強化 |
| **v17.2** | `bun` / `Vite` / `define` / `postcss` | CLI 強化 | 開発体験の底上げ |
| **v17.3** | `output()` | 新機能 | Signal 時代の Output。`@Output` の代替 |
| **v17.3** | `HostAttributeToken` | 新機能 | host node の attribute を inject 可能に |
| **v17.3** | `Router Guard 型` | 開発体験 | `MaybeAsync<GuardResult>` の追加 |
| **v17.3** | `双方向バインディング migration` | 開発体験 | `[(ngModel)]="a && b"` の自動分解機能 |
| **v17.3** | `Material 3 切り替え` | コンポーネント | デザインシステムの大規模刷新 |

### 移行時のメモ

* まずは `ng generate @angular/core:control-flow` で `@if` / `@for` へ機械的に移行する。
* 移行後は `CommonModule` の `import` を見直して、不要なバンドルサイズを削減する。
* `Signal Inputs` に移行すると `ngOnChanges` が不要になる。代わりに `previousValue` などは取れなくなるため、必要なら RxJS の `pipe` 等で代替対応する。
* Material 3 への移行はカラーリングの変化が大きいので段階対応推奨。
* **v17 は変更点が多いので、早めに上げるのが吉（リリース直後の OnAir でもそう言われていた）。**
