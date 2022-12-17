["前のエントリ"](http://qiita.com/ksh-fthr/items/5bfc3583e85ed124f4bb) で モジュールを prepend すると, オーバーライドがおきると書いた.
ではモジュールを prepend したとき, オーバーライドしたメソッドのなかで super を実行するとどうなるか.

## 確認環境
* Ruby v2.3.7p456

## prepend 時の super の挙動

```ruby:when_prepend_behavior_of_prepend.rb
# モジュールのメソッドで super を実行する
module M
  def hoge
    puts "hoge is defined by module"
    super
  end
end

# モジュールを prepend するクラス
class ClsPrependModule
  prepend M
  def hoge
    puts "hoge is defined by ClsPrependModule"
  end
end
```

## 挙動の確認
```ruby:verify_behavior1.rb
ClsPrependModule.new.hoge # :=> hoge is defined by module
                          # :=> hoge is defined by ClsPrependModule
```

## 結論
prepend されたモジュールの, オーバーライドすることになるメソッドで super を実行すると, オーバーライドされる側のメソッド, つまり prepend したクラスの同名メソッドが実行されることがわかった.

## おまけ
じゃあ, super で実行されるメソッド(つまり prepend したクラスのメソッド)で, さらに super を実行するとどうなるか.

```ruby:verify_exec_super_in_prepended_class.rb
# モジュールのメソッドで super を実行する
module M
  def hoge
    puts "hoge is defined by module"
    super
  end
end

# モジュールを prepend するクラス
class ClsPrependModule
  prepend M
  def hoge
    puts "hoge is defined by ClsPrependModule"
    super
  end
end
```

## 挙動の確認
```ruby:verify_behavior2.rb
# 実行
ClsPrependModule.new.hoge

# 結果
hoge is defined by module
hoge is defined by ClsPrependModule
NoMethodError: super: no superclass method `hoge' for #<ClsPrependModule:0x00007fb0a3a8ee58>
	from (irb):15:in `hoge'
	from (irb):8:in `hoge'
	from (irb):18
	from /usr/bin/irb:11:in `<main>'
```

1. モジュールのメソッドがコールされ実行される
1. super で prepend したクラスのメソッドがコールされ実行される
1. prepend したクラスの上位クラス(ここでは Objectクラス)では「hoge()メソッド」が定義されていないので NoMethodError が発生した
