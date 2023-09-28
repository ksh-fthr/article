
# 環境
| バージョン           | 確認方法 |                    |
| :------------------- | :------- | ------------------ |
| Windows10 Home 64bit |          |                    |
| Python               | 3.7.2    | $ python --version |

# やりたいこと

以下の要件を満たしたい。

- dict を要素として持つリストがあり、dict の内容はそれぞれ同じ構成をしている
- dict はあるプロパティで識別できる
- 識別可能なプロパティを軸に他のプロパティの合算値( 合計値 )を出したい

# サンプル

次のような例を考える。

* 各データにはプロパティとして「分類」「分野」「価格」があり、「分野」毎の「価格の合算値( 合計値 )」を出す

## 元データ

```json
[
  {
    "分類": "技術書",
    "分野": "Python",
    "価格": 3200
  },
  {
    "分類": "技術書",
    "分野": "Python",
    "価格": 1800
  },
  {
    "分類": "技術書",
    "分野": "JavaScript",
    "価格": 3300
  }
]
```



## 整形後のデータ

次のような形に整形したい。

```json
[
  {
    "分類": "技術書",
    "分野": "Python",
    "価格": 5000
  },
  {
    "分類": "技術書",
    "分野": "JavaScript",
    "価格": 3300
  }  
]
```



# 結論から

次の実装で実現できた。

## スクリプト

```python
# -*- coding: utf-8 -*-

from typing import List

src_list: List[dict] = [
  {
    "分類": "技術書",
    "分野": "Python",
    "価格": 3200
  },
  {
    "分類": "技術書",
    "分野": "Python",
    "価格": 1800
  },
  {
    "分類": "技術書",
    "分野": "JavaScript",
    "価格": 3300
  }
]


def get_target_dict(target_key: str, target_value:str, src_list: List[dict]) -> dict:
  """リスト中から引数の `target_key` と `target_value` に合致する情報を取得する

  Arguments:
    target_key {str} -- 取得対象のプロテパティの Key
    target_value {str} -- 取得対象のプロパティの Value
    src_list {List[dict]} -- 取得対象の情報が入っている可能性のあるリスト

  Returns:
    dict -- `target` に合致する情報

  Description:
    合致する情報がない場合は None を返却する
  """
  # for src in src_list:
  #   for k, v in src.items():
  #     if k == target_key:
  #       if v == target_value:
  #         return src

  # こちらの方がいいかも
  for src in src_list:
    if (target_key in src) and (src[target_key] == target_value):
      return src
  return None

def sum_by(target_key: str, calc_key: str, src_lsit: List[dict]) -> List[dict]:
  """リスト中から引数の `target_key` と `target_value` に合致した合計値を出す

  Arguments:
    target_key {str} -- 合算対象のプロパティの Key
    calc_key {str} -- 計算対象のプロパティの Key
    src_list {List[dict]} -- 取得対象の情報が入っている可能性のあるリスト

  Returns:
    list[dict] -- カテゴリ毎の書籍の合計を出したリスト
  """
  dest_list: List[dict] = []
  dest_dict = {}
  for src in src_lsit:
    dest_dict = get_target_dict(target_key, src[target_key], dest_list)
    # if dest_dict:
    #   for k, v in src.items():
    #     if k == calc_key:
    #       dest_dict[k] += v

    # こちらの方がいいかも
    if dest_dict:
      if calc_key in src:
        dest_dict[calc_key] += src[calc_key]
    else:
      dest_list.append(src)

  return dest_list

# 合算値を出す
total_list: List[dict] = sum_by('分野', '価格', src_list)
print(total_list)
```

## 結果

[サンプルの整形後のデータ](#整形後のデータ) で示したものと同じ結果が得られた。

```python
[
  {
    '分類': '技術書',
    '分野': 'Python',
    '価格': 5000
 	},
  {
    '分類': '技術書',
    '分野': 'JavaScript',
    '価格': 3300
  }
]
```



# ポイントの説明

## sum_by 関数

```python
def sum_by(target_key: str, calc_key: str, src_lsit: List[dict]) -> List[dict]:
  """リスト中から引数の `target_key` と `target_value` に合致した合計値を出す

  Arguments:
    target_key {str} -- 合算対象のプロパティの Key
    calc_key {str} -- 計算対象のプロパティの Key
    src_list {List[dict]} -- 取得対象の情報が入っている可能性のあるリスト

  Returns:
    list[dict] -- カテゴリ毎の書籍の合計を出したリスト
  """
  dest_list: List[dict] = []
  dest_dict = {}
  for src in src_lsit:
    dest_dict = get_target_dict(target_key, src[target_key], dest_list)
    # if dest_dict:
    #   for k, v in src.items():
    #     if k == calc_key:
    #       dest_dict[k] += v

    # こちらの方がいいかも
    if dest_dict:
      if calc_key in src:
        dest_dict[calc_key] += src[calc_key]
    else:
      dest_list.append(src)

  return dest_list
```

### 抽象化したかった

固定のプロパティにのみ対応するのではなく、 **実行時に対応するプロパティを決められる** ように引数で **合算対象** の項目と **計算対象** の項目を指定可能としている。



### 返却用のリストに合算対象の dict がいるか

大元のループ処理自体は元データであるリストに対して行うが、その中で返却するリストに **合算対象の dict がいるかいないか** が大きなポイントとなる。( 合算対象の dict の取得は [get_target_dict 関数](#get_target_dict-関数) で実装している )

で、

- いれば -> 指定された **計算対象のプロパティ** に対して計算する
- いなければ -> 返却するリストのそのまま放り込む

とすることで、 **合算対象がいない単体の dict も漏らさない** ようケアしている。

## get_target_dict 関数

```python
def get_target_dict(target_key: str, target_value:str, src_list: List[dict]) -> dict:
  """リスト中から引数の `target_key` と `target_value` に合致する情報を取得する

  Arguments:
    target_key {str} -- 取得対象のプロテパティの Key
    target_value {str} -- 取得対象のプロパティの Value
    src_list {List[dict]} -- 取得対象の情報が入っている可能性のあるリスト

  Returns:
    dict -- `target` に合致する情報

  Description:
    合致する情報がない場合は None を返却する
  """
  # for src in src_list:
  #   for k, v in src.items():
  #     if k == target_key:
  #       if v == target_value:
  #         return src

  # こちらの方がいいかも
  for src in src_list:
    if (target_key in src) and (src[target_key] == target_value):
      return src
  return None
```

### 抽象化したかった

やりたかったことは [sum_by 関数](#sum_by-関数) と同じ。固定のプロパティにのみ対応するのではなく、 **実行時に対応するプロパティを決められる** ように引数で **取得対象の key** と **取得対象の value** を指定可能としている。



## 蛇足ながら

あくまでサンプルコードということで例外処理や引数に対するバリデーションは省いている。



# まとめにかえて

Python で提供されているライブラリで実現できそうなものが見つからなかったので、本記事で紹介した実装で実現しています。
「もっとこうした方が良い」とか「このライブラリで実現できる」等、ご意見あればよろしくお願いします。
