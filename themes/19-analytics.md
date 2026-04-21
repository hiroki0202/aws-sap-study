# テーマ19: データ分析基盤

> 🟡 所要日数: 2日 | 座学 → 問題演習

---

## 座学

## Part 1: SAAからの差分 — 分析サービスの全体像

SAAではAthena、Redshift、EMRの存在を軽く学びました。SAPでは**各サービスの使い分け**と**データレイク設計**が問われます。

AWSの分析サービスを役割別に整理します。

| サービス | 役割 |
|---------|------|
| **S3 + Lake Formation** | データレイク（構造化/非構造化データの保管・カタログ・権限管理） |
| **Glue** | ETL（データ抽出・変換・ロード）、Data Catalog |
| **Athena** | S3上のデータに対するSQLアドホッククエリ |
| **Redshift** | データウェアハウス（大規模分析） |
| **Redshift Spectrum** | RedshiftからS3のデータに直接クエリ |
| **EMR** | Hadoop/Sparkベースの分散処理 |
| **QuickSight** | BIダッシュボード |
| **OpenSearch** | ログ・全文検索分析 |

---

## Part 2: データレイク設計 — S3 + Glue + Athena

**データレイクの基本**: あらゆる形式のデータ（CSV、JSON、Parquet、画像、動画）をS3に保管し、必要に応じて様々なツールで処理する。

**AWS Glue**の役割:
- **Crawler**: S3のデータをスキャンしてスキーマを推論、Data Catalogに登録
- **Data Catalog**: メタデータ（テーブル、スキーマ、パーティション）の中央リポジトリ
- **ETL Jobs**: Sparkベースでデータ変換（CSV→Parquet、クレンジング、結合）
- **Workflows**: ETLパイプラインのオーケストレーション

**Athena**の役割:
- S3のデータに**SQLで直接クエリ**
- サーバーレス（インフラ管理なし）
- 料金: スキャンしたデータ量 $5/TB
- コスト最適化: **Parquet形式 + パーティション + 圧縮**でスキャン量を削減

**Parquetの重要性**: 列指向フォーマットで、必要な列だけ読み込む。CSVと比較して10倍以上の圧縮率。Athena/Redshift Spectrumの料金を劇的に削減。

---

## Part 3: Redshift — データウェアハウス

**Redshift**はペタバイト規模のデータウェアハウスで、構造化された分析データに対するSQLクエリを高速実行します。

**主要な機能**:

**Redshift Spectrum**: Redshiftからクラスター外のS3データに直接クエリ。データのロード不要で、ホット/コールドデータの階層化が可能。

**RA3 ノード**: コンピュートとストレージを分離したアーキテクチャ。ストレージは自動スケール、コンピュートのみ料金発生。従来のDSxに対してストレージコスト削減。

**AQUA（Advanced Query Accelerator）**: ハードウェアアクセラレーション層で集計クエリを最大10倍高速化。

**Concurrency Scaling**: クエリが多数同時実行された場合、一時的に追加クラスタを起動して並列処理。キューイング遅延なし。

**Redshift Serverless**: キャパシティ管理不要のサーバーレス版。予測困難なワークロードに適する。

**Data Sharing**: 他のRedshiftクラスターから読み取り専用アクセスを共有。データコピー不要。

---

## Part 4: EMR — 分散処理フレームワーク

**EMR**はHadoop、Spark、Presto、Hive、HBaseなどのオープンソース分散処理フレームワークをマネージドで提供します。

**EMRのユースケース**:
- 機械学習の特徴量エンジニアリング（Spark）
- 大規模ログ集計（Hadoop MapReduce）
- リアルタイムストリーム処理（Flink）
- ゲノム解析、科学計算

**EMRのクラスター種類**:
- **Transient Cluster**: ジョブ完了後に自動削除（コスト最適化）
- **Long-Running Cluster**: 常時稼働、連続処理向け
- **EMR on EKS**: KubernetesでEMRを実行
- **EMR Serverless**: サーバーレス版（新）

**EMRとGlueの使い分け**:
- Glue: マネージドETL、カスタム処理が少ないケース、サーバーレス運用
- EMR: 複雑な分散処理、既存のHadoop/Sparkスキル活用、カスタマイズが必要

---

## Part 5: Lake Formation — データレイクの権限管理

**Lake Formation**はデータレイクのセキュリティ・ガバナンスを提供するサービスです。

**機能**:
- **きめ細かいアクセス制御**: テーブル/列/行レベルの権限管理
- **データカタログ**: Glue Data Catalogを基盤としたメタデータ管理
- **Tag-Based Access Control（TBAC）**: タグでアクセス制御を統一管理
- **Cross-Account Sharing**: アカウント間でのデータ共有

**従来（S3バケットポリシー）との違い**: S3バケットポリシーはオブジェクト全体へのアクセス制御しかできませんが、Lake Formationは「このテーブルのこの列だけ」「この行だけ」といったきめ細かい制御が可能。BIツールがAthenaやRedshift Spectrum経由でアクセスする際に権限が適用されます。

---

## Part 6: OpenSearch — ログ・検索分析

