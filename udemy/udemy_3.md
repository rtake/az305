# インフラストラクチャソリューション
## ネットワーキング
### 負荷分散のオプション
- グローバルレベルでの負荷分散は Front Door および Traffic Manager でサポート
  - Application Gateway はリージョンレベル
  - Load Balancer はゾーン冗長(可用性ゾーン全体で高可用性を保証)
- HTTP(S) トラフィックの負荷分散には Front Door および Application Gateway が推奨される
  - DNS ベースの負荷分散には Traffic Manager を使用する
  - ネットワーク層(L4)の負荷分散には Load Balancer が推奨される

### Azure Front Door
- グローバル負荷分散および CDN をはじめとするサイトアクセラレーションサービスを提供
  - Contents Delivery Network: 各地のキャッシュサーバーに Web サイトのコンテンツを保存, リクエストしたユーザーに近い場所のサーバーから配信できるようにする
    - Azure CDN との違い: 特になし?
  - 動的コンテンツのキャッシュ及び配信も可能

### Traffic Manager
- DNS ベースのグローバルロードバランサ
  - ドメインレベルの負荷分散になる
  - TTL が長い場合には変更に時間がかかるため, Front Door ほど高速なフェールオーバーは難しい
- [Traffic Manager の例](https://learn.microsoft.com/ja-jp/azure/traffic-manager/traffic-manager-how-it-works#traffic-manager-example)
  - DNS サーバに CNAME として Traffic Manager のドメイン名を登録
  - クライアントからの要求を解決する際に Traffic Manager が自動的に負荷分散する

### Application Gateway
- L7 負荷分散
  - Web Application Firewall
  - パスベースの負荷分散
  - Cookie ベースのセッションアフィニティ
  - SSL/TLS オフロード
- 負荷分散対象が複数ある場合には**ラウンドロビン**アルゴリズムでリクエスト先が決定される

### 仮想 WAN の SKU について
- Basic SKU でサポートされているのはサイト間 VPN 接続のみ
- ExpressRoute を含めるには Standard が必要

### サービスエンドポイントとプライベートエンドポイント
#### サービスエンドポイント
- **対象の仮想ネットワークのサブネットのみ適用可能**
  - **ExpressRoute や VPN で接続されたオンプレミスのリソースは使用できない**
- 追加料金は発生しない

#### プライベートエンドポイント
- エンドポイントのプライベート IP アドレスに到達できれば接続できるので, ExpressRoute や VPN で接続できる
- 追加費用が発生

## App Service
- **自動スケーリング**, 継続的デプロイ, Entra ID 認証がサポート
  - 自動スケーリングは Standard 以上でサポート
- **リージョン障害から保護するには, 別のリージョンに App Service を構成し, Traffic Manager で負荷分散する**
  - 負荷分散は Front Door でも可

## Logic Apps
- オンプレミスに接続するには, Azure およびオンプレミス環境でオンプレミスデータゲートウェイの作成が必要
  - [オンプレミスの SQL Server に接続する](https://learn.microsoft.com/ja-jp/azure/connectors/connectors-create-api-sqlazure?tabs=consumption#connect-to-on-premises-sql-server)

## SQL
### Azure Database Migration Service
- オンプレミスの SQL Server を Azure SQL Managed Instance, SQL VM, SQL Database に移行できる
- **オンライン移行がサポートされているのは SQL Managed Instance のみ**
- **バックアップとしてストレージアカウントが必要**

### SQL Server Migration Assistant
- SQL Server **以外**から SQL Server への移行をサポートする
- **バックアップファイルのアップロード先としてストレージアカウントが必要**

### Data Migration Assistance
- アセスメント機能と移行機能がある
  - アセスメント: 互換性の問題を検出
  - SQL Managed Instance への移行はサポート外

## Event Hubs
- Event Hubs をトリガーとして Functions を起動し, Cosmos DB に書き出すことが可能

## Entra ID
### アクティビティログ
- ユーザーの作成, ロールの割り当てなどの操作は Entra ID の監査ログ(AuditLogs)に記録される
- ストレージアカウント, Log Analytics, **Event Hubs** への出力が可能(複数可能)
  - **Log Analytics では, Cosmos DB への出力ができない**


## Container Registory
- Linux ベースの Docker コンテナーをホストできる
  - *App Service では不可?*[要検証]


# データストレージ
## SQL
- 予約容量の購入は `予約`から可能
- ソフトウェアアシュアランス付きの SQL Server ライセンスを所有する場合には**ハイブリッド特典**が提供される
  - 割引きになるのは**仮想コア**ベースのモデルのみ

### バックアップのサポート
- Azure SQL Database では自動バックアップのみサポート
  - **手動バックアップはサポートされていない**
- **Azure SQL Managed Instance では手動バックアップの取得が可能**

### SQL Database
#### geo レプリケーション
- 主目的はビジネス継続性ソリューションだが, 以下のシナリオでも使用可能
  - **データベースへのトラフィック負荷を減らす**
  - 最小のダウンタイムでデータベースを他のサーバに移行
  - アプリケーションのアップグレード時にフェールバックコピーを保存しておく
- **アクティブ geo レプリケーション**または**自動フェールオーバーグループ**(後発)で作成できる
  - フェールオーバーグループでは接続文字列が変更されないので管理コストが少なく, 推奨される
  - アクティブ geo レプリケーションでは複数のレプリカやプライマリと同じリージョンでのレプリカの作成がサポートされている
  - **SQL Managed Instance ではサポートされていない**

## Cosmos DB
- 複数のリージョンで同時に書き込みできる
- 複数の整合性レベルをサポートする(上のものほど強い)
  - Strong(厳密)
    - ユーザーが最新のコミット済みの書き込みを読み取ることが保証される
  - Bounded staleness(有界整合性制約)
    - アイテムのバージョン数や遅延時間の制約の範囲内で厳密な整合性が保たれる
  - Session
    - 同じセッションからのクエリは順序が保たれる
  - Consistent prefic(一貫性のあるプレフィックス)
    - 書き込みの順序は保証されるが, 最新のデータではなく過去のある時点までのデータが提供される
  - Eventual(最終的)
    - 最終的には一貫性が保証されるが, 順序が保証されない

## Databricks
- Apache Spark に基づくビッグデータおよび機械学習プラットフォーム
  - Apache Spark: 分散処理フレームワーク
- クラスターで **Azure Data Lake Storage 資格情報(クレデンシャル)パススルー** を有効にすると, そのクラスターで実行するコマンドはストレージにアクセスするためのサービスプリンシパルを構成しなくても, Entra ID で Azure Data Lake Storage にアクセスできるようになる
  - クラスター: コンピューティングリソースと構成の単位
  - **クレデンシャルパススルーの利用には Premium プランが必要**
  - 今後は Unity Catalog でサポートされる?

## Data Factory
- ETL 操作やデータ統合をサポートするサービス

### 統合ランタイム
- 異なるネットワーク環境間のデータ統合をサポート
- Data Factory と Synapse で実装
- ランタイムの種類
  - Azure
  - **セルフホステッド統合ランタイム**
    - クラウドデータストアとプライベートネットワーク内のデータストア間のコピーアクティビティの実行や, オンプレミスまたは仮想ネットワーク内のコンピューティングリソースに対する変換アクティビティのディスパッチがサポートされている
  - Azure-SSIS
    - Server Integration Services


# ID, ガバナンス, 監視
## Monitor
- 診断ログの出力先には以下が選択できる
  - **ストレージアカウント**
  - **Log Analytics ワークスペース**
  - **イベントハブ**

## Log Analytics の保有日数
- デフォルトで30日, 最大で730日に設定可能

## オンプレミスアプリケーションと Entra ID の連携
- 下記の操作を順番に行う
  - Application Proxy をデプロイする(デプロイしていない場合のみ)
  - Windows サーバにコネクタをインストール
  - Entra ID にオンプレミスアプリを追加
  - 条件付きアクセスポリシーの登録
    - MFA を要求するように設定できる

## セキュリティの既定値群
- Entra ID ライセンスの Free レベルでサポート
- 下記の機能を有効化できる
  - すべてのユーザーに対して、多要素認証への登録を必須にする
  - 管理者に多要素認証の実行を要求する
  - 必要に応じてユーザーに多要素認証の実行を要求する
  - レガシ認証プロトコルをブロックする
    - 古いプロトコルによる認証要求がブロックされる
  - Azure portal へのアクセスなどの特権が必要な作業を保護する

## Entra Connect Health
- 同期エラーをメール通知することができる

## SSO プロトコル
- OpenID Connect & OAuth
- SAML
  - AuthnRequest 要素を Entra ID に送信する
- パスワードベース
  - サインインページの URL を入力すると, HTML を解析してフィールドが自動検出される

## Key Vault
- Key Vault のデータはペアになっているリージョンにレプリケーション(セカンダリリージョンにルーティング)される
- リージョン間のフェールオーバーのときには読み取り専用モードになり, 削除などの操作ができなくなる


# ビジネス継続性ソリューション
## VM
### ディスクのバックアップ
- バックアップ作成時には AzCopy ではなく Site Recorvery を使用する
  - コピー後に変更が入ると対応が取れなくなる

## SQL の長期リテンション
- 最大10年のバックアップを保存できる