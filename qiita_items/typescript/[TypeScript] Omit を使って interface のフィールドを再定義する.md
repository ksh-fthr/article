# [TypeScript] Omit を使って interface のフィールドを再定義する

# はじめに
すでに定義されているインターフェースのフィールドはそのままに、特定のフィールドの型のみ変更したいケースがありました。
本記事では以下の 2ケース を扱い、その実現方法について提示します。

- ケース1: フラットなインターフェース
- ケース2: 入れ子となったインターフェース

# 前提

フィールドの型がすべて `string` のインターフェース `BaseModel` があります。
この `BaseModel` が本記事で扱うインターフェースのすべての基点です。

```typescript
interface BaseModel {
    'a': string;
    'b': string;
    'c': string;
    'd': string;
}
```

# ケース-1: フラットなインターフェース

インターフェース `BaseModel` に対して `a`, `b` のフィールドの型を `number` に変更したいです。
しかしながら、この `BaseModel` は複数の箇所で使われている可能性もあり、直接この `BaseModel` に対してフィールドの型を変更するのはリスキーです。

## そのまま継承して型を変更することはできない

かといって、`BaseModel` を継承してフィールド `a`,`b` の値を`number` にしてもエラーが発生します。
以下の定義は `a`, `b` を `number` で再定義しようとしていますが、`TS2430` が発生します。

```typescript
// 単純な再定義では TS2430 が発生する
interface ExtendBaseModelA extends BaseModel {
    'a': number;
    'b': number;
}
```

## Omit を使って対応する

