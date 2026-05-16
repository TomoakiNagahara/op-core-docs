# Unit System Technical Overview

## Scope

This document describes the current technical structure behind `OP()->Unit()` access and unit replacement.

## Current Layers

The current unit system is split across these core parts:

- `asset/core/class/Unit.class.php`
- `asset/core/trait/OP_UNIT_MAPPER.php`
- `asset/core/interface/*.php`
- `asset/config/unit.php`

## Access Pattern

There are two main access styles:

- `OP()->Unit('App')`
- `OP()->Unit()->App()`

The second style is supported only for units that are defined in `OP_UNIT_MAPPER`.

The first style remains the generic access form.

It is the practical way to call units that are not officially exposed through typed interface-based mapper methods.

## Replacement Model

The replacement model works like this:

1. a typed mapper method such as `App()` is called
2. the mapper reads unit mapping config
3. the mapped unit name is resolved
4. the resolved unit is instantiated or reused
5. the returned object is expected to satisfy the corresponding interface

## Configuration Source

The application-side mapping source is:

- `asset/config/unit.php`

The relevant config section is:

- `mapping`

## Interface Role

The interfaces under `asset/core/interface/` define the expected contract for units.

Examples:

- `IF_APP`
- `IF_CI`
- `IF_LAYOUT`
- `IF_WEBPACK`

This allows callers to rely on a typed contract while the mapped unit name may vary.
