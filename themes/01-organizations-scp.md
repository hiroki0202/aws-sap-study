# テーマ1: AWS Organizations + SCP + OU設計

> 🔴 所要日数: 3-4日 | 座学 → ハンズオン → 問題演習

---

## 座学

### SAAからの知識差分

| SAAレベル | SAPで追加で問われること |
|-----------|----------------------|
| Organizationsの存在を知っている | OU設計戦略、SCP継承の動作を**設計判断として使える** |
| SCPが「制限をかけるもの」と知っている | **具体的にどう書くか**、何に適用でき何にできないか |
| IAMポリシーを書ける | SCP/IAM/権限境界の**3つの制御レイヤーの関係**を理解 |

---

### 1. AWS Organizationsの基本構造

```
管理アカウント（Management Account）
└── Root（組織のルート）
    ├── OU: Production
    │   ├── Account A
    │   └── Account B
    ├── OU: Development
    │   └── Account C
    └── OU: Security
        └── Account D（ログ集約用）
```

**重要ポイント:**
- 管理アカウント自体にはSCPは効かない（常にフルアクセス）
- SCPはメンバーアカウントにのみ適用
- 1つのアカウントは1つのOUにしか所属できない

---

### 2. SCPの仕組み

#### SCPとは何か
- **許可の上限（ガードレール）** を定義するもの
- SCP自体は権限を**付与しない**。あくまで「ここまでしか許可しない」という天井
- IAMで許可されていても、SCPで許可されていなければ**実行不可**

#### 評価の仕組み（アクセス許可の決定フロー）

```
リクエスト発生
  ↓
① SCPで許可されているか？ → NO → 拒否（ここで終了）
  ↓ YES
② IAMポリシーで許可されているか？ → NO → 拒否
  ↓ YES
③ リソースベースポリシーで許可されているか？
  ↓
④ 権限境界で許可されているか？
  ↓
→ 最終的に許可
```

**つまり：SCP → IAM → リソースポリシー → 権限境界 の全部がOKでないとアクションは実行できない**

#### SCPの継承

```
Root にアタッチされたSCP
  ↓ 継承
OU にアタッチされたSCP（Rootのものも有効）
  ↓ 継承
Account に適用される有効なSCP = Root + OU + Account直接アタッチ の交差（AND条件）
```

**重要：継承は「加算」ではなく「交差（積集合）」**

例：
- Root SCPで `ec2:*` と `s3:*` を許可
- OU SCPで `ec2:*` のみ許可
- → そのOU配下のアカウントは `ec2:*` のみ使える（s3は使えない）

#### SCPの適用対象

| 対象 | SCPの影響 |
|------|----------|
| メンバーアカウントのIAMユーザー | ✅ 受ける |
| メンバーアカウントのIAMロール | ✅ 受ける |
| メンバーアカウントのルートユーザー | ✅ **受ける**（重要！） |
| 管理アカウントのユーザー | ❌ 受けない |
| サービスリンクロール | ❌ 受けない |

> 💡 **SAPで最頻出ポイント:** SCPはルートユーザーにも適用される。IAMポリシーではルートユーザーを制御できないが、SCPなら可能。

---

### 3. SCPの2つのアプローチ

#### ホワイトリスト型（許可リスト）
「指定したアクションだけ許可し、他はすべて拒否」

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:*",
        "s3:*",
        "rds:*"
      ],
      "Resource": "*"
    }
  ]
}
```

#### ブラックリスト型（拒否リスト）
「特定のアクションだけ禁止し、他はデフォルトで許可」

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "ec2:RunInstances"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "ec2:InstanceType": ["t3.micro", "t3.small", "t3.medium"]
        }
      }
    }
  ]
}
```

> 💡 **実務のベストプラクティス:** デフォルトの `FullAWSAccess` ポリシーを残しつつ、ブラックリスト型で「やってほしくないこと」だけを明示的に拒否する構成が多い。

---

### 4. よくあるSCPユースケース（SAP頻出）

#### ① リージョン制限
「東京と大阪以外でのリソース作成を禁止」

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": ["ap-northeast-1", "ap-northeast-3"]
        },
        "ArnNotLike": {
          "aws:PrincipalARN": "arn:aws:iam::*:role/OrganizationAdmin"
        }
      }
    }
  ]
}
```

#### ② 暗号化強制
「S3にSSE-KMSなしでPutObjectすることを禁止」

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "s3:PutObject",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": "aws:kms"
        }
      }
    }
  ]
}
```

