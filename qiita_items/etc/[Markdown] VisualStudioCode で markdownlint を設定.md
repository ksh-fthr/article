# はじめに

Visual Studio Code の 拡張機能 に [markdownolint](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint) を追加しました。  
makrdwonlint は markdown の文法チェックを行ってくれるツールなのですが、項目によってはエラーを出さずにそのままとしておきたい、というケースがあります。

この記事は markdownlint の設定と、上記のようなケースにおける設定について備忘録として残すものです。

# まずは導入から

Visual Studio Code の 拡張機能 として markdownlint をインストールします。  
画像のように左側のペインから拡張機能のアイコンをクリックし、入力欄に `markdownlint` と入力してください。  
こちらの環境ではすでにインストール済みですので `インストール` の表示がでていませんが、未インストールの場合は他の拡張機能のように `インストール` の表示がでますので、クリックしてインストールを行ってください。

(画像挿入)

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

## 見出し1 を複数設定したら怒られた

markdownlint のルールでは 1 ドキュメント中に「見出し1」は 1 つだけ許容するルールとなっております。  
1 ドキュメント中に「見出し1」を複数設定すると

```normal
MD025/single-title/single-h1: Multiple top-level headings in the same documentmarkdownlintMD025
```

が発生し、該当する箇所が「波下線」で修飾されます。

(画像挿入)

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

(画像挿入)

## <details> タグによる折りたたみで怒られた

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

(画像挿入)

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

(画像挿入)

# まとめにかえて

markdownlint の導入〜エラースルーの設定について触れました。  
文中でも記載しましたが、プロジェクトやチームでは気軽にエラーをスルーさせるのではなく、プロジェクトやチームの方針に従い設定していただくべきかと思います。  
ご留意のほど、よろしくお願いいたします。

# 参考

- [markdownlint](https://github.com/DavidAnson/markdownlint)
- [markdownling-Rules](https://github.com/DavidAnson/markdownlint/blob/v0.26.2/doc/Rules.md)
