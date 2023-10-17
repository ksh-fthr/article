# はじめに

ジェネリックとはなんなの？ という疑問を解決するための学習の記録です。

# 確認環境

| 環境                                    | 備考                |
| ------------------------------------- | ----------------- |
| [StackBlitz](https://stackblitz.com/) | 実装お試しができるクラウドサービス |

# そもそもジェネリックって？

[Java とかだと ジェネリクス](https://docs.oracle.com/javase/jp/7/technotes/guides/language/generics.html) と言われてます。言語によってこの辺の呼称がまちまち。
なんだったら TypeScript でもジェネリクスとかジェネリックスとか呼ばれることもあります。

ジェネリックとは **総称型** と言われるものです。任意のクラス、メソッドに対して任意の型を紐付けることを可能にする仕組みです。
[ここ](https://typescript-jp.gitbook.io/deep-dive/type-system/generics)  とか [ここ](https://typescriptbook.jp/reference/generics) で詳しく解説されています。

# どんなときに役立つ？

## 配列で扱う型を指定したい

たとえば配列の変数を定義する際に、**この型の配列ですよ** というのがわかると嬉しい。
あと指定した型以外を設定しようとしたり、指定した型に存在しないメンバを設定しようとすると怒ってくれるともっと嬉しい。
そんな嬉しいことを担当してくれるのがジェネリックです。

サンプルコードで見ていきます。

```typescript
// Array でジェネリックの型指定を行う際に指定する interface
interface IHoge {
  hoge: string;
  piyo: number;
  bar: boolean;
}

// IHoge 型の配列を宣言
// 下記は const aHoge: IHoge[] と同義
const aHoge: Array<IHoge> = [];

// これは設定する値の型が異なるのでエラーになる
// Argument of type 'number' is not assignable to parameter of type 'IHoge'.(2345)
aHoge.push(1);

// これも設定する値の型が異なるのでエラーになる
// Argument of type 'string' is not assignable to parameter of type 'IHoge'.(2345)
aHoge.push("ほげ");

// これは変数宣言時に指定した IHoge の型に沿ったデータなので OK
aHoge.push({
  hoge: "ほげ",
  piyo: 100,
  bar: false,
})
```

と、こんな感じで Array で扱う型を指定することができて、更に異なる型を扱おうとするエラーを返してくれます。
ちなみに `IHoge 型の配列を宣言` する場合、コメントにあるとおり下記のような書き方も出来ます。

```typescript
//  IHoge 型の配列の宣言はこう書いても良い
const aHoge: IHoge[] = [];
```

## 関数やメソッドの引数に汎用性を持たせたい

[TypeScript ではメソッドオーバーロードもサポートしてます](https://typescriptbook.jp/reference/functions/overload-functions) が、Java や C# 等のオーバーロードとは異なります。
[リンク先の実装](https://typescriptbook.jp/reference/functions/overload-functions#%E3%82%AA%E3%83%BC%E3%83%90%E3%83%BC%E3%83%AD%E3%83%BC%E3%83%89%E9%96%A2%E6%95%B0%E3%81%AE%E6%96%87%E6%B3%95) を参考に以下のコードを紹介します。

```typescript
/**
 * メソドッド引数の型が異なるオーバーロードの実装例
 * オーバーロードを共用型で実装している
 */

// オーバーロードするメソッドシグネチャを宣言しておく
interface IHoge {
  // メソッドシグネチャ部分
  hoge(arg: string);
  hoge(arg: string[]);
}

class CHoge implements IHoge {
  // 引数を 共用型 で指定することで汎用さを実現している
  hoge(arg: string | string[]) {
    console.log(arg);
  }
}

const cHoge = new CHoge();
cHoge.hoge("ほげ");
cHoge.hoge(["ほげ", "ぴよ"]);

// Logs:
// ほげ
// ["ほげ", "ぴよ"]
```

このサンプルコードですと、メソッドの実体は引数が **共用型** で定義されています。扱いたい型が増えればその分共用型で指定する型を追加しなければならず、少々煩雑です。
こうした煩雑さをジェネリックを使って解消できます。以下はジェネリックを使った実装です。

```typescript
/**
 * メソドッド引数の型が異なるオーバーロードの実装例
 * オーバーロードをジェネリックを使用して実現することで任意の型を扱うことができる
 */

// オーバーロードするメソッドシグネチャを宣言しておく
interface IHoge {
  // メソッドシグネチャ部分
  hoge<T>(arg2: T);
}

class CHoge implements IHoge {
  hoge<T>(arg: T) {
    console.log(arg)
  }
}

const cHoge = new CHoge();
cHoge.hoge<string>("ほげ");
cHoge.hoge<number>(100);
cHoge.hoge<{}>({hoge: "ほげほげ"});
cHoge.hoge<string[]>(["ほげ", "ぴよ"])

// Logs:
// ほげ
// 100
// {hoge: "ほげほげ"}
// ["ほげ", "ぴよ"]
```

先のサンプルコード同様、こちらの実装方法も Java や C# のようなオーバーロードとはかなり毛色が異なりますが、引数を共有型で指定するよりは I/F がスッキリしました。
実務で使う場合は具象クラスの役割や実際に渡ってくる引数の型に注意を払う必要がありますが、ジェネリックを使う場面が想像できたと思います。

**補足 )**
上のサンプルコードは メソッド引数の数が同じで型が異なるケース には対応できますが、**メソッド引数の数が異なるオーバーロード** には対応できていません。
**メソッド引数の数が異なるオーバーロード** に対応する場合、[こちらのジェネリックを使わない実装例](https://typescript-jp.gitbook.io/deep-dive/type-system/functions#brdo) では **第二引数以降をオプショナルで指定** することで実現していますが、ジェネリックを用いた場合でも同様な実装になります。

以下にサンプルコードを示します。

```typescript
/**
 * メソドッド引数の数が異なるオーバーロードの実装例
 * こちらはジェネリックを使った実装でも同じように第二引数以降はオプショナルにする必要がある
 */

// オーバーロードするメソッドシグネチャを宣言しておく
interface IHoge {
  // メソッドシグネチャ部分
  hoge(arg: string, arg2?: string, arg3?: string);
}

class CHoge implements IHoge {

  // 第二引数以降を オプショナル で指定することで汎用さを実現している
  hoge(arg: string, arg2?: string, arg3?: string) {
    console.log(arg);
    if (arg2) {
      console.log(`${arg}, ${arg2}`);
    }
    if (arg3) {
      console.log(`${arg2}, ${arg2}, ${arg3}`);
    }
  }
}

const cHoge = new CHoge();
cHoge.hoge("ほげ");
cHoge.hoge("ほげ", "ぴよ");
cHoge.hoge("ほげ", "ぴよ", "ふが");
```

# クラスの型にも指定できます

ジェネリックはクラスの型としても利用できます。

## 基本

ジェネリックをクラスの型として利用する場合は次のように クラス名のあとに `<...>` と指定します。

```typescript
/**
 * 任意の型をとるクラス
 * ジェネリックで指定された型はそのクラスで扱う型となる
 */
class GHoge<T> {
  value: T;
  getValue(): T {
    return this.value;
  }
}

// string を指定してインスタンスを生成する
// - これで GHoge　は string を扱うインスタンスとして生成された.
// - GHoge のフィールドである value には string しか設定できない
const gHoge = new GHoge<string>();
gHoge.value = 'Hoge';

console.log(gHoge.getValue()); // Hoge と出る


// value に string 以外を設定するとエラーになる
//
// Type 'boolean' is not assignable to type 'string'.(2322)
// (property) GHoge<string>.value: string
gHoge.value = false;


// number でも同じ.
// 
// Type 'number' is not assignable to type 'string'.(2322)
// (property) GHoge<string>.value: string
gHoge.value = 1;
```

クラスに対してジェネリックを用いて型を指定することで、その **クラスのインスタンスが扱う型 を任意に設定することができる** ことがわかりました。
ジェネリックを使うことで **インスタンスを生成した際に扱う型の自由度が高くなった** とともに、**指定した型以外はエラーとする厳格さ** を獲得できたことになります。

## 応用

### 型引数は複数指定可能

クラスで扱う型は複数指定することもできます。
やり方は 型引数を指定する際に `,` で区切るだけです。

```typescript
/**
 * 型引数を複数指定できるクラス
 */
class GHoge<T, R> {
  tValue: T;
  rValue: R;

  getTValue(): T {
    return this.tValue;
  }

  getRValue(): R {
    return this.rValue;
  }
}

// 型引数を 2つ 受け取るクラスなので、インスタンス生成時も型を複数指定する
// ここでは string と number を扱うインスタンスを生成する
const gHoge = new GHoge<string, number>();
gHoge.tValue = 'Hoge';
gHoge.rValue = 100;

console.log({
  tValue: gHoge.getTValue(),
  rValue: gHoge.getRValue()
});

// Logs:
// {tValue: "Hoge", rValue: 100}
```

### 型引数は既定値も設定できる

型引数は既定値としての型を指定することもできます。既定値を指定してクラスを定義した場合、

- 型指定せずにインスタンスを生成する
- 型指定してインスタンスを生成する

の 2パターン でインスタンスを生成できます。
前者の場合は **既定値の型** を扱うインスタンスが生成され、後者の場合は( こちらはいままで見てきたとおり )  **指定した型** を扱うインスタンスが生成されます。
もちろん、**既定値の型を指定して** インスタンスを生成することもできます。

```typescript
/**
 * 型引数の既定値を指定して任意の型をとるクラス
 */
class GHoge<T = string> {
  tValue: T;

  getTValue(): T {
    return this.tValue;
  }
}

///////////////////////////////////////////////////
// 型引数を指定せずにインスタンスを生成してみる
///////////////////////////////////////////////////

// 既定値として string を指定しているので、型引数なしでインスタンスを生成できる
const gHogeDefault = new GHoge();
gHogeDefault.tValue = 'Hoge';

console.log({
  tValue: gHogeDefault.getTValue(),
});

// Logs:
// {tValue: "Hoge"}

// この状態で tValue に number を指定するとエラーになる
// Type 'number' is not assignable to type 'string'.(2322)
// const gHoge: GHoge<string>
gHogeDefault.tValue = 100;


///////////////////////////////////////////////////
// 型引数を指定してインスタンスを生成してみる
///////////////////////////////////////////////////

// 今度は型引数を指定してインスタンスを生成する
const gHogeNumber = new GHoge<number>();
gHogeNumber.tValue = 100;

console.log({
  tValue: gHogeDefault.getTValue(),
});

// Logs:
// {tValue: 100}

// 型引数に number を指定してインスタンスを生成しているので string を設定しようとするとエラーになる
// Type 'string' is not assignable to type 'number'.(2322)
// (property) GHoge<number>.tValue: number
gHogeNumber.tValue = "Hoge";


///////////////////////////////////////////////////
// 型引数の既定値と同じ型を指定してインスタンスを生成してみる
///////////////////////////////////////////////////

// 今度は型引数の既定値と同じ型を指定してインスタンスを生成する
const gHogeString = new GHoge<string>();
gHogeString.tValue = "Hoge";

console.log({
  tValue: gHogeString.getTValue(),
});

// Logs:
// {tValue: "Hoge"}

// 型引数に string を指定してインスタンスを生成しているので number を設定しようとするとエラーになる
// Type 'number' is not assignable to type 'string'.(2322)
// (property) GHoge<string>.tValue: string
gHogeString.tValue = 100;
```

### 型引数は制約を設けることもできる

ここで言う「制約」とは **特定のクラスを継承したクラスのみ受けつける** という意味になります。
インスタンス生成時、ジェネリックの型引数には任意の型を指定できますが、扱う型を制限したいケースもあります。そんなときにここで示す方法が役に立ちます。

```typescript
class BaseHoge {
  value: string;
}

class ChildHoge extends BaseHoge {
  value2: number;
}

/**
 * ジェネリックで扱う型を 特定クラス か その子クラス に限定する
 */
class GHoge<T extends BaseHoge> {
  tValue: T;

  getTValue(): T {
    return this.tValue;
  }
}


////////////////////////////////////////////////////
// 型引数に子クラスを指定して　インスタンスを生成する
////////////////////////////////////////////////////
const gHogeChild = new GHoge<ChildHoge>();

// 子クラスのインスタンスをセット
gHogeChild.tValue = new ChildHoge();
console.log({
  value: gHogeChild.getTValue(),
  instance: gHogeChild.getTValue() instanceof ChildHoge,
});

// Logs:
// { instance: true, value: ChildHoge }

//--------------------------------------------------
// ベースクラスのインスタンスをセットするとエラー
// ( 今回のサンプルコードでは ベースクラスと子クラスでフィールドに差分があるため )
// ( ベースクラスと子クラスのフィールドに差分がなければエラーにならない )
//
// Property 'value2' is missing in type 'BaseHoge' but required in type 'ChildHoge'.(2741)
gHogeChild.tValue = new BaseHoge();


////////////////////////////////////////////////////
// 型引数 に Base クラスを指定してもインスタンスは生成できる
////////////////////////////////////////////////////
// ベースクラスのインスタンスをセット
const gHogeBase = new GHoge<BaseHoge>();
gHogeBase.tValue = new BaseHoge();
console.log({
  value: gHogeBase.getTValue(),
  instance: gHogeBase.getTValue() instanceof BaseHoge,
});

// Logs:
// { instance: true, value: BaseHoge }

//--------------------------------------------------
// 子クラスのインスタンスをセットしても問題なし
gHogeBase.tValue = new ChildHoge();
console.log({
  value: gHogeBase.getTValue(),
  instance: gHogeBase.getTValue() instanceof ChildHoge,
});

// Logs:
// { instance: true, value: ChildHoge }

// ただベースクラスに存在しないフィールドにはアクセスできない
// Property 'value2' does not exist on type 'BaseHoge'. Did you mean 'value'?(2551)
gHogeBase.getTValue().value2 = '#1';


////////////////////////////////////////////////////
// BaseHoge を継承していない型は指定できない
////////////////////////////////////////////////////

// Type 'string' does not satisfy the constraint 'BaseHoge'.(2344)
const gHogeNotExtends = new GHoge<string>();
```

## ジェネリックメソッド

以下の特徴を持ちます。

- 引数、戻り値、ロケール変数の型を **メソッドを呼び出す際に決められる** 
- メソッド名の直後に `<...>` の形式で型引数を宣言する

### ジェネリックメソッドの型

```typescript
// 型引数が一つの場合
function gFuncHoge<T>(arg1: T, arg2: T): T {}

// 型引数が二つの場合
function gFuncHoge<T, U>(arg1: T, arg2: U): U {}
```

### サンプルコード

```typescript
/**
 * 任意の型を引数にとり、その引数をオブジェクトで返却する関数
 */
function gFuncHoge<T>(arg1: T, arg2: T): { arg1: T, arg2: T } {
  return {
    arg1,
    arg2
  }
}
console.log(gFuncHoge("ほげ", "ぴよ"))
// Logs:
// {arg1: "ほげ", arg2: "ぴよ"}


/**
 * 任意の型を複数 引数にとり、その引数をタプルで返却する関数
 */
function gFuncHogeMulti<T, U>(arg1: T, arg2: U): [T, U] {
  return [
    arg1,
    arg2
  ]
}
console.log(gFuncHogeMulti("ほげ", 100))
// Logs:
// ["ほげ", 100]


class Hoge {
  /**
   * 任意の　型の配列を引数にとり、その配列を結合して返却するメソッド
   */
  static gFuncConcat<T>(data: T[], data2: T[]): T[] {
    return data.concat(data2)
  }
}
console.log(Hoge.gFuncConcat(["ほげ"], ["ぴよ"]))
// Logs:
// ["ほげ", "ぴよ"]
```

# 参考

- [TypeScript Deep Dive 日本語版 > ジェネリック型](https://typescript-jp.gitbook.io/deep-dive/type-system/generics)  
- [サバイバルTypeScript > ジェネリクス (generics)](https://typescriptbook.jp/reference/generics) 
