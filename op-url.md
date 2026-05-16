# OP()->URL() Technical Overview

## Overview

In current implementation, `OP()->URL()` is a thin wrapper defined in:

- `asset/core/trait/OP_ONEPIECE.php`

It delegates to:

- `asset/core/function/ConvertURL-2.php`

## Current Responsibilities

`OP()->URL()` converts:

- a meta path
- or a local full path

into a URL relative to the document root, when possible.

## Current Behavior

### Meta path input

Example:

```php
OP()->URL('app:/foo/bar/')
```

This does not always mean:

```php
'/foo/bar/'
```

That shorter result is only true when `app:/` and `doc:/` are effectively the same root.

More generally, `OP()->URL('app:/foo/bar/')` returns the URL path relative to `doc:/`.

For example, if:

```text
doc:/ = /var/www/html/
app:/ = /var/www/html/myapp/
```

then the result becomes:

```php
'/myapp/foo/bar/'
```

### Full path input

If a full path is under the application root, the implementation first normalizes it toward `app:/...` and then converts it into a URL path.

### Current request input

If the input is:

```php
'.'
```

the function returns:

- `REQUEST_SCHEME`
- `HTTP_HOST`
- `REQUEST_URI`

combined as the current full request URL.

## [DOC-GAP] `'.'` Returns a Full URL with FQDN

In current implementation, `OP()->URL('.')` returns a full URL including:

- scheme
- host
- request URI

That means it includes FQDN-level information.

This is a gap from the broader framework preference not to include FQDN at the framework URL abstraction level.

## [DOC-FUTURE] Future Direction for `'.'`

This behavior is expected to be corrected in the future.

The current As-Is behavior should be documented as an implementation detail, not as the long-term intended design.

### Query string

If the input includes a query string, the query part is separated first and reattached later.

### Directory slash

If the resolved full path is a directory, the implementation appends a trailing slash when needed.

## Current Restrictions

### Asset root is rejected

If the resolved full path is under `RootPath('asset')`, the function emits a notice and does not treat it as a normal public URL target.

Examples are covered in:

- `asset/core/ci/OP/URL.php`

### Non-document-root paths are rejected

If the resolved full path is not under `$_SERVER['DOCUMENT_ROOT']`, the function emits a notice or returns false in current behavior.

## Operational Meaning

The practical meaning is:

- `app:/...` is the most natural and stable input for `OP()->URL()`
- document-root-visible application paths can be URL-converted safely
- repository-internal paths are not automatically public URLs

## Relation to Meta Path Internals

`OP()->URL()` depends on the existing meta path machinery.

It is conceptually a URL-side wrapper over:

- `ConvertPath()`
- `RootPath()`

It does not replace the lower-level functions.

It provides a unified public access point from `OP()`.
