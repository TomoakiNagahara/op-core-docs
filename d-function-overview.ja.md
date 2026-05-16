# `D()` 関数の技術概要

## 概要

技術的には、`D()` は framework における debug 出力の entry function です。

ただし、すべての整形処理を自分で行うわけではありません。

少量の制御ロジックを行ったあと、実際の formatting は Dump unit に委譲します。

## 現在の技術フロー

現在の流れは次の通りです。

1. `D()` が呼ばれる
2. framework が requestor が administrator かどうかを確認する
3. administrator でなければ `D()` は即 return する
4. Dump unit がインストールされていれば `OP()->Unit()->Dump()->Mark(...)` に委譲する
5. Dump unit がなければ fallback として native `var_dump()` を使う

## 意味

つまり、`D()` は単なる formatting helper ではありません。

次の役割も持っています。

- 制御された debug access point
- admin 保護された debug 出力機構
- core 側の debug 呼び出しと Dump unit renderer を繋ぐ bridge

## 対象範囲

この概要文書は、`D()` の一般的な技術責務を説明するものです。

個別の実装は次で詳述します。

- `D()` 関数自体の仕様
- Dump unit の rendering 挙動
