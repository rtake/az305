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
- 仮想コアでは, 予約割引を受けられる
- DTU は事前構成済み
- サーバーレスは従量課金

## SQL Managed Instance
- 共通言語ランタイム(CLR), SQL Server エージェントなどをサポート
  - SQL Server エージェント: ジョブをスケーリングできる
- アプリケーションの再設計無しでリフトアンドシフト移行が可能
- 仮想コアモードを使用, 最大 CPU コアと最大ストレージを定義できる

## スケーラビリティ
- SQL Database で**動的スケーラビリティ**がサポートされている

### スケールアップ(垂直)
- コンピューティングサイズを変更
- エラスティックデータベースプールで実装する

### スケールアウト(水平)
- データベースを追加または削除して, 容量や全体のパフォーマンスを調整する
- **Elastic Database クライアントライブラリ**で管理する
- スケールアウトの方法

#### シャーディング
- データを分割し複数のデータベースに分散して保管する

####  読み取りスケールアウト
- 読み取り専用ワークロードをオフロードすることで負荷分散する
- SQL Managed Instance では, Bussiness Critical で自動プロビジョニングされる
- SQL Database では, Bussiness Critical と Premium で自動プロビジョニングされる
  - Hyperscale では, セカンダリレプリカが作成されていれば読み取りスケールアウトを使用できる

## 可用性
### General Purpose / Standard
- プライマルレプリカでは, ローカル VM のSSD が一時データベースとして用いられる
- データファイルとログファイルは Premium ブロック BLOB(LRS 冗長) で保存される
- バックアップファイルは Standard 汎用 v2(RA-GRS) に保存される
  - つまりグローバル冗長
- プライマリで障害発生時には, 新しい SQL Server インスタンスが起動する

### Business Critical / Premium
- **Always On 可用性グループ**と同じような方法で可用性を保証
  - プライマリデータベースとセカンダリデータベースのグループを使用する
- データファイルとログファイルはローカル VM の SSD にある
- プライマリで障害発生時には, グループ内のレプリカにフェールオーバーする

### Hyperscale
- SQL Database でのみ使用できる

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