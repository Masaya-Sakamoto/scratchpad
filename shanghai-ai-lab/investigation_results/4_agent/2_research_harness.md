# 4.2 ResearchHarness - 軽量Agentランタイム

## 1. ResearchHarness概要

### 基本機能
- **軽量Agentランタイム**: 汎用ツール使用LLM Agentの公平なベンチマーク評価
- **隔離されたワークスペース**: 各実行を分離
- **永続ターミナル**: トーンをまたぐ状態維持
- **完全なトレース**: JSONL形式で保存

### 主要特徴
1. **ワークスペース隔離**: 各Runを独立した環境
2. **状態管理**: 前回のセル作成した変数を次ターンで利用
3. **トレース記録**: 全操作をJSONL形式で保存
4. **コンテキスト圧縮**: 履歴を学習データとして再利用

## 2. 技術アーキテクチャ

### コンポーネント
```
ResearchHarness
├── ワークスペースマネージャ
├── ターミナルランタイム
├── ファイルシステム
├── メモリ管理
└── トレースレコーダ
```

### ランタイム環境
- **Python Kernel**: 永続的状態維持
- **Shell**: 命令行実行
- **ファイルシステム**: 隔離されたワークスペース
- **画像/PDF表示**: 結果表示機能

## 3. 導入設定

### 基本インストール
```bash
# ResearchHarnessインストール
pip install research-harness

# 設定ファイル作成
mkdir -p ~/.config/research-harness
cat > ~/.config/research-harness/config.yaml <<EOF
model:
  endpoint: http://localhost:8000/v1
  name: Agents-A1
  max_tokens: 4096

workspace:
  base_dir: /workspace/research-harness
  isolation: docker
  
trace:
  format: jsonl
  save_dir: /logs/research-harness
EOF
```

### Docker環境
```bash
# Dockerイメージビルド
docker build -t research-harness-env .

# コンテナ起動
docker run -it --rm \
  -v $(pwd):/workspace \
  -v /logs:/logs \
  research-harness-env
```

## 4. Agentワークフロー

### 基本構成
```
Agent Cycle
├── <thought>: 次の行動計画
├── <python>: Pythonコード実行
├── observation: 結果取得
└── 次のターンへ
```

### 実行パラメータ
- **Timeout**: 1 actionあたり60秒
- **Max Turns**: 最大30ターン
- **Max Tokens**: 1ターン最大4,096トークン
- **Context**: Trajectory context 24,576トークン

### 使用例
```python
from research_harness import ResearchHarness

harness = ResearchHarness()

# 実験実行
result = harness.run(
    task="CDC FluViewデータを分析",
    max_turns=10,
    timeout=60
)

# 結果取得
print(result.observations)
print(result.final_report)
```

## 5. ワークスペース管理

### 隔離構造
- **独立したディレクトリ**: 各Run用
- **永続状態**: トーンを跨いでの変数保持
- **ファイルアクセス**: ワークスペース内のみ
- **ネットワーク**: 制限された外部アクセス

### 状態管理
```python
# 変数保持
df = pd.read_csv("data.csv")
analysis = df.groupby("category").mean()

# 次ターンで再利用可能
result = analysis.plot()
```

## 6. トレース・監査

### JSONL形式
```json
{"turn": 1, "thought": "データをロードする", "python": "df = pd.read_csv(...)", "observation": "成功"}
{"turn": 2, "thought": "分析する", "python": "result = df.groupby(...)", "observation": "結果取得"}
```

### 学習データ再利用
- **履歴圧縮**: 重要な情報を抽出
- **トレーニングセット**: 教師付き微調整用
- **品質評価**: 成功/失敗のラベル付け

## 7. 外部依存

### 必須機能
- **OpenAI互換API**: モデルエンドポイント
- **Pythonライブラリ**: 科学計算用
- **ファイルシステム**: 読み書き

### 外部サービス
- **WebSearch/WebFetch**: Serper/Jina（外部Credential必要）
- **PDF処理**: MinerU（外部サービス）
- **論文検索**: 外部API

### 完全閉域対応
- **ローカル検索**: 内部インデックス
- **ローカルMinerU**: 論文解析
- **内部データベース**: 論文インデックス

## 8. 完全閉域設定

### 外部アクセス制限
```yaml
# config.yaml
security:
  allow_external_access: false
  whitelist:
    - localhost
    - 127.0.0.1
  sandbox:
    network: restricted
    filesystem: isolated
```

### ローカル代替
1. **WebSearch**: 内部検索エンジン
2. **WebFetch**: ローカルHTTPプロキシ
3. **PDF解析**: ローカルMinerU
4. **論文検索**: 内部データベース

## 9. 評価・ベンチマーク

### ResearchClawBench評価
- **タスク数**: 40件のreal-science task
- **分野**: 10分野横断
- **評価基準**: raw dataからpublication-quality report

### ベンチマーク手順
1. **タスク設定**: 問題定義
2. **Agent実行**: 自律的研究
3. **評価**: 専門家ラベルによる評価
4. **スコアリング**: 得点計算

## 10. 導入ステップ

### フェーズ1: 基本セットアップ（1週間）
1. ResearchHarnessインストール
2. 基本設定確認
3. テスト実行

### フェーズ2: 最適化（2週間）
1. 環境最適化
2. 監視システム
3. トレース分析

### フェーズ3: 完全閉域（2週間）
1. 外部アクセス制限
2. ローカル代替実装
3. セキュリティ強化

### フェーズ4: 本番環境（3週間）
1. 本番データ使用
2. 大規模評価
3. 学習データ収集
