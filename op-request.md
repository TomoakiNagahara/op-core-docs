# `OP()->Request()` Technical Overview

## Scope

This document describes the current technical behavior behind `OP()->Request()`.

## Entry Point

The public accessor is implemented in:

- `asset/core/trait/OP_ENV.php`

The method:

- caches the parsed request in a static variable
- loads `asset/core/include/Request.php` only once
- returns either the whole request array or one keyed value

## Current Resolution Flow

The current flow is:

1. `OP()->Request()` is called
2. `OP_ENV::Request()` loads `asset/core/include/Request.php`
3. `Request.php` initializes `$_request = []`
4. if `$_SERVER['SHELL']` exists, it loads `RequestShell.php`
4. if `OP::isShell()` is true, it loads `RequestShell.php`
5. otherwise, it loads `RequestWeb.php`
6. the final result is passed through `Encode()`
7. the encoded array is cached and reused

## Web Request Resolution

`asset/core/include/RequestWeb.php` currently does this:

1. if `CONTENT_TYPE` starts with `application/json`
2. read `php://input`
3. `json_decode(..., true)`

Otherwise:

- if `REQUEST_METHOD === 'POST'`, copy `$_POST`
- else, copy `$_GET`

So current web resolution is not a merge of GET and POST at the same time.

It selects:

- JSON
  or
- POST
  or
- GET

depending on the current request conditions.

## CLI Request Resolution

`asset/core/include/RequestShell.php` iterates over `$_SERVER['argv']`.

Arguments that contain `=` are split into:

- key
- value

and stored in the request array.

Arguments without `=` are ignored by the current implementation.

## Final Encoding

After request collection, `asset/core/include/Request.php` passes the result through:

- `Encode($_request)`

So the returned request values are the encoded result, not the raw collected array.

## Design Intention of This Encoding

The current design intentionally prefers returning encoded values over returning fully raw request values.

The practical reason is that humans often forget output escaping.

So, from the framework's point of view, returning an already encoded value is considered safer than returning a fully raw value by default.

This is a trade-off, not a claim that the result is universally safe in every context.

## Current Access Semantics

- `OP()->Request()`
  returns the whole parsed request array
- `OP()->Request('key')`
  returns one value or `null`

## Current Limits

Important current As-Is details:

- JSON detection accepts values that start with `application/json`
- web GET and POST are not merged together in one request array
- CLI parsing expects `key=value`
- request values are cached after first load

## [DOC-NOTE] Design Interpretation

This separation is also consistent with the framework preference for explicit behavior.

GET and POST are technically both request inputs, but they are not treated as one silently merged source in the current implementation.

That keeps source ambiguity lower in the default path.

## [DOC-FUTURE] Non-Goal

At the current design stage, dedicated `PUT`, `PATCH`, and `DELETE` request parsing is not planned for `OP()->Request()`.
