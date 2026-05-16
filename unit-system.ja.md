# Unit System の技術概要

## 対象範囲

この文書は、`OP()->Unit()` アクセスと unit 差し替えを支える現行の技術構造を説明します。

## 現行の層構造

現行の unit system は次の core 要素に分かれています。

- `asset/core/class/Unit.class.php`
- `asset/core/trait/OP_UNIT_MAPPER.php`
- `asset/core/interface/*.php`
- `asset/config/unit.php`

## access pattern

主な access style は 2 つあります。

- `OP()->Unit('App')`
- `OP()->Unit()->App()`

後者の style は、`OP_UNIT_MAPPER` に定義されている unit だけが使えます。

前者の style は generic な access form として残っています。

これは、typed interface-based mapper method として公式に露出していない unit を呼ぶための実務的な手段です。

## replacement model

差し替え model は次の流れで動きます。

1. `App()` のような typed mapper method が呼ばれる
2. mapper が unit mapping config を読む
3. mapping 後の unit 名を決定する
4. その unit を instantiate または再利用する
5. 戻り値は対応する interface を満たすことが期待される

## 設定元

application 側の mapping 設定元は次です。

- `asset/config/unit.php`

関連する config section は次です。

- `mapping`

## interface の役割

`asset/core/interface/` 配下の interface 群は、unit に期待される contract を定義します。

例:

- `IF_APP`
- `IF_CI`
- `IF_LAYOUT`
- `IF_WEBPACK`

これにより、呼び出し側は typed contract に依存しつつ、背後の mapped unit 名は差し替え可能になります。
