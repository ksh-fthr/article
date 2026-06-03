## はじめに

⚠️ **この記事は Angular v18.0 〜 v18.1 のリリース時に ng-japan OnAir で扱われた内容を、自分用の視聴メモを整理したものです。**
Zoneless（experimental）の導入や `@let` 構文の追加、Router の柔軟化など、v17 で蒔いた種が一気に芽吹いたバージョンとなりました。

---

## 参考資料

* **Angular Blog**: Angular v18 is now available
* **ng-japan OnAir #78**: Monthly Angular 4月号
* **ng-japan OnAir #79**: Monthly Angular 5月号（v18リリース）
* **ng-japan OnAir #80**: Monthly Angular 6月号
* **Navigating the New Era of Angular**: Zoneless Change Detection Unveiled
* **Event Dispatch in Angular**
* **Template local variables with @let in Angular**

---

## 目次

* [v18.0] Zoneless Change Detection (experimental)
* [v18.0] `ignoreChangesOutsideZone` と Coalescing デフォルト有効
* [v18.0] Router の大幅強化 (`RedirectCommand` / `Route.redirectTo` の関数化)
* [v18.0] ng-content の Fallback Content
* [v18.0] Forms: 統一された Control State Change Events
* [v18.0] Migrations: HttpClientModule の自動置き換え
* [v18.0] Event Dispatch (JSAction)
* [v18.0] i18n hydration developer preview
* [v18.0] ExperimentalPendingTasks API
* [v18.0] その他 stable 化 (@defer / Built-in control flow / Material 3 / angular.dev)
* [v18.1] @let テンプレートローカル変数
* [v18.1] afterRender / afterNextRender の再設計
* [v18.1] その他の改善 (TS 5.5 / toSignal / browserUrl / UrlTree / コンパイラ拡張 / CLIオプション / CSP nonce)
* [v18.1] CDK / Material の細かい改善
* [RFC] DOM Interaction / Hybrid Rendering
* アップグレード体験談: esbuild と CommonJS 依存の落とし穴
* まとめ

---

## [v18.0] Zoneless Change Detection (experimental)

v18 最大の目玉。`zone.js` を使わずに変更検知を行うための experimental（実験的）API が追加されました。

```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { provideExperimentalZonelessChangeDetection } from '@angular/core';

bootstrapApplication(App, {
  providers: [
    provideExperimentalZonelessChangeDetection(),
  ]
});

```

### Zoneless で何が変わるか

* `NgZone` がなくなり、バックグラウンドでの自動変更検知が行われなくなります。
* そのため、変更検知のトリガは開発者が意図的に踏み込む、または特定のイベントを契機にする形になります。
* 具体的には、以下のいずれかが発生したタイミングで変更検知が実行されます。
* `ChangeDetectorRef.markForCheck()` の呼び出し
* `ComponentRef.setInput` の呼び出し
* テンプレートで参照している Signal の更新
* host / template の listener（DOMイベント等）が発火したとき
* dirty（変更あり）マークされた view が attach されたとき
* view が remove されたとき



### OnPush 互換

* Zoneless は `ChangeDetectionStrategy.OnPush` と互換性があります。すでに OnPush で作動しているコンポーネントであれば、Zoneless に移行しても基本的には問題なく動作します。
* つまり「普段から OnPush を意識してコードを設計していれば、Zoneless への移行コストは最小限で済む」ということになります。

### Native await for zoneless apps

* Zoneless 構成にすることで、これまで `zone.js` が非同期処理を追跡するためにパッチ（差し替え）していた `async`/`await` を、ブラウザ本来のネイティブな挙動として直接扱えるようになります。
* これにより、デバッグが容易になり、バンドルサイズが減少するメリットが生まれます。

### CDK / Material は対応済

* Angular CDK および Angular Material は、すでに Zoneless 動作をサポートしているため、これらを利用しているアプリケーションであれば安心して Zoneless 化にチャレンジできます。

---

## [v18.0] `ignoreChangesOutsideZone` と Coalescing デフォルト有効

Zoneless とは別の文脈として、`zone.js` を維持したアプリケーションであっても、デフォルトの変更検知オプションが一部変更されました。

### ignoreChangesOutsideZone

`BootstrapOptions`（または `NgZoneOptions`）に新しいオプションが追加されました。

