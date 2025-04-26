## はじめに

かなり過去になりますが、以前に下記で node.js を導入する記事を書きました。

- [[Node.js] Node.js の導入(Linux, Mac編)](https://qiita.com/ksh-fthr/items/c272384f73f8e319733c)
- [[Node.js] Node.js の導入(Windows編)](https://qiita.com/ksh-fthr/items/fc8b015a066a36a40dc2)

そして今更な話ではありますが、今回は [anyenv](https://github.com/anyenv/anyenv) を使った [nodenv](https://github.com/nodenv/nodenv) の導入についての記事なります。
( 毎回調べるの手間なので備忘録がてら記事に残します )


## anyenv インストール手順(  Linux / chromeos / mac  )

### 1. 依存パッケージのインストール( まだなら )

#### linux / chromeos

```bash
sudo apt update
sudo apt install -y git curl build-essential
```

#### mac

```bash
xcode-select --install
```


### 2. anyenv をインストール

```bash
git clone https://github.com/anyenv/anyenv ~/.anyenv
```


### 3. シェルの設定ファイルに anyenv を追加

#### bash の場合( `~/.bashrc` )

```bash
echo 'export PATH="$HOME/.anyenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(anyenv init -)"' >> ~/.bashrc
```

#### zsh の場合( `~/.zshrc` )

```bash
echo 'export PATH="$HOME/.anyenv/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(anyenv init -)"' >> ~/.zshrc
```

#### 設定を反映( bash / zsh 共通)

```bash
exec $SHELL -l
```

### 4. anyenv を初期化

```bash
anyenv install --init
```

**これで、`~/.anyenv/envs` 以下に各言語のバージョン管理ツールをインストールできるようになります！**


### 補足: anyenv-update を入れておくと便利

```bash
git clone https://github.com/znz/anyenv-update.git "$(anyenv root)/plugins/anyenv-update"
```

これで `anyenv-update` が入りました。
あとは必要に応じて下記を実行することで

```bash
anyenv update
```

自分の環境に入っている

- anyenv 本体のアップデート
- インストール済みの各 env ツール( nodenv, pyenv, goenv, rbenv など )のアップデート
- インストール済みのプラグイン( たとえば pyenv-virtualenv ) のアップデート

といったものが一括更新できるようになります。


## nodenv をインストールして Node.js を管理する

```bash
anyenv install nodenv
exec $SHELL -l          # シェルを再読み込み
nodenv install 23.11.0  # 好きなバージョンをインストール
nodenv global 23.11.0
```

**📝 memo 📝**
同じようにして [pyenv](https://github.com/pyenv/pyenv), [goenv](https://github.com/go-nv/goenv), [rbenv](https://github.com/rbenv/rbenv) なども使えます。


### インストール可能な nodejs のバージョンを確認する

#### 事前準備: `node-build` が必要

`nodenv` 単体ではバージョンリストを出せないので、Node.js のビルド機能を提供する `node-build` を一緒に入れておく必要があります。

```bash
nodenv install 23.11.0  # ←みたいにバージョン指定でインストールできるのは node-build があるおかげ
```

次のコマンドで `node-build` のインストールを行います。

```bash
git clone https://github.com/nodenv/node-build.git "$(nodenv root)"/plugins/node-build
```

これで OK。


#### インストール可能なバージョン一覧を表示

バージョン一覧は...

```bash
nodenv install -l
```

or

```bash
nodenv install --list
```

で確認できます。
で、その実行例が下記です。

**出力例( 一部 )**

```bash
% nodenv install --list
18.20.8
20.19.0
22.14.0
23.11.0
graal+ce-19.2.1
graal+ce_java11-20.0.0
graal+ce_java8-20.0.0

Only latest stable releases for each Node implementation are shown.
Use 'nodenv install --list-all / -L' to show all local versions.
```

**📝 memo 📝**
もし `nodenv install -l` を打っても何も出ない場合は、`node-build` が入ってないか、パスが通ってない可能性があります。
その場合はご自身の環境を見直してみてください。

### 補足: よく使うコマンド
[こちら](https://github.com/nodenv/nodenv?tab=readme-ov-file) から転載。

#### [Installing Node versions](https://github.com/nodenv/nodenv?tab=readme-ov-file#installing-node-versions)

バージョンの一覧を出すのとインストールのためのコマンド。

```bash
# list latest stable versions:
nodenv install -l

# list all local versions:
nodenv install -L

# install a Node version:
nodenv install 23.11.0
```

#### [Uninstalling nodenv](https://github.com/nodenv/nodenv?tab=readme-ov-file#uninstalling-nodenv)

`nodenv` をアンインストールする場合は下記で削除してやればよいです。
これで `${nodenv root}/versions/` 配下のすべてのバージョンが削除されます。

```bash
rm -rf `nodenv root`
```

#### [環境全体に反映(nodenv global)](https://github.com/nodenv/nodenv?tab=readme-ov-file#nodenv-global)

バージョン名を `~/.nodenv/version` ファイルに書き込むことで、すべてのシェルで使用される Node のグローバルバージョンを設定します。
このバージョンは、アプリケーション固有の `.node-version` ファイル、または `NODENV_VERSION` 環境変数によって上書きすることができます。

```bash
nodenv global 23.11.0
```

#### [アプリローカル環境に反映(nodenv local)](https://github.com/nodenv/nodenv?tab=readme-ov-file#nodenv-local)

カレントディレクトリの `.node-version` ファイルにバージョン名を書き込むことで、ローカルのアプリケーション固有の Node バージョンを設定します。
このバージョンはグローバル・バージョンをオーバーライドし、`NODENV_VERSION` 環境変数を設定するか nodenv シェル・コマンドでオーバーライドすることができます。

```bash
nodenv local 23.11.0
```

解除する場合は下記を実行すれば良いです。

```bash
nodenv local --unset
```

#### [nodenv version](https://github.com/nodenv/nodenv?tab=readme-ov-file#nodenv-version)

インストールされているバージョンを知りたければ下記を実行します。

```bash
nodenv versions
  18.20.8
  20.19.0
  22.14.0
  * 23.11.0 (set by /Users/will/.nodenv/version)
```

#### [nodenv rehash](https://github.com/nodenv/nodenv?tab=readme-ov-file#nodenv-rehash)

新しい node バージョン( ※-1 ) や npm パッケージをインストールした後にこのコマンドを実行します。
そうすることで `nodenv` がインストールした新しいコマンドを認識できるようになります。

```bash
nodenv rehash
```

※-1
新しい node バージョンをインストールすると標準で入ってるバイナリが増えるケースがあります。
そうしたときにも `nodenv rehash` を行うことで認識されます。

こちらをインストールすることで自動化できます。

- [package-rehash plugin ](https://github.com/nodenv/nodenv-package-rehash)
