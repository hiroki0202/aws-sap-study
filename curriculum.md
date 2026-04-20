# 📋 カリキュラム

> **全25テーマ / 約420問 / 6週間完走プラン**

---

## 進捗管理

### Phase 1: ガバナンス・セキュリティ基盤（Week 1）
- [ ] テーマ1: Organizations + SCP + OU設計（🔴 3-4日）
- [ ] テーマ2: IAM高度設計（🟡 2日）
- [ ] テーマ3: セキュリティ監査（🟡 2日）

### Phase 2: ネットワーク設計（Week 2）
- [ ] テーマ4: Transit Gateway（🔴 3-4日）
- [ ] テーマ5: PrivateLink + VPCエンドポイント（🟡 2日）
- [ ] テーマ6: Route 53高度設計（🟡 2日）
- [ ] テーマ7: Global Accelerator + CloudFront + Lambda@Edge（🟡 2日）

### Phase 3: データベース（Week 3）
- [ ] テーマ8: Aurora高度設計（🔴 3-4日）
- [ ] テーマ9: RDS + RDS Proxy + DMS（🟡 2日）
- [ ] テーマ10: NoSQL使い分け（🟡 2日）

### Phase 4: ストレージ（Week 3-4）
- [ ] テーマ11: S3セキュリティ（🔴 3-4日）
- [ ] テーマ12: S3運用（🟡 2日）
- [ ] テーマ13: 共有ストレージ（🟢 1日）

### Phase 5: サーバーレス + イベント駆動（Week 4）
- [ ] テーマ14: Lambda統合（🟢 1日）
- [ ] テーマ15: 非同期処理パターン（🔴 3-4日）
- [ ] テーマ16: API Gateway（🟢 1日）

### Phase 6: コンピュート + コスト（Week 4-5）
- [ ] テーマ17: Auto Scaling戦略（🔴 3-4日）
- [ ] テーマ18: コスト最適化（🟡 2日）

### Phase 7: データ分析（Week 5）
- [ ] テーマ19: 分析基盤（🟡 2日）
- [ ] テーマ20: ストリーム処理（🟡 2日）

### Phase 8: AI/ML + IoT + メディア（Week 5-6）
- [ ] テーマ21: AI/MLサービス（🟢 1日）
- [ ] テーマ22: IoT（🟢 1日）
- [ ] テーマ23: メディア配信 + 通知（🟢 1日）

### Phase 9: 移行 + DevOps（Week 6）
- [ ] テーマ24: 移行戦略 + ハイブリッド（🟡 2日）
- [ ] テーマ25: CI/CD + DevOps（🟢 1日）

---

## 凡例

| マーク | 意味 | 日数 | 内容 |
|--------|------|------|------|
| 🔴 | 重い | 3-4日 | 座学 + ハンズオン + 問題演習 |
| 🟡 | 普通 | 2日 | 座学 + 問題演習 |
| 🟢 | 軽い | 1日 | 座学 + 問題を同日 |

---

## 各テーマの学習フロー

```
Day 1: 座学（概念・比較表・ポイントを読む）
  ↓
Day 2: ハンズオン（🔴テーマのみ）
  ↓
Day 3: 問題演習（間違えたら解説を確認）
  ↓
数日後: 間違えた問題だけ再挑戦
```

---

## 全体サマリー

| Phase | テーマ数 | 所要日数 | 練習問題数 |
|-------|---------|---------|-----------|
| Phase 1: ガバナンス | 3 | 7-8日 | ~65問 |
| Phase 2: ネットワーク | 4 | 9-10日 | ~60問 |
| Phase 3: データベース | 3 | 7-8日 | ~55問 |
| Phase 4: ストレージ | 3 | 6-7日 | ~45問 |
| Phase 5: サーバーレス | 3 | 5-6日 | ~45問 |
| Phase 6: コンピュート+コスト | 2 | 5-6日 | ~35問 |
| Phase 7: データ分析 | 2 | 4日 | ~35問 |
| Phase 8: AI/ML+IoT+メディア | 3 | 3日 | ~50問 |
| Phase 9: 移行+DevOps | 2 | 3日 | ~50問 |
| **合計** | **25** | **49-52日** | **~420問** |

---

## 想定スケジュール

