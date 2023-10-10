備忘録として。
掲題のケースですっきり書けるやり方をメモしておく。
( 他に良い方法があればご指摘いただけますと幸いです )

# 確認環境
少し古いが下記の環境で確認。

|         | バージョン | 確認方法    |
| ------- | -------- | ---------- |
| python3 | v3.7.12  | python3 -V |


# 単純なリストの場合
## 単一の条件にマッチ
```python
hoge_list = [1,2,3,4,5,6,7,8,9,10]

# hoge_list の値が 5 のものを抽出して新しいリストを作る
filtered_hoge_list = [ hoge for hoge in hoge_list if hoge == 5]

# 出力して確認
# 条件に指定した値がリストの要素になっていることが確認できる
filtered_hoge_list
[5]
```


## 複数の条件にマッチ

```python
hoge_list = [1,2,3,4,5,6,7,8,9,10]

# hoge_list の値が [5, 6, 7] のものを抽出して新しいリストを作る
filtered_hoge_list = [ hoge for hoge in hoge_list if hoge in (5,6,7)]

# 出力して確認
# 条件に指定した値がリストの要素になっていることが確認できる
filtered_hoge_list
[5, 6, 7]
```

# dictionary をリストにしている場合
## 単一の条件にマッチ
```python
foo_dict_list = [
    { 'id': 1, 'summary': 'aaaaaaaa' },
    { 'id': 2, 'summary': 'bbbbbbbb' },
    { 'id': 3, 'summary': 'cccccccc' },
    { 'id': 4, 'summary': '44444444' },
    { 'id': 5, 'summary': 'eeeeeeee' }
]

# foo_dict_list の id が 4 のものを抽出して新しいリストを作る
filtered_foo_dict_list = [foo for foo in foo_dict_list if foo['id'] == 4]

# 出力して確認
# 条件に指定した値がリストの要素になっていることが確認できる
filtered_foo_dict_list
[{'id': 4, 'summary': '44444444'}]
```

## 複数の条件にマッチ

```python
foo_dict_list = [
    { 'id': 1, 'summary': 'aaaaaaaa' },
    { 'id': 2, 'summary': 'bbbbbbbb' },
    { 'id': 3, 'summary': 'cccccccc' },
    { 'id': 4, 'summary': '44444444' },
    { 'id': 5, 'summary': 'eeeeeeee' }
]

# foo_dict_list の id が [1,2,3] のものを抽出して新しいリストを作る
filtered_foo_dict_list = [foo for foo in foo_dict_list if foo['id'] in (1,2,3)]

# 出力して確認
# 条件に指定した値がリストの要素になっていることが確認できる
filtered_foo_dict_list
[{'id': 1, 'summary': 'aaaaaaaa'}, {'id': 2, 'summary': 'bbbbbbbb'}, {'id': 3, 'summary': 'cccccccc'}]
```

# 他のやり方
@shiracamus 様から [コメント](https://qiita.com/ksh-fthr/items/6c1d923f01cc2ca43693#comment-50aa6956f7b65090d1e7) で filter 関数を使った方法についてご提示頂きました。

# 単純なリストの場合
## 単一の条件にマッチ

```python
hoge_list = [1,2,3,4,5,6,7,8,9,10]

# パターン-1
filtered_hoge_list = [*filter(5 .__eq__, hoge_list)]

# 出力して確認
# 条件に指定した値がリストの要素になっていることが確認できる
filtered_hoge_list
[5]

# パターン2, こちらは lambda 関数と併用してフィルタリング
filtered_hoge_list = [*filter(lambda hoge: hoge == 5, hoge_list)]

# 出力して確認
# 条件に指定した値がリストの要素になっていることが確認できる
filtered_hoge_list
[5]
```

## 複数の条件にマッチ

```python
hoge_list = [1,2,3,4,5,6,7,8,9,10]

# パターン-1
filtered_hoge_list = [*filter((5,6,7).__contains__, hoge_list)]

# 出力して確認
# 条件に指定した値がリストの要素になっていることが確認できる
filtered_hoge_list
[5, 6, 7]

# パターン2, こちらは lambda 関数と併用してフィルタリング
filtered_hoge_list = [*filter(lambda hoge: hoge in (5,6,7), hoge_list)]

# 出力して確認
# 条件に指定した値がリストの要素になっていることが確認できる
filtered_hoge_list
[5, 6, 7]
```

# dictionary をリストにしている場合
## 単一の条件にマッチ

```python
foo_dict_list = [
    { 'id': 1, 'summary': 'aaaaaaaa' },
    { 'id': 2, 'summary': 'bbbbbbbb' },
    { 'id': 3, 'summary': 'cccccccc' },
    { 'id': 4, 'summary': '44444444' },
    { 'id': 5, 'summary': 'eeeeeeee' }
]

# lambda 関数と併用してフィルタリング
filtered_foo_dict_list = [*filter(lambda foo: foo["id"] == 4, foo_dict_list)]

# 出力して確認
# 条件に指定した値がリストの要素になっていることが確認できる
filtered_foo_dict_list
[{'id': 4, 'summary': '44444444'}]
```

## 複数の条件にマッチ

```python
foo_dict_list = [
    { 'id': 1, 'summary': 'aaaaaaaa' },
    { 'id': 2, 'summary': 'bbbbbbbb' },
    { 'id': 3, 'summary': 'cccccccc' },
    { 'id': 4, 'summary': '44444444' },
    { 'id': 5, 'summary': 'eeeeeeee' }
]

# lambda 関数と併用してフィルタリング
filtered_foo_dict_list = [*filter(lambda foo: foo["id"] in (1,2,3), foo_dict_list)]

# 出力して確認
# 条件に指定した値がリストの要素になっていることが確認できる
filtered_foo_dict_list
[{'id': 1, 'summary': 'aaaaaaaa'}, {'id': 2, 'summary': 'bbbbbbbb'}, {'id': 3, 'summary': 'cccccccc'}]
```
