# Meta Path Technical Overview

## Overview

In current ONEPIECE Framework implementation, meta paths are handled mainly through:

- `OP()->Path()`
- `RootPath()`
- `ConvertPath()`
- `CompressPath()`

The root labels themselves are registered during bootstrap.

## Current Root Registration

The current bootstrap registers root labels in:

- `asset/bootstrap/include/root.php`

Current registrations include:

- `real`
- `op`
- `git`
- `doc`
- `app`
- `asset`
- `core`
- `unit`

These are only the roots currently registered by the framework bootstrap.

The system itself also allows additional custom labels to be registered by user code.

The main root constants are defined in:

- `asset/core/include/Define.php`

Notably:

- `_ROOT_DOC_`
- `_ROOT_APP_`
- `_ROOT_OP_`
- `_ROOT_GIT_`

`_ROOT_OP_` is the newer constant name intended to replace `_ROOT_GIT_`.

## `OP()->Path()`

`OP()->Path()` is the main public entry point.

Historically, this convenience method was introduced in the 2030 generation.

Its introduction has three main reasons:

- path handling was too cumbersome when split across lower-level functions
- the framework wanted a unified interface
- path access should always be reachable from `OP()`

Current behavior:

- if the given string starts with `/`, it converts a full path into a meta path through `CompressPath()`
- otherwise, it treats the string as a meta path and converts it into a full path through `ConvertPath()`

This means `OP()->Path()` is bi-directional.

In practical terms, it is a wrapper and unified entry point over:

- `RootPath()`
- `ConvertPath()`
- `CompressPath()`

## Historical Shift from `OP:/` and `git:/`

In the first generation of the framework, `OP:/` referred to the `op-core` directory.

From the 2020 generation, `op-core` shifted to `core:/`.

Now that the old meaning of `OP:/` is no longer the active one, `op:/` is intended to represent the top directory of the framework.

That is why `_ROOT_OP_` was added as the newer root constant name.

In the 2030 generation, both `git:/` and `op:/` should be understood as available during the compatibility transition period.

There is still a large amount of code that uses `git:/`.

The intended direction is gradual replacement toward `op:/`, not immediate removal.

## `ConvertPath()`

`ConvertPath()` decodes a meta path into a local full path.

This function already existed in the 2020 generation.

Current characteristics:

- trims the input
- separates query string when present
- rejects paths that start with `/`
- rejects parent traversal such as `..`
- extracts the meta label before `:/`
- resolves the label through `RootPath($meta)`
- appends the remainder to the registered root

When called through `OP()->Path()`, it is used with `throw_exception = false` and `file_exists = false`.

That means the public convenience path conversion is permissive in current usage.

## `CompressPath()`

`CompressPath()` converts a full local path back into a meta path.

This function also already existed in the 2020 generation.

Current behavior:

- normalizes directory trailing slash
- reads the registered root list from `RootPath()`
- checks the path against known roots in reverse order
- returns the first matching meta expression such as `app:/...` or `git:/...`

It also contains current handling for `real:/` and conversion back toward repository-based roots.

## Current Root Registration During the Transition

The current bootstrap root registration already exposes both:

- `op`
- `git`

This matches the compatibility transition policy of the 2030 generation.

So, in the current As-Is implementation:

- `op:/` is already usable
- `git:/` is still usable for backward compatibility

## Why `app:/` Is Operationally Important

One of the most practical parts of the system is `app:/`.

Because `app:/` is resolved from `_ROOT_APP_`, the application can be deployed in different locations under the document root without changing path expressions throughout the codebase.

That is one of the reasons meta paths improve portability and deployment flexibility.

## Historical Layering

The historical layering is:

- 2020: `RootPath()`, `ConvertPath()`, and `CompressPath()` already provide the main meta path mechanics
- 2030: `OP()->Path()` is added as a unified convenience entry point over those mechanics

So the 2030 design simplifies usage, but it does not invent the meta path concept from scratch.

## Knowledge Boundary

For normal human application usage, the expected interface is `OP()->Path()`.

The lower-level helpers:

- `RootPath()`
- `ConvertPath()`
- `CompressPath()`

still matter for implementation and AI reasoning, but they are internal mechanics rather than the primary interface humans are expected to learn.

## Notes

- The current implementation supports more labels than only `doc:/`, `app:/`, `op:/`, and `git:/`
- Developers and end users may register their own labels through the same root registration mechanism
- Those three are still the most important conceptual examples for most application-level usage
- Current implementation details belong to op-core and should not be confused with higher-level framework philosophy
