# ストレージ
## アカウントの種類
- Standard 汎用 v2
- Premium ブロック BLOB
  - "Storage V2 Premium" はこれのこと
- Premium ファイル共有
- Premium ページ BLOB

## BLOB
- ブロック BLOB, ページ BLOB などが選択できる
  - ブロック BLOB は大量のデータを効率的にアップロードするのに最適化されており, 通常はこちらを選択する
  - ページ BLOB は仮想マシンのディスクのプラットフォーム
- Front Door の WAF によるアクセス管理が可能(セキュアな通信だけを通す)

### ポイントインタイムリストア
- ブロック BLOB を以前の状態に復元できる
- 標準の Standard 汎用 v2 ストレージアカウントの**ブロック BLOB のみサポートされる**
- **論理的な削除, 変更フィード, BLOB のバージョン管理が有効になっている必要あり**

### Data Lake Storage Gen2
- ディレクトリで管理できる
- HDFS(Hadoop Distributed File System) によるデータ移行がサポートされている
- アクセス制御リスト(ACL) がサポートされている
  - 粒度の細かい制御が可能
  - RBAC と共存可能

### 不変ストレージ
- **汎用 v2 と Premium ブロック BLOB** でサポート

## Table Storage
- 毎秒 20,000 トランザクションまでサポート
  - より高速な処理が必要な場合は Cosmos DB などを使用する

## アクセス層
- アカウント単位で, デフォルトのアクセス層を設定できる(ホットまたはクール)
- アクセス層の設定はブロック BLOB でのみサポートされる
- Premium ブロック BLOB はライフサイクル管理でレベルを変更できない(削除のみサポート)

## ユーザー委任 SAS
- Entra ID でセキュリティ保護された SAS のこと
- ベストプラクティスは Entra ID を使用した認証だが, SAS が必要な場合はユーザー委任 SAS を使う
- **保存されたアクセスポリシーはユーザー委任 SAS ではサポートされない**
- **BLOB でのみサポートされる**

## Storage の暗号化
- BLOB とキューは, クライアント側の暗号化がサポートされる
  - Storage にアップロードする前に(アプリケーション内部で)暗号化
  - .NET, Java, Python でサポートされる
  - Key Vault を使用
- **サービス側の暗号化が推奨される**(クライアントライブラリで脆弱性が検出されたため)

## フェールオーバーとフェールバック
- フェールバックの手順は以下:
  - 事前に GRS を構成しておく
  - フェールオーバー実施
    - セカンダリリージョンが新しいプライマリになる
    - 元のプライマリのデータが削除される
    - **ストレージアカウントが LRS に変換される(geo 冗長が失われる)**
      - **フェールバックのためには再度 geo 冗長を構成しなおす必要がある**
  - フェールバックの実施
- [ストレージ アカウントのカスタマー マネージド フェールオーバーのしくみ](https://learn.microsoft.com/ja-jp/azure/storage/common/storage-failover-customer-managed-unplanned?tabs=grs-ra-grs#the-failover-and-failback-process)


# SQL
## SQL Database
- サーバーレスコンピューティング(1秒ごとの課金)がサポートされている
- General Purpose 以上でオートスケーリングがサポートされる
- **フェールオーバー時のデータ消失を防ぐには Premium または Bussiness Critical が必要**

## SQL Managed Instance
- オンプレミスからの移行が簡便
  - Data Migration Service を使用
- CLR(Common Language Runtime) をサポート
  - .NET Framework の一部で, データベースでストアドプロシージャを実行できる
  - **SQL Database ではサポートされない**
- 最大サイズは 16TB

## 分散トランザクション(エラスティックデータトランザクション)
- 複数のデータベースにまたがるトランザクションを実行する機能
- Azure SQL Database と Azure SQL Managed Instance でサポートされる
- .NET または Transact-SQL(T-SQL) で開発できる
  - **T-SQL はサーバーサイドの処理**
  - **Azure SQL Database では T-SQL はサポートされていない**

## ゾーン冗長
- SQL Database の全モデル(仮想コア)で使用可能
  - DTU の場合は Premium のみ
- SQL Managed Instances ではプレビュー段階
- SQL Databse の方が安価に実現可能


# Database for MySQL - フレキシブルサーバー
- バースト可能, General Purpose, Business Critical の 3レベルが提供される
- ゾーン冗長高可用性を設定可能(バースト可能以上)
  - [Azure Database for MySQL - フレキシブル サーバーのサービス レベル](https://learn.microsoft.com/ja-jp/azure/mysql/flexible-server/concepts-service-tiers-storage#service-tiers-size-and-server-types)
- *リードレプリカを作成することでフェールオーバー時のダウンタイムを最小化できる[要出典]*
  - 手動で構成することはできない?


# Cosmos DB
- Azure Cosmos DB for PostgreSQL で, Geo-reoplication をサポートする DB を構築可能

## Cosmos DB の API
- NoSQL
  - データをドキュメント形式で格納
- MongoDB
  - ドキュメント構造に BSON 形式を使用
- PostgreSQL
- Aoache Cassandra
  - CQL(Cassandra Query Language) を使用
- Apache Gremlin
- Table
  - Table Storage を移行可能


# Synapse Analytics
- 超並列処理(MPP: Massive Parallel Processing)

## パイプライン
- Data Factory に基づいている
- Data Lake Storage Gen2 へのデータ取り込みが可能

## SQL プール
- プール: コンピューティングリソースを抽象化したもので, 以下の三種類がある
  - サーバーレス SQL プール 
  - 専用 SQL プール
    - 分散テーブル構築が可能
    - **計画されたインジェストなどはこちらを使用するのが良い**
  - サーバーレス Apache Spark プール
    - Delta Lake の更新をサポートしているのは **Apach Spark プール**のみ
      - Delta Lake: オープンソースストレージレイヤー

## Azure Synapse Link for SQL
- Azure SQL Database/Server や Cosmos DB のオペレーショナルデータに対するリアルタイム分析が可能


# Data Explorer
- **水平スケーリングをサポート**
  - ペタバイト級のデータの格納も可能
- KQL で操作可能


# Data Share
- 組織外部とのデータの共有
- Blob Storage, Data Lake Storage Gen1/2, SQL Database, Synapse Analytics などでサポートされる


# Analysis Services
- BI 機能との連携