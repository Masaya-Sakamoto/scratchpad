# 調査メモ（統合版）

## 1. ResearchClawBench（arXiv:2606.07591）
**基本構成**
- タスク数：40タスク、10科学ドメイン
- 評価 harness：ResearchHarness（軽量、Web検索/ファイル/実行ツール）
- 評価方法：RADSスコア（50点=目標論文レベル）
- 実験環境：NVIDIA A100 GPU、SGLang推論エンジン使用

## 2. SciDisco（arXiv:2607.28990）
**基本構成**
- SciThèque：データ→仮説→検証サンドボックス
- 検証システム：DAGベース（隠された証拠グラフ）
- 学習：DiscoPO（ターンレベル報酬）、Slimeフレームワーク、SGLang
- 学習パラメータ：バッチサイズ16、グループサイズ8
- 実験環境：NVIDIA A100 GPU

## 3. 上海AI実験室データセンターAcme（arXiv:2403.07648）
**クラスタ構成**
- Seren: 286ノード、8GPU/ノード、128CPU/ノード、1TBメモリ/ノード、200Gb/sネットワーク
- Kalos: 302ノード、5GPU/ノード、2,048CPU/ノード、1TBメモリ/ノード、1,000Gb/sネットワーク
- 総GPU数：4,704 A100

**ワークロード特性**
- ジョブ持続時間：従来DLワークロードの2.7-12.8倍短い
- 評価ジョブ：92.9%だがGPU使用量0.8%
- 前学習ジョブ：3.2%だがGPU使用量94.0%
- GPUメモリ使用量：75%（60GB）、GPU利用率：99%
- 評価ワークロードでGPUアイドル時間が長い（モデル読み込み29.5%、前処理19.0%）
- 頻繁なジョブ失敗（特に前学習ジョブ）

**システム最適化**
- 耐障害性前学習：非同期チェックポイント、故障診断、自動復旧
- 評価のための分離スケジューリング：メトリクス計算の分離、ワークロードバランス

## 4. 推論エンジン設定（InternLM/lmdeploy, Agents-A1）
**SGLang設定**
- `--tp-size 1`（tensor parallelism）
- `--mem-fraction-static 0.8`（GPUメモリの80%）
- `--context-length 262144`（256Kコンテキスト）
- 推論エンドポイント：`http://localhost:8000/v1`

**vLLM設定**
- `--tensor-parallel-size 1`
- `--max-model-len 262144`
- 推論エンドポイント：`http://localhost:8000/v1`

**OpenAI互換API**
- `/v1/chat/completions`
- `/v1/models`
- `/v1/completions`

## 5. Kubernetes設計（既存資料）
**CNI/CSI**
- CNI: Calico（高性能）またはCilium（eBPFベース）
- CSI: Longhorn（ブロック）、MinIO（オブジェクト）、CephFS/NFS（共有ファイル）

**サービスメッシュ**
- Istio（完全閉域、mTLS有効化）

**GPUノード構成**
- CPU: 32コア以上
- メモリ: 256GB以上
- GPU: A100 × 4-8
- ネットワーク: InfiniBand 100Gb/s（学習用）

**ストレージクラス**
- 長期ブロックストレージ：Longhorn（3レプリカ）
- オブジェクトストレージ：MinIO（100TB+）
- 共有ファイルストレージ：CephFS/NFS

## 6. SCP自己ホスト（既存資料）
**Hub-Spokeモデル**
- SCP Hub: 計画・スケジューリング・権限管理
- SCP Server: ローカルモデル・DB・計算機・実験装置管理

**ハードウェア要件**
- Hub: CPU 16コア、メモリ 64GB、ディスク 500GB SSD
- Data Server: CPU 32コア、メモリ 128GB、GPU A100 × 4
- Instrument Server: CPU 16コア、メモリ 64GB、I/O 高速インターフェース

## 7. 不足情報
1. **ストレージ設計詳細** - 容量、I/O性能、配置戦略
2. **Kubernetesバージョン** - 具体的なバージョン、CNI/CSI設定
3. **運用実績メトリクス** - ベンチマーク値、パフォーマンスデータ
4. **ネットワークトポロジ** - 詳細な接続構成

