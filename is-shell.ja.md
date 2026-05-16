# `isShell()` の技術仕様

## 範囲

この document は、`OP()->isShell()` の現行 technical behavior を説明します。

## 実装経路

`OP()->isShell()` は `OP_ENV` によって提供されます。

現在の path は次の通りです。

1. `OP()->isShell()`
2. `OP_ENV::isShell()`

現在の実装は次の file にあります。

- `asset/core/trait/OP_ENV.php`

## 判定ルール

現在の rule は次の通りです。

1. `PHP_SAPI` と `cli` を比較する
2. `PHP_SAPI === 'cli'` なら `true` を返す
3. それ以外は `false` を返す

code としては、現在の判定は次と同等です。

```php
return PHP_SAPI === 'cli';
```

## Cached Result はない

`isShell()` は現在、結果を static variable に保存しません。

呼び出されるたびに `PHP_SAPI` を直接確認します。

`PHP_SAPI` は process-level の PHP runtime value であり、ひとつの request lifecycle 中に変わらないため、この形で問題ありません。

## Framework 内での意味

`isShell()` は、framework における現在の PHP CLI detection method です。

shell execution behavior と HTTP execution behavior を分けるために使われます。

`isShell()` の影響を受ける behavior の例:

- request loading path selection
- MIME header output avoidance
- route calculation behavior
- localhost detection
- shell-safe debug または encoding behavior

## `isLocalhost()` との関係

`OP()->isLocalhost()` は最初に `isShell()` を確認します。

そのため、現在の設計では shell execution は localhost として扱われます。

## `isCLI()`

現在の `OP_ENV` には `isCLI()` method はありません。

現在の framework 用語は `isShell()` です。

AI または contributor が現在の codebase で PHP CLI execution を確認する必要がある場合は、次を使います。

```php
OP()->isShell()
```

または、一部 core code で使われている static form を使います。

```php
OP::isShell()
```

## Naming Note

`isShell()` は、実質的には現在の `is CLI runtime` check です。

この名前は historical framework terminology です。

project が compatibility または naming layer を追加すると明示的に決めない限り、新しい `isCLI()` wrapper を安易に導入してはいけません。
