モジュールを include したときと prepend したときでは継承順序が異なる.
結果, 同じモジュールを include したときと prepend したときでは当然挙動が変わってくる.

## 確認環境
* Ruby v2.3.7p456

## モジュール定義とクラス定義

```ruby:define_module_define_class.rb
# まずはモジュール定義
module M
  def hoge
    puts "hoge is defined by module"
  end
end

# 定義したモジュールを include するクラス
class ClsInludeModule
  include M
  def hoge
    super
    puts "hoge is defined by ClsIncludeModule"
  end
end

# 定義したモジュールを prepend するクラス
class ClsPrependModule
  prepend M
  def hoge
    super
    puts "hoge is defined by ClsPrependModule"
  end
end
```

## 継承関係の確認
```ruby:verify_inheritane.rb
# モジュールを include したクラス
ClsInludeModule.ancestors # :=> [ClsInludeModule, M, Object, Kernel, BasicObject]

# 継承関係の確認(モジュールを prepend したクラス)
ClsPrependModule.ancestors # :=> [M, ClsPrependModule, Object, Kernel, BasicObject]
```

## 挙動の確認
```ruby:verify_behavior.rb
# モジュールを include したクラス
ClsInludeModule.new.hoge # :=> hoge is defined by module
                         # :=> hoge is defined by ClsIncludeModule

# モジュールを prepend したクラス
ClsPrependModule.new.hoge # :=> hoge is defined by module
```

## 結論
* モジュールを include した場合
 * モジュールは include を行ったクラスの 「上」に来る
 * したがって同名メソッドが定義されている場合, super で呼び出した際には モジュールのメソッドが実行される

* モジュールを prepend した場合
 * モジュールは prepend を行ったクラスの 「下」に来る
 * この場合, prepend を行ったクラスのメソッドは prepend したモジュールのもので オーバーライド される
 * したがって prepend を行ったクラスのメソッドは実行されない
