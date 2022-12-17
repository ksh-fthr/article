タイトルの環境でハマったので備忘録として残す。
同じ現象でつまづいた方の一助となれば。。。

# 前提

* Vue CLI でプロジェクトを作成していること
* 単一ファイルコンポーネントであること（[^1]）
* 動作確認は ```$ npm run serve``` で起動した環境で行なっている

[^1]: Vue.js　のコンポーネントを単独のファイルとして作成する機能<br?>拡張子「.vue」のファイルのことで`<template>`, `<script>`, `<style>` のブロックで構成されている。

# 環境
| ツール | Version | 備考 |
|----|-----|--------|
| Vue | 2.5.17 | |
| Vue CLI | 3.0.5 | |
| Buefy | 0.7.0 | Vue.js 用の UI コンポーネント, MIT ライセンス, [公式はこちら](https://buefy.org/) |

# 本題の前に
## Buefy とは
Buefy は Vue.js 用の UI コンポーネント。
[公式](https://buefy.org/) をみると上記がそのまま書かれている。以下は公式のトップページから拝借したもの。

> Lightweight UI components for Vue.js based on Bulma

ライセンスは [GitHub - buefy](https://github.com/buefy/buefy) から MIT とわかる。

## Get Started

公式の [Get Started](https://buefy.org/documentation/start) に沿って進めればすぐに利用できる。

# 本題
## 公式の Modal サンプルコード実行時にエラーが発生

さて、本題のモーダルについて。
Buefy ではモーダルの実現についても公式ページの [Modal](https://buefy.org/documentation/modal) にサンプルコードが載っている。
で、以下のサンプルコードをそのまま使ってみたら

```vue:公式のModalサンプルコード(LaunchComponentModal)
<template>
    <section>
        <button class="button is-primary is-medium"
            @click="isComponentModalActive = true">
            Launch component modal
        </button>

        <b-modal :active.sync="isComponentModalActive" has-modal-card>
            <modal-form v-bind="formProps"></modal-form>
        </b-modal>
    </section>
</template>

<script>
    const ModalForm = {
        props: ['email', 'password'],
        template: `
            <form action="">
                <div class="modal-card" style="width: auto">
                    <header class="modal-card-head">
                        <p class="modal-card-title">Login</p>
                    </header>
                    <section class="modal-card-body">
                        <b-field label="Email">
                            <b-input
                                type="email"
                                :value="email"
                                placeholder="Your email"
                                required>
                            </b-input>
                        </b-field>

                        <b-field label="Password">
                            <b-input
                                type="password"
                                :value="password"
                                password-reveal
                                placeholder="Your password"
                                required>
                            </b-input>
                        </b-field>

                        <b-checkbox>Remember me</b-checkbox>
                    </section>
                    <footer class="modal-card-foot">
                        <button class="button" type="button" @click="$parent.close()">Close</button>
                        <button class="button is-primary">Login</button>
                    </footer>
                </div>
            </form>
        `
    }

    export default {
        components: {
            ModalForm
        },
        data() {
            return {
                isComponentModalActive: false,
                formProps: {
                    email: 'evan@you.com',
                    password: 'testing'
                }
            }
        }
    }
</script>
```

ブラウザで確認したときに次のエラーが発生し、オーバーレイだけが表示されてモーダルが表示されなかった。
このとき動作確認は [前述](#前提) のとおり ```$npm run serve``` で起動した環境で行なっている。

### 発生したエラー

```vue:発生したエラー
vue.runtime.esm.js?2b0e:587 [Vue warn]: You are using the runtime-only build of Vue where the template compiler is not available. Either pre-compile the templates into render functions, or use the compiler-included build.

found in

---> <ModalForm>
       <BModal>
         <BuefyParts> at src/components/BuefyParts.vue
           <SampleBuefy> at src/views/SampleBuefy.vue
             <App> at src/App.vue
               <Root>
```

### エラー発生時の画面
オーバーレイだけが表示され、肝心のモーダルは表示されない。
右側のコンソールには [発生したエラー](#発生したエラー) で記したエラーが表示されている。

<img width="1419" alt="modal-error.png" src="https://qiita-image-store.s3.amazonaws.com/0/193342/d58114ac-8bdd-db16-db09-f2974e48f4a1.png">


## 解決方法

[Vue CLI の issue #2359](https://github.com/vuejs/vue-cli/issues/2359) に記載があった。該当するコメントは [こちら](https://github.com/vuejs/vue-cli/issues/2359#issuecomment-419138167)。

`vue.config.js` は Vue CLI で作成したプロジェクトにはデフォルトで生成されていないので、自分で新規に作成する。
で、前傾のコメントのとおり `vue.config.js` に下記を記載する。

### vue.config.js

```javascript:vue.config.js
module.exports = {
  configureWebpack: {
    resolve: {
      alias: {
        vue$: 'vue/dist/vue.esm.js'
      }
    }
  }
}
```

### 正常にモーダルが表示された画面

これで `$npm run serve` で起動し直して確認すると、問題のエラーは発生せずにオーバーレイ、モーダル共に正常に表示された。

<img width="1420" alt="modal-normal.png" src="https://qiita-image-store.s3.amazonaws.com/0/193342/afbc2c38-f582-5fc6-d046-17998f8204d8.png">


# 参考
* [Vue CLI 'webpack' template doesn't work out of the box](https://github.com/vuejs/vue-cli/issues/2359) 
* [公式 -> インストール -> ランタイム + コンパイラとランタイム限定の違い](https://jp.vuejs.org/v2/guide/installation.html#ランタイム-コンパイラとランタイム限定の違い)
* [Rails5.1でVue.jsで単一ファイルコンポーネントのエラーがでる(省略)](https://qiita.com/magaya0403/items/3fbe9aa20c6a66b76662)


# ソースコード

確認に使ったソースコードは [こちら](https://github.com/ksh-fthr/vue-work/tree/feat_buefy) にアップしてあるのでご参考まで。
( [vue.config.js](https://github.com/ksh-fthr/vue-work/blob/feat_buefy/vue.config.js) と [BuefyParts.vue](https://github.com/ksh-fthr/vue-work/blob/feat_buefy/src/components/BuefyParts.vue) が今回の内容に該当するコード )
