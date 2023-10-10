## 想定するケース

* HogeService というクラスがある
* HogeService のメソッド: exec() は他のクラスのインスタンスを引数で受け取る
* HogeService は exec() において、引数で渡されたインスタンスのメソッドを実行する
* ただし HogeService のメソッドに引数でセットされるインスタンスのクラスは固定ではない

つまり次のようなケースを想定する。

  * HogeService

    ```typescript:HogeService
    export class HogeService {
      public exec(instance: any) {
        instance.exec();
      }
    }
    ```

  * 引数で渡されるインスタンスのクラス-1 ( Bar1Service )

    ```typescript:Bar1Service
    export class Bar1Service {
      public exec() {
        console.log('Bar1Service-exec()');
      }
    }
    ```

  * 引数で渡されるインスタンスのクラス-2 ( Bar2Service )

    ```typescript:Bar2Service
    export class Bar2Service {
      public exec() {
        console.log('Bar2Service-exec()');
      }
    }
    ```

HogeService のインスタンスメソッド: exec() は Bar1Service、もしくは Bar2Service のインスタンスを受け取る。
このとき HogeService が受け取るインスタンスの型を ```any``` ではなく ```ジェネリック``` で受け取りたいのだが、単純に HogeService のインスタンスメソッドで

  * 単純なジェネリクスの指定

    ```typescript:HogeService
    export class HogeService {
      public exec<T>(instance: T) {
        instance.exec();
      }
    }
    ```

とするだけでは

  * 文法エラー

    ```:エラー
    [ts] プロパティ 'exec' は型 'T' に存在しません。
    ```

が出てしまう。
本記事ではこのエラーが出ずにジェネリックでインスタンスを受け取り、そのインスタンスのメソッドを使用する方法について提示する。

## 解決方法

解決方法は以下のとおり。

* 引数として渡されるクラス
  * **抽象クラスを用意**する
  * 抽象クラスを継承する
* インスタンスを受け取る側のクラス
  * メソッド定義時、 **ジェネリックを抽象クラスを拡張した形式で指定**する

これでジェネリックでインスタンスを受け取り、そのインスタンスのメソッドの実行を実現できる。
以下、実際にコードを記述する。


## 引数として渡されるクラス

### 抽象クラスを用意する

```typescript:AbstractBar
export abstract class AbstractBarService {
  public abstract exec();
}
```

### 抽象クラスを継承する

```typescript:Bar1Service
import { Injectable } from '@angular/core';
import { AbstractBarService } from './abstract-bar.service';

@Injectable()
export class Bar1Service extends AbstractBarService{

  constructor() {
    super();
  }

  public exec() {
    console.log('Bar1Service-exec()');
  }
}
```

```typescript:Bar2Service
import { Injectable } from '@angular/core';
import { AbstractBarService } from './abstract-bar.service';

@Injectable()
export class Bar2Service extends AbstractBarService {

  constructor() {
    super();
  }

  public exec() {
    console.log('Bar2Service-exec()');
  }
}
```

## インスタンスを受け取る側のクラス

### メソッド定義時、ジェネリックを抽象クラスを拡張した形式で指定する

```typescript:HogeService
import { Injectable } from '@angular/core';
import { AbstractBarService } from './abstract-bar.service';

@Injectable()
export class HogeService {

  constructor() { }

  public exec<T extends AbstractBarService>(instance: T) {
    instance.exec();
  }
}
```

## まとめ

以上で掲題の「TypeScript のジェネリックで、引数で渡されたインスタンスのメソッドを使いたい」が実現できた。
本文の繰り返しとなるが、ポイントは

* 引数となるクラス側
  * **抽象クラスを用意**する
* 引数を受け取るクラス側
  * メソッド定義時、**ジェネリックを抽象クラスを拡張した形式で指定**する

である。
これで TypeScript による型定義の恩恵を受けられるし、コード補間機能 ( Visual Studio Code で確認 ) の恩恵も受けられる。

## ソースコード

今回の記事で作成したコードは [こちら](https://github.com/ksh-fthr/angular-work/tree/feat_generics/src/app/service) にアップしてあるのでご参考まで。


## 捕捉
### @pCYSl5EDgo 様からのコメントより転記

ジェネリックと ```abstract``` を用いずに [想定するケース](#想定するケース) に対する回答を　@pCYSl5EDgo 様から頂きましたので、コードを転記いたします。

```typescript:ジェネリックとabstractを用いない回答
export class HogeService {
  public exec(instance: {exec:()=>any}) {
    instance.exec();
  }
}
```

確かにシンプルでいいですね。これで上記までの実装と同じ結果が得られます。
ご指摘にもあるとおり、ジェネリックと ```abstract``` を用いるのはやり過ぎなケースにおいて有用な対応だと思います。
> @pCYSl5EDgo 様

ご指摘、ありがとうございました。
