# [TypeScript] reduce を使って新しいオブジェクトを生成する

# はじめに

やりたいことはタイトルのとおりです。この記事では次の2つの要件に対する実装案を提示します。

- 要件-1
  - 既存のオブジェクトから特定のフィールドを取り出して新しいオブジェクトを生成する
- 要件-2
  - 既存のオブジェクトから全く同じフィールドをもつ新しいオブジェクトを生成する

# 要件-1: 特定のフィールドを取り出して新しいオブジェクトを生成する

下記のような `objectA` があって、ここからフィールド `a`, `b` だけを抽出した新しい `objectB` を生成したいケースを想定します。

```typescript
interface ModelType {
    [key: string]: string    
}

const objectA: ModelType = {
    'a': 'AAAA',
    'b': 'BBBB',
    'c': 'CCCC'
};
```

あらかじめ抽出したいフィールドのキー値を指定した配列を用意します。
そのキー値配列に対して [reduce](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce) を使って `objectA` からデータを設定していきます。

```typescript
const keys: string[] = [
    'a',
    'b'    
];

// キー値の配列にあるフィールドだけを取り出して objectB を作る
// このやり方だと if 文 や filter を使ったフィルタリングがいらない
const objectB = keys.reduce(
    (obj, key) => {
        obj[key] = objectA[key];
        return obj;
    },
    // 型指定をしないと TS7053 で下記が発生する
    // Element implicitly has an 'any' type because expression of type 'string' can't be used to index type '{}'.
    // No index signature with a parameter of type 'string' was found on type '{}'.
    {} as ModelType
);

console.log(objectB); // { 'a': 'AAAA', 'b': 'BBBB' } が出力される
```

# 要件-2: 全く同じフィールドをもつ新しいオブジェクトを生成する

前掲の `objectA` から同じフィールドを持つ `objectC` を作りたいケースです。
( 要は ObjectA のコピーです, [Lodash の cloneDeep](https://lodash.com/docs/4.17.15#cloneDeep) を使いたくないときとかの代替としての実装案です )

`objectC` が持ちたいフィールドは `objectA` と同じなので、`objectA` からキー値のフィールドを生成します。
あとは 要件-1 と同じです。生成したキー値配列に対して [reduce](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce) を使って `objectA` からデータを設定していきます。


```typescript
// キー値の配列を objectA から作る
const keys2 = Object.keys(objectA) as (keyof typeof objectA)[];
console.log(keys); // ["a", "b", "c"]  が出力される

// 作成したキー値配列から reduce を使って objectA から objectC を作る
const objectC = keys2.reduce(
    (obj, key) => {
        obj[key] = objectA[key];
        return obj;
    },
    {} as ModelType
);

console.log(objectC); // { 'a': 'AAAA', 'b': 'BBBB', 'c': 'CCCC' } が出力される 
```

上記はキー値配列の生成と object の生成をつなげて一つの処理にまとめることも出来ます。

```typescript
// objectA から キー値配列を作り、reduce を使って新しい objectD を作る処理をひとまとめに行う
const objectD = (
    Object.keys(objectA) as (keyof typeof objectA)[]
).reduce(
    (obj, key) => {
        obj[key] = objectA[key];
        return obj;
    },
    {} as ModelType
);

console.log(objectD); // { 'a': 'AAAA', 'b': 'BBBB', 'c': 'CCCC' } が出力される 
```

# 補足

オブジェクトの比較をしたい場合は  [Lodash の isEqual](https://lodash.com/docs/4.17.15#isEqual) を使えば簡単にできます。

```typescript
import * as _ from 'lodash';

console.log(_.isEqual(objectA, objectB)); // fase が出力される
console.log(_.isEqual(objectA, objectC)); // true が出力される
console.log(_.isEqual(objectA, objectD)); // true が出力される

```

# 参考
- [MDN Web Docs - Array.prototype.reduce()](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)
- [TypeScript Documentation - Keyof Type Operator](https://www.typescriptlang.org/docs/handbook/2/keyof-types.html)
