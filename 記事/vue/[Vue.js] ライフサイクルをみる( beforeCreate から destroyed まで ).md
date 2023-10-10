# はじめに

Vue.js には _**ライフサイクル**_ という概念があって、Vue.js で作られたページはこのライフサイクルをもとに処理が実行されていく。
で、Vue.js を使うにあたり、ライフサイクルを理解しているとしていないとでは効率や作法といった点で違いが出てくると思うので、まずはこれを学んでいく。

( すでに [こちら](https://swallow-incubate.com/archives/blog/20190422/) や [こちら](https://qiita.com/chan_kaku/items/7f3233053b0e209ef355) に先達の方々の素晴らしい記事があるが、自分でも記事を書くことで知識の定着を狙う )



# 前提

- Vue CLI でプロジェクトを作成していること
- 単一ファイルコンポーネントであること( [^1] )
- 動作確認は `$ npm run serve` で起動した環境で行なっている



# 環境

|         | Version | 備考 |
| :------ | :------ | :--- |
| Vue     | 2.6.11  |      |
| Vue CLI | 4.1.1   |      |



# ライフサイクル

Vue.js におけるライフサイクルは以下の _**8 つ**_ ｡

|      | ライフサイクル | タイミング                                        | 備考                                                         |
| ---- | -------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| 1    | beforeCreate   | インスタンスは生成されたがデータが初期化される前  | **Create の前** とか言っているくせに **インスタンスは生成されている** 点に注意 |
| 2    | created        | インスタンスが生成され､且つデータが初期化された後 | こちらは名前の通りで､直感的にわかりやすい                    |
| 3    | beforeMount    | インスタンスが DOM 要素にマウントされる前         | こちらも名前の通りでわかりやすい                             |
| 4    | mounted        | インスタンスが DOM 要素にマウントされた後         | 同上                                                         |
| 5    | beforeUpdate   | データは更新されたが DOM に適用される前           | **Update の前** とか言っているくせに **データは更新されている** 点に注意<br />とはいえ､ Update が **DOM の更新** を指すならば､名前のとおりか |
| 6    | updated        | データが更新され､且つ DOM に適用された後          | こちらは名前の通りで､直感的にわかりやすい                    |
| 7    | beforeDestroy  | Vue インスタンスが破壊される前                    | 同上                                                         |
| 8    | destroyed      | Vue インスタンスが破壊された後                    | 同上                                                         |

暗記するものでもないが､ **`create`, `mount`, `update`, `destroy` の 4 つに対して､それぞれ before と after の動きがある** と､捉えれば覚えやすい｡

公式の [こちら](https://jp.vuejs.org/v2/guide/instance.html#ライフサイクルダイアグラム) からライフサイクルの流れを描いた図を転載させていただく｡



## ライフサイクルダイアグラム

 ![lifecycle.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/22496f23-a2ae-cb44-f2df-c7126d865fe6.png)


# 実際に動きを確認する

## 確認用コード

ライフサイクルの動きを見るために､各ライフサイクルのフックに対してログを仕込んだ｡
後述の [スクリーンショット](#スクリーンショット) で､本コードの説明を図とともに見ていく｡

```vue
<template>
  <div :class="$style.lifecycle">
    <input
      v-model="properties.message"
      :class="$style.message"
      placeholder="edit me">
    <p>Message is: {{ properties.message }}</p>
  </div>
</template>

<script>
export default {
  name: 'Lifecycle',
  data: function() {
    return {
      properties: {
        message: 'default value.',
      },
    }
  },

  /* ################################ ライフサイクル ################################ */
  /**
   *  [公式](https://jp.vuejs.org/v2/api/index.html#beforeCreate) から拝借｡
   *
   * データの監視とイベント/ウォッチャのセットアップより前の、インスタンスが初期化されるときに同期的に呼ばれます。
   */
  beforeCreate: function() {
    try {
      this.properties.message = 'set value on beforeCreate.'
      console.log(`message is ${this.properties.message}`)
    } catch (e) {
      console.log(e)
    }
  },

  /**
   * [公式](https://jp.vuejs.org/v2/api/index.html#created) から拝借｡
   *
   * インスタンスが作成された後に同期的に呼ばれます。
   * この段階では、インスタンスは、データ監視、算出プロパティ、メソッド、watch/event コールバックらの
   * オプションのセットアップ処理が完了したことを意味します。
   * しかしながら、マウンティングの段階は未開始で、`$el` プロパティはまだ利用できません。
   */
  created: function() {
    this.properties.message = 'set value on created.'
    console.log(`[LifeCycle] created. this.properties.message = ${this.properties.message}`)
  },

  /**
   * [公式](https://jp.vuejs.org/v2/api/index.html#beforeMount) から拝借｡
   *
   * `render` 関数が初めて呼び出されようと、マウンティングが開始される直前に呼ばれます。
   */
  beforeMount: function() {
    this.properties.message = 'set value on beforeMount.'
    console.log(`[LifeCycle] beforeMount. this.properties.message = ${this.properties.message}`)
  },

  /**
   * [公式](https://jp.vuejs.org/v2/api/index.html#mounted) から拝借｡
   *
   * 新たに作成される `vm.$el` によって置き換えられる `el` に対して、インスタンスがマウントされたちょうど後に呼ばれます。
   * ルートインスタンスがドキュメントの中の要素にマウントされる場合、`vm.$el` も `mounted` が呼び出されるときにドキュメントの中に入ります。
   * `mounted` は 全ての子コンポーネントもマウントされていることを保証**しない**ことに注意してください。
   * ビュー全体がレンダリングされるまで待つ場合は、 `mounted` の代わりに
   * [vm.$nextTick](https://jp.vuejs.org/v2/api/index.html#vm-nextTick) を使うことができます。
   *
   * このフックはサーバサイドレンダリングでは呼ばれません。
   */
  mounted: function() {
    this.properties.message = 'set value on mounted.'
    console.log(`[LifeCycle] mounted. this.properties.message = ${this.properties.message}`)
  },

  /**
   * [公式](https://jp.vuejs.org/v2/api/index.html#beforeUpdate) から拝借｡
   *
   * データが変更されるとき、DOM が適用される前に呼ばれます。
   * これは、更新前に既存の DOM にアクセスするのに適しています。
   * 例: 手動で追加されたイベントリスナを削除する
   *
   * このフックはサーバサイドレンダリングでは呼ばれません。
   * サーバサイドでは初期描画のみ実行されるためです。
   */
  beforeUpdate: function() {
    // 注意!!
    // beforeUpdate と updated で同じ変数に対してデータを更新かけると無限ループに陥る
    console.log(`[LifeCycle] beforeUpdate. this.properties.message = ${this.properties.message}`)
  },

  /**
   * [公式](https://jp.vuejs.org/v2/api/index.html#updated) から拝借｡
   *
   * データが変更後、仮想 DOM が再描画そしてパッチを適用によって呼ばれます。
   * このフックが呼び出されるとき、コンポーネントの DOM は更新した状態になり、このフックで DOM に依存する操作を行うことができます。
   * しかしがながら、ほとんどの場合、無限更新ループに陥る可能性があるため、このフックでは状態を変更するのを回避すべきです。
   *
   * `updated` は 全ての子コンポーネントも再レンダリングされていることを保証**しない**ことに注意してください。
   * ビュー全体が再レンダリングされるまで待つ場合は、 `updated` の代わりに
   * [vm.$nextTick](https://jp.vuejs.org/v2/api/index.html#vm-nextTick) を使うことができます。
   * このフックはサーバサイドレンダリングでは呼ばれません。
   */
  updated: function() {
    // 注意!!
    // beforeUpdate と updated で同じ変数に対してデータを更新かけると無限ループに陥る
    console.log(`[LifeCycle] updated. this.properties.message = ${this.properties.message}`)
  },

  /**
   * [公式](https://jp.vuejs.org/v2/api/index.html#beforeDestroy) から拝借｡
   *
   * > Vue インスタンスが破棄される直前に呼ばれます。
   * この段階ではインスタンスはまだ完全に機能しています。
   *
   * **このフックはサーバサイドレンダリングでは呼ばれません。**
   */
  beforeDestroy: function() {
    console.log(`[LifeCycle] beforeDestroy. this.properties.message = ${this.properties.message}`)
  },

  /**
   * [公式](https://jp.vuejs.org/v2/api/index.html#destroyed) から拝借｡
   *
   * Vue インスタンスが破棄された後に呼ばれます。
   * このフックが呼ばれるとき、Vue インスタンスの全てのディレクティブはバウンドしておらず、
   * 全てのイベントリスナは削除されており、そして全ての子の Vue インスタンスは破棄されています。
   *
   * このフックはサーバサイドレンダリングでは呼ばれません。
   */
  destroyed: function() {
    console.log(`[LifeCycle] destroyed. this.properties.message = ${this.properties.message}`)
  },
}
</script>

<style module>
.lifecycle {
    margin: 20px;
}

.message {
    width: 400px;
}
</style>

```



## 1. beforeCreate > mounted まで

前掲のコードをベースに次の **3パターン** で動きを確認する｡

1. 変数 `properties.message` の更新を `created` で止める
2. 変数 `properties.message` の更新を `beforeMount` で止める
3. 変数 `properties.message` の更新を `mounted` で止める

以下､順に見ていく｡



### 1.1. 変数 `properties.message` の更新を `created` で止める

![1.1.beforeCreate2mounted.JPG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/54106842-a708-d021-9dd6-6d888344fa9c.jpeg)

```vue
<script>
// ...
// 省略
// ...

  data: function() {
    return {
      properties: {
        message: 'default value.',
      },
    }
  },
  beforeCreate: function() {
    try {
      this.properties.message = 'set value on beforeCreate.'
      console.log(`message is ${this.properties.message}`)
    } catch (e) {
      console.log(e)
    }
  },
  created: function() {
    this.properties.message = 'set value on created.'
    console.log(`[LifeCycle] created. this.properties.message = ${this.properties.message}`)
  },
  beforeMount: function() {
    console.log(`[LifeCycle] beforeMount. this.properties.message = ${this.properties.message}`)
  },
  mounted: function() {
    console.log(`[LifeCycle] mounted. this.properties.message = ${this.properties.message}`)
  },
// ...
// 省略
// ...
</script>
```

前掲の [ライフサイクルダイアグラム](#ライフサイクルダイアグラム) と合わせて考えると､次の動きとなっていることが理解できる｡

1. インスタンス生成時にライフサイクルが初期化される
2. `beforeCreated` で `properties.message` の値を `set value on beforeCreate.` に更新しようとするが､このとき例外が発生している
   - つまり､このタイミングでは **まだインスタンスの生成が終わったいない** ため､ `data` に定義された変数を **参照できない**
   - 今回のコードでは例外情報をログ出力したあと､そのまま次の処理に進むので､`created` が実行される
3. `created` では `properties.message`  を `set value on created.` で更新している
   - ログ出力されている情報から､正常にデータが更新されたことがわかる
4. 続く `beforeMount` 並びに `mounted` で `properties.message` を更新していないので､ **`created` フックでセットした `set value on created.` がそのまま DOM にセットされて画面上に表示** されている



### 1.2. 変数 `properties.message` の更新を `beforeMount` で止める

![1.2.beforeCreate2mounted.JPG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/97035951-8ebb-2656-1b86-15843f31a951.jpeg)

```vue
<script>
// ...
// 省略: `beforeMount` まで省く
// ...
  beforeMount: function() {
    this.properties.message = 'set value on beforeMount.'
    console.log(`[LifeCycle] beforeMount. this.properties.message = ${this.properties.message}`)
  },
  mounted: function() {
    console.log(`[LifeCycle] mounted. this.properties.message = ${this.properties.message}`)
  },
// ...
// 省略
// ...
</script>
```

`created` までの動きは前項と同じ｡ **差分となる `beforeMount` 以降** を見る｡

1. `beforeMount` で変数 `properties.message` に `set value on beforeMount.` をセットしている
   - ログ出力されている情報から､`created` でセットされた後に本フックで正常にデータが更新されたことがわかる
   - 続く `mounted` で `properties.message` を更新していないので､ **`beforeMount` フックでセットした `set value on beforeMount.` がそのまま DOM にセットされて画面上に表示** されている



### 1.3. 変数 `properties.message` の更新を `mounted` で止める

![1.3_beforeCreate2updated.JPG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/a1f57bcb-04ed-307e-71d6-e8b93bdd6473.jpeg)

```vue
<script>
// ...
// 省略: `mounted` まで省く
// ...
  mounted: function() {
    this.properties.message = 'set value on mounted.'
    console.log(`[LifeCycle] mounted. this.properties.message = ${this.properties.message}`)
  },
  beforeUpdate: function() {
    console.log(`[LifeCycle] beforeUpdate. this.properties.message = ${this.properties.message}`)
  },
  updated: function() {
    console.log(`[LifeCycle] updated. this.properties.message = ${this.properties.message}`)
  },
// ...
// 省略
// ...
</script>
```

`beforeMount` までの動きは前項と同じ｡ **差分となる `mounted` **を見る｡

1. `mounted` で変数 `properties.message` に `set value on mounted.` をセットしている
   - ログ出力されている情報から､正常にデータが更新されたことがわかる

注目したいのは `mounted` 以降のログ｡
`beforeUpdate` ､並びに `updated` で変数の更新は行っていない｡ にも関わらず､  **`beforeUpdate` と `updated` のログが出ている** ことから､ `mounted` で変数を更新していることと併せて次のことがわかる｡

1. `beforeMount` > `mounted` の間に **DOM の更新が行われた**
2. この時点で画面上には `set value on beforeMount.` が表示される
3. その後 `mounted` のフックが実行される
4. で､ `mounted` で変数を更新しているので､ `beforeUpdate`､次いで `updated` が実行された
5. **DOM の更新は `beforeUpdate` と `updated` の間に行われる** ので画面上には **`mounted` フックで更新した値である `set value on mounted.` が表示** されている
6. これは [ライフサイクルダイアグラム](#ライフサイクルダイアグラム) の流れに合致する



### 注意点

いわゆる **コンストラクタ** で行うような､ **Vue インスタンスに対する初期処理** は `created`､もしくは `beforeMount` で行うと良さそう｡
どちらで行うのがより良いのかは､実際にプロダクトを作っていく上で判断したい｡

ただ `mounted` でやるのは実行コストが高そうなので止めておく｡ 理由は前掲のとおりで､`mounted` で DOM に紐づく変数を更新すると､それに引っ張られて `beforeUpdate` と `updated` のフックも実行されるから｡



## 2. beforeUpdate > updated まで

こちらは特にパターンを確認することもない｡前掲のコードとともに動きを見てみる｡

![2_beforeUpdate2updated.JPG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/38f3b541-863a-3daf-d5e8-0e87ee5d6ed6.jpeg)

```vue
<script>
// ...
// 省略: `beforeUpdate` まで省く
// ...
  beforeUpdate: function() {
    // 注意!!
    // beforeUpdate と updated で同じ変数に対してデータを更新かけると無限ループに陥る
    console.log(`[LifeCycle] beforeUpdate. this.properties.message = ${this.properties.message}`)
  },
  updated: function() {
    // 注意!!
    // beforeUpdate と updated で同じ変数に対してデータを更新かけると無限ループに陥る
    console.log(`[LifeCycle] updated. this.properties.message = ${this.properties.message}`)
  },
// ...
// 省略
// ...
</script>
```

初期表示の画面は以下の状態で表示されていた｡

| 項目             | 表示されていた文字列                 |
| ---------------- | ------------------------------------ |
| テキスト入力欄   | `set value on created.`              |
| 入力内容の表示欄 | `Message is: set value on created. ` |

上記を前提に､ここでは画面上でデータを更新したときの動きを確認した｡
行った内容は次のとおり｡

1. テキスト入力欄に表示されている `set value on created.` の後に `> updated` の文字列を入力



入力した内容に対するログの出方はスクリーンショットの画像の通りで､

1. `beforeUpdate` でログ出力
2. `update` でログ出力

と `beforeUpdate` > `update` の順で出力されており､ [ライフサイクルダイアグラム](#ライフサイクルダイアグラム) の流れに合致することが確認できた｡



### 注意点

で､ここではスクリーンショットを貼らなかったが､ソースコードのコメントに記載したとおり､

```javascript
// beforeUpdate と updated で同じ変数に対してデータを更新かけると無限ループに陥る
```

ので注意｡

これは次の動きによるものだと思う｡

1. `beforeUpdate` で変数を更新
2. `updated` が走り､ここでも変数を更新
3. するとまた `beforeUpdate` が走る
4. 以下､ **ループ**



**`beforeUpdate` と `updated` での変数の扱いには充分に注意** したい｡



## 3. beforeDestroy > destroyed まで

こちらも特にパターンを確認することもない｡前掲のコードとともに動きを見てみる｡

![3_beforeDestroy2destroyed.JPG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/5f816949-539f-9ab8-e416-82dcb83046db.jpeg)

```vue
<script>
// ...
// 省略: `beforeUpdate` まで省く
// ...
  beforeDestroy: function() {
    console.log(`[LifeCycle] beforeDestroy. this.properties.message = ${this.properties.message}`)
  },
  destroyed: function() {
    console.log(`[LifeCycle] destroyed. this.properties.message = ${this.properties.message}`)
  },
// ...
// 省略
// ...
</script>
```

画面上部の `Life Cycle ` がこれまでライフサイクルの確認をしていた画面｡ 
`beforeDestroy` と `destroyed` の確認をするため､隣の `Sample Buefy` に移動してみた｡

ログには `beforeDestroy` > `destroyed` の順で出力されており､ここも [ライフサイクルダイアグラム](#ライフサイクルダイアグラム) の流れに合致することが確認できた｡



### 注意点

`beforeDestroy` は重要な役割を持つ｡

`beforeDestroy` はいわゆる **デストラクタ** 的な役割を持つと解釈してよく､このフックで **子コンポーネント** や独自で用意した **イベントリスナー** とか **タイマー処理** とかの後始末をした方が良い｡

前掲の [ライフサイクル](#ライフサイクル) で触れたとおり`destroyed` は **Vue インスタンスの破棄した後** に実行されるフックなので､そこでやろうとしても **遅すぎる** ためである｡



# ライフサイクルの注意点

公式ページの [こちら](https://jp.vuejs.org/v2/guide/instance.html#インスタンスライフサイクルフック) に注意点が記載されているので転載しておく｡
上記によると､ 

> 全てのライフサイクルフックは、`this` が Vue インスタンスを指す形で実行されます。

とのこと｡

またこれに関連して

> インスタンスプロパティまたはコールバックで[アロー関数](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Functions/Arrow_functions) を使用しないでください。例えば、 `created: () => console.log(this.a)` や `vm.$watch('a', newValue => this.myMethod())` です。アロー関数は `this` をもたないため、`this` は他の変数と同様に見つかるまで親スコープをレキシカルに探索され、そしてしばしば、`Uncaught TypeError: Cannot read property of undefined` または `Uncaught TypeError: this.myMethod is not a function` のようなエラーが発生します。

とある｡

以下､語弊があるかもしれないが､自分なりの解釈を｡

上記で記載されているアロー関数に対する記述は通常メリットとして捉えられて､そのためにアロー関数は `const self = this;`といったことをせずにメッセージの送り元を扱える｡

だが Vue.js におけるライフサイクルではこれがデメリットとなり得るとのこと｡うっかりアロー関数を使って迷路に迷い込まないよう､充分に注意したい｡



# まとめにかえて

`beforeCreate` から `destroyed` までの動きをざっと見てきた｡ その中での注意点を書き出すことでまとめに代えたい｡

- Vue インスタンスの初期処理は `created` か `beforeMount` で行うのが良さげ
- `beforeUpdate` と `updated` で DOM に関係する変数の更新を **下手に行う** と **無限ループに陥る可能性がある** ので注意する
- `beforeDestroy` では Vue インスタンスの後始末を行う

DOM に紐づく変数の更新については､ **[算出プロパティとウォッチャ](https://jp.vuejs.org/v2/guide/computed.html)** もあるので､今後はそちらも見ていきたい｡

# ソースコード

今回の記事で動作確認に使用したコードは下記にアップしております｡
ご参考まで。( 以下は ブランチのリンクですが､ master にもマージ済みです )

- [ksh-fthr/vue-work > feature/#life_cycle](https://github.com/ksh-fthr/vue-work/tree/feature/%23life_cycle)

 

# 参考

## 公式

- [Vue.js インスタンス](https://jp.vuejs.org/v2/guide/instance.html)
- [Vue.js オプション / ライフサイクルフック](https://jp.vuejs.org/v2/api/index.html#オプション-ライフサイクルフック)
- [算出プロパティとウォッチャ](https://jp.vuejs.org/v2/guide/computed.html)


## 上記以外

- [第３回 – Vue.jsのライフサイクル等の簡易逆引きリファレンス](https://swallow-incubate.com/archives/blog/20190422/)
- [Vueのライフサイクルを完全に理解した](https://qiita.com/chan_kaku/items/7f3233053b0e209ef355)

[^1]: Vue.js　のコンポーネントを単独のファイルとして作成する機能<br>拡張子「.vue」のファイルのことで`<template>`, `<script>`, `<style>` のブロックで構成されている。
