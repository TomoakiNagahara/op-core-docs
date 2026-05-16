# OP の技術概要

## 概要

技術的には、`OP()` は `\OP\OP` の singleton 的な instance を返します。

この instance が framework への access hub として機能します。

## 主な構造

技術構造は次の通りです。

1. global function `OP()` が呼ばれる
2. その結果、生成済みの `\OP\OP` object が返される
3. `\OP\OP` class は trait を通じて framework の各機能を公開する

## これにより得られるもの

この構造により、framework は次を実現しています。

- global な access point
- 再利用される生成済み object
- 多数の framework 機能を公開する facade 的 class
- `\OP\OP` 内部の trait ベース構成

## 対象範囲

この概要文書は、次の関係を説明するものです。

- `OP()` 関数
- `\OP\OP` class
- その class が利用する trait 群

個別の挙動は、関数・class・trait の各文書で詳述します。
