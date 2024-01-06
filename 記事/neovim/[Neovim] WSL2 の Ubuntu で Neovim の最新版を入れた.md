# はじめに

下記の環境で [Neovim](https://github.com/neovim) の最新版を入れた作業の備忘録です。
同様のケースで必要に迫られた方の一助となれば幸いです。

# 環境

| 環境 | バージョン | 備考 |
| ---- | --------- | ---- |
| Windows11 Home | 22H2 | |
| WSL |  v2.0.9.0 | `wsl --version` で確認 |
| Neovim | v0.9.5 | 本手順で入れたもの |

# 前提

WSL2, ならびに Ubuntu がインストールされていること。
インストールは [こちら](https://learn.microsoft.com/ja-jp/windows/wsl/install) を参照しながら進めました。

# 課題

WSL2 で入れた Ubuntu において、`apt install neovim` でインストールした Neovim は最新版ではありませんでした。

私は Neovim のプラグインである [telescope.nvim](https://github.com/nvim-telescope/telescope.nvim) を使いたいのですが、このプラグインは `v0.9.0` 以上のNeovim を必要とします。


> ( [Getting Started](https://github.com/nvim-telescope/telescope.nvim?tab=readme-ov-file#getting-started) より転載 )
>
> Neovim (v0.9.0) or the latest neovim nightly commit is required for telescope.nvim to work. The neovim version also needs to be compiled with LuaJIT, we currently do not support Lua5.1 because of some ongoing issues.


# やったこと

ということで、最新版の Neovim を入れるべく、[こちら](https://github.com/neovim/neovim/blob/master/INSTALL.md#linux) の手順に従い AppImage から最新版をインストールしました。

## 古い Neovim の削除

まず `apt install` で入れた Neovim を削除します。

```bash
% sudo apt remove neovim
```

## AppImage から Neovim をインストール

次に AppImage から Neovim をインストールします。

```bash
% curl -LO https://github.com/neovim/neovim/releases/latest/download/nvim.appimage
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 10.6M  100 10.6M    0     0  4291k      0  0:00:02  0:00:02 --:--:-- 7477k

% chmod u+x nvim.appimage
% ./nvim.appimage
```

## エラー発生

問題がなければ上記でインストールされるようですが、私の環境では次のエラーがでました。

```bash
% ./nvim.appimage
dlopen(): error loading libfuse.so.2

AppImages require FUSE to run.
You might still be able to extract the contents of this AppImage
if you run it with the --appimage-extract option.
See https://github.com/AppImage/AppImageKit/wiki/FUSE
for more information
```

## エラーへの対応～インストール完了

前掲の [インストール手順](https://github.com/neovim/neovim/blob/master/INSTALL.md#linux) にあるエラー時の手順に則り、次の手順を実行しました。

```bash
% ./nvim.appimage --appimage-extract
# ログは省略

% ./squashfs-root/AppRun --version
NVIM v0.9.5
Build type: Release
LuaJIT 2.1.1692716794

   system vimrc file: "$VIM/sysinit.vim"
  fall-back for $VIM: "/__w/neovim/neovim/build/nvim.AppDir/usr/share/nvim"

Run :checkhealth for more info

% sudo mv squashfs-root /
% sudo ln -s /squashfs-root/AppRun /usr/bin/nvim
% which nvim
/usr/bin/nvim

% nvim --version
NVIM v0.9.5
Build type: Release
LuaJIT 2.1.1692716794

   system vimrc file: "$VIM/sysinit.vim"
  fall-back for $VIM: "/__w/neovim/neovim/build/nvim.AppDir/usr/share/nvim"

Run :checkhealth for more info
```

無事、Neovim の最新版がインストールされました。


# 参考
- [WSL を使用して Windows に Linux をインストールする方法](https://learn.microsoft.com/ja-jp/windows/wsl/install)
- [UbuntuでNvimを最新にする](https://zenn.dev/temple_c_tech/articles/upgrade-nvim)
- [Neovim > INSTALL](https://github.com/neovim/neovim/blob/master/INSTALL.md)

