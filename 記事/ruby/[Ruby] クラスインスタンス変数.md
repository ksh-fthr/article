Ruby Gold の学習中, 変数のスコープでつまづいたのでメモ.
つまづいたのは「クラスインスタンス変数」.

## 確認環境
* Ruby v2.3.7p456

## クラスインスタンス変数の確認をするコード

```ruby:class_instance_val.rb
class Cls
  # 変数「@cls_instance_val」を宣言: [cls_instance_val]をセット
  @cls_instance_val = 'cls_instance_val'

  # クラスメソッドから参照(get)
  def Cls.get_class_instance_val
    @cls_instance_val
  end

  # クラスメソッドから参照(set)
  def Cls.set_class_instance_val(val)
    @cls_instance_val = val
  end

  # インスタンスメソッドから参照(get)
  def get_class_instance_val
    @cls_instance_val
  end

  # インスタンスメソッドから参照(set)
  def set_class_instance_val(val)
    @cls_instance_val = val
  end
end

cls = Cls.new
#
# クラスメソッドからアクセス
Cls.get_class_instance_val # :-> 初期値「cls_instance_val」が返却される
#
# インスタンスメソッドからアクセス
cls.get_class_instance_val # :-> nil が返却される
#
# インスタンスメソッドからアクセス
cls.set_class_instance_val('instance_val') # :-> 「instance_val」をセット
#
# インスタンスメソッドからアクセス
cls.get_class_instance_val # :-> 「instance_val」が返却される
#
# ではここでクラスメソッドからアクセスしたらどうなるか
Cls.get_class_instance_val # :-> 初期値「cls_instance_val」が返却される
#
# おなじくここでクラスメソッドから「hogehoge」をセットし, インスタンスメソッドからアクセスしたら
# なにが返却されるか
Cls.set_class_instance_val('hogehoge')
cls.get_class_instance_val # :-> 「instance_val」」が返却される
```

## 結論
* クラスのスコープで定義した変数とメソッドのスコープで定義した変数は「同名でも別の変数」となる
* したがって「クラスメソッドからアクセスした変数」と「インスタンスメソッドからアクセスした変数」は別物である
* クラスのスコープで定義した変数を「クラスインスタンス変数」という
* メソッドのスコープで定義した変数を「インスタンス変数」という

## コメントいただいた内容を追記(2017.07.30)
上記の結論を Ruby のオブジェクトの観点で整理し直すと次のようになります。（@kts_hさんから頂戴したご指摘を反映しました）

* インスタンス変数はインスタンスの中に存在する。
* Cls クラスも Class クラスのインスタンスであるから、その内部にインスタンス変数（クラスインスタンス変数）を持つことができる。
* メソッドは、レシーバとなったインスタンス内のインスタンス変数を操作する。
* インスタンスメソッドのレシーバは、普通のインスタンスなので、普通に、インスタンス内のインスタンス変数を操作する。
* クラスメソッドのレシーバは、クラスなので、操作できるのは、レシーバとなったクラス内のインスタンス変数である。
