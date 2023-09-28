## Diamond Problem って？
[Reactive プログラミング](https://developer.mamezou-tech.com/blogs/2022/07/19/reactive-programming/) において 割りと古くからとりあげられるトピックです。

今回はいくつかの記事を見ながら Diamond Problem について触れていきます。

### お詫びから
Diamond Problem について言及されている記事を探したさい、海外の記事ばかりがヒットしたためそちらを参照してます。
記事の解釈に誤りがありましたら、申し訳ありませんがご指摘お願いします。

## [RX GLITCHES AREN'T ACTUALLY A PROBLEM](https://staltz.com/rx-glitches-arent-actually-a-problem.html) から引用
`Observable` な情報における不整合が発生するケースを示すもので、これを示す例として [こちらの記事](https://staltz.com/rx-glitches-arent-actually-a-problem.html) には次の一文があります。

> **When do glitches happen?** Usually glitches are demonstrated by giving the classical diamond case. An Observable A is transformed into Observables B and C, then those are combined to create Observable D:
> (翻訳)
> 不具合はどのような時に起こるのでしょうか？通常、不具合は古典的なダイヤモンドのケースで示されます。観測値Aが観測値BとCに変換され、それらが組み合わされて観測値Dが作られる：
>
> ```typescript
> const alphabet = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h'];
> const a = Observable.timer(0, 1000);  // 0-----1-----2-----3-----4------
> const b = a.map(i => alphabet[i]);    // a-----b-----c-----d-----e------
> const c = a.map(i => i * i);          // 0-----1-----4-----9-----16-----
> const concat = (_1, _2) => String(_1).concat(String(_2));
> const d = b.combineLatest(c, concat); // a0-----(b0b1)(c1c4)(d4d9)(e9e16)
>                          // desired was: a0-----b1----c4----d9----e16---
>  ```

ほかにも Diamond Problem について触れている記事を紹介します。

## [I changed my mind. Angular needs a reactive primitive](https://dev.to/this-is-angular/i-changed-my-mind-angular-needs-a-reactive-primitive-n2g) から引用
 **Component inputs and the diamond problem** の項目では以下の記述があります。

> If each component input gets a new value at the same time, each input observable would fire in succession, so if you needed to combine input values inside the component, a `combineLatest` would fire once for each input observable. Selectors can't help with this, since component inputs aren't part of the global store.
>
> This problem actually has a name: [The diamond problem](https://dev.to/modderme123/super-charging-fine-grained-reactive-performance-47ph#reactive-algorithms).
> (翻訳)
> 各コンポーネントの入力が同時に新しい値を取得すると、各入力observableが連続して起動するため、コンポーネント内で入力値を組み合わせる必要がある場合、combineLatestが各入力observableに対して1回起動することになる。コンポーネントの入力はグローバルストアの一部ではないので、セレクタはこれを助けることができません。  
>
> この問題には、実は「ダイヤモンド問題」という名前がついています。

## [# Super Charging Fine-Grained Reactive Performance](https://dev.to/modderme123/super-charging-fine-grained-reactive-performance-47ph) から引用

こちらは前項の引用文でリンクされた記事のセクションですが、 **[Reactive Algorithms](https://dev.to/modderme123/super-charging-fine-grained-reactive-performance-47ph#reactive-algorithms)** で触れています。
(画像も引用元から拝借)

> The charts below consider how a change being made to `A` updates elements that depend on `A`.
>
> Let's consider two core challenges that each algorithm needs to address. The first challenge is what we call the diamond problem, which can be an issue for eager reactive algorithms. The challege is to not accidentally evaluate `A,B,D,C` and then evaluate `D` a second time because `C` has updated.  
Evaluating `D` twice is inefficient and may cause a user visible glitch.
> (翻訳)
> 以下のグラフは、Aを変更することで、Aに依存する要素がどのように更新されるかを考えています。  
>  
> ここで、各アルゴリズムが取り組むべき2つの中核的な課題について考えてみましょう。最初の課題は、ダイヤモンド問題と呼ばれるもので、eager reactiveアルゴリズムで問題となることがあります。この問題は、A,B,D,Cを誤って評価し、Cが更新されたために2回目のD評価を行わないようにすることです。  
Dを2回評価することは非効率的であり、ユーザーの目に見える不具合を引き起こす可能性があります。
>
> <img width="125" alt="diamond-problem-diagram.png (2.0 kB)" src="https://img.esa.io/uploads/production/attachments/17638/2023/06/16/102830/f7dd1fdf-40ca-4d5e-9212-afb9146f4205.png">

## まとめ
複数のストリームを扱うさい、意図せずにそれぞれのストリームで流れてきた値を処理するケースが発生することがあり、これを Diamond Problem と呼びます。
それを回避するためには...

1. Signals を使用する
[前掲の記事](https://dev.to/this-is-angular/i-changed-my-mind-angular-needs-a-reactive-primitive-n2g) にもあるように [Signals](https://angular.jp/guide/signals) を利用するが良い方法のようです。
私自身、まだ理解が浅いためここでは紹介のみに留めさせて頂きます。

2. `combineLatest` を使う際は注意を払う
`combineLatest` の性質上、Diamond Problem を引き起こす可能性が高いです。それは [こちらの記事](https://blog.strongbrew.io/combine-latest-glitch/) でも言及されています。



## 参考
- [リアクティブプログラミング (Reactive Programming)](https://developer.mamezou-tech.com/blogs/2022/07/19/reactive-programming/)
- [I changed my mind. Angular needs a reactive primitive](https://dev.to/this-is-angular/i-changed-my-mind-angular-needs-a-reactive-primitive-n2g) > #Component inputs and the diamond problem
- [Super Charging Fine-Grained Reactive Performance > #Reactive Algorithms](https://dev.to/modderme123/super-charging-fine-grained-reactive-performance-47ph#reactive-algorithms)
- [RX GLITCHES AREN'T ACTUALLY A PROBLEM](https://staltz.com/rx-glitches-arent-actually-a-problem.html)
- [A glitch in combineLatest (and how to fix it!)](https://blog.strongbrew.io/combine-latest-glitch/)