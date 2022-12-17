# はじめに

Python3 で指定した日がその月の第N週かを判断したかった。



# 環境

|                      | バージョン | 確認方法           |
| -------------------- | ---------- | ------------------ |
| Windows10 Home 64bit |            |                    |
| Python               | 3.7.2      | $ python --version |



# 結論

calendar モジュールを利用することで対応できた。



# 実装

```python
from datetime import datetime
from calendar import Calendar

def __get_week_number(target_date: datetime) -> int:
  """指定された日付からその月の「第N週」かを取得する

  Arguments:
      target_date {datetime} -- 第N週を取得する対象の日付情報

  Returns:
      int -- 第N週の`N`
  """
  # カレンダーを日曜日始まりで作成する( firstweekday=6 は日曜日始まりを指定 )
  cl = Calendar(firstweekday=6)

  # `monthdays2calendar()` を利用して第N週かを判定する。
  #
  # 当該メソッドからは (日付, 曜日) のタプルが週単位で要素が入っているリストが返却されるので、
  # それをもとに処置対象日付が第N週かを判断する
  # なお各曜日の値は次のとおり
  #
  # 6: 日曜日
  # 0: 月曜日
  # 1: 火曜日
  # 2: 水曜日
  # 3: 木曜日
  # 4: 金曜日
  # 5: 土曜日
  #
  # 例) 2019年5月だったら以下のリストが返却される
  # [
  #   [(0, 6), (0, 0), (0, 1), (1, 2), (2, 3), (3, 4), (4, 5)],
  #   [(5, 6), (6, 0), (7, 1), (8, 2), (9, 3), (10, 4), (11, 5)],
  #   [(12, 6), (13, 0), (14, 1), (15, 2), (16, 3), (17, 4), (18, 5)],
  #   [(19, 6), (20, 0), (21, 1), (22, 2), (23, 3), (24, 4), (25, 5)],
  #   [(26, 6), (27, 0), (28, 1), (29, 2), (30, 3), (31, 4), (0, 5)]
  # ]
  #
  # 前後の月の日付は「0」で設定される

  # ここからが実際に第N週かを取得する処理
  month_cl = cl.monthdays2calendar(target_date.year, target_date.month)
  week_num = 1
  for week in month_cl:
    for day in week:
      if day[0] == target_date.day:
        return week_num
    week_num += 1

  # ここには来ないハズ
  # => 本当なら例外吐くのが良いと思う
  # => もしくはコール元で判定して例外吐くとか
  return 0

date = datetime(2019,6,3)
week_num = __get_week_number(date)

# 「指定した日( 6/3 ) は 6 月の第 2 週です」が出力される
print('指定した日( {}/{} ) は {} 月の第 {} 週です'.format(date.month, date.day, date.month, week_num))

```

ではポイントについて簡単な説明を。



## calendar モジュールの import

ますは calendar モジュールを import するところから。
標準ライブラリなので別途インストールすることなく import 文で取り込める。

```python
from calendar import Calendar
```



## 日曜日始まりとする

指定した日付が第N週であるのかを判断するにあたり、**週の初まりを日曜日** としてしたかった。
これは次のコードで実現できる。

```python
cl = Calendar(firstweekday=6)
```



## 一ヶ月が一週間単位でまとまっているリストを取得する

これには `monthdays2calendar()` を使って実現した。

```python
month_cl = cl.monthdays2calendar(target_date.year, target_date.month)	
```

詳細はコード中のコメントを参照いただくとして、上記の式で「 **指定した月が一週間単位でまとまったリスト** 」が取得できる。
で、各週のリストの要素は `(日付, 曜日(数字))` のタプルなので、これを利用して指定した日が第N週なのかを判定できる。



## 第N週かを判定する

あとはこれまでに取得した情報を元に判定処理を実装するだけ。

```python
week_num = 1
for week in month_cl:
  for day in week:
    if day[0] == target_date.day:
      return week_num
  week_num += 1
```

1. 週の初めは1週からなので `week_num = 1` として
2. 外のループで一ヶ月のリスト(週単位) を回す。このとき `week_num ` のインクリメントを忘れずに行う
3. 中のループで一週間のリスト(日付と曜日のタプル) を回す
4. タプルの日付と指定された日付が一致したら `week_num ` を返り値に復帰する



## 例外処理

コードをご覧になればお分かりのとおり、例外処理をいれていない。
もし似たようなケースでご利用の際は例外処理はご自身でお願いします。

```python
# ここには来ないハズ
# => 本当なら例外吐くのが良いと思う
# => もしくはコール元で判定して例外吐くとか
return 0
```



# まとめ

「Python 第N週」とかでググると `datetime` の `isocalendar()` を使った方法はいくらでも出てくるのだけれど、これだと **その年の1月1日から数えて第N週か** になるので今回の要件に合わなかった。

他にやり方あるかもしれないが、自分が探した限りでは今回の要件である「 **指定した日がその月の第N週か** 」を判定する記事が見つからなかったので足掻いてみた。



他により良いやり方をご存じの方、もしくは発見した方はご教示いただけると嬉しいです。



# 参考

- [【python】calendarモジュールの使い方](https://www.haya-programming.com/entry/2018/04/26/235451)
- [Pythonで週番号を計算する](https://water2litter.net/rum/post/python_datetime_isoweekday/)
- [Python その日が今年の何週目か datetime](http://tama-game.hateblo.jp/entry/2017/03/18/111140)
