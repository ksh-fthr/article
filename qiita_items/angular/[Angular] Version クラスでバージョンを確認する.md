Version クラスを利用すると使用している Angular のバージョンを確認できる。
Version クラスについては [本家の API リファレンス](https://angular.io/api/core/Version) を参照。

## バージョン確認のためのコード

### クラス
```typescript:app.component.ts
import { Component, AfterContentInit, AfterContentChecked, ContentChild } from '@angular/core';

// Angular のバージョン情報を取得するための import
import { VERSION } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {

  /**
   * Angular のバージョン情報を確認するためのプロパティ
   *
   * @private
   * @memberof AppComponent
   */
  public versionInfo = VERSION;

  constructor() {}
}
```

### テンプレート

```html:app.component.html
<div class="informationArea">
  <table class="informationTable">
    <caption>
      使用している Angular のバージョンは...
    </caption>
    <tr>
      <th>full</th>
      <th>major</th>
      <th>minor</th>
      <th>patch</th>
    </tr>
    <tr>
      <td>{{versionInfo.full}}</td>
      <td>{{versionInfo.major}}</td>
      <td>{{versionInfo.minor}}</td>
      <td>{{versionInfo.patch}}</td>
    </tr>
  </table>
</div>
```

### 結果

![angular_version.png](https://qiita-image-store.s3.amazonaws.com/0/193342/453d1702-260f-1fc3-9bf4-fcb8fce399ea.png)

Angular のバージョンが **4.4.6** であることが確認できた。


## angular/cli で確認

angular/cli のバージョンも確認してみると...

```:バージョン確認
$ ng -v
    _                      _                 ____ _     ___
   / \   _ __   __ _ _   _| | __ _ _ __     / ___| |   |_ _|
  / △ \ | '_ \ / _` | | | | |/ _` | '__|   | |   | |    | |
 / ___ \| | | | (_| | |_| | | (_| | |      | |___| |___ | |
/_/   \_\_| |_|\__, |\__,_|_|\__,_|_|       \____|_____|___|
               |___/
@angular/cli: 1.3.0
node: 9.2.1
os: win32 x64
@angular/animations: 4.4.6
@angular/common: 4.4.6
@angular/compiler: 4.4.6
@angular/core: 4.4.6
@angular/forms: 4.4.6
@angular/http: 4.4.6
@angular/platform-browser: 4.4.6
@angular/platform-browser-dynamic: 4.4.6
@angular/router: 4.4.6
@angular/cli: 1.3.0
@angular/compiler-cli: 4.4.6
@angular/language-service: 4.4.6
```

Version クラスを使った確認結果と同じバージョンであることが確認できた。

## 使いどころ

アプリで使用している Angular のバージョンを画面のフッターに明示したい､といったケースにおいて､この方法だと動的にバージョンを取得できるので便利だと思う｡

## 参考
* [本家の API リファレンス - Version](https://angular.io/api/core/Version)
