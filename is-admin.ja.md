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

## 技術的な特徴

localhost の shortcut は、設定された admin IP との比較より先に実行されます。

つまり、現行設計では localhost 判定が常に優先されます。

これは偶然ではなく、意図された設計です。

## [DOC-FUTURE] 将来方針

将来的には、この localhost shortcut を application 設定で切り替え可能にしたい意図があります。

その変更が入る場合、現行の localhost 優先ルールは、固定された hardcoded 仕様ではなく、config で制御される policy decision に変わる可能性があります。
