

# はじめに

後述の環境において Ruby の `2.2.3` をインストールする必要に迫られたものの、スムーズにはいかずに四苦八苦したので備忘録として残します。
似たような状況に陥った方の一助となれば幸いです。( あと未来の自分のため )



# 環境

| 環境     | バージョン         | 備考                                                        |
| -------- | ------------------ | ----------------------------------------------------------- |
| macOS    | 10.14.x ( Mojave ) |                                                             |
| Homebrew | v2.1.15            |                                                             |
| rbenv    | v1.1.2             |                                                             |
| ruby     | v2.5.1p57          | v2.2.3 のインストールを試みた際に有効になっていたバージョン |



# 発生したエラー

まずは単純に `rbenv` から `2.2.3` のインストールを実行した。
(`${USER}` はログインユーザを示す)

```bash
$ rbenv install 2.2.3
ruby-build: using openssl from homebrew
Downloading ruby-2.2.3.tar.bz2...
-> https://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.3.tar.bz2
Installing ruby-2.2.3...

WARNING: ruby-2.2.3 is past its end of life and is now unsupported.
It no longer receives bug fixes or critical security updates.

ruby-build: using readline from homebrew

BUILD FAILED (OS X 10.14.6 using ruby-build 20191004)

Inspect or clean up the working tree at /var/folders/b1/6bz_xs_j5kgd1z4dj_3wssqc0000gp/T/ruby-build.20191022152952.43808
Results logged to /var/folders/b1/6bz_xs_j5kgd1z4dj_3wssqc0000gp/T/ruby-build.20191022152952.43808.log

Last 10 log lines:
installing capi-docs:         /Users/${USER}/.rbenv/versions/2.2.3/share/doc/ruby
The Ruby openssl extension was not compiled.
ERROR: Ruby install aborted due to missing extensions
Configure options used:
  --prefix=/Users/${USER}/.rbenv/versions/2.2.3
  --with-openssl-dir=/usr/local/opt/openssl@1.1
  --with-readline-dir=/usr/local/opt/readline
  CC=clang
  LDFLAGS=-L/Users/${USER}/.rbenv/versions/2.2.3/lib
  CPPFLAGS=-I/Users/${USER}/.rbenv/versions/2.2.3/include
$ brew list | grep openssl
openssl@1.1
```

通常ならこれで `2.2.3` がインストールされて万々歳、というところが

```bash
BUILD FAILED (OS X 10.14.6 using ruby-build 20191004)
```

と、ビルドに失敗している。。。

エラー情報を見ていくと、次の一文がある｡

```bash
The Ruby openssl extension was not compiled.
```

じゃあこれでググるか、といことで [こちらのページ](http://to-developer.com/blog/?p=1388) にたどり着いた。
で、当該ページを参考にしつつ以下のコマンドを実行。



# エラー解消のために実行したコマンド

## brew のアップデート

```bash
$ brew update
```



## openssl のインストール

```bash
$ brew install openssl
```



## オプションを指定して 2.2.3 のインストール

```bash
$ RUBY_CONFIGURE_OPTS="--with-readline-dir=`brew --prefix readline` --with-openssl-dir=`brew --prefix openssl`" rbenv install 2.2.3
Downloading ruby-2.2.3.tar.bz2...
-> https://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.3.tar.bz2
Installing ruby-2.2.3...

WARNING: ruby-2.2.3 is past its end of life and is now unsupported.
It no longer receives bug fixes or critical security updates.

Installed ruby-2.2.3 to /Users/${USER}/.rbenv/versions/2.2.3
```

無事インストールできた。

あとはインストールした `2.2.3` を有効にしてやれば良い。

```bash
$ rbenv rehash
$ rbenv global 2.2.3
$ ruby -v
ruby 2.2.3p173 (2015-08-18 revision 51636) [x86_64-darwin18]
```

と、上記コマンドを実行して、`2.2.3` が有効になった。



# 参考

* [[Ruby][Mac]The Ruby openssl extension was not compiled. Missing the OpenSSL lib?エラー](http://to-developer.com/blog/?p=1388)
