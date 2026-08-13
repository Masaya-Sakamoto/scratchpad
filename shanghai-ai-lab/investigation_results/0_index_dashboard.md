---
aliases: [プロジェクトダッシュボード]
tags:
  - 上海AI実験室
  - オンプレミス
  - 科学Agent
  - design
  - research
  - phase1
  - phase2
  - phase3
  - phase4
  - phase5
  - phase6
  - phase7
---
# 上海AI実験室オンプレミス再構成プロジェクト
> Shanghai AI Labの実験環境をオンプレミスで再構成するための包括的なドキュメント集

## 🚀 快速アクセス

### 主要ドキュメント
- [[0_replication_design|メイン設計ドキュメント]] - 全体像とコンセプト
- [[10_investigation_plan|調査計画]] - 6-12ヶ月の実装計画
- [[0_summary|要約]] - 主要発見のサマリー

### 調査結果フォルダ
- [[1_infrastructure_overview|🖥️ 基盤インフラ]] - Kubernetes, ストレージ, ネットワーク
- [[2_platform_overview|⚙️ プラットフォーム]] - CNI, CSI, 監視
- [[3_data_overview|💾 データレイク]] - データ管理, セキュリティ
- [[4_agent_overview|🤖 エージェント]] - Agents-A1, ResearchHarness
- [[5_evaluation_overview|📊 評価環境]] - ResearchClawBench, SciThèque
- [[6_security_overview|🔒 セキュリティ]] - アクセス制御, 監査

## 🏷️ 調査項目別リンク

### インフラストラクチャ
- [[1_kubernetes_design|Kubernetes設計]]
- [[1_cni_csi_selection|CNI/CSI選定]]

### プラットフォーム
- [[1_scp_self_hosted|SCP自己ホスト]]

### データ
- [[2_data_lake_design|データレイク設計]]

### エージェント
- [[1_agents_a1_deployment|Agents-A1展開]]
- [[2_research_harness|ResearchHarness]]

### 評価
- [[1_research_clawbench|ResearchClawBench]]
- [[2_scithèque_environment_compiler|SciThèque環境コンパイラ]]

### セキュリティ
- [[1_security_audit_system|セキュリティ監査システム]]

## 📈 進捗状況

### フェーズ1: 基盤インフラ (2-3週間)
- [ ] ハードウェア調達・設置
- [ ] Kubernetesクラスタ構築

### フェーズ2: 基盤ソフトウェア (2週間)
- [ ] CNI/CSI設定
- [ ] 監視基盤

### フェーズ3: SCP基盤 (3-4週間)
- [ ] SCP Hubデプロイ
- [ ] SCP Serverデプロイ
- [ ] ツール統合

### フェーズ4: データ基盤 (3-4週間)
- [ ] データレイク構築
- [ ] データ収集

### フェーズ5: エージェント基盤 (3-4週間)
- [ ] Agents-A1展開
- [ ] ResearchHarness

### フェーズ6: 評価環境 (4-6週間)
- [ ] ResearchClawBench
- [ ] SciThèque環境

### フェーズ7: セキュリティ (3-4週間)
- [ ] アクセス制御
- [ ] 監査システム
