# LLMジャッジのメタ評価

LLM-as-a-Judge の信頼性を、人間ラベルとの一致度 (Cohen's kappa) で定量的に検証するハンズオンノートブックです。

「ジャッジを作っただけでは信用できない。メタ評価で検証して、低ければ改善する」という考え方を、実際にデータを作って動かしながら確認することを目的としています。

## 対応する記事

Qiita: (公開後に URL を記載)

## 内容

| ファイル | 説明 |
|---|---|
| `meta_evaluation_verify.ipynb` | 3 つの LLM ジャッジを人間ラベルと突き合わせ、一致率と Cohen's kappa を計算するノートブック |

## 前提条件

- 有償版の Databricks ワークスペース (基盤モデル API / ジャッジ用 LLM エンドポイントを使用するため)
- Unity Catalog 対応のワークスペース
- 以下のライブラリ
  - `databricks-agents`
  - `mlflow`
  - `scikit-learn` (Cohen's kappa の計算に使用)

## ノートブックの構成

| ステップ | 内容 |
|---|---|
| 環境とジャッジの準備 | パッケージインストール、カタログ・スキーマ作成、3 つの LLM ジャッジ (`no_hallucination` / `intent_match` / `politeness`) の定義 |
| 人間ラベルの用意 | 代表的な問い合わせと応答のペア 20 件に、人間判定の pass/fail ラベルを付与 |
| 評価の実行 | 同じデータに対してジャッジを実行し、判定値を取得 |
| メタ評価 | 人間ラベルとジャッジ判定を突き合わせ、単純一致率と Cohen's kappa、混同行列を算出 |

## 設計上のポイント

- **ジャッジ用 LLM はアプリ用と別系統を選ぶ**。同じ LLM でアプリとジャッジを動かすと Self-Enhancement Bias (自己選好バイアス) が生じやすいため、本ノートブックでは以下のように分離しています。
  - アプリ側 (応答生成): `databricks-meta-llama-3-3-70b-instruct`
  - ジャッジ側 (判定): `databricks-claude-sonnet-4-6`
- **単純一致率ではなく Cohen's kappa を使う**。単純一致率は「偶然の一致」を見抜けないため、偶然分を除いた kappa で判定基準の一致を測ります。本番運用のジャッジは kappa 0.6 以上が目安です。
- **メタ評価には人間ラベルの分散が必要**。人間ラベルが単一クラス (全件 pass など) に偏ると kappa が計算上 0 になり、機能しません。

## 注意事項

- ノートブック内のカタログ名・スキーマ名 (`takaakiyayoi_catalog.llmops_meta_eval`) やエンドポイント名はサンプル値です。実行前に自身の環境の値へ置き換えてください。
- 評価対象は前提となる別記事で設計した 3 つの LLM ジャッジです。決定論的な Python scorer は人間判定との一致が自明なため、メタ評価の対象から外しています。

## 参考

- [LLM-as-a-Judgeの設計 ― make_judge と Python scorer を使い分けて LLMアプリの品質を測る](https://qiita.com/taka_yayoi/items/45b632bba87d96395576)
- [MLflow Prompt Registryでカナリアリリース ― プロンプト改善を段階的にリリースする](https://qiita.com/taka_yayoi/items/9a6d328b61ccb21b2134)
- [Databricks Free EditionだけでLLMOpsのコアループを1周する](https://qiita.com/taka_yayoi/items/dbea1db0e651b9773d35)
