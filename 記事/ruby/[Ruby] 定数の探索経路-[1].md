定数の定義がある場合とない場合の動きを確認する.

## 確認環境

* Ruby v2.0.0p648
* Ruby v2.3.7p456

## [1]まずは素直な例. 登場するモジュール, クラスで定数が定義されている
```ruby:const_normal.rb
#
# 登場するモジュール, クラスで定数が定義されている
#
module Mdl
  CONST_VAR = "Mdl#CONST_VAR"
end

class Cls1
  include Mdl
  CONST_VAR = "Cls1#CONST_VAR"
end

class Cls2 < Cls1
  CONST_VAR = "Cls2#CONST_VAR"
end

# 継承チェーン
Cls2.ancestors # :=> [Cls2, Cls1, Mdl, Object, Kernel, BasicObject]

# Cl2 で定義した定数が出力される
Cls2::CONST_VAR # :=> Cls2#CONST_VAR
```

## [2]定数はモジュールと親クラスに定義. 子クラスでは定義しない. 親クラスで include
```ruby:const_module_parent_include_parent.rb
#
# 定数はモジュールと親クラスに定義され, 子クラスでは定義しない
#
module Mdl
  CONST_VAR = "Mdl#CONST_VAR"
end

class Cls1
  include Mdl
  CONST_VAR = "Cls1#CONST_VAR"
end

class Cls2 < Cls1
end

# 継承チェーン
Cls2.ancestors # :=> [Cls2, Cls1, Mdl, Object, Kernel, BasicObject]

# Cls1 で定義した定数が出力される
Cls2::CONST_VAR # :=> Cls1#CONST_VAR
```

## [3]定数はモジュールと親クラスに定義. 子クラスでは定義しない. 子クラスで include
```ruby:const_module_parent_include_child.rb
#
# 定数はモジュールと親クラスに定義され, 子クラスでは定義しない
#
module Mdl
  CONST_VAR = "Mdl#CONST_VAR"
end

class Cls1
  CONST_VAR = "Cls1#CONST_VAR"
end

class Cls2 < Cls1
  include Mdl
end

# 継承チェーン
Cls2.ancestors # :=> [Cls2, Mdl, Cls1, Object, Kernel, BasicObject]

# Mdl で定義した定数が出力される
Cls2::CONST_VAR # :=> Mdl#CONST_VAR
```

## [4]定数はモジュールと親クラスに定義. 子クラスでは定義しない. 親クラスで prepend
```ruby:const_module_parent_prepend_parent.rb
#
# 定数はモジュールと親クラスに定義され, 子クラスでは定義しない
#
module Mdl
  CONST_VAR = "Mdl#CONST_VAR"
end

class Cls1
  prepend Mdl
  CONST_VAR = "Cls1#CONST_VAR"
end

class Cls2 < Cls1
end

# 継承チェーン
Cls2.ancestors # :=> [Cls2, Mdl, Cls1, Object, Kernel, BasicObject]

# Cls1 で定義した定数が出力される
Cls2::CONST_VAR # :=> Cls1#CONST_VAR
```

## [5]定数はモジュールと親クラスに定義. 子クラスでは定義しない. 子クラスで prepend
```ruby:const_module_parent_prepend_child.rb
#
# 定数はモジュールと親クラスに定義され, 子クラスでは定義しない
#
module Mdl
  CONST_VAR = "Mdl#CONST_VAR"
end

class Cls1
  CONST_VAR = "Cls1#CONST_VAR"
end

class Cls2 < Cls1
  prepend Mdl
end

# 継承チェーン
Cls2.ancestors # :=> [Mdl, Cls2, Cls1, Object, Kernel, BasicObject]

# Mdl で定義した定数が出力される
Cls2::CONST_VAR # :=> Mdl#CONST_VAR
```

## [6]定数はモジュールと親クラスに定義. 子クラスでも定義する. 子クラスで include
```ruby:const_module_parent_child_incude_child.rb
#
# 定数はモジュールと親クラスに定義され, 子クラスでも定義する
#
module Mdl
  CONST_VAR = "Mdl#CONST_VAR"
end

class Cls1
  CONST_VAR = "Cls1#CONST_VAR"
end

class Cls2 < Cls1
  include Mdl
  CONST_VAR = "Cls2#CONST_VAR"
end

# 継承チェーン
Cls2.ancestors # :=> [Cls2, Mdl, Cls1, Object, Kernel, BasicObject]

# Cls2 で定義した定数が出力される
Cls2::CONST_VAR # :=> Cls2#CONST_VAR
```

## [7]定数はモジュールと親クラスに定義. 子クラスでも定義する. 子クラスで prepend
```ruby:const_module_parent_child_prepend_child.rb
#
# 定数はモジュールと親クラスに定義され, 子クラスでも定義する
#
module Mdl
  CONST_VAR = "Mdl#CONST_VAR"
end

class Cls1
  CONST_VAR = "Cls1#CONST_VAR"
end

class Cls2 < Cls1
  prepend Mdl
  CONST_VAR = "Cls2#CONST_VAR"
end

# 継承チェーン
Cls2.ancestors # :=> [Mdl, Cls2, Cls1, Object, Kernel, BasicObject]

# Cls2 で定義した定数が出力される
Cls2::CONST_VAR # :=> Cls2#CONST_VAR
```

## 結論
* 定数の探索は継承チェーンを辿って行われる
* 定数の定義がモジュール, 親クラスだけで行われている場合
 * モジュールの include が親クラスで行われれば, 親クラスで定義された定数が見つかって終わり( 本記事の[2]のケース )
 * モジュールの include が子クラスで行われれば, モジュールで定義された定数が見つかって終わり( 本記事の[3]のケース )
 * モジュールの prepend が親クラスで行われても, 親クラスで定義された定数が見つかって終わり( 本記事の[4]のケース )
 * モジュールの prepend が子クラスで行われれば, モジュールで定義された定数が見つかって終わり( 本記事の[5]のケース )

* 定数の定義がモジュール, 親クラス、子クラスで行われている場合
 * モジュールの inlude が子クラスで行われても, 子クラスで定義された定数が見つかって終わり( 本記事の[6]のケース )
 * モジュールの prepend が子クラスで行われても, 子クラスで定義された定数が見つかって終わり( 本記事の[7]のケース )

