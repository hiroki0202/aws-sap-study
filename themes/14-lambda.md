# テーマ14: Lambda統合パターン

> 🟡 所要日数: 2日 | 座学 → 問題演習

---

## 座学

## Part 1: SAAからの差分 — Lambdaで深く問われる領域

SAAでLambdaの基本（イベントドリブン、無制限スケール、従量課金、基本的なトリガー）は学びました。SAPでは運用・性能・統合の観点で深く問われます。

**コールドスタートの軽減**（Provisioned Concurrency、SnapStart）、**同時実行数の制御**（Reserved Concurrency、アカウント全体の上限）、**VPC統合**（Hyperplane ENI）、**非同期呼び出しと Destinations**、**Lambda Extensions と Layers**、**Function URLs と API Gateway の使い分け**。

---

## Part 2: コールドスタート問題とその解決

Lambdaはリクエストが来てから関数コンテナを起動します。初回呼び出しやアイドル後の起動時、**コールドスタート**（関数のコンテナ初期化、ランタイム起動、コードロード）が数百ミリ秒〜数秒発生します。Javaや.NETなど重量級ランタイムでは数秒かかることもあります。

ユーザー体験が重要なAPIでは、コールドスタートが顕在化するのは問題です。SAPレベルでは以下の対策を理解している必要があります。

**Provisioned Concurrency（プロビジョニング済み同時実行）**:
指定した数のLambdaコンテナを事前に温かい状態で維持します。このコンテナはリクエストを即座に処理でき、コールドスタートが発生しません。Auto Scalingで時間帯に応じて数を自動調整することも可能です。

**SnapStart（Java向け）**:
Java 11/17/21のランタイムで利用可能な機能で、初期化済みのコンテナのスナップショットをキャッシュし、リクエスト時にスナップショットから即座に復元します。追加コストなしでコールドスタートを**最大90%削減**できます。Javaの重いクラスローディング問題を解決する決定打です。

**ランタイムの選択**:
- Python、Node.jsはランタイムが軽く、Javaほどコールドスタートが深刻でない
- Rust、Goは**カスタムランタイム**として実装可能で非常に高速
- 重量級フレームワーク（Spring Bootなど）を避け、軽量なマイクロフレームワークを使う

---

## Part 3: 同時実行数とスケーリング制御

Lambdaのデフォルトでは、アカウント全体で **1,000同時実行** の上限があります。複数の関数で共有されるため、あるLambdaが暴走すると他のLambdaまで実行できなくなる可能性があります。

**Reserved Concurrency（予約同時実行）**:
特定の関数に「この数まで同時実行を保証し、かつこの数を超えて実行させない」という上限を設定します。

ユースケース：
- 重要な関数に最低限のキャパシティを保証
- 暴走リスクのある関数の上限を設定して他の関数への影響を防ぐ
- RDSに接続するLambdaを制限（接続数制限対策、ただしRDS Proxyが推奨）

**Provisioned Concurrency（プロビジョニング済み同時実行）**:
Reserved Concurrencyとは別の概念で、「事前に温めたコンテナを指定数維持する」機能です（Part 2参照）。両方を同時に使うことも可能（Reserved範囲内でProvisioned分が温かく、超過分はオンデマンド）。

**スケーリングの挙動**:
- 最初の1分間に1,000インスタンスまでバースト起動（リージョンによって500/1,000/3,000）
- その後は1分あたり500インスタンスずつ増加
- 急激なスパイクではThrottlingが発生する可能性あり

---

## Part 4: LambdaのVPC統合とネットワーキング

初期のLambda VPC統合は、関数起動のたびにENIを作成しており、コールドスタートに**10秒以上**追加されることがありました。現在の**Hyperplane ENI**方式では、複数のLambdaインスタンスがENIを共有し、コールドスタートへの影響はほぼゼロです。

**VPC接続が必要なケース**:
- RDS/Auroraへの接続
- ElastiCacheへの接続
- 社内APIやオンプレミスリソースへの接続

