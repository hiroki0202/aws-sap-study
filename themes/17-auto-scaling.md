# テーマ17: Auto Scaling 高度設計

> 🟡 所要日数: 2日 | 座学 → 問題演習

---

## 座学

## Part 1: SAAからの差分 — Auto Scalingで深く問われる領域

SAAでEC2 Auto Scaling Group（ASG）、基本的なTarget Tracking・Step Scaling・Scheduled Scalingは学びました。SAPでは以下を深堀りします。

**Warm Pools**（ウォームアップ済みインスタンスの保持）、**Predictive Scaling**（機械学習による予測スケーリング）、**ライフサイクルフック**、**Capacity Rebalancing**（Spot中断対応）、**Mixed Instances Policy**（複数インスタンスタイプ組み合わせ）、**Placement Groups**、**Application Auto Scaling**（ECS/DynamoDBなどの自動スケール）。

---

## Part 2: Auto Scalingのスケーリングポリシー

**Simple Scaling（非推奨）**:
- 1つの閾値で1段階のスケール操作
- 古い方式で、現在はStep/Target Trackingに置き換え推奨

**Step Scaling**:
- 複数のステップで段階的にスケール
- 例: CPU 50-70% → +2台、70-90% → +5台、90%+ → +10台
- 細かい制御が可能

**Target Tracking Scaling**:
- 目標メトリクス値（例: 平均CPU 50%）を維持するよう自動調整
- 内部的にCloudWatchアラームを2つ作成（スケールアウト用とスケールイン用）
- 設定が最も簡単で、多くの場合に推奨

**Scheduled Scaling**:
- 時間帯ごとにキャパシティを変更（毎朝9時に5台、毎晩22時に2台）
- 予測可能なトラフィックパターン向け

**Predictive Scaling**:
- 機械学習で過去のトラフィックから24時間先を予測し、事前にスケール
- CloudWatchのメトリクス履歴14日以上が必要
- 始動に時間がかかるインスタンス（大型・起動遅い）に特に有効

---

## Part 3: Warm Pools — 起動時間の短縮

EC2インスタンスの起動には時間がかかります（特にアプリケーションの初期化、ウォームアップが必要な場合）。スケールアウト時に「新しいインスタンスが立ち上がってトラフィックを受けられるまで数分」かかると、ピーク対応が遅れます。

**Warm Pool**は、ASGの「待機中」のインスタンスプールです。インスタンスは**Stopped状態または Running状態**で保持され、スケールアウトが必要になったときに**Hibernated / Stopped からの再開 または Running からの即時投入**で素早くサービスインできます。

**3つの状態**:
- **Stopped**: コスト最小（ストレージ料金のみ）。起動に数十秒
- **Hibernated**: メモリ状態を保存。起動はStoppedより速い
- **Running**: 常時動作、起動ゼロ秒。コストは通常と同じ

ライフサイクルフックと組み合わせて、起動後のウォームアップ処理（アプリの初期化、キャッシュの準備など）を完了させてからWarm Poolに入れる設計が一般的です。

---

## Part 4: ライフサイクルフック

**ライフサイクルフック**は、EC2インスタンスがASGに追加・削除される過程で**一時停止**させ、カスタム処理を実行する機能です。

**2つの Hook**:
- **Pending:Wait**: インスタンス起動後、`InService`になる前に一時停止。ユーザーデータ以降の初期化処理、デプロイ、設定完了を待つ
- **Terminating:Wait**: インスタンス終了前に一時停止。接続中のリクエストの完了待機、ログ送信、クリーンアップ

タイムアウト（デフォルト1時間）内に `CompleteLifecycleAction` APIを呼ぶか、タイムアウトで自動的に次の状態に遷移します。

---

## Part 5: Mixed Instances Policy と Spot活用

ASGの**Mixed Instances Policy**は、複数のインスタンスタイプとOn-Demand/Spotの混合を1つのASGで管理する機能です。

**設定項目**:
- **複数のインスタンスタイプ**: m5.large、m5a.large、m5n.large など複数を指定。それぞれの優先順位とウェイト
- **On-Demand Base Capacity**: 最低限保証するOn-Demandの台数（例: 常時10台はOn-Demand）
- **On-Demand Percentage Above Base Capacity**: ベース以上の部分で、何%をOn-Demandにするか（例: 10%）