#### ③ 高額インスタンスタイプの禁止
「p4d（GPU）インスタンスの起動を禁止」

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "ec2:RunInstances",
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "StringLike": {
          "ec2:InstanceType": ["p4d.*", "p3.*", "inf1.*"]
        }
      }
    }
  ]
}
```

#### ④ CloudTrail無効化の防止
「CloudTrailの停止・削除を禁止」

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "cloudtrail:StopLogging",
        "cloudtrail:DeleteTrail"
      ],
      "Resource": "*"
    }
  ]
}
```

---

### 5. OU設計パターン（SAP頻出）

#### パターン①: 環境別OU（基本形）

```
Root
├── OU: Production（本番SCP：厳格）
├── OU: Staging（検証SCP：やや緩い）
├── OU: Development（開発SCP：緩い）
└── OU: Security（ログ集約・監査）
```

**用途:** 環境ごとにSCPの厳しさを変えたい場合

#### パターン②: 部門別OU + 共通制限

```
Root ← 共通SCP（全社リージョン制限等）
├── OU: 営業部門
├── OU: 開発部門
└── OU: 研究部門 ← 追加SCP（高額GPU制限等）
```

**用途:** 全社共通ルールをRootに、部門固有ルールを各OUに

#### パターン③: 中間OUによる共通化

```
Root
├── OU: CoreBusiness ← 共通SCP
│   ├── OU: 部門A
│   └── OU: 部門B
└── OU: Exceptions ← 別のSCP
    └── OU: 特殊プロジェクト
```

**用途:** 2つ以上の部門で共通SCPを使いたいが、他部門は除外したい場合。中間OUに共通SCPを置くことで**1か所で管理**できる。

#### パターン④: セキュリティアカウント集約

```
Root
├── OU: Workloads
│   ├── Account: サービスA
│   └── Account: サービスB
├── OU: Security ← 監査・ログ
│   └── Account: SecurityHub集約
└── OU: Infrastructure ← ネットワーク
    └── Account: Transit Gateway管理
```

**用途:** AWS Control Tower推奨構成。ログ集約とネットワーク集中管理を分離。

---

### 6. SCP vs IAM vs 権限境界 の比較

| | SCP | IAMポリシー | 権限境界 |
|--|-----|-----------|---------|
| 何を制御する？ | アカウント全体の**上限** | ユーザー/ロールの**許可** | IAMエンティティの**上限** |
| 適用対象 | OU / アカウント | IAMユーザー / ロール | IAMユーザー / ロール |
| ルートユーザーに効く？ | ✅ はい | ❌ いいえ | ❌ いいえ |
| 権限を付与する？ | ❌ しない（上限のみ） | ✅ する | ❌ しない（上限のみ） |
| 管理する人 | 組織管理者 | アカウント管理者 | アカウント管理者 |
| 典型ユースケース | 全社ガードレール | 日常のアクセス制御 | IAM操作の委任時 |

> 💡 **SAP試験での判断基準:**
> - 「全アカウントに一括適用」→ **SCP**
> - 「特定ユーザーの権限設定」→ **IAMポリシー**
> - 「IAMロール作成は許すが、付与できる権限を制限したい」→ **権限境界**

---

### 7. SCPではできないこと

| できないこと | 代替手段 |
|------------|---------|
| 特定のタグの値を強制する | タグポリシー |
| S3のデータ内容を検査する | Amazon Macie |
| リソース変更の検知・通知 | AWS Config |
| 通信ポートの制御 | Firewall Manager |
| 特定ユーザーだけに例外を与える（SCP内で） | IAMポリシー + 条件 |

---

### 8. 試験で狙われるひっかけポイント

1. **「IAMで許可されているのに操作できない」→ SCPを疑え**
2. **「複数アカウントで同時に問題発生」→ 組織レベルの設定（SCP）を確認**
3. **「管理アカウントから制御したい」→ SCPは管理アカウントから設定**
4. **「ルートユーザーも制御したい」→ IAMでは不可、SCPなら可能**
5. **「通知だけでなく強制したい」→ Config/Budgetsではなく、SCPで拒否**
6. **「新規アカウントにも自動適用」→ OU単位のSCPならOUに入れるだけで適用**
7. **「SCPをアカウント個別に適用」→ 管理煩雑、OU単位が推奨**

---

## ハンズオン

### 前提条件
- AWSアカウントが1つ以上ある（フリーティア可）
- まだOrganizationsを使っていない場合は新規作成

### 手順

#### Step 1: Organizationsを作成する

1. AWSマネジメントコンソールにログイン
2. **AWS Organizations** サービスを開く
3. 「組織を作成」をクリック
4. 「すべての機能を有効にする」を選択して作成

#### Step 2: OUを作成する

