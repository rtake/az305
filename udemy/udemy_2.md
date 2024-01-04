# データストレージソリューション
## オンライントランザクション処理(OLTP: OnLine Transaction Processing)
- リアルタイムのデータ処理を行うための方法
- 以下のデータストアが OLTP の要件を満たす
  - SQL Database
  - VM 内の SQL Server
  - Database for MySQL
  - Database for PostgreSQL
- **Synapse Analytics ではサポートされない**
- [オンライン トランザクション処理 (OLTP)](https://learn.microsoft.com/ja-jp/azure/architecture/data-guide/relational-data/online-transaction-processing#oltp-in-azure)


## Data Factory
- 受け取ったメッセージの内容に応じて処理を切り替えることができる
- パイプラインに If Condition アクティビティを組み込むことで実現可能
- Functions アクティビティを組み込んで処理を実行させることも可能
- パイプラインを UI で編集可能, JSON で出力もできる
- ファイルサーバーからストレージにデータをコピーするためには, 元サーバーに**セルフホステッド統合ランタイム**をインストールする必要がある


## ストレージ
### 不変ストレージ
- BLOB でサポート

### ライフサイクル管理
- Standard 汎用 v2, Premium ブロック BLOB, Blob Storage(レガシー) でサポートされる
- **Premium ブロック BLOB はアクセス層をサポートしておらず, BLOB の削除だけが可能**

### 冗長性
- Standard 汎用 v2: LRS, ZRS, GRS, GZRS がサポート
- Premium ブロック BLOB: LRS, ZRS
- Premium ページ BLOB: LRS
- Premium ファイル共有: LRS, ZRS

### ファイル
#### トランザクション最適化 Tier
- Premium が提供する待機時間は必要としないが, トランザクション負荷が高いワークロードに使用
  - Premium の場合, ミリ秒以内の待機時間が保証される


## SQL
### データベースファイル
- SQL Server では以下のファイルでデータを管理する
  - プライマリデータファイル(.mdf)
    - データベース内の他のファイルへの参照を含む, データベース自体に関する情報を含む
  - セカンダリデータファイル(.ndf, 省略可能)
    - データの分散に使用する
  - トランザクションログ(.ldf)
    - データベース復旧時に使用するログファイルが格納される
  - https://learn.microsoft.com/ja-jp/sql/relational-databases/databases/database-files-and-filegroups?view=sql-server-ver16#Recommendations
- マネージドディスクまたはページ BLOB に保存する

### SQL Database
#### 購入モデル
- 購入モデルは仮想コア, DTU の2種類
- **仮想コア**は予約割引を受けられる

#### サービスレベル
- 100 TB をサポートできるのは 仮想コアの Hyperscale のみ
- [仮想コア購入モデルを使用した単一データベースに対するリソース制限](https://learn.microsoft.com/ja-jp/azure/azure-sql/database/resource-limits-vcore-single-databases?view=azuresql)

## VM
- 仮想マシンのディスクにはマネージドディスクが推奨される


# インフラストラクチャソリューション
## API Management
- 仮想ネットワーク接続時に外部と内部を選択する
  - 外部モードではパブリック IP アドレスを割り当て, インターネット経由で API にアクセスできるようになる


## キャッシュ
- エンドユーザーの近くにキャッシュを保管するには CDN を使用する
  - 後発の Front Door が推奨されている
- アプリケーションの近くにキャッシュを保管するには Cashe for Redis を使用する
  - データストアのキャッシュはアプリケーションの近くにデプロイする


## ネットワーキング
### Traffic Manager
- 以下のルーティングがサポートされている
  - 優先順位ルーティング
    - プライマリがダウンしたときにセカンダリにルーティングできる
  - 加重トラフィックルーティング
    - 重みづけに基づいて分散する
  - パフォーマンストラフィックルーティング
    - インターネット待機時間テーブルから, 待ち時間が最も短くなるエンドポイントを選択する
  - 地理的トラフィックルーティング


# ビジネス継続性ソリューション
## SQL
### VM 上の SQL Server
- SQL Server IaaS Agent 拡張機能の自動バックアップを利用できる
- BLOB ストレージに保存される
  - コスト最小化のために LRS を使用する(GRS は保存コストが約 2 倍になり, リージョン間のデータ転送料もかかる)


## VM
- 仮想マシンスケールセットはリージョンを跨がないため, 異なる2つのリージョンに VM をデプロイし, Traffic Manager や Front Door で負荷分散することが必要


# ID, ガバナンス, および監視ソリューション
## Entra
### Entra ID
#### Entra Domain Service
- 仮想ネットワーク上にドメインコントローラーを構築する PaaS サービス
- LDAP(Lightweight Directory Access Protocol) による認証をサポート
  - ネットワーク内の情報を一元管理する仕組み
  - Entra は LDAP をサポートしていないので, Entra Domain Service が必要
- Entra 上のユーザーが自動的に同期される

### マネージド ID
#### VM でマネージド ID を取得する
- **IMDS(Instance Metadata Service)** にリクエストを送る
  - `http://169.254.169.254/metadata/identity/` など
  - IMDS のアドレスはルーティング不可能で, VM からのみアクセスできる
- マネージド ID を使う場合は, 内部的には IMDS からアクセストークンを取得している
- アプリからリソースにアクセスしたい場合などに使用する

### Entra ID Protection
- **多要素認証登録ポリシー**によって, MFA のセットアップを要求できる
- 条件付アクセスポリシーと組み合わせて使用する
  - **セキュリティの既定値群と条件付アクセスポリシーは共存できない**


## Monitor
- Log Analytics ワークスペースに収集したログを検索してアラートルールを作成するにはシグナルの種類に"ログ"を選択する
  - SigninLogs テーブルを参照する
  - [チュートリアル:Azure リソースのログ クエリ アラートを作成する](https://learn.microsoft.com/ja-jp/azure/azure-monitor/alerts/tutorial-log-alert#select-a-log-query-and-verify-results)


## ガバナンス
### Blueprints
- 定義と割り当てからなる
  - 定義: 何をデプロイする必要があるか
  - 割り当て: 何がデプロイされたか
- つまり**デプロイされたリソースに接続される**