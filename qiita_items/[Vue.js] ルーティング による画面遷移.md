# はじめに

Vue.js には URL の移動を伴う画面遷移の手段として **ルーティング** という仕組みが用意されている。
本記事では簡単な例を元に以下の 2点について見ていく。

- [ルーティングの基本的な使い方](#基本)
- [ルーティング時にパラメータを渡す方法](#進んだ使い方)


# 前提

- Vue CLI でプロジェクトを作成していること
- 単一ファイルコンポーネントであること( [^1] )
- 動作確認は `$ npm run serve` で起動した環境で行なっている



# 環境

|         | Version | 備考                                                         |
| ------- | ------- | ------------------------------------------------------------ |
| Vue     | 2.6.11  | [公式はこちら](https://jp.vuejs.org/index.html)|
| Vue CLI | 4.1.1   | [公式はこちら](https://cli.vuejs.org/)|
| Vue Router | 3.0.7 | Vue.js の公式ルータ, [公式はこちら](https://router.vuejs.org/ja/) |
| Buefy   | 0.9.2   | Vue.js 用の UI コンポーネント, MIT ライセンス, [公式はこちら](https://buefy.github.io/) |


# 事前準備

## vue-router の追加

package.json に `vue-router` を追加して `npm install` でモジュールを追加する｡
今回の記事で扱ったバージョンは `3.0.7`｡

```javascript:package.json
// 省略
{
  "dependencies": {
    "vue-router": "3.0.7",  // コレを追加
  }
}
// 省略
```

```bash:vue-routerを追加
$ npm install
added 1 package from 1 contributor and audited 1299 packages in 6.541s
```

# 基本

この項目で扱う内容は次の ３点｡

- ルーティングで扱いたいページの追加
- ルーティングを管理するファイルの追加
- Vue.js 上でルーティングを扱うための 2つ のファイル修正

で、表にまとめると以下のようになる。

| ファイル  | 新規 / 修正 | 備考                         |
| --------- | ----------- | ---------------------------- |
| *.vue     | 新規        | ルーティングで扱いたいページ |
| router.js | 新規        | ルーティングを管理する       |
| main.js   | 修正        | ルーティングを扱うために修正 |
| App.vue   | 修正        | 同上                         |

では上から順に見ていく｡

## ページ( `*.vue` )の追加

ルーティングで扱いたいページを実装する｡
とは言っても、ここで扱う内容に特別なものはない｡単純に 単一コンポーネントでページを実装しただけである｡

```vue:追加したページ
<template>
  <div :class="$style.parent">
    <div :class="$style.child">
      <h1>Routing Test Page</h1>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Routing'
}
</script>

// スタイルは割愛
```


## router.js ファイルを新規作成

こちらはルーティングを管理するための JavaScript ファイル｡

```javascript:router.js
import Vue from 'vue'
import Router from 'vue-router'
import Home from './views/Home.vue'
import Routing from './views/Routing.vue'
//
// 他のコンポーネントは省略
//

Vue.use(Router)

export default new Router({
  // デフォルトの挙動では URL に `#` が含まれる.
  // URL から hash を取り除くには `mode:history` を指定する
  mode: 'history',                  
  base: process.env.BASE_URL,
  routes: [
    {
      path: '/',
      name: 'home',
      component: Home
    },
    //
    // 省略
    //
    {
      path: '/routing',
      name: 'routing',
      component: Routing 
    }
  ]
})
```

短いコードではあるがポイントがいくつかある。以下､順に見ていく｡

1. `vue-router` の `import`
   - [事前準備](#事前準備) で導入した `vue-router` を `import` している
   - ここは特筆すべきことはなく､単純に `vue-router` を使うための宣言である

2. `vue-router` の登録
   - `import` した `vue-router` を Vue で使えるように `Vue.use()` で指定する
   - これにより Vue アプリ上で `vue-router` を使ったルーティングが可能になる

3. `vue-router` のインスタンス生成と `export`
   - `vue-router` のインスタンスを生成し､それを `export` している
   - これにより 各コンポーネントでは 本 JS ファイル(`router.js` ) を `import` することなく **`this.$router` で参照** することができる

4. ルーティングの設定
   - `routes` プロパティを配列で定義し､その中に以下を設定することで path に応じたコンポーネントが呼び出される
   - ここの内容でルーティングのためのルールを設定することになる｡ 公式の [Vue Router](https://router.vuejs.org/) を充分に理解しておきたい


| 項目      | 説明                                            | 備考                                                         |
| --------- | ----------------------------------------------- | ------------------------------------------------------------ |
| path      | URL の path                                     | `<router-link>` で指定する文字列を間違えないように注意すること |
| name      | `<router-link>` や `router.push` で指定する名前 | 両者を使わないのであれば設定不要。だが設定しておいた方が良いと思う |
| component | 呼び出されるコンポーネント                      | `<router-view />` が配置された箇所でレンダリングされる       |


## main.js の修正

このファイルの修正は単純で､ アプリケーション上でルーティングを行うために､前項で作成した

- `router.js` の `import`
- Vue インスタンス生成時に `router.js` のインスタンスのセット

の 2点 を行っている｡

```javascript:main.js
import Vue from 'vue'
import App from './App.vue'

// ルーティングのために追加
import router from './router'

Vue.config.productionTip = false

new Vue({
  router, // ルーティングのために追加
  render: h => h(App),
}).$mount('#app')
```

## App.vue の修正

こちらは前項で挙げた `main.js` の

```javascript:main.js
new Vue({
  router, // ルーティングのために追加
  render: h => h(App),
}).$mount('#app')
```

の部分で読み込まれていて、アプリケーションの起点となるファイルである｡
この `App.vue` では、テンプレート部分で `router.js` で定義したルーティングルールとの紐付けを行っている｡

```vue:App.vue
<template>
  <div id="app">
    <div 
      id="nav" 
      class="tab-area-base">
      <ul class="tab-menu-base">
　　　　　<!-- `router.js` で定義したルーティングルールとの紐付けを行っている -->
        <li><router-link to="/">Home</router-link></li>
        <!-- *** -->
        <!-- 省略 -->
        <!-- *** -->
        <li><router-link to="/routing">Rounting Test</router-link></li>
      </ul>
    </div>
    <router-view />
  </div>
</template>

// ▼ ここから削除 ( ルーティングによって不要となるため )
//<script>
//import HelloWorld from './components/HelloWorld.vue'
//
//export default {
//  name: 'App',
//  components: {
//    HelloWorld
//  }
//}
//</script>
// ▲ ここまで削除

// スタイルは割愛
```

このファイルもいくつかポイントがあるので順に見ていく｡

1. `<template>` 部分
   - `<router-link>` 
     - 遷移先の設定を行っている
     - ここで `to=` で設定しているのが [router.js で設定](#routerjs-ファイルを新規作成) したパスとなる
     - 指定する文字列を間違うと **リンクをクリックしても遷移しない** うえに､**コンソール上にエラーも出ない** ため､意外と不具合の原因を探しづらいので注意されたい
   - `<router-view />`
     - ここの部分に `<router-link>` から `router.js` を経由して呼び出されたコンポーネントの内容が描画される
     - これが無いと コンポーネントを呼び出しているの描画されない ということになるので注意
2. `<script>` 部分
   - コード中のコメントにもあるとおり､ルーティングを使うことで本ファイルでのコンポーネントの呼び出しは不要になった



## 動作確認

ここまでルーティングの基本について見てきた｡
では実際にどう動くのかをキャプチャをもとに見ていく｡

### ルーティング前

[App.vue の修正前](#appvue-の修正) のコードにあるように､`HelloWorld.vue` がそのまま表示されている｡

![スクリーンショット 2020-09-13 1.13.51.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/90349717-cb1c-750b-6cf3-1164bdbb2a85.png)


### ルーティング後

1. 初期表示 or Home タブを選択
   ルーティングの実装後｡
   画面にはタブが表示され､ `Home` タブの URL である `http://localhost:8080` には初期画面として [ルーティング前](#ルーティング前) と同じ `HelloWorld.vue` の画面が表示されている｡
   ![スクリーンショット 2020-09-21 20.52.17.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/b9cf6778-253d-3bab-84d0-1120c70a197c.png)


2. Routing Test タブを選択
   `Routing Test` タブの URL である `http://localhost:8080/routing` では､新たに実装したページである `Routing.vue` の画面が表示されている｡
   ![706e67.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/cd4311e4-2235-5c5f-a3bf-c498d01fd001.png)


以上､ルーティングの基礎について見てきたが､決められたルールに沿うことで、少ない手間で URL を指定したページ遷移が実現できることがわかった｡
次の項目では **ルーティング時にパラメータの受け渡しを行う** 方法について見ていく。



# 進んだ使い方

`vue-router` を使ったルーティングでは **任意の値をパラメータとして受け渡す** ために

- URL のパターンマッチングによるページ遷移
- `router.push` を使ったページ遷移

といった方法が用意されている。

以下、それぞれの方法について見ていく。



# パターンマッチングによるページ遷移

URL のパターンマッチングによってパラメータの受け渡しを行うケース。ここで挙げる例は以下のとおり｡

- ユーザ情報の詳細を持つコンポーネント `UserDetail.vue` がある
- そのコンポーネントに対してユーザIDを URL で指定する
- 指定したユーザ情報の詳細を表示するページに遷移する



[基本](#基本) の項目でルーティングのための準備はできているので、本項では `router.js` への追記とユーザ情報の詳細を持つコンポーネント `UserDetail.vue` について扱う。



## router.js への追記

パターンマッチングでのページ遷移の実現にあたり、まずは `router.js` で **URL でのパラメータの受け渡し** のための設定を記述する。

具体的には以下のとおり。

```javascript:router.js
import Router from 'vue-router'
import UserDetail from './views/UserDetail.vue'

Vue.use(Router)

export default new Router({
    {
      path: '/users/:id',
      name: 'user-detail',
      component: UserDetail 
    }
  ]
})
```

ポイントは `path: '/users/:id'` の部分。 **`path` 内の URL に `:` を使用する** ことでパターンマッチングを実現する。
コンポーネントでは `$route.params` から、ここで指定したパラメータ名と **同じ名前でアクセス** することで値を取得できる。



## UserDetail.vue

`UserDetail.vue` は `router.js` で URL によるパターンマッチングでの遷移を行う際に指定されたコンポーネント。

前述の説明のとおり、下記のコードでは **`$route.params.id` とすることで `'/users/:id'` で指定した `id` の値を取得** している。

```vue:UserDetail.vue
<template>
  <div :class="$style.component">
    <h1>This page is user detail.</h1>
    <div
      :class="$style.userinfo">
      <table>
        <th :class="$style.item">
          ITEM
        </th>
        <th :class="$style.value">
          VALUE
        </th>
        <!-- users のリストにアクセスする際、インデックスは 0 からなので受け取った id の値から `-1` する -->
        <tr
          v-for="(value, name) in users[$route.params.id - 1]"
          :key="name">
          <td :class="$style.item">
            {{ name }}
          </td>
          <td :class="$style.value">
            {{ value }}
          </td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  name: 'UserDetail',
  data: function () {
    // 返却するオブジェクト users は本コンポーネントで表示するユーザ情報
    // 本来ならば DB 等で保持するのだが、今回は記事用のサンプルコードということでリストで持たせている
    return {
      users: [
        {
          id: 1,
          name: 'hogehoge',
          live: 'Japan Tokyo',
          phone: 'NNN-XXXX-HHHH',
          gender: 'male',
          mail: 'hogehoge@mail.com'
        },
        {
          id: 2,
          name: 'barbar',
          live: 'Japan Kanagawa',
          phone: 'NNN-XXXX-BBBB',
          gender: 'male',
          mail: 'barbar@mail.com'
        },
        {
          id: 3,
          name: 'piypiyo',
          live: 'Japan Kanagawa',
          phone: 'NNN-XXXX-PPPP',
          gender: 'female',
          mail: 'piypiyo@mail.com'
        },
        {
          id: 4,
          name: 'fugafuga',
          live: 'Japan Chiba',
          phone: 'NNN-XXXX-FFFF',
          gender: 'male',
          mail: 'fugafuga@mail.com'
        },
        {
          id: 5,
          name: 'varvar',
          live: 'Japan Saitama',
          phone: 'NNN-XXXX-VVVV',
          gender: 'female',
          mail: 'varvar@mail.com'
        }
      ],
    }
  }
}
</script>

// スタイルは割愛
```



## 動作確認

動作確認の結果が以下のキャプチャ。

URL に

- `http://localhost:8080/users/1` を指定することで `id: 1` のユーザ情報詳細
- `http://localhost:8080/users/4` を指定することで `id: 4` のユーザ情報詳細

が、ぞれぞれ表示されることが確認できた。

![スクリーンショット 2020-11-07 12.10.50.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/695cb711-d918-cf3e-3098-3273d1510acf.png)


![スクリーンショット 2020-11-07 12.11.14.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/14599a12-2ca6-4953-bab4-0e96962943fb.png)



# ボタンアクションによるページ遷移

前掲の [パターンマッチング](#パターンマッチングによるページ遷移) で **URL上でパラメータがセットされたケース** の例について触れた｡
ここではボタンアクションによる **パラメータの受け渡しを伴うページ遷移** について扱う｡

ここで挙げる例は以下のとおり｡

- ユーザのリストを表示するコンポーネント `UserList.vue` と、その親コンポーネント `Users.vue` がある
- コンポーネント ``UserList.vue`` でテーブルから任意のレコードを選択して `Show more seleted...` ボタンをクリックすることで
- 指定したユーザ情報の詳細を表示するコンポーネント `UserDetail.vue` を呼び出す( 遷移する )



## コードを見る前に

上記 3 つのコンポーネントの概要を表にすると以下のとおり｡

| コンポーネント | 概要                                 | 子コンポーネント | 備考                                                         |
| -------------- | ------------------------------------ | ---------------- | ------------------------------------------------------------ |
| Users.vue      | User 情報を持つ                      | UserList.vue     |                                                              |
| UserLive.vue   | User 情報をテーブルで表示する        | なし             |                                                              |
| UserDetails    | 指定された User 情報の詳細を表示する | なし             | [前項](#userdetailvue) で扱っているのでここではコードを扱わない |



このうち `Users.vue` と `UserDetail.vue` は同じデータオブジェクトをそれぞれのコンポーネント内で定義している｡
本来ならばこれらのデータは `DB` なり `localStorage` で持つなり､なんらかの手段でデータの共有化を図るべきなのだけれども､今回は **ルーティングが主題** であるためデータの持ち方については考慮外とした｡



## Users.vue

`Users.vue` はユーザの一覧をオブジェクトとして持ち、画面描画時に子コンポーネントである `UserList.vue` にユーザ情報を渡すだけの単純なもの。

- コンポーネントの親子関係
- 親コンポーネント → 子コンポーネントへのデータ授受

についても少し見てみたいと思い試してみた。
( コンポーネント間のデータのやりとりについては、別途記事を設けて見ていきたい )

```vue:Users.vue
<template>
  <div :class="$style.component">
    <UserList :properties="properties" />
  </div>
</template>

<script>
// @ を指定することで `/src` の代替となる
import UserList from '@/components/UserList.vue'

export default {
  name: 'Users',

  components: {
    UserList
  },
  data: function () {
    // ここで返却するデータは子コンポーネント `UserList.vue` で表示するユーザ情報
    // 本来ならば DB 等で保持するのだが、今回は記事用のサンプルコードということでリストで持たせている
    return {
      properties: {
        users: [
          {
            id: 1,
            name: 'hogehoge',
            live: 'Japan Tokyo',
            phone: 'NNN-XXXX-HHHH',
            gender: 'male',
            mail: 'hogehoge@mail.com'
          },
          {
            id: 2,
            name: 'barbar',
            live: 'Japan Kanagawa',
            phone: 'NNN-XXXX-BBBB',
            gender: 'male',
            mail: 'barbar@mail.com'
          },
          {
            id: 3,
            name: 'piypiyo',
            live: 'Japan Kanagawa',
            phone: 'NNN-XXXX-PPPP',
            gender: 'female',
            mail: 'piypiyo@mail.com'
          },
          {
            id: 4,
            name: 'fugafuga',
            live: 'Japan Chiba',
            phone: 'NNN-XXXX-FFFF',
            gender: 'male',
            mail: 'fugafuga@mail.com'
          },
          {
            id: 5,
            name: 'varvar',
            live: 'Japan Saitama',
            phone: 'NNN-XXXX-VVVV',
            gender: 'female',
            mail: 'varvar@mail.com'
          }
        ]
      }
    }
  }
}
</script>

// スタイルは割愛
```



## UserList.vue

`UserList.vue` は親コンポーネントである `Users.vue` から受け取ったデータをテーブルで表示するコンポーネント。

テーブルの実現には [Buefy の Table](https://buefy.org/documentation/table) を使用している。

```vue:UserList.vue
<template>
  <div>
    <h1>This page is user list.</h1>
    <div :class="$style.userlist">
      <!-- Buefy のテーブルを使って実現 -->
      <!-- https://buefy.org/documentation/table/ -->
      <b-table
        :data="properties.users"
        :columns="columns"
        :striped="true"
        :hoverable="true"
        :selected.sync="selected" />
    </div>
    <div :class="$style.showmore">
      <b-button
        type="is-info"
        @click="showMoreInformation">
        Show more selected...
      </b-button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'UserList',
  props: {
    properties: {
      type: Object,
      'default': () => { return null },
    }
  },
  data: function() {
    return { 
      // `selected`, `columns` は Buefy のテーブルを使用する際に必要なパラメータ
      // https://buefy.org/documentation/table/
      selected: null,
      columns: [
        {
          field: 'id',
          label: 'ID',
          width: '50',
          numeric: true
        },
        {
          field: 'name',
          label: 'NAME',
          width: '400',
          centered: true
        },
        {
          field: 'mail',
          label: 'MAIL',
          width: '400',
          centered: true
        },
      ]
    }
  },
  methods: {
    showMoreInformation: function() {
      // アロー関数で定義すると `this` で `selected` が参照できない｡
      // 詳細は https://qiita.com/_Keitaro_/items/d48733a19c10889e2365 を参照のこと｡
      if (!this.selected) {
        alert('No data selected...')
        return false
      }
      const selected = this.selected
      this.$router.push({
        name: 'user-detail',
        params: { id: selected['id'] }
      })
    }
  }
}
</script>

// スタイルは割愛
```

本コンポーネントのポイントはボタンクリック時に実行されるメソッドである `showMoreInformation` 。
レコードを選択した状態で `showMoreInformation` が実行されると `UserDetail.vue` コンポーネントが表示され、選択されたユーザ情報の詳細が表示される。

その動きを実現しているのが下記の部分。

```javascript
      const selected = this.selected
      this.$router.push({
        name: 'user-detail',
        params: { id: selected['id'] }
      })
```

`this.$router.push` の引数に `name` と `params` プロパティをセットすることでルーティングを実現させている。

ここで `name` と `params.id` は [パターンマッチング の router.js](#routerjs-への追記) で設定した

```javascript
    {
      path: '/users/:id',
      name: 'user-detail',
      component: UserDetail 
    }
```

の部分とリンクしている。
それぞれの相関を表にすると次のとおり。

|      | UserList.vue のコード             | router.js のコード |
| ---- | -------------------------------- | ----------------- |
| name | `name: 'user-detail'`            | `name: 'user-detail'` |
| id   | `params: { id: selected['id'] }` | `path: '/users/:id'` |


こうした `name` プロパティを利用したルーティングを **名前付きルート** というらしい。



## 動作確認

動作確認の結果が以下のキャプチャ。

- `User List` タブで表示されたテーブルから `id: 3` のレコードを選択して `Show more selected...` ボタンをクリック
- `UserDetail` タブに遷移して `id: 3` のユーザ情報詳細が表示される

ことが確認できた。

![スクリーンショット 2020-11-07 11.46.55.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/fc48cc26-4d91-a067-4371-35dc393318c0.png)

![スクリーンショット 2020-11-07 11.47.11.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/d770adaf-a6e7-1e7f-00bd-d18e89e77de4.png)



# まとめ
## [基本](#基本)
- ルーティングには `vue-router` を使うと簡単に実現できる
- ルーティングを管理するために `router.js` を実装して `main.js` で `import` することで利用する
- `App.vue` では以下を行ってうことで画面遷移を実現する
  - `router-link` で遷移先の指定
  - `router-view` で遷移先のページの表示

## [進んだ使い方](#進んだ使い方)
- ルーティング時に任意の値をパラメータとして渡すことができる
- そのための手段として以下を用いる
    - URL パラメータとして指定する
    - `router.push` 時にパラメータとして指定する
- `router.push` 時に遷移先として `name` を指定することを [名前付きルート](https://router.vuejs.org/ja/guide/essentials/named-routes.html) という



# ソースコード

今回の記事で動作確認に使用したコードは下記にアップしております｡
ご参考まで。
( ~~以下は ブランチのリンクですが､ master にもマージ済みです~~ )
( master は Vue v3.x 系に移行しました. 本記事の内容は下記ブランチでご確認ください )

- [ksh-fthr/vue-work > feature/#routing](https://github.com/ksh-fthr/vue-work/tree/feature/%23routing)


# 参考

## 公式

- [Vue Router](https://router.vuejs.org/)
- [Vue.js - ルーティング](https://jp.vuejs.org/v2/guide/routing.html)
    - [名前付きルート](https://router.vuejs.org/ja/guide/essentials/named-routes.html)
- [Vue.js - プラグイン](https://jp.vuejs.org/v2/guide/plugins.html)


[^1]: Vue.js　のコンポーネントを単独のファイルとして作成する機能<br>拡張子「.vue」のファイルのことで`<template>`, `<script>`, `<style>` のブロックで構成されている。