**VPC Lambdaでインターネットアクセスが必要な場合**:
Lambda関数はVPC接続するとデフォルトでインターネットに出られません。NAT Gatewayを経由させるか、VPCエンドポイントで必要なAWSサービスに接続します。VPCエンドポイントの利用はNAT Gatewayコスト削減になります。

**ENI枯渇問題**: 大量のLambdaがVPC接続する場合、Hyperplane ENIでもサブネットのIPアドレスが枯渇する可能性があります。サブネットのCIDRを広めに取る、関数を複数サブネットに分散配置する設計が重要です。

---

## Part 5: 非同期呼び出しと Destinations

Lambdaの呼び出しパターンには3種類あります。

1. **同期呼び出し**: 呼び出し元が結果を待つ（API Gateway、同期Lambda呼び出し）
2. **非同期呼び出し**: 呼び出し後すぐに戻る、Lambda内部のキューで処理（S3イベント、SNS、EventBridgeなど）
3. **ストリームベース**: Lambdaがポーリング（Kinesis、DynamoDB Streams、Kafka、MSK）

**非同期呼び出しのリトライとDLQ**:
非同期Lambdaは失敗時に自動で最大2回リトライ（合計3回実行）し、それでも失敗するとエラーとなります。失敗したイベントを逃さないために**Dead Letter Queue（DLQ）**に送信できます。DLQにはSQSまたはSNSを指定できます。

**Lambda Destinations**（DLQの進化版）:
成功時・失敗時の両方で、処理結果を別のサービスに送信できる機能です。

- **On Success**: SQS / SNS / EventBridge / 別のLambda
- **On Failure**: SQS / SNS / EventBridge / 別のLambda

DLQはエラーのみでしたが、Destinationsは成功イベントも扱えて「成功後のワークフロー連鎖」を実装できます。

---

## Part 6: Lambda Layers と Extensions

**Lambda Layers**は、複数のLambda関数で共通のライブラリ・依存関係を共有する仕組みです。

- 重い依存（NumPy、Pandas、boto3など）を1つのLayerに集約
- 最大5つのLayerを1つの関数にアタッチ可能
- 関数コードのサイズを節約し、デプロイ速度も向上

**Lambda Extensions**は、Lambda実行環境内で**並行して動作するプロセス**（ログ収集、監視、シークレット取得など）を実行する仕組みです。

ユースケース：
- Datadog、New Relicなどの監視エージェント
- Secrets Managerからのキャッシング取得（APIコール削減）
- カスタムログ収集

---

## Part 7: Function URLs vs API Gateway

**Function URLs**は、Lambda関数に直接HTTPSエンドポイントを割り当てる機能です。API Gatewayなしで関数を公開できます。

| 項目 | Function URLs | API Gateway |
|------|--------------|-------------|
| 用途 | 単一関数の公開 | 複数APIの統合、本格的なAPI管理 |
| 認証 | IAMまたはパブリック | IAM / Cognito / カスタム / Lambdaオーソライザー |
| スロットリング | Lambda同時実行数のみ | API Key、Usage Plan、細かい制御 |
| 使用量管理 | なし | ✅ |
| キャッシュ | なし | ✅ |
| WAF統合 | なし | ✅ |
| 料金 | Lambda料金のみ | API Gateway料金 + Lambda料金 |

**使い分け**: シンプルなWebhookや内部ツールはFunction URLs、パブリックAPIやAPI管理が必要なケースはAPI Gateway。

---

## 練習問題

### 問題1

ある保険会社のモバイルアプリは、Javaで書かれたLambda関数（Spring Boot）をAPI Gateway経由で呼び出しています。朝の営業開始時の最初のリクエストから数回は、レスポンスが**5〜7秒**かかるという苦情が顧客から出ています。その後のリクエストは100 ms程度で正常に返っています。

開発チームの調査で、これはJava Lambdaのコールドスタート（初期化・クラスロード）に時間がかかることが原因と判明しました。追加コストを最小限に抑えつつ、コールドスタートの影響を軽減したいと考えています。

