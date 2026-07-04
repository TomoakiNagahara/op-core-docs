# `D()` の使い方

## 概要

`D()` は ONEPIECE Framework code で使う標準の debug output function です。

administrator にだけ見える debug output が必要な時に、application code や framework code から直接呼び出します。

## 基本の呼び出し

```php
D($value);
```

## label と値

`D()` は複数の引数を受け取れます。

よく使う形は、短い label を先に渡し、そのあとに確認したい値を渡す形です。

```php
D('admin', OP()->isAdmin());
```

これは、dump したい値が boolean、`null`、空文字、または debug output 上で label があると読みやすい値の場合に便利です。

## 複数の値

複数の値を 1 回の呼び出しで dump できます。

```php
D($request, $result, $status);
```

引数は 1 つの debug mark としてまとめられ、Dump unit がインストールされている場合は Dump unit によって rendering されます。

## 表示条件

`D()` output は、requester が administrator と判断された場合だけ表示されます。

administrator 判定については `asset/core/docs/is-admin.ja.md` を参照してください。

`D()` の背後にある技術的な流れは `asset/core/docs/d-function-overview.ja.md` を参照してください。
