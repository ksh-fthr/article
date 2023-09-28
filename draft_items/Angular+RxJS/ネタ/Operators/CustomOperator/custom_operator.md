## カスタムオペレータを自作する
公式には [こちら](https://rxjs.dev/guide/operators#creating-custom-operators) にサンプルがあります。
これをベースに実装の仕方と動きを見ていきます。

### 関数 | メソッドの形
カスタムオペレータは [pipe](https://rxjs.dev/api/index/function/pipe) を使って実装します。
その形は次のとおりです。

```typescript
const somethingToDo = (): UnaryFunction<Observable<T>, Observable<T> => {
  return pipe(
    // pipe 内に行いたい処理を記述して、それを戻り値とする
    // 処理する対象の値は上位のストリームから流れてきた値
  );
}
```

#### ポイント
- 関数 | メソッドとして
    - カスタムオペレータは関数、もしくはクラスの static メソッド、ないしインスタンスメソッドとしても定義できる
    - 汎用性をもたせる意味で `export` つきの外部関数とするのも良いし、ある役割であることを明示する意図のもとクラスを用意するの良い
- 戻り値の型
    - 戻り値の型は [UnrayFunction](https://rxjs.dev/api/index/interface/UnaryFunction) 型.
        - [UnrayFunction](https://rxjs.dev/api/index/interface/UnaryFunction)
        - あるパラメータTを受け取り、別のパラメータRを返す関数を記述する関数型インターフェイスです。
    - ジェネリクスに指定する型は pipe 内で行われた処理で戻される型
- 実装
    - `pipe` の引数に処理したい実装を記述する
    - 処理する対象の値は上位のストリームから流れてきた値. カスタムオペレータの引数で渡されるものではないことに注意

### サンプルコード
次のサンプルコードはシンプルなものですが、カスタムオペレータの感覚を掴むのにちょうど良いと思います。
実装している内容は以下の 2点 です。

- 偶数を返すカスタムオペレータ
- 引数を受け取り、その引数で割り切れる値を返すカスタムオペレータ

```typescript
import { filter, Observable, of, pipe, UnaryFunction } from "rxjs";

/**
 *  カスタムオペレーター: 偶数のみ抽出する
 */
const filterEvenNumber = (): UnaryFunction<Observable<number>, Observable<number>> => {
  return pipe(
    // 処理対象の x は上位のストリームから流れてきた値
    filter((x: number) => x % 2 === 0)
  );
}

/**
 *  カスタムオペレーター: 指定した値で割り切る値を算出
 */
const divisionBy = (divisor: number): UnaryFunction<Observable<number>, Observable<number>> => {
  return pipe(
    // 処理対象の x は上位のストリームから流れてきた値
    filter((x: number) => x % divisor === 0)
  );
}

// 1~10 までの値をストリームに流し、カスタムオペレータでフィルタした値をログに出す
const bSubject = of(1,2,3,4,5,6,7,8,9,10);
bSubject.pipe(
  filterEvenNumber(), // 偶数を返す
  divisionBy(4)       // 4 で割り切れる値を返す
).subscribe({
  next: (x) => {
    console.log(x);
  },
  error: (err) => {
    console.log(err)
  },
  complete() {
    console.log('done');
  },
});

// Logs:
// 4
// 8
// done
```

## 別のストリームを処理してみる
上述のサンプルコードでは流れてきたストリームに対して処理しました。
では流れてきたストリームとは別のストリームを結合して処理できるのか？　ここではそれについて見てみます。

以下は前傾のコードに次の 2点 を加えたものです。

- 新たなストリームを流す  `anotherSubject` 
- `bSubject` と `anotherSubject` の値をうけとり、両者の同値のデータを返すカスタムオペレータ

```typescript
import { filter, map, Observable, of, pipe, UnaryFunction, withLatestFrom } from "rxjs";

/**
 *  カスタムオペレーター: 偶数のみ抽出する
 */
const filterEvenNumber = (): UnaryFunction<Observable<any>, Observable<any>> => {
  return pipe(
    filter((x: number) => x % 2 === 0)
  );
}

/**
 *  カスタムオペレーター: 指定した値で割り切る値を算出
 */
const divisionBy = (divisor: number): UnaryFunction<Observable<number>, Observable<number>> => {
  return pipe(
    filter((x: number) => x % divisor === 0)
  );
}


//////////////////////////////////
const anotherSubject = of(8);

/**
 * カスタムオペレーター: 別のストリームの値と同値のものを抽出
 */
const filterSameValue = () => {
  return pipe(
    withLatestFrom(anotherSubject),
    filter(([x, y]) =>  x === y),
    map(([x, _]) => x)
  );
}

const bSubject = of(1,2,3,4,5,6,7,8,9,10);
bSubject.pipe(
  filterEvenNumber(),  // 偶数を返す
  divisionBy(4),       // 4 で割り切れる値を返す
  filterSameValue()    // bSubject のストリームと anotherSubject のストリームで一致した値を返す
).subscribe({
  next: (x) => {
    console.log(x);
  },
  error: (err) => {
    console.log(err)
  },
  complete() {
    console.log('done');
  },
});

// Logs:
// 8
// done
```

## 別のストリームは引数で受け取ることもできる
ここで見るコードは前項で挙げたものとほとんど一緒です。
違いは以下の2点です。( 引数を受け取っているので関数名も修正してます )

- `filterSameValueAs` 関数の引数に `Observable` を与えた
- `filterSameValueAs` 関数で処理するストリームが引数で与えられた `Observable` になった

```typescript
import { filter, map, Observable, of, pipe, UnaryFunction, withLatestFrom } from "rxjs";

/**
 *  カスタムオペレーター: 偶数のみ抽出する
 */
const filterEvenNumber = (): UnaryFunction<Observable<any>, Observable<any>> => {
  return pipe(
    filter((x: number) => x % 2 === 0)
  );
}

/**
 *  カスタムオペレーター: 指定した値で割り切る値を算出
 */
const divisionBy = (divisor: number): UnaryFunction<Observable<number>, Observable<number>> => {
  return pipe(
    filter((x: number) => x % divisor === 0)
  );
}


//////////////////////////////////
const anotherSubject = of(8);

/**
 * カスタムオペレーター: 別のストリームの値と同値のものを抽出
 */
const filterSameValueAs = (stream: Observable<number>) => {
  return pipe(
    withLatestFrom(stream),
    filter(([x, y]) =>  x === y),
    map(([x, _]) => x)
  );
}

const bSubject = of(1,2,3,4,5,6,7,8,9,10);
bSubject.pipe(
  filterEvenNumber(),               // 偶数を返す
  divisionBy(4),                    // 4 で割り切れる値を返す
  filterSameValueAs(anotherSubject) // bSubject のストリームと anotherSubject のストリームで一致した値を返す
).subscribe({
  next: (x) => {
    console.log(x);
  },
  error: (err) => {
    console.log(err)
  },
  complete() {
    console.log('done');
  },
});

// Logs:
// 8
// done
```

特筆することはありませんが、引数で任意の Observable を受けることもできる、ということをご紹介させていただきました。

### 通常のメソッドをカスタムオペレータでラップしてみる
カスタムオペレータは `Observable` を扱わないメソッドもラップして使用できます。
次の例は前掲の 偶数抽出 のカスタムオペレータの計算部分をメソッドに切り出し、それをラップした単純なものですが、雰囲気を掴んで頂ければと思います。

```typescript
import { filter, map, Observable, of, pipe, UnaryFunction, withLatestFrom } from "rxjs";

/**
 *  偶数算出
 */
const calculateEvenNumber = (x: number) => {
    return x % 2 === 0
}

/**
 *  カスタムオペレーター: 偶数のみ抽出する
 */
 const filterEvenNumber = (): UnaryFunction<Observable<any>, Observable<any>> => {
  return pipe(
    // 計算部分を別メソッドに切り出した
    // ( 言い換えると、目的に沿うのであれば既存メソッドをラップすることでカスタムオペレータを実装できる )
    filter((x: number) => calculateEvenNumber(x))
  );
}


const bSubject = of(1,2,3,4,5,6,7,8,9,10);
bSubject.pipe(
  filterEvenNumber(),            // 偶数を返す
).subscribe({
  next: (x) => {
    console.log(x);
  },
  error: (err) => {
    console.log(err)
  },
  complete() {
    console.log('done');
  },
});

// Logs:
// 2
// 4
// 6
// 8
// 10
// done
```

## ユースケース
- コード可読性の向上
    - pipe 内で `map` や `filter` 等々、RxJS で提供されるオペレータを使って処理を書く事が殆どだと思いますが、それだと `pipe` 内のコード量が多くなりがちです
    - それぞれのオペレータでやりたいことをカスタムオペレータ化させること、また適切な関数 | メソッド名をつけることで、`pipe` 内のコード量の削減と見通しの良さの実現を期待できます
    - もう一つ付け加えると、`pipe` 内で行う処理の流れをカスタムオペレータに閉じ込めることで、その処理の流れは まとまりとして必要なものである ということを明示できるメリットもあります
    - ( つまり安易に変更しないでくださいね、と注意する役目も持たせることができる )
- 既存関数 | メソッド の流用
    - Observable を処理する目的ではなく実装されていたメソッドを `pipe` 内で使いたい、というケースが合った場合、カスタムオペレータでラップすることで対応できます

## 参考
-  [3-3.Observable のリファクタリング](https://zenn.dev/lacolaco/books/angular-after-tutorial/viewer/rxjs-custom-operator)
