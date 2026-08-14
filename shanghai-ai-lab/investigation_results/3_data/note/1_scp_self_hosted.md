# SCP自己ホスト (3_scp_self_hosted.md)

## 出典元
- 資料: `/home/user/doc/scratchpad/shanghai-ai-lab/investigation_results/3_data/1_scp_self_hosted.md`

## SCPアーキテクチャ
- **モデル**: Hub-Spoke（1 Hub + 複数Server）
- **SCP Hub**: 計画・スケジューリング・権限管理
- **SCP Server**: ローカルモデル・DB・計算機・実験装置管理

## ハードウェア要件
- **Hub**: CPU 16コア以上、メモリ 64GB以上、ディスク 500GB SSD
- **Data Server**: CPU 32コア以上、メモリ 128GB以上、GPU A100 × 4
- **Instrument Server**: CPU 16コア以上、メモリ 64GB以上、I/O 高速インターフェース

## ソフトウェア要件
- **OS**: Linux（Ubuntu 20.04+、CentOS 8+）
- **コンテナ**: Docker 20.10+
- **Kubernetes**: 1.25+（推奨）
- **GPUドライバー**: NVIDIA 515+

## ツール統合
- **科学ツール**: 2,200+（データ取得、分析ツール、専門ツール、可視化）
- **Scientific Skill**: 200+（生命科学、材料科学、地球科学、化学）

## 権限管理
- **ユーザー**: 研究者、管理者、ゲスト
- **ロール**: Admin（全権限）、Researcher（実験実行権限）、Viewer（閲覧のみ）
- **グループ**: プロジェクトベース

## 監査・トレーサビリティ
- **監査ログ**: 全API呼び出し、JSON形式、90日以上保持
- **トレーサビリティ**: 実験ID、プロベナンス、再現性記録

## 完全閉域設計
- **外部アクセス制御**: ホワイトリスト、ネットワークポリシー
- **データ分離**: 暗号化、アクセス制御、監査

## 推奨構成（初期）
- **Hub**: 1ノード（8コア/32GB/200GB SSD）
- **Data Server**: 1ノード（16コア/64GB/A100×4）
- **Instrument Server**: 1ノード（8コア/32GB）

## 拡張計画
- **フェーズ1**: Hub + Data Server、100-500ツール、10-50 Skill
- **フェーズ2**: 複数Data Server、Instrument Server追加、1,000-2,000ツール、100-200 Skill
- **フェーズ3**: 高可用性Hub、複数Server、2,200+ツール、200+ Skill、Wet Lab統合

## 不足情報
- データソース詳細（具体的なデータソース、ライセンス）
- 品質評価（データの品質評価基準）
- 運用実績メトリクス（データ転送量、パフォーマンス）
- ネットワークトポロジ詳細（データ転送のネットワーク構成）