こんなときは **[Omit](https://www.typescriptlang.org/docs/handbook/utility-types.html#omittype-keys) を使ってフィールドを削除** し、その上で **型を再定義したインターフェースを作る** ことで実現できます。

```typescript
// Omit を使って対象のフィールドを削除した後に再定義する
type OmitField = 'a' | 'b';
interface ExtendBaseModelB extends Omit<BaseModel, OmitField> {
    // 'a', 'b' を number で定義しても TS2430 は発生しない
    'a': number;
    'b': number;
}
```

## エラーの有無を検証する

`BaseModel` とそれを継承してフィールドの再定義を行った `ExtendBaseModelB` に対して値の設定を行い、エラーの有無を確認します。

### BaseModel の検証

`BaseModel` のフィールドの型はすべて `string` です。よって各フィールドに文字列を指定すればエラーは発生しません。

```typescript
// BaseModel の型をもつ baseModelA には全フィールドに対して string が設定できる
const baseModelA: BaseModel = {
    'a': 'AAAA',
    'b': 'BBBB',
    'c': 'CCCC',
    'd': 'DDDD',
};
```

ですが、`string` 以外の値を設定すると `TS2322` が発生します。

```typescript
// BaseModel の型をもつ baseModelB に string 以外の型を設定するとエラー(TS2322)となる
const baseModelB: BaseModel = {
    'a': 1, // TS2322 が発生する
    'b': 'BBBB',
    'c': 'CCCC',
    'd': 'DDDD',
}
```

### ExtendBaseModelB の検証

`BaseModel` を継承して作ったインターフェース `ExtendBaseModelB` は `a`, `b` のフィールドには `number` 型を指定しています。
したがって `a`, `b` に `number` を指定してもエラーにはなりません。

```typescript
// ExtendBaseModelB の型をもつ dxtendBaseModelB1 には 'a', 'b' には number が設定できる
const dxtendBaseModelB1: ExtendBaseModelB = {
    'a': 1,
    'b': 2,
    'c': 'CCCC',
    'd': 'DDDD',
};
```

逆に `a`, `b` に対して `string` を指定すると `TS2322` が発生します。

```typescript
// ExtendBaseModelB の型をもつ dxtendBaseModelB2 の 'a' や  'b' に string 以外の型を設定するとエラー(TS2322)となる
const dxtendBaseModelB2: ExtendBaseModelB = {
    'a': 1,
    'b': 'BBBB', // TS2322 が発生する
    'c': 'CCCC',
    'd': 'DDDD',
}
```

なお蛇足ですが、`a`, `b` のフィールドそのものを指定しない場合は `TS2739` が発生します。
これは `BaseModel` を指定していても同じです。

```typescript
// ExtendBaseModelB の型をもつ dxtendBaseModelB3 に 'a', 'b' を設定しないとエラー(TS2739)になる
const dxtendBaseModelB3: ExtendBaseModelB = {
    'c': 'CCCC',
    'd': 'DDDD',
}
```

# ケース-2: 入れ子となったインターフェース

インターフェースはすべてのフィールドがフラットで定義されているとは限りません。
別のインターフェースを指定したフィールドが存在する、いわゆる **入れ子構造** になっているケースもあります。

例えばフィールドの型がすべて `string` である `ChildBaseModel` があります。

```typescript
interface ChildBaseModel {
    'e': string;
    'f': string;
    'g': string;
}
```

そして `BaseModel` を継承した `ExtendBaseModelC` を定義し、型が `ChildBaseModel` である フィールド `cb` を持たせます。
この入れ子になったフィールド `cb` の型である **`ChildBaseModel` が持つフィールドの再定義** を見ていきます。

```typescript
interface ExtendBaseModelC extends BaseModel {
    cb: ChildBaseModel;
}
```

## やることは一緒

とはいえ、入れ子となっていてもやることは一緒です。
入れ子部分である `cb` の フィールド `g` の型を `number` にしたいとき、 [Omit](https://www.typescriptlang.org/docs/handbook/utility-types.html#omittype-keys) を使って新しく型を作ってやります。

```typescript
type OmitFieldCB = 'g';
interface ExtednChildModel extends Omit<ChildBaseModel, OmitFieldCB> {
    'g': number;
}

// ExtednChildModel を入れ子にしたモデルを定義する
interface ExtendBaseModelD extends BaseModel {
    cb: ExtednChildModel;
}
```

## エラーの有無を検証する

入れ子となったフィールドに対して値の設定を行い、エラーの有無を確認します。

### ExtendBaseModelC の検証

`ExtendBaseModelC` は入れ子のインターフェース含めてすべてのフィールドの型が `string` です。
以下はすべてのフィールドに対して文字列を設定しているのでエラーは発生しません。

```typescript
const extendBaseModelC: ExtendBaseModelC = {
    'a': 'AAAA',
    'b': 'BBBB',
    'c': 'CCCC',
    'd': 'DDDD',
    cb: {
        'e': 'EEEE',
        'f': 'FFFF',
        'g': 'GGGG'
    }
}
```

次のコードでは入れ子である `cb` のフィールド `g` に対して数値を設定しています。
このとき `cb[g]` には `TS2322` が発生します。これは文字列の定義に対して数値を設定したためです。

```typescript
const extendBaseModelC: ExtendBaseModelC = {
    'a': 'AAAA',
    'b': 'BBBB',
    'c': 'CCCC',
    'd': 'DDDD',
    cb: {
        'e': 'EEEE',
        'f': 'FFFF',
        'g': 3 // TS2322 が発生する
    }
}
```

### ExtendBaseModelD の検証

こんどは入れ子である `cb` の型が `ExtednChildModel` である `ExtendBaseModelD` について確認します。
`ExtednChildModelD` はフィールド `g` の型が `number` です。

以下は `cb[g]` を数値で設定してもエラーとなりません。

```typescript
const extendBaseModelD: ExtendBaseModelD = {
    'a': 'AAAA',
    'b': 'BBBB',
    'c': 'CCCC',
    'd': 'DDDD',
    cb: {
        'e': 'EEEE',
        'f': 'FFFF',
        'g': 3
    }
}
```

ですが、次のように `cb[g]` に数値以外を設定すると `TS2322` が発生します。

```typescript
const extendBaseModelD: ExtendBaseModelD = {
    'a': 'AAAA',
    'b': 'BBBB',
    'c': 'CCCC',
    'd': 'DDDD',
    cb: {
        'e': 'EEEE',
        'f': 'FFFF',
        'g': 'GGGG' // cb['g'] に number 以外を設定したら TS2322 が発生する
    }
}
```

# 参考
- [TypeScript Documentation > Utility Types > Omit](https://www.typescriptlang.org/docs/handbook/utility-types.html#omittype-keys)
- [サバイバルTypeScript > Omit](https://typescriptbook.jp/reference/type-reuse/utility-types/omit)
