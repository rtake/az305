# IaaS(VM) 向けの高可用性およびディザスターリカバリー
- SQL Server に備わっている以下の機能を使用して可用性を高めることができる
  - Always On フェールオーバークラスターインスタンス
  - Always On 可用性グループ(AG)
    - データおよびトランザクションを保護する
  - ログ配布
    - データおよびトランザクションを保護する

## Always On フェールオーバークラスターインスタンス(FCI)
- SQL Server のインスタンス全体(バイナリ, ログイン, SQL Server エージェントジョブ, データベース, ...)を保護できる
- 現在のノードが使用できなくなると別のノードでインスタンス全体が再起動される
- SQL Server のインストール時に構成する(途中で変換することはできない)

## Site Recovery
- 異なる**リージョン**に VM をレプリケートする
- RTO は 15分未満, RPO は 1時間
  - [https://learn.microsoft.com/ja-jp/azure/site-recovery/site-recovery-sql](https://learn.microsoft.com/ja-jp/azure/site-recovery/site-recovery-sql)

## アーキテクチャの例
### 単一リージョンの高可用性の例
- 高可用性のみが必要で, ディザスターリカバリーは不要なケースは **Always On 可用性グループ**を選択する
- **Always On フェールオーバークラスターインスタンス** も選択可能

### ディザスターリカバリーの例
1. 複数リージョンにまたがった Always On 可用性グループ
2. 分散可用性グループ: 複数の Always On 可用性グループ をデプロイする
3. ログ配布
4. Site Recovery

# PaaS 向けの高可用性およびディザスターリカバリー
## SQL Database
- **アクティブ geo レプリケーション**と**自動フェールオーバーグループ**が選択できる

## SQL Database Managed Instance
- 自動フェールオーバーグループを選択できる

## Database for MySQL
- SLA で 99.99 % の可用性が保証されている
- 障害発生時には**組み込みのフェールオーバーメカニズム**が作動する
- アプリケーションに再試行を組み込む必要あり
- 読み取りレプリカをサポート
  - 処理の負荷をオフロードできる
  - 読み取りレプリカは別リージョンに存在するので可用性も向上できる
- 自動スケールは Basic 以外で利用可能

## Database for PostgreSQL
- MySQL と同様
- Citus によってスケールアウトと高可用性がサポートされる


# Backup
- コンテナーにバックアップコピー, 回復ポイント, バックアップポリシーが保存される
  - Backup コンテナー
    - Backup のみで使用される
    - Database for PostgreSQL, BLOB, ディスクなどがサポートされる
  - Recovery Service コンテナー
    - Backup または Site Recovery で使用できる
    - VM および VM 上の SQL や SAP HANA, Files など
- LRS や GRS を使用できる


# Site Recovery
1. VM のレプリケート: プライマリリージョンからセカンダリリージョンにフェールオーバーする
2. オンプレミスの VM をレプリケートする
3. ワークロードをレプリケートする
4. BCDR のタスクを自動化する
5. ...


# リソース別バックアップ方法
## BLOB のバックアップと回復
- 継続的バックアップが提供される(バックアップのスケジュールが不要)
- 論理的な削除, バージョン管理, ポイントインタイムリストア, リソースロックで保護できる

## Files のバックアップと回復
- 共有スナップショットを取得できる
- REST API やクライアントライブラリ, Azure CLI, PowerShell による手動作成のほか, **Backup とバックアップポリシーを使用して自動化も可能**
- スナップショットのある Files を削除することはできない(スナップショットを削除する必要がある)

### Files の自動バックアップ
- Recovery Container でバックアップを構成すると, ストレージアカウントにスナップショットが保持される(Recovery Container にはデータは送信されない)

## VM のバックアップ
- Backup で実行できる
- VM のスナップショットを作成後, Recovery Container にスナップショットが転送される

## SQL Server のバックアップ
- SQL Database と SQL Managed Instance はバックアップを自動で作成する
  - 既定で GRS BLOB に格納される
- SQL Database の自動バックアップは最大で 35 日間復元できるが, **長期保有ポリシー**を使うと最大で 10 年間, RA-GRS の BLOB にバックアップを格納できる