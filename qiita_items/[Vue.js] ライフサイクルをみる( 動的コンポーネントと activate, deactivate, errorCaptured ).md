# はじめに

Vue.js におけるライフサイクルについて [こちら](https://qiita.com/ksh-fthr/items/2a9f173c706ef6939f93) でみた｡
ただ [公式のこちら](https://jp.vuejs.org/v2/api/index.html#%E3%82%AA%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3-%E3%83%A9%E3%82%A4%E3%83%95%E3%82%B5%E3%82%A4%E3%82%AF%E3%83%AB%E3%83%95%E3%83%83%E3%82%AF) には上記の記事で扱った以外のフックとして以下の 3 つが取り上げられている｡

- **_activated_**
- **_deactivated_**
- **_errorCaptured_**

本記事ではこれら 3 つのフックについて動きを見ていく｡



# お詫び

記事中､コンポーネントの名称について スクリーンショット と 文中の単語について表記ゆれがあります( `Life Cycle`, `LifeCycle`, `Lifecycle` 等 )が､同じコンポーネントを指しております｡
お見苦しくて申し訳ありません｡



# 前提

- Vue CLI でプロジェクトを作成していること
- 単一ファイルコンポーネントであること( [^1] )
- 動作確認は `$ npm run serve` で起動した環境で行なっている
- `activated` と `deactivated` の説明のために動的コンポーネントを扱っているが､動的コンポーネントそのものについては触れない



# 環境

|         | Version | 備考 |
| ------- | ------- | ---- |
| Vue     | 2.6.11  |      |
| Vue CLI | 4.1.1   |      |


# 今回扱うライフサイクルフック

|      | フック        | タイミング                       | 備考                                                         |
| ---- | ------------- | -------------------------------- | ------------------------------------------------------------ |
| 1    | activated     | コンポーネントが活性化するとき   | 同上                                                         |
| 2    | deactivated   | コンポーネントが非活性になるとき | 同上                                                         |
| 3    | errorCaptured | エラーが発生したとき             | 子孫コンポーネントからのエラーをキャッチする<br />[**エラー伝播ルール**](https://jp.vuejs.org/v2/api/index.html#errorCaptured) を把握しておくこと |



# activate と deactivate

前掲の公式ページによると､ `activate` と `deactivate` には次の 2 つが参照として挙がっている｡

- [組み込みコンポーネント - keep-alive](https://jp.vuejs.org/v2/api/index.html#keep-alive)
- [動的コンポーネント - keep-alive](https://jp.vuejs.org/v2/guide/components-dynamic-async.html#dynamic-component-demo)

まず **組み込みコンポーネント** の方を覗いてみると､次の説明がある｡

[公式ページから引用]

> 動的コンポーネント周りでラップされるとき、`<keep-alive>` はそれらを破棄しないで非アクティブなコンポーネントのインスタンスをキャッシュします。`<trasition>` に似ていて、`<keep-alive>` はそれ自身 DOM 要素で描画されない抽象型コンポーネントです。`activated` と `deactivated` ライフサイクルフックはそれに応じて呼び出されます。
>
> コンポーネントが `<keep-alive>` 内部でトグルされるとき、`activated` と `deactivated` ライフサイクルフックはそれに応じて呼び出されます。
>
> > 2.2.0 以降では、`<keep-alive>`ツリーの中の全てのネストされたコンポーネントに対して、`activated` および `deactivated` を発行します。
>
> 主に、コンポーネント状態を保存したり、再描画を避けるために使用されます。


ということで､ 

- `activate` と `deactivate` はライフサイクルの仲間であり
- `<keep-alive>` でコンポーネントを囲むことで､
    - 囲まれたコンポーネントは **コンポーネントの切り替え** が行われると **`activate` と `deactivate` のフックが走る**
    - このとき切り替えられたコンポーネントの Vue インスタンスは **破棄されずに残る**

ことが分かる｡
**動的コンポーネント** については､ここで取り上げると長くなるので割愛させていただく｡前掲の公式ページ( [動的コンポーネント - keep-alive](https://jp.vuejs.org/v2/guide/components-dynamic-async.html#dynamic-component-demo) )を参照されたい｡

では実際の動きをコードで見てみる｡

## 確認用コードについて

- 動的コンポーネントの動きとあわせて見るため､親コンポーネントと 2 つの子コンポーネント､計 3 つのコンポーネントを用意した
- なお子コンポーネントについては､ログ出力の内容やコンポーネント名を除き､同じ構造である
- これは両者とも同じ構造の方が動作確認時に比較しやすいとの意図のもと､そうしている( 冗長ではあるがご容赦いただきたい )



## 親コンポーネント( LifeCycle2 )

```vue
<template>
  <div :class="$style.lifecycle2">
    <div :class="$style.wrapchild">
      <input 
        id="Lifecycle21" 
        v-model="selectedComponent" 
        type="radio" 
        value="Lifecycle21">
      <label for="Lifecycle21">Lifecycle21</label>
    </div>
    <div :class="$style.wrapchild">
      <input 
        id="Lifecycle22"
        v-model="selectedComponent" 
        type="radio" 
        value="Lifecycle22">
      <label for="Lifecycle22">Lifecycle22</label>
    </div>

    <h1>selected component is ... {{ selectedComponent }}.</h1>

    <keep-alive>
      <component 
        :is="selectedComponent" 
        :class="$style.lifecycle2_child_area"/>
    </keep-alive>

  </div>
</template>

<script>
import Lifecycle21 from '@/components/LifeCycle2-1.vue'
import Lifecycle22 from '@/components/LifeCycle2-2.vue'

export default {
  name: 'Lifecycle2',
  components: {
    Lifecycle21,
    Lifecycle22
  },
  data: function() {
    return {
      selectedComponent: '',
    }
  },

  beforeUpdate: function() {
    console.log('[LifeCycle2] beforeUpdate.')
  },

  updated: function() {
    console.log('[LifeCycle2] updated.')
  },

  beforeDestroy: function() {
    console.log('[LifeCycle2] beforeDestroy.')
  },

  destroyed: function() {
    console.log('[LifeCycle2] destroyed.')
  },

  errorCaptured: function() {
    console.log('[LifeCycle2] errorCaptured.')
  }, 
}
</script>

// スタイルは割愛
```



## 子コンポーネント1( LifeCycle21 )

```vue
<template>
  <div :class="$style.lifecycle2_child">
    <p>input on LifeCycle21.</p>
    <input
      :class="$style.message"
      v-model="properties.message"
      placeholder="edit me">
  
    <p>Message is: {{ properties.message }}</p>
  </div>
</template>

<script>
export default {
  name: 'LifeCycle21',
  data: function() {
    return {
      properties: {
        message: 'default value.',
      },
    }
  },

  beforeDestroy: function() {
    console.log('[LifeCycle2-1] beforeDestroy.')
  },

  destroyed: function() {
    console.log('[LifeCycle2-1] destroyed.')
  },

  activated: function() {
    console.log('[LifeCycle2-1] activated.')
  },

  deactivated: function() {
    console.log('[LifeCycle2-1] deactivated.')
  },
}
</script>

// スタイルは割愛
```



## 子コンポーネント2( LifeCycle22 )

```vue
<template>
  <div :class="$style.lifecycle2_child">
    <p>input on LifeCycle22.</p>
    <input
      :class="$style.message"
      v-model="properties.message"
      placeholder="edit me">
  
    <p>Message is: {{ properties.message }}</p>
  </div>
</template>

<script>
export default {
  name: 'LifeCycle22',
  data: function() {
    return {
      properties: {
        message: 'default value.',
      },
    }
  },

  beforeDestroy: function() {
    console.log('[LifeCycle2-2] beforeDestroy.')
  },

  destroyed: function() {
    console.log('[LifeCycle2-2] destroyed.')
  },

  activated: function() {
    console.log('[LifeCycle2-2] activated.')
  },

  deactivated: function() {
    console.log('[LifeCycle2-2] deactivated.')
  },
}
</script>

// スタイルは割愛
```



## 動作確認

1. **初期表示( LifeCycle2 を表示 )**
   - ![1_1_initial_display.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/2c43a8fb-affa-7b79-99c4-24db8140a02b.png)
   - 本稿のコードでは `beforeCreate`, `created` のライフサイクルフックを設けていないので､ログには何も出力されない

2. **LifeCycle21 を選択**
   - ![1_2_select_lifcycle21.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/d4952d81-812f-e9a7-db09-389b134088a2.png)
   - `LifeCycle21` の `activated` がログにでている
     - このとき親コンポーネントの `beforeUpdate` と `updated` で囲まれていることがわかる
     - つまり **動的コンポーネントが生成される** イコール **親コンポーネントのDOMが更新される** ということが､これでわかる

3. **LifeCycle21 でデータ入力**
   - ![1_3_input_on_lifecycle21.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/40d4b5b3-9639-f6f8-c027-03b553053133.png)
   - データ入力した際､ログには何も出力されていない
     - 子コンポーネントの方で `beforeUpdate` と `updated` のライフサイクルフックを設けていないので､子コンポーネントのログがでていないのは当たり前
     - 親コンポーネントの方でもログが出ていないのが面白い
     - 子コンポーネントで入力された情報は､**あくまで子コンポーネント内で閉じた動き** であり､そこで DOM が更新されても **親コンポーネントに影響しない** ことがわかる

4. **LifeCycle22 を選択**
   - ![1_4_select_lifcycle22.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/f24e9afc-337f-3f53-07fb-a142bde72de9.png)
   - 子コンポーネントを切り替えたことで､親コンポーネントの方で `beforeUpdate` と `updated` が走った
   - コンポーネントが切り替えられたことで､ `LifeCycle21` の方では **非活性** になるため `deactivate` が走っている
   - 逆に `LifeCycle22` は **活性** になるため `activate` が走っている

5. **LifeCycle21 を再度選択**
   - ![1_5_reselect_lifcycle21.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/89a89bc7-0364-9a62-ef20-470a8cf10e27.png)
   - 本動作確認におけるキモがこちら
     - `LifeCycle22` >> `LifeCycle21` に戻ったとき､ **前に入力していた文字列が残っている**
     - つまり `<keep-alive>` で動的コンポーネントを囲むことで､ **囲まれたコンポーネントは切り替えが行われてもインスタンスの破棄は行われず**､また **インスタンスで保持していた値もそのまま残る**､ということがこれで確認できた

6. **画面上部のメニューから別ページを選択**
   - ![1_6_leave_from_lifecycle2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/073f62b0-fbc3-e6d3-0a43-93327b1355cd.png)
   - では動的コンポーネントの切り替えではなく､**親コンポーネントごと別のコンポーネントに切り替え**たらどうなるか
   - ログからは追える実行順序が興味深い
     1. 親コンポーネントの `beforeDestroy` が走り
     2. 選択していた `LifeCycle21` の `deactivate` が走る. 
     3. そして､同じく `LifeCycle21` の `beforeDestroy` と `destroyed` が走る
     4. 次にもう一つの子コンポーネントである `LifeCycle22` の `beforeDestroy` と `destroyed` が走り
     5. 最後に親コンポーネントの `destroyed` が走っている

7. **LifeCycle2 に戻る >> LifeCycle21 を選択**
   - ![1_7_revisit_to_lifecycle2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/8fc81610-63b7-0be7-8223-361380769079.png)
   - 移動先のページから再度 `LifeCycle2` を表示したときの動きを確認する
     - ログの出方は本動作確認の 手順-2 と同じ
     - 注目したいのは **入力欄の値がデフォルト値になっている** こと
     - 一つ前の手順で確認したログからもわかるが､ **親コンポーネントごと別のコンポーネントに切り替えると､`<keep-alive>` で囲っていても､子コンポーネントのインスタンスも破棄される** ということが確認できた	



# 動的コンポーネントを `<keep-alive>` で囲まなかったら

上記は `<keep-alive>` で動的コンポーネントを囲ったときの動きで､期待通り `activate` と `deactivate` が走ることが確認できた｡
では動的コンポーネントを `<keep-alive>` で囲まなかったときにどう動くのかを見ておきたい｡

なお本項では親コンポーネントの `<template>` をいじるだけで､子コンポーネントの変更はないので､子コンポーネントのコードは割愛する｡



## 親コンポーネント( LifeCycle2 )

動的コンポーネントを `<keep-alive>` で囲まなかったときにどう動くのか ?

```vue
<template>
  <div :class="$style.lifecycle2">

    <-- ******** -->
    <-- 差分は割愛 -->
    <-- ******** -->
    <component
      :is="selectedComponent" 
      :class="$style.lifecycle2_child_area"/>

  </div>
</template>

// テンプレートの差分だけなのでスクリプトは割愛

// スタイルは割愛
```



## 動作確認

1. **初期表示( LifeCycle2 を表示 )**
   - ![2_1_initial_display.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/24de4a03-8fa0-075e-76a2-9e06b59b431e.png)
   - 本稿のコードでは `beforeCreate`, `created` のライフサイクルフックを設けていないので､ログには何も出力されない

2. **LifeCycle21 を選択 >> LifeCycle21 でデータ入力**
   - ![2_2_input_on_lifecycle21.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/8809db59-979e-d94b-b157-4f96cce94808.png)
   - 子コンポーネントである `LifeCycle21` で `activated` のログが出ていない
   - このことから､`<keep-alive>` で囲むことで､`activate` が有効になることがわかる

3. **LifeCycle22 を選択**
   - ![2_3_select_lifcycle22.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/ac788ed0-7782-f89c-bac5-899233b623a1.png)
   - 親コンポーネントでの `beforeUpdate` と `updated` のログは出ているが､子コンポーネントである `LifeCycle21` で `deactivated` のログが出ていない
   - 前手順と同じく､ここでも `<keep-alive>` で囲むことで､`deactivate` が有効になることがわかる

4. **LifeCycle21 を再選択**
   - ![2_4_reselect_lifcycle21.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/fd6ec7ec-1543-3129-45cf-0eaf70e73818.png)
   - 親コンポーネントでの `beforeUpdate` と `updated` のログは出ているが､子コンポーネントである `LifeCycle21` で `activated` のログが出ていない
   - また `LifeCycle22` を選択する前に入力していた文字列が **デフォルト文字列** に置き換わっている


ということで､

- 動的コンポーネントを `<keep-alive>` で囲まないと､コンポーネントの切り替えが行われたときに **コンポーネントの破棄と生成** が行われること

が確認できた｡
逆説的にいうと､

- 動的コンポーネントを `<keep-alive>` で囲むことで `activate` と `deactivate` がフックされること
- またそうすることで､動的コンポーネントは別コンポーネントに切り替わってもインスタンスは非活性になるだけで生き続けること
- 再度もとのコンポーネントに切り替わることで､非活性 >> 活性に戻ること

と言うことができる｡



# errorCaptured

`errorCaptured` についても､公式ページから説明を引用させていただく｡

> 任意の子孫コンポーネントからエラーが捕捉されるときに呼び出されます。フックは、エラー、エラーをトリガするコンポーネントインスタンス、そしてどこでエラーが捕捉されたかの文字列情報、これら 3 つの引数を受け取ります。フックはエラーがさらにもっと伝播するのを防ぐために、false を返すことができます。

ということで､前傾のコードをベースにしつつ､こちらも動きを見ていく｡



## 親コンポーネント( LifeCycle21 )

```vue
//
// スクリプトの差分だけなのでテンプレートは割愛
//
<script>
export default {
  name: 'LifeCycle2',
  //
  // 差分のみ抜粋
  //
  errorCaptured: function(error, component, info) {
    console.log('[LifeCycle2] errorCaptured.')
    console.log(error)
    console.log(component)
    console.log(info)
    
    // エラーの伝播を防ぐために、`false` を返す
    // つまり更に上の親へエラーをあげることはせずに､ここで止める
    return false
  },
}
</script>

// スタイルは割愛
```



## 子コンポーネント( LifeCycle21 )

どちらか片方のコンポーネントでエラーを発生させれば､`errorCaptured` のフックは確認できるので､今回は `LifeCycle21` の方でエラーを発生させた｡

```vue
//
// スクリプトの差分だけなのでテンプレートは割愛
//
<script>
export default {
  name: 'LifeCycle21',
  //
  // 差分のみ抜粋
  //
  beforeCreate: function() {
    // わざと例外を発生させて､
    // 親コンポーネントである LifeCycle2.vue の `errorCaptured()` のフックに引っ掛けたい
    this.properties.message = 'set on beforeCreate.'
  },
}
</script>

// スタイルは割愛
```



## 動作確認

1. **初期表示( LifeCycle2 を表示 )**
   - ![3_1_initial_display_err.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/c2c5f2f7-ad9c-708f-2aa0-ec45b66cbe05.png)
   - 本稿のコードでは `beforeCreate`, `created` のライフサイクルフックを設けていないので､ログには何も出力されない｡

2. **LifeCycle21 を選択してエラーを発生させる**
   ![3_2_errorCaptured.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/027c4a7f-91ad-abcd-fdab-ba76d4ce8063.png)
   - `LifeCycle21` を選択したことで､当該コンポーネントのインスタンスが動的に生成された
   - そのタイミングで子コンポーネントである `LifeCycle21P` で `beforeCreate` のフックが走り､例外が発生する
   - 発生した例外をフックして､親コンポーネントである `LifeCycle2` で `errorCaptured` が走った


`errorCaptured` は 3 つの引数から以下の情報を得ることができる｡

1. 例外情報
2. 例外が発生したコンポーネント情報
3. 例外が発生し場所に関する情報を含む文字列

上記を踏まえ､改めてログを見てみると､先に上げた例外発生時の動きと `errorCaptured` の引数から得た情報が合致していることがわかる｡



# まとめ

## activate と deactivate

- 子コンポーネントを動的コンポーネントとして扱い､`<keep-alive>` で囲むことで `activate` と `deactivate` のフックが実行される

- `<keep-alive>` で囲まれた子コンポーネントは､コンポーネントの切り替えが発生してもインスタンスは **非活性になるだけ **で **破棄されずに生き残る**
- でも **親コンポーネントごと別のコンポーネントに切り替えられたら `<keep-alive>` で囲んでいても子コンポーネントのインスタンスは破棄される**
- 動的コンポーネントを `<keep-alive>` で囲まなかったら､通常のインスタンスと同じで **コンポーネントの切り替え** で **インスタンスの生成と破棄** が行われる



## errorCaptured

- **子コンポーネントでエラー** が発生したら **親コンポーネント側** のこのフックが走る
- `errorCaptured` は 3 つの引数から以下の情報を得ることができる｡
  1. 例外情報
  2. 例外が発生したコンポーネント情報
  3. 例外が発生し場所に関する情報を含む文字列
- `false` で復帰することでエラーがさらにもっと伝播するのを防ぐことができる


# ソースコード
今回の記事で動作確認に使用したコードは下記にアップしております｡
ご参考まで。( 以下は ブランチのリンクですが､ master にもマージ済みです )

- [ksh-fthr/vue-work > feature/#life_cycle2](https://github.com/ksh-fthr/vue-work/tree/feature/%23life_cycle2)


# 参考

## 公式

- [Vue.js オプション / ライフサイクルフック](https://jp.vuejs.org/v2/api/index.html#%E3%82%AA%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3-%E3%83%A9%E3%82%A4%E3%83%95%E3%82%B5%E3%82%A4%E3%82%AF%E3%83%AB%E3%83%95%E3%83%83%E3%82%AF)
- [組み込みコンポーネント - keep-alive](https://jp.vuejs.org/v2/api/index.html#keep-alive)
- [動的コンポーネント - keep-alive](https://jp.vuejs.org/v2/guide/components-dynamic-async.html#dynamic-component-demo)

[^1]: Vue.js　のコンポーネントを単独のファイルとして作成する機能<br>拡張子「.vue」のファイルのことで`<template>`, `<script>`, `<style>` のブロックで構成されている。
