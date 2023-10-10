# はじめに

mac -> rasbperrypi4 に ssh 接続を行うのに毎回アカウント情報入力するのも面倒なので､公開鍵認証を使って接続することにした｡
今後､環境が変わるたびに調べるのもどうかと思ったので､備忘録として残す｡

# 環境

|          | バージョン                                                   | 備考          |
| -------- | ------------------------------------------------------------ | ------------- |
| macOS    | 10.14.6                                                      | Mojave        |
| raspbian | 10.3                                                         | Raspberry Pi4 |
| ssh      | OpenSSH_7.9p1 Raspbian-10+deb10u2, OpenSSL 1.1.1d  10 Sep 2019 |               |

# mac 側でやったこと

## 秘密鍵と公開鍵を生成する

- 使用するコマンド: `ssh-keygen`

- 実行する内容:
  - `RSA` 方式の秘密鍵と公開鍵を作成する

```bash
$ ssh-keygen -t rsa -b 2048
```

- オプション説明

| オプション | 概要     | 説明                                                         |
| ---------- | -------- | ------------------------------------------------------------ |
| -t         | 方式     | 作成する鍵の暗号化形式を「rsa」（デフォルト）、「dsa」「ecdsa」「ed25519」から指定する |
| -b         | ビット数 | 作成する鍵のビット数を指定する（RSA形式の場合、デフォルトは2048bit） |

これで作業を行ったユーザのホームディレクトリ直下の `.ssh/` に次の構成で秘密鍵と公開鍵が作成された

```bash:秘密鍵と公開鍵の配置先
~/.ssh
├ id_rsa     # 秘密鍵
└ id_rsa.pub # 公開鍵
```

## 公開鍵を接続先に送りつつ authorized_keys に登録する

- 使用するコマンド: `ssh-copy-id `

- 実行する内容:

  -  接続先( RaspberryPi4 )のユーザIDとIPアドレスを指定して公開鍵ファイルを送る

```bash
$ ssh-copy-id -i ~/.ssh/id_rsa.pub pi@XXX.XXX.XXX.XXX # IP アドレスは RaspberryPi のもの
```

- オプション説明

| オプション | 概要 | 説明                           |
| ---------- | ---- | ------------------------------ |
| -i         | 方式 | コピーする鍵ファイルを指定する |

## つないでみる

ここまでの作業で

- RaspberryPi4 に公開鍵が登録された

ので、実際に接続できるかを以下のコマンドで確認する。

- 使用するコマンド: `ssh`

- 実行する内容:

  - 接続先( RaspberryPi4 )のユーザIDとIPアドレスを指定して接続する

```bash
$ ssh -i id_rsa pi@XXX.XXX.XXX.XXX
```

- オプション説明

| オプション | 概要       | 説明                                   |
| ---------- | ---------- | -------------------------------------- |
| -i         | IDファイル | 接続に使用する秘密鍵ファイルを指定する |

## エイリアスを作って接続を簡単に

上記の方法でも問題なく接続できるが、これだと毎回

-  秘密鍵ファイル
-  接続先情報(ユーザID + IPアドレス)

を指定しなければならず面倒くさい。
というわけで、*エイリアス* を作って接続を簡便化する。

エイリアスの作り方は以下のとおり。

- `config` ファイルの作成と編集

```bash
$ vi ~/.ssh/config
```

- 編集内容

```bash:configファイルの中身
Host pi                                    # エイリアス名
  HostName XXX.XXX.XXX.XXX                 # 接続先( IP アドレス )
  User pi                                  # 接続先のユーザ名
  IdentityFile /Users/hogehoge/.ssh/id_rsa # 接続元の秘密鍵ファイルのフルパス
```

## つないでみる( エイリアス作成後 )

- 使用するコマンド: `ssh`

- 実行する内容:

  - `~/.ssh/config` で指定してエイリアスを指定して接続する

```bash
$ ssh pi
```

## パーミッション の変更
config ファイルのパーミッションが `666` とかだった場合､接続に失敗する｡

```bash
$ ls -l
total 40
-rw-rw-rw-  1 hogehoge  staff    87  5 19 23:05 config
-rw-------  1 hogehoge  staff  1856  5 19 22:40 id_rsa
-rw-r--r--  1 hogehoge  staff   418  5 19 22:40 id_rsa.pub
-rw-r--r--  1 hogehoge  staff   351  5 19 22:38 known_hosts
$ ssh pi
Bad owner or permissions on /Users/hogehoge/.ssh/config
```

