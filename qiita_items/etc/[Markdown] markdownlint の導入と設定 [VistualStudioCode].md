# はじめに

Visual Studio Code の 拡張機能 に [markdownolint](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint) markdown の文法チェックを行ってくれるツールがあります。  
これは誤った書き方だったり良くない書き方をチェックしてくれるのでとても重宝するのですが、項目によってはエラーを出さずにそのままとしておきたい、というケースがあります。

この記事は markdownlint の設定と、上記のようなケースにおける設定について備忘録として残すものです。

# まずは導入から

Visual Studio Code の 拡張機能 として markdownlint をインストールします。  
画像のように左側のペインから拡張機能のアイコンをクリックし、入力欄に `markdownlint` と入力してください。

こちらの環境ではすでにインストール済みですので `インストール` の表示がでていませんが、未インストールの場合は他の拡張機能のように `インストール` の表示がでますので、クリックしてインストールを行ってください。

![isntall-markdownlint](https://user-images.githubusercontent.com/3907225/208241965-ffde4aa3-28c9-470b-abfc-0421dd1e7903.png)

# markdownlint の設定

Visual Studio Code における markdownlint の設定を行います。  
まずは次のように `.vscode/settings.json` に設定のためのエントリを追加します。  
( `.vscode/settings.json` がない場合はファイルを作成してください )

```json:settings.json
{
    // markdownlint の設定
    "markdownlint.config": {
        // 必要に応じてここに設定を追加していきます
    }
}
```

# markdownlint のルール

下記が markdownlint のルールに関するドキュメントです。

- [markdownlint-Rules](https://github.com/DavidAnson/markdownlint/blob/v0.26.2/doc/Rules.md)

# markdownlint エラーをスルーする設定

今回は markdownlint で出しているエラーを黙らせたいので、そのための設定をしていきます。

```normal
※ !!!!ご注意!!
ここでご紹介する内容はあくまで私個人の設定によるものです。
推奨を目的としたものではありませんので、その旨ご承知おきください。

markdownlinter の設定につきましては、プロジェクトやチームの方針に従い設定していただくのが良いかと存じます。
```

## 見出し1 を複数設定したら怒られたのでスルーする

markdownlint のルールでは 1 ドキュメント中に「見出し1」は 1 つだけ許容するルールとなっております。  
1 ドキュメント中に「見出し1」を複数設定すると

```normal
MD025/single-title/single-h1: Multiple top-level headings in the same documentmarkdownlintMD025
```

が発生し、該当する箇所が「波下線」で修飾されます。

![markdownlint-error01](https://user-images.githubusercontent.com/3907225/208241958-4b5ce85b-14df-4d3f-8940-9dad672c58c0.png)


これをスルーさせるためには、前述の `.vscode/settings.json` に次の設定を追加します。

```json:settings.json
{
  // markdownlint の設定
  "markdownlint.config": {
    "MD025": false,  // single-title/single-h1: Multiple top-level headings in the same documentmarkdownlintMD025 を黙らせる
  }
}
```

これで当該エラーが出なくなりました。

![markdownlint-error-through01](https://user-images.githubusercontent.com/3907225/208241987-d54af281-7c5e-4f87-9b8a-d4c854d35db9.png)

## `<details>` タグによる折りたたみで怒られたのでスルーする

ドキュメント中の内容で折りたたみたいケースがあります。  
そういうときは次のように記載することで実現できます。

```markdown
<details>
<summary>折りたたみたい内容の概要</summary>

ここに折りたたみたい内容を記載する

</details>
```

しかしながら、上記のような記載をすると

```normal
MD033/no-inline-html: Inline HTML [Element: summary]markdownlintMD033
```

が発生し、該当する箇所が「波下線」で修飾されます。

![markdownlint-error02](https://user-images.githubusercontent.com/3907225/208241995-b3e9c717-1528-4c6b-91fa-8ea8518b88a8.png)

これをスルーさせるためには、前述の `.vscode/settings.json` に次の設定を追加します。

```json:settings.json
{
  // markdownlint の設定
  "markdownlint.config": {
    "MD033": false,  // no-inline-html: Inline HTML [Element: details]markdownlintMD033 を黙らせる
  }
}
```

これで当該エラーが出なくなりました。

![markdownlint-error-through02](https://user-images.githubusercontent.com/3907225/208242003-e46d1336-7a5a-4f6c-a900-33fa0bbad86b.png)

## インデントを `2` にするように怒られたのでエディタの設定を変える

markdownlint にインデントを `2` とするよう怒られた

![スクリーンショット 2022-12-19 21 41 29](https://user-images.githubusercontent.com/3907225/208432690-8f3135e4-dbc9-4a39-bb39-c8dcff8e7d43.png)
```normal
MD007/ul-indent: Unordered list indentation [Expected: 2; Actual: 4]markdownlintMD007
```

ので、その対応のための設定を行います。  
( スルーするのではなく Visual Studio Code の設定でインデントを `2` とします )

### markdown 用の設定エントリを追加する

言語ごとに設定を行いたいので、 [Visual Studio Code 公式のこちら](https://code.visualstudio.com/docs/getstarted/settings#_language-specific-editor-settings) を参考に Markdown 用の設定を行いたいので、`.vscode/settings.json` に次のセクションを追加します。

```json:settings.json
{
  // Markdown 用の設定を行うためのセクション
  "[markdown]": {
    // TODO: Markdown を編集する際のエディタの設定を追加していく
  },
}
```

### インデントを `2` に設定する

先程追加したセクションに次のように設定します。

```json:settings.json
  "[markdown]": {
    "editor.tabSize": 2, 
  },
```

# まとめにかえて

markdownlint の導入〜エラースルーの設定について触れました。

文中でも記載しましたが、本記事でご紹介した内容は個人での利用を目的としております。  
( 本来、文法チェックを行うツールが出すエラーは無闇に黙らせるものではないと考えます )

つきましては、気軽にエラーをスルーさせるのではなく、プロジェクトやチームの方針に従い設定していただくべきかと思いますので、ご留意のほどよろしくお願いいたします。

しつこいとは思いますが再度記載いたします。

> ```normal
> ※ !!!!ご注意!!
> ここでご紹介する内容はあくまで私個人の設定によるものです。
> 推奨を目的としたものではありませんので、その旨ご承知おきください。
>
> markdownlinter の設定につきましては、プロジェクトやチームの方針に従い設定していただくのが良いかと存じます。
> ```

## 補足

短いものではありますが、この記事で設定した settings.json の内容です。

```json:settings.json
{
  "[markdown]": {
    "editor.tabSize": 2,
  },
 
  // markdownlint の設定
  "markdownlint.config": {
    "MD025": false,  // single-title/single-h1: Multiple top-level headings in the same documentmarkdownlintMD025 を黙らせる
    "MD033": false   // no-inline-html: Inline HTML [Element: details]markdownlintMD033 を黙らせる
  }
}
```

# 参考

- [markdownlint](https://github.com/DavidAnson/markdownlint)
- [markdownling-Rules](https://github.com/DavidAnson/markdownlint/blob/v0.26.2/doc/Rules.md)
- [Visual Studio Code Docs > User and Workspace Settings](https://code.visualstudio.com/docs/getstarted/settings#_language-specific-editor-settings)
