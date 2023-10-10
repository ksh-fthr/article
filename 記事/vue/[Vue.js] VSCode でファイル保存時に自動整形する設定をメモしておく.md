# はじめに
タイトルのとおり｡
Vue.js + VSCode の環境で､ファイル保存時に自動整形する際の設定で時間がかかったのでメモを残します｡
あくまで自分のためのメモなので細かい情報とか説明は載せていないです｡

# 環境
|          | バージョン            | 備考 |
| -------- | ------------------ | --- |
| macOS    | 10.14.x ( Mojave ) |     |
| Node.js  | v10.16.3           |     |
| npm      | v6.9.0             |     |
| yarn     | v1.21.1            |     |
| Vue.js   | v2.6.10            |     |

# 設定
## package.json
```javascript:package.json
{
  "name": "hogehoge",
  "private": true,
  "dependencies": {
    "vue": "2.6.10",
    "vue-eslint-parser": "6.0.4",
    "vue-loader": "15.7.1",
    "vue-router": "3.0.7",
    "vue-template-compiler": "2.6.10",
    "vuex": "3.1.1"
  },
  "devDependencies": {
    "eslint": "6.8.0",
    "eslint-config-prettier": "6.9.0",
    "eslint-plugin-prettier": "3.1.2",
    "eslint-plugin-vue": "6.1.2",
    "webpack-dev-server": "3.7.2"
  }
}
```

上記を設定した状態で `npm install` or `yarn install` で必要な npm モジュールが環境にインストールされる｡


## .eslintrc.js
```javascript:.eslintrc.js
module.exports = {
  plugins: [
    'vue'
  ],
  extends: [
    'eslint:recommended',
    'plugin:vue/recommended'
  ],
  rules: {
    'vue/html-closing-bracket-newline': [2, {'multiline': 'never'}],
    'no-extra-parens': 1,
    'no-multi-spaces': 2,
    'no-multiple-empty-lines': [2, {'max': 1}],
    'func-call-spacing': [2, 'never'],
    'no-unneeded-ternary': 2,
    'semi': [2, 'never'],
    'quotes': [2, 'single'],
    'no-var': 2,
    'indent': [2, 2],
    'space-in-parens': [2, 'never'],
    'no-console': 0,
    'comma-spacing': 2,
    'computed-property-spacing': 2,
    'key-spacing': 2,
    'keyword-spacing': 2,
  }
}

```
* [Configuring ESLint](https://eslint.org/docs/user-guide/configuring)
* [eslint-plugin-vue](https://github.com/vuejs/eslint-plugin-vue)
* [eslint-plugin-vue > Available rules](https://eslint.vuejs.org/rules/)

## settings.json
```javascript:settings.json
{
  "javascript.format.insertSpaceBeforeFunctionParenthesis": true,
  "typescript.format.insertSpaceBeforeFunctionParenthesis": true,
  "editor.tabSize": 2,
  "editor.formatOnSave": false,
  "eslint.enable": true,
  "files.associations": {
    "*.vue": "vue"
  },
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "vue"
  ],
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```

あ､一応上記の設定にあたっての備忘録がてらに｡｡｡
最初､[こちら](https://qiita.com/diggy-mo/items/bb01bcb54237f16bb008) を参考に設定していたのだが､ `eslint.validate` の設定で

```javascript:ワーニング箇所
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    {
      "language": "vue",
      "autoFix": true,
    },
  ],
```

としていたところ､

```
Auto Fix is enabled by default. Use the single string form.
```

というワーニングが発生していた.
ググってみたところ [ESLint not working in VS CODE?](https://dev.to/pixari/eslint-not-working-in-vs-code-5g4d) で Tips が紹介されていたので､それを元に修正して前掲の形にしたところ､ワーニングは解消された｡

あと [こちら](https://eslint.vuejs.org/user-guide/#editor-integrations) にも当該ワーニング箇所の記述方法が示されていた


# 参考
* [もうprettierで消耗したくない人へのvueでのeslint設定](https://qiita.com/diggy-mo/items/bb01bcb54237f16bb008)
* [ESLint not working in VS CODE?](https://dev.to/pixari/eslint-not-working-in-vs-code-5g4d)
* [Configuring ESLint](https://eslint.org/docs/user-guide/configuring)
* [eslint-plugin-vue](https://eslint.vuejs.org/)
    * [eslint-plugin-vue( GitHub )](https://github.com/vuejs/eslint-plugin-vue)
    * [eslint-plugin-vue > Available rules](https://eslint.vuejs.org/rules/)
