# Site Recovery
- Backup に比べて RTO(Recovery Time Objective, 目標復旧時間) と RPO(Recovery Point Objective, 目標復旧時点) が短い
- フェールオーバーは Recovery, RTO は Backup

## SQL のバックアップ
- VM 上またはオンプレミス SQL Server の場合, RTO は 15 分未満, RPO は 1 時間未満
- VM 上またはオンプレミス SQL Server の場合には以下も使用可能
  - Always On 可用性グループ
    - データベースは可用性レプリカによってホストされる
      - プライマリ
      - セカンダリ(最大 8 個)
    - 一部データが失われる可能性あり
  - フェールオーバークラスタリング
  - ミラーリング
- その他, SQL Server のディザスターリカバリーも使用できる
  - アクティブ Geo レプリケーション
    - 読み取り可能なセカンダリデータベースを作成する
    - 手動フェールオーバーのみがサポートされる
  - 自動フェールオーバーグループ
    - 別リージョンへのレプリケーションやフェールオーバーを管理できる
    - アクティブ Geo レプリケーションが有効化された場合に使用可能
    - マネージドインスタンス, エラスティックプール, SQL Database でサポート

## モビリティサービス
- レプリケーション時の通信を管理する


# バックアップ
## Resource Guard
- コンテナに対する重要な操作を保護することができる


# Windows Server
## ボリュームシャドウコピーサービス(VSS)
- バックアップ作成時に使用