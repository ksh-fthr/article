## はじめに

Angular v20 から、`ng generate component` コマンドで作成されるコンポーネントは、デフォルトで `standalone: true` が付与されるようになりました。[^1]
[^1]: これは Angular の CLI 設定 ( standalone: true ) によるものであり、Angular 自体が standalone: true を暗黙的に解釈するわけではありません。

これに伴い VSCode 上では `standalone: false` を明示しても、自動的に削除されることがあります。[^2]
[^2]: これは Angular CLI の設定ファイル ( angular.json ) で `standalone: true` がデフォルトになっているためです。エディタが削除しているのではなく、Angular の Language Service による挙動です。

例えば、以下のように `standalone: false` を明示しても、

```ts
@Component({
  standalone: false,
})
```

保存時に自動的に削除されてしまうことがあります。これは CLI の設定によって `standalone: true` が前提とされているためであり、Angular Language Service による補完動作の一環です。

本記事では **既存プロジェクトに残っている非standalone コンポーネント** と、**v20 で作成した standalone コンポーネント** を共存させる方法、および **standalone コンポーネントのルーティング構成** について整理します。

## 非standalone コンポーネントのルーティング

以前の記事[^3] で扱ったように、非standalone コンポーネントは基本的に `@NgModule` を経由して Angular に認識させます。
[^3]: [[Angular] ルーティング による画面遷移](https://qiita.com/ksh-fthr/items/91c85a06998314c95648)

具体的には次の構成としています。

:::info

* コンポーネントごとに `FeatureModule` を作成
* 各 `FeatureModule` をまとめる `FeatureAllPageRoutingModule` を用意
* 最終的に `AppModule` で読み込む

:::

例：`FeatureTabModule` では `TabBaseComponent` を登録し、`FeatureAllPageRoutingModule` のルート定義に組み込みます。

**FeatureTabModule**

```ts
@NgModule({
    declarations: [SwitchTabComponent, TabBaseComponent, TabAComponent, TabBComponent],
    imports: [CommonModule],
    exports: [SwitchTabComponent, TabBaseComponent, TabAComponent, TabBComponent],
})
export class FeatureTabModule {}
```

**FeatureAllPageRoutingModule**

```ts
const ROUTE_TABLE: Routes = [
    { path: 'tab', component: TabBaseComponent },
];
@NgModule({
    imports: [RouterModule.forRoot(ROUTE_TABLE)],
    exports: [RouterModule],
})
export class FeatureAllPageRoutingModule {}
```

**AppModule**

`AppModule` ではこれらのモジュールをまとめて読み込みます。

```ts
@NgModule({
    declarations: [AppComponent],
    bootstrap: [AppComponent],
    imports: [
        FeatureAllPageRoutingModule,
        FeatureTabModule,
    ],
    providers: [
    ],
})
export class AppModule {}
```

## standalone コンポーネントの共存

v20 では新しく作成したコンポーネントはすべて **standalone** です。
非standalone のように `@NgModule` に登録する必要はなく、 **利用する側のコンポーネントで `imports` に追加するだけ** で使用できます。

```ts
@Component({
    selector: 'app-child',
    standalone: true,
    template: `<p>子コンポーネントです</p>`,
})
export class ChildComponent {}
```

```ts
@Component({
    selector: 'app-parent',
    standalone: true,
    template: `<app-child></app-child>`,
    imports: [ChildComponent], // ← standalone を直接 import. これだけで良い
})
export class ParentComponent {}
```

## standalone コンポーネントのルーティング

今回追加したのは `DesignPatternBaseComponent` と `StatePatternClientComponent` の2つです。
URL は以下のように構成します。

```txt
/design-pattern/state-pattern-client
```

### ポイント

:::note info

* `DesignPatternBaseComponent` が親コンポーネント
* その配下に `StatePatternClientComponent` を子ルートとして配置
* `FeatureDesignPatternModule` を `forChild` ルーティングで定義し、最終的に `FeatureAllPageRoutingModule` で集約

:::

**FeatureDesignPatternModule**

```ts
@NgModule({
    // standalone コンポーネントは NgModule の imports に直接指定できる（v14+）
    imports: [
        RouterModule.forChild([{
            path: '',
            component: DesignPatternBaseComponent,
            children: [{ path: 'state-pattern-client', component: StatePatternClientComponent }],
        }]),
        DesignPatternBaseComponent,
        StatePatternClientComponent,
    ],
    exports: [RouterModule],
})
export class FeatureDesignPatternModule {}
```

:::note info

`NgModule` の `imports` に standalone コンポーネントを直接指定することで、モジュール経由でもルーティングやテンプレートでの利用が可能になります。v14 以降で正式にサポートされた機能です。

:::

**FeatureAllPageRoutingModule**

これを `FeatureAllPageRoutingModule` に組み込みます。

```ts
const ROUTE_TABLE: Routes = [
    {
        path: 'design-pattern',
        loadChildren: () => import('./feature-design-pattern.module').then((m) => m.FeatureDesignPatternModule),
    },
];
@NgModule({
    declarations: [ReadmeComponent],
    imports: [RouterModule.forRoot(ROUTE_TABLE)],
    exports: [ReadmeComponent, RouterModule],
})
export class FeatureAllPageRoutingModule {}
```

**AppModule**

そして `FeatureAllPageRoutingModule` は `AppModule` に登録します。
※ ここは 非standalone コンポーネントの構成から変わりはありません。

```ts
@NgModule({
    declarations: [AppComponent],
    bootstrap: [AppComponent],
    imports: [
        FeatureAllPageRoutingModule,
    ],
})
export class AppModule {}
```

こうして `/design-pattern` でベース、`/design-pattern/state-pattern-client` で子コンポーネントが表示されるようになります。

## standalone コンポーネントと 非standalone コンポーネントを共存させた構成

```txt
AppModule
├── FeatureAllPageRoutingModule
│   ├── 非standalone: TabModule → TabComponent
│   └── standalone: FeatureDesignPatternModule → FeatureDesignPatternModule → StatePatternClientComponent
```

## まとめ

:::note info

* Angular v20 以降、CLI で生成されるコンポーネントはデフォルトで standalone になる
* **非standalone コンポーネント** は従来通り `@NgModule` 経由で利用可能
* **standalone コンポーネント** は `imports` に追加するだけで利用可能（`NgModule` 不要）
* ルーティングは `forChild` を用いて standalone と非standalone を同居させられる
* 段階的に standalone へ移行することで、既存資産を活かしつつ新しい開発スタイルを取り入れられる

:::

## ソースコード

今回の記事で扱ったコードは [こちら](https://github.com/ksh-fthr/angular-work) にあります。
ご興味あればご参照ください。

* [AppModule](https://github.com/ksh-fthr/angular-work/blob/develop/src/app/app.module.ts)
* [FeatureAllPageRoutingModule](https://github.com/ksh-fthr/angular-work/blob/develop/src/app/modules/feature-all-page-rounting.module.ts)
* [FeatureDesignPatternModule](https://github.com/ksh-fthr/angular-work/blob/develop/src/app/modules/feature-design-pattern.module.ts)
  * [FeatureDesignPatternModule](https://github.com/ksh-fthr/angular-work/blob/develop/src/app/component/design-pattern/design-pattern-base.component.ts)
  * [StatePatternClientComponent](https://github.com/ksh-fthr/angular-work/blob/develop/src/app/component/design-pattern/state/state-pattern-client.component.ts)
