[[Node.js] Node.js の導入(Windows編)](http://qiita.com/ksh-fthr/items/fc8b015a066a36a40dc2) の Linux, Mac 版。
こちらは [nodist](https://github.com/marcelklehr/nodist) ではなく、[n](https://github.com/tj/n) を使って管理する。

( 2018/02/18 Mac 環境について追記 )

## n
[n](https://github.com/tj/n) は Node.js のバージョンを管理するツール｡ [nodist](https://github.com/marcelklehr/nodist) の Linux版みたいな感じ。
これを入れることでバージョン単位で Node.js を導入することができ､状況に応じてバージョンを切り替えることができる｡

## この記事を実施した環境
* Linux
 * LinuxMint 18
 * n v.2.1.7
 * node ~~v8.4.0~~ v9.4.0
 * npm ~~v5.3.0~~ v5.6.0

* Mac
 * macOS High Sierra( 10.13.x )
 * n v.2.1.7
 * Node.js v9.4.0
 * npm v5.6.0
 
## 最初にやること
nodist と違って、こちらはすでに npm がインストールされていることが前提となる。

### Linux
次のコマンドで npm を管理者権限でインストールする。

```bash:npmのインストール
$ sudo apt-get install npm
```

### Mac
[本家](https://nodejs.org/ja/) から最新版　( 2018/02/18 時点では v9.5.0 )　をダウンロードしてインストールする。

以降、本記事で記載するコマンドは基本的に管理者権限で実行すること。

## n の導入
次のコマンドを実行してインストールを行う。

```bash:nのインストール
$ sudo npm install -g n
```

## node のバージョン管理

1. n で使用できるオプションの確認
次のコマンドを実行して、n が扱えるオプションを確認する。

```bash:nのオプションを確認
$ n --help

Usage: n [options] [COMMAND] [args]

Commands:

  n                              Display downloaded Node.js versions and install selection
  n latest                       Install the latest Node.js release (downloading if necessary)
  n lts                          Install the latest LTS Node.js release (downloading if necessary)
  n <version>                    Install Node.js <version> (downloading if necessary)
  n install <version>            Install Node.js <version> (downloading if necessary)
  n run <version> [args ...]     Execute downloaded Node.js <version> with [args ...]
  n run <version> [args ...]     Execute downloaded node <version> with [args ...]
  n which <version>              Output path for downloaded node <version>
  n exec <vers> <cmd> [args...]  Execute command with modified PATH, so downloaded node <version> and npm first
  n rm <version ...>             Remove the given downloaded version(s)
  n prune                        Remove all downloaded versions except the installed version
  n --latest                     Output the latest Node.js version available
  n --lts                        Output the latest LTS Node.js version available
  n ls                           Output downloaded versions
  n ls-remote [version]          Output matching versions available for download
  n uninstall                    Remove the installed Node.js

Options:

  -V, --version         Output version of n
  -h, --help            Display help information
  -p, --preserve        Preserve npm and npx during install of Node.js
  -q, --quiet           Disable curl output. Disable log messages processing "auto" and "engine" labels.
  -d, --download        Download only
  -a, --arch            Override system architecture
  --all                 ls-remote displays all matches instead of last 20
  --insecure            Turn off certificate checking for https requests (may be needed from behind a proxy server)
  --use-xz/--no-use-xz  Override automatic detection of xz support and enable/disable use of xz compressed node downloads.

Aliases:

  install: i
  latest: current
  ls: list
  lsr: ls-remote
  lts: stable
  rm: -
  run: use, as
  which: bin

Versions:

  Numeric version numbers can be complete or incomplete, with an optional leading 'v'.
  Versions can also be specified by label, or codename,
  and other downloadable releases by <remote-folder>/<version>

    4.9.1, 8, v6.1    Numeric versions
    lts               Newest Long Term Support official release
    latest, current   Newest official release
    auto              Read version from file: .n-node-version, .node-version, .nvmrc, or package.json
    engine            Read version from package.json
    boron, carbon     Codenames for release streams
    lts_latest        Node.js support aliases

    and nightly, rc/10 et al
```

1. 安定版の確認
とりあえず安定版(stable)を利用したいので、次のコマンドを実行して安定版を確認する。
~~この記事を書いた時点では v8.4.0 が安定版だった。~~
( 2018/02/18 の本記事更新時点では、 Mac 環境においては本家のサイトからダウンロードしてきた最新版 pkg とバージョンが前後するが、v9.4.0 が安定版だった )

```bash:安定版を確認
$ sudo n --stable
9.4.0
```

1. 安定版のインストールと切り替え
次のコマンドを実行して安定版をインストールして切り替える。

```bash:安定版のインストールと切り替え
$ sudo n stable

install : node-v9.4.0
mkdir : /usr/local/n/versions/node/9.4.0
fetch : https://nodejs.org/dist/v9.4.0/node-v9.4.0-darwin-x64.tar.gz
######################################################################## 100.0%
installed : v9.4.0

$ node -v
v9.4.0
```

## 終わり
以上で Linux / Mac 環境における Node.js のバージョン管理を行える環境が整った。
プロキシ環境下の設定や npm のアップデートなどは [Windows版の記事](http://qiita.com/ksh-fthr/items/fc8b015a066a36a40dc2) と同じ。
ただし環境変数にプロキシを設定する方法は使用しているシェルによって異なるので注意。
