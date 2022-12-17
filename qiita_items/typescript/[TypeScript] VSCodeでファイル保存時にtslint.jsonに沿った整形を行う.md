# 前提
* `tslint.json` は設定済み



# 拡張機能と設定
次の拡張機能と設定で実現できる。

## 拡張機能
本記事で扱う TSLint の拡張機能は以下の2つ｡
どちらか一つで実現できるので､好きな方を選べば良い｡

* [TSLint](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-tslint-plugin)
or
* [TSLint(deprecated)](https://marketplace.visualstudio.com/items?itemName=eg2.tslint)

## 設定
設定の対象は TSLint, TSLint(deprecated) のどちらも同じ｡
VSCode の設定ファイルである `setting.json` を編集する。`setting.json` はユーザ設定でもワークスペース設定でもどちらでもお好きな方で。

### TSLint の設定
まずは [TSLint](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-tslint-plugin) の設定｡

```json:setting.json
{
  // エディタの自動整形機能は Off にしておく
  "editor.formatOnSave": false,
  "editor.formatOnPaste": false,
  "editor.formatOnType": false,

  // 拡張機能が提供している自動整形のプロパティ
  // ファイル保存時に tslint.json のルールに則って自動整形する
  "editor.codeActionsOnSave": {
    "source.fixAll.tslint": true
  }
}
```

# TSLint(deprecated)の方の設定
次は [TSLint(deprecated)](https://marketplace.visualstudio.com/items?itemName=eg2.tslint) の設定｡

```json:setting.json
{
  // エディタの自動整形機能は Off にしておく
  "editor.formatOnSave": false,
  "editor.formatOnPaste": false,
  "editor.formatOnType": false,

  // 拡張機能が提供している自動整形のプロパティ
  // ファイル保存時に tslint.json のルールに則って自動整形する
  "tslint.autoFixOnSave": true,
}
```


# 要点
両者ともに

1. エディタ( VSCode )が提供する自動整形のプロパティは無効にする
1. 拡張機能が提供している自動整形のプロパティを有効にする

のがポイント｡
とくに 1 点目の「エディタ( VSCode )が提供する自動整形のプロパティ」はハマりポイント｡

```json:setting.json(抜粋)
  "editor.formatOnSave": false,
  "editor.formatOnPaste": false,
  "editor.formatOnType": false,
```

ファイル保存時に自動整形する設定である `editor.formatOnSave` を `true` に設定して機能を ON にすると期待通りに動かない｡
どうもこの設定が ON の場合は **エディタの自動整形の方が優先的に働く** らしく、`tslint.json` のルールが適用されなくなる。

例えば

* 文字列を「シングルクォーテーション」でくくるというルールを決めているのに、ファイル保存時に「ダブルクォーテーション」に変換される
* メソッドの引数で勝手に改行されてしまう

とかいう目にあう。
もし同じような現象にあっている方が居られたら、上記を見直すと良いかと。
