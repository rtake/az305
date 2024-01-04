# 非リレーショナルデータベース
## ストレージアカウント
### 種類
- Standard 汎用 v2
- Premium ブロック BLOB
- Premium ファイル共有
- Premium ブロック BLOB
  - OS, 仮想マシン用データディスク, データベースなどに適する

### 冗長性
#### Geo 冗長
- セカンダリリージョンはプライマリリージョンに基づいて決定される(変更不可)
- GRS と GZRS はプライマリリージョンでのレプリケーションに違いがある
  - セカンダリリージョンではどちらも LRS

### BLOB Storage
- アクセスオプションは以下の4つ:
  - Premium BLOB Storage
    - SSD 使用
    - 待機時間は 1 桁ミリ秒
  - ホット
    - 待機時間はミリ秒
  - クール
    - 最小ストレージ存在期間は 30 日
  - アーカイブ
    - 最小ストレージ存在期間は 180 日
    - **リハイドレートに要する時間は 15時間**
- **不変ストレージ**がサポートされる

### Files
- NAS やファイルサーバとして使用可能
- File Sync でオンプレミスに Files のキャッシュを保存できる
- 以下のストレージ層がサポートされる
  - Premium
  - トランザクション最適化
  - ホットアクセス層
  - クールアクセス層
- マネージドファイルサービスである **NetApp Files** も利用できる

### マネージドディスク
- 仮想マシンによって使用される
- Ultra Disk(SSD), Premium SSD, Standard SSD, Standard HDD などが選択可能
  - Prod には **Premium SSD** が適する
  - Ultra Disk は利用できるリージョンが限られる

### 暗号化オプション
- Azure Disk Encryption(ADE): 仮想ハードディスク(VHD) の暗号化
- Server-Side Encryption(SSE): 物理ディスクの暗号化(保存時の暗号化, Azure Storage 暗号化)

### セキュリティ
- サービスエンドポイントを使うと, ストレージへのトラフィックを Azure のバックボーンネットワーク上に維持できる
  - パブリック IP が不要だが, プライベート IP の有効化が必要
- プライベートエンドポイントを使うと, 特定のサービス用に専用の接続を提供できる
  - 仮想ネットワーク内にある他のリソースからのアクセスを拒否できる


# リレーショナルデータベース
## SQL Database
### 価格オプション
- サービス階層
  - 仮想コアベース
    - General Purpose
    - Business Critical
    - Hyperscale
  - DTU: 事前構成済
    - Basic
    - Standard
    - Premium
  - 仮想コアでは, 予約割引を受けられる
- コンピューティングレベル
  - プロビジョニング済
  - サーバーレス: 従量課金
- デプロイモデル
  - 単一データベース
  - エラスティックプール