| 週あたり学習時間 | 完走日数 |
|-----------------|---------|
| 週12時間（平日1h + 土日3.5h） | 8週間 |
| 週16.5時間（平日1.5h + 土日4.5h） | **6週間** |
| 週20時間（平日2h + 土日5h） | **5週間** |

---

## 各テーマ詳細

### Phase 1: ガバナンス・セキュリティ基盤

#### テーマ1: Organizations + SCP + OU設計 🔴
| 項目 | 内容 |
|------|------|
| 座学 | SCPの仕組み（継承・評価順序・ルートユーザーへの影響）、OU設計パターン（環境別・例外OU・セキュリティ集約）、SCPユースケース（リージョン制限・暗号化強制・インスタンスタイプ制限）、SCP vs IAM vs Permissions Boundary |
| ハンズオン | Organizations作成 → OU作成 → SCP作成（リージョン制限）→ OUにアタッチ → 継承動作確認 |
| 問題数 | ~25問 |

#### テーマ2: IAM高度設計 🟡
| 項目 | 内容 |
|------|------|
| 座学 | 権限境界（Permissions Boundary）、AssumeRole + 信頼ポリシー、STSセッションタグ、aws:PrincipalOrgID条件、外部ID（External ID） |
| ハンズオン | なし |
| 問題数 | ~15問 |

#### テーマ3: セキュリティ監査 🟡
| 項目 | 内容 |
|------|------|
| 座学 | CloudTrail整合性検証、詳細イベントセレクタ、IAM Access Analyzer、Security Hub（CISベンチマーク）、Macie、Inspector、Config + Conformance Pack + 委任管理者 |
| ハンズオン | なし（Security Hub有効化だけでもOK） |
| 問題数 | ~25問 |

---

### Phase 2: ネットワーク設計

#### テーマ4: Transit Gateway 🔴
| 項目 | 内容 |
|------|------|
| 座学 | ルートテーブル分離によるVPC間通信制御、ハブ＆スポーク型構成、TGWピアリング vs TGW統合、オンプレ接続（Direct Connect + TGW）、ルート伝播 |
| ハンズオン | 2-3VPC作成 → TGW作成 → アタッチメント → ルートテーブル分離で通信制御確認 |
| 問題数 | ~20問 |

#### テーマ5: PrivateLink + VPCエンドポイント 🟡
| 項目 | 内容 |
|------|------|
| 座学 | Interface vs Gateway Endpoint、PrivateLinkの一方向アクセス、Storage GatewayはInterface Endpointのみ、S3 VPCエンドポイント + バケットポリシー（aws:SourceVpce）、共有サービスVPCへの接続 |
| ハンズオン | S3ゲートウェイエンドポイント作成 |
| 問題数 | ~10問 |

#### テーマ6: Route 53高度設計 🟡
| 項目 | 内容 |
|------|------|
| 座学 | Application Recovery Controller（ARC）+ CloudWatch連携、フェイルオーバー/レイテンシー/加重/地理ルーティング、ALIASレコード（Apexドメイン）、ヘルスチェック（18%ルール、CloudWatchアラーム連携）、プライベートホストゾーン + VPC DNS設定 |
| ハンズオン | フェイルオーバー構成（2ALB+ヘルスチェック） |
| 問題数 | ~15問 |

#### テーマ7: Global Accelerator + CloudFront + Lambda@Edge 🟡
| 項目 | 内容 |
|------|------|
| 座学 | Global Accelerator（TCP/UDP、静的IP、エッジルーティング）、CloudFront OAC（S3）+ カスタムヘッダー+WAF（ALB）、Lambda@Edge（地理/Cookie動的オリジン切替）、API Gatewayエッジ最適化モード、署名付きURL vs 署名付きCookie |
| ハンズオン | CloudFront + S3 OAC構築 |
| 問題数 | ~15問 |

---

### Phase 3: データベース

#### テーマ8: Aurora高度設計 🔴
| 項目 | 内容 |
|------|------|
| 座学 | Aurora Global Database（RPO設定、リードレプリカ展開）、Multi-Master（MySQL互換のみ）、Readerエンドポイント、Redshift Federated Query、Aurora Serverless v2 |
| ハンズオン | Auroraクラスター作成 + リードレプリカ追加 + Readerエンドポイント接続確認 |
| 問題数 | ~20問 |