**Amazon OpenSearch Service**（旧Elasticsearch Service）は、ログ分析や全文検索用のマネージドサービスです。

**ユースケース**:
- アプリケーションログのリアルタイム検索・可視化
- ユーザー行動分析
- セキュリティログ分析（SIEM）
- Eコマースの商品検索

**データ投入パターン**:
1. Kinesis Data Firehose → OpenSearch
2. Logstash / Fluentd → OpenSearch
3. CloudWatch Logs Subscription → OpenSearch

**OpenSearch Serverless**: キャパシティ管理不要の新オプション。予測困難なワークロードや間欠的な検索負荷に適合。

---

## 練習問題

### 問題1

ある製造業の企業では、IoTデバイスから毎日数TBのセンサーデータがS3に保管されています。データは半構造化JSONで、アナリストが月次で集計分析を行いたいと考えています。

要件：
1. アナリストが SQL で簡単にクエリできる
2. サーバー管理を不要にしたい
3. クエリごとにスキャンするデータ量を最小化してコスト削減

最適な構成はどれですか？

<details>
<summary>選択肢を見る</summary>

A. Redshiftクラスターを立ち上げてデータをロードし、RedshiftでSQLクエリを実行

B. Glue Crawlerでデータをカタログ化し、ETLジョブでJSONをParquet形式 + パーティション（年/月/日）に変換。Athenaで SQLクエリ。Parquetとパーティションでスキャン量を削減

C. EMRクラスターでHiveを起動しクエリする

D. RDSにデータをロードしてクエリする

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

Glue（ETL）+ Athenaの組み合わせが正解です。

- **Glue Crawler**: S3のJSONデータを自動スキャンしスキーマを推論、Data Catalogに登録
- **ETL → Parquet + パーティション**: 列指向フォーマットで圧縮率10倍、パーティションで必要な範囲のみスキャン
- **Athena**: サーバーレスでSQL実行、スキャンデータ量のみ課金（$5/TB）
- **コスト**: 10 TBのCSVをクエリするとAthena料金$50/クエリ、ParquetとPartitioningで数GBまで減らせば$数ドル以下に

- A: Redshiftはデータウェアハウスとして強力ですが、インフラ管理が必要で、半構造化JSONデータの月次アドホック分析にはオーバースペック
- C: EMRはHadoopクラスターの管理が必要で、サーバー管理不要の要件に反する
- D: RDSは大規模分析には適さず、数TBスケールでは性能・コスト両面で問題

</details>

---

### 問題2

ある大手小売企業では、過去5年分の売上データ（500 TB）をRedshiftで分析しています。最近、新たに以下の要件が出ました。

1. 過去1年のデータは頻繁にクエリするためRedshiftクラスター内に保持
2. 1〜5年前のデータはまれにしかクエリしないがSQLで検索できるようにしたい
3. S3にデータをコピーせず、既存のRedshiftからのクエリで両方のデータを結合したい

最適な構成はどれですか？

<details>
<summary>選択肢を見る</summary>

A. 5年分全てをRedshiftに保持し続ける

B. 古いデータをRedshiftからエクスポートしS3に移動。Redshift Spectrumを使って、Redshiftから直接S3上のデータに対してSQLクエリを実行。ホットデータ（1年以内）とコールドデータ（1〜5年）をJOINで結合可能

C. 5年分のデータをEMRに移行する

D. 5年分をAthenaで全て処理する

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

Redshift Spectrumが正解です。

- **ホットデータ**: 頻繁アクセスの1年分はRedshiftクラスター内で高速アクセス
- **コールドデータ**: 古いデータはS3に配置し、Spectrumで直接クエリ
- **統合クエリ**: Redshift内のテーブルとS3のデータをJOINで結合可能
- **コスト削減**: Redshiftクラスターを小さくでき、コールドデータはS3の安価なストレージに

- A: 500 TB全てをRedshiftに保持するとストレージコストが大きい
- C: EMRはHadoop/Sparkの分散処理で、SQLワークロードにはRedshiftが適する
- D: Athena単独ではRedshiftの最適化（分散キー、ソートキー等）の利点を失う

</details>

---

### 問題3

ある金融機関では、S3のデータレイクに複数部門のデータを集約しています。以下の要件があります。

1. 営業部は`/sales/` 配下に読み取りアクセス（ただし個人情報カラムはマスク）
2. 分析部は全テーブルに読み取りアクセス（PIIは見える）
3. 監査部は全データに読み取りアクセス、かつアクセス監査ログが必要
4. BIツール（QuickSight、Tableau）からAthenaでアクセスする際にも権限が適用される

S3バケットポリシーで管理すると粒度が粗すぎるため、最適な方法はどれですか？

<details>
<summary>選択肢を見る</summary>

A. Lake Formationで列レベル・行レベルの権限を定義し、部門ごとにPermission Setを作成する。Athenaクエリ実行時に権限が自動適用され、マスキング・フィルタリングが働く

B. S3バケットポリシーで各部門にPrefix単位の権限を付与する

C. Glue Data Catalogに直接権限を設定する

