## `zip` コマンドで簡易的に暗号化( ZipCrypto )

### zip コマンドの形式

```bash
zip -e output.zip file1 file2
```

### zip コマンドのオプション説明

- `-e` オプション: 暗号化
- `output.zip`: 作成される ZIP ファイル名
- `file1 file2`: 圧縮したいファイル( スペース区切りで複数指定可 )

### zip コマンドの実行例

```bash
zip -e resume.zip resume1.pdf resume2.pdf
```

パスワードの **入力と確認** が求められます

## `7z`( 7-Zip )を使用して AES-256 で暗号化( より強固 )

macOS では `7z` コマンドの代わりに `7zz` を使います( Homebrew でインストール可能 )。

### 7zz コマンドの形式

```bash
7zz a -tzip -p -mem=AES256 output.zip file1 file2
```

### 7zz コマンドのオプション説明

- `a`: アーカイブ作成
- `-tzip`: ZIP形式
- `-p`: パスワード指定
- `-mem=AES256`: AES-256による強力な暗号化
- `output.zip`: 作成される ZIP ファイル名
- `file1 file2`: 圧縮したいファイル( スペース区切りで複数指定可 )

### 7zz コマンドの実行例

```bash
7zz a -tzip -p -mem=AES256 resume.zip resume1.pdf resume2.pdf
```

このコマンドでは、実行時に安全な対話形式でパスワードが求められます。

### パスワードを直接指定する形式( 非推奨 )

```bash
7zz a -tzip -phogehoge -mem=AES256 resume.zip resume1.pdf resume2.pdf
```

上記のように -p のあとに直接パスワードを記述すると、コマンド履歴などに残ってしまうため セキュリティ上好ましくありません。
少しでも安全性を高めるために、対話形式( `-p` のみ )での入力をおすすめします。

## セキュリティ上の注意点

- zip -e による暗号化は ZipCrypto 方式で、暗号強度は高くありません。
- より強固な暗号化を求める場合は、AES-256 を使用できる 7z の利用が推奨されます。

### 参考

**zip**
- zipコマンドの公式マニュアル( 英語 ):  [https://linux.die.net/man/1/zip](https://linux.die.net/man/1/zip)

**7zip**
- 7-Zip公式ドキュメント: [https://sevenzip.osdn.jp/](https://sevenzip.osdn.jp/)