| オプション設定値 | 挙動の概要 |
| --- | --- |
| **`false` (v18 デフォルト)** | NgZone 外での非同期トリガを「無視しない」 |
| **`true` (v17 までのデフォルト挙動)** | NgZone 外での非同期トリガを「無視する」 |

* **[v17 以前]**
```typescript
bootstrapApplication(App);
// NgZone 外（例：スクロールバーの移動など）の細かなイベントは変更検知の対象外として無視されていた

```


* **[v18 以降 / デフォルト]**
```typescript
bootstrapApplication(App);
// NgZone 外のイベントも検知対象に含まれるようになる

```



⚠️ **注意点**：これまで無視されていたスクロールイベント等の細かな処理が、毎回変更検知のトリガになってしまうため、パフォーマンスの劣化につながる可能性があります。従来の動作（v17 までの挙動）を意図的に維持したい場合は、以下のように `ignoreChangesOutsideZone: true` を明示的に指定する必要があります。

```typescript
provideZoneChangeDetection({ ignoreChangesOutsideZone: true })

```

### Coalescing がデフォルト有効

`zone.js` ありの構成でも `OnPush` 互換を保つために、Coalescing（イベントの合流処理）がデフォルトで有効になりました。

* **Coalescing** とは、`zone.js` にオプトイン（追加機能）で組み込まれていたもので、同一の DOM イベント内で発生した複数の変更検知要求を1つのマイクロタスクに合流させて、まとめて1回だけ変更検知を走らせる仕組みです。

```typescript
bootstrapApplication(App, {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
  ]
});

```

⭐ **頭の整理**：「zoneless」という言葉には、以下の2つの意味が混同されがちなので注意しましょう。

1. **Experimental zoneless** が正式にオプトインとして使えるようになったこと。
2. **zone.js 自体のデフォルト挙動が変化**し、Coalescing などが初期状態で有効になったこと。

---

## [v18.0] Router の大幅強化

### Guard で `RedirectCommand` を返せるように

v17.1 では `router.navigate()` の第2引数として `NavigationBehaviorOptions` (`skipLocationChange` や `replaceUrl`) を指定できるようになりましたが、v18 からはこの動作が Guard（ガード）内でも可能になりました。

* **[v17 以前]**
```typescript
canActivate: [
  () => router.parseUrl('/404'), // URL の遷移履歴やブラウザ履歴を個別に制御できない
]

```


* **[v18 以降]**
```typescript
canActivate: [
  () => new RedirectCommand(
    router.parseUrl('/404'),
    { skipLocationChange: true }, // 遷移履歴を残さずに 404 にリダイレクト可能
  ),
]

```



※ `RedirectCommand` は Guard および Resolver（リゾルバー）の両方で返却可能なほか、エラーハンドラ `withNavigationErrorHandler` でも返却可能になりました。

### `Route.redirectTo` に関数を指定可能

これまでは遷移先のパスとして静的な `string` のみを指定できましたが、v18 からは関数を渡して動的に遷移先を構築できるようになりました。

```typescript
const routes: Routes = [
  {
    path: 'legacy/:id',
    redirectTo: ({ params }) => `/new-path/${params['id']}`,
  },
];

```

※ ただし、これは純粋なリダイレクト専用の処理であり、ブロックガード（アクセス制限など）のような高度な使い方はできません。その場合は、従来どおり Guard を使用します。

### `withNavigationErrorHandler` で `RedirectCommand` を返却可能

ルーティングエラー発生時のエラーハンドリングがさらに柔軟になり、特定のエラークラスに応じてリダイレクト先をコマンド制御できるようになりました。

```typescript
provideRouter(
  routes,
  withNavigationErrorHandler((error) => {
    if (isAuthError(error)) {
      return new RedirectCommand(router.parseUrl('/login'));
    }
    // 通常のエラー時の処理
  }),
);

```

---

## [v18.0] ng-content の Fallback Content

親コンポーネントからコンテンツが渡されなかった場合に、デフォルトの要素（代替コンテンツ）を表示させることができるようになりました。
これは 2016 年に Issue が作成されてから、約 8 年越しに対応された「非常に古い宿題」の解決です。

```typescript
@Component({
  selector: 'my-comp',
  template: `
    <ng-content select="header">Default header</ng-content>
    <ng-content select="footer">Default footer</ng-content>
  `
})
class MyComp {}

@Component({
  template: `
    <my-comp>
      <!-- footer のみコンテンツを渡す -->
      <footer>New footer</footer>
    </my-comp>
  `
})
class MyApp {}

```