- [Azure SQL Database とは](https://learn.microsoft.com/ja-jp/azure/azure-sql/database/sql-database-paas-overview?view=azuresql#compute-tiers)

### 可用性
- ローカル冗長可用性は**全てのサービスレベルでサポートされる**
- ゾーン冗長可用性は **Basic(DTU), Standard(DTU) ではサポート外**
- [Azure SQL Database の高可用性](https://learn.microsoft.com/ja-jp/azure/azure-sql/database/high-availability-sla?view=azuresql&tabs=azure-powershell#zone-redundant-availability)

#### ローカル冗長
##### Basic / Standard / General Purpose
- **リモート記憶域可用性モデル**
- プライマリレプリカでは, ローカル VM のSSD が一時データベースとして用いられる
- データファイルとログファイルは Premium ブロック BLOB(LRS 冗長) で保存される
- バックアップファイルは Standard 汎用 v2(RA-GRS) に保存される(**グローバル冗長**)
  - *これがリモート記憶域可用性モデルと言われる理由[要出典]*
- プライマリで障害発生時には, 新しい SQL Server インスタンスが起動する(**フェールオーバー**)

##### Premium / Businnes Critical
- **ローカルストレージ可用性モデル**, つまりコンピューティングリソースとストレージ (SSD) が 1 つのノードに統合される
- データファイルとログファイルはローカル SSD に配置される
- 高可用性は **Always On 可用性グループ**と同じようにして実装される
  - クラスターにはプライマリレプリカと, 最大 3 つのセカンダリレプリカが含まれる
  - プライマリで障害発生時には, グループ内のレプリカにフェールオーバーする
    - このとき, データの損失はない(フェールオーバー先となる完全に同期されたノードが常に存在するため)
- このアーキテクチャにより**読み取りスケールアウト**がサポートされる
  - 読み取り専用の SQL 接続をセカンダリレプリカの 1 つにリダイレクトする機能

##### Hyperscale
- [Hyperscale でのデータベースの高可用性](https://learn.microsoft.com/ja-jp/azure/azure-sql/database/service-tier-hyperscale?view=azuresql#database-high-availability-in-hyperscale)

#### ゾーン冗長
##### Basic / Standard
- (サポート無し)

##### General Purpose
- データファイルとログファイルは Premium ブロック BLOB(ZRS 冗長) で保存される
- ステートレス計算レイヤーが 3 つの Availability Zone で稼働しており, フェールオーバー時にすぐに使用できる

##### Business Critical / Premium
- Always On 可用性グループと制御リング(ゲートウェイ)がゾーンをまたがって構成され, Traffic Manager によって制御される
- **追加のデータベース冗長を構成しないため, 追加料金なしで使用できる**

##### Hyperscale
- [ゾーン冗長 Hyperscale データベースを作成する](https://learn.microsoft.com/ja-jp/azure/azure-sql/database/hyperscale-create-zone-redundant-database?view=azuresql&tabs=azure-powershell)

## SQL Managed Instance
- アプリケーションの再設計無しでリフトアンドシフト移行が可能
  - 共通言語ランタイム(CLR), SQL Server エージェントなどをサポート
    - SQL Server エージェント: ジョブをスケーリングできる
- **仮想コアモードを使用**
  - 最大 CPU コアと最大ストレージを定義できる(**動的スケーリング**)

### サービスレベル
- General Purpose
- Businnes Critical

### 可用性
- ローカル冗長可用性はサポートされている(アーキテクチャは SQL Database と同じ)
- **ゾーン冗長は General Purpose のパブリックプレビュー段階で, Business Critical で使用できる**
- [Azure SQL Managed Instance の高可用性](https://learn.microsoft.com/ja-jp/azure/azure-sql/managed-instance/high-availability-sla?view=azuresql)

## フェールオーバー
- **Service Fabric** によって自動実行される
  - SQL Managed Instance では手動実行もサポートされる(SQL Database では手動実行は不可能)
- Business Critical と Premium では, フェールオーバー先となる完全に同期されたノードが常に存在するため, **レプリカ間でデータ損失なしにフェールオーバーできる**
  - *おそらく Hyperscale も損失なくフェールオーバー可能[要検証]*

## スケーラビリティ
- SQL Database と SQL Managed Instance の両方で動的スケーラビリティがサポートされている

### スケールアップ(垂直)
- コンピューティングサイズを動的に変更すること
- SQL Database では**エラスティックプール**を使用することでプール内のデータベースのグループごとの最大リソース制限を設定できる
- [最小限のダウンタイムでデータベース リソースを動的に拡張 - Azure SQL データベース と Azure SQL Managed Instance](https://learn.microsoft.com/ja-jp/azure/azure-sql/database/scale-resources?view=azuresql)

### スケールアウト(水平)
- データベースを追加または削除して, 容量や全体のパフォーマンスを調整すること
- **Elastic Database クライアントライブラリ**で管理する
- スケールアウトの方法
  - シャーディング - データを分割し複数のデータベースに分散して保管する
  - 読み取りスケールアウト -  読み取り専用ワークロードをオフロードすることで負荷分散する

#### 読み取りスケールアウト
##### SQL Database
- Basic, Standard, General Purpose では使用できない
- Premium, Bussiness Critical では既定で有効化される
- Hyperscale では既定で有効化されるが, セカンダリレプリカが作成されていない場合は自動的に無効になる

##### SQL Managed Instance
- General Purpose では使用できない
- **Bussiness Critical では自動的に有効になる**

## 別リージョンへのレプリケーション
- 自動フェールオーバーグループ
  - 別リージョンにデータベースを複製するので追加コストがかかる
   - Premium / Business Critical のゾーン冗長構成は追加コストがかからない
- アクティブ geo レプリケーション
  - SQL Managed Instance ではサポートされない

## データ暗号化
### 保存データ
- 透過的データベース暗号化(TDE: Transparent Data Encryption)
  - データはディスク上のデータページに書き込まれるときに暗号化され, データページがメモリに読み込まれるときに解読される
- 暗号化レベル: Always Encrypted
- バックアップも暗号化される
- データベース暗号化キー(DEK) で全体が暗号化される
  - マネージド TDE とカスタマーマネージド TDE がサポートされる

### 移動中データ
- SSL/TLS で暗号化
- レベル: Always Encrypted
- オンプレミスとつなぐ場合には, VPN か ExpressRoute を使用する

### 処理中のデータ
- 動的データマスクで, 指定したフィールドを非表示にできる
- マスキングポリシーはポータルから設定できるほか, PowerShell や REST API でも設定できる

## SQL Edge
- IoT および IoT Edge のデプロイ向けに最適化されたリレーショナルデータベースエンジン
- コンテナ化された Linux アプリケーション
- エッジコンポーネントとの連携

## Cosmos DB
- Table Storage 向けアプリケーションは Cosmos DB Table API に移行できる
  - 同じ SDK で操作できる
- 速度の SLA が保証される


# データ統合
## Data Factory
- ジョブのスケジューリングが可能

## Data Lake
- 複数のソースからリアルタイムデータを直接取り込むことができる
- Geo 冗長は手動で構成する必要がある

## Databricks
- ビッグデータ処理と機械学習を単一のプラットフォームで行うことができる
- Apache Spark に基づいている
- **行フィルターと列マスクがサポート**
  - [行フィルターと列マスクを使用して機密性の高いテーブル データをフィルター処理する](https://learn.microsoft.com/ja-jp/azure/databricks/data-governance/unity-catalog/row-and-column-filters)

## Synapse Analytics
- サーバーレスデータや大規模データに対してクエリを実行できる
- 超並列処理(MPP)アーキテクチャ
- **最大 128個のクエリを実行可能**

### コンポーネント
- Synapse SQL プール
- Synapse Spark プール: Apache Spark を実行してデータを処理するサーバーのクラスター
- Synapse パイプライン: データの移動と変換. **Data Factory の機能が適用される**
- Synapse Link: Cosmos DB に接続できる
- Synapse Studio: Web ベース IDE

## Stream Analytics
- リアルタイム分析を行うイベント処理エンジン
- CSV, JSON, Avro のイベント処理がサポートされる
- Event Hub や IoT Hub, BLOB Storage からデータを取り込む