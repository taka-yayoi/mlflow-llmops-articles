# MLflow × CrewAI マルチエージェント可観測性 (Databricks版)

MLflowのブログ [Seeing Inside Your Multi-Agent System with MLflow](https://mlflow.org/blog/observability-multi-agent-part-1/) のローカル例 ([公式リポジトリ](https://github.com/oleksandrabovkun/mlflow-examples/tree/main/mlflow-crewai-observability)) を、Databricksのマネージド MLflow と基盤モデルで動くように書き換えたノートブックです。

CrewAIによる逐次マルチエージェントを題材に、autologで実行を可視化し (トレース)、その上に挙動を判定するメトリクスを重ねる (3つの柱) という、可観測性の二層構成を再現します。

## 含まれるもの

- `crewai_mlflow_observability_databricks.ipynb` — セットアップ、クルー定義、トレース確認、3つのメトリクスの柱、異常系デモまでを通しで含む単一ノートブックです。

## 何をするノートブックか

「国内製造業における生成AI導入の利点と課題」を分析する、3エージェントの逐次パイプラインです。

```mermaid
flowchart LR
    P["リサーチプランナー<br/>調査計画"] -->|state_handoff| AN["アナリスト<br/>分析"]
    AN -->|state_handoff| SY["統合担当<br/>レポート"]
```

可観測性は2つの層で構成されます。トレースが「何が起きたか」を、メトリクスが「それが許容範囲だったか」を教えます。

- 土台: `mlflow.crewai.autolog()` による自動トレース。`Crew.kickoff` の下に各タスク、エージェント、LLM呼び出しがネストされます。
- 上層: 3つのメトリクスの柱 (ルーティング / 状態ハンドオフ / 運用テレメトリ) をカスタム属性として記録します。

## 元リポジトリとの違い (Databricks適応)

本ノートブックは、元のローカル例を題材に、Databricks上で再現する際の差分とハマりどころをまとめたものです。完全に動くオリジナルのコードは元リポジトリを参照してください。主な違いは次のとおりです。

- トラッキングをローカルMLflowサーバーからマネージド MLflow (`mlflow.set_tracking_uri("databricks")`) に変更しています。
- LLMをOpenAIからDatabricks基盤モデルに変更し、OpenAI互換エンドポイント (`openai/` プロバイダー) 経由で呼んでいます。
- ハンドオフ効率スコアを、空白区切りの単語重複から文字bigramの重複に変更しています (日本語は空白で単語区切りされないため)。
- 共有状態をSQLiteからノートブック内のメモリ内ストアに簡略化しています。
- 元リポジトリにあるWorld Bankの実データツールは含めていません (後述の限界を参照)。

## 実行方法

Databricksワークスペースにノートブックをインポートし、上から順に実行します。

1. 1セル目で `mlflow[databricks]`、`crewai`、`nest_asyncio` をインストールします。
2. エンドポイント名 (`databricks-llama-4-maverick` 等) を環境に合わせて変更します。
3. 正常系のランを実行し、続けて異常系のセルを実行します。

詳細は [CrewAIトレース (Databricks 日本語ドキュメント)](https://docs.databricks.com/gcp/ja/mlflow3/genai/tracing/integrations/crewai) を参照してください。本番ではトークンをハードコードせず、Databricksシークレットまたは AI Gateway を使ってください。

## 再現時のハマりどころ

同じ構成を試す人がほぼ確実に踏むため、回避策込みでノートブックに反映しています。

- **イベントループのガード**: ノートブックのカーネルが既にイベントループを動かしているため、同期 `kickoff()` が弾かれます。イベントループを持たないワーカースレッドで実行し、`contextvars.copy_context()` でトレースコンテキストを伝播させて回避します。
- **`cache_breakpoint` の混入**: CrewAIの既知の不具合 ([issue #5886](https://github.com/crewAIInc/crewAI/issues/5886)) で、基盤モデルのエンドポイントに未知のフィールドとして弾かれます。breakpointを付与する関数を無効化して回避します。
- **autologの非致命的なERROR**: 長期メモリ用スパンのパッチ1つだけがバージョン差で失敗しますが、Crew / Task / Agent / LLM のトレースは正常に効きます。メモリ機能未使用なら実害はありません。

## MLflow UIで見るポイント

- ルートの `Crew.kickoff` の下に、各タスク・エージェント・LLM呼び出しがネストされているか。
- `crew_run_with_metrics` スパンの属性に総実行時間・出力量が乗っているか。
- `state_handoff` スパンの `handoff.efficiency_score` が極端に低くないか (低い場合はコンテキストの取りこぼし)。
- 異常系で `numeric_report_validation` の `validation.ungrounded_numbers` に、注入した数値が現れるか。

## 限界

- 数値の突き合わせは簡易チェック (機械的な数値の比較) です。数字が一致しても意味の正しさは保証しません。
- 今回はソース自体がLLM生成のため、検証が見ているのは段間の一貫性であって事実性ではありません。実データを返すツール (DatabricksならUnity Catalogのテーブルや `ai_query` など) と突き合わせると、グラウンディング検証に格上げできます。
- ループ系の指標 (`routing.loop_detected`) は、スーパーバイザー型やツールを持つ構成で効きます。今回の逐次構成では構造的に発生しにくいため、デモには含めていません。

## 参考

- 元ブログ (Part 1): https://mlflow.org/blog/observability-multi-agent-part-1/
- 元リポジトリ: https://github.com/oleksandrabovkun/mlflow-examples/tree/main/mlflow-crewai-observability
- CrewAIトレース (Databricks 日本語ドキュメント): https://docs.databricks.com/gcp/ja/mlflow3/genai/tracing/integrations/crewai