**【表示される結果】**

* `header`：親から渡されていないため、Fallback Content である `Default header` が表示されます。
* `footer`：親から `New footer` が渡されているため、`New footer` に置き換わって表示されます。

---

## [v18.0] Forms: 統一された Control State Change Events

`AbstractControl`（および `FormGroup`、`FormControl` などの派生クラス）におけるあらゆる状態変更イベントを、1つの共通 Observable である `events` を通して一元的に追跡できるようになりました。

* **[v17 以前]** statusChanges や valueChanges などの限られたイベントしかなく、touched や pristine などの細かな状態の変化を個別に購読・判別するのは非常に困難でした。
```typescript
this.form.statusChanges.subscribe(/* ... */);

```


* **[v18 以降]**
```typescript
this.form.events.subscribe((event) => {
  if (event instanceof PristineChangeEvent) {
    // pristine 状態の変化を検知
  }
  if (event instanceof TouchedChangeEvent) {
    // touched 状態の変化を検知
  }
  if (event instanceof StatusChangeEvent) {
    // バリデーションステータスの変化を検知
  }
  if (event instanceof ValueChangeEvent) {
    // 値の変化を検知
  }
});

```



---

## [v18.0] Migrations: HttpClientModule の自動置き換え

`ng update` を実行する際、すでに非推奨（Deprecated）扱いになっていた以下のモジュールが、自動的に関数型の `provideHttpClient` シリーズへ機械置換されます。

* `HttpClientModule` $\rightarrow$ `provideHttpClient()`
* `HttpClientJsonpModule` $\rightarrow$ `provideHttpClient(withJsonpSupport())`
* `HttpClientXsrfModule` $\rightarrow$ `provideHttpClient(withXsrfConfiguration(...))`

⚠️ **注意点**：これは依存関係の解決方法が根本から切り替わる、インパクトの大きいマイグレーションです。テスト環境等でリグレッションが発生する可能性がゼロではないため、移行後の動作確認は通常以上に慎重に行ってください。

---

## [v18.0] Event Dispatch (JSAction)

サーバーサイドレンダリング（SSR）におけるハイドレーション周りの機能改善です。

SSR において、HTML がサーバー側で構築されてブラウザへ送られた後、クライアント側で Angular アプリの読み込みと初期化（ハイドレーション）が完全に完了するまでには、わずかな「時間差（ラグ）」が存在します。この期間中、ユーザーが画面上のボタンをクリックしても一切無反応になってしまう問題がありました。

これを解決するために、Google の社内フレームワーク **Wiz** で長年培われてきた技術である **JSAction（Event Dispatch）** が Angular に移植されました。

1. **Angular が起動する前**に、ユーザーが操作したクリックイベント等の挙動をバックグラウンドで一時的に記録・キャッシュしておく。
2. **Angular のハイドレーションが完了した後**に、キャッシュされていたイベント順に処理をシミュレートして流し直す。

※ ただし、これは「Angular の初期起動速度自体が遅い」という根本原因を直接解決するものではないため、その部分のチューニングは今後のバージョンで引き続き進められます。

---

## [v18.0] i18n hydration developer preview

多言語対応（国際化 / i18n）を行っているアプリケーションでの SSR ハイドレーションが、developer preview として新しくサポートされました。

```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { provideClientHydration, withI18nSupport } from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [
    provideClientHydration(withI18nSupport()),
  ]
});

```

※ 普段多言語機能を利用していないプロダクトには直接関係ありませんが、グローバル展開を想定している大規模アプリにとっては待ちに待った重要な機能追加です。

---

## [v18.0] ExperimentalPendingTasks API

主にサードパーティライブラリ開発者やテストフレームワーク向けの、低レイヤーの新しい API です。Angular の実行エンジン上に、未処理 of 非同期タスクが残っているかどうかを精密に判定できます。

```typescript
import { ExperimentalPendingTasks } from '@angular/core';

const pendingTasks = inject(ExperimentalPendingTasks);
const removeTask = pendingTasks.add(); // タスクの追加を宣言

try {
  await someAsyncWork();
} finally {
  removeTask(); // 非同期タスクの完了を通告して削除
}

```

※ 注意点として、非同期タスクの状態は自動でトラッキングされないため、開発者が手動で `add()` と `removeTask()` のペアを呼んでやる必要があります。SSR のレンダリング完了判定や、テストライブラリの待機処理などでの活用が想定されています。

