# `D()` Usage

## Overview

`D()` is the standard debug output function for ONEPIECE Framework code.

Call it directly from application or framework code when you need administrator-visible debug output.

## Basic Call

```php
D($value);
```

## Label And Value

`D()` accepts multiple arguments.

A common pattern is to pass a short label first, followed by the value being checked.

```php
D('admin', OP()->isAdmin());
```

This is useful when the dumped value is a boolean, `null`, an empty string, or another value that benefits from a label in the debug output.

## Multiple Values

Multiple values can be dumped in one call.

```php
D($request, $result, $status);
```

The arguments are kept together as one debug mark and rendered by the Dump unit when it is installed.

## Visibility

`D()` output is visible only when the requester is considered an administrator.

For the administrator check, see `asset/core/docs/is-admin.md`.

For the technical flow behind `D()`, see `asset/core/docs/d-function-overview.md`.
