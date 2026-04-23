# AWS SAP-C02 学習カリキュラム

## 学習方針
- **順番**: 座学 → ハンズオン → 問題演習 → 間違い復習
- **ベース**: SAA取得済み＋実務2年
- **目標期間**: 5-8週間（1日1.5-3時間）
- **問題総数**: 420問（6回分の模擬試験）

## AWS公式 出題比率

| Domain | 内容 | 比率 |
|---|---|---|
| Domain 1 | 複雑な組織向けソリューション設計 | 26% |
| Domain 2 | 新ソリューション設計 | 29% |
| Domain 3 | 既存ソリューションの継続的改善 | 25% |
| Domain 4 | 移行とモダナイゼーション | **20%** |

---

## Phase 1: ガバナンス・セキュリティ基盤（Week 1）

### テーマ1: Organizations + SCP + OU設計 🔴 3-4日
**座学ポイント:**
- SCPの仕組み（継承・評価順序・ルートユーザーへの影響）
- OU設計パターン（環境別・例外OU・セキュリティ集約アカウント）
- SCPユースケース（リージョン制限・暗号化強制・インスタンスタイプ制限）
- SCP vs IAM vs Permissions Boundary の違い

**ハンズオン:**
- Organizations作成 → OU作成 → テストアカウント追加
- SCP作成（特定リージョン以外拒否）→ OUにアタッチ
- SCPの継承動作を確認

**問題演習:** ~25問
- exam-01: Q11, Q19, Q28
- exam-02: Q37
- exam-03: Q7, Q14, Q26, Q55
- exam-04: Q1, Q17, Q20, Q28, Q57, Q63, Q64
- exam-05: Q1, Q4, Q9, Q29, Q31, Q38, Q40, Q41, Q61
- exam-06: Q1, Q9, Q31, Q41, Q61, Q71

---

### テーマ2: IAM高度設計 🟡 2日
**座学ポイント:**
- 権限境界（Permissions Boundary）の仕組みと用途
- AssumeRole + 信頼ポリシー（クロスアカウント）
- STSセッションタグによるアクセス制御
- aws:PrincipalOrgID条件キー
- 外部ID（External ID）によるリスク軽減

**ハンズオン:** なし（座学+問題で十分）

**問題演習:** ~15問
- exam-01: Q24
- exam-02: Q22, Q25
- exam-03: Q36, Q39
- exam-04: Q21, Q63
- exam-05: Q3, Q21, Q40, Q73
- exam-06: Q3, Q40, Q75

---

### テーマ3: セキュリティ監査 🟡 2日
**座学ポイント:**
- CloudTrail整合性検証（ログ改ざん検出）
- CloudTrail詳細イベントセレクタ（Object Lambda等）
- IAM Access Analyzer（外部共有可視化、最小権限推奨）
- Security Hub（CISベンチマーク、Organizations統合）
- Amazon Macie（S3機密データ検出）
- Amazon Inspector（脆弱性スキャン）
- AWS Config + Conformance Pack + 委任管理者

**ハンズオン:** なし（Security Hub有効化だけでもOK）

**問題演習:** ~25問
- exam-01: Q7, Q12, Q13, Q14, Q15
- exam-02: Q12, Q47, Q49
- exam-03: Q7, Q12, Q30, Q47, Q52, Q57, Q72
- exam-04: Q35, Q45, Q47, Q51, Q54, Q68, Q70
- exam-05: Q6, Q14, Q28, Q47, Q49, Q53
- exam-06: Q6, Q14, Q28, Q47, Q49, Q53

---

## Phase 2: ネットワーク設計（Week 2）

### テーマ4: Transit Gateway 🔴 3-4日
**座学ポイント:**
- ルートテーブル分離によるVPC間通信制御
- ハブ＆スポーク型構成
- TGWピアリング vs TGW統合
- オンプレミス接続（Direct Connect + TGW）
- ルート伝播の仕組み

**ハンズオン:**
- 2-3VPC作成 → TGW作成 → アタッチメント
- ルートテーブル分離を設定し通信制御を確認
- 片方のVPCを分離する設定

