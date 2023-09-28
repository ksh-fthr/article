# はじめに
GitHub の dependabot が煩わしくなってきました。
私の場合は主に `package.json` での依存関係でアラートがでるのですが、

1. `dependabot alerts` がでる
1. 内容をみる
1. 手元で依存関係をアップデートなり `npm autdit` で確認して個別に `npm install` したりで対応
1. `package.json` や `package-lock.json` を `push `

といったことをやってきました。
が、先述のとおりこの一連の作業が煩わしくなってきたところ、 Renovate を導入して自動化できるという話を耳にしたので試してみようと思います。

とは言うものの、右も左もわからない状態ではなにが正しいかも分からない、ということろでまずは公式を頼りに進めます。

# 導入
GitHub リポジトリに Renovate を導入します。

手順はこちらのドキュメントを参照。
- [Installing and onboarding Renovate into repositories](https://docs.renovatebot.com/getting-started/installing-onboarding/)

## Renovate のインストール
1. https://github.com/apps/renovate からインストール
   (画像取り忘れていたので [公式ページ](https://docs.renovatebot.com/getting-started/installing-onboarding/)) から拝借)
![01-install.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/03f39b4c-4eee-8589-2ead-df3cabfada3b.png)

1. すべてのリポジトリを対象とするか対象とするリポジトリを絞るか
   ここでは「Only select repositories」を選択
![02-repository-access.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/5ec1949a-486d-5244-33f9-dac86e8eec64.png)

1. 対象のリポジトリを選択
![03-select-repository.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/0116fe72-911d-747c-5e15-4b0268e7e52d.png)


# インストール後の Renovate へのアクセス
Renovate はアカウントに紐づくものらしく、一度インストールしてしまえばあとはアカウントから設定を見直すことができます。
( リポジトリ単位のインストールは不要で「どのリポジトリに Renovate を適用するか」を設定することになります )


1. アカウントの `Settings` から設定画面を開く( 画像は割愛 )
1. 画面左のペインから `Applications` をクリック
![02-setting-applications.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/ce0bb665-c415-142b-f8e6-f9a9d590b4a5.png)

1. `Installed GitHub Apps` タブに `Renovate` がある
![03-configure.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/d5bc0ca4-9811-f8e0-607a-1e0f3d02ac5f.png)

1. `Configure` からアクセス
![04-config.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/18525134-6db1-3699-ed87-9400ef103a5b.png)

1. Renovate の (アカウントに対する) 設定ページに飛ぶ
(途中パスワードによる認証を挟むケースもある)
![05-renovate-settings.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/9a21fe76-d930-46c8-7738-50983b1eb0f3.png)



# 設定
「導入」によってリポジトリに `renovate.json` が登録されるので、それを編集します。
編集項目についてはこちらのドキュメントを参照。
- [Configuration Options](https://docs.renovatebot.com/configuration-options/)

## 導入後の初期設定
Renovate を導入後は `renovate.json` は次のようになっていました。

```json:renovate.json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:base"
  ]
}
```

## Dependency Dashboard の無効化
前掲の設定は「ベースとなる設定」とのことですが、この状態でリポジトリを見ると Issue に Dependency Dashboard という Issue が発行されていました。

![DependencyDashboard.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/4a429132-d527-41b6-54c8-87f7f1be9b95.png)

Dependency Dashboard は Renovate の方で判断してくれた、「バージョンアップしたほうが良い情報」を提示してくれ、かつ Pull Request も作成してくれるインターフェースをもつお助けツールのようです。
ですが、余計なお世話的な部分も多分にあるので無効化したいと思います。

設定は [こちらのドキュメント](https://docs.renovatebot.com/configuration-options/#dependencydashboard) を参考に `renovate.json` を次のように編集します。
( 両方設定する必要はなく、どちらかを設定すれば良い )


**[やり方-1]**
```json:renovate.json
{
  "extends": ["config:base", ":disableDependencyDashboard"]
}
```

もしくは

**[やり方-2]**
```json:renovate.json
{
  "extends": [
    "config:base"
  ],
  "dependencyDashboard": false
}
```

で Dependency Dashboard を無効化できます。

## スケジューリング
Renovate の更新や PR の生成、自動マージのスケジューリングについて記載します。

| 目的 | パラメータ | 備考 |
| --- | -------- | ---- |
| Renovate の更新 | [schedule](https://docs.renovatebot.com/configuration-options/#schedule) | スケジュールオプションは、Renovateのアップデートを行う曜日や月の時間帯を定義することができます。|
|自動マージ        | [automergeSchedule](https://docs.renovatebot.com/configuration-options/#automergeschedule) | automergeScheduleオプションは、RenovateがPRを自動生成する週や月の時間帯を定義するために使用します。 |

※ 備考欄はそれぞれリンク先のドキュメントから引用。


## パッケージに個別の設定を盛り込む
PR の自動マージやその対象となるバージョンの種類、パッケージのマッチングについて記載します。

### major バージョンアップを対象外とする
後述の [自動マージとその対象となるバージョンの種類](#自動マージとその対象となるバージョンの種類) で自動マージの対象を絞れるが、そもそも `major` バージョンに対する更新 PR を作成したくない場合は次の設定を記述する。

```json:renovate.json
{
   "major": {
     "enabled": false
  }
}
```

### 自動マージとその対象となるバージョンの種類
自動マージの際に `major` を対象としたくないので [こちらのドキュメント](https://docs.renovatebot.com/configuration-options/#automerge) を参考に設定します。

| 目的 | パラメータ | 備考 |
| --- | ------- | ---- |
| 対象とするバージョンの種類を指定する | [matchUpdateTypes](https://docs.renovatebot.com/configuration-options/#matchupdatetypes) | このフィールドは、更新の種類に対してルールをマッチさせるために使用します。

※ 備考欄はそれぞれリンク先のドキュメントから引用。

### パッケージのマッチング
また特定のパターンに当てはまる package や特定の名称にマッチする package に対してグルーピングしたいので `matchpackagepatterns` を指定します。

なお 類似の設定で `matchpackagenames` がありますが、こちらは「完全一致」を目的とした場合に使用します。

| 目的 | パラメータ | 備考 |
| --- | -------- | ---- |
|特定のパターンを指定する | [matchpackagepatterns](https://docs.renovatebot.com/configuration-options/#matchpackagepatterns) | このフィールドを使うと、正規表現を書かなくてもパッケージプレフィックスにマッチします。
| 特定の名称を指定する   | [matchpackagenames](https://docs.renovatebot.com/configuration-options/#matchpackagenames) | このフィールドは、パッケージルールの中で一つ以上の完全一致をさせたい場合に使用します。

※ 備考欄はそれぞれリンク先のドキュメントから引用。

## 補足
Renovate ではデフォルトで、 CI 等でテストが実行され問題なければ自動マージが行われるということです。
ただリポジトリによっては CI やテストの設定を行っていないケースもあると思います。
そうしたリポジトリに対して自動マージを行いたい場合は次のパラメータを設定します。

```json:renovate.json
{
  "requiredStatusChecks": null,
}
```

- **参考**
    - [Automerge Enabled, but does not merge #639](https://github.com/renovatebot/config-help/issues/639)
    - [Automerge without tests](https://github.com/renovatebot/renovate/issues/351)
    - [Update dependencies with Renovate](https://dev.to/arxeiss/update-dependencies-with-renovate-37on)

# 設定例
下記設定例のコメント (`// 始まりの文章` ) は実際にはつけないでください。文法エラーになります。

```json:renovate.json
{
  // ベースとなる設定を読み込む
  "extends": [
    "config:base"
  ],

  // Dependency Dashboard を無効化する
  "dependencyDashboard": false,

  // ステータスチェックを行わない
  "requiredStatusChecks": null,

  // Renovate の更新を毎日 00:00-23:59 の間に１時間ごとに行う
  "schedule": [
    "every 1 hour after 00:00 and before 23:59 every day"
  ],

  // PR の生成と自動マージを毎日 00:00-23:59 の間に１時間ごとに行う
  "automergeSchedule": [
    "every 1 hour after 00:00 and before 23:59 every day"
  ],

　　　　// major の更新は対象外とする
  "major": {
    "enabled": false
  },

  // パッケージごとのルール決め
  "packageRules": [
    {
      // `major` 以外の PR は自動マージとする
      "matchUpdateTypes": ["minor", "patch", "pin", "digest"],
      "automerge": true
    },
    {
      // `major` 以外の PR は自動マージとする
      // 加えて `eslint` という文字列にマッチした場合はグループ化する
      "matchUpdateTypes": ["minor", "patch", "pin", "digest"],
      "matchPackagePatterns": ["eslint"],
      "groupName": "eslint",
      "automerge": true
    }
  ]
}
```

上記で作成された PR の例 (バージョンのピン留めの PR)
![PR の例.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/b1a467c5-4b17-0622-fb33-35292c563392.png)

- eslint 関連の修正がまとまっていること
- schedule　の設定が `renovate.json` の内容に沿っていること
- automerge の設定が `renovate.json` の内容に沿っていること

が確認できる。

# 設定エラー
`renovate.json` の設定内容に誤りがあると issue が発行されて教えてくれます。
こんな感じです。助かりますね。

![renovate-json-error.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/66a99f49-28d9-f518-682a-d6737948c645.png)


# まとめにかえて
`renovate.json` の設定内容については試行錯誤中です。

