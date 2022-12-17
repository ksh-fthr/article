# 本記事をお読みになる前に
掲題のとおり、本記事では Python のソースコードにおいてインデントレベル `2` で自動整形する設定について扱っております。
しかしながら、コメントにて @shiracamus 様からご指摘いただいたように、推奨されているインデントレベルは `4` であり、本記事の内容はそれから外れたものとなりますのでご注意ください。

既存ソースコードに合わせたり、プロジェクトの方針として `4` 以外を設定する、といったケースにおいて本記事が参考になればと思います。

以下、ご指摘いただいた内容を転記いたします。

> 既存コードに合わせてインデントを2にするのはいいですが、そうでないならインデントは4が推奨ですね。
> 
> 参考: [PEP8: Pythonコーディングスタイルガイド](https://pep8-ja.readthedocs.io/ja/latest/)
> 
> > コードのレイアウト
> > インデント
> > 1レベルインデントするごとに、スペースを4つ使いましょう。

@shiracamus 様
ご指摘ありがとうございました。

# はじめに
掲題の要件を実現するのに右往左往したので、今後のための備忘録として記事にします。
また同じことをしようとしている方々への一助となれば幸いです。

# 環境
* Windows10 Pro 64bit
* Python 3.7.2
* VSCode( Visual Studio Code )

# 前提
* VSCode に Python の拡張( [Python( ms-python.python )](https://marketplace.visualstudio.com/items?itemName=ms-python.python) ) を入れている
* 静的解析や自動整形に次のツールを採用している
    * [flake8](https://pypi.org/project/flake8/)
        * 文法チェックツール
        * [pep8](https://pep8-ja.readthedocs.io/ja/latest/) に準拠したチェックをしてくれる
    * [autopep8](https://pypi.org/project/autopep8/)
        * 自動整形ツール
        * [pep8](https://pep8-ja.readthedocs.io/ja/latest/) に沿うよう自動整形してくれる
* `flake8` や `autopep8` のインストールについては各公式の方法を参照のこと


# 結論から
Visual Studio Code で次の設定を行うことで実現できた。

## Visual Studio Code の作業用設定ファイルを編集
プロジェクトディレクトリの直下に設定ファイルを作成して編集する。

* 配置先

```:VSCodeの設定ファイル(settings.json)配置先
<プロジェクトディレクトリ>
├ .vscode
│    └ settings.json
│
```

* 設定内容

```json:python用の設定
{
  "python.pythonPath": "C:\\{{Python実行環境へのPATH}}\\Python\\Python37\\python.exe",
  "files.eol": "\n",                          // 改行コードは「LF」
  "python.linting.enabled": true,             // 文法チェックを行う
  "python.linting.pylintEnabled": false,      // pylint は使わない
  "python.linting.pep8Enabled": false,        // pep8 は使わない
  "python.linting.flake8Enabled": true,       // 文法チェックにflake8 を使う
  "python.linting.flake8Args": [              // flake8 の設定
    "--ignore=E111, E114, E402, E501"
  ],
  "python.formatting.provider": "autopep8",   // 自動整形に autopep8 を使う
  "python.formatting.autopep8Args": [         // autopep8 の設定
    "--indent-size=2",
    "--ignore=E111, E114, E402, E501"
  ],
  "[python]": {
    "editor.tabSize": 2,                      // インデントレベルは「2」
    "editor.formatOnSave": true,              // 保存時に自動整形を行う
    "editor.formatOnPaste": false,            // ペースト時に自動整形を行わない
    "editor.formatOnType": false,             // 入力後に自動整形を行わない
  },
}
```

# 設定内容の説明
## flake8
### flake8 を使う設定
Python( ms-python.python ) を入れると文法チェックとして

* Pylint
* pep8
* flake8

の設定ができるようになる。
今回は `flake8` を使うので、そのための設定を以下の流れで行う。

1. `python.linting.pylintEnabled` はデフォルトで `true` になっていたので `false` に設定する
1. `python.linting.flake8Enabled`、`python.linting.pylintEnabled` 以外には `python.linting.pep8Enabled` があり、こちらはデフォルトで `false` が設定されていたが今回は作業用設定ファイルの方でも明示的に `false` にしている
1. これで `python.linting.flake8Enabled` だけが **有効** になった( 文法チェックツールに `flake8` が設定された )

### flake8 でチェックする内容を設定する
* `"--ignore=E111, E114, E402, E501"` で次のチェックを無視する設定を行った。
    * E111
        * `indentation is not a multiple of four` をチェックする
    * E114
        * `indentation is not a multiple of four (comment)` をチェックする
    * E402
        * `module level import not at top of file` をチェックする
    * E501
        * `line too long` をチェックする

### 設定内容
上記までの内容は [先](#visual-studio-code-の作業用設定ファイルを編集) に挙げた設定のなかで以下の部分に該当する。

```json:python用の設定(flake8の設定部分を抜粋)
  "python.linting.enabled": true,             // 文法チェックを行う
  "python.linting.pylintEnabled": false,      // pylint は使わない
  "python.linting.pep8Enabled": false,        // pep8 は使わない
  "python.linting.flake8Enabled": true,       // 文法チェックにflake8 を使う
  "python.linting.flake8Args": [              // flake8 の設定
    "--ignore=E111, E114, E402, E501"
  ],
```

### 補足(エラーコードについて)
pep8 のエラーコードについては [こちら](https://pep8.readthedocs.io/en/latest/intro.html#error-codes) を参照。

## autopep8
### autopep8 を使う設定
* `python.formatting.provider` で自動整形ツールを設定する
* 選択肢は `autopep8`、`black`、`yapf` で、この中から自動整形ツールを指定することができる
* 今回は次のとおり `autopep8` を指定した。
  * `"python.formatting.provider": "autopep8"`

### autopep8 で自動整形する内容を設定する
* `--indent-size=2` を設定することで自動整形の際にインデントを `2` で整形する
* `--ignore=E402` で `import` 文をファイルの先頭に移動する自動整形を抑止する

### 設定内容
上記までの内容は [先](#visual-studio-code-の作業用設定ファイルを編集) に挙げた設定のなかで以下の部分に該当する。

```json:python用の設定(autopep8の設定部分を抜粋)
  "python.formatting.provider": "autopep8",   // 自動整形に autopep8 を使う
  "python.formatting.autopep8Args": [         // autopep8 の設定
    "--indent-size=2",
    "--ignore=E111, E114, E402, E501"
  ],
```

## flake8, autopep8 以外の設定
他の言語には適用したくない場合、次の用に `"[python]"` でカテゴライズすることで適用範囲を限定することができる。
今回はインデントレベルや自動整形について以下の内容で設定している。

```json:python用の設定(flake8,autopep8以外の設定部分を抜粋)
  "[python]": {
    "editor.tabSize": 2,                      // インデントレベルは「2」
    "editor.formatOnSave": true,              // 保存時に自動整形を行う
    "editor.formatOnPaste": false,            // ペースト時に自動整形を行わない
    "editor.formatOnType": false,             // 入力後に自動整形を行わない
  },
```

# 参考
* [【Python】pipの使い方](https://www.task-notes.com/entry/20150810/1439175600)
* [flake8-Quickstart](http://flake8.pycqa.org/en/latest/index.html#quickstart)
* [Pythonでインデントをスペース2つにした際の周辺ツールの設定](https://tarosky.co.jp/tarog/942)