こういうときは､パーミッションを `644` もしくは `600` に変えれば接続に成功する｡

```bash
$ chmod 600 config
$ ls -l
total 40
-rw-------  1 hogehoge  staff    87  5 19 23:05 config
-rw-------  1 hogehoge  staff  1856  5 19 22:40 id_rsa
-rw-r--r--  1 hogehoge  staff   418  5 19 22:40 id_rsa.pub
-rw-r--r--  1 hogehoge  staff   351  5 19 22:38 known_hosts
$ ssh pi
Linux raspberrypi 4.19.97-v7l+ #1294 SMP Thu Jan 30 13:21:14 GMT 2020 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed May 27 20:32:51 2020 from XXX.XXX.XXX.NNN
pi@raspberrypi:~ $
```

# RaspberryPi4 でやったこと

## authorized_keys の登録を確認

以下に `authorized_keys` が登録されていることを確認する。

```bash
~/.ssh
└ authorized_keys
```

## パスワード認証を無効にする

せっかく公開鍵認証による接続ができるようになったので、従来行っていた `パスワード認証` を無効にしておく。
やることは `sshd_config` の編集と `sshd` の再起動。

- `sshd_config` の編集

```bash
$ sudo vi /etc/ssh/sshd_config
```

- 編集内容

```bash:sshd_configの編集
# 次の内容を追記する、もしくは yes -> no に変更する
PasswordAuthentication no
```

- `ssh` の再起動

```bash
$ sudo service ssh restart
```

## IPアドレスを固定する

IP アドレスが固定されていないと、RaspberryPi4 が起動するたびに IP アドレが変わってしまう可能性がある。
それだとクライアント( mac ) から接続できなくなるので、IP アドレスを固定しておく。

やることは `dhcpd.conf` の編集。

- `dhcpd.conf` の編集

```bash
$ vi /etc/dhcpcd.conf
```

- 編集内容

```bash:dhcpd.confの編集
interface wlan0
static ip_address=XXX.XXX.XXX.XXX/24       # 固定値としたいIPアドレス
static routers=XXX.XXX.XXX.NNN             # ルーターのIPアドレス
static domain_name_servers=XXX.XXX.XXX.NNN # DNS のIPアドレス
# ipv6 については使ってないので設定しない
```

# おまけ

## 公開鍵認証による接続を行うクライアントを増やしたい場合
やることは上でやったことと同じ｡以下の作業を行うことで実現できる｡

1. クライアント側で公開鍵と秘密鍵を生成する
2. 生成した公開鍵をホスト( RaspberryPi4 に登録する )

### クライアント側で公開鍵と秘密鍵を生成する
[秘密鍵と公開鍵を生成する](#秘密鍵と公開鍵を生成する) で実施したコマンドを実行し､公開鍵と秘密鍵を生成する｡

### 生成した公開鍵をホスト( RaspberryPi4 )に登録する
生成した公開鍵を RaspberryPi4 に登録する｡
ただし､このとき､RaspberryPi4 で [パスワード認証を無効にする](#パスワード認証を無効にする) をやっていると､`scp` で送ることもできない｡
そういうときは､すでに公開鍵認証で接続可能なクライアントに公開鍵ファイルを送って､そこで登録してもらえば良い｡

登録の際は以下の様に `authorized_keys` に対して追記する｡

```bash
$ cat id_rsa.pub >> ~/.ssh/authorized_keys
```
※念の為､作業前に `authorized_keys` はバックアップを取っておくと安心｡


# 参考

- [Raspberry Piに公開鍵認証を使ってssh接続する](https://tool-lab.com/raspi-key-authentication-over-ssh/)

- [【 ssh-keygen 】コマンド――SSHの公開鍵と秘密鍵を作成する](https://www.atmarkit.co.jp/ait/articles/1908/02/news015.html)

- [【 ssh-copy-id 】コマンド――SSHの公開鍵を登録する](https://www.atmarkit.co.jp/ait/articles/1908/15/news023.html)

- [【 ssh 】コマンド――リモートマシンにログインしてコマンドを実行する](https://www.atmarkit.co.jp/ait/articles/1701/26/news015.html)

- [ssh - リモートマシンにSSHでログイン - Linuxコマンド](https://webkaru.net/linux/ssh-command/)

- [~/.ssh/configについて](https://qiita.com/passol78/items/2ad123e39efeb1a5286b)

  