---

## [v18.0] その他 stable 化

いくつかの主要機能が「developer preview」を卒業し、プロダクションレディな **stable（安定版）** としてリリースされました。

| 機能 / 概念 | v17 までのステータス | v18 でのステータス |
| --- | --- | --- |
| **`@defer` (Deferrable Views)** | developer preview | **stable** （あるプロダクトでは初期ロードのバンドルサイズが 50% 削減された実績も） |
| **Built-in control flow (`@if` / `@for`)** | developer preview | **stable** |
| **Angular Material 3** | v17.3 からのオプトイン | **stable** （将来的なバージョンでは完全なデフォルトになる予定） |
| **angular.dev** | developer preview | **stable** （従来の公式ドキュメントサイト `angular.io` は自動リダイレクトされます） |

⚠️ **注意点**：Angular Material 3 は stable になりましたが、v18 時点では自動で有効化されているわけではありません。将来のリリースに備え、今から Material 3 へのデザイン移行計画を立てておくことが推奨されます。

---

## [v18.1] @let テンプレートローカル変数

v18.1 のマイナーアップデートから追加された、テンプレートエンジンの非常に強力な新しい構文です。テンプレートファイル（HTML）の中で、直接ローカル変数（定数）を定義できるようになりました。

```html
@let user = user$ | async;
@let greeting = user ? 'Hello, ' + user.name : 'Loading';

<h1>{{ greeting }}</h1>

```

これまで `*ngIf="user$ | async as user"` や、意味のないダミーのコンテナタグに `as` 構文を組み合わせて無理やり変数化していたハック的実装が、これからは `@let` を使って非常に素直かつクリーンに記述できるようになります。

⚠️ **ただし注意点**：便利だからといって、テンプレート内に複雑なビジネスロジックや計算式を書き連ねることは避けてください。`@let` に巨大な三項演算子などをぶら下げ始めると、コンポーネントクラス（TypeScript）が担うべき責務がテンプレートに漏れ出し、コードの可読性が大きく低下します。

---

## [v18.1] afterRender / afterNextRender の再設計

レンダリング完了後に特定のDOM処理を安全に実行するための `afterRender` API が、developer preview の中で大きく再設計され、インターフェースが変更されました。

* **[v18.0 以前]** 従来の書き方では、個々のコールバック関数に対してどのフェーズで実行するかを指定する必要があったため、フェーズ間で状態を受け渡すための「共有変数（一時的な退避変数）」をコンポーネントのスコープ等で定義しなければならず、コードが煩雑になりがちでした。
```typescript
import { afterRender, AfterRenderPhase } from '@angular/core';

let size: DOMRect | null = null; // 共有用の変数が必要だった

afterRender(() => {
  size = nativeEl.getBoundingClientRect(); // 読み込みフェーズ
}, { phase: AfterRenderPhase.EarlyRead });

afterRender(() => {
  otherNativeEl.style.width = size!.width + 'px'; // 書き込みフェーズ
}, { phase: AfterRenderPhase.Write });

```


* **[v18.1 以降]** 新設計では、コールバックを直接複数呼ぶ代わりに1つの設定オブジェクト（Object）を渡し、各フェーズのメソッド間で戻り値をチェーンさせることができるようになりました。
```typescript
afterRender({
  earlyRead: () => nativeEl.getBoundingClientRect(), // earlyRead から返却された値が
  write: (rect) => {
    otherNativeEl.style.width = rect.width + 'px'; // 自動的に write の引数として渡る
  },
});

```


これにより、余計な共有変数スコープの定義が不要になり、レンダリングフローにおける「読み（Read）」と「書き（Write）」の依存関係を1箇所にきれいにカプセル化できるようになりました。
※ `ng update` の実行によって自動的にコード変換（マイグレーション）が適用されますが、構造が大きく変わるため、アップデート直後に必ず挙動を確認しておくと安心です。

---

## [v18.1] その他の改善

### TypeScript 5.5 のフルサポート

v18.1 から最新の TypeScript v5.5 が正式にサポートされました。プロジェクトをアップデートする際は、`tsconfig.json` の `compilerOptions` などの記述も TypeScript 5.5 の仕様に準じているか見直しておきましょう。

### `toSignal` に equality function を追加

RxJS の Observable から Signal へ変更をストリーム変換する `toSignal` 関数に、値の同一性を判定するためのカスタム等価関数（`equal`）がオプション指定できるようになりました。

