# データストレージソリューション
## SQL
### SQL Database
#### エラスティックプール
- 複数の DB を移行する際にはエラスティックプールを使用するのが適切
  - エラスティックプール: リソースプールを作成し, 複数のデータベースで共有できるようにする
- > プールに追加できるデータベースが多ければ多いほど、節約量も多くなります。
  - [Azure SQL Database におけるエラスティック プールを利用した複数のデータベースの管理およびスケーリング](https://learn.microsoft.com/ja-jp/azure/azure-sql/database/elastic-pool-overview?view=azuresql#use-other-sql-database-features-with-elastic-pools)

#### サービスレベル
- **レイテンシを最低限に押さえるには Premium または Bussiness Critial を選択**
  - ローカル SSD を利用するため高速
- Premium または Bussiness Critical ではゾーン冗長を選べる

#### ゾーン冗長
- 仮想コアを使用する全てのレベルでサポートされる
  - General Purpose, Bussiness Critical, Hyperscale
- DTU を使用するレベルでは Premium だけがサポートされている
- セカンダリレプリカ(最大 3 つ)が含まれるので, 追加のデータベース冗長を作成せず, 追加料金がかからない
- 障害時のフェールオーバーも自動的に行われる
- [Azure SQL Database の高可用性](https://learn.microsoft.com/ja-jp/azure/azure-sql/database/high-availability-sla?view=azuresql&tabs=azure-powershell#overview)

#### フェールオーバー
- プライマリノードは常にセカンダリノードに変更をプッシュするため, フェールオーバー先のセカンダリノードは常に完全な状態になっている

#### 透過的なデータ暗号化に必要なキー
- RSA または RSA HSM
- キー長は 2048 または 3072 バイト

### SQL Managed Instance
- 共通言語ランタイム(CLR: Common Language Runtime)がサポートされている
  - SQL Database ではサポートされない
- 最新の SQL Server との互換性が備わっているので変更が少なく済む

### VM 上の SQL Server の推奨事項(キャッシュポリシー)
- トランザクションログディスク: なし
- データディスク: 読み取り専用


## Azure Storage Account
- ストレージアカウントの最大容量は 5 PiB

### BLOB
#### 階層型名前空間
- Data Lake Storage Gen2 と同じ?
- 汎用 v2, Premium ブロック BLOB で構成できる

#### 不変ストレージ
- WORM(Wtire Once, Read Many) 状態で保存できる
- 汎用 v2 または Premium ブロック BLOB でサポート
- "時間ベースのアイテム保持ポリシー", "訴訟ホールドポリシー" がある

#### Premium ブロック BLOB
- 最も回復力を高められるのは ZRS
  - GRS はサポートされていない


## Cosmos DB
- 整合性レベルのうち, 複数リージョンにまたがる構成の場合には"厳密"の SLA は対象外
  - 複数リージョンの場合は有界整合性制約が最も強力な整合性レベルになる
- スループットやレイテンシ, 月間稼働時間に関する SLA が提供されている

## Azure Time Series Insights
- 時系列データ向けフルマネージドサービス
- > IoT Hubまたは、Event Hubからストリーミングデータを受け取り、ほぼリアルタイムでデータを処理、保存、クエリ、視覚化することができます。

## Azure Data Factory
- データ連携ツール
- オンプレミスからストレージアカウントへの連携も可能


# インフラストラクチャソリューション
## 仮想ネットワーク
- サイト間 VPN を作成する場合, オンプレミスと VNet のアドレス空間が重複してはいけない
- ゲートウェイサブネット `GatewaySubnet` には /29 以上のアドレス空間が必要(/27 以上が推奨される)

### Azure Bastion
- HTTPS(443) で Vnet に接続し, RDP/SSH(3389/22) で仮想マシンに接続

## Azure Batch
- 大規模な並列コンピューティングやハイパフォーマンス コンピューティング (HPC) のバッチ ジョブを効率的に実行するためのサービス


## Azure Migrate
- オンプレミスから Azure への移行をサポート
- 評価ツールからサイズの推定やコストの見積もりができる


## プライベートリンク
- ExpressRoute で PaaS のリソースにアクセスするにはプライベートエンドポイントを使用
- プライベート IP アドレスで接続できるようになる


## Azure Function
- C#, JS/TS, Java, Python(Linux のみ), PowerShell Core がランタイムとしてサポートされている
- 最大インスタンス数は Win: 200, Linux: 100(従量課金)
- プランは以下
  - 従量課金
    - タイムアウト期間が最大で10分
    - 受信 IP の制限は可能
  - Premium
    - **Vnet 接続をサポート**(ネットワークインジェクションを有効にする)
    - インスタンスが常にウォーム状態
    - タイムアウトは無制限
  - 専用
    - 手動スケーリング可能
- **ログ書き込みは不可**

## API Management
- ロジックアプリをバックエンドの API に設定可能
- rate-limit ポリシーを設定可能
  - 指定期間あたりの呼び出しレートを指定数に制限する
  - 超えた分は `429 Too Many Request` を返す

## Microsoft Defender for Cloud
### Just-In-Time VM アクセス (JIT)
- 一時的に RDP や SSH を許可する


## 負荷分散
- [負荷分散サービスの使い分け](https://learn.microsoft.com/ja-jp/azure/architecture/guide/technology-choices/load-balancing-overview)

### Azure Front Door
- WAF, TLS 終端, URL ベースのルーティング, Cookie ベースのセッションアフィニティ機能 などがサポートされる
- [レート制限](https://learn.microsoft.com/ja-jp/azure/web-application-firewall/afds/waf-front-door-rate-limit)をサポート
  - > ソケット IP アドレスからの異常に高いレベルのトラフィックを検出してブロックできます
  - 
- グローバルサービス
  - **リージョン障害から保護可能**


# ビジネス継続性ソリューション
## 仮想マシン
### 専用ホスト
- 複数の仮想マシンをホストする物理サーバーを提供する
- 可用性ゾーンごとに作成する必要がある


## Site Recovery
- 復旧計画を作成することでフェールオーバーを自動化できる


# ID, ガバナンス, および監視ソリューション
## Microsoft Entra ID
### アクセスレビュー
> ユーザーのアクセス許可を定期的にレビューし、適切なユーザーのみが継続的なアクセス権を持っていることを確認することができます。
- レビューされなかった場合のアクションも設定可能
  - アクセス許可を自動で削除する, など

### 管理グループ
- **同じテナントを信頼するサブスクリプション**をまとめて管理できる
  - 異なるサブスクリプションを1つの管理グループにまとめることはできない

### Azure Blueprints
- プロジェクトの設計図
  - リソースグループ, ARM テンプレート, ポリシー割り当て, ロール割り当てなどを含む(アーティファクト)
  - ARM テンプレートより上位の概念?
- 管理グループやサブスクリプションに対して保存できる
  - **同じブループリントを別のテナントに適用することは不可**
- 割り当てはサブスクリプション単位で実行することが必要
  - 管理グループ単位で割り当てることはできない

### マネージド ID
- ユーザー割り当てマネージド ID は複数のリソースに関連付けることができる
  - システム割り当ての場合はリソースごと
  - 複数の仮想マシンに紐づけ, まとめてアクセス可能にできる

### Microsoft Entra アプリケーションプロキシ
- > オンプレミスの Web アプリケーションへのセキュリティで保護されたリモート アクセスを提供します。Microsoft Entra ID にシングル サインオンした後、ユーザーは、外部の URL または内部のアプリケーション ポータルから、クラウド アプリケーションとオンプレミス アプリケーションの両方にアクセスできます。
- [アプリケーション プロキシの動作](https://learn.microsoft.com/ja-jp/entra/identity/app-proxy/application-proxy#how-application-proxy-works)
  - アプリケーションへのアクセス時に, プロキシによって Entra のサインインページに送られる
  - サインインに成功すると, ユーザーにトークンが送信される
  - トークンをプロキシを介してアプリケーションプロキシコネクタに送信する
  - コネクタはユーザーの代わりに必要な認証を実行
  - コネクタがアプリケーションに要求を送信
  - コネクタとプロキシ経由で通信する

### Privileged Identity Management (PIM)
- Premium P2 ライセンスが必要
- 下記の機能あり
  - 期限付きのアクセス権の割り当て
  - 特権ロールがアクティブ化されたときに通知を受ける
  - アクセスレビューの実施
  - ロールに関するアクティビティを表示


## Azure Key Vault
- 別の Vault への移行に必要な権限は下記
  - 移行元のシークレットに対する Get 権限
    - 全てのシークレットを取得する List 権限もある
  - 移行先のシークレットに対する Set 権限


## Monitor
### Log Analytics に保存されるログ
- Event: Win のイベントログ
- Syslog: Linux のイベントログ
- Perf: パフォーマンスカウンター
- Heartbeat: エージェントの正常性を示すログ