#### テーマ9: RDS + RDS Proxy + DMS 🟡
| 項目 | 内容 |
|------|------|
| 座学 | RDS Proxyの仕組み（ターゲット＝Writer/Readerエンドポイント）、Lambda + RDS Proxy接続プール、リードレプリカ（クロスリージョン）、Multi-AZ vs リードレプリカの違い、DMS（データ検証、CDC、SCT）、VPC内LambdaのNAT Gateway要件 |
| ハンズオン | Lambda → RDS Proxy → Aurora接続 |
| 問題数 | ~15問 |

#### テーマ10: NoSQL使い分け 🟡
| 項目 | 内容 |
|------|------|
| 座学 | DynamoDB（Streams、Global Tables、DAX）、DocumentDB（MongoDB互換）、Keyspaces（Cassandra互換）、Neptune（グラフDB）、MemoryDB（Redis互換+永続化）、Timestream（時系列DB）、ElastiCache Redis/Memcached（Auto Discovery） |
| ハンズオン | なし（比較表を作って整理） |
| 問題数 | ~20問 |

---

### Phase 4: ストレージ

#### テーマ11: S3セキュリティ 🔴
| 項目 | 内容 |
|------|------|
| 座学 | OAC + バケットポリシー、Object Lock（コンプライアンス/ガバナンスモード）、Glacier Vault Lock（WORM）、SCP暗号化強制（s3:x-amz-server-side-encryption条件）、KMS GenerateDataKey、VPCエンドポイント+バケットポリシー |
| ハンズオン | S3バケット作成+デフォルト暗号化（CMK）、OAC+CloudFront構成、オブジェクトロック設定 |
| 問題数 | ~20問 |

#### テーマ12: S3運用 🟡
| 項目 | 内容 |
|------|------|
| 座学 | ストレージクラス（Standard/IA/Glacier Flexible/Deep Archive）、Intelligent-Tiering、Transfer Acceleration + マルチパート、CRR + RTC、S3 Select / Range GET、Athenaコスト最適化（Parquet + パーティション） |
| ハンズオン | Transfer Acceleration有効化 |
| 問題数 | ~15問 |

#### テーマ13: 共有ストレージ 🟢
| 項目 | 内容 |
|------|------|
| 座学 | EFS（マルチAZ NFS共有）、FSx for Windows（SMB/AD統合）、FSx for Lustre、Storage Gateway（File/Volume）、DataSync vs File Gateway |
| ハンズオン | なし |
| 問題数 | ~10問 |

---

### Phase 5: サーバーレス + イベント駆動

#### テーマ14: Lambda統合 🟢
| 項目 | 内容 |
|------|------|
| 座学 | VPC内Lambda + NAT Gateway要件、Lambda環境変数の限界→動的取得、Secrets Manager + 自動ローテーション、SSM SecureString + KMS、RDS Proxy連携 |
| ハンズオン | なし |
| 問題数 | ~15問 |

#### テーマ15: 非同期処理パターン 🔴
| 項目 | 内容 |
|------|------|
| 座学 | API Gateway→SQS→EC2ワーカー、SQS Standard vs FIFO、重複排除ID、CloudWatchカスタムメトリクス+ターゲット追跡、DynamoDB Streams+Lambda+SNS、EventBridgeクロスアカウント、S3→Lambda→DynamoDB |
| ハンズオン | S3→Lambda→DynamoDB構築、SQS FIFOキュー作成+送受信 |
| 問題数 | ~20問 |

#### テーマ16: API Gateway 🟢
| 項目 | 内容 |
|------|------|
| 座学 | Usage Plan+APIキー（クライアント別スロットリング）、エッジ最適化/リージョン/プライベート、スロットル制限（429エラー）診断 |
| ハンズオン | なし |
| 問題数 | ~10問 |

---

### Phase 6: コンピュート + コスト

#### テーマ17: Auto Scaling戦略 🔴
| 項目 | 内容 |
|------|------|
| 座学 | ターゲット追跡 vs ステップ vs シンプル、RequestCountPerTarget、AZウェイト調整、容量リバランス、ライフサイクルフック、スポット+リザーブド+オンデマンド混合、Auto Scaling動作原則（起動先→終了後） |
| ハンズオン | ASG+ALB作成、ターゲット追跡ポリシー設定、スケール動作確認 |
| 問題数 | ~15問 |

