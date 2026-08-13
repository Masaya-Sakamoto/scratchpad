---
aliases: [Agents-A1ローカル展開]
tags:
  - 上海AI実験室
  - エージェント
  - Agents-A1
  - LLM
---
# 4.1 Agents-A1ローカル展開

>[!info] 関連ドキュメント
> ⬆️ 親カテゴリ: [[4_agent_overview|エージェント基盤]]^
> 🔗 関連技術: [[2_research_harness|ResearchHarness]]

## 1. Agents-A1概要

### モデル特性
- **タイプ**: 35B Mixture-of-Experts（MoE）
- **コンテキスト長**: 256K（最大）
- **推論エンジン**: SGLang、vLLM
- **API**: OpenAI互換エンドポイント
- **用途**: 長期的検索、工学、科学研究、指示従順、ツール使用

### 主要機能
1. **Agentic reasoning**: 複雑な目標を分解、計画、適応
2. **Tool use**: 関数呼び出し、外部API、コード解釈
3. **Long context**: 拡張コンテキストの維持
4. **Instruction following**: 詳細な制約の追跡

## 2. 推論エンジン選定

### SGLang
- **特徴**: 高性能推論サーバー
- **推奨設定**:
```bash
python -m sglang.launch_server \
  --model-path InternScience/Agents-A1 \
  --port 8000 \
  --tp-size 1 \
  --mem-fraction-static 0.8 \
  --context-length 262144 \
  --reasoning-parser qwen3 \
  --tool-call-parser qwen3_coder
```

### vLLM
- **特徴**: 高速推論、スケーラブル
- **推奨設定**:
```bash
vllm serve InternScience/Agents-A1 \
  --port 8000 \
  --tensor-parallel-size 1 \
  --max-model-len 262144 \
  --reasoning-parser qwen3 \
  --enable-auto-tool-choice \
  --tool-call-parser qwen3_coder
```

### 比較
- **SGLang**: 安定性、詳細制御
- **vLLM**: 性能、スケーラビリティ
- **推奨**: vLLM（性能優先）、SGLang（安定性優先）

## 3. ハードウェア要件

### 最小構成（推論）
- **GPU**: NVIDIA A100 80GB × 1
- **RAM**: 64GB以上
- **CPU**: 16コア以上
- **Disk**: 100GB SSD（モデルウェイト用）

### 推奨構成（推論）
- **GPU**: NVIDIA A100 80GB × 2
- **RAM**: 128GB以上
- **CPU**: 32コア以上
- **Disk**: 200GB NVMe SSD

### 学習構成（Post-training）
- **GPU**: NVIDIA A100 80GB × 8
- **RAM**: 512GB以上
- **CPU**: 64コア以上
- **Network**: InfiniBand 100Gb/s

## 4. 導入ステップ

### ステップ1: 環境準備
```bash
# NVIDIAドライバー確認
nvidia-smi

# Dockerインストール
curl -fsSL https://get.docker.com | sh

# NVIDIA Container Toolkit
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/libnvidia-container/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

### ステップ2: モデルダウンロード
```bash
# Hugging Face CLIインストール
pip install huggingface-cli

# モデルダウンロード（約65GB）
huggingface-cli download InternScience/Agents-A1 --local-dir ./Agents-A1
```

### ステップ3: サーバー起動
```bash
# vLLMサーバー起動
vllm serve ./Agents-A1 \
  --port 8000 \
  --tensor-parallel-size 1 \
  --max-model-len 262144 \
  --dtype bfloat16

# 別ターミナルでテスト
curl http://localhost:8000/v1/models
```

## 5. OpenAI互換API

### エンドポイント
- **モデルリスト**: `GET /v1/models`
- **チャット**: `POST /v1/chat/completions`
- **embeddings**: `POST /v1/embeddings`

### クライアント使用例
```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="not-needed")

response = client.chat.completions.create(
    model="Agents-A1",
    messages=[
        {"role": "system", "content": "あなたは科学的研究の専門家です。"},
        {"role": "user", "content": "このデータセットを分析してください。"}
    ],
    max_tokens=4096,
    temperature=0.7
)
```

## 6. 最適化

### メモリ最適化
- **KVキャッシュ**: 効率的なメモリ使用
- **バッチ処理**: 複数リクエストの同時処理
- **量化**: INT8/INT4（性能 vs 精度トレードオフ）

### 性能最適化
- **バッチサイズ**: ワークロードに最適化
- **コンテキスト長**: 必要最小限に設定
- **トランケーション**: 不要な情報のカット

## 7. 監視・管理

### メトリクス収集
- **推論パフォーマンス**: 応答時間、スループット
- **リソース使用率**: GPUメモリアクセス、CPU使用率
- **エラーレート**: 失敗したリクエストの割合

### 管理ツール
- **Prometheus**: メトリクス収集
- **Grafana**: 可視化
- **Alertmanager**: 通知

## 8. トラブルシューティング

### 常见问题
- **メモリ不足**: GPUメモリの最適化
- **低速**: バッチサイズ調整
- **エラー**: ログ確認、設定見直し

### デバッグ
```bash
# vLLMログ詳細
vllm serve ./Agents-A1 --log-level DEBUG

# メモリ使用状況
nvidia-smi dmon -s pmu

# プロセス監視
ps aux | grep vllm
```

## 9. セキュリティ

### 認証
- **APIキー**: 必須（未来機能）
- **トークン**: 一時的なアクセス
- **IP制限**: 許可されたIPのみ

### 暗号化
- **通信**: TLS/SSL
- **保存**: データ暗号化
- **キー管理**: 安全な保管

## 10. 拡張計画

### フェーズ1: 基本推論（1週間）
- 単一GPU、基本設定
- 最小構成テスト

### フェーズ2: 最適化（2週間）
- 性能チューニング
- 監視システム

### フェーズ3: 高可用性（3週間）
- 複数GPU
- 負荷分散
- フェイルオーバー

### フェーズ4: 学習対応（4週間）
- Post-training準備
- 大規模GPUクラスター