**Capacity Rebalancing**: Spotインスタンスが中断の警告（2分前）を受けると、自動的に代替インスタンスを起動して切り替える機能。中断ダウンタイムを最小化。

**典型パターン**:
- Base Capacity: 10（常時On-Demand）
- OD percentage Above Base: 20%
- Spot: 80%
- 結果: ベースライン10 + 変動分は80%がSpot、20%がOn-Demand

---

## Part 6: Application Auto Scaling

EC2 Auto Scalingが EC2 向けである一方、**Application Auto Scaling**は他のAWSリソースのスケーリングを管理します。

| サービス | スケール対象 |
|---------|------------|
| ECS | サービスのタスク数 |
| DynamoDB | Provisionedキャパシティ（RCU/WCU） |
| Aurora | Reader数 |
| Lambda | Provisioned Concurrency |
| Spot Fleet | 容量 |
| EMR | ノード数 |
| SageMaker | エンドポイントインスタンス数 |

これらもTarget Tracking、Step Scaling、Scheduled Scalingをサポートします。

---

## Part 7: Placement Groups

ASG と併用されることが多い**Placement Groups**は、EC2インスタンスの物理配置を制御します。

- **Cluster**: 同じAZ内の物理的に近接した配置。低レイテンシ・高帯域幅（HPC、ML）
- **Spread**: 複数の物理ハードウェアに分散配置（最大7台/AZ）。ハードウェア障害耐性
- **Partition**: 論理パーティションに分けて配置（最大7パーティション/AZ）。HadoopやCassandraなどの分散システム

---

## 練習問題

### 問題1

ある動画配信サービスでは、ASGでEC2インスタンスを管理しています。アプリケーションは Tomcat + JVMで、起動してからトラフィックを受けられるまで約8分かかります（JVMウォームアップ、キャッシュロード、依存DB接続など）。

ピーク時にスケールアウトが発生しても、新しいインスタンスがサービスインするまでに8分かかるため、その間ユーザーリクエストが既存インスタンスに集中しレスポンスが遅くなる問題があります。

運用負荷を抑えつつ、スケールアウト時の応答性を大幅に改善する方法はどれですか？

<details>
<summary>選択肢を見る</summary>

A. ASGの最小キャパシティを大幅に増やし、常にピーク時の規模で運用する

B. Warm Poolを設定し、Stopped状態の事前ウォームアップ済みインスタンスを複数台保持する。スケールアウト時はStopped状態のインスタンスをStart（30〜60秒で起動）してすぐにサービスインできる

C. インスタンスタイプをもっと大きいものに変更する

D. Application Load Balancerのヘルスチェック間隔を短くする

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

Warm Poolが正解です。Warm Poolには事前にアプリケーション初期化まで完了したインスタンスをStopped状態で保持でき、スケールアウト時には数十秒で起動してサービスインできます。

- **起動時間の大幅短縮**: 8分 → 30〜60秒
- **コスト効率**: Stopped状態はストレージ料金のみ
- **ライフサイクルフックとの組み合わせ**: Pending:Waitでウォームアップを完了してからWarm Poolに入れる設計

- A: 常時ピーク規模で運用するとコストが大幅増加
- C: インスタンスタイプの変更は起動時間に直接影響しない（むしろ大きいインスタンスは初期化に時間がかかることもある）
- D: ヘルスチェック間隔の変更は新インスタンスの準備時間には影響しない

</details>

---

### 問題2

あるSaaS企業のCRMアプリケーションは、毎平日の朝9時〜12時にアクセスが集中し、他の時間帯は非常に少ないというパターンを長年維持しています。過去のトラフィックデータは2年以上蓄積されており、規則的なパターンがCloudWatchで観察されています。

現在はTarget Tracking Scalingで運用していますが、朝のピーク時にスケールアウトが間に合わずレスポンスが遅くなる問題があります。

スケールアウトを事前に完了させ、ピーク開始時点で十分なキャパシティを確保する最適な方法はどれですか？

<details>
<summary>選択肢を見る</summary>

A. Scheduled Scalingで毎平日朝8時45分に最小キャパシティを増やす