#### テーマ18: コスト最適化 🟡
| 項目 | 内容 |
|------|------|
| 座学 | Compute Savings Plans（EC2/Fargate/Lambda横断）、RI vs Savings Plans、Budgets+SCP+EventBridge、Compute Optimizer、Service Quotas+Lambda自動申請、License Manager、CUR+Athena、Batch+Fargate Spot、RI共有スコープ |
| ハンズオン | Budgetsアラート設定 |
| 問題数 | ~20問 |

---

### Phase 7: データ分析

#### テーマ19: 分析基盤 🟡
| 項目 | 内容 |
|------|------|
| 座学 | Athena（S3直接、Parquet+パーティション）、Redshift（RA3、Spectrum）、Federated Query、EMR+Spark SQL、Glue（ETL、Data Catalog）、OpenSearch Service、QuickSight、Lake Formation |
| ハンズオン | なし |
| 問題数 | ~20問 |

#### テーマ20: ストリーム処理 🟡
| 項目 | 内容 |
|------|------|
| 座学 | Data Firehose（Lambda変換+Parquet出力）、Managed Flink（状態付きリアルタイム集計）、MSK（Kafka互換）、Kinesis Data Streams、KCL+DynamoDBチェックポイント、AppFlow（SaaS連携） |
| ハンズオン | なし |
| 問題数 | ~15問 |

---

### Phase 8: AI/ML + IoT + メディア

#### テーマ21: AI/MLサービス 🟢
| 項目 | 内容 |
|------|------|
| 座学 | SageMaker、Comprehend（NLP）、Translate（機械翻訳+用語集）、Polly（TTS）、Transcribe（STT）、Textract（OCR+請求書）、Kendra（AI検索）、Personalize（レコメンド）、Fraud Detector（不正検知）、Rekognition（画像） |
| ハンズオン | なし（比較表作成） |
| 問題数 | ~25問 |

#### テーマ22: IoT 🟢
| 項目 | 内容 |
|------|------|
| 座学 | IoT Core（MQTT、デバイス認証）、IoT Events（状態遷移イベント）、IoT SiteWise（産業データ、OEE）、IoT Greengrass（エッジML）、IoT Device Defender/Management、IoT 1-Click、Kinesis Video Streams |
| ハンズオン | なし |
| 問題数 | ~15問 |

#### テーマ23: メディア配信 + 通知 🟢
| 項目 | 内容 |
|------|------|
| 座学 | MediaConvert（HLS変換）、Elastic Transcoder、Kinesis Video Streams（H.264）、Pinpoint（マルチチャネル）、SES（トランザクションメール）、Connect+Lex+Lambda（音声ボット）、AppSync（GraphQL+オフライン同期） |
| ハンズオン | なし |
| 問題数 | ~10問 |

---

### Phase 9: 移行 + DevOps

#### テーマ24: 移行戦略 + ハイブリッド 🟡
| 項目 | 内容 |
|------|------|
| 座学 | Application Migration Service（MGN）、DMS（フルロード+CDC、データ検証、SCT）、Snowball Edge+DMS、Discovery Agent vs Connector、Transfer Family（SFTP）、Outposts+S3 on Outposts、ECS/EKS Anywhere、EKS Distro、Elastic Disaster Recovery、Managed Blockchain、AppFlow、Data Exchange |
| ハンズオン | なし（ドキュメント+比較表） |
| 問題数 | ~30問 |

#### テーマ25: CI/CD + DevOps 🟢
| 項目 | 内容 |
|------|------|
| 座学 | CodePipeline+CodeBuild+CodeDeploy、SAM+CodeDeploy（Lambda段階的デプロイ）、Proton、CodeArtifact、CodeGuru、Device Farm、CloudFormation StackSets、Fn::Cidr、SSM RunPatchBaseline+メンテナンスウィンドウ、Beanstalk Blue/Green+RDS分離、Health Dashboard、ACM証明書（リージョン単位） |
| ハンズオン | なし |
| 問題数 | ~20問 |
