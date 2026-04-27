# theLook eCommerce データ基盤 & 分析

アパレルECサイトのオープンデータ（theLook eCommerce）を使い、データ基盤の構築から分析までを一気通貫で実施したプロジェクトです。

## 使用技術

SQL / Python / BigQuery / Cloud Storage / Dataform / Looker Studio / Git

## データソース

- **theLook eCommerce**（BigQuery公開データセット）
- 架空のアパレルECサイトの顧客・注文・商品・Webイベント・物流データ
- 7テーブル構成（users, orders, order_items, products, inventory_items, events, distribution_centers）

## データ基盤の構成

Dataformで4層構造のデータパイプラインを構築しました。

| Step | レイヤー | ディレクトリ | 役割 |
|---------|---------|------------|------|
| 1 | Raw | `definitions/raw/` | 生データ |
| 2 | Adapter | `definitions/adapter/` | 前処理（個人情報のマスキング・タイムゾーン変換）のみ行ったテーブル |
| 3 | Util | `definitions/util/` | 利用頻度の高い組み合わせの結合など便利なデータ加工処理の結合・集計を行った中間テーブル |
| 4 | Wide | `definitions/wide/` | AdapterとUtilのすべてのテーブルをLEFT JOINで結合した分析に必要なカラムが揃った最終テーブル |
| 5 | Mart | `definitions/wide/` | BIツールなどで分析やグラフ作成ができない場合にレポート用に集計したテーブル |

### 設計意図

- **Raw → Adapter を分離**：生データに手を加えず保全することで、クレンジングにミスがあってもRawからやり直せる
- **Utilで共通処理を集約**：分析のたびに同じ加工処理を書かなくて済むように、汎用的な中間テーブルをまとめた
- **WideでAdapterとUtilをすべて結合**：AdapterとUtilのすべてのテーブルをLEFT JOINで1つのワイドテーブルにまとめ、Wideを参照するだけで必要なデータにアクセスできるようにした
- **Martで分析目的別に分離**：分析レポートと1:1で対応するテーブルを作成し、BIツールから直接参照できるようにした

## ディレクトリ構成

```
thelook-data-platform/
├── definitions/
│   ├── raw/        # 生データ取り込み
│   ├── adapter/    # 前処理（マスキング・型変換）
│   ├── util/       # 汎用的な中間テーブル
│   └── wide/       # 結合済みワイドテーブル
│   └── mart/       # 分析レポート用テーブル
├── README.md
└── workflow_settings.yaml
```

## レイヤー構成図

```
raw → adapter → util → wide → mart → BIツール（Looker Studio）
```