B. Predictive Scalingを有効化する。過去14日以上のトラフィックデータから機械学習で24時間先を予測し、ピーク到来前に事前にスケールアウトする。Target Trackingと組み合わせて使える

C. インスタンスタイプを大型にして単一インスタンスの処理能力を上げる

D. CloudFrontでAPIレスポンスをキャッシュする

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

Predictive Scalingが正解です。機械学習でトラフィックパターンを予測し、ピーク到来前にインスタンスをスケールアウトします。

- **予測ベースの事前スケール**: 24時間先まで予測
- **Target Trackingとの併用**: 予測スケールで事前に増やし、実際のメトリクスが予測とずれた場合はTarget Trackingで動的調整
- **トラフィックパターンが規則的**: 過去のデータから精度の高い予測が可能

- A: Scheduled Scalingも有効ですが、事業成長や祝日などの要因でパターンが変動するとチューニングが必要。Predictive Scalingは自動的にパターンを学習・適応
- C: 単一インスタンスの性能向上は垂直スケールですが、水平スケール（ASG）と組み合わせる方が一般的
- D: CloudFrontキャッシュは読み取り系には有効ですが、CRMアプリのような動的処理の多いAPIには効果限定的

</details>

---

### 問題3

ある企業では、EC2 Auto Scaling Groupで100台規模のWebサーバーを運用しています。コスト削減のため一部をSpotインスタンスにしたいですが、Spotは中断の可能性があるため、サービスの可用性を損なわないようにする必要があります。

要件：
1. 最低30台はOn-Demandで常時稼働（ベースライン）
2. それ以上の変動キャパシティは、80%をSpot、20%をOn-Demandで運用
3. 複数のインスタンスタイプ（m5、m5a、m5n、m5nd）で在庫切れリスクを減らす
4. Spotが中断される前に代替インスタンスを自動起動する

最適な設定はどれですか？

<details>
<summary>選択肢を見る</summary>

A. 2つの別々のASGを作成し、1つはOn-Demand専用、もう1つはSpot専用にする

B. ASGのMixed Instances Policyで複数インスタンスタイプを指定、On-Demand Base Capacity=30、On-Demand percentage Above Base Capacity=20%に設定。Capacity Rebalancingを有効化してSpot中断の警告受信時に代替起動を自動実行

C. EC2 Fleetで管理する

D. Spot Fleet単体で管理する

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

Mixed Instances Policy + Capacity Rebalancingが正解です。

- **Base Capacity**: 30台をOn-Demandで固定確保（要件1）
- **Percentage Above Base**: 20% On-Demand・80% Spot（要件2）
- **Multiple Instance Types**: m5、m5a、m5n、m5ndの4種類でSpot在庫切れリスクを分散（要件3）
- **Capacity Rebalancing**: 中断警告受信時に代替起動（要件4）

全要件を1つのASGで管理でき、運用がシンプル。

- A: 2つのASGを別々に管理するとスケーリング判断やALBターゲットの整合性が複雑。Mixed Instances Policyの方が管理負荷が低い
- C: EC2 Fleet単体ではASGのスケーリングやヘルスチェック管理がなく、Webサーバー用途には不向き
- D: Spot Fleet単体ではOn-Demandとの混合ができない

</details>

---

### 問題4

あるEコマースサイトでは、ECS Fargateでバックエンドサービスを運用しています。セールのような急激なトラフィック増加時にスケールが追いつかず、一方でアイドル時はタスクが無駄にコストを発生させています。

要件：
1. CPUが平均60%を維持するように自動スケール
2. セール時の急激な増加に対応
3. アイドル時はタスク数を最小に
4. スケーリングの設定を極力シンプルに

最適な方法はどれですか？

<details>
<summary>選択肢を見る</summary>

A. EC2 Auto ScalingをECS上で使う

B. Application Auto Scalingで ECS Service のタスク数に対してTarget Tracking Scalingを設定、目標値をECSServiceAverageCPUUtilization 60%にする。最小タスク数・最大タスク数を指定

C. Lambda関数でECS APIを呼んで独自にスケーリング管理する

D. EventBridgeの定期スケジュールでタスク数を手動変更する

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

