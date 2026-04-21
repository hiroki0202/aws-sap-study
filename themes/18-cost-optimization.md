# テーマ18: コスト最適化

> 🟡 所要日数: 2日 | 座学 → 問題演習

---

## 座学

## Part 1: SAAからの差分 — コスト最適化の全体像

SAAではReserved Instance・Savings Plans・Spotの基本を学びました。SAPでは**全社横断のコスト管理**と**アーキテクチャレベルの最適化判断**が問われます。

領域は以下です。**コンピュート料金モデル**（RI、Savings Plans、Spot、Dedicated）、**ストレージコスト最適化**（S3ライフサイクル、EBS gp3、EFS IA）、**データ転送コスト**（VPCエンドポイント、CloudFront）、**コスト可視化**（Cost Explorer、Budgets、CUR、Cost Categories）、**組織横断の購入計画**（Organizationsでの共有）。

---

## Part 2: コンピュート料金モデルの体系

**On-Demand**: 従量課金、予約なし。最も柔軟だが最も高い。

**Reserved Instances（RI）**:
- 1年/3年の契約、前払い・一部前払い・なし
- **Standard RI**: インスタンスタイプ固定、最大75%割引
- **Convertible RI**: インスタンスタイプ変更可、最大54%割引
- 特定リージョン・AZに紐付く、EC2専用

**Savings Plans**:
- 1年/3年、$/時間のコミットメント
- **Compute Savings Plans**: EC2、Lambda、Fargate横断で利用可、最大66%割引、**最も柔軟**
- **EC2 Instance Savings Plans**: 特定ファミリーに紐付く、最大72%割引
- **SageMaker Savings Plans**: SageMaker向け

**Spot Instances**:
- 未使用EC2キャパシティを最大90%割引で利用
- AWSが2分前通知で中断する可能性
- バッチ処理、ステートレスWeb、CI/CDに向く

**Dedicated Hosts / Dedicated Instances**:
- 物理ハードウェアを専有（コンプライアンス・ライセンス要件）

**使い分けマトリクス**:

| ワークロード | 推奨 |
|-----------|------|
| 24時間安定稼働、タイプ固定 | Standard RI（3年、一括前払い） |
| 複数のEC2・Lambda・Fargateで変動 | Compute Savings Plans |
| ステートレスバッチ処理 | Spot |
| 本番DB | On-Demand（RIも可） |
| BYOL（ライセンス持ち込み） | Dedicated Hosts |

---

## Part 3: ストレージコスト最適化

**S3**:
- Intelligent-Tieringで自動最適化
- ライフサイクルで段階的アーカイブ（Standard → IA → Glacier → Deep Archive）
- Glacier Instant Retrieval は「アーカイブだが即時取り出し」という中間層

**EBS**:
- **gp3**（SSD）: gp2より20%安価、IOPSとスループットを独立調整可能
- **sc1、st1**（HDD）: 大容量・低頻度アクセス向け
- **スナップショット**: 差分ベース、頻度・保持期間の見直し

**EFS**:
- Lifecycle Managementで自動IA移行
- One Zone-IAで最大86%コスト削減（AZ冗長不要な場合）

---

## Part 4: データ転送コスト — 見落としがちな出費

データ転送コストは構造が複雑でよく見落とされます。

**コストが発生するパターン**:
- AWS → インターネット（アウトバウンド）: $0.09/GB程度
- AZ間通信: $0.01/GB（双方向でそれぞれ課金）
- リージョン間通信: $0.02〜0.09/GB
- VPCピアリング: AZ間$0.01/GB

**コスト削減テクニック**:
1. **VPCエンドポイント**: S3/DynamoDBへのアクセスでNAT Gateway料金を排除
2. **CloudFront**: オリジンへのデータ転送を最適化、エッジでキャッシュ
3. **Direct Connect**: 大量データ転送でインターネットより安価
4. **リージョンの選択**: 同一リージョンでのサービス配置、不要なリージョン跨ぎを避ける
5. **Compression**: ネットワーク転送前の圧縮

**NAT Gateway vs VPCエンドポイント**: プライベートサブネットのLambda/EC2からS3へ大量アクセスする場合、NAT Gateway経由だとデータ転送料金が発生。Gateway Endpoint（S3）を使えば無料。

---

## Part 5: コスト可視化・管理ツール

**Cost Explorer**: コスト傾向の可視化、予測、サービス別・リージョン別・タグ別に分析、RI Utilization / Coverage レポート

**AWS Budgets**: 予算アラート（日/月/四半期/年）、Cost / Usage / Savings Plans / RI に対するアラート、アクション付き Budgets（閾値超過でIAMポリシー適用、インスタンス停止など）

**Cost and Usage Reports（CUR）**: 最も詳細な課金データ（時間単位、リソース単位）、S3に出力、Athena・QuickSightで分析、第三者ツールへの連携

**Cost Categories**: カスタムのコスト分類を作成（部門別、プロジェクト別）、複数アカウント・複数タグの組み合わせで柔軟に分類