**問題演習:** ~20問
- exam-01: Q1
- exam-03: Q32, Q44, Q69
- exam-04: Q38, Q44, Q54
- exam-05: Q20, Q24, Q35, Q54, Q63
- exam-06: Q20, Q24, Q63

---

### テーマ5: PrivateLink + VPCエンドポイント 🟡 2日
**座学ポイント:**
- Interface Endpoint vs Gateway Endpoint
- PrivateLinkの一方向アクセス特性
- Storage GatewayはInterface Endpointのみ対応
- S3 VPCエンドポイント + バケットポリシー（aws:SourceVpce）
- 共有サービスVPCへのPrivateLink接続

**ハンズオン:** S3ゲートウェイエンドポイント作成

**問題演習:** ~10問
- exam-02: Q5
- exam-03: Q37, Q45
- exam-04: Q52
- exam-05: Q7, Q39, Q63
- exam-06: Q39, Q63

---

### テーマ6: Route 53高度設計 🟡 2日
**座学ポイント:**
- Application Recovery Controller（ARC）+ CloudWatch連携
- フェイルオーバー vs レイテンシー vs 加重 vs 地理ルーティング
- ALIASレコード（ApexドメインでALB指定）
- ヘルスチェック（18%ルール、CloudWatchアラーム連携）
- プライベートホストゾーン + VPC DNS設定必須

**ハンズオン:** フェイルオーバー構成（2ALB+ヘルスチェック）

**問題演習:** ~15問
- exam-01: Q20, Q31, Q34
- exam-02: Q29
- exam-03: Q28, Q43, Q51
- exam-04: Q30, Q45
- exam-05: Q30, Q43, Q59, Q69
- exam-06: Q43, Q59

---

### テーマ7: Global Accelerator + CloudFront + Lambda@Edge 🟡 2日
**座学ポイント:**
- Global Accelerator: TCP/UDP両対応、静的IP、エッジルーティング
- CloudFront: OAC（S3）、カスタムヘッダー+WAF（ALB）
- Lambda@Edge: 地理/Cookie/User-Agentによる動的オリジン切替
- API Gatewayエッジ最適化モード
- CloudFront署名付きURL vs 署名付きCookie

**ハンズオン:** CloudFront + S3 OAC構築

**問題演習:** ~15問
- exam-01: Q2, Q4, Q22
- exam-02: Q3
- exam-03: Q23, Q35, Q47
- exam-04: Q18, Q24
- exam-05: Q2, Q8, Q10, Q19, Q36, Q72
- exam-06: Q2, Q7, Q8, Q10, Q19, Q72

---

## Phase 3: データベース（Week 3）

### テーマ8: Aurora高度設計 🔴 3-4日
**座学ポイント:**
- Aurora Global Database（RPO設定、リードレプリカ展開）
- Aurora Multi-Master（MySQL互換のみ）
- Readerエンドポイント（読み取り分散）
- Redshift Federated Query（Aurora PostgreSQLに直接クエリ）
- Aurora Serverless v2（自動スケーリング）

**ハンズオン:**
- Auroraクラスター作成 + リードレプリカ追加
- Readerエンドポイント経由の接続確認

**問題演習:** ~20問
- exam-01: Q44
- exam-02: Q17, Q19, Q20
- exam-03: Q34
- exam-04: Q12, Q14, Q34, Q72, Q74, Q75
- exam-05: Q55, Q58, Q61, Q64, Q65, Q69
- exam-06: Q13, Q33, Q51, Q55, Q58, Q59, Q64, Q65, Q69

---

### テーマ9: RDS + RDS Proxy + DMS 🟡 2日
**座学ポイント:**
- RDS Proxyの仕組み（ターゲット＝Writer/Readerエンドポイント）
- Lambda + RDS Proxyの接続プール
- RDSリードレプリカ（クロスリージョン対応）
- Multi-AZ vs リードレプリカの違い
- DMS（データ検証機能、CDC、Schema Conversion Tool）
- VPC内LambdaのNAT Gateway要件

