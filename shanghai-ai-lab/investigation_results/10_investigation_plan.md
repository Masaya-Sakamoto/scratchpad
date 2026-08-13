---
aliases: [調査計画]
tags:
  - 調査計画
  - 上海AI実験室
  - オンプレミス
  - research
  - phase
---
# 上海AI実験室 オンプレミス環境 調査計画

>[!info]
> ⬆️ 親ダッシュボード: [[0_index_dashboard|プロジェクトダッシュボード]]

> 上海AI実験室の研究プラットフォーム「端砚」をオンプレミスで実現するための具体的な調査計画

## 📋 目次
- [[0_replication_design|メイン設計ドキュメント]]
- [[0_summary|要約]]
- [[#調査結果フォルダ|📁 調査結果フォルダ]]

## 🎯 調査範囲
### 基盤インフラ
- [[1_kubernetes_design|Kubernetes設計]] → [[1_infrastructure_overview|基盤インフラ]]

### 基盤ソフトウェア
- [[1_cni_csi_selection|CNI/CSI選定]] → [[2_platform_overview|プラットフォーム]]

### SCP
- [[1_scp_self_hosted|SCP自己ホスト]] → [[3_data_overview|データレイク]], [[4_agent_overview|エージェント基盤]]

### データ
- [[2_data_lake_design|データレイク設計]] → [[3_data_overview|データレイク]]

### エージェント
- [[1_agents_a1_deployment|Agents-A1展開]] → [[4_agent_overview|エージェント基盤]]
- [[2_research_harness|ResearchHarness]] → [[4_agent_overview|エージェント基盤]]

### 評価
- [[1_research_clawbench|ResearchClawBench]] → [[5_evaluation_overview|評価環境]]
- [[2_scithèque_environment_compiler|SciThèque環境コンパイラ]] → [[5_evaluation_overview|評価環境]]

### セキュリティ
- [[1_security_audit_system|セキュリティ監査システム]] → [[6_security_overview|セキュリティ]]

---

## 1. 調査背景

### 背景
上海AI実験室の研究プラットフォーム「端砚」は、**算力→データ→モデル→Agent→自主実験**の5層構造で、オンプレミスでの再現が可能であることが確認されました。

### 目的
- 上海AI実験室の環境をオンプレミスで実現するための具体的な調査
- Kubernetesクラスタ上での実現可能性の検証
- 階層的な調査項目の構成と調査計画の立案

### 対象
- 上海AI実験室の公式発表・GitHub/Hugging Face・論文
- モデルではなく実験環境そのもの
- オンプレミスでの再現可能性

## 2. 調査概要

### 主要発見
1. **技術的実現可能性**: SCP、Agents-A1、ResearchClawBenchなど主要コンポーネントが利用可能
2. **データ戦略**: PB級データ全体を複製する必要はなく、中規模構造化科学データから開始
3. **セキュリティ**: 完全閉域設計が可能
4. **アプローチ**: 漸進的拡張が推奨

### 調査範囲
- 基盤インフラ（Kubernetes、ネットワーク、ストレージ）
- 基盤ソフトウェア（CNI、CSI、監視）
- SCP（Scientific Computing Platform）
- データレイク（データ管理、セキュリティ）
- エージェント基盤（Agents-A1、ResearchHarness）
- 評価環境（ResearchClawBench、SciThèque）
- セキュリティ・監査

## 3. 調査結果サマリー

### 1. 基盤インフラ
- **Kubernetes設計**: 3-7ノードの制御平面、4-8ノードのワーカー、2-4ノードのGPUノード
- **ストレージ**: Longhorn（ブロック）、MinIO（オブジェクト）、CephFS（共有）
- **ネットワーク**: Calico/Cilium、Istio、NGINX/Traefik
- **監視**: Prometheus、Grafana、ELK/Loki

### 2. 基盤ソフトウェア
- **CNI**: Calico（高性能）、Cilium（eBPF）
- **CSI**: Longhorn（分散ブロック）、MinIO（オブジェクト）
- **自動スケーリング**: Cluster Autoscaler、KEDA、VPA

### 3. SCP自己ホスト
- **アーキテクチャ**: Hub-Spokeモデル
- **要件**: Hub（16C/64GB）、Server（32C/128GB/4xGPU）
- **ツール**: 2,200+科学ツール、200+Scientific Skill
- **監査**: 完全なトレーサビリティ

### 4. データレイク
- **ストレージ**: MinIO（100TB+）
- **データソース**: UCI（678）、OpenML（475）、FRED（151）など
- **分野別**: Economics（452）、Medicine（334）、Biology（325）
- **品質評価**: Completeness、Consistency、Task Utility

### 5. Agents-A1
- **モデル**: 35B MoE、256Kコンテキスト
- **推論**: vLLM/SGLang（OpenAI互換）
- **要件**: A100×1（推論）、×8（学習）
- **機能**: Agentic reasoning、Tool use、Long context

### 6. ResearchHarness
- **機能**: 軽量Agentランタイム、隔離ワークスペース
- **パラメータ**: 60秒/ターン、30ターン最大、4096トークン
- **使用例**: CDC FluView分析

### 7. ResearchClawBench
- **タスク**: 40件、10分野
- **評価**: Pass@1、Pass@5、Rad Average
- **結果**: Claude Code平均21.5、Agents-A1 40.0

### 8. SciThèque
- **環境**: 1,686環境（Tabular 1,024、Time series 287など）
- **フロー**: Dataset→Profiling→Hypothesis→DAG→Verifier→Sandbox
- **特徴**: Hidden Evidence DAG、Verifier、Sandbox

### 9. セキュリティ
- **原則**: 完全閉域、最小権限、多層防御
- **アクセス制御**: RBAC、ACL、機密レベル分類
- **監査**: 全操作ログ、不変保存、90日保持

## 4. 調査計画（6-12ヶ月）

### フェーズ1: 基盤インフラ（2-3週間）
1. **ハードウェア調達・設置**（1週間）
   - サーバー、ストレージ、ネットワーク機器
   - GPUカード（A100 80GB）
2. **Kubernetesクラスタ構築**（2週間）
   - 制御平面（3-5ノード）
   - ワーカーノード（4-8ノード）
   - GPUノード（2-4ノード）

### フェーズ2: 基盤ソフトウェア（2週間）
1. **CNI/CSI設定**（1週間）
   - Calico/Ciliumインストール
   - Longhorn/MinIO設定
2. **監視基盤**（1週間）
   - Prometheus/Grafana
   - ログ管理（ELK/Loki）

### フェーズ3: SCP基盤（3-4週間）
1. **SCP Hubデプロイ**（1週間）
   - Hubコンポーネント
   - 権限管理
2. **SCP Serverデプロイ**（2週間）
   - Data Server
   - Instrument Server
3. **ツール統合**（1週間）
   - 科学ツール（2,200+）
   - Scientific Skill（200+）

### フェーズ4: データ基盤（3-4週間）
1. **データレイク構築**（2週間）
   - MinIO設定
   - バケット設計
2. **データ収集**（2週間）
   - 初期1,686環境相当
   - ライセンス確認

### フェーズ5: エージェント基盤（3-4週間）
1. **Agents-A1展開**（2週間）
   - vLLM/SGLang設定
   - 推論サーバー
2. **ResearchHarness**（1週間）
   - 環境設定
   - テスト実行

### フェーズ6: 評価環境（4-6週間）
1. **ResearchClawBench**（2週間）
   - 評価セットアップ
   - ベンチマーク実行
2. **SciThèque環境**（2-4週間）
   - 環境コンパイラ
   - 実験環境生成

### フェーズ7: セキュリティ（3-4週間）
1. **アクセス制御**（2週間）
   - RBAC設定
   - ネットワークポリシー
2. **監査システム**（2週間）
   - 監査ログ
   - 不変保存

## 5. 推奨アプローチ

### 1. 漸進的アプローチ
- **Pilot**: 数TB〜数十TB、最小構成
- **研究部門**: 数十〜数百TB、中規模
- **大規模**: 数百TB〜PB、完全構成

### 2. SCPを背骨に
- Hub-Spoke構成を基盤として採用
- 公式のSelf-Hosted機能を最大化
- 段階的拡張

### 3. オープンソース優先
- 公開コンポーネントを最大化活用
- 自社開発は最小限に
- 継続的なメンテナンス

### 4. セキュリティ最優先
- 完全閉域設計を最初から考慮
- 監査証跡の確保
- 継続的なセキュリティ改善

## 6. 期待される成果

### 短期的（6ヶ月）
- **基盤環境**: Kubernetesクラスタ、SCP、データレイク
- **エージェント**: Agents-A1推論、ResearchHarness
- **評価**: ResearchClawBench評価

### 中期的（9ヶ月）
- **実験環境**: SciThèque型環境1,000-2,000個
- **学習パイプライン**: trajectory収集、SFT/RL
- **セキュリティ**: 完全閉域、監査システム

### 長期的（12ヶ月）
- **科学データ**: 分野別データセット拡張
- **Agent能力**: 自律的研究能力の向上
- ** reproducibility**: 完全な再現性・監査性

## 7. リスクと対策

### 技術的リスク
1. **GPU不足**: 段階的導入、クラウド連携
2. **データ品質**: 品質評価、クリーンアップ
3. **パフォーマンス**: 最適化、スケールアウト

### 運営リスク
1. **スキルギャップ**: トレーニング、外部支援
2. **予算超過**: 段階的投資、ROI評価
3. **セキュリティ**: 継続的監査、更新

## 8. 結論

上海AI実験室の環境は、**「実行可能・検証可能な環境」**という核心概念を重視した実装が推奨されます。PB級データ全体を複製するのではなく、中規模構造化科学データから始め、SCPを背骨に据えた構成が現実的です。

主要なコンポーネントはオープンソースで利用可能であり、Kubernetesクラスタ上での実現が可能です。6-12ヶ月の計画で、段階的に拡張していくアプローチが最も効果的です。
