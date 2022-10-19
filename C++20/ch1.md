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

## 機能検査マクロ

### 使い方

`<version>`の`__cpp_lib_xxx`を使う。

```C++
// <version>
#include <version>
#ifdef __cpp_lib_three_way_comparison
#   include <compare>
#else
#   error Spaceship has not yet landed
#endif
```

もしくは、C++17で導入された`__has_include()`を使う。

```C++
// __has_include
#if __has_include(<compare>)
#   include <compare>
#else
#   error Spaceship has not yet landed
#endif
```

## Safer  templates using concepts

### 使い方

`<concepts>`を使用。(C++20)

```C++
#include <iostream>
#include <concepts>

// std::integralは、<concepts>の中にいるconceptの一種
// ここで、複数conceptsを結合する(||を使う)
template <typename T> 
concept Numeric = std::integral<T> || std::floating_point<T>;

template <typename T> requires Numeric<T>
T arg42(const T & arg) {
    return arg + 42;
}

int main()
{
    std::cout<< arg42(0) <<'\n';
}
```

もしくは、`<type_traits>`を使う。(C++11)

```C++
template<typename T> requires is_integral<T>::value  // value is bool
constexpr double avg(vector<T> const& vec) {
    double sum{ accumulate(vec.begin(), vec.end(),
      0.0)
    };
    return sum / vec.size();
}

// traitsはもともと、constexpr boolのtemplate
template<typename T>　constexpr bool is_gt_byte{ sizeof(T) > 1 };
```

組み合わせて、

```C++
template<typename T>
concept Numeric = is_gt_byte<T> && (integral<T> || floating_point<T>);
```

その他の形式：

```C++
template<Numeric T>
T arg42(const T & arg) {
    return arg + 42;
}
```

```C++
// templateキーワードは不要
// typenameをautoに変える
// autoの前に、限定Numericを表す
auto arg42(Numeric auto & arg) {
    return arg + 42;
}
```

## Ranges and Views

`<ranges>`に参照

- Range: begin()とend()の間の要素
- View: rangeから作成された、lazyなrange
- View Adapter: RangeからViewを作成するもの

### 使い方

```C++
namespace ranges = std::ranges;
namespace views = std::views;

// ranges::take_view
const vector<int> nums{ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
auto result = ranges::take_view(nums, 5);
for (auto v: result) cout << v << " ";

// views::take
auto result = nums | views::take(5);
for (auto v: result) cout << v << " ";

// View Adapter
auto result = nums | views::take(5) | views::reverse;

// views::filter()
auto result = nums | views::filter([](int i){ return 0 == i % 2; });

// views::transform()
auto result = nums | views::transform([](int i){ return i * i; });

// views::iota(value, bound)、新しく臨時rangeを作成
auto rnums = views::iota(1) | views::take(200);
```



