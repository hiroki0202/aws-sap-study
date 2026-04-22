# テーマ24: 移行 + ハイブリッド

> 🔴 所要日数: 3日 | 座学 → 問題演習

---

## 座学

## Part 1: SAAからの差分 — 移行サービスの全体像

SAAではSnowball、DMSの存在を軽く学びました。SAPでは**移行戦略の選択**と**大規模移行の設計**が深く問われます。

**6R の移行戦略**:

| 戦略 | 内容 |
|------|------|
| **Rehost（Lift-and-shift）** | そのまま移行。Server Migration Service、VMware Cloud on AWS |
| **Replatform** | OSや DB を変更（例: Oracle → Aurora PostgreSQL） |
| **Refactor / Re-architect** | AWS ネイティブへ再設計（モノリス → マイクロサービス） |
| **Repurchase** | SaaS に置き換え |
| **Retain** | 残す（コンプライアンス等） |
| **Retire** | 廃止 |

**AWS Migration Services**:

| サービス | 役割 |
|---------|------|
| **Migration Hub** | 移行の一元管理、進捗追跡 |
| **Application Migration Service（MGN）** | サーバーのRehost、旧Server Migration Service後継 |
| **Database Migration Service（DMS）** | DB移行（テーマ9） |
| **Schema Conversion Tool（SCT）** | スキーマ変換 |
| **DataSync** | ファイル・オブジェクトの大量転送 |
| **Snow Family** | 物理デバイスでの大量データ転送 |
| **Storage Gateway** | オンプレ・AWS間のブリッジ |
| **Transfer Family** | SFTP/FTPS/FTPでのファイル転送 |

---

## Part 2: AWS Application Migration Service（MGN）

**MGN**はサーバーをRehost（Lift-and-shift）で移行するサービスです。旧 Server Migration Service (SMS) の後継。

**動作原理**:
1. オンプレミスのサーバーに **Replication Agent** をインストール
2. AgentがEBSへの継続的ブロックレベルレプリケーション
3. テスト時やカットオーバー時に、レプリケーションの最新データから EC2 インスタンスを起動
4. DNS切り替えで本番切替

**メリット**:
- **最小ダウンタイム**: 継続レプリケーションで、切替時のダウンタイムは数分
- **テスト可能**: 本番影響なしでテストインスタンス起動
- **コスト効率**: 切替完了まではEBSストレージコストのみ
- **自動化**: MGNが移行プロセスをオーケストレート

**ユースケース**: オンプレミスのWindows/LinuxサーバーをそのままEC2に移行、VMwareやHyper-V上のVMをAWSへ。

---

## Part 3: DataSync — ファイル・オブジェクト大量転送

**DataSync**はオンプレミスとAWS間のデータ転送をマネージドで行うサービスです。

**主要機能**:
- **送信元**: NFS、SMB、HDFS、S3（他社）、Azure Blob
- **送信先**: S3、EFS、FSx
- **スケジューリング**: ワンショット、定期実行
- **帯域制御**: 業務時間帯は帯域制限、夜間は全帯域など
- **整合性検証**: 転送後に MD5 / SHA-256 で検証
- **増分転送**: 変更分のみ転送

**Snowballとの使い分け**:
- **DataSync**: ネットワーク経由、継続的同期、数TB〜数十TB
- **Snowball**: 物理デバイス、ワンショット大量移行、数十TB〜PB、ネットワーク帯域が不足の場合

---

## Part 4: Snow Family

**Snow Family**は物理デバイスでAWSに大量データを送る手段です。

| デバイス | 容量 | 特徴 |
|---------|------|------|
| **Snowcone** | 8 TB / 14 TB | 小型、エッジコンピューティング、持ち運び可能 |
| **Snowball Edge Storage Optimized** | 80 TB | ストレージ重視 |
| **Snowball Edge Compute Optimized** | 42 TB | コンピューティング重視（EC2実行可能） |
| **Snowmobile** | 最大100 PB | 50フィートのトラック、PB規模の移行 |

**利用フロー**:
1. AWSコンソールでデバイスを注文
2. データセンターにデバイス配送
3. オンプレミスでデータをコピー
4. AWSに返送
5. AWSがS3にインポート

**ユースケース**:
- 10 TB以上の移行でインターネット帯域が不足
- ネットワークコストが高額（Direct Connectやネットワーク費用より安価）
- ネットワーク接続不可の場所（船舶、遠隔地）
- Snowball Edge Compute Optimizedはエッジでの処理（災害現場、リモート拠点）

---

## Part 5: ハイブリッドクラウド戦略

**ハイブリッドクラウドサービス**:

**AWS Direct Connect**: 専用線でオンプレミス→AWS接続（テーマ4参照）

**AWS Site-to-Site VPN**: IPsec VPN、低コストDR

**AWS Outposts**: AWSのハードウェアをオンプレに設置、AWSサービスをオンプレで稼働
- 用途: 低レイテンシ要件、データ主権、規制対応

**AWS Wavelength**: 5G通信事業者のエッジにAWS Compute/Storageを配置、ウルトラ低レイテンシ

