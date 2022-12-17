# はじめに

[Vue.js の学習用リポジトリ](https://github.com/ksh-fthr/vue-work) で扱う Vue.js のバージョンを v2.x から v3.x にアップデート & [Vite](https://ja.vitejs.dev/) を導入したので、その際の手順を記録として残したいと思います。<br />
今更感はありますが、Vue.js のバージョンアップを今後考えておられる方々への一助となれば幸いです。

# 環境
## v2.x 
|         | Version | 備考                |
|---------|---------|---------------------|
| npm     | 6.14.15 | `npm -v` で確認     |
| node    | 14.17.6 | `node -v` で確認    |
| Vue     | 2.7.8   | package.json で指定 |
| Vue CLI | 5.0.1   | `vue -V` で確認     |

## v3.x
|         | Version | 備考                |
|---------|---------|---------------------|
| npm     | 8.19.2  | `npm -v` で確認     |
| node    | 18.12.0 | `node -v` で確認    |
| Vue     | ^3.1.0  | package.json で指定 |
| Vue CLI | 5.0.8   | `vue -V` で確認     |
| Vite    | ^3.2.2  | package.json で指定 |


# Vue2 -> Vue3 への移行

## 注意点

[Vue2 の構成](https://github.com/ksh-fthr/vue-work/tree/v2.x) のときは UI フレームワークとして [Buefy](https://buefy.org/) を導入していたのですが、[Buefy は Vue3 に対応しないことが明言されている](https://github.com/buefy/buefy#quick-start) ので Vue3 への移行に際し取り除きました。

参考:
- https://github.com/buefy/buefy/issues/2505#issuecomment-996986390
- https://github.com/buefy/buefy/issues/2505#issuecomment-997000720

## 移行手順
[移行ビルド->インストール](https://v3.ja.vuejs.org/guide/migration/migration-build.html#%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB) に沿って作業します。

**この手順を実施しているときの `npm`, `node` のバージョンは次のとおりです。**

|         | Version | 備考                |
|---------|---------|---------------------|
| npm     | 6.14.15 | `npm -v` で確認     |
| node    | 14.17.6 | `node -v` で確認    |


### 1. `vue upgrade` の実施
作業対象のリポジトリでは vue-cli を用いてプロジェクトを作成しておりますので、ここでは

> vue-cli を使用している場合: vue upgrade で最新の @vue/cli-service にアップグレードします。

に従いました。

### 2. package.json の編集
同じく移行ビルドに従い、package.json を編集しました。
変更点は以下のとおりです。

```json
diff --git a/package.json b/package.json
index 587b77f..cfc0c48 100644
--- a/package.json
+++ b/package.json
@@ -15,18 +15,18 @@
     "jsdoc": "3.6.11",
     "jsdoc-vuejs": "4.0.0",
     "serialize-javascript": "6.0.0",
-    "vue": "2.7.13",
+    "vue": "^3.1.0",
+    "@vue/compat": "^3.1.0",
     "vue-eslint-parser": "8.3.0",
-    "vue-loader": "15.10.0",
+    "vue-loader": "^16.0.0",
     "vue-router": "3.6.5",
-    "vue-template-compiler": "2.7.13",
     "vuex": "3.6.2"
   },
   "devDependencies": {
     "@vue/cli-plugin-babel": "5.0.8",
     "@vue/cli-plugin-eslint": "5.0.8",
     "@vue/cli-service": "5.0.8",
-    "buefy": "0.9.22",
+    "@vue/compiler-sfc": "^3.1.0",
     "eslint": "8.26.0",
     "eslint-config-prettier": "8.5.0",
     "eslint-config-vue": "2.0.2",
```

### 3. vue.config.js の編集
同じく移行ビルドに従い、vue.config.js を編集しました。
変更点は以下のとおりです。

```js
diff --git a/vue.config.js b/vue.config.js
index 22e96c9..6d0505e 100644
--- a/vue.config.js
+++ b/vue.config.js
@@ -1,4 +1,21 @@
 module.exports = {
+  chainWebpack: config => {
+    config.resolve.alias.set('vue', '@vue/compat')
+
+    config.module
+      .rule('vue')
+      .use('vue-loader')
+      .tap(options => {
+        return {
+          ...options,
+          compilerOptions: {
+            compatConfig: {
+              MODE: 2
+            }
+          }
+        }
+      })
+  },
   configureWebpack: {
     resolve: {
       alias: {
```

### 4. main.js の編集
先述しましたとおり、Buefy は Vue3 では非対応なので、main.js から該当する箇所を削除しました.
変更点は次のとおりです。

```js
diff --git a/src/main.js b/src/main.js
index a0a3d22..1246c41 100644
--- a/src/main.js
+++ b/src/main.js
@@ -1,7 +1,7 @@
 import Vue from 'vue'
-import Buefy from 'buefy'
-import 'buefy/dist/buefy.css'
-Vue.use(Buefy)
+//import Buefy from 'buefy'
+//import 'buefy/dist/buefy.css'
+//Vue.use(Buefy)

 import App from './App.vue'
 import router from './router'
```

### 5. Vue2 のコンパイル設定を削除(vue.config.js を再編集)
ここまでで `Vue2 -> Vue3` へ移行するための作業は完了しているのですが、この状態で `npm run serve` として Web アプリを起動させると次のエラーがでました。

```bash
# /path/to/repository/root は本リポジトリのルートディレクトリを示す
Module not found: Error: Can't resolve 'vue' in '/path/to/repository/root/src'
```

これを解決するために再度 `vue.config.js` を編集します。
変更点は次のとおりです。

```js
diff --git a/vue.config.js b/vue.config.js
index 6d0505e..8fe109e 100644
--- a/vue.config.js
+++ b/vue.config.js
@@ -16,12 +16,5 @@ module.exports = {
         }
       })
   },
-  configureWebpack: {
-    resolve: {
-      alias: {
-        vue$: 'vue/dist/vue.esm.js'
-      }
-    }
-  }
 }
```

### 6. Buefy 関連のコードを削除
ここでは詳細は割愛しますが、[Vue2 の構成](https://github.com/ksh-fthr/vue-work/tree/v2.x) で実装していた内容から Buefy 関連のコードを削除しました。
詳細にご興味あるかたは次項の Git ログをご参照ください。

### 補足
ここまでの変更点は Git ログとして確認できます。
ご興味あれば下記をご参照ください。

- [Vu2e -> Vue3 移行時のログ-1(wip, v3 に移行中.)](https://github.com/ksh-fthr/vue-work/commit/01688be60258bda77701adfcc1f63f1b157152c8?diff=split)
- [Vue2 -> Vue3 移行時のログ-2(v2 の頃に指定していたコンパイル設定を削除.)](https://github.com/ksh-fthr/vue-work/commit/bde2196e8fc734063dd14d48cc445c01b657f407?diff=split)
- [Vue2 -> Vue3 移行時のログ-3(Buefy 関連のコードを削除.)](https://github.com/ksh-fthr/vue-work/commit/03d922fa906982246e0e4ea9b35de8c0a0fa8b04?diff=split)
- [Vue2 -> Vue3 移行時のログ-4(Buefy の table コンポーネントで実装していたのを vue.js ネイティブの実装に変更.)](https://github.com/ksh-fthr/vue-work/commit/2f2d4d4a8e4a12e340ac8669ef60781d5c4bb9f1?diff=split)


# Vite の導入
ここまでで Vue3 への移行が完了しました。次に Vite を導入していきます。

## なぜ Vite を導入したのか
とにかく起動が速いという噂を聞くにつけ、試してみたいとう興味本位からです。
公式では次のページで Vite 導入について語られています。

- [なぜ Vite なのか](https://ja.vitejs.dev/guide/why.html#%E3%81%AA%E3%81%9C-vite-%E3%81%AA%E3%81%AE%E3%81%8B)


## 導入手順
では実際に行った手順を追っていきます。

**この手順を実施しているときの `npm`, `node` のバージョンは次のとおりです。**

|      | Version | 備考             |
|------|---------|------------------|
| npm  | 8.19.2  | `npm -v` で確認  |
| node | 18.12.0 | `node -v` で確認 |

(Vite が求める Node.js のバージョン)
[公式(日本語)](https://ja.vitejs.dev/guide/#%E6%9C%80%E5%88%9D%E3%81%AE-vite-%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%82%92%E7%94%9F%E6%88%90%E3%81%99%E3%82%8B)から転載.

> 互換性について
> 
> Vite は Node.js 14.18+、16+ のバージョンが必要です。ただし、一部のテンプレートではそれ以上のバージョンの Node.js を必要としますので、パッケージマネージャが警告を出した場合はアップグレードしてください。

### 1. Vite のインストール
次のコマンドでインストールを実施します。

```bash
$ npm i -D vite @vitejs/plugin-vue
```

### 2. package.json の編集
Vite を使用するべく `package.json` を編集します。
変更点は次のとおりです。

Vite の導入に当たり、`scripts` セクションで指定するコマンドが `vite` に置き換わっていること、`vue-cli` 関連のパッケージが削除されていることにご注目ください。

```json
diff --git a/package.json b/package.json
index cfc0c48..72bf14c 100644
--- a/package.json
+++ b/package.json
@@ -3,36 +3,50 @@
   "version": "0.1.0",
   "private": true,
   "scripts": {
-    "serve": "vue-cli-service serve",
-    "build": "vue-cli-service build",
-    "lint": "vue-cli-service lint",
+    "serve": "vite",
+    "build": "vite build",
+    "lint": "eslint --fix 'src/*.{js, vue}' && eslint --fix 'src/**/*.{js, vue}'",
     "jsdoc": "./node_modules/.bin/jsdoc -c ./jsdoc-conf.json"
   },
   "dependencies": {
-    "@vue/test-utils": "2.2.1",
     "axios": "0.27.2",
+    "core-js": "^3.8.3",
     "jest": "27.5.1",
     "jsdoc": "3.6.11",
     "jsdoc-vuejs": "4.0.0",
     "serialize-javascript": "6.0.0",
-    "vue": "^3.1.0",
-    "@vue/compat": "^3.1.0",
-    "vue-eslint-parser": "8.3.0",
-    "vue-loader": "^16.0.0",
-    "vue-router": "3.6.5",
-    "vuex": "3.6.2"
+    "vue": "^3.2.13",
+    "vue-router": "^4.1.6",
+    "vuex": "^4.1.0"
   },
   "devDependencies": {
-    "@vue/cli-plugin-babel": "5.0.8",
-    "@vue/cli-plugin-eslint": "5.0.8",
-    "@vue/cli-service": "5.0.8",
-    "@vue/compiler-sfc": "^3.1.0",
-    "eslint": "8.26.0",
-    "eslint-config-prettier": "8.5.0",
-    "eslint-config-vue": "2.0.2",
-    "eslint-plugin-prettier": "4.2.1",
-    "eslint-plugin-vue": "9.7.0",
-    "postcss": "8.4.18",
-    "webpack-dev-server": "4.11.1"
-  }
+    "@babel/core": "^7.12.16",
+    "@babel/eslint-parser": "^7.12.16",
+    "@vitejs/plugin-vue": "^3.2.0",
+    "eslint": "^7.32.0",
+    "eslint-config-prettier": "^8.5.0",
+    "eslint-plugin-vue": "^8.0.3",
+    "vite": "^3.2.2",
+    "vue-eslint-parser": "^9.1.0"
+  },
+  "eslintConfig": {
+    "root": true,
+    "env": {
+      "node": true
+    },
+    "extends": [
+      "plugin:vue/vue3-essential",
+      "eslint:recommended"
+    ],
+    "parserOptions": {
+      "parser": "@babel/eslint-parser"
+    },
+    "rules": {}
+  },
+  "browserslist": [
+    "> 1%",
+    "last 2 versions",
+    "not dead",
+    "not ie 11"
+  ]
 }
```

### 2. vite.config.js の作成
Vite の導入にともない設定ファイルを作成します。
内容は次のとおりです。

```js
diff --git a/vite.config.js b/vite.config.js
new file mode 100644
index 0000000..9e60e4d
--- /dev/null
+++ b/vite.config.js
@@ -0,0 +1,15 @@
+import vue from '@vitejs/plugin-vue'
+import path from 'path'
+
+export default {
+  plugins: [vue()],
+  server: {
+    port: 8080
+  },
+  resolve: {
+    alias: {
+      '@': path.resolve(__dirname, './src')
+    }
+  },
+}
+
```

### 3. index.html の編集
Vite の導入にともない index.html も Vite の作法にそって修正しました。
変更点は index.html の移動と `<link>` タグの修正、ならびに `<scritp>` タグの追加です。

1. index.html の移動
```bash
$ pwd
# /path/to/repository/root は本リポジトリのルートディレクトリを示す
/path/to/repository/root
#
# public 配下にある index.html をリポジトリルート直下に移動.
$ mv public/index.html index.html
```

2. index.html の編集

```html
diff --git a/public/index.html b/index.html
similarity index 83%
rename from public/index.html
rename to index.html
index 6184529..9fad8df 100644
--- a/public/index.html
+++ b/index.html
@@ -4,7 +4,7 @@
     <meta charset="utf-8">
     <meta http-equiv="X-UA-Compatible" content="IE=edge">
     <meta name="viewport" content="width=device-width,initial-scale=1.0">
-    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
+    <link rel="icon" href="/favicon.ico">
     <title>vue-work</title>
   </head>
   <body>
@@ -12,6 +12,7 @@
       <strong>We're sorry but vue-work doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
     </noscript>
     <div id="app"></div>
+    <script type="module" src="/src/main.js"></script>
     <!-- built files will be auto injected -->
   </body>
 </html>
```

### 4. vue.config.js の削除
`Vue2 -> Vue3` の移行で編集した `vue.config.js` ですが、Vite の導入により `vuew.config.js` が不要になりました。
ファイルそのものが不要ですので削除します。

### 5. vue-router, vuex のバージョンアップに伴う修正
Vite の導入にあわせて `vue-router@4.x系`, `vuex@4.x 系` を導入しました。
これに伴い `router.js`, `store.js`, `main.js` の編集が発生しましたので、その内容を示します。

#### router.js の編集
変更点は次のとおりです。

```js
diff --git a/src/router.js b/src/router.js
index 82eeb95..14ea5a0 100644
--- a/src/router.js
+++ b/src/router.js
@@ -1,5 +1,4 @@
-import Vue from 'vue'
-import Router from 'vue-router'
+import { createRouter, createWebHistory } from 'vue-router'
 import Home from './views/Home.vue'
 import Users from './views/Users.vue'
 import LifeCycle from './views/LifeCycle.vue'
@@ -7,44 +6,43 @@ import LifeCycle2 from './views/LifeCycle2.vue'
 import UserDetail from './views/UserDetail.vue'
 import Routing from './views/Routing.vue'

-Vue.use(Router)
+const routes = [
+  {
+    path: '/',
+    name: 'home',
+    component: Home
+  },
+  {
+    path: '/routing',
+    name: 'routing',
+    component: Routing
+  },
+  {
+    path: '/life-cycle',
+    name: 'llfe-cycle',
+    component: LifeCycle
+  },
+  {
+    path: '/life-cycle2',
+    name: 'llfe-cycle2',
+    component: LifeCycle2
+  },
+  {
+    path: '/users',
+    name: 'users',
+    component: Users
+  },
+  {
+    path: '/users/:id',
+    name: 'user-detail',
+    component: UserDetail
+  },
+]

-export default new Router({
-  // デフォルトの挙動では URL に `#` が含まれる.
-  // URL から hash を取り除くには `mode:history` を指定する
-  mode: 'history',
-  base: process.env.BASE_URL,
-  routes: [
-    {
-      path: '/',
-      name: 'home',
-      component: Home
-    },
-    {
-      path: '/routing',
-      name: 'routing',
-      component: Routing
-    },
-    {
-      path: '/life-cycle',
-      name: 'llfe-cycle',
-      component: LifeCycle
-    },
-    {
-      path: '/life-cycle2',
-      name: 'llfe-cycle2',
-      component: LifeCycle2
-    },
-    {
-      path: '/users',
-      name: 'users',
-      component: Users
-    },
-    {
-      path: '/users/:id',
-      name: 'user-detail',
-      component: UserDetail
-    },
-  ]
+const router = createRouter({
+  history: createWebHistory(),
+  routes,
 })

+export default router
+
```

#### store.js の編集
変更点は次のとおりです。

```js
diff --git a/src/store.js b/src/store.js
index e7a3696..9a5eb75 100644
--- a/src/store.js
+++ b/src/store.js
@@ -1,17 +1,10 @@
-import Vue from 'vue'
-import Vuex from 'vuex'
-
-Vue.use(Vuex)
-
-export default new Vuex.Store({
-  state: {
-
-  },
-  mutations: {
-
-  },
-  actions: {
+import { createStore } from 'vuex'

+export const store = createStore({
+  state() {
+    return {
+      count: 1
+    }
   }
 })
```

#### main.js の編集
変更点は次のとおりです。

```js
diff --git a/src/main.js b/src/main.js
index a933e41..4f442fb 100644
--- a/src/main.js
+++ b/src/main.js
@@ -1,14 +1,10 @@
-import Vue from 'vue'
-
+import { createApp } from 'vue'
 import App from './App.vue'
 import router from './router'
-import store from './store'
-
-Vue.config.productionTip = false
+import { store } from './store'

-new Vue({
-  router,

-  store,
-  render: h => h(App)
-}).$mount('#app')
+const app = createApp(App)
+app.use(router)
+app.use(store)
+app.mount('#app')
```

### 補足
Vite 導入手順につきましても、ここまでの変更点を Git ログとして確認できます。
ご興味あれば下記をご参照ください。

- [Vite の導入-1(wip, vite に移行中.)](https://github.com/ksh-fthr/vue-work/commit/6cca4cac0192388f20dc032e630c5e5c85c83701)
- [Vite の導入-2(vite 移行 & 起動確認まで OK.)](https://github.com/ksh-fthr/vue-work/commit/cbb144509c761a24d2ef40a9898238ec788b3c2a)
- [Vite の導入-3(eslint によるチェック& prettier 整形を導入.)](https://github.com/ksh-fthr/vue-work/commit/bf862b961f436f367f94fb4692642bde46a333ac)
- [Vite の導入-4(vue-cli で使用していた設定ファイルを削除.)](https://github.com/ksh-fthr/vue-work/commit/45c792f71788d7119c7c71ccaaa227a11748276b)

## Vite 導入前後の起動時間
`Vue2 + vue-cli` と `Vue3 + Vite` での比較ですので参考程度となりますが、両者の Web アプリ起動時間を示します。

### Vue2 + vue-cli で起動

```bash
$ npm run serve
 DONE  Compiled successfully in 6913ms                                                       15:04:08


  App running at:
  - Local:   http://localhost:8080/
  - Network: http://192.168.2.111:8080/

  Note that the development build is not optimized.
  To create a production build, run npm run build.
```

### Vue3 + Vite で起動

```bash
$ npm run serve
  VITE v3.2.2  ready in 431 ms

  ➜  Local:   http://localhost:8080/
  ➜  Network: use --host to expose
```

### 起動時間比較

|                | 起動時間  |
|----------------|---------:|
| Vue2 + vue-cli | 6,913ms  |
| Vue3 + Vite    |   431ms  |

前述のとおり Vue2 と Vue3 の違い、更には Vue2 は Buefy 込み、という違いがありますが vue-cli と Vite とでは 10倍 以上の差がでました。
Vite は速い、ということが実感できました。


# まとめにかえて
手間取ったところ。

- Vue3 への移行について
  - Vue2 で Buefy を入れていたことが足かせになった
  - が、それも Buefy を取り除き、該当箇所も見直すことで対応できた
- Vite への移行について
  - `vue-router` や `vuex` を `4.x系` に更新したことで、`router.js`, `store.js` への修正、それに伴う `main.js` への修正が必要になった。

それぞれの移行作業については、公式サイト含め、参考にしたサイトに非常にお世話になりました。
この場にて深く感謝申し上げます。


# 参考
- [Vue.js-公式(日本語), 移行ビルド](https://v3.ja.vuejs.org/guide/migration/migration-build.html#%E7%A7%BB%E8%A1%8C%E3%83%92%E3%82%99%E3%83%AB%E3%83%88%E3%82%99)
- [Vite-公式(日本語)](https://ja.vitejs.dev/)
- [vue-cliをviteに移行する](https://zenn.dev/kazuwombat/articles/9357f6b1ccca8c)

# ソースコード
今回の記事で動作確認に使用したコードは下記にアップしてあるのでご参考まで。

- [Vue v2.x の構成](https://github.com/ksh-fthr/vue-work/tree/v2.x)
- [Vue v3.x の構成](https://github.com/ksh-fthr/vue-work/tree/v3.x)