1. Organizations コンソールで「Root」をクリック
2. 「アクション」→「組織単位を作成」
3. 以下の2つのOUを作成：
   - `Production`
   - `Development`

#### Step 3: テスト用SCPを作成する（リージョン制限）

1. 左メニューから「ポリシー」→「サービスコントロールポリシー」
2. 「ポリシーを作成」をクリック
3. 名前: `DenyNonTokyoRegion`
4. 以下のJSON:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyNonApprovedRegions",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": ["ap-northeast-1"]
        },
        "ArnNotLike": {
          "aws:PrincipalARN": "arn:aws:iam::*:role/aws-service-role/*"
        }
      }
    }
  ]
}
```

5. 保存

#### Step 4: SCPをOUにアタッチする

1. 「組織単位」から `Development` をクリック
2. 「ポリシー」タブ → 「アタッチ」
3. 先ほど作成した `DenyNonTokyoRegion` を選択
4. アタッチ

#### Step 5: 動作確認

1. Developmentに所属するアカウント（または管理アカウント以外）にログイン
2. リージョンを `us-east-1` に切り替え
3. EC2 コンソールで何かアクションを試みる
4. **「権限がありません」エラーが出ればSCPが有効に動作している** ✅

#### Step 6: 継承の確認

1. Root にもう1つSCPを作成（例：`DenyDeleteCloudTrail`）
2. Root にアタッチ
3. Development配下のアカウントで、CloudTrail削除を試みる
4. **Root SCPもOU SCPも両方有効になっていることを確認**

#### クリーンアップ
- SCPをデタッチ → 削除
- OUを削除（中にアカウントがない状態にしてから）
- 不要であれば Organizations を無効化

---

## 練習問題

### 問題1

ある製造グループでは、AWS Organizationsで全世界の拠点のAWSアカウントを管理しています。最近、複数のアカウントで同時に「Lambda関数のデプロイができない」という報告が上がりました。調査したところ、各アカウントのIAMロールには`lambda:CreateFunction`が明示的に許可されています。

この問題の原因として最も可能性が高いものはどれですか？

<details>
<summary>選択肢を見る</summary>

A. Lambda関数のメモリ上限を超えている

B. 各アカウントのIAMロールの信頼ポリシーが正しくない

C. 最近展開されたSCPでLambdaの操作が制限されている

D. AWS Configルールが非準拠と判定してブロックしている
</details>

<details>
<summary>正解と解説を見る</summary>

**正解: C**

SCPはIAMポリシーの上位に位置する「許可の上限」であり、SCPで許可されていないアクションは、IAMでいくら許可していても実行できません。複数アカウントで同時に発生している点から、組織レベルの設定変更（SCP）が原因と推定できます。

- A: メモリ上限はデプロイ可否に直接影響しない
- B: IAMは問題ないと明示されている
- D: AWS Configは検知のみで操作をブロックしない
</details>

---

### 問題2

ある金融グループでは、AWS Organizationsで以下のOU構成を取っています：

- OU: Trading（証券取引システム）
- OU: BackOffice（事務処理システム）

両部門に共通して「ap-northeast-1以外でのリソース作成を禁止」するSCPを適用したいが、今後Tradingにはさらに「特定のEC2インスタンスタイプのみ許可」という制限も追加する予定です。

SCPの重複管理を避け、将来の変更も一か所で済むようにするには、どのOU構成変更が最適ですか？

<details>
<summary>選択肢を見る</summary>

A. 両方のOUに同じSCPを個別にアタッチし、Trading側にはもう1つSCPを追加する

B. 共通制限用の中間OUを作成し、TradingとBackOfficeをその子OUとして配置する。共通SCPは中間OUに、Tradingの追加制限はTrading OUにアタッチする

C. Trading OUをBackOffice OUの子として配置し、共通SCPをBackOffice OUにアタッチする

D. 全てのSCPを組織のRootにアタッチし、条件キーで適用対象を分ける
</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

SCPは親OUから子OUに継承されるため、共通制限を中間（親）OUに集約し、個別制限は各子OUに配置するのがベストプラクティスです。これにより：
- 共通SCPの変更は1か所で完了
- 各部門の独自制限も柔軟に追加可能
- OU間の分離性も維持

- A: 共通SCPが2か所に存在し、変更のたびに両方更新が必要
- C: 継承の方向が逆。BackOfficeの制限がTradingに漏れる
- D: 条件キーでの分岐は複雑で保守性が低い
</details>

---

### 問題3

あるヘルスケア企業は、50以上のAWSアカウントをOrganizationsで管理しています。コンプライアンス要件により、全アカウントのS3バケットに保存されるデータは必ずカスタマー管理KMSキー（CMK）で暗号化されている必要があります。

一部のエンジニアが暗号化指定なしでファイルをアップロードしてしまう事故が繰り返し発生しており、今後は技術的にこれを防止したいと考えています。各アカウントへの個別設定は避け、一元管理したいという要件もあります。

この要件を満たす最適な方法はどれですか？

<details>
<summary>選択肢を見る</summary>

A. 各アカウントでAWS Configルールを展開し、暗号化されていないオブジェクトを検出して通知する

B. IAMポリシーで全ユーザーにs3:PutObject時のSSE-KMS指定を必須とするポリシーを配布する

C. SCPでs3:PutObjectリクエストにs3:x-amz-server-side-encryptionヘッダーがない場合を拒否する

D. Amazon Macieを有効化し、暗号化されていないオブジェクトをスキャン・削除する
</details>

<details>
<summary>正解と解説を見る</summary>

**正解: C**

SCPを使えば、組織単位で「暗号化指定のないPutObjectを拒否」するルールを強制できます。これにより：
- 全アカウントに一括適用可能
- エンジニアの操作ミスを技術的にブロック
- 個別アカウントへのポリシー配布不要

- A: Configは検知と通知のみ。ブロックはできない
- B: IAMポリシーの配布は各アカウントで必要。一元管理にならない
- D: Macieはデータの検出が目的。リクエスト自体のブロックは不可
</details>

---

### 問題4

ある研究機関では、IT部門のアカウント管理者にIAMロールの作成権限を委任しています。しかし最近、管理者が`AdministratorAccess`相当の権限を持つロールを誤って作成し、セキュリティインシデントにつながりました。

今後は、IAMロールの作成自体は引き続き許可しつつ、作成されるロールに付与できる権限の上限を制限したいと考えています。

この要件に最も適した制御手段はどれですか？

<details>
<summary>選択肢を見る</summary>

A. SCPでiam:CreateRoleを条件付きで制限する

B. IAM権限境界（Permissions Boundary）を管理者に適用する

C. AWS Configで作成されたロールの権限を監視し、違反時に通知する

D. IAMポリシーから管理者のiam:CreateRole権限を削除する
</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

IAM権限境界（Permissions Boundary）は、「IAMユーザーやロールが持てる最大権限」を制限する仕組みです。管理者にPermissions Boundaryを設定すれば：
- IAMロールの作成は可能（操作自体は許可）
- ただし、作成したロールが持てる権限は境界内に制限される
- 「AdministratorAccess」のような広範なポリシーを付けても、実効権限は境界に収まる

- A: SCPはアカウント全体の制限であり、「作成するロールの中身」まではAWS Organizations以外から制御できない
- C: 監視は事後対応であり、防止にはならない
- D: 作成権限を削除すると委任の目的を達成できない
</details>

---

### 問題5

あるグローバル企業では、AWS Organizationsで以下のOU構成を運用しています：

- Root
  - OU: APAC（アジア太平洋）
  - OU: EMEA（欧州・中東）
  - OU: Americas（南北アメリカ）

情報セキュリティ部門は、全リージョンの全アカウントでCloudTrailとAWS Configが有効になっている状態を維持し、これらが無効化・削除されないようにガードレールを設けたいと考えています。新しいアカウントが追加された場合にも自動的にこのガードレールが適用される必要があります。

最小限の管理作業でこの要件を実現する方法はどれですか？

<details>
<summary>選択肢を見る</summary>

A. 各アカウントにIAMポリシーを配布し、cloudtrail:StopLoggingとconfig:DeleteConfigurationRecorderを明示的にDenyする

B. 組織のRootに対してSCPをアタッチし、CloudTrailの停止・削除およびConfigの無効化を禁止する

C. AWS Configルールで各アカウントの設定を監視し、違反が見つかった場合にSNSで管理者に通知する

D. 各OUに個別のSCPを作成し、それぞれの地域ポリシーに合わせた内容でアタッチする
</details>

<details>
<summary>正解と解説を見る</summary>

**正解: B**

組織のRoot（最上位）にSCPをアタッチすれば、全OU・全アカウントに自動的に継承されます。新規アカウントが追加されても、どのOU配下に入れてもRootのSCPが継承されるため、追加設定は不要です。

- A: IAMポリシーの配布は各アカウントで必要であり一元管理にならない。ルートユーザーにも適用されない
- C: 通知は事後対応。無効化を防止できない
- D: 共通ルールなのに各OUに個別設定するのは無駄。RootでOK
</details>

---

### 問題6

あるスタートアップ企業が急成長し、AWSアカウントが30を超えました。コスト管理のため、各チームが使用できるAWSサービスを制限したいのですが、一部のチームは機械学習サービス（SageMaker等）を使用しており、他のチームには使わせたくないという要件があります。

セキュリティ担当者は「将来チームが増えても、設定の手間を最小限にしたい」と考えています。

この要件に最も適した構成はどれですか？

<details>
<summary>選択肢を見る</summary>

A. 各アカウントのIAMユーザーに個別のポリシーを設定し、SageMakerへのアクセスを必要に応じて許可/拒否する

B. 組織のRootにSageMakerの使用を許可するSCPを適用し、不要なアカウントに対して個別にDenyのSCPをアタッチする

C. OUを「ML許可」と「ML禁止」に分け、ML禁止OUにSageMakerをDenyするSCPをアタッチする

D. AWS Service Catalogを使い、SageMakerを含む製品のみを特定チームに提供する
</details>

<details>
<summary>正解と解説を見る</summary>

**正解: C**

OU単位でSCPを管理することで：
- 新しいチームが追加されても、適切なOUに入れるだけで自動的にSCPが適用
- ML利用が許可されたチームは「ML許可」OUへ
- それ以外は「ML禁止」OUへ
- 変更があった場合もOUを移動するだけ

- A: 個別管理は30アカウント超ではスケールしない
- B: 個別アカウントへのSCP適用は管理煩雑
- D: Service Catalogはプロビジョニング制御であり、サービス利用自体の制限には不向き
</details>

---

### 問題7

あるIT企業のセキュリティチームが、自社のAWS環境を調査したところ、以下の状況が判明しました：

- 3つのアカウントで同時にRDSのスナップショット作成が失敗している
- 各アカウントのIAMロールには `rds:CreateDBSnapshot` が明示的に許可されている
- 前日にOrganizationsチームが新しいSCPをデプロイしていた
- CloudTrailログでは `AccessDenied` が記録されている

この状況を最も迅速に解決するための第一歩はどれですか？

<details>
<summary>選択肢を見る</summary>

A. 各アカウントにログインし、IAMロールのポリシーを確認する

B. AWS Configで設定変更履歴を確認し、影響を受けたリソースを特定する

C. Organizations管理アカウントからSCPを確認し、rds:CreateDBSnapshotが制限されていないか調べる

D. AWSサポートに問い合わせて、サービス障害が発生していないか確認する
</details>

<details>
<summary>正解と解説を見る</summary>

**正解: C**

「複数アカウントで同時に発生」「IAMは問題ない」「SCPが最近デプロイされた」という3つの手がかりから、組織レベルの制御（SCP）を最優先で確認すべきです。管理アカウントにログインし、該当OUまたはアカウントにアタッチされているSCPの内容を確認するのが最も迅速な解決法です。

- A: IAMは問題ないと確認済み。遠回り
- B: Configは設定変更の記録であり、SCPの確認には直接つながらない
- D: 3アカウント同時の場合、サービス障害よりSCPが原因の可能性が圧倒的に高い
</details>

---

### 問題8

大手保険会社のクラウド統括部門は、AWS Organizationsを以下のように構成しています。

- Root
  - OU: 共通基盤（ネットワーク・セキュリティ）
  - OU: 損害保険事業
  - OU: 生命保険事業

現在、「損害保険事業」と「生命保険事業」の両方に共通して適用すべきSCPが5つあり、変更のたびに2つのOUを別々に更新しなければなりません。一方、「共通基盤」には別のSCPが必要です。

SCPの管理効率を上げつつ、各事業の独立性を維持するためのOU構成変更として最適なものはどれですか？

<details>
<summary>選択肢を見る</summary>

A. 「保険事業」という中間OUを作成し、「損害保険」「生命保険」OUをその子OUとして配置する。共通SCPは「保険事業」OUにアタッチする

B. 共通SCPをすべて組織のRootにアタッチし、「共通基盤」OUには別途例外用のSCPを追加する

C. 「損害保険」と「生命保険」のOUを統合し、1つのOUで運用する

D. 各アカウントに直接SCPをアタッチし、OUレベルでの管理をやめる
</details>

<details>
<summary>正解と解説を見る</summary>

**正解: A**

中間OUを挟むことで、共通SCPを1か所に集約でき、変更も1回で2事業に反映されます。各事業OUには固有の制限を追加することも可能で、独立性も維持できます。

- B: Rootに全部置くと共通基盤にも適用されてしまう。例外処理が複雑化
- C: 統合すると事業間の権限分離ができなくなる
- D: 個別アカウントへの直接適用はスケールしない
</details>