最適な対策はどれですか？

<details>
<summary>選択肢を見る</summary>

A. Lambda関数をNode.jsに書き換えて軽量化する

B. Java Lambda関数でSnapStartを有効化し、初期化済み状態のスナップショットから高速起動できるようにする

C. API Gateway にCloudFrontを挟んで、レスポンスをキャッシュする

D. 定期的にLambdaを呼び出すEventBridgeルールを作り、コンテナが冷えないよう「ウォーム」状態を維持する

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

Lambda SnapStart が正解です。SnapStartはJava 11/17/21で利用可能な機能で、関数初期化後のスナップショットを保持し、リクエスト時にスナップショットから即座に復元することでコールドスタートを**最大90%削減**します。追加コストなしで有効化できます。Spring Bootのような重量級フレームワークでも大幅に改善されます。

- A: Node.jsへの書き換えは根本解決ですが、アプリケーションの大規模改修が必要で「追加コストを最小限」の要件に反します
- C: CloudFrontのキャッシュは読み取りAPIで有効ですが、書き込み系APIやパーソナライズされたレスポンスには適用できません
- D: ウォームアップ手法は関数を呼び続けるためコストが増加します。またスケール時に新しいコンテナが起動した場合はコールドスタートが発生し、完全な解決にはなりません。Provisioned ConcurrencyやSnapStartの方が信頼性が高いです

</details>

---

### 問題2

あるスタートアップの本番環境では、AWSアカウント全体でLambdaの同時実行数が上限（1,000）に達し、重要な決済処理のLambdaが起動できずエラーが返る問題が発生しました。原因は、バックグラウンドの分析処理Lambdaが大量のSQSメッセージをトリガーに暴走的に起動し、同時実行数を使い切っていたためです。

決済処理は最重要機能であり、最低でも300の同時実行数を常に確保したいと考えています。分析処理の同時実行は200までに制限し、他の処理への影響を防ぎたいです。

適切な設定はどれですか？

<details>
<summary>選択肢を見る</summary>

A. 決済処理Lambdaに Reserved Concurrency = 300 を設定し、分析処理Lambdaに Reserved Concurrency = 200 を設定する。これで決済は300のキャパシティが保証され、分析は200までしか実行されず、残りの500は他の関数で共有される

B. 決済処理にProvisioned Concurrency = 300を設定し、分析処理はそのままにする

C. AWS Support に連絡してアカウントの同時実行上限を10,000に引き上げる

D. 分析処理LambdaをStep Functionsに書き換え、Step Functionsの並列度で制御する

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: A**

Reserved Concurrencyが正解です。

- **決済処理の保証**: Reserved Concurrency = 300を設定すると、その300のキャパシティは決済処理のみに確保され、他の関数が使い切っていても決済は起動できる
- **分析処理の制限**: Reserved Concurrency = 200を設定すると、分析処理は200を超えて起動できない（暴走防止）
- **他の関数**: 1,000 - 300 - 200 = 500が「予約されていないプール」として他の関数で共有される

- B: Provisioned Concurrencyは「事前に温めたコンテナ」を提供するもので、同時実行数の予約・制限とは別の機能です。コールドスタート対策には有効ですが、本件の要件（上限確保・制限）には向きません
- C: 同時実行上限を引き上げても分析処理の暴走は止まらず、結果として決済処理を脅かし続けます
- D: Step Functionsへの書き換えは大規模変更で、より簡単なReserved Concurrencyで解決可能です

</details>

---

### 問題3

あるECサイトでは、注文処理をLambda関数で実装しています。注文が失敗した場合（在庫エラー、決済失敗など）には、失敗イベントを別の処理ラインに送って人間による確認・再処理できるようにしたいです。また、成功した注文は別のSQSキューに送り、出荷処理ラインに流したいと考えています。

現在の実装ではLambda関数内で結果を判定してSQSに書き込む処理を自前で書いていますが、エラーハンドリングの抜け漏れが発生しています。また、Lambdaのランタイム例外（メモリ不足、タイムアウトなど）で成功・失敗の判定自体ができずイベントが失われるケースもあります。