```typescript
toSignal(obs$, {
  equal: (a, b) => a.id === b.id, // オブジェクトの特定プロパティをキーに同一性を判定
});

```

※ ただし、ng-japan OnAir の解説でもあったように、アプリケーション全体で `toSignal` を乱発・多用して RxJS と Signal をパッチワークのように継ぎはぎするのはあまりお勧めされません。「本格的に Signal の恩恵を受けたい場合は、橋渡しに頼るのではなくコンポーネント全体を Signal ベースに書き換えていくべき」とのことです。

### Router: UrlTree を `routerLink` の入力に指定可能

`[routerLink]` ディレクティブのバインディング先として、これまでのパスの配列（`[ '/users', id ]`）だけでなく、構築済みの `UrlTree` オブジェクトをそのまま指定できるようになりました。

```html
<a [routerLink]="urlTree">Go</a>

```

### Router: `NavigationBehaviorOptions.browserUrl`

ルータが実際にコンポーネントを切り替える内部的な遷移先 URL と、ユーザーに対してアドレスバーで見せる表示用 URL を完全に切り分ける機能が追加されました。

```typescript
router.navigateByUrl('/error/404', {
  browserUrl: router.parseUrl('/users/123/missing'),
});

```

例えば、「ページの実体は `/error/404` というエラー共通コンポーネントに遷移させたいが、ユーザーのアドレスバーには原因となった `/users/123/missing` という本来のパスを残したままにしたい」といった場合に非常に便利です。

### コンパイラ拡張: テンプレート診断の強化（`()`の付け忘れチェック）

HTML テンプレート内でメソッド呼び出しを行う際に、メソッドの後ろに括弧 `()` を付け忘れてプロパティ参照のようになってしまっているバグを、コンパイラがビルド時に自動検出してコンパイルエラーとして警告してくれるようになりました。

```html
<!-- 【NG】() の付け忘れ：従来は静かに無視されていたものがコンパイルエラーになります -->
<button (click)="handleClick">Click</button>

<!-- 【OK】メソッド実行が明示されているため通過します -->
<button (click)="handleClick()">Click</button>

```

### CLI: CSP nonce を script src tags に追加

セキュリティ対策（コンテンツセキュリティポリシー / CSP）を強化するため、出力される HTML の `<script src>` タグに対して、ビルド時に指定した `nonce` を自動付与するオプションが追加されました。

```html
<!-- 出力されるHTMLのイメージ -->
<script src="main.js" nonce="YOUR_NONCE_VALUE"></script>

```

※ これまでインラインのスタイルシートや一部のスクリプトにしか nonce が届きにくかった点が改善され、インフラ層での CSP ガードがより厳格かつ綺麗に設定できるようになります。

### Language Service の autocompletion（自動補完）強化

開発中にまだ TypeScript コード側で `import` を宣言していないコンポーネントがある状態でも、エディタ（VS Code等）の HTML テンプレート上でタグ名を入力し始めると、Language Service が自動で該当コンポーネントを補完リストに表示し、`import` 文まで自動補完してくれるようになりました。これによってテンプレート編集時の手戻りが大幅に削減されます。

### CLI: `--inspect` オプションの拡張

`ng serve` や `ng test` などの開発サーバー・ランナーを起動する際に、Node.js 側のデバッガを紐づけるための `--inspect` オプションが強化されました。

```bash
% ng serve --inspect

```

これにより、SSR/SSG の処理を実行中に Node.js 側のデバッガをより深く潜り込ませて、ブラウザ外で動いている Angular アプリケーションの挙動を直接ステップ実行しながら調査できます。

---

## [v18.1] CDK / Material の細かい改善

### cdk drag-drop: mixed orientation サポート

Angular CDK のドラッグ＆ドロップ機能（`@angular/cdk/drag-drop`）が進化し、リスト要素の並び順が「横一列（水平）」と「縦一列（垂直）」が混在（mixed）しているグリッド風のレイアウトであっても、ドラッグ中のプレースホルダー移動や並べ替えの動作が自然に追従するようになりました。

### Angular Material: button の color が DI 経由で設定可能に

従来、個々のボタンのカラーリングなどを変更する際はテンプレート上で属性値を渡すなどしていましたが、v18.1 からは Dependency Injection（DI）コンテキストを利用して、アプリケーション全体、あるいはコンポーネントツリーの特定の階層以下に適用するデフォルト設定としてボタンカラーなどを渡すことが可能になりました。

