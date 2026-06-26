# Snowflake Gen1 vs Gen2 ウェアハウス TPC-H ベンチマーク

Snowflake の第1世代(Gen1)と第2世代(Gen2) X-Small ウェアハウスの性能を、TPC-H ベンチマーククエリで比較した実験結果をまとめたリポジトリです。

## 実験概要

| 項目 | 内容 |
|------|------|
| 比較対象 | Gen1 XS (STANDARD_GEN_1) vs Gen2 XS (STANDARD_GEN_2) |
| データソース | `SNOWFLAKE_SAMPLE_DATA` (TPC-H) |
| スケールファクタ | SF1 (0.2GB), SF10 (2.5GB), SF100 (25GB), SF1000 (259GB) |
| クエリ数 | 8クエリ (Q1, Q3, Q5, Q6, Q9, Q12, Q18, Q21) |
| 実行回数 | 各クエリ5回 (中央値を採用) |
| クレジット単価 | Gen1: 1.0 credit/h, Gen2: 1.35 credit/h |

## 実験条件

- 結果キャッシュ無効化: `ALTER SESSION SET USE_CACHED_RESULT = FALSE`
- Query Acceleration Service 無効化 (公平な比較のため)
- 各クエリ5回実行し中央値(Median)を計測値として採用
- ウェアハウスは各実行前にSUSPEND/RESUMEでキャッシュクリア

## 対象クエリ

| クエリ | 特性 | 参照テーブル |
|--------|------|-------------|
| Q1 | 集計・GROUP BY (単一テーブルスキャン) | LINEITEM |
| Q3 | 3テーブルJOIN + TOP-N | CUSTOMER, ORDERS, LINEITEM |
| Q5 | 6テーブルJOIN (リージョン集計) | CUSTOMER, ORDERS, LINEITEM, SUPPLIER, NATION, REGION |
| Q6 | 単純フィルタ+集計 (テーブルスキャン性能) | LINEITEM |
| Q9 | 6テーブルJOIN + LIKE (利益計算) | PART, SUPPLIER, LINEITEM, PARTSUPP, ORDERS, NATION |
| Q12 | 2テーブルJOIN + CASE集計 | ORDERS, LINEITEM |
| Q18 | サブクエリ + 3テーブルJOIN (大量注文) | CUSTOMER, ORDERS, LINEITEM |
| Q21 | EXISTS/NOT EXISTS + 4テーブルJOIN | SUPPLIER, LINEITEM, ORDERS, NATION |

## 主要結果

### 性能改善率 (Gen2 vs Gen1)

| スケール | データ量 | 平均改善率 | 最大改善 |
|----------|----------|-----------|----------|
| SF1 | 0.2GB | ~20% | Q12: 34% |
| SF10 | 2.5GB | ~25% | Q5: 40% |
| SF100 | 25GB | ~37% | Q12: 53% |
| SF1000 | 259GB | ~30% | Q12: 58% |

### 結論

1. **性能面**: Gen2はSF100(25GB)以上で平均30〜37%の実行時間短縮を達成
2. **コスト面**: クレジット単価+35%だが、実行時間短縮により大規模データではコスト同等〜やや有利
3. **推奨**: 25GB以上の分析ワークロードではGen2を推奨
4. **DMLへの期待**: 本ベンチマークはSELECTのみだが、Gen2はDML(INSERT/UPDATE/DELETE/MERGE)に対する最適化が公式に明記されており、ETL・データパイプラインではさらに大きな効果が期待される

## ファイル構成

```
.
├── README.md                                        # 本ファイル
├── tpch_benchmark_report.py                         # PDF生成スクリプト (データ・チャート・レイアウト含む)
└── TPC-H_Benchmark_Report_Gen1_vs_Gen2.pdf          # 生成済みレポート (15ページ, 日本語)
```

## PDFレポートの再生成

```bash
# 必要パッケージ
pip install fpdf2 matplotlib numpy

# 実行 (macOS: Arial Unicode フォントが /Library/Fonts/ に必要)
python tpch_benchmark_report.py
```

フォントパスは `tpch_benchmark_report.py` 内の `FONT_PATH` 変数で変更可能です。

## 実験環境

- Snowflake アカウント: SFSEAPAC (AWS ap-northeast-1)
- 実施日: 2026年6月
- データソース: `SNOWFLAKE_SAMPLE_DATA` (Snowflake提供のサンプルデータ)

## ライセンス

MIT
