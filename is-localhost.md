# `isLocalhost()` Technical Details

## Scope

This document describes the current technical behavior behind `OP()->isLocalhost()`.

## Implementation Path

`OP()->isLocalhost()` is provided by `OP_ENV`.

The current path is:

1. `OP()->isLocalhost()`
2. `OP_ENV::isLocalhost()`
3. `asset/core/include/isLocalhost.php`

## Cached Result

`OP_ENV::isLocalhost()` stores the computed result in a static variable.

That means the localhost decision is calculated once per request lifecycle and then reused.

## Decision Rule

The current rule is:

1. If `isShell()` returns `true`, return `true`
2. Otherwise, read `$_SERVER['REMOTE_ADDR']`
3. If the remote address is `127.0.0.1`, return `true`
4. If the remote address is `::1`, return `true`
5. Otherwise, return `false`

## Shell Behavior

Shell execution is always treated as localhost in the current design.

This means `OP()->isLocalhost()` returns `true` when the process is running under PHP CLI.

## HTTP Behavior

For HTTP requests, the decision depends only on `$_SERVER['REMOTE_ADDR']`.

The current localhost addresses are:

- `127.0.0.1`
- `::1`

Host names such as `localhost` are not checked by this function.

Forwarded headers such as `X-Forwarded-For` are not checked by this function.

## Relation To `isAdmin()`

`OP()->isAdmin()` checks `isLocalhost()` first.

Because of that, localhost access is always treated as admin access in the current design.

## `isCLI()`

There is no current `isCLI()` method in `OP_ENV`.

The framework uses `isShell()` for PHP CLI detection.

## Technical Characteristic

The implementation intentionally keeps the localhost decision small and direct.

It is an environment check, not a host-name parser and not a proxy-aware client IP resolver.

## [DOC-FUTURE] Future Direction

The localhost-to-admin shortcut may become configurable in the future.

If that happens, `isLocalhost()` may remain a low-level environment check while admin policy moves further into configuration.