**Local Zones**: 主要都市にAWSリソースを配置（メトロエリア向け低レイテンシ）

**VMware Cloud on AWS**: VMware環境をAWSで実行、既存のVMware移行パスに最適

---

## Part 6: 大規模移行の進め方

**1. Assessment（評価）**:
- **Migration Evaluator**（旧TSO Logic）: オンプレミス環境の分析、AWS 上での TCO 推定
- **AWS Application Discovery Service**: サーバー情報（依存関係、構成、パフォーマンス）の自動収集

**2. Planning**:
- **Migration Hub**: アプリケーション単位でグループ化、依存関係管理
- 移行の波（Wave）を計画: 依存関係の少ないシステムから

**3. Execution**:
- MGN（サーバー）、DMS（DB）、DataSync（ファイル）の並行実行
- テストカットオーバー → 本番カットオーバー

**4. Optimization**:
- 移行後のコスト最適化、スケーリング、セキュリティ強化

---

## 練習問題

### 問題1

ある大手企業では、オンプレミスVMware 環境で 500 の Windowsサーバー を運用しています。これらを最小のダウンタイム・最小のコスト・最小のアプリケーション変更でAWSに移行したいと考えています。移行期間中はオンプレとAWSの両方で運用され、段階的に切り替えます。

最適な戦略・サービスはどれですか？

<details>
<summary>選択肢を見る</summary>

A. 各サーバーを手動でEC2に再構築する

B. AWS Application Migration Service（MGN）を使ってRehost（Lift-and-shift）戦略で移行。Replication Agentがオンプレサーバーに継続レプリケーション、カットオーバー時の最小ダウンタイムで切替可能

C. 全てSaaS（Office 365 など）に移行する

D. Lambdaで再実装する

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

AWS Application Migration Service（MGN）が正解です。

- **Lift-and-shift**: アプリケーション変更なしでRehost
- **継続レプリケーション**: ブロックレベルでのレプリケーションで最新データを維持
- **最小ダウンタイム**: カットオーバーは数分
- **テスト可能**: 本番影響なしでテスト起動
- **大規模対応**: 500台のサーバー移行を並行で自動化可能

- A: 手動再構築は時間・労力が膨大、500台には現実的でない
- C: SaaS置き換えは要件外（アプリケーション変更最小）
- D: Lambda再実装は大規模なアプリケーション書き換え、要件外

</details>

---

### 問題2

ある放送局では、オンプレミスに 500 TBの動画アーカイブがあり、これを全てS3に移行したいと考えています。以下の制約があります。

1. オンプレのインターネット帯域は 1 Gbps しかない
2. 帯域を業務トラフィックで常時使用するため、移行に全帯域を割り振れない
3. 移行期間を1ヶ月以内に収めたい

計算: 500 TB ÷ 1 Gbps = 約46日（帯域フル使用でも）

最適な移行手段はどれですか？

<details>
<summary>選択肢を見る</summary>

A. DataSyncで継続的にS3に同期する

B. AWS Snowball Edge Storage Optimized（80 TB／台）を複数台利用して物理配送する。ネットワーク経由より大幅に高速・低コスト

C. Direct Connectを敷設する

D. S3 CLIで手動アップロードする

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

AWS Snowball Edge Storage Optimizedが正解です。

- **ネットワーク回避**: インターネット帯域を使わないため業務影響なし
- **容量**: 1台80 TB、500 TBなら7台並行で対応
- **期間**: デバイス到着 → データコピー → 返送 → AWS取り込みで1〜2週間
- **コスト**: データ転送料金（$0.09/GB）を回避、デバイス1台数百ドル

- A: DataSyncもネットワーク経由で、1 Gbps全帯域でも46日、部分帯域ではさらに長期化
- C: Direct Connect敷設は数週間〜数ヶ月かかり、工事・月額費用も大きい
- D: 手動アップロードは非現実的

</details>

---

### 問題3

ある企業では、オンプレミスのNFS上にある300 TBの業務ファイルをAWS Elastic File System（EFS）に継続的に同期したいと考えています。ファイルは日々更新されており、増分の差分同期が必要です。変更されたファイルだけを効率的に転送するサービスを使いたいです。

最適な構成はどれですか？

<details>
<summary>選択肢を見る</summary>

A. Snowballで初期コピー、その後は rsync コマンドで手動同期

B. AWS DataSyncを使い、オンプレミスのNFSからEFSへの継続同期を設定。変更ファイルのみの増分転送、整合性検証（チェックサム）、スケジュール実行対応

C. Storage Gateway File Gatewayで同期

D. CloudEndure Migrationで同期

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

AWS DataSyncが正解です。

- **増分転送**: 変更されたファイルのみ転送、効率的な継続同期
- **整合性検証**: 転送後にMD5/SHA-256で検証
- **NFS → EFS**: DataSyncはこの組み合わせをネイティブサポート
- **スケジュール実行**: 毎日・毎時のスケジュール可能
- **帯域制御**: 業務時間帯の帯域制限設定

