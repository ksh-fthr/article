フロントエンドからバックエンドに対して、同一ホストだがポートが異なる構成で疎通したいケースがある。
たとえば

* フロントエンドが ```localhost:4200/```
* バックエンドが ```localhost:3000/```

といったケースである。
~~Angular では~~ 通常、こうしたケースだと CORS によるエラーが発生してしまうのだが、Angular では Angular CLI でプロキシ設定ファイルを用意することで対応できる。

## この記事を実施した環境

* Windows10 Home 64bit
* node v8.2.1
* npm v4.0.5
* angular/cli v1.2.6
* Angular v4.3.2

## プロキシ設定ファイルの用意(proxy.conf.json)

次の例だとフロントエンドの 「/app」にリクエストがきた場合、バックエンドの「localhost:3000」にフォワードされる設定となる。

```json:proxy.conf.json
{
  "/app": {
    "target": "http://localhost:3000",
    "pathRewrite": {"^/app": ""}
  }
}
```

## package.json の編集
プロキシ設定ファイルを用意したら、それを使用してアプリケーションを起動する設定を package.json に記載する。
具体的には、次の「start」の内容を package.json の「scripts」の要素に記載する。

```json:package.json
{
  "scripts": {
    "start": "ng serve --proxy-config proxy.conf.json"
  }
}
```

## プロキシ設定を反映させてアプリケーションを起動

前項までの設定を終えたら、次のコマンドを実行することでプロキシ設定を反映させた状態でアプリケーションが起動する。

```js
// プロキシ設定を反映させる場合
$ npm run start
```

## プロキシ設定を反映させたくない場合

プロキシ設定を反映させたくない場合、直接 ng serve でアプリケーションを起動させれば良い。

```js
// プロキシ設定を反映させたくない場合
$ ng serve
```

## 参考

* [Proxy To Backend](https://github.com/angular/angular-cli/blob/master/docs/documentation/stories/proxy.md)
* [webpack dev server#proxy](https://github.com/webpack/docs/wiki/webpack-dev-server#proxy)