**AWS Compute Optimizer**: 機械学習でEC2/EBS/Lambda/ASG/ECSの最適化を推奨。「このインスタンスはm5.xlargeからm5.largeにダウンサイズで$X/月削減」など具体的提案

**Trusted Advisor**: コスト最適化、セキュリティ、パフォーマンス、耐障害性、サービス制限の推奨。ビジネス・エンタープライズサポート以上でフル機能

---

## Part 6: Organizations でのコスト管理

**統合請求（Consolidated Billing）**:
- Organizations の管理アカウントに全アカウントの請求を集約
- **ボリューム割引の共有**: 全アカウントの使用量を合算してティア割引を適用
- **RI/Savings Plansの共有**: 未使用分を他のアカウントで利用可能

**RI/Savings Plans Sharing**:
- デフォルトで有効、組織全体で共有
- 個別アカウントで無効化も可能
- 「購入計画専用のアカウント」を作り、そこで大量購入して全社で使う運用が一般的

---

## 練習問題

### 問題1

あるスタートアップでは、EC2（Web）・Lambda（API処理）・Fargate（バッチ）の3つを本番運用しており、どれも継続的に使うが、今後1年でLambda/Fargateの使用量が大幅に増える見込みです。コミットメントベースの割引を利用したいですが、将来の使用量構成が変化する予定なので柔軟性も必要です。

最適な購入戦略はどれですか？

<details>
<summary>選択肢を見る</summary>

A. EC2用にStandard RI（3年）、Lambda・Fargateは従量課金のまま

B. Compute Savings Plans（1年・3年）を購入する。EC2・Lambda・Fargate横断で利用でき、インスタンスタイプ・サービス変更でも自動適用され柔軟性がある

C. EC2 Instance Savings Plansで最大割引を得る

D. Spotインスタンスに全面移行する

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

Compute Savings Plansが正解です。EC2、Lambda、Fargate横断で利用でき、インスタンスタイプ・リージョン・ファミリー・サービスの変更に対して自動適用されます。最大66%の割引が得られ、将来のLambda/Fargate使用量増加にも追従します。

- A: Standard RIはEC2のみ対応、Lambda/Fargateに割引が効かない
- C: EC2 Instance Savings Plansは特定ファミリーに紐付き、Lambda/Fargateに適用不可
- D: 本番Webが全てSpotでいいかは用途次第で、可用性リスクが高い

</details>

---

### 問題2

ある企業のVPCでは、プライベートサブネットのEC2インスタンスがNAT Gateway経由でS3から大量のデータ（月10 TB）をダウンロードしています。月額のNAT Gatewayデータ処理料金が$450を超え、経費の問題になっています。

コストを削減する最適な方法はどれですか？

<details>
<summary>選択肢を見る</summary>

A. EC2インスタンスをパブリックサブネットに移動してNAT Gatewayを廃止する

B. VPCにS3のGateway Endpointを作成し、ルートテーブルに追加する。S3へのトラフィックはGateway Endpoint経由でAWS内部ネットワークを通り、NAT Gateway経由を回避。追加料金ゼロ

C. EC2をCloudFront経由でS3にアクセスするように変更する

D. NAT GatewayをNAT Instanceに置き換える

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

S3 Gateway Endpointが正解です。完全無料で、S3のトラフィックはAWS内部ネットワークを通りNAT Gatewayを使いません。設定はVPCにEndpointを作成しルートテーブルに追加するだけで、月10 TBのNAT Gateway データ処理料金がほぼゼロになります。

- A: パブリックサブネット配置はセキュリティリスク（インターネットから直接到達可能）
- C: CloudFrontはキャッシュには有効だが、大量ダウンロード用途でNAT Gateway代替にはならない
- D: NAT Instance自作は管理負荷のみ増え、データ処理料金の削減は限定的

</details>

---

### 問題3

ある大手企業では、AWS Organizationsで20の子会社アカウントを管理しています。各アカウントで独自にEC2 Reserved Instancesを購入しており、以下の問題が発生しています。

1. あるアカウントが購入したRIが別のアカウントの同じインスタンスタイプに自動適用されない（購入した子会社のみが割引）
2. 全社横断でのRI適用率が低く、未使用のRIがある一方で、別のアカウントではOn-Demandで支払っているケースがある

これを解決する最適な方法はどれですか？

<details>
<summary>選択肢を見る</summary>

A. 各アカウントで購入を継続し、RI余剰の可視化はCost Explorerで手動管理

B. AWS Organizationsの統合請求（Consolidated Billing）を有効化し、RI/Savings Plansの共有を有効にする。組織全体で未使用RIが他アカウントの該当インスタンスに自動適用され、全社の割引率が最大化される

C. 全てのRIを解約してSpotインスタンスに移行

D. 子会社ごとに別Organizationsを作成して独立運用

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

Organizations の Consolidated Billing + RI/Savings Plans Sharing が正解です。

- **統合請求**: 全アカウントの使用量を合算してボリューム割引のティアが向上
- **RI/SP Sharing**: 未使用のRI/Savings Plansが組織内の他アカウントに自動適用
- **結果**: アカウントAが購入してアカウントBで消費、というパターンで全社の適用率が向上
- **運用**: 購入専用アカウントを作って集中管理する運用も可能