この問題を改善する最適な機能はどれですか？

<details>
<summary>選択肢を見る</summary>

A. Lambda Destinationsを有効化し、On Successに成功時のSQSキュー、On Failureに失敗時のSQSキューを設定する。成功・失敗はLambdaランタイムが自動判定し、ランタイム例外時も確実に失敗ルートが実行される

B. Dead Letter Queue（DLQ）を設定する

C. Step Functionsでワークフローを作成し、成功・失敗の分岐を記述する

D. Lambda関数内でtry-catchを厳密に書き、SQSへの書き込みを確実に行う

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: A**

Lambda Destinationsが正解です。Destinationsは成功と失敗の両方で結果を別のサービスに自動送信する機能で、Dead Letter Queueの上位互換です。

- **On Success**: 成功時の送信先（SQS/SNS/EventBridge/Lambda）
- **On Failure**: 失敗時の送信先（ランタイムエラー、タイムアウト、メモリ不足でも確実に発火）
- **非同期呼び出し限定**: 注文処理を非同期呼び出しで行う場合に使用
- 関数コード内のロジックなしで、プラットフォームが確実にイベント連鎖を保証

- B: DLQは失敗時のみ対応で、成功ルートには使えません。Destinationsの方が機能的に広い
- C: Step Functionsは有効ですが、大規模な書き換えが必要です。Destinationsは既存のLambdaに対して外部設定のみで機能追加できます
- D: Lambda関数コード内のtry-catchだけでは、ランタイム自体の例外（OOM、タイムアウト、コンテナクラッシュ）では catch ブロックに到達しません

</details>

---

### 問題4

ある企業のLambda関数（Node.js）は、AWS Secrets Managerから都度シークレットを取得しています。関数が高頻度で実行されるため、Secrets Manager APIの呼び出し料金と遅延が問題になってきました。

1リクエストあたり3回のSecrets Manager APIコールが発生しており、月額数万円のコストと、コールドスタート時のSecrets Manager呼び出しによる100〜200 msの追加レイテンシが顕在化しています。

シークレットの取得効率を改善する最適な方法はどれですか？

<details>
<summary>選択肢を見る</summary>

A. Lambda関数のコード内で取得したシークレットをグローバル変数にキャッシュし、コンテナの再利用時は再取得しない

B. AWS Parameter Store Lambda Extension（または Secrets Manager Extension）を有効化し、実行環境レベルでシークレットをキャッシュする。Secrets Managerへの呼び出し数が大幅に削減され、レイテンシも改善される

C. シークレットを環境変数に直接記載する

D. Lambda Provisioned Concurrencyを有効化して初期化コストを削減する

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

Lambda Extensions（Secrets Manager / Parameter Store Extension）が正解です。Extensionは Lambda 実行環境に並行プロセスとして動作し、シークレットを実行環境レベルでキャッシュします。

- Secrets Manager への API コール数が大幅に削減（最大で90%以上減）
- シークレット取得が localhost レベルのレイテンシ（数ミリ秒）に
- シークレットのローテーションも自動対応

- A: コード内のグローバル変数キャッシュも有効ですが、コンテナごとに独立して取得する必要があり、Extensionのような共有キャッシュには及びません。またシークレットローテーション時の更新ロジックも自作する必要があります
- C: シークレットを環境変数に直接記載するのはセキュリティ違反です（CloudFormationテンプレートに平文で残る、暗号化されない、ローテーション不可）
- D: Provisioned Concurrencyはコールドスタートの初期化コストは削減しますが、Secrets Manager APIコール自体は毎回発生します

</details>

---

### 問題5

あるSaaS企業では、Lambda関数（Python）からVPC内のRDS Auroraに接続する必要があります。開発環境でテストしたところ、以下の問題が発生しました。