**ハンズオン:** Lambda → RDS Proxy → Aurora接続

**問題演習:** ~15問
- exam-01: Q31
- exam-02: Q17, Q50
- exam-03: Q41, Q50
- exam-04: Q46, Q48
- exam-05: Q25, Q46, Q48, Q51
- exam-06: Q13, Q25, Q33, Q48, Q51

---

### テーマ10: NoSQL使い分け 🟡 2日
**座学ポイント:**
- DynamoDB: Streams、Global Tables、DAX、PartiQL
- DocumentDB: MongoDB互換、ドキュメント指向
- Amazon Keyspaces: Cassandra（CQL）互換
- Amazon Neptune: グラフDB（Gremlin/SPARQL）
- Amazon MemoryDB: Redis互換 + Multi-AZ + 永続化
- Amazon Timestream: 時系列DB、統計関数、階層ストレージ
- ElastiCache Redis vs Memcached（Auto Discovery）

**ハンズオン:** なし（比較表を作って整理）

**問題演習:** ~20問
- exam-02: Q15, Q28, Q46
- exam-03: Q34, Q36, Q39, Q41, Q53, Q61, Q63
- exam-04: Q34, Q39, Q61
- exam-05: Q3, Q6, Q15, Q17, Q30, Q38
- exam-06: Q3, Q15, Q17, Q30, Q38

---

## Phase 4: ストレージ（Week 3-4）

### テーマ11: S3セキュリティ 🔴 3-4日
**座学ポイント:**
- OAC（Origin Access Control）+ バケットポリシー
- S3 Object Lock（コンプライアンスモード vs ガバナンスモード）
- S3 Glacier Vault Lock（WORM制御）
- SCP による暗号化強制（s3:x-amz-server-side-encryption条件）
- KMS: GenerateDataKey（SSE-KMS PutObject必須）
- VPCエンドポイント + バケットポリシー（aws:SourceVpce）
- タグベースコスト配分レポート

**ハンズオン:**
- S3バケット作成 + デフォルト暗号化（CMK）
- OAC + CloudFront構成
- オブジェクトロック設定

**問題演習:** ~20問
- exam-01: Q4, Q40
- exam-03: Q24, Q25, Q47
- exam-04: Q3, Q20, Q24, Q25, Q33, Q43, Q58, Q71
- exam-05: Q4, Q8, Q39, Q45, Q47, Q62
- exam-06: Q8, Q39, Q45, Q47, Q62, Q71

---

### テーマ12: S3運用（ライフサイクル・転送・レプリケーション） 🟡 2日
**座学ポイント:**
- ストレージクラス使い分け（Standard/IA/Glacier Flexible/Deep Archive）
- S3 Intelligent-Tiering（アーカイブ階層への自動移行）
- S3 Transfer Acceleration + マルチパートアップロード
- S3 CRR（クロスリージョンレプリケーション）+ RTC
- S3 Select / Range GET（部分読み取り）
- Athenaコスト最適化（Parquet + パーティション）

**ハンズオン:** Transfer Acceleration有効化

**問題演習:** ~15問
- exam-01: Q10, Q18, Q21, Q23, Q26, Q55
- exam-02: Q16, Q26, Q41
- exam-03: Q48, Q66, Q71, Q75
- exam-04: Q5, Q46, Q57, Q67, Q71
- exam-05: Q5, Q35, Q36, Q46, Q57
- exam-06: Q5, Q35, Q36, Q46, Q57

---

### テーマ13: 共有ストレージ（EFS・FSx・Storage Gateway） 🟢 1日
**座学ポイント:**
- EFS: マルチAZ NFS共有、複数EC2同時マウント
- FSx for Windows: SMBプロトコル、AD統合
- FSx for Lustre: HPC向け
- Storage Gateway: File Gateway（NFS→S3透過）、Volume Gateway（iSCSI）
- DataSync vs File Gatewayの使い分け

**ハンズオン:** なし

