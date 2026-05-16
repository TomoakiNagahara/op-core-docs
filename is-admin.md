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

## Impact On `D()`

`D()` checks `isAdmin()` before it renders debug output.

The current `D()` gate is:

```php
if(!OP\OP::isAdmin() ){
	return;
}
```

That means `D()` output is visible only when `isAdmin()` returns `true`.

After the planned removal of the `isLocalhost()` shortcut from `isAdmin()`, localhost access will no longer see `D()` output unless the configured admin IP explicitly matches the current `$_SERVER['REMOTE_ADDR']`.

For local development, that normally means the local admin configuration should explicitly set the admin IP to `127.0.0.1` when the developer expects `D()` output in the browser.

## Impact On Error Notice Output

Framework errors are collected by the core error path and later passed to the Notice flow.

The Notice unit currently checks `isAdmin()` when deciding how to handle stored notices:

```php
if( OP()->isAdmin() ){
	Dump($notice);
}else{
	Mail($notice);
}
```

That means admin requesters receive screen/debug output, while non-admin requesters are routed to mail notification.

After the planned `isAdmin()` change, localhost access will no longer automatically receive on-screen notice dumps.

If local development does not explicitly configure `127.0.0.1` as the admin IP, notices that used to appear on screen may be treated as non-admin notices and routed to mail notification instead.

This is an expected consequence of making administrator access depend only on an explicit admin IP.

## Development Configuration Impact

The planned change does not remove localhost development support.

It changes localhost development support from an implicit rule to an explicit configuration choice.

For development environments that should keep the current developer experience:

- configure the admin IP as `127.0.0.1` when browser requests arrive from IPv4 localhost
- configure the admin IP as the actual `$_SERVER['REMOTE_ADDR']` value seen by PHP
- do not rely on `isLocalhost()` to grant admin behavior indirectly

For production environments:

- configure exactly one real administrator IP
- do not rely on localhost as a hidden administrator source
- be aware that proxy or web-server topology must preserve a meaningful `$_SERVER['REMOTE_ADDR']` for this policy to work

## Technical Characteristic

The localhost shortcut is implemented before the configured admin IP comparison.

That means localhost always wins first in the current design.

This is intentional, not incidental.

## Security Discussion

The current localhost shortcut is convenient for development, but it also means the admin decision is not based only on the explicitly configured admin IP.

In the current implementation, the following chain is possible:

```text
isShell() === true
  -> isLocalhost() === true
  -> isAdmin() === true
```

The same is true for HTTP requests whose `$_SERVER['REMOTE_ADDR']` is `127.0.0.1` or `::1`.

The security concern is not that `isLocalhost()` itself is complex or header-spoofable.

The concern is that `isAdmin()` currently treats localhost as an implicit administrator source before it checks the explicitly configured admin IP.

## [DOC-PRIORITY1] Near-Future Change

The intended near-future change is to remove the `isLocalhost()` dependency from `isAdmin()`.

This change is expected within the next few weeks.

After that change, `isAdmin()` should determine administrator access by comparing exactly one explicit admin IP address with the current request's remote address.

The intended rule is:

```text
$_SERVER['REMOTE_ADDR'] === OP::_ADMIN_IP_
```

Under that model:

- localhost is not automatically admin
- shell execution is not automatically admin through `isLocalhost()`
- `127.0.0.1` is admin only when it is explicitly configured as `OP::_ADMIN_IP_`
- `::1` is admin only when it is explicitly configured as `OP::_ADMIN_IP_`

For normal production operation, configuring `OP::_ADMIN_IP_` as `::1` would be unrealistic because it would not let a remote administrator become admin.

That is why the practical production model is a single explicitly configured administrator IP address, normally the address that PHP actually sees in `$_SERVER['REMOTE_ADDR']` for the administrator.

## Guidance For AI And Contributors

Do not preserve or reintroduce `isLocalhost()` as an implicit admin shortcut when editing `isAdmin()`.

Do not treat shell execution as administrator access through `isAdmin()`.

Keep `isLocalhost()` as a low-level environment check.

Keep `isAdmin()` as an explicit administrator IP decision.

If localhost administrator access is needed for development, configure the admin IP explicitly as `127.0.0.1` in the relevant local configuration.
