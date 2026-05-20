# MLflow Trace × Unity Catalog Storage ウォークスルー

MLflow 3 のトレースを OpenTelemetry (OTel) 形式で Unity Catalog の Delta テーブルに保存し、Databricks SQL で分析するハンズオンノートブックです。

`span` / `logs` / `metrics` の区別と、Delta テーブルと MLflow UI の関係を、実際にデータを作成・確認しながら理解することを目的としています。

## 対応する記事

Qiita: (公開後に URL を記載)

## 内容

| ファイル | 説明 |
|---|---|
| `tracing_scoring_and_tables.ipynb` | UC trace storage のセットアップから SQL 分析までを行うノートブック |

## 前提条件

- Unity Catalog 対応の Databricks ワークスペース
- ワークスペース管理者によるプレビュー機能の有効化 (`Databricks 上の OpenTelemetry`、`半構造化データの読み取り最適化のためのバリアントシュレッディング`)
- Unity Catalog でカタログ・スキーマを作成する権限
- `CAN USE` 権限を持つ Databricks SQL ウェアハウス
- MLflow Python ライブラリ 3.11 以降

## ノートブックの構成

| Part | 内容 |
|---|---|
| Part 1 | セットアップ。UC trace storage のテーブル命名ルールと、独立エクスペリメントへのバインド |
| Part 2 | `@mlflow.trace` による span の送信。トレース ID の URI 形式の扱い |
| Part 3 | `mlflow.log_feedback()` によるスコア (Assessment) の書き込み |
| Part 4 | `_trace_unified` / `_trace_metadata` ビューと基盤テーブルの中身の確認 |
| Part 5 | SQL によるトレース分析 (レイテンシ、件数推移、エラー抽出など) |

## 使い方

1. ノートブックを Databricks ワークスペースにインポートする
2. Part 1 のセル内の以下の変数を自身の環境に合わせて変更する
   - `CATALOG_NAME` / `SCHEMA_NAME` / `TABLE_PREFIX_NAME`
   - `MLFLOW_TRACING_SQL_WAREHOUSE_ID` (SQL ウェアハウス ID)
   - `EXPERIMENT_NAME`
3. 上から順にセルを実行する

## 注意事項

- UC trace storage は 2026年5月時点でパブリックプレビューです。トレース取り込みのレート制限や、個別トレースの削除が SQL 経由のみといった制限事項があります。最新の制限事項は公式ドキュメントを確認してください。
- ノートブック内のカタログ名・スキーマ名・SQL ウェアハウス ID はサンプル値です。コミット前に自身の環境の値に置き換えるか、プレースホルダに戻してください。

## 参考ドキュメント

- [OpenTelemetryトレースをUnity Catalogに保存する](https://docs.databricks.com/aws/ja/mlflow3/genai/tracing/trace-unity-catalog)
- [Unity Catalogに保存されているOpenTelemetryトレースを照会する](https://docs.databricks.com/aws/ja/mlflow3/genai/tracing/observe-with-traces/query-dbsql)
