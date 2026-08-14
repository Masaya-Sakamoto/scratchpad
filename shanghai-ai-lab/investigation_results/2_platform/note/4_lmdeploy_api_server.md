# InternLM/lmdeploy API Server Documentation

## 概要
- LMDeploy: LLMの圧縮、デプロイ、サービングのためのツールキット
- OpenAI互換APIサーバーを提供

## 推論エンジン設定
- SGLang: `--tp-size 1`（テンソル並列化）、`--mem-fraction-static 0.8`（GPUメモリの80%）、`--context-length 262144`（256Kコンテキスト）
- vLLM: `--tensor-parallel-size 1`、`--max-model-len 262144`
- 推論エンドポイント: `http://localhost:8000/v1`

## OpenAI互換API
- `/v1/chat/completions`
- `/v1/models`
- `/v1/completions`

## 不足情報
- Kubernetes設定詳細（バージョン、CNI/CSI）
- ストレージ設計詳細（容量、I/O性能、配置戦略）
- 運用実績メトリクス（ベンチマーク値）
- ネットワークトポロジ詳細
