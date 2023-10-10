
# ユースケース

複数のリポジトリに対して **同じラベルを展開したい**､ 言い方を変えると､ **複数のリポジトリでラベルを共有したい** ケースがある｡
このとき手作業でラベルを設定していくのは手間である｡
そこで [GitHub Action](https://github.co.jp/features/actions) を使って､任意のイベントをフックして実現したい｡



# 使用するツール

- [github-labeler](https://github.com/b4b4r07/github-labeler)



# 構成

```bash:リポジトリ構成
test_workflow
 + .github/
     + workflows/
     |     + unfolding_labels.yml
     + labels.yml
```



# 事前準備

## Personal access tokens の作成

### 手順

**ユーザの設定** で以下を行う｡

1. 画面右上のユーザアイコンをクリック
2. `Settings` をクリック
3. 左側ペインで `Developer settings` を開く
4. 左側メニューで `Personal access tokens` を開く
5. `Generate new token` から
   1. `Note` を入力
      - ここで入力する文字列は [後述](#作成したトークンのリポジトリへの登録) の **リポジトリ-Secrets** の `Name` で指定する
   2. **ワークフロー と リポジトリ に対するアクセスを許可** したいので次の項目にチェックする( 登録後も変更可能 )
      1. `repo`
      2. `admin:repo_hook`
      3. `delete_repo`
      4. `workflow`
6. `Generate token` でトークンを作成
    - ここで生成されたトークンは [後述](#作成したトークンのリポジトリへの登録) の **リポジトリ-Secrets** の `Value` で設定する


### 注意点

作成したトークンは **ここでコピーしておかないと二度と参照できない** ので､忘れずにコピーしておくこと｡

もし忘れてしまった場合は `Regenerate token` で再作成できるが､生成されるトークンは前回とは別のものになってしまう｡



## 作成したトークンのリポジトリへの登録

### 手順

**GitHub Action を設定するリポジトリ** で **Secrets に登録する** ために以下を行う｡

1. `settings` を開く
2. 左側ペインで **`Secrets`** を選ぶ
3. `New secret` から
   1. `Name` を入力
      - ここで入力する文字列は後述の **ワークフローファイル(`unfolding_labels.yml`)** で `GITHUB_TOKEN` を指定する際に使用する
   2. `Value` を入力
4. `Add secret` で登録

### 注意点

トークンを登録する際､ **改行が含まれていると処理が実行された際にエラー** となり､その後 **処理が進まなくなる**｡

```bash:トークンに改行が含まれているときのログ(改行とコメントは筆者が行った)
Sync labels1s
labeler: 2020/09/19 08:12:45 create "WIP" in ksh-fthr/FileUtilityTools
Run b4b4r07/github-labeler@master
/usr/bin/docker run --name e33467befdb6c9c8b44828bb5b4adc7268cad_378975 --label 9e3346 --workdir /github/workspace --rm -e GITHUB_TOKEN -e INPUT_CONFIG -e INPUT_IMPORT -e INPUT_DRYRUN -e HOME -e GITHUB_JOB -e GITHUB_REF -e GITHUB_SHA -e GITHUB_REPOSITORY -e GITHUB_REPOSITORY_OWNER -e GITHUB_RUN_ID -e GITHUB_RUN_NUMBER -e GITHUB_ACTOR -e GITHUB_WORKFLOW -e GITHUB_HEAD_REF -e GITHUB_BASE_REF -e GITHUB_EVENT_NAME -e GITHUB_SERVER_URL -e GITHUB_API_URL -e GITHUB_GRAPHQL_URL -e GITHUB_WORKSPACE -e GITHUB_ACTION -e GITHUB_EVENT_PATH -e GITHUB_PATH -e GITHUB_ENV -e RUNNER_OS -e RUNNER_TOOL_CACHE -e RUNNER_TEMP -e RUNNER_WORKSPACE -e ACTIONS_RUNTIME_URL -e ACTIONS_RUNTIME_TOKEN -e ACTIONS_CACHE_URL -e GITHUB_ACTIONS=true -e CI=true -v "/var/run/docker.sock":"/var/run/docker.sock" -v "/home/runner/work/_temp/_github_home":"/github/home" -v "/home/runner/work/_temp/_github_workflow":"/github/workflow" -v "/home/runner/work/_temp/_runner_file_commands":"/github/file_commands" -v "/home/runner/work/test_workflow/test_workflow":"/github/workspace" 9e3346:7befdb6c9c8b44828bb5b4adc7268cad

# invalid header field value "Bearer ***\n" としてエラーが出ている
2020/09/19 08:12:45 [ERROR] failed to fetch labels ksh-fthr/FileUtilityTools: Get https://api.github.com/repos/ksh-fthr/FileUtilityTools/labels?per_page=10: net/http: invalid header field value "Bearer ***\n" for key Authorization

# ラベルを展開しようとするが､このまま進まない
Post https://api.github.com/repos/ksh-fthr/FileUtilityTools/labels: net/http: invalid header field value "Bearer ***\n" for key Authorization
labeler: 2020/09/19 08:12:45 create "WIP" in ksh-fthr/FileUtilityTools
```





# 設定内容

## ワークフローファイル( unfolding_labels.yml )

```yaml:GitHubActionのトリガーと処理
name: unfolding labels

on:
  issues:
    types: [opened]
    paths:
      - .github/labels.yml

jobs:
  sync:
    name: Run
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@1.0.0
      - name: Sync labels
        uses: b4b4r07/github-labeler@master
        env:
          GITHUB_TOKEN: ${{ secrets.admin_token }}
```

簡単ではあるが説明を｡

### issue 発行をイベントトリガーとする
**issue 発行** を **GitHub Action 発火のイベント** とするのが下記の部分｡
`.github/labels.yml` で展開するラベル定義ファイルを指定している｡

この部分を変更することで､任意のイベントをトリガーに設定することができる｡
イベントについては以下を参照｡


- [ワークフローをトリガーするイベント](https://docs.github.com/ja/actions/reference/events-that-trigger-workflows#) を参照｡

```yaml:issue発行をGitHubActionのトリガーとする
on:
  issues:
    types: [opened]
    paths:
      - .github/labels.yml
```

### GitHub Action で実行される処理
そして GitHub Acton 発火時に実行される処理が下記の部分｡
こちらについては [github-labeler](https://github.com/b4b4r07/github-labeler) のサンプルを流用した｡
詳細はリンク先を参照されたい｡

```yaml:実行される処理
jobs:
  sync:
    name: Run
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@1.0.0
      - name: Sync labels
        uses: b4b4r07/github-labeler@master
        env:
          GITHUB_TOKEN: ${{ secrets.admin_token }}
```



## ラベル管理用ファイル( labels.yml )

```yaml:ラベルの定義､および展開したいリポジトリと設定したいラベルを設定
# ラベルの定義
labels:
  - name: "CS"
    description: "カスタマーサクセス"
    color: "d4c5f9"
  - name: "DEV"
    description: "開発"
    color: "bfe5bf"
  - name: "Document"
    description: "ドキュメント"
    color: "bfe5bf"
  - name: "Release"
    description: "リリース"
    color: "bfdadc"
  - name: "Hotfix"
    description: "不具合修正"
    color: "fef2c0"

# 展開対象のリポジトリと設定するラベル
repos:
  - name: ksh-fthr/test_workflow
    # labels で定義したラベルから設定する
    labels:
      - CS
      - DEV
      - Document
      - Release
      - Hotfix

  - name: ksh-fthr/FileUtilityTools
    # labels で定義したラベルから設定する
    labels:
      - CS
      - DEV
      - Document
      - Release
      - Hotfix

```

このファイルで定義された **ラベル** が､同じく設定されている **リポジトリ** に展開される｡
こちらも詳細は [github-labeler](https://github.com/b4b4r07/github-labeler) を参照されるのが良いが､以下に簡単な説明を｡

### ラベルの定義
見たままではあるが､`labels` をトップに

| 項目 | 説明 | 備考 |
| --- | ---- | --- |
| name | ラベルの名前 |  |
| description | ラベルの説明 |  |
| color | ラベルの色 | 16進数で表現 |

を設定している｡
ここで設定した情報が 後述の [issue 発行後](#issue-発行後) に載せたキャプチャのように表示される｡

```yaml:ラベルの定義(抜粋)
labels:
  - name: "CS"
    description: "カスタマーサクセス"
    color: "d4c5f9"
#
# 省略
#
```



### 展開対象のリポジトリと設定するラベル
`repos` をトップに

| 項目 | 説明 | 備考 |
| --- | ---- | --- |
| name | 展開したいリポジトリ |  |
| labels | 指定したリポジトリに設定したラベル | 上で定義しているラベルを子要素に指定する |

を設定することで､ **展開したいリポジトリ( 複数指定可能 )** と **そのリポジトリに設定したいラベル** を設定している｡

```yaml:リポジトリとラベルの設定(抜粋)
# 展開対象のリポジトリと設定するラベル
repos:
  - name: ksh-fthr/test_workflow
    # labels で定義したラベルから設定する
    labels:
      - CS
      - DEV
      - Document
      - Release
      - Hotfix
#
# 省略
#
```



# 実行してみる

今回は `issue` 発行時にラベルを別のリポジトリに登録したい｡

## issue 発行前後のラベルの状態
issue 発行前と発行後で 2つ のリポジトリに対してラベルが展開されていることを確認する｡
対象のリポジトリは次の 2つ｡

- ksh-fthr/test_workflow
- ksh-fthr/FileUtilityTools


### issue 発行前
- ksh-fthr/test_workflow
    - ![スクリーンショット 2020-09-19 18.55.47.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/8a7563ae-50f3-da62-5135-2ad14e9e67d0.png)

- ksh-fthr/FileUtilityTools
    - ![スクリーンショット 2020-09-19 18.56.57.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/2ef3af96-3c9b-06c6-ae85-ad7a0b51ad65.png)


### issue 発行後
- ksh-fthr/test_workflow
    - ![スクリーンショット 2020-09-19 18.55.59.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/0ab563bb-d837-a8ee-efa9-f009fa517606.png)

- ksh-fthr/FileUtilityTools
    - ![スクリーンショット 2020-09-19 18.59.55.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/84a714ee-cf97-8b18-2b07-09123adb756c.png)


## GitHub Action を確認
![スクリーンショット 2020-09-19 22.41.25.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/ea524041-2375-a231-cc7b-8228f7ac2bb8.png)

↑ から `Sync labels` のブロックを抜き出したのがこちら｡
`test_workflow` と `FileUtilityTools` の 2 つのリポジトリに対してラベルが展開されているのがわかる｡

```bash:正常にラベルの展開が実行されたときのログ
Sync labels
labeler: 2020/09/19 09:53:36 delete "hoge" in ksh-fthr/test_workflow
Run b4b4r07/github-labeler@master
/usr/bin/docker run --name e3346c5edaf3b074147a293961cd9f86922a1_7b86f7 --label 9e3346 --workdir /github/workspace --rm -e GITHUB_TOKEN -e INPUT_CONFIG -e INPUT_IMPORT -e INPUT_DRYRUN -e HOME -e GITHUB_JOB -e GITHUB_REF -e GITHUB_SHA -e GITHUB_REPOSITORY -e GITHUB_REPOSITORY_OWNER -e GITHUB_RUN_ID -e GITHUB_RUN_NUMBER -e GITHUB_ACTOR -e GITHUB_WORKFLOW -e GITHUB_HEAD_REF -e GITHUB_BASE_REF -e GITHUB_EVENT_NAME -e GITHUB_SERVER_URL -e GITHUB_API_URL -e GITHUB_GRAPHQL_URL -e GITHUB_WORKSPACE -e GITHUB_ACTION -e GITHUB_EVENT_PATH -e GITHUB_PATH -e GITHUB_ENV -e RUNNER_OS -e RUNNER_TOOL_CACHE -e RUNNER_TEMP -e RUNNER_WORKSPACE -e ACTIONS_RUNTIME_URL -e ACTIONS_RUNTIME_TOKEN -e ACTIONS_CACHE_URL -e GITHUB_ACTIONS=true -e CI=true -v "/var/run/docker.sock":"/var/run/docker.sock" -v "/home/runner/work/_temp/_github_home":"/github/home" -v "/home/runner/work/_temp/_github_workflow":"/github/workflow" -v "/home/runner/work/_temp/_runner_file_commands":"/github/file_commands" -v "/home/runner/work/test_workflow/test_workflow":"/github/workspace" 9e3346:c5edaf3b074147a293961cd9f86922a1
labeler: 2020/09/19 09:53:35 create "CS" in ksh-fthr/test_workflow
labeler: 2020/09/19 09:53:35 create "CS" in ksh-fthr/FileUtilityTools
labeler: 2020/09/19 09:53:35 create "DEV" in ksh-fthr/test_workflow
labeler: 2020/09/19 09:53:35 create "DEV" in ksh-fthr/FileUtilityTools
labeler: 2020/09/19 09:53:35 create "Document" in ksh-fthr/test_workflow
labeler: 2020/09/19 09:53:36 create "Document" in ksh-fthr/FileUtilityTools
labeler: 2020/09/19 09:53:36 create "Release" in ksh-fthr/test_workflow
labeler: 2020/09/19 09:53:36 create "Release" in ksh-fthr/FileUtilityTools
labeler: 2020/09/19 09:53:36 create "Hotfix" in ksh-fthr/test_workflow
labeler: 2020/09/19 09:53:36 create "Hotfix" in ksh-fthr/FileUtilityTools
labeler: 2020/09/19 09:53:36 delete "hoge" in ksh-fthr/test_workflow
```


# 注意点

**ラベル管理用ファイル( `labels.yml` )** で設定した内容が反映されるという点に留意しておくこと｡

逆に言うと､共有対象のリポジトリですでに登録されているラベルも､ `labels.yml` に存在しなければ本フックで実行されたタイミングで削除される｡

```bash:labels.ymlに定義されていないラベルが削除されているのがわかる
Sync labels
labeler: 2020/09/19 09:06:35 delete "レビュー待ち" in ksh-fthr/FileUtilityTools
Run b4b4r07/github-labeler@master
/usr/bin/docker run --name e33467193efd91f6f43e2968ecbb4116ba8d7_e802b5 --label 9e3346 --workdir /github/workspace --rm -e GITHUB_TOKEN -e INPUT_CONFIG -e INPUT_IMPORT -e INPUT_DRYRUN -e HOME -e GITHUB_JOB -e GITHUB_REF -e GITHUB_SHA -e GITHUB_REPOSITORY -e GITHUB_REPOSITORY_OWNER -e GITHUB_RUN_ID -e GITHUB_RUN_NUMBER -e GITHUB_ACTOR -e GITHUB_WORKFLOW -e GITHUB_HEAD_REF -e GITHUB_BASE_REF -e GITHUB_EVENT_NAME -e GITHUB_SERVER_URL -e GITHUB_API_URL -e GITHUB_GRAPHQL_URL -e GITHUB_WORKSPACE -e GITHUB_ACTION -e GITHUB_EVENT_PATH -e GITHUB_PATH -e GITHUB_ENV -e RUNNER_OS -e RUNNER_TOOL_CACHE -e RUNNER_TEMP -e RUNNER_WORKSPACE -e ACTIONS_RUNTIME_URL -e ACTIONS_RUNTIME_TOKEN -e ACTIONS_CACHE_URL -e GITHUB_ACTIONS=true -e CI=true -v "/var/run/docker.sock":"/var/run/docker.sock" -v "/home/runner/work/_temp/_github_home":"/github/home" -v "/home/runner/work/_temp/_github_workflow":"/github/workflow" -v "/home/runner/work/_temp/_runner_file_commands":"/github/file_commands" -v "/home/runner/work/test_workflow/test_workflow":"/github/workspace" 9e3346:7193efd91f6f43e2968ecbb4116ba8d7
labeler: 2020/09/19 09:06:33 create "CS" in ksh-fthr/FileUtilityTools
labeler: 2020/09/19 09:06:34 create "DEV" in ksh-fthr/FileUtilityTools
labeler: 2020/09/19 09:06:34 create "Document" in ksh-fthr/FileUtilityTools
labeler: 2020/09/19 09:06:34 create "Release" in ksh-fthr/FileUtilityTools
labeler: 2020/09/19 09:06:34 create "Hotfix" in ksh-fthr/FileUtilityTools
#
# labels.yml に定義されていないラベルは削除されている
labeler: 2020/09/19 09:06:35 delete "Ship it!" in ksh-fthr/FileUtilityTools
labeler: 2020/09/19 09:06:35 delete "WIP" in ksh-fthr/FileUtilityTools
labeler: 2020/09/19 09:06:35 delete "dependencies" in ksh-fthr/FileUtilityTools
labeler: 2020/09/19 09:06:35 delete "レビュー修正待ち" in ksh-fthr/FileUtilityTools
labeler: 2020/09/19 09:06:35 delete "レビュー待ち" in ksh-fthr/FileUtilityTools
```


# ハマりどころ

[事前準備](#事前準備) で扱っている **[Personal access tokens の作成](#personal-access-tokens-の作成)** をしていても､ **[作成したトークンのリポジトリへの登録](#作成したトークンのリポジトリへの登録)** を行っていないと `403 Resource not accessible by integration` が発生する｡

```bash:権限エラーが発生している
labeler: 2020/09/19 06:16:10 create "WIP" in ksh-fthr/FileUtilityTools
POST https://api.github.com/repos/ksh-fthr/FileUtilityTools/labels: 403 Resource not accessible by integration []
```

# このあとやりたいこと
- **`dry-run`** との併用
    - [github-labeler](https://github.com/b4b4r07/github-labeler) を見るに `dry-run` の機能は提供されている
    - が､issue 発行時に **dry-run フラグを たてる / たてない** の制御ができるのかな


# ソースコード
今回の記事で動作確認に使用したコードは下記にアップしております｡
ご参考まで。

- [ksh-fthr/test_workflow](https://github.com/ksh-fthr/test_workflow)

# 注意事項
本記事の公開にあわせて [作成したトークンのリポジトリへの登録](#作成したトークンのリポジトリへの登録) で登録した Secrets は削除しております｡ご承知おきください｡

# 参考
- [GitHub Action](https://github.co.jp/features/actions)
- [github-labeler](https://github.com/b4b4r07/github-labeler)
- [ワークフローをトリガーするイベント](https://docs.github.com/ja/actions/reference/events-that-trigger-workflows)
- [個人アクセストークンを使用する](https://docs.github.com/ja/github/authenticating-to-github/creating-a-personal-access-token)
- [Githubのラベル定義を宣言的に集中管理してみた](https://tech.actindi.net/2019/11/13/090900)