**問題演習:** ~10問
- exam-01: Q45
- exam-02: Q34, Q44, Q51
- exam-03: Q40, Q42, Q73, Q74
- exam-04: Q4, Q42, Q57
- exam-05: Q42, Q56, Q57, Q60
- exam-06: Q42, Q56, Q60

---

## Phase 5: 移行 + ハイブリッド（Week 4）※ Domain 4 / 配点20%

> この Phase を Week 4 に前倒しすることで、配点20%の重量級テーマに十分な復習時間を確保する。

### テーマ24: 移行戦略 + ハイブリッド 🟡 3日
**座学ポイント:**
- AWS Application Migration Service (MGN): リフト&シフト
- AWS DMS: フルロード+CDC、データ検証機能、Schema Conversion Tool
- Snowball Edge + DMS: 大容量低帯域環境
- Application Discovery Service: Agent（詳細メトリクス）vs Connector（構成情報）
- AWS Transfer Family: SFTPマネージド+S3バックエンド
- AWS Outposts + S3 on Outposts: 物理管理+AWSライク
- ECS Anywhere / EKS Anywhere / EKS Distro
- AWS Elastic Disaster Recovery: RPO数秒、RTO数分
- AWS Managed Blockchain: Hyperledger Fabric
- AppFlow: SaaS（Salesforce/Marketo）ノーコード連携

**ハンズオン:** なし（ドキュメント+比較表）

**問題演習:** ~30問
- exam-01: Q43
- exam-02: Q33, Q34, Q38, Q39, Q43, Q45, Q48, Q53, Q58, Q59
- exam-03: Q10, Q38, Q40, Q48, Q53, Q56, Q58, Q59
- exam-04: Q10, Q17, Q19, Q22, Q23, Q26, Q40, Q48, Q53
- exam-05: Q17, Q19, Q23, Q26, Q29, Q48
- exam-06: Q4, Q16, Q17, Q23, Q26, Q29, Q48, Q60

---

## Phase 6: サーバーレス + イベント駆動（Week 4-5）

### テーマ14: Lambda統合 🟢 1日
**座学ポイント:**
- VPC内Lambda + NAT Gateway（外部アクセス必須）
- Lambda環境変数の限界 → DynamoDB/SSMで動的取得
- Secrets Manager + 自動ローテーション
- SSM Parameter Store SecureString + KMS
- RDS Proxy連携（テーマ9と重複あり）

**ハンズオン:** なし（テーマ9のハンズオンで対応済み）

**問題演習:** ~15問
- exam-01: Q17, Q50
- exam-02: Q35
- exam-03: Q16, Q19, Q23, Q50
- exam-04: Q23, Q66
- exam-05: Q2, Q23, Q25, Q44
- exam-06: Q2, Q25, Q44

---

### テーマ15: 非同期処理パターン 🔴 3-4日
**座学ポイント:**
- API Gateway → SQS → EC2ワーカー（即時応答+非同期処理）
- SQS Standard vs FIFO（At-Least-Once vs Exactly-Once）
- FIFO + 重複排除ID + コンテンツベース重複排除
- CloudWatchカスタムメトリクス（バックログ）+ ターゲット追跡スケーリング
- DynamoDB Streams + Lambda + SNS（リアルタイム変更検知）
- EventBridgeクロスアカウント（中央バス+Lambda）
- S3イベント → Lambda → DynamoDB

**ハンズオン:**
- S3 → Lambda → DynamoDB 構築
- SQS FIFOキュー作成 + メッセージ送受信

**問題演習:** ~20問
- exam-01: Q50, Q60
- exam-02: Q8, Q10, Q12
- exam-03: Q11, Q40, Q55, Q63, Q71
- exam-04: Q11, Q27, Q30, Q40, Q55
- exam-05: Q11, Q18, Q30, Q66, Q74
- exam-06: Q11, Q18, Q30, Q66, Q74

---

