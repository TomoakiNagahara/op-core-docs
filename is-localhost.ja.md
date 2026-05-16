# `isLocalhost()` の技術仕様

## 範囲

この document は、`OP()->isLocalhost()` の現行 technical behavior を説明します。

## 実装経路

`OP()->isLocalhost()` は `OP_ENV` によって提供されます。

現在の path は次の通りです。

1. `OP()->isLocalhost()`
2. `OP_ENV::isLocalhost()`
3. `asset/core/include/isLocalhost.php`

## Cached Result

`OP_ENV::isLocalhost()` は計算結果を static variable に保存します。

そのため、localhost 判定は request lifecycle ごとに 1 回だけ計算され、その後は再利用されます。

## 判定ルール

現在の rule は次の通りです。

1. `isShell()` が `true` を返したら `true` を返す
2. それ以外では `$_SERVER['REMOTE_ADDR']` を読む
3. remote address が `127.0.0.1` なら `true` を返す
4. remote address が `::1` なら `true` を返す
5. それ以外は `false` を返す

## Shell Behavior

現在の設計では、shell execution は常に localhost として扱われます。

つまり、process が PHP CLI 上で動作している場合、`OP()->isLocalhost()` は `true` を返します。

## HTTP Behavior

HTTP request では、判定は `$_SERVER['REMOTE_ADDR']` だけに依存します。

現在の localhost address は次の通りです。

- `127.0.0.1`
- `::1`

`localhost` のような host name は、この function では確認しません。

`X-Forwarded-For` のような forwarded header も、この function では確認しません。

## `isAdmin()` との関係

`OP()->isAdmin()` は最初に `isLocalhost()` を確認します。

そのため、現在の設計では localhost access は常に admin access として扱われます。

## `isCLI()`

現在の `OP_ENV` には `isCLI()` method はありません。

framework は PHP CLI detection に `isShell()` を使います。

## 技術的特徴

この implementation は、localhost 判定を意図的に小さく直接的に保っています。

これは environment check であり、host-name parser でも proxy-aware client IP resolver でもありません。

## [DOC-FUTURE] Future Direction

localhost を admin 扱いにする shortcut は、将来 configurable になる可能性があります。

その場合でも、`isLocalhost()` は low-level environment check として残り、admin policy はさらに configuration 側へ移る可能性があります。
