# `isAdmin()` の技術仕様

## 対象範囲

この文書は、`OP()->isAdmin()` の技術的な挙動を説明します。

## 実装経路

`OP()->isAdmin()` は `OP_ENV` により提供されます。

現行の経路は次です。

1. `OP()->isAdmin()`
2. `OP_ENV::isAdmin()`
3. `asset/core/include/isAdmin.php`

## 結果のキャッシュ

`OP_ENV::isAdmin()` は、計算結果を static 変数に保持します。

そのため、admin 判定は request lifecycle の中で一度だけ計算され、その後は再利用されます。

## 判定ルール

現行のルールは次です。

1. `isLocalhost()` が `true` を返したら `true`
2. そうでなければ `$_SERVER['REMOTE_ADDR']` と `OP::_ADMIN_IP_` を比較する
3. 一致したら `true`
4. それ以外は `false`

## 設定元

application 側の設定元は次です。

- `asset/config/admin.php`

関連する設定値は次です。

- `OP::_ADMIN_IP_`

また、CI bootstrap では必要に応じて admin の fallback 値が自動投入される場合があります。

## 他機能との関係

`isAdmin()` の戻り値は、例えば次の挙動に直接影響します。

- `D()`
- error notice の画面描画
- admin 専用の shutdown 出力

そのため、実装自体は小さくても、運用上はかなり重要な判定です。

## `D()` への影響

`D()` は debug output を描画する前に `isAdmin()` を確認します。

現在の `D()` gate は次です。

```php
if(!OP\OP::isAdmin() ){
	return;
}
```

つまり、`D()` output は `isAdmin()` が `true` を返す場合だけ表示されます。

予定されている `isAdmin()` からの `isLocalhost()` shortcut 削除後は、configured admin IP が現在の `$_SERVER['REMOTE_ADDR']` と明示的に一致しない限り、localhost access でも `D()` output は表示されません。

local development では、browser で `D()` output を期待する場合、通常は local admin configuration で admin IP を `127.0.0.1` として明示設定する必要があります。

## Error Notice Output への影響

framework error は core error path で収集され、その後 Notice flow に渡されます。

Notice unit は現在、保存された notice をどう扱うか判断するときに `isAdmin()` を確認します。

```php
if( OP()->isAdmin() ){
	Dump($notice);
}else{
	Mail($notice);
}
```

つまり、admin requestor は screen/debug output を受け取り、non-admin requestor は mail notification に流れます。

予定されている `isAdmin()` 変更後は、localhost access は自動的には on-screen notice dump を受け取りません。

local development で `127.0.0.1` を admin IP として明示設定していない場合、これまで画面に出ていた notice が non-admin notice として扱われ、mail notification 側に流れる可能性があります。

これは administrator access を明示的な admin IP だけに依存させることによる、想定された結果です。

## Development Configuration への影響

予定されている変更は、localhost development support を削除するものではありません。

localhost development support を、暗黙 rule から明示的な configuration choice に変えるものです。

現在の developer experience を維持したい development environment では、次のようにします。

- browser request が IPv4 localhost から届く場合は、admin IP を `127.0.0.1` として設定する
- PHP が実際に見る `$_SERVER['REMOTE_ADDR']` value を admin IP として設定する
- `isLocalhost()` が間接的に admin behavior を付与することに依存しない

production environment では、次のようにします。

- 実際の administrator IP を 1 つだけ設定する
- localhost を hidden administrator source として使わない
- この policy が機能するには、proxy や web-server topology が意味のある `$_SERVER['REMOTE_ADDR']` を保持している必要がある

## 技術的な特徴

localhost の shortcut は、設定された admin IP との比較より先に実行されます。

つまり、現行設計では localhost 判定が常に優先されます。

これは偶然ではなく、意図された設計です。

## Security Discussion

現行の localhost shortcut は開発時には便利ですが、admin 判定が明示的に設定された admin IP だけに基づかない、という性質も持ちます。

現行実装では、次の chain が成立します。

```text
isShell() === true
  -> isLocalhost() === true
  -> isAdmin() === true
```

`$_SERVER['REMOTE_ADDR']` が `127.0.0.1` または `::1` の HTTP request でも同じです。

security concern は、`isLocalhost()` 自体が複雑であるとか、header spoofing されやすいという点ではありません。

問題は、`isAdmin()` が明示的に設定された admin IP を確認する前に、localhost を暗黙の administrator source として扱っている点です。

## [DOC-PRIORITY1] 近い将来の変更

近い将来、`isAdmin()` から `isLocalhost()` への依存を削除する予定です。

この変更は、今後数週間以内に行われる想定です。

変更後の `isAdmin()` は、明示的に設定された単一の admin IP address と、現在の request の remote address の比較だけで administrator access を判定するべきです。

想定される rule は次です。

```text
$_SERVER['REMOTE_ADDR'] === OP::_ADMIN_IP_
```

この model では次のようになります。

- localhost は自動的には admin にならない
- shell execution は `isLocalhost()` 経由で自動的には admin にならない
- `127.0.0.1` は、`OP::_ADMIN_IP_` として明示設定された場合だけ admin になる
- `::1` は、`OP::_ADMIN_IP_` として明示設定された場合だけ admin になる

通常の production operation で `OP::_ADMIN_IP_` を `::1` に設定することは現実的ではありません。remote administrator が admin になれないためです。

そのため、実用上の production model は、PHP が administrator について `$_SERVER['REMOTE_ADDR']` として実際に見る address を、単一の administrator IP address として明示設定する形です。

## AI と contributor 向け guidance

`isAdmin()` を編集するときに、`isLocalhost()` を implicit admin shortcut として維持または再導入してはいけません。

shell execution を `isAdmin()` 経由で administrator access として扱ってはいけません。

`isLocalhost()` は low-level environment check として維持します。

`isAdmin()` は explicit administrator IP decision として維持します。

development 用に localhost administrator access が必要な場合は、関連する local configuration で admin IP を `127.0.0.1` として明示設定します。
