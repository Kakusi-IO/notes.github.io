# New C++20 Features

## Format

- printf: 型検査がない。脆弱しすぎる。
- <<: overloadによって実装。複雑しすぎる。
- 対策はformat

### 使い方

```C++
format("some text{}\n", 123);
// place holder{}の中で順序指定が可能
format("some text {1} {0}\n", 123, 789);

// コロンで始める、桁数を指定
format(":.5", pi);
format(":.<10", 42); // 合計10文字出力して、"."で補足して、"42"を左端で表示
format(":.>10", 42);
format(":,^10", 42);
```

### その他

`std::print()`　C++23実装予定

## Complie-time string and vector

### 使い方

```C++
constexpr auto use_string() {
    string str{"string"};
    return str.size();
}
```

コンパイル時にこの関数の戻り値はすでに決めたので、メモリーが解放される。

> constexpr vs const:
>
> 両方とも一度初期化したらたら変更できない。
>
> それで、constexprは、コンパイル時に決定されるので、より高速的。

## Compare ints safely

intとunsignedを比較する際に、予定外の結果が出る可能性がある。

### 使い方

```C++
#include <utility>
cmp_less(x, y);
cmp_equal(x, y);
cmp_greater(x, y);
cmp_greater_equal(x, y);
```

## Three-way comparisons

`<=>`演算子で3-way比較。結果は`<compare>`ヘッダで定義している。

```c++
// int間の比較
strong_ordering::equal    // operands are equal
strong_ordering::less     // lhs is less than rhs
strong_ordering::greater  // lhs is greater than rhs
    
// floatが入る
partial_ordering::equivalent  // operands are equivelant
partial_ordering::less        // lhs is less than rhs
partial_ordering::greater     // lhs is greater than rhs
partial_ordering::unordered   // if an operand is unordered
```

<0, ==0, >0とも取得できる。

ユーザー定義クラスで、六つの比較演算子を全部overload必要がなく、これ一つoverloadすればよい。

### 使い方

```C++
#include <compare>
static_assert((7 <=> 42), 0);
```

```C++
#include <compare>
struct Num {
    int a;
    constexpr Num(int a) : a{a} {}
    auto operator<=>(const Num&) const = default;
};
```

### 原理

別の比較演算子をoverloadしなっかたクラスは、たとえばa < bを使う時、自動的にa <=> b < 0に変換する。