---

## [RFC] DOM Interaction / Hybrid Rendering

v18 では、SSR/SSG 周りの設計方針を根本から改変するための2本の RFC（Request for Comments）が議論・提案されていました。これは将来の v19 以降の大きな伏線となる機能です。

### RFC: DOM Interaction

サーバーサイド（SSR）で構築した画面を引き継ぐ（ハイドレーション）際、これまでの `ElementRef` の乱用を見直し、`DomRef` や `ElementRef` の役割を段階的に整理・廃止していく方向性が示されました。
「通常の Web アプリであれば、ElementRef の直接アクセスはほぼ 100% 無くせるはずであり、それが本当に必要なニッチな箇所だけをどう綺麗に残すか」という、Angular 本来のマルチプラットフォーム（Web/Mobile/Server）思想を強めるための議論です。

### RFC: Hybrid Rendering

もっとも強力な将来方針。これまでは「アプリ全体を SSR にする」「全体を SSG にする」といった大雑把な切り替えしかできませんでしたが、**ルート（パス）単位でレンダリングの合意（SSR / SSG / CSR）を自在に指定できる仕組み**です。

```typescript
import { ServerRouteConfig } from '@angular/ssr';

export const serverRouteConfig: ServerRouteConfig[] = [
  { path: '/login', mode: 'server' },     // SSR（毎回サーバーで動的ビルド）
  { path: '/**', mode: 'prerender' },    // SSG（事前プリレンダリングで静的書き出し）
];

```

※ 「ログイン画面のように毎回アクセス元によって処理を切り分けたい画面は `server（SSR）` で動的に処理し、それ以外のコンテンツページや利用規約は `prerender（SSG）` で静的に配信する」といった、高度なハイブリッド構成が設定ファイル一発で実現可能になります。

---

## アップグレード体験談: esbuild と CommonJS 依存の落とし穴

実際に趣味の学習用アプリを v17 から v18 にアップデートした際の実体験メモです。ビルドの設定を新しい `use-application-builder`（esbuild ベース）に有効化した状態で起動したところ、以下の不気味なエラーに直面しました。

```text
Uncaught Error: Dynamic require of "microphone-stream" is not supported

```

### 【原因】

* `use-application-builder`（webpack $\rightarrow$ esbuild へのビルドエンジンの移行）を選択したことで、内部ビルドが高速な esbuild に切り替わります。
* しかし、esbuild は基本的に **ESM（ES Modules）が標準の設計思想**であるため、CommonJS（CJS）向けに書かれた古いライブラリ（この場合は `microphone-stream`）の動的な `require()` 記述をサポートしていません。
* そのため、動的読み込みが処理できず、アプリ起動直後の DevTools 側でエラーとなって停止してしまいました。

### 【対応方針の検討】

直面した際に取れる選択肢とメリット・デメリットは以下の通りです。

| 対応方針 | メリット | デメリット |
| --- | --- | --- |
| **① webpack ビルドに戻す** | 一時的にすぐ解決する。 | 今後 esbuild が標準になる方向なので、根本対策にならず先延ばしになるだけ。 |
| **② 既存パッケージを ESM 対応版に置き換える** | 根本対応となり、ビルド最適化も進む。 | 代替パッケージの選定と、元のコードを書き直すためのリプレースコストが発生する。 |
| **③ 当該依存を Angular のビルド外で扱う** | 部分的に隔離できる。 | 設計の複雑化を招く。 |

⚠️ **対策のベストプラティス**：esbuild への本格移行を目前に控え、`node_modules` 配下の主要な外部ライブラリが **ESM に対応しているか**をあらかじめ確認（監査）しておく必要があります。これを行うことで、本番デプロイ時の予期せぬ事故を未然に防ぐことができます。
※ ちなみに、webpack から esbuild に切り替えることで、**ビルド時間が劇的に短縮される** という素晴らしい恩恵が得られます（Qiita 等でもその効果が広く実証されています）。

---

## まとめ

### v18.x系の主要トピック（一覧表）

