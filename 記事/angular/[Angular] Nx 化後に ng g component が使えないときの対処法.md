## はじめに

この記事では、もともと単体の Angular プロジェクトを運用していた環境に Nx を導入した場合に、`ng g component` が使えずに発生するエラーと、そのケースでの正しいコンポーネント生成手順を解説します。  
Nx 導入後は Angular CLI が使っていた `angular.json` が削除され、`project.json` によるプロジェクト管理に切り替わるため、Angular CLI 単体ではワークスペースを認識できません。
このハマりポイントに悩まされた経験をもとに、解決方法をまとめます。  

同じような単一アプリを Nx 化したケースでハマりやすいポイントも含めて解説するので、Nx 導入後の Angular 開発で困ったときの参考にしてください。

## 背景

**nx を導入した状況**
もともと Angular CLI で構築した単体の Angular アプリを運用していたが、あとから Nx を導入しました。

**nx を導入したあとの現在**
`angular.json` は削除され、ルート直下に `project.json` が生成されました。
アプリは 1 プロジェクトのみ存在している状態です。

**project.json の状況**
ルート直下の `project.json` には以下のように定義されています。

```json
{
  "$schema": "node_modules/nx/schemas/project-schema.json",
  "name": "angular-app",
  "projectType": "application",
  "sourceRoot": "src",
  ...
}
```

ここで注目すべきは `"name": "angular-app"` という項目で、この名前が Nx 上のプロジェクト名になります。

## 発生したエラー

**Angular CLI で生成しようとした場合**
次のコマンドを実行し、コンポーネントを生成しようとしました。

```bash
ng g component src/app/component/desgin-pattern/state/verify-state-pattern
```

**エラー**
しかしながら下記エラーが発生し、コンポーネントの生成に失敗しました。

```bash
Error: This command is not available when running the Angular CLI outside a workspace.
```

## やってみたこと

**Nx CLI での実行**
`ng` が通らないのであれば `nx` で、という考えで次のコマンドを実行しました。

```bash
nx g @nrwl/angular:component src/app/component/desgin-pattern/state/verify-state-pattern
```

※ `nx` はプロジェクト内にインストール済みです
※ package.json から抜粋

```json
  "devDependencies": {
    ...
    "@nx/angular": "21.3.11",
    "@nx/workspace": "21.3.11",
    ...
  }
```

が、これはエラーになりました。

```bash
zsh: command not found: nx
```

どう path が通っていないようです。

**@nrwl/angular が未インストールの場合**
ならば、ということで `npx` を使って実行してみました。

```bash
npx nx g @nrwl/angular:component src/app/component/desgin-pattern/state/verify-state-pattern
```

が、今度は次のエラーになりました。

```bash
NX   Unable to resolve @nrwl/angular:component.
Unable to resolve local plugin with import path @nrwl/angular
```

## 原因

ここまでに発生したエラーの原因をまとめます。

- Nx 導入後は `angular.json` がなく、Angular CLI ( `ng` ) 単体ではワークスペースを認識できない
- Nx CLI ( `nx` ) が未インストール、または `@nrwl/angular` が依存に入っていないとコンポーネント生成が失敗する
- `--project` を指定しないと、Nx がどのプロジェクトに生成するか判断できない

## 解決方法

### 1. Nx CLI をインストール

```bash
npm install --save-dev nx
# または
npm install -g nx
```

後者ならばグローバルで `nx` がインストールされるので path が通ります。

ですが、グローバルを汚したくない方は前者でプロジェクト内にインストールしておくのが良いでしょう。
その場合は `npx` をつかって `npx nx` のように実行する必要があります。

### 2. @nrwl/angular をインストール

`nx` がインストールされいても `nrwl` がインストールされていなければ前掲のエラーになります。
従いまして、次のコマンドを実行して `nrwl` をインストールしておきます。

```bash
npm install --save-dev @nrwl/angular
# または yarn add -D @nrwl/angular
```

### 3. コンポーネント生成

ここまでの手順を踏むことでコンポーネントが正常に生成されるようになりました。

```bash
npx nx g @nrwl/angular:component src/app/component/desgin-pattern/state/verify-state-pattern

 NX  Generating @nrwl/angular:component

CREATE src/app/component/desgin-pattern/state/verify-state-pattern.css
CREATE src/app/component/desgin-pattern/state/verify-state-pattern.spec.ts
CREATE src/app/component/desgin-pattern/state/verify-state-pattern.ts
CREATE src/app/component/desgin-pattern/state/verify-state-pattern.html
```

### 4. デフォルトプロジェクトを設定しておくと安心

`nx.json` にデフォルトプロジェクトとして下記を設定しておくと、後述の理由で安心です。

```json
{
  "defaultProject": "angular-app"
}
```

1. defaultProject の役割
    - Nx で複数プロジェクトを管理している場合、 `ng` や `nx g` コマンドで生成先のプロジェクトを明示せずに使えるようにするための設定
    - 例えば複数のアプリやライブラリがあるワークスペースで便利
    - 指定しておくと、`--project` を省略してもそのプロジェクトを対象にコマンドが実行される
2. 単一アプリ Nx 化のケース
    - ワークスペースにプロジェクトが 1 つしかない場合、Nx は自動的にそのプロジェクトを対象に生成する
    - そのため defaultProject を設定しなくても問題なく動作する
    - 設定しておいてもエラーにはならず、単に「省略可能になっている」という意味合いになる
3. 注意点
    - 将来的に別のアプリやライブラリを追加する可能性がある場合は、defaultProject を設定しておくと `--project` を毎回書かなくて済む
    - 単一アプリなら設定は任意で、削除しても動作には影響なし

## 単一アプリを Nx 化したケース特有のハマりポイント

- 従来の Angular CLI では `ng g component` がそのまま動くと思い込みやすい
- Nx 化後は `apps/` や `libs/` ディレクトリが作られず、ルート直下にコンポーネントが生成される
- 「Angular CLI だけではコンポーネント生成できない」というギャップに注意
- src/app/component や src/app/service といったように作成したい path を指定してやらないとルート直下に生成される点に注意

## ポイント

- Nx 化した既存 Angular プロジェクトでは、Angular CLI 単体での generate は基本的に不可
- コンポーネント生成は **npx nx g @nrwl/angular:component** が正しい方法

## まとめ

- Nx 化後は `ng g component` は使えず、必ず Nx CLI ( `nx` ) を利用する
- `@nrwl/angular` がインストールされていることを確認する
- 単一アプリを Nx 化した場合は「以前通り動く」という錯覚に注意
- src/app/component や src/app/service といったように作成したい path を指定しやらないとルート直下に生成される点に注意
