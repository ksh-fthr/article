# はじめに

下記の環境で発生した掲題のエラー対応についての備忘録です。
同様のケースでお困りの方への一助となれば幸いです。

# 環境

| 環境 | バージョン | 備考 |
| ---- | --------- | ---- |
| Windows11 Home | 22H2 | |
| WSL |  v2.0.9.0 | `wsl --version` で確認 |
| Rancher Desktop | v1.11.1 | メニュー > Help > About Rancher Desktop で確認 |


# 前提

WSL2, ならびに Ubuntu と Rancher Desktop がインストールされていること。
Ubuntu のインストールは [こちら](https://learn.microsoft.com/ja-jp/windows/wsl/install) を、 Rancher Desktop のインストールは [こちら](https://zenn.dev/rhene/articles/rancher-desktop-for-windows-with-wsl2) を参照しながら進めました。

# 課題

手前味噌で恐縮ですが、自身の React と Go の学習用リポジトリである [react-and-echo-work](https://github.com/ksh-fthr/react-and-echo-work) を前掲の環境で `docker-compose` から起動しようとした際に以下のエラーが発生しました。

## エラー

```bash
failed to solve: error getting credentials - err: exit status 127, out: ``
```

エラーの詳細は下記になります。

```bash
% docker-compose -f docker-compose-backend.yml up --build
[+] Building 2.5s (4/6)                                                                                                                docker:default
 => [mysql internal] load .dockerignore                                                                                                          0.1s
[+] Building 2.7s (6/6) FINISHED                                                                                                       docker:default
 => [mysql internal] load .dockerignore                                                                                                          0.1s
 => => transferring context: 2B                                                                                                                  0.0s
 => [mysql internal] load build definition from Dockerfile                                                                                       0.1s
 => => transferring dockerfile: 109B                                                                                                             0.0s
 => [api internal] load .dockerignore                                                                                                            0.1s
 => => transferring context: 34B                                                                                                                 0.0s
 => [api internal] load build definition from Dockerfile                                                                                         0.1s
 => => transferring dockerfile: 643B                                                                                                             0.0s
 => ERROR [mysql internal] load metadata for docker.io/library/mysql:8.2                                                                         2.6s
 => ERROR [api internal] load metadata for docker.io/library/golang:1.21.5                                                                       2.5s
------
 > [mysql internal] load metadata for docker.io/library/mysql:8.2:
------
------
 > [api internal] load metadata for docker.io/library/golang:1.21.5:
------
failed to solve: error getting credentials - err: exit status 127, out: ``
```


# やったこと

[エラー内容](#エラー) で調べたところ、[こちらのページ](https://aquasoftware.net/blog/?p=1703) のトラブルシュート( **pull 絡みの操作で credentials のエラー** )に本件と同現象への対応が書かれておりましたので、それを参考に次の設定を行いました。

## 設定ファイルの編集

今回のケースは WSL 上で発生しているので、`~/.docker/config.json` を編集します。
私の環境では同ファイルが存在しておりませんでしたので、新規に作成しました。

```bash
% cd ~/.docker/
% touch config.json
```

そして `config.json` を次の内容で編集しました。

```json:config.json
{
  "credsStore": "pass"
}
```

これで設定は終了です。
上記設定終了後に `% docker-compose -f docker-compose-backend.yml up --build` を実行したところ、前掲のエラーは発生せず無事起動しました。

# 補足

`~/.docker/config.json` に設定する `"credsStore": "pass"` については公式に説明がありますのでご参考までに。

- [Docker-docs-ja > docker login > 認証情報ストアの設定方法](https://docs.docker.jp/engine/reference/commandline/login.html#id11)

また認証関連の記事として下記も参考にさせていただきました。ありがとうございます。

- [docker loginの認証はどこで実行されるか](https://qiita.com/autotaker1984/items/9b88b234b34b0e26fbdd)


# 参考

- [WSL を使用して Windows に Linux をインストールする方法](https://learn.microsoft.com/ja-jp/windows/wsl/install)
- [WindowsでもサクサクDocker (Rancher Desktop with WSL2)](https://zenn.dev/rhene/articles/rancher-desktop-for-windows-with-wsl2)
- [プロキシ環境下で Docker Desktop から Rancher Desktop への移行する](https://aquasoftware.net/blog/?p=1703)
- [Docker-docs-ja > docker login](https://docs.docker.jp/engine/reference/commandline/login.html)
- [docker loginの認証はどこで実行されるか](https://qiita.com/autotaker1984/items/9b88b234b34b0e26fbdd)