1. 最初のLambda呼び出しで10秒以上の遅延が発生する（Lambda VPC統合の古い問題）
2. 同時実行数が増えると、ENI作成の遅延でさらに悪化する
3. VPC統合するとインターネットアクセスができなくなり、外部APIを呼べない

これらの問題を解決する最適な構成はどれですか？

<details>
<summary>選択肢を見る</summary>

A. 現代のLambda VPC統合はHyperplane ENIで高速化されており、10秒の遅延は発生しない。新しいLambdaランタイムを使用し、同じVPC内でNAT GatewayまたはVPCエンドポイント経由で外部APIへのアクセスを設定する

B. Lambda関数をVPC外で実行し、パブリックエンドポイントでAurora Data APIを使う

C. RDS Proxy経由でのみアクセスし、Lambda自体はVPC外に配置する

D. 全てのLambdaをEC2インスタンスに移行する

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: A**

Hyperplane ENIが正解です。Lambdaの初期のVPC統合では、関数起動のたびにENIを作成しコールドスタートに10秒以上の追加遅延がありました。現在は**Hyperplane ENI**（複数のLambdaインスタンスでENIを共有する方式）に切り替わっており、VPC統合によるコールドスタート増加はほぼゼロです。

外部インターネットアクセスについては、VPC内のNAT Gatewayで出口を確保するか、VPCエンドポイント（Interface Endpoint）で必要なAWSサービスに接続します。

- B: Aurora Data APIはAurora Serverless向けで、通常のプロビジョンドAuroraでは使えません
- C: Lambda自体をVPC外に配置するとAuroraへのプライベートアクセスができません。RDS Proxyを使う場合もLambdaはVPC内にある必要があります
- D: EC2への移行はLambdaのスケーラビリティ・運用簡素化のメリットを失います

</details>

---

### 問題6

あるスタートアップでは、GitHubから送られるWebhookイベント（push、PR openedなど）をLambda関数で処理しています。現在はAPI Gatewayの前段にカスタムオーソライザーを配置し、Lambdaで受信処理を行っています。

この構成を見直したところ、以下が分かりました。

1. 単一のLambda関数だけを公開しており、API Gatewayの高度な機能（複数API統合、使用量プラン、キャッシュなど）は一切使っていない
2. 認証はWebhook送信元の署名検証（HMAC）をLambda内で実装しており、API Gatewayの認証機能も使っていない
3. API Gatewayの料金が無視できない額になっている

運用をシンプルにし、コストを削減する最適な構成はどれですか？

<details>
<summary>選択肢を見る</summary>

A. Lambda Function URLsを有効化し、API Gatewayを廃止する。Function URLsはLambdaに直接HTTPSエンドポイントを割り当てる機能で、API Gatewayの料金が不要になる

B. API Gatewayを維持し、使用していない機能を無効化する

C. ALBをフロントに配置し、ALBのLambdaターゲットにルーティングする

D. CloudFrontでLambda関数を直接呼び出す構成にする

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: A**

Lambda Function URLsが正解です。Function URLsはLambda関数に直接HTTPSエンドポイント（例: `https://<id>.lambda-url.region.on.aws/`）を割り当てる機能で、API Gatewayなしで関数を公開できます。

- **料金**: Lambda料金のみ（API Gateway料金不要）
- **認証**: IAM または パブリック（現状のHMAC署名検証はLambda内で継続可能）
- **設定**: Lambda コンソールで有効化するだけ

本件は「API Gatewayの高度な機能を使っていない」「認証もLambda内で実装済み」というケースでFunction URLsが最適です。

- B: 使用していない機能の無効化は可能でも、API Gateway自体の料金（リクエスト課金）は継続して発生します
- C: ALBでもLambdaを呼び出せますが、ALBの固定月額料金（NW料金）が発生し、単一のLambda公開用途にはオーバースペックです
- D: CloudFrontは Lambda を直接呼び出せません（Lambda@Edge は特殊ケース）。CloudFront + Function URLs は可能ですが、本件の要件（シンプル化）には Function URLs 単独が最適

</details>
