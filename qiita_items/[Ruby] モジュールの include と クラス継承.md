モジュールの include とクラスの 継承 について.
モジュールの include とクラスの 継承 は, それを行うタイミングによって include と 継承 の挙動が変わるのでメモ.

## 確認環境
* Ruby v2.3.7p456

## パターン[1]: 下位クラスで include と 継承 を行った場合

まずはモジュールと親クラスで同名メソッドが定義されていて, それらを inlude, 継承 した場合の動きを確認する.

## モジュール定義とクラス定義
```ruby:define_module_define_class.rb
# まずはモジュール定義
module Mdl
  def hoge
    puts "hoge is defined by Mdl"
  end
end

# Mdl のメソッドと同名メソッドを定義したクラス
class Cls1
  def hoge
    puts "hoge is defined by Cls1"
  end
end

# Mdl を include し, かつ Cls1 を継承するクラス
class Cls2 < Cls1
  include Mdl
end
```

## 継承関係の確認
```ruby:verify_inheritance.rb
Cls2.ancestors # :=> [Cls2, Mdl, Cls1, Object, Kernel, BasicObject]
```

## 挙動の確認
```ruby:verify_behavior.rb
# hoge は Mdl, Cls1 どちらのメソッドが実行されるか
Cls2.new.hoge # :=> hoge is defined by Mdl
```

## パターン[2]: 上位クラスで include を行った場合
次にモジュールと親クラスで同名メソッドが定義されていて, 親クラスでモジュールを inlude した場合の動きを確認する.

## モジュール定義とクラス定義
```ruby:define_module_define_class2.rb
# まずはモジュール定義
module Mdl
  def hoge
    puts "hoge is defined by Mdl"
  end
end

# Mdl を include し, かつ Mdl のメソッドと同名メソッドを定義したクラス
class Cls1
  include Mdl
  def hoge
    puts "hoge is defined by Cls1"
  end
end

# Cls1 を継承するクラス
class Cls2 < Cls1
end
```

## 継承関係の確認
```ruby:verify_inheritance2.rb
Cls2.ancestors # :=> [Cls2, Cls1, Mdl, Object, Kernel, BasicObject]
```

## 挙動の確認
```ruby:verify_behavior2.rb
# hoge は Mdl, Cls1 どちらのメソッドが実行されるか
Cls2.new.hoge # :=> hoge is defined by Cls1
```

## 結論
Cls2.ancestors を行ったときの結果がすべて.
つまり

* パターン[1]
 * Cls2.ancestors # :⇒ [Cls2, Mdl, Cls1, Object, Kernel, BasicObject]
 * Mdl が継承チェーン上, Cls2 の直上に来ているので, Mdl.hoge が最初に見つかり実行される

* パターン[2]
 * Cls2.ancestors # :⇒ [Cls2, Cls1, Mdl, Object, Kernel, BasicObject]
 * Cls1 が継承チェーン上, Cls2 の直上に来ているので, Cls1.hoge が最初に見つかり実行される
