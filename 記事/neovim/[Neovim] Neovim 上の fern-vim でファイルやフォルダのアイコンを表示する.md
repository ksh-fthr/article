## はじめに

本記事は [Neovim](https://github.com/neovim) 上で [vim-fern](https://github.com/lambdalisue/vim-fern) を用いた環境で、ファイルツリー上でアイコンを表示するにあたっての備忘録です。

とはいっても、ここで記す設定はほぼ こちら

https://qiita.com/youichiro/items/b4748b3e96106d25c5bc

の記事で示されたものです。
vim-fern の設定について大変お世話になりました。素晴らしい記事をあげていただきありがとうございます。

本記事はその中で、私自身が忘れがちな部分について扱います。( ゆえに備忘録です )

## 環境

|   | 説明 |
|---| ---- |
| Windows11 Home | バージョンは `22H2` |
| WSL | バージョンは `v2.0.9.0` |
| [Neovim](https://github.com/neovim) | Vim ベースのテキストエディタ |
| [vim-fern](https://github.com/lambdalisue/vim-fern) | Vim, Neovim 上で動くファイルツリービューア |
| [lua](https://www.lua.org/) | スクリプト言語. 本記事で扱う環境では Neovim の環境設定で使っている | 
| [packer](https://github.com/wbthomason/packer.nvim) | プラグインマネージャー |


**補足**

↑ の表にも記載していますが、本記事では Neovim の設定に `Lua` を用いています。
後述の設定はすべて `Lua` での記述になりますので予めご承知おきください。

## 事前準備

### フォントのインストール

フォント: `Menlo Nerd Font` を下記からインストールします。これは後行程で同フォントを用いたプラグインの設定を行うためです。

- https://github.com/yumitsu/font-menlo-extra/blob/master/Menlo-Regular-Normal.ttf

![download-menlo-nerd-font.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/99293cd0-d4a9-deaa-080f-c501b6c7d3e3.png)

### ターミナルの設定

前項でインストールしたフォントをターミナルのフォントに設定します。これもの後行程の設定のために必要です。

![font-setting01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/329764f6-bcec-50d5-8c0f-0acbdaa27c9c.png)
![font-setting02.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/628dda1e-8005-7d34-0cc6-8b15a5de8006.png)


※ ここでは ペンギンアイコンの Ubuntu を対象にしたキャプチャを貼っていますが、Windows PowerShell 以降のすべての環境で同じフォント( `Menlo Nerd Font` )を設定しています。


## プラグイン

### プラグインのインストール

vim-fern 上でアイコン表示を行うためのプラグインをインストールします。
私はプラグインマネージャとして [packer](https://github.com/wbthomason/packer.nvim) を使っているので、次のように設定しました。

```lua
-- Install your plugins here
packer.startup(function(use)
  -- ..(略)..
  use({'lambdalisue/nerdfont.vim'})                -- アイコン表示のための拡張
  use({'lambdalisue/fern-renderer-nerdfont.vim'})  -- アイコン表示のための拡張
  use({'lambdalisue/glyph-palette.vim'})           -- アイコン表示のための拡張
  -- ..(略)..

  -- Automatically set up your configuration after cloning packer.nvim
  -- Put this at the end after all plugins
  if PACKER_BOOTSTRAP then
    require("packer").sync()
  end
end)

```

### プラグインの設定

アイコン表示のためのプラグインはこれでインストールされました。
続いて次の設定を設定ファイルに記載します。

```lua
-- Nerdfont を使う
vim.cmd('let g:fern#renderer="nerdfont"')

-- アイコンに色をつける
vim.cmd([[
  augroup my-glyph-palette
    autocmd! *
    autocmd FileType fern call glyph_palette#apply()
    autocmd FileType nerdtree,startify call glyph_palette#apply()
  augroup END
]])
```

## 設定後の表示

これで

1. フォントのインストール
2. アイコン表示のためのプラグインのインストール
3. プラグインの設定

ができました。

貼付したキャプチャは [こちら](https://github.com/ksh-fthr/react-and-echo-work) や [こちら](https://github.com/ksh-fthr/streamlit-work) のリポジトリを Neovim で開いたものです。

![file-icon-view01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/02b4db5d-b4ee-06d2-6f1f-7e4ad3e449c1.png)
![file-icon-view02.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/93af7942-2f66-a8f9-dad3-31087bc1a836.png)

Go や JavaScript, Python, その他の言語や環境のファイルでアイコン表示されているのがお分かりいただけるかと思います。


## 参考

- [VimをVSCodeライクにする](https://qiita.com/youichiro/items/b4748b3e96106d25c5bc)
- [NeovimのためのLua入門 Lua基礎編](https://zenn.dev/slin/articles/2020-10-19-neovim-lua1)

