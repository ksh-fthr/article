# はじめに

本記事の趣旨はタイトルの通り｡
WSL( Windows Subsystem for Linux ) 環境で Rails 環境を構築したときのメモ｡

# 本記事を実施した環境

| ツール                      | バージョン                  | 確認方法              |
| --------------------------- | --------------------------- | --------------------- |
| Windows 10 Home 64bit       |                             |                       |
| Windows Subsystem for Linux |                             |                       |
| Ubuntu                      | 18.04.2 LTS (Bionic Beaver) | $ cat /etc/os-release |
| rbenv                       | 1.1.2-2                     | $ rbenv -v            |
| Ruby                        | 2.6.1p33                    | $ ruby -v             |
| Rails                       | 5.2.3                       | $ rails -v            |
| gem                         | 3.0.1                       | $ gem -v              |
| Bundler                     | 2.0.1                       | $ bundler -v          |
| n                           | 4.1.0                       | $ n --version         |
| Node.js                     | 10.15.3                     | $ node -v             |
| npm                         | 6.4.1                       | $ npm -v              |

# WSL( Windows Subsystem for Linux ) を入れよう

WIndows 10 の Professional でも Home でも WSL は入れられる｡
WSL の導入は次の記事に従って作業すればスムーズに行えた｡

- [WSL（Windows Subsystem for Linux）を使ってみた](https://qiita.com/Brutus/items/f26af71d3cc6f50d1640)
  ( 画像ありの分かりやすい記事､ありがとうございました )

# WSL を入れたら

## 1. 環境を最新にしよう

Ubuntu のターミナルから次のコマンドを実行して環境を更新する｡
`sudo` はスーパーユーザでの実行権限を付与するコマンド。`apt-get` は管理者権限での実行が必要なので、`sudo`をつけて実行する。

```bash:環境を更新する
$ sudo apt-get update
$ sudo apt-get upgrade
```

# Rails 環境を構築しよう

## 注意

実行環境で Kaspersky が起動していると、以降の作業で通信を行う際にエラーとなって各種ツールのインストールやRailsプロジェクトの作成に失敗する。

ここでは Kaspersky を停止して作業を継続した。

## 1. 各種ライブラリを入れよう

[rbenv を入れた](#3-rbenv-を入れよう) あとに rbenv 経由で  Ruby を入れるのだが、その際に [ruby-build という rbenv のプラグインを入れる](#4-ruby-build-を入れよう) 必要がある。
で、その ruby-build のインストールにあたって色々と必要なライブラリがある。
ここでは実際に Ruby を入れる際に発生する余計なエラーを回避すべく、それらを事前に入れておく。

```bash:各種ライブラリのインストール
$ sudo apt-get install make
$ sudo apt-get install gcc
$ sudo apt-get install -y libssl-dev libreadline-dev zlib1g-dev
```

## 2. Node.js を入れよう

こちらも同じく事前にエラーを回避するための手順。
Rails を実行する際に JavaScript のエンジンが必要になるのだが、 Node.js を入れておくことでこれに対応できる。

- 参考
  - [なぜ、Ruby on Railsの動作にNode.jsが必要なのか](https://www.xenos.jp/~zen/blog2/index.php/2019/03/19/post-1831/)

(注)
これはあくまで WSL 上の Ubuntu における環境構築手順なので、Windows 上ですでに Node.js を入れていてもこの手順は実施する必要がある。

インストールは次のコマンドを実行する。

```bash:Node.jsのインストール
$　sudo apt-get install npm
```

これで Node.js そのものはインストールされたのだが `n` というツールを入れることで Node.js のバージョン管理を行うことができる。
`n` のインストールは次のコマンドを実行する。

```bash:nのインストール
$ sudo npm install -g n
$ sudo n stable
```

(注)
手前味噌で恐縮だが、`n` による Node.js のバージョン管理については [別記事](https://qiita.com/ksh-fthr/items/c272384f73f8e319733c) で記載しているので、ご興味あればそちらも参照されたい。
そちらの記事では LinuxMint 18 が確認環境だが、同じ Debian ベースのデストリビューションなので違和感なく進められると思う。

## 3. rbenv を入れよう

どうせやるなら rbenv で Ruby のバージョンを管理したい。
ということで、 rbenv を GitHub から `git clone` で取ってくる。

```bash:rbenvをインストール
$ git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
```

rbenv がインストールできたら次のコマンドを実行して rbenv の PATH を通す。

```bash:rbenvのPATHを通す
＄ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
```

そして次のコマンドを実行して rbenv の設定を行う。

```bash:rbenvの設定を行う
$ ~/.rbenv/bin/rbenv init
```

と打つと

```bash:実行結果
# Load rbenv automatically by appending
# the following to ~/.bashrc:

eval "$(rbenv init -)"
```

とでるので、`eval "$(rbenv init -)"` を `.bashrc` に追記する。
まとめると、`.bashrc` には次の2行が追記されることになる。

```bash:追記される2行
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init -)"
```

最後にターミナルを再起動して環境変数の反映させる。
( `$ source ~/.bashrc` か、もしくは `$ exec $SHELL -l` でも反映できる )

- 参考
  - [公式 の README](https://github.com/rbenv/rbenv#basic-github-checkout)

## 4. ruby-build を入れよう

rbenv をインストールしたら次は Ruby を入れるためのプラグイン、 ruby-build を入れる。

```bash:ruby-buildをインストール
＄ git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
```

ここで [先の手順](#1-各種ライブラリを入れよう) でインストールしたライブラリが効いてくる。
これらのライブラリを入れていない場合は、逐次エラーが発生してその都度ライブラリをインストールしなければならず、非常に鬱陶しい。

## 5. Ruby を入れよう

ようやく Ruby のインストールである。
Ruby のバージョンは任意のもので良い。ここでは `2.6.1` を入れてみる。
次のコマンドを実行して Ruby のバージョン 2.6.1 をインストールする。

```bash:Rubyのバージョン2.6.1をインストール
$ rbenv install 2.6.1
$ rbenv rehash
$ rbenv global 2.6.1
```

で、Ruby のバージョンは。。。

```bash:Rubyのバージョンを確認
$ ruby -v
ruby 2.6.1p33 
```

と、指定したバージョン( `2.6.1` ) がインストールされていることが確認できた。

## 6. Rails を入れよう

Ruby のインストールまで終わったら、次は Rails を入れるための作業にはいる。
Rails は gem 経由でインストールする。バージョンは指定しない。

```bash:Railsのインストール
$ gem install rails
Fetching railties-5.2.3.gem
/home/hogehoge/.rbenv/rbenv.d/exec/gem-rehash/rubygems_plugin.rb:6: warning: Insecure world writable dir /home/hogehoge/.rbenv/versions in PATH, mode 040777
Successfully installed railties-5.2.3
Successfully installed rails-5.2.3
Parsing documentation for railties-5.2.3
Installing ri documentation for railties-5.2.3
Parsing documentation for rails-5.2.3
Installing ri documentation for rails-5.2.3
Done installing documentation for railties, rails after 0 seconds
2 gems installed
```

ということで、2019年05月現在、`5.2.3` のバージョンがインストールされた。
既存の Rails プロジェクトにあわせたバージョンを指定したい場合は次のように指定してやれば良い。
( ここでは `5.2.0` を指定した )

```bash:Railsのインストール(バージョン指定)
$ gem install rails -v 5.2.0
```

## 7. sqlite を入れよう

Rails を使うにあたり SQLite3 のインストールが必要になる。SQLite そのものは gem 経由でインストールするのだが、ここでも依存ライブラリを先に入れておかなければならない。

まずは依存ライブラリのインストールから。

```bash:SQLite3の依存ライブラリのインストール
$ sudo apt-get install libsqlite3-dev
```

次に `SQLite3` のインストール。

```bash:SQLite3のインストール
$ gem install sqlite3 -v 1.3.13
```

## 8. bundler を入れよう

Rails では一旦プロジェクトを作ると、そのプロジェクトで扱っているライブラリは Gemfile で管理され、それらは bundler でインストールすることになる。
ということで、 bundler をインストールしておく。

```bash:bundlerのインストール
$ gem install bundler
```

# Rails プロジェクトを作ろう

## 1. Rails プロジェクトを作ろう

ここまでで Ruby のインストールと Rails のインストール、およびそれらを使う際に必要な最低限のライブラリのインストールが完了した。
実際に Rails プロジェクトを作って起動してみる。

プロジェクトの作成は次のコマンドで行う。

```bash:Railsのプロジェクト作成
$ rails new sample-app
```

## 2. Rails プロジェクトを起動しよう

これで雛形ができたので、プロジェクト直下に移動してアプリケーションを起動する。

```bash:Railsアプリの起動
$ cd sample-app/
$ rails s
=> Booting Puma
=> Rails 5.2.3 application starting in development 
=> Run `rails server -h` for more startup options
Puma starting in single mode...
* Version 3.12.1 (ruby 2.6.1-p33), codename: Llamas in Pajamas
* Min threads: 5, max threads: 5
* Environment: development
* Listening on tcp://localhost:3000
Use Ctrl-C to stop
```

あとはブラウザから http://localhost:3000 にアクセスして、次のページが見れれば環境構築は完了。

![rails_app.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/f9c9b533-3e48-cec0-e286-97cd7ebbfe23.png)


# Rails プロジェクトを Windows 側に配置して Ubuntu からシンボリックリンクを貼って参照しよう

上記までで Ubuntu on WSL の環境に Rails を導入できたが、このままだと Windows 環境から Rails プロジェクトの編集ができない。

だが発想の逆転で、Windows 環境にあるプロジェクトに対して Ubuntu on WSL からシンボリックリンクを貼れば以下が実現できる。

1. Rails プロジェクトを Windows から編集できる
2. Rails アプリの起動は Ubuntu on WSL から行える

ちなみに Ubuntu on WSL から Windows のフォルダは `/mnt/c/` から参照できる。
なので、Windows 側のフォルダに作成した Rails のプロジェクトを Ubuntu 側から参照しに行っても良いのだが、毎回 `/mnt/c` から参照しにくのは流石に面倒臭い。

ということで、シンボリックリンクである。
ここでやることの目的は

- Ubuntu から Windows への参照の手間を減らすこと

で、やることは以下の2つ。

1. あらかじめ Windows のユーザホームフォルダ直下に `Rails` フォルダを作成しておく
2. Ubuntu のユーザホームディレクトリから `Rails` フォルダに対してシンボリックリンクを貼る

以下の作業で `lns -s` を実行し、`Rails` をフォルダにシンボリックリンクを貼る。

```bash:シンボリックリンクを貼る
$ pwd
/home/hoge                      # Ubuntu 上のホームディレクトリ
$ ln -s /mnt/c/Users/hoge/Rails # シンボリックリンクを貼る
```

`ls -l` を実行すると、

```bash:シンボリックリンクを確認
$ ls -l
total 0
lrwxrwxrwx 1 hoge hoge 27 May 22 23:15 Rails -> /mnt/c/Users/hoge/Rails
```

上記のとおり、Windows のフォルダに対してシンボリックリンクが貼れている。

あとはシンボリックリンクを貼ったフォルダに対して

- 既存の Rails プロジェクトをここに移動させる
- ここで Rails プロジェクトを新たに作成する

等を行うことで、Windows から Rails プロジェクトを編集することができる。

これで

- Rails アプリの管理は Ubuntu on WSL
- Rails アプリの編集は Windows

という構成ができた｡

# 所感

気分によって今日は Mac、今日は Windows という感じで作業ができると嬉しいと思い、Windows での Rails 環境構築を実施してみた。
とりあえずこれで Windows で Rails の開発ができるようになったはずなので、これから遊んでみようと思う。

# 参考

- [WSL（Windows Subsystem for Linux）を使ってみた](https://qiita.com/Brutus/items/f26af71d3cc6f50d1640)
- [なぜ、Ruby on Railsの動作にNode.jsが必要なのか](https://www.xenos.jp/~zen/blog2/index.php/2019/03/19/post-1831/)
- [rbenv の公式](https://github.com/rbenv/rbenv)
