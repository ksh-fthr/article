この記事は TypeScript Advent Calendar 2023 9日目の記事になります｡


https://qiita.com/advent-calendar/2023/typescript

# はじめに

普段使っていて悩むことがままある TypeScript のアレコレ。
`null | undefined` のチェックで使う各演算子や型づけ、`type` とか `keyof` とか諸々です｡

本記事ではその辺を改めて見直すことで理解を深めます。


# 確認環境

| 環境                                    | 備考                |
| ------------------------------------- | ----------------- |
| [StackBlitz](https://stackblitz.com/) | 実装お試しができるクラウドサービス |

# 型

## タプル

任意の型を `[]` 内に記述したものを **タプル型** と言います。`[]` 内には複数の型を混在させることもできるし、同じ型を複数指定することもできます。
**型を明示的に指定**した場合と**型推論に任せた**場合で、その変数の**型解釈が異なる**ので注意が必要です。

### 型を明示的に指定

```typescript
// 型を明示的に指定
const hoge: [string, number, boolean] = ["ほげほげ", 0, false];
```

この場合  `hoge` の型は `[string, number, boolean]` です。
したがって次のコードは問題なく解釈されます。

```typescript
// 型を明示的に指定
const hoge: [string, number, boolean] = ["ほげほげ", 10, false];
console.log(hoge[0].substring(1)); // げほげ が出力される
console.log(hoge[1].toFixed(1));   // 10.0 が出力される
console.log(hoge[2] === false);    // true が出力される
```

ですが、次のコードはエラーになります。
これはタプルで指定した型はその順序と数を満たす必要があるためです。

```typescript
const hoge: [string, number, boolean] = ["ほげほげ", 10, false];

// NG: 型エラー(型の順序があっていない)
// Type 'boolean' is not assignable to type 'number'.(2322)
const hoge: [string, number, boolean] = ["ほげほげ", false, 10];

// NG: 型エラー(数が不足している)
// Type '[string]' is not assignable to type '[string, number, boolean]'.
//  Source has 1 element(s) but target requires 3.(2322)
target requires 3.(2322)
const hoge: [string, number, boolean] = ["ほげほげ"];
```

### 型推論に任せる

```typescript
// 型推論に任せる
const hoge = ["ほげほげ", 0, false];
```

この場合 `hoge` の型は `(string | number | boolean)[]` になります。
次のコードはエラーがでます。

```typescript
// 型推論に任せる
const hoge = ["ほげほげ", 10, false];
console.log(hoge[0].substring(1)); // げほげ が出力される(エラー　*-1)
console.log(hoge[1].toFixed(1));   // 10.0 が出力される(エラー　*-2)
console.log(hoge[2] === false);    // true が出力される
```

これは型推論によって **共用型( [後述](#共用型) )** の配列と判断されているためで、要素の `[0]` や `[1]` が `string` や `number` 型のデータである保証がないためです。

下記はそれぞれのエラーの内容です。

```bash
# *-1 の内容
Property 'substring' does not exist on type 'string | number | boolean'.
  Property 'substring' does not exist on type 'number'.(2339)
any
No quick fixes available
```

```bash
# *-2 の内容
Property 'toFixed' does not exist on type 'string | number | boolean'.
  Property 'toFixed' does not exist on type 'string'.(2339)
any
No quick fixes available
```

### タプル利用時の注意点

データの実体から要素を削除しても、該当する要素に対する型づけは残ります。

```typescript
// 型を明示的に指定
const hoge: [string, number, boolean] = ["ほげほげ", 10, false];
console.log(hoge[0].substring(1)); // げほげ が出力される
console.log(hoge[1].toFixed(1));   // 10.0 が出力される
console.log(hoge[2] === false);    // true が出力される

hoge.shift();                      // 先頭の要素(文字列)を削除
console.log(hoge)                  // データは [10, false] となっている
console.log(hoge[0].substring(1)); // 型づけをしているのでコード中警告やエラーはでないが、実行時に Error: hoge[0].substring is not a function となる
```

タプルの利用は控えて `interface` を使った方が無難です。

## 共用型

複数の型を列挙して **そのうちのどれか** であることを示すもので、 `|` で区切ることで表現します。

```typescript
let hoge: string | number; // string or number の型をもつ

// string 型, number 型の値を設定するぶんには問題ない
hoge = "ほげ";  // OK
hoge = 1;      // OK
```

列挙した中に存在しない型を指定するとエラーになります。

```typescript
let hoge: string | number; // string or number の型をもつ

// string 型, number 型以外の値を設定するとエラーになる
// Type 'boolean' is not assignable to type 'string | number'.(2322)
hoge = false;
```

配列でも扱えます。

```typescript
let hoge: (string | number)[];
hoge[0] = "ほげ"; // OK
hoge[1] = 1;     // OK
```

こちらも列挙した中に存在しない型を指定するとエラーになります。

```typescript
let hoge: (string | number)[];
hoge[2] = true;  // NG, Type 'boolean' is not assignable to type 'string | number'.(2322) 
```

:::note info

先に挙げた [タプルの **型推論** の例](#型推論に任せる) は、この **共用型の配列** という解釈をされたことになります。

:::

## 交差型

型を統合した型を指します。何気なく使ってたり見たことがあると思います。
ちなみに後述しております `type ~` での型づけを **型エイリアス** と言います。

```typescript
// 結合対象の型-1
interface IHoge {
  'hoge': string;
}

// 結合対象の型-2
interface IPiyo {
  'piyo': number; 
}

/////////////////////////////////////
// 交差型は & で型を結合する
/////////////////////////////////////
type TBoo = IHoge & IPiyo;

// IHoge のフィールド: hoge(string 型) と IPiyo のフィールド: piyo(number 型) を指定して、
// オブジェクト tBoo が出来る
const tBoo: TBoo = {
  hoge: "ほげほげ",
  piyo: 1
};
console.log(tBoo);  // {hoge: "ほげほげ", piyo: 1}
```

IHoge にも IPiyo にも存在しないフィールドを指定するとエラーになります。
下のコードは 型エラーが発生します。

```typescript
// Type '{ hoge: string; piyo: number; bar: boolean; }' is not assignable to type 'intersectionType'.
//  Object literal may only specify known properties, and 'bar' does not exist in type 'intersectionType'.(2322)
const tBoo: TBoo = {
  hoge: "ほげほげ",
  piyo: 1,
  fuga: false // 存在しないフィールドを指定した
};
```

### 型は3つ以上指定できる

交差型で指定する型は 3つ以上 指定できます。

```typescript
// 結合対象の型-1
interface IHoge {
  'hoge': string;
}

// 結合対象の型-2
interface IPiyo {
  'piyo': number; 
}

// 結合対象の型-3
interface IFuga {
  'fuga': boolean; 
}

type TBoo = IHoge & IPiyo & IFuga;

// IHoge のフィールド: hoge(string 型) と IPiyo のフィールド: piyo(number 型)、それから IFuga のフィールド: fuga(boolean 型) を指定して、オブジェクト tBoo が出来る
const tBoo: TBoo = {
  hoge: "ほげほげ",
  piyo: 1,
  fuga: false,
};
console.log(tBoo);  // {hoge: "ほげほげ", piyo: 1, fuga: false}
```

### 同じことを interface の拡張でも実現できる

交差型と同じことを interface の拡張でも出来ます。

```typescript
/////////////////////////////////////
// 似たようなことは interface の拡張でも出来る
/////////////////////////////////////
interface IBoo extends iHoge, iPiyo {}

// string 型のフィールド: hoge と number 型のフィールド: piyo を指定してオブジェクト iBoo が出来る
const iBoo: IBoo = {
  hoge: "ほげー",
  piyo: 10
};

console.log(iBoo); // {hoge: "ほげー", piyo: 10}
```

interface を拡張したこちらのコードでも、元の interface に存在しないフィールドを指定するとエラーになります。

```typescript
// Type '{ hoge: string; piyo: number; fuga: boolean; }' is not assignable to type 'IBoo'.
//  Object literal may only specify known properties, and 'fuga' does not exist in type 'IBoo'.(2322)
interface IBoo extends iHoge, iPiyo {}
const iBoo: IBoo = {
  hoge: "ほげー",
  piyo: 10,
  fuga: false // 存在しないフィールドを指定した
};
```

## unknown 型

`any` と似て非なるもの。 **型ガード** と一緒に使うことで威力を発揮します。

```typescript
// any は「何でも許す」ので、データの型が string と分からなくても string のメソッド substring が使用できてしまう
const hoge: any = "ほげほげ";
console.log(hoge.substring(1)); // OK, "げほげ" と出る

// unknown　は「なんかよく分からないけれど型づけしておきたいなら良いよ」なので string かどうか分からない
// よって string のメソッド substring を使おうとするとエラーになる
const piyo: unknown = "ほげほげ";
console.log(piyo.substring(1)); // NG, Property 'substring' does not exist on type 'unknown'.(2339)

// なので、 unknown を使う場合は 型ガード と組み合わせる
// これなら string型 のデータだということが保証されるので substring が使える
if (typeof piyo === 'string') {
  console.log(piyo.substring(1)); // OK, "げほげ" と出る
}
```

# null | undefined アレコレ

`null` や `undefined` に対するケアは重要です。知っておくと安心な設定と、その設定をした上での挙動について見ていきます。

## null | undefined を許さないように設定する

 TypeScript で `null` や `undefined` を許容しない設定にするには `tsconfig.json` で下記を設定します。
( 参照: https://typescriptbook.jp/reference/tsconfig/strictnullchecks )

```json
{
  "compilerOptions": {
    "strictNullChecks": true
  }
}
```

こうすると `null` や `undefined` を設定するとエラーになります。

```typescript
let hoge: string;

hoge = "ほげほげ";
hoge = null;      // Type 'null' is not assignable to type 'string'.(2322)
hoge = undefined; // Type 'undefined' is not assignable to type 'string'.(2322)
```

## null | undefined を許さないように設定した上で null | undefined を許可したい

↑ の設定をした上で、なお `null` や `undefined` を指定したい場合は [共用型](#共用型) を使ってあげます。
次のコードは警告なく `null` や `undefined` が設定されます。

```typescript
let hoge: string | null | undefined;
hoge = "ほげほげ";
hoge = null;
hoge = undefined;
```

## `?.` 演算子を使って変数参照時に null | undefined をチェックする

`?.` 演算子 ( **[オプショナルチェーン (optional chaining) 演算子](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Optional_chaining)** といいます ) を使って実現します。
これを使うとコードがスッキリします。

```typescript
//
// `?.` 演算子 を使ったやり方
//
let hoge: string | null | undefined;

// `?.` 演算子を使う
hoge = "ほげほげ";
console.log(hoge?.substring(1));  // OK, げほげ

hoge = null;
console.log(hoge?.substring(1));  // OK, undefined が出る ( null ではない点に注目 )

hoge = undefined;
console.log(hoge?.substring(1));  // OK, undefined が出る
```

非常に見通しが良いです。

:::note info

`null` が設定されたときに `undefined` が返却されていることに注目です。<br>
**`?.` 演算子を使ってアクセスすると、対象の変数が `null | undefined` の場合は `undefined` が返ります。**

:::

ちなみに `?.` 演算子を使わない場合は次のようになって見通しが悪いです。

```typescript
//
// `?.` 演算子 を使わないやり方
//
let hoge: string | null | undefined;

// 単純に if 文を使うか
hoge = "ほげほげ";
if (hoge) { console.log(hoge.substring(1)); }  // OK, げほげ
// もしくは 三項演算子 を使う
hoge ? console.log(hoge.substring(1)) : null;  // OK, げほげ
console.log(hoge ? hoge.substring(1) : null);  // OK, げほげ

hoge = null;
if (hoge) { console.log(hoge.substring(1)); } // OK, でもなにも出ない
hoge ? console.log(hoge.substring(1)) : null; // OK, でもなにも出ない
console.log(hoge ? hoge.substring(1) : null); // OK, null

hoge = undefined;
if (hoge) { console.log(hoge.substring(1)); }      // OK, でもなにも出ない
hoge ? console.log(hoge.substring(1)) : undefined; // OK, でもなにも出ない
console.log(hoge ? hoge.substring(1) : undefined); // OK, undefined
```

更に言うと何もチェックしないと実行時エラーになります。

```typescript
//
// チェックしない
//
let hoge: string | null | undefined;

hoge = "ほげほげ";
console.log(hoge.substring(1)); // OK, げほげ

hoge = null;
// コード上で 'hoge' is possibly 'null'.(18048) と警告がでていて
// 実行時に　Error: Cannot read properties of null (reading 'substring') となる
console.log(hoge.substring(1));

hoge = undefined;
// コード上で 'hoge' is possibly 'null'.(18048) と警告がでていて
// 実行時に　Error: Cannot read properties of null (reading 'substring') となる
console.log(hoge.substring(1));
```

## null | undefined のときに規定値を設定する

`null` | `undefined` が設定されているときに規定値を返却したいケース、そんなときにスッキリ書けるコードが下記です。

`??` 演算子 ( **[Null 合体演算子 (nullish coalescing operator)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing)** といいます ) を使って実現します。

```typescript
//
// `??` 演算子 を使ったやり方
//
let hoge: string | null | undefined;

hoge = "ほげほげ";
console.log(hoge ?? '初期値'); // OK, ほげほげ

hoge = null;
console.log(hoge ?? '初期値'); // OK, 初期値

hoge = undefined;
console.log(hoge ?? '初期値'); // OK, 初期値
```

ちょっと 三項演算子 に似ていますね。上記を 三項演算子 で書き換えると次になります。

```typescript
//
// `??` 演算子 を使わないやり方( 三項演算子を使う )
//
let hoge: string | null | undefined;

hoge = "ほげほげ";
console.log(hoge ? hoge : '初期値'); // OK, ほげほげ

hoge = null;
console.log(hoge ? hoge : '初期値'); // OK, 初期値

hoge = undefined;
console.log(hoge ? hoge : '初期値'); // OK, 初期値
```

## `!` 演算子を使って変数参照時に null | undefined をチェックしない

`!` 演算子 ( **[非nullアサーション演算子(non-null assertion operator)](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#non-null-assertion-operator-postfix-)** といいます ) を使います。

リンク先にはこうあります。

> ```text
> Non-null Assertion Operator (Postfix !)
>
> TypeScript also has a special syntax for removing null and undefined from a type without doing any explicit checking. Writing ! after any expression is effectively a type assertion that the value isn’t null or undefined:
> 
> ( コードは省略)
> 
> Just like other type assertions, this doesn’t change the runtime behavior of your code, so it’s important to only use ! when you know that the value can’t be null or undefined.
> 
> ```

これをコードで見てみると

```typescript
let hoge: string | null | undefined;

hoge = "ほげほげ";
console.log(hoge!.substring(1)); // OK, ほげほげ

hoge = null;
// コード上で警告はでないが
// 実行時に NG, Error: Cannot read properties of undefined (reading 'substring') となる
console.log(hoge!.substring(1));

hoge = undefined;
// コード上で警告はでないが
// 実行時に NG, Error: Cannot read properties of undefined (reading 'substring') となる
console.log(hoge!.substring(1));
```

こんな感じになります。
つまり

:::note info

実際には **その可能性があるのにもかかわらず**、 `null` だろうが `undefined` だろうが **気にせずに (警告を黙らせて) アクセス** します。<br>
**`null  | undefine`  ではないことを実装者がわかっている、かつ間違いがないことが確実な状態で使うものだ**、と理解するのが良いです。

:::

# 型エイリアス

いわゆる `type` を使った型宣言です。主に [タプル](#タプル) や [共用型](#共用型)、あとは [交差型](#交差型) に対して **短い名前** を付与したい場合に使うようです。

## タプルに対して型エイリアスをつける

```typescript
// タプルに対して型エイリアスを付与
type toupleType = [string, number, boolean];

// toupleType 型の変数に対して値を設定してみる
let hogeTouple: toupleType;

// タプルで指定している型をすべて満たしているので OK
hogeTouple = ["ほげほげ", 0, false];

// タプルで指定している型をすべて満たしていないので NG
// NG: 型エラー
// Type '[string]' is not assignable to type 'toupleType'.
//   Source has 1 element(s) but target requires 3.(2322)
hogeTouple = ["ほげほげ"];
```

## 共用型に対して型エイリアスをつける

```typescript
type unionType = string | number;
let hogeUnion: unionType;

// string 型, number 型の値を設定するぶんには問題ない
hogeUnion = "ほげほげ"; // OK
hogeUnion = 1;        // OK

// string 型, number 型以外の値を設定するとエラーになる
// NG: 型エラー, Type 'boolean' is not assignable to type 'unionType'.(2322)
hogeUnion = false;
```

## 交差型に対して型エイリアスをつける

交差型についてはすでに触れていますのでここでの説明は割愛します。
[交差型](#交差型) をご参照ください。

# 型アサーション

これまで見てきたコードで、**フィールドの並びや過不足によるエラー** が出ているものがありました。
これらは **[型アサーション](https://typescript-jp.gitbook.io/deep-dive/type-system/type-assertion)** によって回避できます。

## タプル型のエラーを回避

```typescript
type toupleType = [string, number, boolean];
let hogeTouple: toupleType;

// 以下はそれぞれ toupleType の型定義から外れているので本来ならエラーになる
// だが、 unknown -> unionType とアサーションをチェインするとエラーを回避できてしまう

// タプルで指定している型をすべて満たしていない
hogeTouple = ["ほげほげ"] as unknown as toupleType;

// タプルで指定しているフィールド数より多い
hogeTouple = ["ほげほげ", 0, false, false] as unknown as toupleType;
hogeTouple = ["ほげほげ", 0, false, Object] as unknown as toupleType;

// タプルで指定している型以外を指定
hogeTouple = ["ほげほげ", 0, Object] as unknown as toupleType;
```

## 共用型のエラーを回避

```typescript
type unionType = string | number;
let hogeUnion: unionType;

// string 型, number 型以外の値を設定しているので本来ならエラーになる、
// だが、 unknown -> unionType とアサーションをチェインするとエラーを回避できてしまう
hogeUnion = false as unknown as unionType;
```

### 交差型のエラーを回避

```typescript
interface IHoge {
  'hoge': string;
}

interface IPiyo {
  'piyo': number; 
}

type TBoo = IHoge & IPiyo;

/////////////////////////////////////////
// ケース1. 元の型に存在しないフィールドを設定した
/////////////////////////////////////////
// TBoo型 に存在しないフィールドを指定しているので本来ならエラーになる
// だが、 TBoo とアサーションするとエラーを回避できてしまう
const tBoo: TBoo = {
  hoge: "ほげほげ",
  piyo: 1,
  fuga: false // 存在しないフィールドを指定した
} as TBoo;

/////////////////////////////////////////
// ケース2. 元の型にあるフィールドを設定していない
/////////////////////////////////////////
// TBoo型 に存在するフィールドを指定していないので本来ならエラーになる
// だが、 TBoo とアサーションするとエラーを回避できてしまう
const tBoo: TBoo = {
  hoge: "ほげほげ",
} as TBoo;
```

:::note info

[こちら](https://typescript-jp.gitbook.io/deep-dive/type-system/type-assertion#asshonha) にも記載されているように､型アサーションも無理やりコンパイラを黙らせる手段です。
できれば使わずにプログラミングすることを心がけたいですね。

ところで型アサーションは **キャスト** とは明確に区別されるようです。
詳しくは下記をご参照ください。

- [型アサーションとキャスト](https://typescript-jp.gitbook.io/deep-dive/type-system/type-assertion#asshontokyasuto)
- [型アサーションとキャストの違い](https://typescriptbook.jp/reference/values-types-variables/type-assertion-as#%E5%9E%8B%E3%82%A2%E3%82%B5%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%A8%E3%82%AD%E3%83%A3%E3%82%B9%E3%83%88%E3%81%AE%E9%81%95%E3%81%84)

:::

# インデックスシグネチャ

ブラケット構文(`[]`) での型を示します。

どんなケースで使うかというと、例えば...
interface を定義する際、**任意の型を許容させたい** ケースが発生しました。

そんなときはインデックスシグネチャを使って対応できます。

```typescript
// インデックスシグネチャでフィールドを定義すると...
interface iHoge {
  [key: string]: any;
}

// こんな感じで好き勝手できる
const ihoge: iHoge = {
  hoge: "ほげほげ",
  piyo: 1,
  fuga: false,
  bar: {},
  foo: []
};
```

「好き勝手できる」とは書きましたが、これはフィールドの型を `any` としているからです。
特定の型をフィールドの型に指定すれば、その型のみ許容される interface になります。

# keyof / Lookup Types

`keyof` や `Lookup Types` によって **既存の型から型情報を抽出** できます。
個人的にはちょっとわかり辛い機能です。何度も使うことによって慣れるしかないかな、と感じています。

## keyof

`keyof` によって interface のフィールドセットを抽出できます。
次のコードは interface を定義し、その interface のフィールドセットを取得するものです。

```typescript
interface IHoge {
  a: string;
  b: string;
  c: string;
}

// IHoge のフィールドセットが型エイリアスとして THoge にセットされた
type THoge = keyof IHoge;         // type THoge = keyof IHoge

// THoge を使って型指定してみる
const hogeA: THoge = "a"; // OK
const hogeB: THoge = "b"; // OK
const hogeC: THoge = "c"; // OK
const hogeD: THoge = "d"; // NG: Type '"d"' is not assignable to type 'keyof IHoge'.(2322)
```

`const hogeD: THoge = "d";` ではエラーになりました。これは `keyof IHoge` で取得したフィールドセットに `d` が存在しないためです。
うまく言葉にできるか自信がないですが、このことから **`keyof` で取得したフィールドセットを 型エイリアス とした場合、その型エイリアスを型に指定した変数に設定できる値は取得したフィールドセットに存在する値のみ**、と解釈できます。

なお `THoge` にカーソルを当てるとコメントのように `type THoge = keyof IHoge` と出ますが、`THoge` に設定される中身は `type TKeys = "a" | "b" | "c"` と同義です。次のコードで確認します。

```typescript
type THoge2 = "a" | "b" | "c";
const hogeA2: THoge2 = "a"; // OK
const hogeB2: THoge2 = "b"; // OK
const hogeC2: THoge2 = "c"; // OK
const hogeD2: THoge2 = "d"; // NG: Type '"d"' is not assignable to type 'THoge2'.(2322)
```

前掲のコードと同じように `THoge2` に含まれない `d` ではエラーがでました。
これで改めて `type THoge = keyof IHoge;` で設定された中身は `"a" | "b" | "c"` であることが確認できたと思います。

ちなみに、interface のないオブジェクトに対しても、`typeof` による型判別を噛ませることで同じ事ができます。

```typescript
// interface を指定しないオブジェクトを対象にすると、そのオブジェクトのフィールド名を型エイリアスとして定義できる
const hoge = {
  a: "A",
  b: "B",
  c: "C",
}

type THoge3 = keyof (typeof hoge); // type THoge3 = "a" | "b" | "c" 
const hogeA3: THoge3 = "a"; // OK
const hogeB3: THoge3 = "b"; // OK
const hogeC3: THoge3 = "c"; // OK
const hogeD3: THoge3 = "d"; // NG: Type '"d"' is not assignable to type '"a" | "b" | "c"'.(2322)
```

前掲の 2つ のコードと同じ結果が得られました。

ところで、先に

> その型エイリアスを型に指定した変数に設定できる値は取得したフィールドセットに存在する値のみ

と記載しましたが、これはメソッドの引数でも同じです。
次のコードで確認します。

```typescript
interface IHoge {
  a: string;
  b: string;
  c: string;
}
type THoge = keyof IHoge;         // type kIHoge = keyof IHoge

const oHoge: IHoge = {
  a: "AAAAA",
  b: "BBBBB",
  c: "CCCCC"
};

// オブジェクトとそのフィールド名を受けて、そのオブジェクトの情報を返す
const existField = (oHoge: IHoge, key: THoge): {
  isExist: boolean,
  type: string,
  value: T[F]
} => {
  return {
    isExist: oHoge[key] ? true : false,
    type: typeof oHoge[key],
    value: oHoge[key]
  };
}
console.log(existField(oHoge, "a")); // {isExist: true, type: "string", value: "AAAAA"} 
console.log(existField(oHoge, "b")); // {isExist: true, type: "string", value: "BBBBB"}
console.log(existField(oHoge, "c")); // {isExist: true, type: "string", value: "CCCCC"}

// Argument of type '"d"' is not assignable to parameter of type 'keyof IHoge'.(2345) が出ている
console.log(existField(oHoge, "d")); // {isExist: false, type: "undefined", value: undefined}
```

`THoge` に含まれない `d` を指定すると `Argument of type '"d"' is not assignable to parameter of type 'keyof IHoge'.(2345)` のエラーになりました。

## Lookup Types

`T[K]` という形で **`T型` に存在する `Kフィールドの型`** を表現します。
コード例をあげてみます。

```typescript
interface IHoge {
  a: string;
  b: number;
  c: boolean;
}

type HogeA = IHoge["a"]; // string
type HogeB = IHoge["b"]; // number
type HogeC = IHoge["c"]; // boolean
type HogeMulti = IHoge["a" | "b" | "c"]; // string | number | boolean
```

これを言葉にしますと

- `IHoge` に存在する `a フィールドの型` は `string`
- `IHoge` に存在する `b フィールドの型` は `number`
- `IHoge` に存在する `c フィールドの型` は `boolean`
- `IHoge` に存在する型 は `string`, `number`, `boolean` のいずれか

となります。
これを利用して型エイリアスを作ってみます。

```typescript
interface IHoge {
  a: string;
  b: number;
  c: boolean;
}

// IHoge に存在するフィールドの型を型エイリアスとする
type THoge = IHoge[keyof IHoge]; // string | number | boolean の 共用型 となる
```

上記は Interface がわかっているときに有効ですが、次のようにして汎用性をもたせることもできます。

```typescript
// F は T のフィールドの型 となる
type FieldType<T, F extends keyof T> = T[F];

// IHoge に存在するフィールドの型を型エイリアスとする
interface IHoge {
  a: string;
  b: number;
  c: boolean;
}
type THoge = FieldType<IHoge, keyof IHoge>; // string | number | boolean の 共用型 となる

// IBoo に存在するフィールドの型を型エイリアスとする
interface IBoo {
  A: {},
  B: string,
  C: string
}
type TBoo = FieldType<IBoo, keyof IBoo>; // string | {} の 共用型 となる
```

このコードだけですと、その前のコードとの比較で

```typescript
// FieldType　定義しない
type THoge = IHoge[keyof IHoge]; // string | number | boolean の 共用型 となる

// FieldType 定義した
type THoge = FieldType<IHoge, keyof IHoge>; // string | number | boolean の 共用型 となる
```

コード量が増えただけでメリットが感じられません。
が、`FieldType` の定義を関数に応用してみます。すると

```typescript
// オブジェクトとそのフィールド名を受けて、そのオブジェクトの情報を返す
const existField = <T, F extends keyof T>(oHoge: T, key: F): {
  isExist: boolean,
  type: string,
  value: T[F]
} => {
  return {
    isExist: oHoge[key] ? true : false,
    type: typeof oHoge[key],
    value: oHoge[key]
  };
}

// IHoge のオブジェクト に対して existField を実行する
interface IHoge {
  a: string;
  b: number;
  c: boolean;
}
type THoge = keyof IHoge;         // type kIHoge = keyof IHoge

const oHoge: IHoge = {
  a: "AAAAA",
  b: 100,
  c: true
};
console.log(existField(oHoge, "a")); // {isExist: true, type: "string", value: "AAAAA"} 
console.log(existField(oHoge, "b")); // {isExist: true, type: "number", value: 100}
console.log(existField(oHoge, "c")); // {isExist: true, type: "boolean", value: true}


// IBoo のオブジェクト に対して existField を実行する
interface IBoo {
  A: {},
  B: string,
  C: string
}
const oBoo: IBoo = {
  A: {
    A1: 'A1'
  },
  B: "BBBBB",
  C: "CCCCC"
};
console.log(existField(oBoo, "A")); // {isExist: true, type: "object", value: {A1: "A1"}}
console.log(existField(oBoo, "B")); // {isExist: true, type: "string", value: "BBBBB"}
console.log(existField(oBoo, "C")); // {isExist: true, type: "string", value: "CCCCC"}
```

こんな感じで `existField` 関数が **任意のオブジェクト** を受け取ることができるようになりました。

# Mapped Types

既存の型を変換する仕組みです。interface に対して使用して、そのフィールドを `Readonly` にしたり `Optional` にしたりできます。

## Readonly にする

```typescript
interface IHoge {
  hoge: string;
  piyo: number;
}

// フィールドを readonly にする
type RIHoge = {
  // この [K in keyof IHoge] の部分が Mapped Types.
  readonly [K in keyof IHoge]: IHoge[K];
}

// RIHoge は
// 
// type RIHoge = {
//   readonly hoge: string;
//   readonly piyo: number;
// }
//
// となる
```

ここで `[K in keyof IHoge]: IHoge[K]` は **`IHoge` からフィールド名を順に見ていって、`K` に割り当てる** ということをやっているそうです。( `Product[K]` は [Lookup Types](#lookup-types) ですね )
その結果、コード中のコメントのように `IHoge` と同じ、かつ `readonly` 属性が付与されたフィールドをもつ `RIHoge` が出来上がる、というわけです。

**逆に無効化することもできる**
`-readonly` という表記があって、これを使うと `readonly` 属性を無効化できます。

```typescript
// RIHoge は前掲のコードを参照

// フィールドから readonly を除去する
type DRIHoge = {
  -readonly [K in keyof RIHoge]: RIHoge[K];
}

// DRIHoge は
// 
// type DRIHoge = {
//   hoge: string;
//   piyo: number;
// }
//
// となる
```

## オプショナルにする

同じことが オプショナル の付与・除去についてもできます。

```typescript
interface IHoge {
  hoge: string;
  piyo: number;
}

// フィールドを オプショナル にする
type OIHoge = {
  [K in keyof IHoge]?: IHoge[K];
}

// OIHoge は
// 
// type OIHoge = {
//   hoge?: string | undefined;
//   piyo?: number | undefined;
// }
//
// となる

// フィールドから オプショナル を除去する
type DOIHoge = {
  [K in keyof OIHoge]-?: OIHoge[K];
}

// DOIHoge は
// 
// type DOIHoge = {
//   hoge: string;
//   piyo: number;
// }
//
// となる
```

## readonly と オプショナル の両方を設定することもできる

コードはこんな感じになります。

```typescript
interface IHoge {
  hoge: string;
  piyo: number;
}

// フィールドを readonly と オプショナル にする
type ROIHoge = {
  readonly [K in keyof IHoge]?: IHoge[K];
}

// ROIHoge は
// 
// type ROIHoge = {
//   readonly hoge?: string | undefined;
//   readonly piyo?: number | undefined;
// }
// 
// となる

// フィールドから readonly と オプショナル を除去する
type DROIHoge = {
  -readonly [K in keyof ROIHoge]-?: ROIHoge[K];
}

// DROIHoge は
// 
// type DROIHoge = {
//   hoge: string;
//   piyo: number;
// }
// 
// となる
```

# Conditional Types

条件を提示することで使用する型を振り分ける、なんてこともできます。

```typescript
/**
 * T が U に代入できる( T が U を継承している ) 場合は X型, そうでなければ Y型 
 */
T extends U ? X : Y
```

これが構文になります。そして実際の使い方が下記です。
以下は条件となる継承元の型に一致する型が指定されているケース。

```typescript
// まず最初に ConditinoType としての条件型を定義する
// ------
// never型 については下記参照
// https://typescriptbook.jp/reference/statements/never
type TBaseCondition<T, U> = T extends U ? T : never;

// 定義した条件型を指定して利用できる型を設定する
type TCondition = TBaseCondition<string | boolean | number, boolean | string[]>;
//                               ~~~~~~~~~~~~~~~~~~~~~~~~~~ ~~~~~~~~~~~~~~~~~~
//                                    T extends U の `T`     T extends U の `U`

// boolean のみが条件となるクラスを継承しているので
// TCondition は
// 
// type TCondition = boolean
// 
// となる
```

コメントに記載のとおり `boolean` が一致しているので、`Tcondition` は `boolean` となります。
逆に下記は条件となる継承元の型に一致しないケースです。

```typescript
// 定義した条件型を指定して利用できる型を設定する
type TConditionNever = TBaseCondition<string | boolean | number, number[] | string[]>;
//                                    ~~~~~~~~~~~~~~~~~~~~~~~~~~ ~~~~~~~~~~~~~~~~~~~~
//                                         T extends U の `T`     T extends U の `U`

// どの型も条件となるクラスを継承していないので
// TConditionNever は
// 
// type TConditionNever = never
// 
// となる
```

コメントに記載のとおり、どの型にも一致しないので `never` が設定されます。

## infer を使って表現することもできる

Conditional Types ( 条件型 ) を扱う際に用いるキーワードで、これを使うと型マッチングなるものが可能になるとのこと。
型マッチングでなに？というところでコードで確認します。
勉強不足に付きうまい例が浮かばなかったので、次のコードは TypeScript の `ReturnType` から拝借してます。
( 参考: https://cloudsmith.co.jp/blog/frontend/typescript/2023/02/2293805.html )

```typescript
type SampleReturnType<T> = T extends (...args: any[]) => infer T ? T : any;
```

これで **任意の型と個数の引数を受け取り** 戻り値は **任意の型である any で返す** 型が出来上がるそうです。
その使い方は次のとおりです。

```typescript
// 戻り値が string である 
const sampleFunction = (arg1: number, arg2: string) => {
  return `${arg1.toString()}+${arg2}`;
};

// sampleFunction を SampleReturnType に噛ませる
type ReturnedType = SampleReturnType<typeof sampleFunction>;  // -> string

//
// type ReturnedType = string
//
// となる
```

ここで `sampleFunction` の戻り値を `number` で返してやるようにしてみます。

```typescript
const sampleFunction = (arg1: number, arg2: string) => {
  return Number(`${arg1.toString()}+${arg2}`);
};
type ReturnedType = SampleReturnType<typeof sampleFunction>;  // -> number

//
// type ReturnedType = number
//
// となる
```

と、このように、**`infer` を介して型推論が行われ**、その推論した型が定義される、ということが確認できました。

正直、普段 TypeScript を使っていてお目にかかることはない機能だと思います( 恥ずかしながら実際のところ、改めて勉強するまで見たこともなかった )し、今後も使うことは恐らくないと思いますが、こういうものだと知っておくことは無駄にはならないと思いますので挙げてみました。

# 型関数

既存の型をベースに新しい型を作る仕組みです。
数が多いので先に表でまとめます。

| カテゴリ         | 型関数                       | 変換元 -> 変換先          | やってくれること                         |
| ------------ | ------------------------- | ------------------- | -------------------------------- |
| フィールドの属性を変える | Partial\<T>               | interface -> type   | 指定した型のすべてのフィールドを オプショナル にしてくれる   |
|              | Reaquired\<T>             | interface -> type   | 指定した型のすべてのフィールドを 必須 にしてくれる       |
|              | Readonly\<T>              | interface -> type   | 指定した型のすべてのフィールドを readonly にしてくれる |
| 新しい型を返す      | Record\<K, T>             | interface -> type   | 指定した型のフィールドを持つ型を新たに生成してくれる       |
|              | Pick\<T, K>               | interface -> type   | 指定した型から特定のフィールドだけを抽出してくれる        |
|              | Omit\<T, K>               | interface -> type   | 指定した型から特定のフィールドだけを除去してくれる        |
|              | Exclude\<T, U>            | type -> type        | ベースの型から指定した型を除去してくれる             |
|              | Extract\<T, U>            | type -> type        | ベースの型から指定した型を抽出してくれる             |
|              | NonNullable\<T>           | type -> type        | ベースの型から `null`                   |
|              | Parameters\<T>            | function -> type    | 指定した関数の引数からタプル型を生成してくれる          |
|              | ReturnType\<T>            | function -> type    | 指定した関数の戻り値を新しい型として生成してくれる        |
|              | ConstructorParameters\<T> | constructor -> type | コンストラクタの引数からタプル型を生成してくれる         |

## フィールドの属性を変える

### Partial\<T>

指定した型のすべてのフィールドを オプショナル にしてくれます。

```typescript
interface IHoge {
  hoge: string;
  piyo: number;
  bar: boolean;
}

// IHoge のフィールドは以下のとおり
// (property) IHoge.hoge: string
// (property) IHoge.piyo: number
// (property) IHoge.bar: boolean

// すべてのフィールドを オプショナル にする
type PHoge = Partial<IHoge>;

// PHoge は
//
// type PHoge = {
//   hoge?: string | undefined;
//   piyo?: number | undefined;
//   bar?: boolean | undefined;
// }
//
// となる
```

### Reaquired\<T>

指定した型のすべてのフィールドを 必須 にしてくれます。

```typescript
interface IOHoge {
  hoge?: string;
  piyo?: number;
  bar?: boolean;
}

// IOHoge の各フィールドは以下のとおり
// (property) IOHoge.hoge?: string | undefined
// (property) IOHoge.piyo?: number | undefined
// (property) IOHoge.bar?: boolean | undefined

// すべてのフィールドを 必須 にする
type RHoge = Required<IOHoge>;

// RHoge は
//
// type RHoge = {
//   hoge: string;
//   piyo: number;
//   bar: boolean;
// }
//
// となる
```

### Readonly\<T>

定した型のすべてのフィールドを readonly にしてくれます。

```typescript
interface IHoge {
  hoge: string;
  piyo: number;
  bar: boolean;
}

// すべてのフィールドを Readonly にする
type RHoge = Readonly<IHoge>;

// RHoge は
//
// type RHoge = {
//   readonly hoge: string;
//   readonly piyo: number;
//   readonly bar: boolean;
// }
//
// となる
```

## 新しい型を返す

### Record\<K, T>

指定した型のフィールドを持つ型を新たに生成してくれます。

```typescript
interface IHoge {
  hoge: string;
  piyo: number;
  bar: boolean;
}

// フィールド A の値の型が IHoge である型を作る
type RHoge = Record<'A', IHoge>;

// RHoge は
// 
// type RHoge = {
//   A: IHoge;
// }
//
// となる

// RHoge を型指定した変数は次のような値をとる
const rHoge: RHoge = {
  A: {
    hoge: 'hoge',
    piyo: 1,
    bar: false,
  },
}
```

### Pick\<T, K>

指定した型から特定のフィールドだけを抽出してくれます。

```typescript
interface IHoge {
  hoge: string;
  piyo: number;
  bar: boolean;
}

// IHoge をベースに `hoge` フィールドだけをもつ PHoge を作る
type PHoge = Pick<IHoge, 'hoge'>;

// PHoge は
// 
// type PHoge = {
//   hoge: string;
// }
//
// となる

// PHoge を型指定した変数に対して `hoge` 以外のフィールドを指定するとエラーになる
//
// Type '{ hoge: string; piyo: number; }' is not assignable to type 'PHoge'.
//   Object literal may only specify known properties, and 'piyo' does not exist in type 'PHoge'.(2322)
const rHoge: PHoge = {
  hoge: 'hoge',
  piyo: 1  // ◀ これが不要. PHoge には存在しないフィールド
}
```

### Omit\<T, K>

指定した型から特定のフィールドだけを除去してくれます。

```typescript
interface IHoge {
  hoge: string;
  piyo: number;
  bar: boolean;
}

// IHoge をベースに `hoge` フィールドを除外した OHoge を作る
type OHoge = Omit<IHoge, 'hoge'>;

// OHoge は
// 
// type OHoge = {
//   piyo: number;
//   bar: boolean;
// }
//
// となる

// OHoge を型指定した変数に対して `hoge` フィールドを指定するとエラーになる
//
// Type '{ hoge: string; piyo: number; bar: true; }' is not assignable to type 'OHoge'.
//   Object literal may only specify known properties, and 'hoge' does not exist in type 'OHoge'.(2322)
const rHoge: OHoge = {
  hoge: 'hoge',
  piyo: 1,
  bar: true,
}
```

### Exclude\<T, U>

これまでは intrerface を対象にしていましたが、こちらは対象の型が **共用型** になります。
ベースの型から指定した型を除去してくれます。 

```typescript
type THoge = string | number | boolean;

// THoge から boolean を取り除いた型をつくる
type ExTHoge = Exclude<THoge, boolean>;

// ExTHoge　は
//
// type ExTHoge = string | number
//
// となる


type THoge2 = string | number;

// THoge2 に無い型を指定してもエラーにはならない
type ExTHoge2 = Exclude<THoge2, boolean>;

// ExTHoge2 は
//
// type ExTHoge2 = string | number
//
// となる
```

### Extract\<T, U>

こちらも対象の型が **共用型** になります。
ベースの型から指定した型を抽出してくれます。

```typescript
type THoge = string | number | boolean;

// THoge から boolean と number を抽出した型をつくる
type ExTHoge = Extract<THoge, boolean | number>;

// ExTHoge　は
//
// type ExTHoge = number | boolean
//
// となる


type THoge2 = string | number;

// THoge2 に無い型を指定してもエラーにはならない
type ExTHoge2 = Extract<THoge2, boolean | number>;

// ExTHoge2
//
// type ExTHoge2 = number
//
// となる
```

### NonNullable\<T>

こちらも対象の型が **共用型** になります。
ベースの型から `null` や `undefined` を除外してくれます。

```typescript
type THoge = string | number | boolean | undefined;

// THoge から null, undefined を取り除いた型をつくる
type NonNullableTHoge = NonNullable<THoge>;

// NonNullableTHoge
//
// type NonNullableTHoge = string | number | boolean
//
// となる
```

### Parameters\<T>

ここからはベースとなる対象が **関数** になります。
指定した関数の引数からタプル型を生成してくれます。

```typescript
// string, number, boolean を引数にもつ関数
const fHoge = (arg1: string, arg2: number, arg3: boolean) => {}

// これを typeof とともに Parameters<T> に噛ませると
type THoge = Parameters<typeof fHoge>;

// THoge は
//
// type THoge = [arg1: string, arg2: number, arg3: boolean]
//
// となる

// THoge を型指定した変数には string, number, boolean を設定できる
const tHoge: THoge = [
  'hoge',
  1,
  false,
];

// 指定する順序が違うとエラー
// Type 'boolean' is not assignable to type 'number'.(2322)
const tHoge2: THoge = [
  'hoge',
  false,
  1,
];

// 指定する要素が足りなくてもエラー
// Type '[string, number]' is not assignable to type '[arg1: string, arg2: number, arg3: boolean]'.
//  Source has 2 element(s) but target requires 3.(2322)
const tHoge3: THoge = [
  'hoge',
  1,
];
```

### ReturnType\<T>

指定した関数の戻り値を新しい型として生成してくれます。

```typescript
// 戻り値が string, number である関数
const fHoge = (arg1: string, arg2: number, arg3: boolean): string | number => {
  return ''
}

// これを typeof とともに ReturnType<T> に噛ませると
type THoge = ReturnType<typeof fHoge>;

// THoge は
//
// type THoge = string | number
//
// となる

// THoge を型指定した変数には string か number を設定できる
const tHoge: THoge = 'hoge';
const tHoge2: THoge = 1;

// string | number 以外を指定するとエラー
// Type 'boolean' is not assignable to type 'string | number'.(2322)
const tHoge3: THoge = false;
```

### ConstructorParameters\<T>

コンストラクタの引数からタプル型を生成してくれます。

```typescript
// constructor の引数に string, number, boolean を持つ class CHoge
class CHoge {
  constructor(arg1: string, arg2: number, arg3: boolean) {}
}

// これを typeof とともに ConstructorParameters<T> に噛ませてやる
type TCHoge = ConstructorParameters<typeof CHoge>;

// TCHoge は
//
// type TCHoge = [arg1: string, arg2: number, arg3: boolean]
// 
// となる

// TCHoge を型指定した変数には string, number, boolean を設定できる
const tcHoge: TCHoge = [
  'hoge',
  1,
  false,
];

// 指定する順序が違うとエラー
// Type 'boolean' is not assignable to type 'number'.(2322)
const tcHoge2: TCHoge = [
  'hoge',
  false,
  1,
];

// 指定する要素が足りなくてもエラー
// Type '[string, number]' is not assignable to type '[arg1: string, arg2: number, arg3: boolean]'.
//   Source has 2 element(s) but target requires 3.(2322)
const tcHoge3: TCHoge = [
  'hoge',
  1,
];
```
