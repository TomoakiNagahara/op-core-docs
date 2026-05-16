# `isShell()` Technical Details

## Scope

This document describes the current technical behavior behind `OP()->isShell()`.

## Implementation Path

`OP()->isShell()` is provided by `OP_ENV`.

The current path is:

1. `OP()->isShell()`
2. `OP_ENV::isShell()`

The implementation currently lives in:

- `asset/core/trait/OP_ENV.php`

## Decision Rule

The current rule is:

1. Compare `PHP_SAPI` with `cli`
2. If `PHP_SAPI === 'cli'`, return `true`
3. Otherwise, return `false`

In code terms, the current decision is equivalent to:

```php
return PHP_SAPI === 'cli';
```

## No Cached Result

`isShell()` does not currently store its result in a static variable.

It directly checks `PHP_SAPI` each time it is called.

This is acceptable because `PHP_SAPI` is a process-level PHP runtime value and does not change during one request lifecycle.

## Meaning In The Framework

`isShell()` is the framework's current PHP CLI detection method.

It is used to separate shell execution behavior from HTTP execution behavior.

Examples of behavior affected by `isShell()` include:

- request loading path selection
- MIME header output avoidance
- route calculation behavior
- localhost detection
- shell-safe debug or encoding behavior

## Relation To `isLocalhost()`

`OP()->isLocalhost()` checks `isShell()` first.

Because of that, shell execution is treated as localhost in the current design.

## `isCLI()`

There is no current `isCLI()` method in `OP_ENV`.

The current framework term is `isShell()`.

When AI or contributors need to check for PHP CLI execution in the current codebase, they should use:

```php
OP()->isShell()
```

or the static form used by some core code:

```php
OP::isShell()
```

## Naming Note

`isShell()` is effectively the current `is CLI runtime` check.

The name is historical framework terminology.

Do not introduce a new `isCLI()` wrapper casually unless the project explicitly decides to add that compatibility or naming layer.
