# API Management
## API への認可と認証
- OAuth 2.0 が推奨
- Entra ID で API を保護するには**アプリ登録**が必要

### OAuth 2.0 と Entra ID による認可のワークフロー
- [承認ワークフロー](https://learn.microsoft.com/ja-jp/azure/api-management/api-management-howto-protect-backend-with-aad#authorization-workflow)
  - ユーザーまたはアプリケーションは, バックエンドアプリへのアクセス権を付与するアクセス許可を持つトークンを Entra ID から取得する
  - API Management への API 要求の Authorization ヘッダーにトークンが追加される
  - API Management で `validate-jwt` ポリシーを使用してトークンが検証される


# メッセージング
## Service Bus
### キュー
- 1つのメッセージを処理できるのは1つのコンシューマーに限られる

### トピックとサブスクリプション
- 1つのメッセージを複数のコンシューマーが処理できる
- パブリッシャーは, トピックにメッセージを送信する
- コンシューマーは, トピックのサブスクリプションからメッセージを受信する
  - フィルタリングも可能

## Event Hubs
- データの高スループットな処理とストリームに特化
- データをプルモデルで使用可能にする
  - Event Hubs はイベントを受信するとストリームに追加する
  - サブスクライバーがデータをプルする

### Event Hubs Capture
- Event Hubs で収集したデータを BLOB として永続ディスク(Storage, Data Lake Storage)に格納できる
- Premium 以外のブロック BLOB でサポートされる
- バイナリ(**Avro 形式**)で書き込まれる

## Event Grid
- イベント駆動アプリケーションに向く
- 取り込まれたデータをサブスクライバーにプッシュする


# VM
## 冗長性
- ゾーン冗長以上はサポートされていないので, 複数の VM を別のリージョンに立て, Traffic Manager でトラフィックを分散させるなどのアーキテクチャが必要


# ネットワーキング
## Load Balancer
### Gateway Load Balancer
- NVA を使用する高可用性シナリオに対応する Load Balancer の SKU

## Network watcher
- Traffic Analytics はメトリクスを可視化するサービス
- 特定のトラフィックが到達できるかどうかを検証するには **IP フロー検証**を使う


# Kubernetes Service
## マルチリージョンクラスター
- [マルチリージョン クラスターの AKS ベースライン](https://learn.microsoft.com/ja-jp/azure/architecture/reference-architectures/containers/aks-multi-region/aks-multi-cluster)
  - 複数のリージョンにクラスターをデプロイし, Front Door で負荷分散する
  - 裏側に Application Gateway を立てる