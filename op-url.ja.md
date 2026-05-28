# OP()->URL() 技術概要

## 概要

current 実装において、`OP()->URL()` は次に定義された thin wrapper です。

- `asset/core/trait/OP_ONEPIECE.php`

実際の処理は次へ委譲されます。

- `asset/core/function/ConvertURL-2.php`

## current の責務

`OP()->URL()` は、可能であれば次を

- meta path
- local full path

document root 基準の URL に変換します。

## current の挙動

### meta path 入力

例:

```php
OP()->URL('app:/foo/bar/')
```

これは常に次になるとは限りません。

```php
'/foo/bar/'
```

この短い結果が成立するのは、`app:/` と `doc:/` が実質的に同じ root の場合だけです。

より一般には、`OP()->URL('app:/foo/bar/')` は `doc:/` 基準の URL path を返します。

例えば:

```text
doc:/ = /var/www/html/
app:/ = /var/www/html/myapp/
```

なら、結果は次になります。

```php
'/myapp/foo/bar/'
```

### full path 入力

full path が application root 配下にある場合、実装はまずそれを `app:/...` 側へ正規化し、その後 URL path に変換します。

### current request 入力

入力が次の場合:

```php
'.'
```

関数は次を組み合わせて返します。

- `REQUEST_SCHEME`
- `SERVER_NAME`
- `REQUEST_URI`

つまり current request の完全 URL です。

## [DOC-GAP] `'.'` が FQDN を含む完全 URL を返すこと

current 実装では、`OP()->URL('.')` は次を含む完全 URL を返します。

- scheme
- host
- request URI

つまり FQDN レベルの情報を含みます。

これは、framework の URL abstraction level では FQDN を含めたくないという、より大きな設計上の好みとのギャップです。
実装は `HTTP_HOST` ではなく `SERVER_NAME` を使うようになっていますが、`SERVER_NAME` も application data として完全に信用してはいけません。
request header 由来ではなく server configuration 由来ですが、設定ミスや runtime environment の違いは起こり得ます。
将来も FQDN level の data を返す設計を維持する場合、その host value は明示的に validate または constrain する必要があります。

## [DOC-FUTURE] `'.'` に対する将来方針

この挙動は将来的に修正される予定です。

current の As-Is 挙動としては記録するが、長期的に意図された設計そのものとは見なさない、という扱いにします。

### query string

入力に query string が含まれる場合、query 部分は先に分離され、最後に再付与されます。

### directory の slash

解決された full path が directory の場合、必要に応じて末尾 slash を補います。

## current の制約

### asset root は拒否される

解決された full path が `RootPath('asset')` 配下の場合、この関数は notice を出し、通常の public URL target としては扱いません。

例は次にあります。

- `asset/core/ci/OP/URL.php`

### document root 配下でない path は拒否される

解決された full path が `$_SERVER['DOCUMENT_ROOT']` 配下にない場合、current の挙動では notice を出すか false を返します。

## 運用上の意味

実務上の意味は次です。

- `app:/...` が `OP()->URL()` にとって最も自然で安定した入力である
- document root から見える application path は安全に URL 化できる
- repository 内部 path は自動的に public URL にはならない

## Meta Path 内部機構との関係

`OP()->URL()` は既存の meta path machinery に依存しています。

概念的には、URL 側の wrapper として次に依存しています。

- `ConvertPath()`
- `RootPath()`

下位関数を置き換えるものではありません。

`OP()` から使える unified な public access point を提供するものです。
