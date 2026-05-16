# `D()` Technical Overview

## Overview

At the technical level, `D()` is the framework debug-output entry function.

It does not do all formatting by itself.

Instead, it performs a small amount of control logic and then delegates formatting to the Dump unit.

## Current Technical Flow

The current flow is:

1. `D()` is called
2. the framework checks whether the requester is an administrator
3. if not admin, `D()` returns immediately
4. if the Dump unit is installed, control is delegated to `OP()->Unit()->Dump()->Mark(...)`
5. if the Dump unit is not installed, native `var_dump()` is used as a fallback

## Meaning

This means `D()` is not only a formatting helper.

It is also:

- a controlled debug access point
- an admin-protected debug output mechanism
- a bridge between core debug usage and the Dump unit renderer

## Scope

This overview document describes the general technical responsibility of `D()`.

Detailed implementation is documented separately in:

- core function documentation for `D()`
- Dump unit documentation for rendering behavior
