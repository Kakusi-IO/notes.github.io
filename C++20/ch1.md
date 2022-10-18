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





