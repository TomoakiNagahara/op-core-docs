# `OP()->Request()` の技術概要

## 対象範囲

この文書は、`OP()->Request()` の current 技術挙動を説明します。

## entry point

公開 accessor は次に実装されています。

- `asset/core/trait/OP_ENV.php`

この method は次を行います。

- static 変数に parse 済み request を cache する
- `asset/core/include/Request.php` を一度だけ読む
- request 配列全体、または指定 key の値を返す

## current の解決フロー

current の flow は次です。

1. `OP()->Request()` が呼ばれる
2. `OP_ENV::Request()` が `asset/core/include/Request.php` を読む
3. `Request.php` が `$_request = []` を初期化する
4. `OP::isShell()` が true なら `RequestShell.php` を読む
5. そうでなければ `RequestWeb.php` を読む
6. 最終結果を `Encode()` に通す
7. encode 済み配列を cache して再利用する

## web request の解決

`asset/core/include/RequestWeb.php` は current 実装で次を行います。

1. `CONTENT_TYPE` が `application/json` で始まるなら
2. `php://input` を読む
3. `json_decode(..., true)` する

それ以外では次です。

- `REQUEST_METHOD === 'POST'` なら `$_POST` をコピーする
- そうでなければ `$_GET` をコピーする

したがって current の web 解決は、GET と POST を同時に merge する方式ではありません。

次のいずれかを選びます。

- JSON
  または
- POST
  または
- GET

## CLI request の解決

`asset/core/include/RequestShell.php` は `$_SERVER['argv']` を走査します。

`=` を含む引数は次に分割されます。

- key
- value

そして request 配列に保存されます。

`=` を含まない引数は current 実装では無視されます。

## 最終 encoding

request 値を集めた後、`asset/core/include/Request.php` は結果を次に通します。

- `Encode($_request)`

つまり返される request 値は、生の収集結果ではなく encode 済み結果です。

## この encoding の設計意図

current の設計では、完全な生値を返すより、encode 済みの値を返す方を意図的に選んでいます。

実務上の理由は、人間が出力時の escape を忘れやすいからです。

そのため framework としては、default で完全な生値を返すより、あらかじめ encode 済みの値を返す方が安全だと判断しています。

ただし、これはあらゆる文脈で universally safe だという意味ではなく、明確な trade-off です。

## current の access semantics

- `OP()->Request()`
  parse 済み request 配列全体を返す
- `OP()->Request('key')`
  1つの値、または `null` を返す

## current の制限

重要な current As-Is は次です。

- JSON 判定は `application/json` で始まる値を受ける
- web の GET と POST は同時 merge されない
- CLI parsing は `key=value` を前提とする
- request 値は初回読込後に cache される

## [DOC-NOTE] 設計上の解釈

この分離は、明示的な挙動を好む framework の思想とも整合しています。

GET と POST はどちらも request input ですが、current 実装ではそれらを暗黙に一つへ merge した source としては扱っていません。

その方が default path の source ambiguity を低く保てます。

## [DOC-FUTURE] 非目標

current の設計段階では、`OP()->Request()` に `PUT`, `PATCH`, `DELETE` 専用の request parse を追加する予定はありません。