### テーマ16: API Gateway 🟢 1日
**座学ポイント:**
- Usage Plan + APIキー（クライアント別スロットリング）
- エッジ最適化 vs リージョン vs プライベートエンドポイント
- スロットル制限（429エラー）の原因診断
- CloudFront統合との違い

**ハンズオン:** なし

**問題演習:** ~10問
- exam-01: Q49
- exam-03: Q6, Q15, Q27
- exam-04: Q15, Q49
- exam-05: Q27, Q73
- exam-06: Q27, Q73

---

## Phase 7: CI/CD + DevOps（Week 5）※ Domain 3 / 配点25%

> Domain 3「既存ソリューションの継続的改善」は25%と最重量級。CI/CDとIaCのパターンを早めに押さえる。

### テーマ25: CI/CD + DevOps 🟡 3日
**座学ポイント:**
- CodePipeline + CodeBuild + CodeDeploy: CI/CD統合
- AWS SAM + CodeDeploy: Lambda段階的デプロイ（トラフィックシフト）
- AWS Proton: セルフサービスプロビジョニング
- AWS CodeArtifact: 依存パッケージ管理（マルチアカウント共有）
- CodeGuru Reviewer/Profiler: Java/Python品質分析
- AWS Device Farm: モバイル実機テスト
- CloudFormation StackSets: マルチアカウント展開
- CloudFormation + ユーザーデータ: IaC自動化
- Fn::Cidr関数: VPC CIDR自動生成
- SSM RunPatchBaseline + メンテナンスウィンドウ
- Elastic Beanstalk Blue/Green + RDS分離
- AWS Health Dashboard（Organizations）
- ACM証明書（リージョン単位発行）

**ハンズオン:** なし

**問題演習:** ~20問
- exam-02: Q31, Q42, Q57
- exam-03: Q17, Q25, Q31, Q33, Q42, Q51, Q56, Q57, Q63, Q65, Q74
- exam-04: Q22, Q25, Q31, Q37, Q41, Q44, Q51, Q56, Q59, Q65, Q66
- exam-05: Q16, Q25, Q37, Q44, Q50, Q56, Q59, Q65, Q66
- exam-06: Q16, Q37, Q50

---

## Phase 8: コンピュート + コスト（Week 5-6）

### テーマ17: Auto Scaling戦略 🔴 3-4日
**座学ポイント:**
- ターゲット追跡 vs ステップ vs シンプルスケーリング
- RequestCountPerTarget（ALBメトリクス）
- AZウェイト調整（RDSプライマリAZ優先）
- 容量リバランス（スポット中断予兆検知）
- ライフサイクルフック（初期化完了後にトラフィック受付）
- スポット + リザーブド + オンデマンド ミックス構成
- Auto Scalingの動作原則（起動が先、終了が後）

**ハンズオン:**
- ASG + ALB 作成
- ターゲット追跡ポリシー設定
- スケールアウト/インの動作確認

**問題演習:** ~15問
- exam-01: Q8, Q9
- exam-02: Q1, Q13, Q18
- exam-03: Q5, Q41, Q55
- exam-04: Q5, Q48, Q50, Q52, Q55, Q64
- exam-05: Q5, Q7, Q8, Q21, Q52
- exam-06: Q7, Q21, Q52, Q64

---

### テーマ18: コスト最適化 🟡 2日
**座学ポイント:**
- Compute Savings Plans（EC2/Fargate/Lambda横断）
- EC2 Reserved Instances vs Savings Plans
- Redshift Reserved Nodes
- AWS Budgets + SNS + Lambda + SCP（予算超過自動制限）
- AWS Compute Optimizer（EC2/EBSリサイズ推奨）
- Service Quotas + Lambda自動申請
- AWS License Manager（ライセンス追跡+起動制御）
- Cost and Usage Report (CUR) + Athena
- AWS Batch + Fargate Spot（バッチ処理最適化）
- Organizations RI共有スコープ制御

**ハンズオン:** Budgetsアラート設定