- A: rsyncの手動運用は管理負荷が高く、大規模環境には向かない
- C: Storage GatewayのFile GatewayはAWS S3への同期で、EFSではない
- D: CloudEndureはサーバー移行用（現MGN）で、ファイル同期用ではない

</details>

---

### 問題4

ある金融機関では、規制要件により「顧客データは日本国内の自社データセンター内に物理的に保持する」必要があります。しかし、AWSサービス（Lambda、S3など）も活用したいです。

AWSのサービスを、自社データセンター内に配置して利用する方法はどれですか？

<details>
<summary>選択肢を見る</summary>

A. 自社データセンターからDirect Connectで東京リージョンに接続する

B. AWS Outpostsを自社データセンターに設置する。AWSのハードウェア（ラック）をオンプレに設置して、EC2/EBS/S3/RDS/ECS/EKSなどのAWSサービスをオンプレで実行。データは物理的にオンプレに留まり、AWSコンソールから統合管理

C. VMware Cloud on AWS

D. AWS Snowball Edge Compute Optimized

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

AWS Outpostsが正解です。

- **オンプレミスのAWS**: AWSがハードウェアラックを顧客のデータセンターに納入・設置
- **AWSサービスネイティブ**: EC2、EBS、S3 on Outposts、RDS、ECS、EKSをオンプレで使用
- **データ主権**: 顧客データは物理的にオンプレに留まる
- **統合管理**: AWSコンソール・APIから通常のAWSサービスと同様に管理
- **ユースケース**: 低レイテンシ要件、データ主権、規制対応

- A: Direct Connect接続ではデータはAWSリージョンに保存され、物理要件を満たさない
- C: VMware Cloud on AWSはAWSリージョン内で動作するため、物理的にオンプレではない
- D: Snowball Edgeは一時的なエッジコンピューティング向けで、継続運用には向かない

</details>

---

### 問題5

ある自動車メーカーでは、工場からの低レイテンシリアルタイム処理が必要で、オンプレミス以下の要件を持っています。

1. 工場内機械からのデータを5ミリ秒以下のレイテンシで処理
2. 5G通信事業者のネットワークエッジで処理
3. 広範囲の工場を持つため、通信事業者のエッジ配置を活用したい

最適なAWSサービスはどれですか？

<details>
<summary>選択肢を見る</summary>

A. AWS Outpostsを各工場に設置

B. AWS Wavelengthを使う。通信事業者（KDDI、Verizonなど）の5Gネットワークエッジに配置され、デバイスから5Gネットワーク経由でAWSサービスに極低レイテンシ（10ms未満）でアクセス可能

C. AWS Local Zonesを使う

D. AWS IoT Greengrassを各工場に設置

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

AWS Wavelengthが正解です。

- **5Gエッジ**: 通信事業者の5Gネットワークエッジ（メトロエリア）にAWS Compute/Storageを配置
- **低レイテンシ**: 10ms未満のレイテンシで、5G端末（工場の機械、車両等）からAWSサービスにアクセス
- **5G統合**: 5Gデバイスが直接WavelengthゾーンのAWSサービスに到達
- **ユースケース**: 自動運転、工場IoT、AR/VR、コネクテッドカー

- A: Outpostsは各工場にハードウェア設置でコストが高く、メトロエリア共有には不適
- C: Local Zonesはメトロエリアの低レイテンシ向けだがInstances/App向けで、5Gエッジネットワークの統合はない
- D: Greengrassはデバイス自身でのLambda実行で、5Gネットワークエッジの活用ではない

</details>

---

### 問題6

ある企業では、既存の大規模VMware環境（数百のVM）をAWSに移行する計画です。以下の要件があります。

1. 既存のVMwareスキル、運用プロセス、ツールをそのまま使いたい
2. VMware vCenter、vSAN、NSXなどのVMwareエコシステムをAWS上で使いたい
3. 移行は段階的に、ハイブリッドで両方運用しながら進めたい

最適な構成はどれですか？

<details>
<summary>選択肢を見る</summary>

A. MGNで全VMをEC2に移行する

B. VMware Cloud on AWSを使う。VMware vSphere環境をAWS上で実行でき、既存のVMwareスキル・ツール・プロセスがそのまま使える。オンプレvCenterとAWS上のvCenterを統合管理し、ハイブリッド運用可能

C. 全VMをECSコンテナに再パッケージする

D. AWS Outpostsで VMware を動かす

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

VMware Cloud on AWSが正解です。

- **VMwareエコシステム**: vSphere、vSAN、NSXをAWSインフラ上でネイティブ実行
- **既存スキル活用**: VMware vCenterからの管理、既存のスクリプト・運用プロセスがそのまま使える
- **HCX**: VMware HCXでオンプレvCenterからAWS上のvCenterへ VMを無停止マイグレーション可能
- **ハイブリッド**: オンプレとAWS上のVMware環境を統合管理、段階的移行

- A: MGNはAMI変換を伴い、既存のVMwareエコシステムの継続利用はできない
- C: コンテナ化は大規模な再設計が必要で、既存の運用プロセスを活かせない
- D: Outpostsでは VMware Cloud 相当のサービスは提供されない（通常のAWSサービスのみ）

</details>
