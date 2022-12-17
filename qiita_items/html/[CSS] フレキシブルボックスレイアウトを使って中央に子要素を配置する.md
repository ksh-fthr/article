親要素に対して子要素を中央に配置したいとき、 **CSS フレキシブルボックスレイアウト** という仕様が便利だ。
本記事ではこの仕様を利用した「子要素の中央配置」の方法について触れる。

## 子要素の中央配置を実現するには

親要素に対して次の表に示す CSS プロパティと値を設定すれば良い。

| CSS プロパティ　　     | 役割                  | 値    |説明                       |
|:-----------------|:----------------------|:-----|:--------------------------|
| [display](https://developer.mozilla.org/ja/docs/Web/CSS/display)          |要素の表示方法を定義する    |flex |親要素をフレキシブルなコンテナにする|
| [justify-content](https://developer.mozilla.org/ja/docs/Web/CSS/justify-content)  |水平方向の配置方法を定義する|center|子要素を横中央位置を基準に配置する|
| [align-items](https://developer.mozilla.org/ja/docs/Web/CSS/align-items)      |垂直方向の配置方法を定義する|center|子要素を縦中央位置を基準に配置する|

## サンプルコード

以下に示すのは上記のプロパティを使用したサンプルコード。
画面全体に配置した ```div``` 要素に対して、子要素となる ```div``` 要素を中央に配置するだけの単純な例だが効果は確認できる。

* index.html

```html:index.html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <title>CSS フレキシブルレイアウトを使って中央に子要素を配置する</title>
    <link rel="stylesheet" href="./index.css">
  </head>

  <body>
    <div class="parent">
      <div class="child">
        <label class="text">中央に配置する div 要素</label>
      </div>
    </div>
  </body>
</html>
```

* index.css

```css:index.css
html {
  /* 子孫の要素で height: 100%; でウィンドウの高さを表現するためにココで 100% を指定する */
  height: 100%;
  width: 100%;
}

body {
  /* 子孫の要素で height: 100%; でウィンドウの高さを表現するためにココで 100% を指定する */
  height: 100%;
  width: 100%;

  /* ウィンドウ外周のマージンをなくす */
  margin: 0;
}

.parent {
  height: 100%;
  width: 100%;
  background: rgba(200, 230, 240, 1.0);

  /* 子要素を中央に配置する */
  display: flex;
  justify-content: center;
  align-items: center;
}

.child {
  overflow: hidden;
  height: 250px;
  width: 250px;
  background-color: rgb(229, 241, 247);
  border-radius: 15px;
  box-shadow: 10px 10px 10px rgba(0,0,0,0.4);
  border: solid 1px rgb(0, 0, 0);

  /* 子要素を中央に配置する */
  display: flex;
  justify-content: center;
  align-items: center;
}
```

## ブラウザで表示を確認

<img width="709" alt="スクリーンショット 2018-09-15 22.05.41.png" src="https://qiita-image-store.s3.amazonaws.com/0/193342/47f09566-0562-c064-5a43-62c2c1ab2ceb.png">

このとおり、親要素である ```<div class="parent">``` に対して、子要素の ```<div class="child">``` が中央に配置されているのが確認できた。


## 参考

* [[MDN] CSS flexible box の利用](https://developer.mozilla.org/ja/docs/Web/CSS/CSS_Flexible_Box_Layout/Using_CSS_flexible_boxes)
* [[MDN] display](https://developer.mozilla.org/ja/docs/Web/CSS/display)
* [[MDN] justify-content](https://developer.mozilla.org/ja/docs/Web/CSS/justify-content)
* [[MDN] align-items](https://developer.mozilla.org/ja/docs/Web/CSS/align-items)
