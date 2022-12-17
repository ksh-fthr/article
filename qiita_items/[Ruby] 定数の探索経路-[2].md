**定数が自前の親クラスやモジュールでも定義されていない** 場合の動きを確認する.
具体的には クラスメソッドである **const_missing** の動きを見る.

なおモジュールの include, prepend については ["前のエントリ"](http://qiita.com/ksh-fthr/items/c6ec6c378ab339bc7cd1) で見たので本エントリでは確認しない.

## 確認環境
* Ruby v2.0.0p648
* Ruby v2.3.7p456


## [1]定数, const_missing が未定義の場合
```ruby:undefine_const_missing.rb
module Mdl
end

class Cls1
  include Mdl
end

class Cls2 < Cls1
  def hoge
    puts "CONST_VAR: #{CONST_VAR}"
  end
end

# 継承チェーン
Cls2.ancestors # :=> [Cls2, Cls1, Mdl, Object, Kernel, BasicObject]

# エラー発生
Cls2.new.hoge # :=> NameError: uninitialized constant Cls2::CONST_VAR
```

## [2]const_missing をモジュールと親クラスで定義し, 子クラスでも定義した場合
```ruby:define_const_missing_module_parent_child.rb
module Mdl
  def Mdl.const_missing(id)
    puts "Mdl#const_missing"
  end
end

class Cls1
  include Mdl
  def Cls1.const_missing(id)
    puts "Cls1#const_missing"
  end
end

class Cls2 < Cls1
  def hoge
    puts "CONST_VAR: #{CONST_VAR}"
  end
  def Cls2.const_missing(id)
    puts "Cls2#const_missing"
  end
end

# 継承チェーン
Cls2.ancestors # :=> [Cls2, Cls1, Mdl, Object, Kernel, BasicObject]

# Cls2 で定義した cons_missing が実行される
Cls2.new.hoge # :=> Cls2#const_missing
```

## [3]const_missing をモジュールと親クラスで定義し, 子クラスでは定義しない
```ruby:define_const_missing_module_parent.rb
module Mdl
  def Mdl.const_missing(id)
    puts "Mdl#const_missing"
  end
end

class Cls1
  include Mdl
  def Cls1.const_missing(id)
    puts "Cls1#const_missing"
  end
end

class Cls2 < Cls1
  def hoge
    puts "CONST_VAR: #{CONST_VAR}"
  end
end

# 継承チェーン
Cls2.ancestors # :=> [Cls2, Cls1, Mdl, Object, Kernel, BasicObject]

# Cls1 で定義した cons_missing が実行される
Cls2.new.hoge # :=> Cls1#const_missing
```

## [4]const_missing を Classクラスで定義し, モジュール, 親クラス, 子クラスでもインスタンスメソッドで定義する
```ruby:define_const_missing_class_module_parent_child_instancemethod.rb
class Class
  def const_missing(id)
    puts "Class#const_missing"
  end
end

module Mdl
  def const_missing(id)
    puts "Mdl#const_missing"
  end
end

class Cls1
  include Mdl
  def const_missing(id)
    puts "Cls1#const_missing"
  end
end

class Cls2 < Cls1
  def hoge
    puts "CONST_VAR: #{CONST_VAR}"
  end
  def const_missing(id)
    puts "Cls2#const_missing"
  end
end

# 継承チェーン
Cls2.ancestors # :=> [Cls2, Cls1, Mdl, Object, Kernel, BasicObject]

# Class で定義した cons_missing が実行される
Cls2.new.hoge # :=> Class#const_missing
```

## [5]const_missing を Classクラスで定義し, Objectクラスでインスタンスメソッドで定義する
```ruby:define_const_missing_class_object_instancemethod.rb
class Class
  def const_missing(id)
    puts "Class#const_missing"
  end
end

class Object
  def const_missing(id)
    puts "Object#const_missing"
  end
end

class Cls2
  def hoge
    puts "CONST_VAR: #{CONST_VAR}"
  end
  def const_missing(id)
    puts "Cls2#const_missing"
  end
end

# 継承チェーン
Cls2.ancestors # :=> [Cls2, Object, Kernel, BasicObject]

# Class で定義した cons_missing が実行される
Cls2.new.hoge # :=> Class#const_missing
```

## [6]const_missing を Classクラスで定義し, Objectクラスでクラスメソッドで定義する
```ruby:define_const_missing_class_object_classmethod.rb
class Class
  def const_missing(id)
    puts "Class#const_missing"
  end
end

class Object
  def Object.const_missing(id)
    puts "Object#const_missing"
  end
end

class Cls2
  def hoge
    puts "CONST_VAR: #{CONST_VAR}"
  end
  def const_missing(id)
    puts "Cls2#const_missing"
  end
end

# 継承チェーン
Cls2.ancestors # :=> [Cls2, Object, Kernel, BasicObject]

# Object で定義した cons_missing が実行される
Cls2.new.hoge # :=> Object#const_missing
```

## 結論
* 定数の探索は継承チェーンを辿って行われる
* 定数が見つからない場合, const_missing が実行される
 * const_missing はクラスメソッドで実装されている必要がある
 * Classクラスのインスタンスメソッドは他のクラスのクラスメソッドとして処理される
