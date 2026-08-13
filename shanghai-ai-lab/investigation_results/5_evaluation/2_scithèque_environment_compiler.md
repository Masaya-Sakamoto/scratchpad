# 5.2 SciThèque型実験環境コンパイラ

## 1. SciThèque概要

### 基本コンセプト
- **論文**: arXiv:2607.28990 "Scaling Scientific Discovery Environments for Turn-Level Agentic RL"
- **目的**: 仮説、データセット、隠された証拠グラフ、検証器をタスク環境にコンパイル
- **特徴**: DAGベースの軌道合成、検証器フィルタリング、ターンレベル信用付け

### 科学発見環境（SciDisco）
- **フレームワーク**: スケーラブルな科学発見エージェントの訓練
- **アプローチ**: 現実世界の科学データ上のプロセス監視環境
- **構成**: SciThèque + DiscoPO（強化学習）

## 2. 実験環境コンパイラ

### 環境コンパイルフロー
```
Scientific Dataset
        ↓
  dataset profiling
        ↓
  Hypothesis generation
        ↓
  Reference analysis
        ↓
  Hidden Evidence DAG
        ↓
      Verifier
        ↓
    Sandbox
        ↓
Agent interaction
```

### コンポーネント詳細
1. **Scientific Dataset**: 元データ（構造化/非構造化）
2. **Dataset Profiling**: データ分析・特性抽出
3. **Hypothesis Generation**: 仮説生成（自然言語）
4. **Reference Analysis**: 関連文献分析
5. **Hidden Evidence DAG**: 隠された証拠グラフ
6. **Verifier**: 検証ロジック（エージェントに見せない）
7. **Sandbox**: 隔離された実行環境
8. **Agent Interaction**: エージェントとのインタラクション

## 3. Hidden Evidence DAG

### DAG構造
- **ノード**: 証拠、仮説、計算結果
- **エッジ**: 論理的依存関係
- **隠し**: エージェントには見えないが検証可能
- **検証**: 各アクションが証拠を生成できるか

### DAG生成アルゴリズム
```python
def generate_evidence_dag(dataset, hypotheses):
    dag = {
        "nodes": [],
        "edges": []
    }
    
    # データプロファイリング
    for field in dataset.fields:
        dag["nodes"].append({
            "type": "data_point",
            "id": field.id,
            "value": field.value,
            "verified": False
        })
    
    # 仮説と証拠の接続
    for hypothesis in hypotheses:
        evidence_nodes = find_evidence_for_hypothesis(hypothesis, dataset)
        add_to_dag(dag, evidence_nodes, hypothesis)
    
    return dag
```

## 4. Verifier設計

### 検証ロジック
- **目標**: エージェントのアクションが正しい証拠を生成しているか
- **方法**: 隠された正解ロジック
- **出力**: 検証結果（pass/fail）+ 詳細フィードバック

### Verifier実装
```python
class EvidenceVerifier:
    def __init__(self, hidden_logic):
        self.hidden_logic = hidden_logic
    
    def verify(self, agent_action, evidence_dag):
        # エージェントのアクションがDAGに与える影響
        impact = self.calculate_impact(agent_action, evidence_dag)
        
        # 隠されたロジックで検証
        is_valid = self.hidden_logic.evaluate(impact)
        
        return {
            "valid": is_valid,
            "feedback": self.generate_feedback(impact),
            "evidence_progress": self.calculate_progress(impact)
        }
```

## 5. Sandbox環境

### 隔離特性
- **Python Kernel**: 永続状態維持
- **ファイルシステム**: 隔離されたワークスペース
- **ネットワーク**: 制限された外部アクセス
- **リソース**: 制限されたCPU/メモリ

### 実行パラメータ
- **Timeout**: 1 actionあたり60秒
- **Max Turns**: 最大30ターン
- **Max Tokens**: 1ターン最大4,096トークン
- **Context**: Trajectory context 24,576トークン

## 6. 環境生成

### 環境タイプ別
- **Tabular**: 1,024環境（構造化データ）
- **Time series**: 287環境（時系列データ）
- **Spatial**: 206環境（空間データ）
- **Sequence**: 114環境（シーケンスデータ）
- **Graph**: 55環境（グラフデータ）
- **合計**: 1,686環境
- **Stress Variants**: 504個

### 分野別
- **Economics**: 452環境
- **Medicine**: 334環境
- **Biology**: 325環境
- **Earth Science**: 219環境
- **Engineering**: 127環境
- **Epidemiology**: 119環境

## 7. 導入ステップ

### ステップ1: 環境コンパイラセットアップ（2週間）
1. SciThèqueリポジトリクローン
2. 依存関係インストール
3. 基本設定

### ステップ2: データセット準備（3-4週間）
1. 初期1,686環境相当のデータ収集
2. データプロファイリング
3. 仮説生成

### ステップ3: DAG・Verifier実装（3-4週間）
1. 証拠グラフ生成
2. 検証ロジック実装
3. 隠しロジック実装

### ステップ4: Sandbox環境構築（2週間）
1. 隔離環境作成
2. 実行パラメータ設定
3. テスト実行

## 8. 完全閉域対応

### ローカル環境
- **データ**: ローカルに保存
- **モデル**: ローカルLLM
- **計算**: ローカルリソース
- **ネットワーク**: 外部アクセス制限

### 監査・トレーサビリティ
- **ログ**: 全操作記録
- **不変保存**: 改ざん防止
- **監査**: 完全なトレーサビリティ

## 9. 評価と改善

### 環境品質
- **検証可能**: エージェントが証拠を生成できるか
- **再現性**: 同じ環境で同じ結果
- **多様性**: 異なるアプローチが可能なか

### 継続的改善
- **環境追加**: 新たなデータセット
- **タスク更新**: 新しい研究課題
- **評価改善**: 評価基準の refinement

## 10. トラブルシューティング

### 一般的な問題
- **DAG生成エラー**: データ構造確認
- **Verifier失敗**: 検証ロジック見直し
- **Sandbox問題**: 環境設定確認

### デバッグ
```bash
# 環境コンパイル詳細ログ
export SCITHEQUE_DEBUG=1

# 単一環境デバッグ
./debug_environment --env-id 123

# DAG可視化
./visualize_dag --env-id 123 --output dag.png
```
