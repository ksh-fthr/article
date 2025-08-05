## はじめに

圧縮ファイルを暗号化する際にコマンドラインから作業することが稀に発生するのですが、そのたびに調べるのも手間なので備忘録として記事に起こしました。
似たような状況の方々の参考になれば幸いです。

## `zip` コマンドで簡易的に暗号化( ZipCrypto )

### zip コマンドの形式

```bash
zip -e archive.zip file1 file2
```

### zip コマンドのオプション説明

- `-e` オプション: 暗号化を行う
- `archive.zip`: 作成される ZIP ファイル名
- `file1 file2`: 圧縮したいファイル( スペース区切りで複数指定可 )

### zip コマンドの実行例

```bash
zip -e archive.zip resume1.pdf resume2.pdf
```

パスワードの **入力と確認** が求められます。

## `7z`( 7-Zip )を使用して AES-256 で暗号化( より強固 )

macOS では `7z` コマンドの代わりに `7zz` を使います( Homebrew でインストール可能 )。

### 7zz のインストール方法( macOS )

```bash
brew install p7zip
```

インストール後に `7zz` コマンドが利用可能になります。

### 7zz コマンドの形式

```bash
7zz a -tzip -p -mem=AES256 archive.zip file1 file2
```

### 7zz コマンドのオプション説明

- `a`: アーカイブ作成
- `-tzip`: ZIP形式で出力
- `-p`: パスワード指定
- `-mem=AES256`: AES-256 による強力な暗号化
- `archive.zip`: 作成される ZIP ファイル名
- `file1 file2`: 圧縮したいファイル( スペース区切りで複数指定可 )

### 7zz コマンドの実行例

```bash
7zz a -tzip -p -mem=AES256 archive.zip resume1.pdf resume2.pdf
```

このコマンドでは、実行時に安全な対話形式でパスワードが求められます。

### パスワードを直接指定する形式( 非推奨 )

```bash
7zz a -tzip -phogehoge -mem=AES256 archive.zip resume1.pdf resume2.pdf
```

`-p` のあとに直接パスワードを記述すると、コマンド履歴などに残ってしまうためセキュリティ上好ましくありません。
少しでも安全性を高めるために、対話形式( `-p` のみ )での入力をおすすめします。

## 応用テクニック

### フォルダを丸ごと圧縮する場合

**zip を使用する場合**

```bash
zip -r -e archive.zip target_folder
```

**7zz を使用する場合**

```bash
7zz a -tzip -p -mem=AES256 archive.zip target_folder
```

いずれもフォルダ構造を保持したまま、フォルダを暗号化して圧縮できます。

## セキュリティ上の注意点

- `zip -e` による暗号化は ZipCrypto 方式で、暗号強度は高くありません。
- より強固な暗号化を求める場合は、AES-256 を使用できる `7z` の利用が推奨されます。

## 参考

**zip**
- [zipコマンドの公式マニュアル( 英語 )](https://linux.die.net/man/1/zip)

**7zip**
- [7-Zip公式ドキュメント](https://7-zip.opensource.jp/)
- [7-Zipのコマンドライン操作](https://cgbeginner.net/7-zip/)
- [コマンドでZIPや7zにパスワードを付ける](https://7-zip.opensource.jp/howto/dos-command-password.html)