**問題演習:** ~20問
- exam-02: Q1, Q32
- exam-03: Q4, Q11, Q15, Q59, Q73
- exam-04: Q4, Q13, Q20, Q42, Q46, Q47, Q59, Q68
- exam-05: Q8, Q11, Q20, Q32, Q34, Q41
- exam-06: Q8, Q11, Q20, Q32, Q34, Q41, Q42

---

## Phase 9: データ分析（Week 6）

### テーマ19: 分析基盤（Athena・Redshift・EMR・Glue） 🟡 2日
**座学ポイント:**
- Athena: S3直接クエリ、Parquet+パーティション、コスト＝スキャン量
- Redshift: RA3ノード（ストレージ分離）、Spectrum（S3外部テーブル）
- Redshift Federated Query（Aurora PostgreSQL直接クエリ）
- EMR + Spark SQL: 大規模並列分散処理、S3直接読み込み
- AWS Glue: ETLジョブ、ワークフロー、Data Catalog
- Amazon OpenSearch Service: 全文検索、kuromoji、Kibana
- Amazon QuickSight: BIダッシュボード
- Lake Formation: 行・列レベルアクセス制御

**ハンズオン:** なし（概念理解で十分）

**問題演習:** ~20問
- exam-01: Q21, Q25, Q26, Q27, Q39, Q46, Q48
- exam-02: Q23, Q24, Q53
- exam-03: Q23, Q33, Q44, Q48, Q53, Q58, Q75
- exam-04: Q6, Q23, Q33, Q58, Q67, Q70, Q75
- exam-05: Q13, Q22, Q32, Q54, Q67, Q70
- exam-06: Q5, Q13, Q22, Q32, Q54, Q67, Q70

---

### テーマ20: ストリーム処理（Firehose・Flink・MSK・Kinesis） 🟡 2日
**座学ポイント:**
- Amazon Data Firehose: Lambda変換+Parquet出力+S3/Redshift/OpenSearch
- Amazon Managed Service for Apache Flink: 状態付きリアルタイム集計
- Amazon MSK: Kafka完全互換マネージド
- Kinesis Data Streams: リアルタイム取り込み
- KCL + DynamoDBチェックポイント
- Firehose vs Kinesis Data Streamsの使い分け

**ハンズオン:** なし

**問題演習:** ~15問
- exam-01: Q60
- exam-02: Q20
- exam-03: Q13, Q20, Q21, Q62
- exam-04: Q13, Q21, Q62
- exam-05: Q13, Q21, Q22, Q38
- exam-06: Q22, Q38

---

## Phase 10: AI/ML + IoT + メディア（Week 6）※ 出題比率低め・サービス名と用途の把握で十分

> SageMaker/Rekognition などは「サービス名と用途が言える」レベルで十分。Phase を短縮して移行・DevOps の復習時間に回す。

### テーマ21: AI/MLサービス 🟢 1日
**座学ポイント:**
- Amazon SageMaker: ML統合環境、Model Monitor
- Amazon Comprehend: NLP（感情分析・エンティティ抽出・言語検出）
- Amazon Translate: 機械翻訳、Custom Terminology
- Amazon Polly: TTS（テキスト→音声）、SSML
- Amazon Transcribe: STT（音声→テキスト）、話者分離
- Amazon Textract: OCR + 請求書テンプレート
- Amazon Kendra: AI検索（SaaSコネクタ対応）
- Amazon Personalize: AutoMLレコメンド
- Amazon Fraud Detector: 不正検知
- Amazon Rekognition: 画像・動画分析

**ハンズオン:** なし（サービス比較表を作成）

**問題演習:** ~25問
- exam-02: Q2, Q22
- exam-03: Q2, Q22, Q24, Q29, Q37, Q43, Q46
- exam-04: Q27, Q37, Q41, Q43
- exam-05: Q12, Q27, Q41
- exam-06: Q12

---

