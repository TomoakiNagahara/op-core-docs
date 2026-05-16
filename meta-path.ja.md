# Meta Path 技術概要

## 概要

current の ONEPIECE Framework 実装では、meta path は主に次によって扱われます。

- `OP()->Path()`
- `RootPath()`
- `ConvertPath()`
- `CompressPath()`

meta path の root label 自体は bootstrap 時に登録されます。

## current の root 登録

current bootstrap では、次で root label を登録しています。

- `asset/bootstrap/include/root.php`

current の登録には次が含まれます。

- `real`
- `op`
- `git`
- `doc`
- `app`
- `asset`
- `core`
- `unit`

ただし、これらは framework bootstrap が current で登録している root にすぎません。

仕組み自体は、user code による追加の custom label 登録も許容しています。

主な root 定数は次で定義されています。

- `asset/core/include/Define.php`

特に重要なのは次です。

- `_ROOT_DOC_`
- `_ROOT_APP_`
- `_ROOT_OP_`
- `_ROOT_GIT_`

`_ROOT_OP_` は、`_ROOT_GIT_` を置き換える新しい定数名として意図されています。

## `OP()->Path()`

`OP()->Path()` は主な public entry point です。

歴史的には、この convenience method は 2030 版で導入されました。

導入理由は主に次の 3 つです。

- 下位関数に分かれた path 処理が煩雑だった
- framework として interface を統一したかった
- path access を常に `OP()` から届くようにしたかった

current の挙動は次です。

- 文字列が `/` で始まる場合は、`CompressPath()` により full path を meta path へ変換する
- それ以外は meta path とみなし、`ConvertPath()` により full path へ変換する

つまり `OP()->Path()` は双方向です。

実務上は、これは次をまとめた wrapper 兼 unified entry point です。

- `RootPath()`
- `ConvertPath()`
- `CompressPath()`

## `OP:/` と `git:/` からの歴史的な移行

初版の framework では、`OP:/` は `op-core` directory を指していました。

2020 世代から、`op-core` は `core:/` へ移りました。

古い `OP:/` の意味は既に現役ではないため、`op:/` は framework 全体のトップディレクトリを指す意図になっています。

そのため、新しい root 定数名として `_ROOT_OP_` が追加されました。

2030 世代では、`git:/` と `op:/` の両方が互換移行期間中に利用可能である、と理解するべきです。

現在も `git:/` を使う code は大量に残っています。

意図された方向は、即時削除ではなく、`op:/` への段階的な置き換えです。

## `ConvertPath()`

`ConvertPath()` は meta path を local full path にデコードします。

この関数自体は、すでに 2020 版から存在しています。

current の特徴は次です。

- 入力を trim する
- query string があれば分離する
- `/` で始まる path を拒否する
- `..` のような parent traversal を拒否する
- `:/` より前の meta label を取り出す
- `RootPath($meta)` で root を解決する
- 残りの path を root に連結する

`OP()->Path()` 経由で呼ばれる場合は、`throw_exception = false` かつ `file_exists = false` で使われます。

つまり current の public な convenience conversion は比較的 permissive な使い方になっています。

## `CompressPath()`

`CompressPath()` は full local path を meta path に戻します。

この関数自体も、すでに 2020 版から存在しています。

current の挙動は次です。

- directory の末尾 slash を正規化する
- `RootPath()` から登録済み root 一覧を読む
- 既知 root を逆順で照合する
- `app:/...` や `git:/...` のような最初の一致を返す

また current 実装には、`real:/` の扱いや repository 基準 root への戻しも含まれています。

## 移行期間における current の root 登録

current の bootstrap root 登録は、すでに次の両方を露出しています。

- `op`
- `git`

これは 2030 世代の互換移行方針と一致しています。

つまり current の As-Is 実装では:

- `op:/` はすでに利用可能
- `git:/` も後方互換のため引き続き利用可能

## `app:/` が運用上重要な理由

この仕組みの中でも、特に実務上重要なのは `app:/` です。

`app:/` は `_ROOT_APP_` を基準に解決されるため、application が document root 配下のどこに配置されても、code 全体の path 表現を変えずに済みます。

これが、meta path が portability と deployment flexibility を高める大きな理由のひとつです。

## 歴史的な層構造

歴史的な層構造は次です。

- 2020: `RootPath()`, `ConvertPath()`, `CompressPath()` が meta path の主要 mechanics を担っていた
- 2030: その上に、unified な convenience entry point として `OP()->Path()` が追加された

つまり 2030 は、meta path という概念を新規発明したのではなく、利用しやすく整理した世代と理解できます。

## 知識の境界

通常の人間の application 利用者が覚えるべき interface は `OP()->Path()` です。

下位 helper である

- `RootPath()`
- `ConvertPath()`
- `CompressPath()`

は、実装や AI の推論には重要ですが、人間にとっての primary interface ではありません。

これらは unified entry point の背後にある内部 mechanics として扱われます。

## 補足

- current 実装は `doc:/`, `app:/`, `op:/`, `git:/` 以外の label も扱えます
- 開発者やエンドユーザーは、同じ root registration mechanism を通じて独自 label を追加できます
- ただし application level の説明としては、この 3 つが最も重要な概念例です
- current implementation detail は op-core の責務であり、framework-level の思想そのものとは分けて理解するべきです