| バージョン | 機能 / 概念 | 種別 | 概要 |
| --- | --- | --- | --- |
| **v18.0** | `provideExperimentalZonelessChangeDetection` | 新機能 | Zoneless 変更検知（experimental）のサポート開始 |
| **v18.0** | `ignoreChangesOutsideZone` | 挙動変更 | `NgZone` 外のイベントもデフォルトで検知対象に含まれるように変更 |
| **v18.0** | `Coalescing デフォルト有効` | 挙動変更 | 同一DOMイベント内の複数の検知要求を1つのマイクロタスクに合流 |
| **v18.0** | `RedirectCommand` | 新機能 | Guard / Resolver / ErrorHandler から動的リダイレクト指示が可能に |
| **v18.0** | `Route.redirectTo` の関数化 | 新機能 | パスパラメータを受け取って動的に遷移先パスを返却できる |
| **v18.0** | `ng-content Fallback Content` | 新機能 | 親からコンテンツが渡されなかった時のデフォルト内容を表示可能に |
| **v18.0** | `Forms events Observable` | 新機能 | `touched` や `pristine` などフォームの全状態変化を統一的に購読 |
| **v18.0** | `HttpClientModule 自動マイグレーション` | Breaking | `ng update` で `provideHttpClient` 関数型へ自動置換 |
| **v18.0** | `Event Dispatch (JSAction)` | 新機能 (SSR) | ハイドレーション完了前にユーザーが起こしたクリックイベント等を再送 |
| **v18.0** | `withI18nSupport()` | 新機能 (SSR) | 多言語対応（i18n）環境下での SSR ハイドレーションを開発プレビュー |
| **v18.0** | `ExperimentalPendingTasks` | 新機能 | 非同期タスクの有無を手動判定できる低レイヤー向け API の追加 |
| **v18.0** | `@defer` / `Built-in control flow` / `Material 3` | stable 化 | v17 で登場した各種プレビュー機能がプロダクションレディに安定 |
| **v18.1** | `@let` | 新機能 | テンプレート（HTML）内で直接ローカル変数を宣言可能に |
| **v18.1** | `afterRender / afterNextRender 再設計` | 挙動変更 | フェーズごとのメソッドを1つの Object 渡しに整理し、値をチェーン可能に |
| **v18.1** | `TypeScript 5.5 サポート` | 開発体験 | TS 5.5 の仕様をフルサポート |
| **v18.1** | `toSignal equality` | 新機能 | Signal へのストリーム変換時にカスタム等価比較関数を挟めるオプション |
| **v18.1** | `Router: browserUrl / UrlTree` | 新機能 | 表示用アドレスバーURLの差し替えと、`routerLink` への `UrlTree` 直接渡し |
| **v18.1** | `コンパイラ拡張 (括弧忘れ検出)` | 開発体験 | `(click)="handleClick"` などの括弧 `()` 付け忘れをコンパイルエラーとして警告 |
| **v18.1** | `CLI: --inspect` / `CSP nonce` | 開発体験 / セキュリティ | デバッグ効率の向上と `<script src>` への nonce 動的付与 |

---

## 移行時のメモ

* **Zoneless を試すなら**:
変更検知のパフォーマンスを最大化するため、普段から各コンポーネントを `OnPush` で書いておく習慣が肝要になります。
* **`ignoreChangesOutsideZone` の挙動変更**:
デフォルト変更により、スクロールやドラッグなど「Zone外」で処理したいはずの細かな非同期トリガで、毎回変更検知が走ってしまいパフォーマンスが劣化する恐れがあります。旧来の挙動（検知対象外）を維持したい場合は、`{ ignoreChangesOutsideZone: true }` を明示的に渡しましょう。
* **`HttpClientModule` の置き換え**:
`ng update` 経由で自動マイグレーションがかかりますが、非常に重要な通信レイヤーの変更です。移行テストが手戻り（リグレッション）しないよう入念にチェックしておきましょう。
* **`afterRender` の再設計**:
v18.1 でオブジェクト型での引数渡しに変更されました。古い書き方でも一応動きますが、アップデート時に警告が出るため、公式コードを参考に段階的な移行を進めましょう。
* **esbuild（Application Builder）への切り替え**:
ビルド時間が劇的に短縮されますが、依存パッケージが ESM 規格に対応しているかが勝負の分かれ目になります。不意のビルドエラーを防ぐため、事前にライブラリの棚卸しを行っておくのが安全です。
* **今後のキーワード**:
今後はさらに **Signal と Zoneless** の親和性が強化されます。来る v19 以降の変化を楽に受け止めるために、早めに v18 への移行と設計のブラッシュアップを行っておくのがベストです！