### テーマ22: IoT 🟢 1日
**座学ポイント:**
- IoT Core: MQTT、デバイス認証、ルールエンジン
- IoT Events: 状態遷移ベースのイベント検出
- IoT SiteWise: 産業データ、アセットモデル、OEE、Monitor
- IoT Greengrass: エッジML推論、ローカルLambda
- IoT Device Defender: セキュリティ監査
- IoT Device Management: Fleet Hub、OTA更新
- IoT 1-Click: 物理ボタン+Lambda

**ハンズオン:** なし

**問題演習:** ~15問
- exam-03: Q3, Q8, Q13, Q29
- exam-04: Q29, Q47, Q50, Q72
- exam-05: Q47, Q50, Q72

---

### テーマ23: メディア配信 + 通知 🟢 1日
**座学ポイント:**
- AWS Elemental MediaConvert: オンデマンド動画変換（HLS）
- Amazon Elastic Transcoder: MP4→HLS変換
- MediaLive: ライブ配信
- Kinesis Video Streams: H.264映像取り込み、WebRTC
- Amazon Pinpoint: マルチチャネルキャンペーン
- Amazon SES: トランザクションメール、DKIM/SPF
- Amazon Connect + Lex + Lambda: 音声ボット

**ハンズオン:** なし

**問題演習:** ~10問
- exam-01: Q52
- exam-02: Q21
- exam-03: Q16, Q19
- exam-04: Q16, Q19, Q53
- exam-05: Q12, Q53
- exam-06: Q12

---

## 📊 全体サマリー

| Phase | テーマ | 所要日数 | 対応問題数 | 主なDomain |
|-------|--------|---------|-----------|-----------|
| Phase 1: ガバナンス | テーマ1-3 | 7-8日 | ~65問 | D1/D2 |
| Phase 2: ネットワーク | テーマ4-7 | 9-10日 | ~60問 | D1/D2 |
| Phase 3: データベース | テーマ8-10 | 7-8日 | ~55問 | D2 |
| Phase 4: ストレージ | テーマ11-13 | 6-7日 | ~45問 | D2 |
| **Phase 5: 移行** | **テーマ24** | **3日** | **~30問** | **D4(20%)** |
| Phase 6: サーバーレス | テーマ14-16 | 5-6日 | ~45問 | D2 |
| **Phase 7: CI/CD** | **テーマ25** | **3日** | **~20問** | **D3(25%)** |
| Phase 8: コンピュート+コスト | テーマ17-18 | 5-6日 | ~35問 | D2/D3 |
| Phase 9: データ分析 | テーマ19-20 | 4日 | ~35問 | D2 |
| Phase 10: AI/ML+IoT+メディア | テーマ21-23 | 3日 | ~50問 | D2 |
| **合計** | **25テーマ** | **52-55日** | **~420問** | |

---

## ✅ 進捗管理

各テーマ完了時にここにチェック：

- [ ] テーマ1: Organizations + SCP
- [ ] テーマ2: IAM高度設計
- [ ] テーマ3: セキュリティ監査
- [ ] テーマ4: Transit Gateway
- [ ] テーマ5: PrivateLink + VPCエンドポイント
- [ ] テーマ6: Route 53高度設計
- [ ] テーマ7: Global Accelerator + CloudFront
- [ ] テーマ8: Aurora高度設計
- [ ] テーマ9: RDS + RDS Proxy + DMS
- [ ] テーマ10: NoSQL使い分け
- [ ] テーマ11: S3セキュリティ
- [ ] テーマ12: S3運用
- [ ] テーマ13: 共有ストレージ
- [ ] **テーマ24: 移行戦略 + ハイブリッド** ← Phase 5に前倒し
- [ ] テーマ14: Lambda統合
- [ ] テーマ15: 非同期処理パターン
- [ ] テーマ16: API Gateway
- [ ] **テーマ25: CI/CD + DevOps** ← Phase 7に前倒し
- [ ] テーマ17: Auto Scaling戦略
- [ ] テーマ18: コスト最適化
- [ ] テーマ19: 分析基盤
- [ ] テーマ20: ストリーム処理
- [ ] テーマ21: AI/MLサービス
- [ ] テーマ22: IoT
- [ ] テーマ23: メディア配信 + 通知
