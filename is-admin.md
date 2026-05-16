# `isAdmin()` Technical Details

## Scope

This document describes the technical behavior behind `OP()->isAdmin()`.

## Implementation Path

`OP()->isAdmin()` is provided by `OP_ENV`.

The current path is:

1. `OP()->isAdmin()`
2. `OP_ENV::isAdmin()`
3. `asset/core/include/isAdmin.php`

## Cached Result

`OP_ENV::isAdmin()` stores the computed result in a static variable.

That means the admin decision is calculated once per request lifecycle and then reused.

## Decision Rule

The current rule is:

1. If `isLocalhost()` returns `true`, return `true`
2. Otherwise, compare `$_SERVER['REMOTE_ADDR']` with `OP::_ADMIN_IP_`
3. If they match, return `true`
4. Otherwise, return `false`

## Configuration Source

The application-side configuration source is:

- `asset/config/admin.php`

The relevant setting is:

- `OP::_ADMIN_IP_`

In CI bootstrap, fallback admin values may also be injected automatically when needed.

## Relation to Other Features

The return value of `isAdmin()` directly affects behavior such as:

- `D()`
- error notice screen rendering
- admin-only shutdown output

This is why the admin decision is operationally important even though the implementation itself is small.

## Technical Characteristic

The localhost shortcut is implemented before the configured admin IP comparison.

That means localhost always wins first in the current design.

This is intentional, not incidental.

## [DOC-FUTURE] Future Direction

There is an intention to make this localhost shortcut configurable by application settings in the future.

If that change is introduced, the current fixed-first localhost rule may become a policy decision controlled by configuration rather than a permanently hardcoded rule.