Application Auto Scaling（Target Tracking）が正解です。ECSサービスのタスク数を自動調整する標準的な方法です。

- **Target Tracking Scaling**: `ECSServiceAverageCPUUtilization`を60%に保つようタスク数を調整
- **急増対応**: メトリクスが上昇すると自動的にタスクを増加
- **アイドル時**: 最小タスク数までスケールイン
- **設定がシンプル**: 目標値のみを指定、内部ロジックはApplication Auto Scalingが管理

- A: EC2 Auto ScalingはEC2インスタンス向けで、Fargateのタスクには使えない
- C: 自作スケーリングは開発・運用コストが高く、標準機能で十分
- D: 手動スケジュールは変動に対応できない

</details>

---

### 問題5

ある HPC（High Performance Computing）ワークロードでは、数百台のEC2インスタンスが低レイテンシで相互通信する必要があります。各インスタンスは大容量のデータを交換しながら並列計算を実行します。インスタンス間の通信遅延が計算時間に直結するため、できるだけ近接した物理配置にしたいと考えています。

最適な配置方法はどれですか？

<details>
<summary>選択肢を見る</summary>

A. Placement Groupのクラスター配置（Cluster Placement Group）で同一AZの物理的に近接したハードウェアにインスタンスを配置する

B. Placement Groupのスプレッド配置（Spread）で物理的に分散させる

C. 複数のAZにインスタンスを分散配置して可用性を高める

D. リージョンを複数に分けて分散処理する

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: A**

Cluster Placement Groupが正解です。同一AZ内の物理的に近接したハードウェアにインスタンスを配置し、最低レイテンシ（マイクロ秒単位）と最大帯域幅を提供します。HPCワークロードに最適です。

- **低レイテンシ**: 物理的近接によりネットワーク遅延が最小
- **高帯域幅**: Enhanced Networkingで最大25 Gbps（HPCインスタンスでは100 Gbps以上）
- **同一AZ**: AZ跨ぎの遅延がない

- B: Spreadは障害耐性向けで、HPCのような低レイテンシ要件には不向き
- C: 複数AZ分散はネットワーク遅延が増えるため、HPCには向かない
- D: 複数リージョン分散は更にレイテンシが増加

</details>

---

### 問題6

ある金融サービスのASGで、インスタンス終了時に以下の処理が必要という要件があります。

1. 処理中のトランザクションを完了させる（最大5分）
2. セッションキャッシュをS3にエクスポートする（2分）
3. 終了ログをCloudWatch Logsに送信（1分）

ASGはヘルスチェック失敗やスケールインで自動的にインスタンスを終了しますが、上記の処理が完了する前に終了されると、トランザクション損失やデータ損失が発生する可能性があります。

これを解決する最適な方法はどれですか？

<details>
<summary>選択肢を見る</summary>

A. インスタンスのシャットダウンスクリプト（cloud-init）で処理を実行する

B. ASGにTerminating:Waitライフサイクルフックを設定し、インスタンス終了時に一時停止。カスタム処理（トランザクション完了、S3エクスポート、ログ送信）の完了後にCompleteLifecycleActionを呼ぶことでASGが終了を続行する

C. CloudWatch Eventsで終了イベントをキャッチしてLambdaで処理する

D. ASGのインスタンス保護機能を有効化する

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

Terminating:Waitライフサイクルフックが正解です。

- **一時停止**: インスタンス終了が開始されると、ASGがインスタンスを`Terminating:Wait`状態で一時停止
- **カスタム処理の実行**: インスタンス内のスクリプトまたはLambda経由で必要な処理を実行
- **完了通知**: 処理完了後、`CompleteLifecycleAction` APIを呼ぶことでASGが終了プロセスを続行
- **タイムアウト**: デフォルト1時間（最大48時間）まで待機可能

- A: cloud-initのシャットダウンスクリプトはタイミング保証がなく、ASGが強制終了する可能性がある
- C: CloudWatch Eventsでは終了後のイベントキャッチで、終了前の処理保護にはならない
- D: インスタンス保護は「ASGがこのインスタンスを終了させない」機能で、スケールインを防ぐだけ。通常の終了プロセスで本件の処理を実行する仕組みではない

</details>