- A: 手動管理は運用負荷が高く、自動化されない
- C: Spot全面移行は本番サービスの可用性リスクが大きい
- D: Organizations分離は割引共有の機会を失うため逆効果

</details>

---

### 問題4

ある企業では、CloudWatch Logsに大量のアプリケーションログを保存しており、ログの保存料金が月額$2,000を超えています。調査すると、ログの95%は「トラブルシューティング時のみ参照」される程度で、通常の運用では使われていないことが判明しました。

長期保管のコストを最小化しつつ、必要時には検索可能にする最適な構成はどれですか？

<details>
<summary>選択肢を見る</summary>

A. CloudWatch Logsの保持期間を1日に短縮する

B. ログを Kinesis Data Firehose 経由で S3 に配信し、S3ライフサイクルで30日後にGlacier、1年後にDeep Archiveに移行。検索はAthenaでオンデマンド実行

C. CloudWatch Logsの保持期間を10年に延長して全データを保持する

D. DynamoDBにログを保存する

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

Firehose + S3 + Athenaの組み合わせが正解です。

- **S3 Standard**: CloudWatch Logsの約1/3の料金
- **S3ライフサイクル**: 30日後IA、1年後Deep Archiveで大幅なコスト削減
- **Athena**: S3上のログをSQLで検索可能、実行時のみ課金
- **運用**: ログはCloudWatch Logsに短期（例: 7日）保持してリアルタイム監視、長期はS3にアーカイブ

- A: 保持期間1日では必要時の参照ができない
- C: 10年保持はCloudWatch Logsの高額な料金で保持することになり、むしろコスト増
- D: DynamoDBはログストアには向かず、料金も高い

</details>

---

### 問題5

ある企業では、複数の事業部門・プロジェクトが1つのAWSアカウントで運用されており、どの部門がどれだけコストを発生させているか可視化できていません。経理部門から「部門別・プロジェクト別のコスト分類レポートを毎月出力してほしい」という要求が出ています。

タグが一部リソースに設定済みですが、一部リソースにはタグが付いていません。リソースごとに柔軟な分類が必要です。

最適な方法はどれですか？

<details>
<summary>選択肢を見る</summary>

A. 全リソースにタグを100%付けてから、Cost Explorerでタグ別レポートを確認する

B. Cost Categoriesを使い、複数のタグ・アカウント・サービスの組み合わせで柔軟な分類ルールを定義。タグが未設定のリソースにも「分類ルール」で自動グルーピング。Cost Explorerやコストレポートでカテゴリ別に分析

C. リソースグループを使って部門別にまとめる

D. アカウントを部門ごとに分割する

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

Cost Categoriesが正解です。

- **柔軟な分類**: タグ、アカウント、サービス、リージョンなどの複数条件でカスタム分類を定義
- **タグ未設定リソースも対応**: 「このサービスかつこのリージョンのリソースは部門A」というルール定義が可能
- **Cost Explorerとの統合**: Cost Categoriesでの分類がCost Explorerのディメンションになる
- **複雑な組織体系に対応**: 部門とプロジェクトの両軸で分類可能

- A: 全リソースへのタグ付けは理想ですが実現が困難で、タグ運用ルールの徹底も課題
- C: リソースグループはリソースの管理単位で、コスト分類には直接使えない
- D: アカウント分割は大規模な移行が必要で、短期的な可視化要件には過剰

</details>

---

### 問題6

ある企業では、複数のEC2インスタンスがオーバープロビジョニングされている疑いがあります。CPU使用率が平均10%未満のインスタンスが多数存在しますが、どれをダウンサイズすべきか、どのくらいの節約になるかを個別に調査するには時間がかかります。

最も効率的にサイジング最適化を進める方法はどれですか？

<details>
<summary>選択肢を見る</summary>

A. 自作スクリプトでCloudWatchメトリクスを定期収集し、閾値以下のインスタンスを検出する

B. AWS Compute Optimizerを有効化する。機械学習でEC2・EBS・Lambda・ASG・ECSの最適化を自動分析し、「このインスタンスはm5.2xlargeからm5.largeへダウンサイズで$X/月削減可能」といった具体的な推奨事項を提供

C. 全EC2を一律にダウンサイズする

D. Trusted Advisorのコスト最適化チェックを手動で確認する

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

AWS Compute Optimizerが正解です。

- **自動分析**: CloudWatchメトリクス（CPU、メモリ、ネットワーク、ディスク）を機械学習で分析
- **具体的推奨**: 「このインスタンスはこのタイプに変更で$X/月削減」まで提案
- **対象**: EC2、EBS、Lambda、ASG、ECS Fargate
- **リスク分類**: 推奨を「リスクなし」「リスクあり」で分類、移行判断を支援

- A: 自作スクリプトは開発・運用コストが高く、Compute Optimizerと同等の分析精度は困難
- C: 一律ダウンサイズは使用率の高いインスタンスの性能問題を起こす
- D: Trusted Advisorも有効ですが、Compute Optimizerの方が詳細な機械学習ベースの分析を提供

</details>
