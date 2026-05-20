# mlflow-llmops-articles

MLflow を中心とした LLMOps 関連の Qiita 記事で使用した、検証用ノートブックをまとめたリポジトリです。各記事に対応するフォルダに、記事中のコードを通しで実行できるノートブックを置いています。

## 記事とノートブック

| フォルダ | テーマ | 対応記事 |
|---|---|---|
| [`meta-evaluation`](./meta-evaluation) | LLMジャッジのメタ評価。人間ラベルとの一致度 (Cohen's kappa) でジャッジの信頼性を定量化する | (Qiita 記事 URL を記載) |
| [`otel-uc-trace-storage`](./otel-uc-trace-storage) | MLflow トレースを OpenTelemetry 形式で Unity Catalog に保存し、Databricks SQL で分析する | (Qiita 記事 URL を記載) |

各フォルダの内容・前提条件・実行手順は、それぞれの `README.md` を参照してください。

## 使い方

1. 対象のノートブック (`.ipynb`) を Databricks ワークスペースにインポートする
2. ノートブック冒頭のカタログ名・スキーマ名・SQL ウェアハウス ID などの変数を、自身の環境に合わせて変更する
3. 上から順にセルを実行する

ノートブック内のカタログ名・スキーマ名・エンドポイント名などはサンプル値です。実行前に自身の環境の値へ置き換えてください。

## 関連書籍

これらの記事で扱うトピックは、共著で執筆した『MLflowで実践するLLMOps──生成AIアプリケーションの実験管理と品質保証』(技術評論社・エンジニア選書、2026年4月発売) で体系的に解説しています。MLflow 3 の 4 つのコアコンポーネント (Tracing / Evaluate & Monitor / Prompt Registry / AI Gateway) を軸に、LLMアプリケーションを「運用し続ける」ための方法をまとめた一冊です。
