# はじめに
[こちら](https://qiita.com/ksh-fthr/items/29db7c5c7268ee1802c5) の記事で 「Flask で CSV データを作成し、それを REST-API のレスポンスとして返却する」 やり方について触れた。
だがこれだと複数のデータをCSVファイルとして送るのにちょっと手間がかかる( 例えばファイル名とCSVデータの dict をリストにして返却するとか )し、API を利用する側もリストをループで回して処理しないといけない。

流石にそれは手間なので、複数のCSVファイルを ZIP で固めて REST-API で返却する方法を実現してみよう、というのが本記事の骨子となる。



# 環境

|                      | バージョン | 確認方法           |
| -------------------- | ---------- | ------------------ |
| Windows10 Home 64bit |            |                    |
| Python               | 3.7.2      | $ python --version |
| Flask                | 1.0.2      | $ flask --version  |



# やりたいこと

改めてやりたいことを。

- 複数の CSV ファイルを ZIP で固めて REST-API のレスポンスで返却する。ただし ZIP そのものを返却するのではなく JSON 形式のフォーマットで返却する
- その際、ZIPファイルをそのまま設定することはできないので base64 エンコードした上で文字列化したものを設定する

```javascript
{
  'fileName': string, // ファイル名
  'zip': string       // ZIP ファイルのデータを base64文字列 にコンバートしたもの
}
```

- 上述の通り、基本的なスタンスとして 「CSV ファイルを固めた ZIP ファイルをレスポンスで返却する」 というのがあるので、恒常的にファイルを保持しない



# 実装

以下のクラスで

-  CSVファイルの出力
- ZIP ファイルの出力
- base64 エンコードして文字列化
- 出力したファイルの削除

を行っている。

```python
from flask import Flask
from flask_restful import Resource
from datetime import datetime, timedelta
import csv
from io import StringIO
import zipfile
import base64
import os

TMP_PATH = './tmp'


class Zip(Resource):
  def get(self) -> dict:
    """CSVファイルを複数作ってZIPに固めて返却する
    Returns:
      Response -- レスポンスオブジェクト
    Description:
      CSVファイルの出力を行った上でZIPに固めて返却する
      ZIPは base64エンコードした上で文字列化する
    """
    # CSVファイル出力のための準備
    os.makedirs(TMP_PATH, exist_ok=True)

    res: dict = self.__create_zip_file()

    # 後始末. 作成した CSV ファイルや ZIP ファイルを削除する
    # CSV生成処理である `create_csv_monthly` の中でやると zip ファイルが掴まれたままで
    # `PermissionError` が発生するので、仕方なくメソッドを抜けたあとに後始末を行う
    self.__delete_files(TMP_PATH)

    return res

  def __create_zip_file(self) -> dict:
    """[summary]
    Returns:
      dict -- ファイル名と base64文字列化したZIPファイルのデータをセットした dict
    """
    #
    #  サンプルコードなのでヘッダもデータも各ファイルで使いまわす
    #

    # ヘッダレコードとボディレコードを作る
    header_record = [
        '名前', '年齢', '住所', '電話番号', '備考'
    ]
    body_record = [
        'ほげ', '99歳', 'ほげ県ほげほげ市', '999-9999-9999', ''
    ]

    # CSVファイル名
    # ファイル名のフォーマットは ${STR}_${STR}_${YYYYMMDD}. とし、${STR} は任意の文字列が入る
    # ${YYYYMMDD} には西暦での年月日が入る
    date_time = datetime.now().strftime('%Y%m%d')
    output_path1 = '{}_{}_{}.csv'.format('好きな', '文字1', date_time)
    output_path2 = '{}_{}_{}.csv'.format('好きな', '文字2', date_time)
    output_path3 = '{}_{}_{}.csv'.format('好きな', '文字3', date_time)

    # CSVファイルを作成する
    with open(self.__make_file_path(TMP_PATH, output_path1), 'w') as f1:
      writer = csv.writer(f1, quotechar='"', quoting=csv.QUOTE_ALL, lineterminator="\n")
      writer.writerow(header_record)
      writer.writerow(body_record)

    with open(self.__make_file_path(TMP_PATH, output_path2), 'w') as f2:
      writer = csv.writer(f2, quotechar='"', quoting=csv.QUOTE_ALL, lineterminator="\n")
      writer.writerow(header_record)
      writer.writerow(body_record)

    with open(self.__make_file_path(TMP_PATH, output_path3), 'w') as f3:
      writer = csv.writer(f3, quotechar='"', quoting=csv.QUOTE_ALL, lineterminator="\n")
      writer.writerow(header_record)
      writer.writerow(body_record)

    # ZIPファイル名の例
    # テスト店_月次集計_20190520.zip
    file_name_zip = '{}_{}_{}.zip'.format('ZIP', 'ファイル', date_time)

    # ZIP ファイルを生成
    with zipfile.ZipFile(self.__make_file_path(TMP_PATH, file_name_zip), 'w', compression=zipfile.ZIP_DEFLATED) as new_zip:
      new_zip.write(self.__make_file_path(TMP_PATH, output_path1), arcname=output_path1)
      new_zip.write(self.__make_file_path(TMP_PATH, output_path2), arcname=output_path2)
      new_zip.write(self.__make_file_path(TMP_PATH, output_path3), arcname=output_path3)

    # ZIP ファイルを base64 エンコード
    # ただしそのままだとバイナリなので JSON 形式でレスポンスを返せない
    # -> decode することで文字列として扱うことで JSON 形式に対応させる
    #
    # つまり
    #   binary ファイル読み込み -> base64encode -> decode で文字列化
    # している
    fzip = open(self.__make_file_path(TMP_PATH, file_name_zip), 'br')
    fzip_64encoded = base64.b64encode(fzip.read())
    res: dict = {
        'fileName': file_name_zip,
        'zip': fzip_64encoded.decode('utf-8')
    }

    return res

  def __make_file_path(self, dir_path: str, file_name: str) -> str:
    """ファイルパスを作成する
    Arguments:
      dir_path {str} -- ディレクトリパス
      file_name {str} -- ファイル名
    Returns:
      str -- ファイル名まで含めたパス
    """
    return '{}/{}'.format(dir_path, file_name)

  def __delete_files(self, dir_path: str) -> None:
    """CSVファイル出力後にできたファイルを削除する
    Arguments:
      dir_path {str} -- ディレクトリパス
    Returns:
      None -- なし
    """
    files: list = os.listdir(dir_path)
    for file in files:
      try:
        # tmp ファイルの下は zip と csv しかないのでディレクトリのケアは必要ない
        target = self.__make_file_path(dir_path, file)
        os.remove(target)
      except:
        # 本来こないハズのルート
        # os.remove() ではディレクトリの削除で例外(`PermissionError`)が発生するが
        # まあ発生しても 数kb 程度のゴミが残るだけなので放っておく
        continue

    return None
```

以下、ポイントについて説明を。

## CSVファイルの出力

まずは ZIP で固めるための CSV ファイルの出力から。

### 出力先ディレクトリを作成する

```python
os.makedirs(TMP_PATH, exist_ok=True)
```

予め出力先ディレクトリを作っておいても良いが、念の為ディレクトリがなかったときのケアとして出力先ディレクトリ作成する。このとき `exist_ok=True` を指定することで **無ければ作る** 動きとなる。
詳細は [公式ドキュメント](#https://docs.python.org/ja/3/library/os.html#os.makedirs) を。

### CSVファイルを出力する

```python
def __create_zip_file(self) -> dict:
  #
  # * ヘッダコメントは省略
  # * CSVヘッダ、ボディ、ファイル名の作成についてもここでは省略
  #
  
  # CSVファイルを作成する
  with open(self.__make_file_path(TMP_PATH, output_path1), 'w') as f1:
    writer = csv.writer(f1, quotechar='"', quoting=csv.QUOTE_ALL, lineterminator="\n")
    writer.writerow(header_record)
    writer.writerow(body_record)

  #
  # あとはファイル名を変えて同じ処理を行うだけなので省略
  #
```

とくに説明することも無いのだけれど、`with open(ファイル名, モード) as f` で CSV ファイル出力処理を行っている。
このときブロックの中で

- writer オブジェクトの生成( 生成時のオプションは下記 )
  - クオートの記号は `"` とする
  - 全てのフィールドをクオートする
  - 改行は「LF(`\n`)」とするこ
- ヘッダの出力
- ボディの出力

を行っている。こちらも詳細は [公式ドキュメント](#https://docs.python.org/ja/3/library/csv.html) を。

## ZIP ファイルの出力

```python
 # ZIP ファイルを生成
  with zipfile.ZipFile(self.__make_file_path(TMP_PATH, file_name_zip), 'w', compression=zipfile.ZIP_DEFLATED) as new_zip:
    new_zip.write(self.__make_file_path(TMP_PATH, output_path1), arcname=output_path1)
    new_zip.write(self.__make_file_path(TMP_PATH, output_path2), arcname=output_path2)
    new_zip.write(self.__make_file_path(TMP_PATH, output_path3), arcname=output_path3)

```

先に挙げた CSV ファイルの出力と形式は似ているが、扱っているモノが違う。こちらは `zipfile` を使って出力を行う。
ここでは圧縮形式に `ZIP_DEFLATED` を指定している。

ブロックの中について少々説明を。

- これより前の処理で出力していた各CSVファイルを指定することで、それらを一つの ZIP に固めている
- `write` メソッドの第2引数にある `arcname` には ZIP で固める際のファイル名を指定する。これを指定しない場合、第1引数である `filename` がそのまま設定される

公式ドキュメントは [こちら](https://docs.python.org/ja/3/library/zipfile.html)。

## base64 エンコードして文字列化

```python
    # ZIP ファイルを base64 エンコード
    # ただしそのままだとバイナリなので JSON 形式でレスポンスを返せない
    # -> decode することで文字列として扱うことで JSON 形式に対応させる
    #
    # つまり
    #   binary ファイル読み込み -> base64encode -> decode で文字列化
    # している
    fzip = open(self.__make_file_path(TMP_PATH, file_name_zip), 'br')
    fzip_64encoded = base64.b64encode(fzip.read())
    res: dict = {
        'fileName': file_name_zip,
        'zip': fzip_64encoded.decode('utf-8')
    }
```

ここはコメントに記載してある通り。

1. 先の処理で出力した ZIP ファイルをバイナリモードで読み込み
2. base64 エンコードし
3. 更にデコードして文字列化

している。
公式ドキュメントは [こちら](https://docs.python.org/ja/3/library/base64.html)。

## 出力したファイルの削除

```python
  def __delete_files(self, dir_path: str) -> None:
    """CSVファイル出力後にできたファイルを削除する
    Arguments:
      dir_path {str} -- ディレクトリパス
    Returns:
      None -- なし
    """
    files: list = os.listdir(dir_path)
    for file in files:
      try:
        # tmp ファイルの下は zip と csv しかないのでディレクトリのケアは必要ない
        target = self.__make_file_path(dir_path, file)
        os.remove(target)
      except:
        # 本来こないハズのルート
        # os.remove() ではディレクトリの削除で例外(`PermissionError`)が発生するが
        # まあ発生しても 数kb 程度のゴミが残るだけなので放っておく
        continue
```

まず `listdir` で指定したディレクトリにあるファイルのリストを取得する。
次いで取得したファイルリストに対してループして一つずつ削除していく。

コメントにも記載しているが、今回挙げた処理では、ここに入ってくるケースだと中にはファイルしか無い筈なのでディレクトリのケアは行っていない。
同じくコメントに記載しているが `os.remove()` ではディレクトリがあると例外を発生させるので注意が必要である。
( `except` のブロックではコメントの内容が少々乱暴ですが、サンプルコードということでご容赦ください )

で、最後にもう少し。
このメソッドは大元の `get` メソッドから `__create_zip_file` の処理が **抜けたあと** に実行されている。
`get` メソッド中のコメントでも触れているが、`__create_zip_file` メソッドでファイル削除を行うと ZIP ファイルの削除の段で `PermissionError` が発生したためこのような実装とした。

ファイル削除やディレクトリ削除については以下のサイトが参考になった。

- [Pythonでファイル・ディレクトリを削除するos.remove, shutil.rmtreeなど](https://note.nkmk.me/python-os-remove-rmdir-removedirs-shutil-rmtree/)



# まとめにかえて

記事にするというところで細々と書いてきたが、一度作ってしまうと定型化できそうな感じ。
ユースケースを想定して色々な用途に応じられるよううまく抽象化すればライブラリ化することもできそう。



# ソースコード

今回の記事で作成したコードは [こちら](https://github.com/ksh-fthr/flask-work/tree/feat/zip)  にアップしてあるのでご参考まで。

   

# 参考

- [雑多なオペレーティングシステムインタフェース](https://docs.python.org/ja/3/library/os.html)
- [CSV ファイルの読み書き](https://docs.python.org/ja/3/library/csv.html)
- [ZIP アーカイブの処理](https://docs.python.org/ja/3/library/zipfile.html)
- [Base16, Base32, Base64, Base85 データの符号化](https://docs.python.org/ja/3/library/base64.html)
- [Pythonでファイル・ディレクトリを削除するos.remove, shutil.rmtreeなど](https://note.nkmk.me/python-os-remove-rmdir-removedirs-shutil-rmtree/)