D. 各部門向けに別々のS3バケットを用意してデータをコピーする

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: A**

Lake Formationが正解です。データレイクのきめ細かいアクセス制御に特化したサービスです。

- **列レベル権限**: 特定のカラム（個人情報）を特定のロールに非表示
- **行レベル権限**: 特定のレコードのみ見せる（部門、地域で絞り込み）
- **統合アクセス**: Athena、Redshift Spectrum、EMR、QuickSightがLake Formationの権限を尊重
- **監査**: CloudTrailでアクセスログを自動記録

- B: S3バケットポリシーはオブジェクト全体への権限しか設定できず、列・行レベルの制御は不可
- C: Glue Data Catalogに直接権限設定しても、列・行レベルの細かい制御はLake Formation の機能
- D: データコピーはコスト・整合性の両面で非効率

</details>

---

### 問題4

ある企業では、アプリケーションログ（1日あたり数TB）を CloudWatch Logsに保管していますが、リアルタイム検索とダッシュボード可視化が必要になってきました。運用チームがトラブルシューティングで特定のエラーパターンを高速検索したいという要件です。

最適な構成はどれですか？

<details>
<summary>選択肢を見る</summary>

A. CloudWatch Logs Insightsで直接検索する

B. CloudWatch LogsからSubscription FilterでKinesis Data Firehoseに流し、OpenSearchに投入。OpenSearch Dashboardsで可視化・全文検索を実現。リアルタイム（数秒遅延）でログが検索可能

C. S3にアーカイブしAthenaで検索する

D. DynamoDBにログを保存する

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

CloudWatch Logs → Firehose → OpenSearchが正解です。

- **リアルタイム性**: Kinesis Data Firehoseで数秒遅延でOpenSearchに投入
- **全文検索**: OpenSearchは全文検索と構造化検索の両方に強い
- **可視化**: OpenSearch Dashboards（Kibana）で柔軟なダッシュボード
- **スケール**: 大量のログ（数TB/日）にも対応

- A: CloudWatch Logs Insightsも検索可能ですが、全文検索性能・可視化機能でOpenSearchに劣る
- C: Athena + S3は集計分析には良いが、リアルタイム検索には遅延がある
- D: DynamoDBはログ検索用途には向かない

</details>

---

### 問題5

ある企業のデータチームでは、複雑な機械学習パイプラインをSparkで実装しています。カスタムのPythonライブラリと複数のSpark設定が必要で、クラスターのサイジングも細かく調整したいと考えています。

最適な実行環境はどれですか？

<details>
<summary>選択肢を見る</summary>

A. Lambda関数でSparkコードを実行する

B. EMR（Transient Cluster）でSparkジョブを実行する。ジョブ完了後にクラスターを自動削除してコスト最適化。カスタムライブラリ・Spark設定・クラスターサイジングが自由にコントロールできる

C. Glue Sparkジョブで実行する

D. Redshiftのストアドプロシージャで実行する

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

EMR Transient Clusterが正解です。

- **カスタム環境**: カスタムライブラリインストール、Spark設定の調整が自由
- **クラスターサイジング**: 必要なノード数・インスタンスタイプを指定可能
- **コスト最適化**: Transient Clusterでジョブ完了後に自動削除、起動中のみ課金
- **Spot活用**: Core NodesにSpotを使って大幅コスト削減

- A: LambdaはSparkクラスターを起動できない（最大メモリ10GB、最大実行時間15分）
- C: Glue Sparkは管理簡単ですが、カスタムライブラリ・Spark設定の自由度はEMRに劣る
- D: RedshiftはSQLベースで、Spark処理は実行できない

</details>

---

### 問題6

あるメディア企業では、50を超えるアカウントで運用するRedshiftクラスターの運用に課題があります。あるアカウントのアナリストが他のアカウントのデータを使って分析したいケースが多発しており、現在はデータを都度コピーしています。

コピーは時間がかかり、データの一貫性も損なわれがちです。データをコピーせずにクロスアカウントで読み取りアクセスできる方法が必要です。

最適な方法はどれですか？

<details>
<summary>選択肢を見る</summary>

A. S3にRedshiftからエクスポートし、他のアカウントにコピーする

B. Redshift Data Sharingを使う。データを物理コピーせずに、他のRedshiftクラスター（別アカウント）から読み取り専用でアクセスできる。データの一貫性が保たれる

C. 全アカウントを1つに統合する

D. DMSで継続的にレプリケーションを設定する

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

Redshift Data Sharingが正解です。

- **データコピー不要**: プロデューサークラスターのデータに、コンシューマークラスターから直接クエリ可能
- **一貫性**: リアルタイムで最新データにアクセス、コピーの遅延なし
- **クロスアカウント対応**: 別のAWSアカウントのRedshiftクラスターに共有可能
- **Organizations統合**: 組織内の複数アカウントで共有が簡単

- A: コピー運用は時間・一貫性の問題が残る
- C: アカウント統合は大規模変更が必要
- D: DMSはマルチソース移行向けで、クロスアカウントの単純な共有にはオーバースペック

</details>
