---
aliases: [SCP自己ホスト]
tags:
  - 上海AI実験室
  - SCP
  - データ
  - エージェント
---
# 3.1 SCP（Scientific Computing Platform）自己ホスト

>[!info] 関連ドキュメント
> ⬆️ 親カテゴリ: [[3_data_overview|データレイク]]^
> 🔗 関連技術: [[2_data_lake_design|データレイク設計]]
> 🔗 関連技術: [[1_agents_a1_deployment|Agents-A1展開]]

## 1. SCPアーキテクチャ

### Hub-Spokeモデル
- **SCP Hub**: 計画・スケジューリング・権限管理
- **SCP Server**: ローカルモデル・DB・計算機・実験装置管理
- **構成**: 1 Hub + 複数Server（分散配置）

### 主要コンポーネント
```
SCP Hub
├── 計画マネージャ
├── スケジューラ
├── 権限管理（IAM）
├── 監査ログ
└── 可視化ダッシュボード

SCP Server (Data)
├── データマネージャ
├── ツールランタイム
├── モデルサーバー
└── 計算リソース

SCP Server (Instruments)
├── 装置マネージャ
├── ドライバ
├── 安全インターロック
└── 実験ワークフロー
```

## 2. 自己ホスト要件

### ハードウェア要件
- **Hub**: 
  - CPU: 16コア以上
  - メモリ: 64GB以上
  - ストレージ: 500GB SSD
- **Server (Data)**:
  - CPU: 32コア以上
  - メモリ: 128GB以上
  - GPU: NVIDIA A100 × 4（推論用）
- **Server (Instruments)**:
  - CPU: 16コア以上
  - メモリ: 64GB以上
  - I/O: 高速インターフェース（USB 3.0/PCIe）

### ソフトウェア要件
- **OS**: Linux（Ubuntu 20.04+、CentOS 8+）
- **コンテナ**: Docker 20.10+
- **Kubernetes**: 1.25+（推奨）
- **GPUドライバー**: NVIDIA 515+

## 3. 導入ステップ

### ステップ1: Hubデプロイ
```bash
# SCP Hubインストール
kubectl apply -f scp-hub/namespace.yaml
kubectl apply -f scp-hub/crds.yaml
kubectl apply -f scp-hub/deployment.yaml
kubectl apply -f scp-hub/service.yaml
```

### ステップ2: Serverデプロイ
```bash
# Data Server
kubectl apply -f scp-server-data/namespace.yaml
kubectl apply -f scp-server-data/deployment.yaml

# Instrument Server
kubectl apply -f scp-server-instrument/namespace.yaml
kubectl apply -f scp-server-instrument/deployment.yaml
```

### ステップ3: 接続設定
```yaml
# Hub-Server接続設定
apiVersion: v1
kind: Secret
metadata:
  name: scp-connection
type: Opaque
data:
  hub-url: base64-encoded-url
  server-token: base64-encoded-token
```

## 4. ツール統合

### 科学ツール（2,200+）
- **データ取得**: APIクライアント、データベースコネクタ
- **分析ツール**: SciPy、NumPy、Pandas
- **専門ツール**: RDKit（化学）、BLAST（生物学）
- **可視化**: Matplotlib、Seaborn、Plotly

### Scientific Skill（200+）
- **生命科学**: Protein DB、BLAST、構造解析
- **材料科学**: Materials Project、結晶解析
- **地球科学**: ERA5、リモートセンシング
- **化学**: 分子モデリング、 docking

### 追加ツール管理
```yaml
# 独自ツール追加
apiVersion: scp.internai/v1
kind: Tool
metadata:
  name: custom-analysis
spec:
  type: python
  image: custom-analysis:v1.0
  env:
    - name: API_KEY
      value: "your-key"
```

## 5. 権限管理

### IAM設計
- **ユーザー**: 研究者、管理者、ゲスト
- **ロール**: 
  - Admin: 全権限
  - Researcher: 実験実行権限
  - Viewer: 閲覧のみ
- **グループ**: プロジェクトベース

### 権限ポリシー
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: researcher-role
rules:
- apiGroups: ["scp.internai"]
  resources: ["experiments"]
  verbs: ["create", "read", "update", "execute"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create", "delete"]
```

## 6. 監査・トレーサビリティ

### 監査ログ
- **対象**: 全API呼び出し、リソース操作
- **形式**: JSON、構造化ログ
- **保持**: 90日以上
- **保存**: 不変ストレージ

### トレーサビリティ
- **実験ID**: 一意の識別子
- **プロベナンス**: データ・計算の起源追跡
- **再現性**: 環境・パラメータの記録

## 7. 完全閉域設計

### 外部アクセス制御
- **ホワイトリスト**: 許可されたIP/ドメイン
- **ネットワークポリシー**: 外部通信制限
- **プロキシ**: 必要に応じて

### データ分離
- **暗号化**: 保存時・転送時
- **アクセス制御**: レベル別
- **監査**: 全アクセス記録

## 8. トラブルシューティング

### 接続問題
- **診断**: `scp-cli status`
- **ログ**: `/var/log/scp/`
- **テスト**: 接続テストスクリプト

### パフォーマンス
- **監視**: メトリクス収集
- **最適化**: リソース調整
- **スケール**: サーバー追加

## 9. 推奨構成（初期）

### Hub
- **ノード数**: 1（後でHA化）
- **CPU**: 8コア
- **メモリ**: 32GB
- **ディスク**: 200GB SSD

### Data Server
- **ノード数**: 1
- **CPU**: 16コア
- **メモリ**: 64GB
- **GPU**: A100 × 4

### Instrument Server
- **ノード数**: 1
- **CPU**: 8コア
- **メモリ**: 32GB
- **I/O**: 高速インターフェース

## 10. 拡張計画

### フェーズ1（初期）: 基本機能
- Hub + Data Server
- 100-500ツール
- 10-50 Skill

### フェーズ2（中規模）: 拡張
- 複数Data Server
- Instrument Server追加
- 1,000-2,000ツール
- 100-200 Skill

### フェーズ3（大規模）: 完全構成
- 高可用性Hub
- 複数Server
- 2,200+ツール
- 200+ Skill
- Wet Lab統合

